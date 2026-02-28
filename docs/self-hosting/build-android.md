# Building the Android Client

> Build the PolyPilot APK and install it on your Android phone.

## Build the APK

From the repo root:

```bash
cd PolyPilot
dotnet build -f net10.0-android
```

The debug APK is at:
```
bin/Debug/net10.0-android/com.microsoft.PolyPilot-Signed.apk
```

## Install via USB

Connect your phone via USB with **USB debugging** enabled:

```bash
# Build and deploy in one step (recommended — handles Fast Deployment correctly)
dotnet build -f net10.0-android -t:Install
```

Then launch:
```bash
adb shell am start -n com.microsoft.PolyPilot/crc64ef8e1bf56c865459.MainActivity
```

> **Important:** Use `dotnet build -t:Install`, not bare `adb install`. Fast Deployment pushes assemblies to `.__override__` on the device, which `adb install` doesn't handle.

## Install via WiFi ADB

> **Warning on macOS:** WiFi ADB requires special handling. Do NOT run `adb kill-server` (it wipes TLS pairing keys). Build with `-p:EmbedAssembliesIntoApk=true` for WiFi deploys because Fast Deployment APKs are stale over WiFi.

```bash
# WiFi deploy (embed assemblies for reliability)
dotnet build -f net10.0-android -p:EmbedAssembliesIntoApk=true -t:Install
```

## Sideload Without a Build Machine

If you've pre-built the APK and want to transfer it:

1. Copy `com.microsoft.PolyPilot-Signed.apk` to your phone (USB, cloud drive, etc.)
2. On the phone, open the APK file
3. Allow "Install from unknown sources" if prompted
4. Install and open

## First Launch on Android

1. Open PolyPilot.
2. You'll see the **Welcome screen** — the app is in Remote mode by default on mobile.
3. Tap **Settings** (gear icon).
4. You need a **Remote URL** — this comes from your desktop. Continue to the connection guides:
   - [LAN Setup](lan-setup.md) for home WiFi
   - [Internet Tunnel](internet-tunnel.md) for mobile data
   - [Dual-Mode Setup](dual-mode-setup.md) for both

## QR Code Connection (Easiest)

The fastest way to connect is via QR code:

1. On your **desktop**, go to Settings → "Direct Sharing" or "DevTunnel" section.
2. A QR code will appear containing the connection URL and token.
3. On your **phone**, go to Settings → tap the **QR scan button**.
4. Scan the QR code — the URL and token are auto-filled.
5. Tap **Save & Reconnect**.

The QR code contains a JSON payload:
```json
{
  "url": "https://your-tunnel.devtunnels.ms",
  "token": "tunnel-access-token",
  "lanUrl": "http://192.168.1.100:4322",
  "lanToken": "your-server-password"
}
```

When both `url` and `lanUrl` are present, the phone auto-switches between them based on network — see [Dual-Mode Setup](dual-mode-setup.md).

> **Backward compatibility:** If only Direct Sharing is running (no tunnel), the QR code also includes `url` and `token` aliases pointing to the LAN address. This ensures older app versions that only read `url`+`token` can still connect.

## Package Details

| Field | Value |
|-------|-------|
| Package name | `com.microsoft.PolyPilot` |
| Launch activity | `crc64ef8e1bf56c865459.MainActivity` |
| Min Android | 7.0 (API 24) |
| Target framework | `net10.0-android` |

## iOS Note

iOS builds are also supported (`dotnet build -f net10.0-ios`) but require a Mac with Xcode. Requires iOS 15.0+. The iOS app works identically in Remote mode. See the main project [copilot-instructions](../../.github/copilot-instructions.md) for iOS-specific build and deploy commands.

## Mobile-Only Behavior

On Android, PolyPilot:
- Only supports **Remote** connection mode (no local Copilot CLI)
- Hides desktop-only menu items (Terminal, VS Code, Fix with Copilot)
- Shows a "Report Bug" link that opens GitHub in the browser
- Displays processing status (elapsed time + tool round count) synced over the bridge

---

**Next:** [LAN Setup](lan-setup.md) →
