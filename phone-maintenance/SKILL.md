---
name: phone-maintenance
description: Comprehensive Android phone maintenance via ADB — debloat, privacy hardening, permission audit, battery/storage analysis, cache clearing, and performance tuning. Use this skill when the user wants to clean up, optimize, audit, or harden their Android phone. Trigger on phrases like "maintain my phone", "clean up my pixel", "debloat android", "phone optimization", "audit my phone", "ADB maintenance", or any request involving phone storage/battery/permissions/bloatware management.
---

# Phone Maintenance (Pixel / Android 16+)

Comprehensive, repeatable maintenance for a Google Pixel via ADB. Interactive — always ask before destructive changes.

## Environment Detection

Detect the ADB routing method at the start:

```bash
# If running inside WSL2, route through cmd.exe
if grep -qi microsoft /proc/version 2>/dev/null; then
  # WSL2: must use cmd.exe to reach Windows ADB
  ADB_PREFIX='cmd.exe /c "cd C:\ &&'
  ADB_SUFFIX='"'
else
  # Native Linux/Mac: direct adb
  ADB_PREFIX=""
  ADB_SUFFIX=""
fi
```

On WSL2, every ADB command follows this pattern:
```bash
cmd.exe /c "cd C:\ && adb -s <DEVICE_IP:PORT> shell <command> 2>&1"
```

The `cd C:\` is mandatory — WSL's UNC path causes "UNC paths are not supported" errors in cmd.exe.

## Pre-flight

Before any changes:
1. Suggest the user backs up important data (photos, messages) if not already synced
2. After connecting, verify device model and Android version:

```bash
getprop ro.product.model
getprop ro.build.version.release
```

3. Confirm it's a Pixel device on Android 14+. If not, STOP and warn that debloat lists and commands are Pixel-specific and may not be safe on other manufacturers.

## Connection

1. Ask user to enable **Wireless debugging** (Settings > Developer Options > Wireless debugging)
2. User provides the pairing IP:port and code
3. Pair: `adb pair <IP:PORT>` with the code
4. Connect: `adb connect <IP:PORT>` (connection port differs from pairing port)
5. Verify: `adb devices` — should show one device
6. If multiple transports appear, note the correct device serial (IP:PORT) and use `-s` flag for all commands

## Modes

### Quick Mode (targeted)

Present this menu and let the user pick:
1. Cache cleanup
2. Storage analysis
3. Battery drain check
4. Permission audit
5. Notification audit
6. App removal
7. Google debloat
8. Privacy hardening
9. Performance tuning

### Full Maintenance (periodic deep clean)

Run all steps in sequence. Ask confirmation at each destructive step.

---

## Step 1: Malware & Sideload Check

```bash
# List sideloaded apps (installed from outside Play Store)
pm list packages -i | grep -v "com.android.vending" | grep -v "com.google.android"
```

For each sideloaded package:
- Show the installer source
- Flag unknown sources
- Check if it's a known legitimate app (user-installed APK) or suspicious

Report findings. Suspicious indicators:
- Installed from `null` installer with no recognizable package name
- Accessibility service active from unknown app
- Device admin privileges held by unknown app

## Step 2: Battery Drain Analysis

```bash
dumpsys batterystats --charged
```

Parse for:
- Top 10 battery consumers by UID
- Map UIDs to package names via `dumpsys package | grep "userId="`
- Wakelocks held longest
- Alarms firing most frequently
- Network usage by UID

Present findings and suggest:
- Background-restrict heavy drainers: `cmd appops set <pkg> RUN_IN_BACKGROUND deny`
- Battery-restrict via standby bucket: `am set-standby-bucket <pkg> restricted`

**Always ask before restricting** — some apps (VPN, sync, accessibility) need background access.

## Step 3: Storage Analysis & Cache Clearing

### Analysis

```bash
dumpsys diskstats
```

Parse the JSON output for per-app cache and data sizes. Present top consumers.

### Cache Clearing

**Use `pm trim-caches` — NOT `pm clear --cache-only`** (the latter hangs on Android 16 and creates zombie processes):

```bash
pm trim-caches 9999999999999
```

This clears all reclaimable cache system-wide. There is no per-app cache-only command on Android 16+.

For targeted data clearing (wipes ALL app data, not just cache):
```bash
pm clear <package>
```
**Only with explicit user confirmation** — this deletes logins, settings, saved data.

### Leftover Detection

Check for orphaned directories from uninstalled apps:
```bash
ls /sdcard/Android/data/ | while read d; do pm list packages | grep -q "${d##*/}" || echo "ORPHAN: $d"; done
```

Ask before removing orphans.

## Step 4: Unused App Identification & Removal

```bash
# Usage stats for last 30 days
dumpsys usagestats
```

Identify apps with zero launches in 30+ days. Cross-reference with:
- System apps (cannot uninstall, can disable)
- User-installed apps (can uninstall)

Present the list sorted by last-used date. For removal:
```bash
# Third-party apps: full uninstall
pm uninstall <package>

