# Self-Hosting Runbook — PolyPilot Remote Control

> **Goal:** Run PolyPilot on your PC as a headless agent server, then steer your AI squads from an Android phone — on local WiFi at home or via mobile data from anywhere.

## Quick-Start Decision Tree

| Where are you? | What you need | Guide |
|----------------|---------------|-------|
| Home (same WiFi) | LAN-only, simplest setup | [LAN Setup](lan-setup.md) |
| Outside (mobile data) | Internet tunnel to reach your PC | [Internet Tunnel](internet-tunnel.md) |
| Both home & outside | Smart URL auto-switching | [Dual-Mode Setup](dual-mode-setup.md) (recommended) |
| VPN user (Tailscale) | Secure overlay network | [Tailscale Setup](tailscale-setup.md) |

## Architecture At-a-Glance

```
┌──────────────────────────┐         WebSocket (port 4322)        ┌─────────────────┐
│  Your PC (Desktop Host)  │ ◄──────────────────────────────────► │  Android Phone   │
│                          │          state-sync protocol         │  (Remote Mode)   │
│  PolyPilot (Mac/Win/Lin) │                                      │                  │
│  ├─ CopilotService       │   ┌────────────────────────┐         │  PolyPilot APK   │
│  ├─ WsBridgeServer:4322  │◄──┤ DevTunnel / Tailscale  │────────►│  ├─ WsBridgeClient│
│  ├─ Copilot CLI (SDK)    │   │ (internet relay)       │         │  └─ Remote Mode   │
│  └─ Agent Squads (.squad)│   └────────────────────────┘         └─────────────────┘
└──────────────────────────┘
```

**Key insight:** The desktop app runs Copilot sessions and exposes a WebSocket bridge on port **4322**. The Android app connects in **Remote Mode** and acts as a thin client — it sends prompts and receives live streaming updates, but all AI processing happens on your PC.

## Guides (Progressive Disclosure)

| # | Document | What it covers |
|---|----------|----------------|
| 1 | [Prerequisites](prerequisites.md) | .NET SDK, Copilot CLI, Android APK, network basics |
| 2 | [Building & Running the Desktop Server](build-desktop.md) | Building from source for Mac/Windows/Linux(Android-target) |
| 3 | [Building the Android Client](build-android.md) | APK build, sideload, and first launch |
| 4 | [LAN Setup](lan-setup.md) | Direct Sharing over local WiFi |
| 5 | [Internet Tunnel](internet-tunnel.md) | DevTunnel or ngrok for mobile-data access |
| 6 | [Dual-Mode Setup](dual-mode-setup.md) | LAN + tunnel with auto-switching |
| 7 | [Tailscale Setup](tailscale-setup.md) | Zero-config VPN alternative |
| 8 | [Squad Management](squad-management.md) | Creating and steering agent squads from your phone |
| 9 | [Security Hardening](security.md) | Passwords, tokens, firewall rules |
| 10 | [Troubleshooting](troubleshooting.md) | Common issues and fixes |

## How It Works (30-Second Version)

1. **Desktop** runs PolyPilot with Copilot CLI in Persistent mode (survives restarts).
2. **Desktop** starts the WsBridge server on port 4322 with a password.
3. **Phone** installs the PolyPilot APK and scans a QR code (or enters URL manually).
4. **Phone** connects in Remote mode — all sessions, chats, and squads appear live.
5. You type prompts and steer agents from your phone; the desktop executes them.

## Common Gotcha

- For same-WiFi setup, enter credentials in **LAN URL** and **LAN Token** fields on phone settings.
- Use **Remote URL** and **Remote Token** only for tunnel/internet access.
- If you see `401 when 101 expected`, the bridge is reachable but WebSocket auth failed. Use **Save & Reconnect** on desktop and phone to resync active token state.

---

**Next step:** [Prerequisites](prerequisites.md) →
