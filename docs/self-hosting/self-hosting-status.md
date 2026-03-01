# Self-Hosting Setup Status

> **Updated by:** Copilot Agent (async coordination file)
> **Last updated:** 2026-03-01 11:05

## Overall Progress

| Step                              | Status | Notes                                                                 |
| --------------------------------- | ------ | --------------------------------------------------------------------- |
| 1. Prerequisites Check            | ✅ Done | .NET 10.0.103, Copilot CLI 0.0.420, maui-android/windows workloads OK |
| 2. Build Desktop Server (Windows) | ✅ Done | Built successfully                                                    |
| 3. Build Android APK              | ✅ Done | APK ready for sideload (~21 MB)                                       |
| 4. Launch Desktop App             | ✅ Done | PID 71664, Copilot server port 4321                                   |
| 5. Verify Copilot Working         | ✅ Done | Bridge responds "WsBridge OK" on localhost:4322                       |
| 6. Enable Direct Sharing          | ✅ Done | Password: `polypilot2026`, auto-starts on launch                      |
| 7. Start DevTunnel                | ✅ Done | Tunnel `neat-pond-kmhjr5g` running, URL verified                      |
| 8. End-to-End Verification        | ✅ Done | Tunnel → Bridge → "WsBridge OK" confirmed                             |

## 🎉 SETUP COMPLETE — Connection Info

### APK Location
```
C:\Workplace\Agents\PolyPilot\PolyPilot\bin\Debug\net10.0-android\com.microsoft.PolyPilot-Signed.apk
```

### Phone Connection (Manual Entry)
Enter these in the Android app's Settings page:

| Field                    | Value                                                             |
| ------------------------ | ----------------------------------------------------------------- |
| **Remote URL**           | `https://pwqr5df9-4322.asse.devtunnels.ms`                        |
| **Remote Token**         | *(use the tunnel access token below or scan QR from desktop app)* |
| **LAN URL** (optional)   | `http://192.168.100.221:4322`                                     |
| **LAN Token** (optional) | `polypilot2026`                                                   |

> **Easiest method:** Open PolyPilot on desktop → Settings → the QR code should appear. Scan it from the phone app.

### Tunnel Access Token (expires in ~24 hours)
```
eyJhbGciOiJFUzI1NiIsImtpZCI6IjI0Q0E0QkJEN0ZBNkNCMzY1QUQ4NEQ1MTA5MkRERkRGRjM3M0EyM0EiLCJ0eXAiOiJKV1QifQ.eyJjbHVzdGVySWQiOiJhc3NlIiwidHVubmVsSWQiOiJuZWF0LXBvbmQta21oanI1ZyIsInNjcCI6ImNvbm5lY3QiLCJleHAiOjE3NzI0MjM5ODYsImlzcyI6Imh0dHBzOi8vdHVubmVscy5hcGkudmlzdWFsc3R1ZGlvLmNvbS8iLCJuYmYiOjE3NzIzMzY2ODZ9.T3DpVAjHjP5U0QLgnpMouM64KyYb2T05l_X6obr4Bdb-AWY5sdvA8XsyjTEw9fJ7__NcQY1G8fN4TzjFCxAhWQ
```

### Desktop Server Password
```
polypilot2026
```

## Resolved Issues

### Issue #1: DevTunnel login token expired — ✅ RESOLVED
- User re-authenticated via `devtunnel user login -g`

### Issue #2: LAN mode needs admin elevation — ✅ RESOLVED
- Bridge returns 400 for non-localhost (LAN IP) requests
- Not needed for DevTunnel mode (which routes through localhost)
- If LAN mode desired, run in elevated PowerShell:
  ```powershell
  netsh http add urlacl url=http://+:4322/ user=Everyone
  New-NetFirewallRule -DisplayName "PolyPilot Bridge" -Direction Inbound -LocalPort 4322 -Protocol TCP -Action Allow
  ```

## What's Running

| Component         | Status    | Details                                   |
| ----------------- | --------- | ----------------------------------------- |
| PolyPilot Desktop | ✅ Running | PID 71664                                 |
| Copilot Server    | ✅ Running | Port 4321, persistent mode                |
| WsBridge          | ✅ Running | Port 4322, password-protected             |
| DevTunnel         | ✅ Running | `neat-pond-kmhjr5g`, auto-starts with app |

## Configuration

- **Settings:** `~/.polypilot/settings.json`
- **Connection mode:** Persistent
- **Copilot port:** 4321
- **Bridge port:** 4322
- **Tunnel ID:** `neat-pond-kmhjr5g`
- **Tunnel URL:** `https://pwqr5df9-4322.asse.devtunnels.ms`
- **Auto-start tunnel:** Yes
- **Direct Sharing:** Enabled
- **LAN IP:** 192.168.100.221

## Steps Log

1. Prerequisites verified: .NET 10.0.103, Copilot CLI 0.0.420, all workloads present
2. Desktop build: `dotnet build -f net10.0-windows10.0.19041.0` — succeeded
3. Android build: `dotnet build -f net10.0-android` — succeeded, APK ready
4. Settings configured: Persistent mode, Direct Sharing on, password set
5. Devtunnel binary copied to `%LOCALAPPDATA%\Microsoft\DevTunnels\` for app discovery
6. Desktop app launched: Copilot connected, bridge running
7. DevTunnel created and hosting on port 4322
8. Access token issued (24h expiry)
9. End-to-end test: `curl https://pwqr5df9-4322.asse.devtunnels.ms/` → "WsBridge OK"
