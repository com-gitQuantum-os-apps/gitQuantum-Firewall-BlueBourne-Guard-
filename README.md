# gitQuantum Firewall+BlueBourne Guard — Full Specification (Official gitQuantum Brand)

**Brand:** gitQuantum — clean, professional, technology-forward identity emphasizing security, reliability, and offline-first autonomy.

**One-line:** gitQuantum Firewall+BlueBourne Guard is an offline-capable, branded firewall mobile app inspired by Nordic Semiconductor's BLE tools and leading Android firewall apps, designed for privacy, control, and developer-friendly extensibility.

---

## Authorship

* **Lead Author / Creator:** Zaccharin Thibodeau ("Zac") — Creator of gitQuantum
* **Development Team:** gitQuantum Engineering Team
* **UX & Branding:** gitQuantum Design Team
* **QA & Testing:** gitQuantum QA Division
* **Technical Contributors / Advisors:** Nordic Semiconductor reference contributors, Open Source Firewall community (NetGuard, AFWall+)

---

## gitQuantum Branding Details

* **Logo:** Minimalist shield icon with stylized "GQ" monogram for launcher and marketing.
* **Colors:** Primary: #1F1F3B (Deep Indigo), Accent: #00BFFF (Quantum Blue), Warning: #FF4500 (Alert Red)
* **Typography:** Modern sans-serif, legible (Roboto / Inter for Android)
* **Naming Convention:** All modules prefixed with `gitquantum` (e.g., `gitquantum-firewall-core`, `gitquantum-tools`)
* **Tagline:** "Full Control. Offline Security. Quantum Reliability."
* **Visual Style:** Flat design, clear network indicators, minimalistic toggles, subtle animations.

---

## Purpose & Goals (gitQuantum)

* Establish gitQuantum as a trusted, secure firewall brand.
* Provide granular per-app, per-network control with offline-first reliability.
* Ensure consistent branded experience across dashboards, logs, rules, and tools.

---

## UX Branding

* All screens feature gitQuantum logo and Quantum Blue highlights.
* Bottom navigation uses branded icons and color accents.
* Lockdown mode highlighted in Alert Red with shield badge.
* DFU and Diagnostics tools match gitQuantum visual identity.

---

## Developer & Module Branding

* Packages: `com.gitquantum.firewall.*` (core engine), `com.gitquantum.firewall.ui` (UI), `com.gitquantum.firewall.tools` (utilities).
* Internal deep links: `gitquantum://dashboard`, `gitquantum://app/{package}`.
* Logs and exports optionally include gitQuantum watermark for enterprise debugging.

---

## Tab List (Bottom Navigation)

1. **Dashboard** — Summary, connection status, quick toggles.
2. **Apps** — Per-app rules, usage stats, allow/block toggles.
3. **Rules** — Rule editor: app/IP/domain rules, schedule, lockdown.
4. **Network** — Live connection monitor, LAN scanner.
5. **Logs** — Connection logs, alerts, exportable JSON/CSV.
6. **Tools** — Offline DFU, diagnostic suite.
7. **Settings** — VPN/root mode, backup, privacy, updates.

---

## Link List (Internal & External)

* **Internal deep links:** `gitquantum://dashboard`, `gitquantum://app/{package}`, `gitquantum://rules/new`, `gitquantum://logs/{sessionId}`
* **External:** Offline Nordic DFU docs, Privacy Policy, optional diagnostic upload endpoint.

---

## Activity List (Android Activities & Fragments)

* `MainActivity` (hosts bottom nav)
* `DashboardFragment`
* `AppsFragment` / `AppDetailActivity`
* `RuleListFragment` / `RuleEditorActivity`
* `NetworkMonitorActivity`
* `LogViewerActivity` / `LogSessionFragment`
* `ToolsFragment` / `DFUActivity`, `DiagnosticsActivity`
* `SettingsActivity` / `AboutActivity`

---

## Service List

* `VpnFilterService` (extends `VpnService`) — local VPN engine
* `IptablesBridgeService` (rooted devices) — iptables backend
* `ConnectionObserverService` — monitors connectivity, applies rules
* `BackgroundCleanerService` — log rotation, backups, purges
* `DfuManagerService` — offline DFU transfers for IoT

---

## Package List