# System apps: disable (reversible)
pm disable-user --user 0 <package>
```

## Step 5: Google Debloat

The following packages are safe to disable on Pixel devices — but **always present the list and let the user choose**. Never batch-disable without confirmation. Some users rely on accessibility features (TalkBack, Switch Access) or Digital Wellbeing.

Safe-to-disable Google system packages on Pixel (Android 16):

| Package | Name | Notes |
|---------|------|-------|
| com.google.android.videos | Google TV | Streaming service |
| com.google.android.projection.gearhead | Android Auto | Car integration |
| com.google.android.apps.tachyon | Google Meet (Duo) | Video calling |
| com.google.android.apps.wellbeing | Digital Wellbeing | Usage tracking |
| com.google.android.apps.magazines | Google News | News aggregator |
| com.google.android.youtube.music | YouTube Music | Unless used for media |
| com.google.android.apps.emojiwallpaper | Emoji Wallpaper | Novelty wallpaper |
| com.google.android.apps.aiicons | AI Icons | AI-generated icons |
| com.google.android.apps.aiwallpapers | AI Wallpapers | AI-generated wallpapers |
| com.google.android.marvin.talkback | TalkBack | Accessibility (keep if needed) |
| com.google.android.accessibility.switchaccess | Switch Access | Accessibility |
| com.google.android.accessibility.soundamplifier | Sound Amplifier | Accessibility |
| com.google.android.uvexposurereporter | UV Exposure Reporter | Pixel weather feature |

**Cannot disable (SecurityException):**
- `com.google.android.devicelockcontroller` — carrier lock enforcement, protected by system

**Proceed with caution:**
- `com.google.ar.core` — ARCore, needed by some camera/maps features
- `com.google.android.apps.wallpaper` — Wallpaper picker (launcher may need it)

Present the list and let user choose which to disable:
```bash
pm disable-user --user 0 <package>
```

## Step 6: Permission Audit

```bash
dumpsys package > /tmp/package_dump.txt
```

Parse and flag questionable permissions:
- **Location** access by non-navigation/weather apps
- **Microphone** access by non-communication apps
- **Camera** access by non-photo apps
- **Contacts/SMS/Call Log** by utility apps
- **BODY_SENSORS** by non-health apps
- **SCHEDULE_EXACT_ALARM** by non-reminder apps

Present findings grouped by risk level. For revocation:
```bash
pm revoke <package> android.permission.<PERMISSION>
```

## Step 7: Notification Spam Audit

```bash
dumpsys notification --noredact
```

Identify apps posting excessive notifications. For blocking:
```bash
# Block ALL notifications from an app
cmd appops set <package> POST_NOTIFICATION ignore
```

Note: Per-channel notification blocking is NOT available via `cmd notification` on Android 16. The only ADB option is all-or-nothing per app.

## Step 8: Privacy Hardening

Apply these settings (ask confirmation as a batch):

Ask user which private DNS provider they prefer:

| Provider | Address | Notes |
|----------|---------|-------|
| AdGuard | `dns.adguard-dns.com` | Ad-blocking, free tier available |
| Cloudflare | `one.one.one.one` | Fast, privacy-focused, no ad-blocking |
| Quad9 | `dns.quad9.net` | Malware-blocking, non-profit |
| NextDNS | `<user-id>.dns.nextdns.io` | Customizable, free tier (300k queries/mo) |
| None | (skip) | Keep current DNS settings |

```bash
# Private DNS (user's choice)
settings put global private_dns_mode hostname
settings put global private_dns_specifier <chosen-provider>

# Disable telemetry/usage reporting
settings put global multi_cb 0
settings put secure send_action_app_error 0

# Disable background scanning
settings put global ble_scan_always_enabled 0
settings put global wifi_scan_always_enabled 0
settings put secure nearby_sharing_enabled 0

# Disable usage & diagnostics
settings put global upload_apk_enable 0
```

Explain what each does before applying.

## Step 9: Performance Optimization

Ask user preferred animation speed:
- **0.5x** — snappy without being jarring (recommended)
- **0x** — instant, no animations at all
- **1x** — keep Android defaults
- **Skip** — don't change animations

```bash
settings put global window_animation_scale <value>
settings put global transition_animation_scale <value>
settings put global animator_duration_scale <value>

# DEX optimization (compiles all apps for speed — takes a few minutes)
cmd package compile -m speed-profile -a -f
```

Note: DEX compilation takes 2-5 minutes and device may feel warm. Run it last.

## Step 10: Health Verification

After all changes, verify nothing broke:

```bash
# Launcher working
cmd shortcut get-default-launcher

# Recent apps functional
am start -a com.android.systemui.recent

# Check running processes for crashes
dumpsys activity processes | grep "crash"

# Storage status
df /data

# Battery health
dumpsys battery
```

Report final state:
- Storage: before → after (% used)
- Apps removed/disabled count
- Permissions revoked count
- Privacy settings applied

## Important Caveats

- **Always ask before destructive changes** — the user controls what gets removed/disabled
- **`pm disable-user --user 0` is reversible** — re-enable with `pm enable <package>`
- **`pm uninstall` for user apps is permanent** — data is lost unless backed up
- **Zombie process risk**: If any command hangs for >30 seconds, it likely hit an unsupported flag. Kill it and try an alternative approach.
- **Post-maintenance**: Suggest user turns off wireless debugging when done (security best practice)

## Reversal

If something breaks after debloat:
```bash
# Re-enable a disabled system app
pm enable <package>

# Restore a permission
pm grant <package> android.permission.<PERMISSION>

# Reset animations to default
settings put global window_animation_scale 1.0
settings put global transition_animation_scale 1.0
settings put global animator_duration_scale 1.0

# Reset DNS to auto
settings put global private_dns_mode opportunistic
settings delete global private_dns_specifier
```