* `com.gitquantum.firewall.core` — engine & rule evaluator
* `com.gitquantum.firewall.ui` — activities & fragments
* `com.gitquantum.firewall.net` — VPN adapters & network utils
* `com.gitquantum.firewall.storage` — database, export/import, encryption
* `com.gitquantum.firewall.tools` — DFU, diagnostics
* Third-party: Kotlin coroutines, Room, Jetpack Navigation, Okio/OkHttp

---

## Path List (Internal Storage)

* Config: `/data/user/0/com.gitquantum.firewall/files/config.json` (encrypted)
* Logs: `/data/user/0/com.gitquantum.firewall/files/logs/`
* Backups: `/storage/emulated/0/Download/gitQuantumBackups/`
* Database: `/data/user/0/com.gitquantum.firewall/databases/rules.db`

---

## Permission List

* Manifest: `BIND_VPN_SERVICE`, `INTERNET`, `ACCESS_NETWORK_STATE`, `FOREGROUND_SERVICE`, optional `WRITE_EXTERNAL_STORAGE` / `READ_EXTERNAL_STORAGE` (legacy)
* Runtime & Security: Keystore encryption, root-mode requests `ACCESS_SUPERUSER` only with warnings

---

## Component List

* **BroadcastReceivers:** `ConnectivityChangeReceiver`, `BootCompletedReceiver`, `PackageAddedReceiver`
* **ContentProviders:** `LogProvider` (optional)
* **WorkManager Jobs:** Backups, purges, update checks
* **Native Components:** Optional NDK packet parsing utilities

---

## Rule Model (JSON)

```json
{
  "ruleId":"uuid",
  "type":"app|ip|domain|global",
  "selector":"com.example.app or 192.168.0.0/16 or example.com",
  "networks":["WIFI","MOBILE","LAN"],
  "action":"ALLOW|BLOCK|LIMIT",
  "schedule":{"from":"00:00","to":"23:59","days":[1,2,3,4,5,6,7]},
  "logging":true
}
```

---

## Logging & Diagnostics

* Connection logs with session IDs, timestamps, app package, destination IP/domain, action applied
* Exportable JSON/CSV, nRF Logger style sessions

---

## AndroidManifest.xml Skeleton

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.gitquantum.firewall">

    <uses-permission android:name="android.permission.BIND_VPN_SERVICE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application
        android:label="gitQuantum Firewall+BlueBourne Guard"
        android:icon="@mipmap/ic_launcher">

        <service android:name=".services.VpnFilterService"
                 android:permission="android.permission.BIND_VPN_SERVICE" />

        <activity android:name=".ui.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>
```

---

## VpnFilterService Kotlin Skeleton

```kotlin
class VpnFilterService : VpnService() {
    private var vpnInterface: ParcelFileDescriptor? = null

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        setupVpn()
        return START_STICKY
    }

    private fun setupVpn() {
        val builder = Builder()
        builder.addAddress("10.0.0.2", 32)
        builder.addRoute("0.0.0.0", 0)
        vpnInterface = builder.establish()
    }

    override fun onDestroy() {
        vpnInterface?.close()
        super.onDestroy()
    }
}
```

---

## Room DB Schema Example

```kotlin
@Entity(tableName = "rules")
data class RuleEntity(
    @PrimaryKey val ruleId: String,
    val type: String,
    val selector: String,
    val networks: String,
    val action: String,
    val scheduleJson: String,
    val logging: Boolean
)

@Dao
interface RuleDao {
    @Query("SELECT * FROM rules") fun getAll(): List<RuleEntity>
    @Insert fun insert(rule: RuleEntity)
    @Delete fun delete(rule: RuleEntity)
}
```

---

## Mockup Flow (Markdown Wireframe)

```
[Dashboard]
  |-> [Apps List] -> [App Detail -> Rule Toggle]
  |-> [Rules List] -> [Rule Editor]
  |-> [Network Monitor]
  |-> [Logs Viewer]
  |-> [Tools] -> [DFU / Diagnostics]
  |-> [Settings]
```

---

## Marketing & Store Presence

* **Play Store / APKPure Listing:** Branded screenshots, offline-first, security features, per-app rules
* **Website / Support:** gitQuantum.store, FAQ, offline setup guides
* **Support Email:** [support@gitQuantum.store](mailto:support@gitQuantum.store)
* **Brand Consistency:** All communications, alerts, in-app messages reflect gitQuantum identity

---

*All technical specifications, tab lists, service lists, database schema, mockups, Kotlin skeletons, and branding are fully integrated under the gitQuantum identity, ready for app store submission.*
