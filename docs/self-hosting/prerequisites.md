# Prerequisites

> Everything you need before starting. Check each box before proceeding.

## Software

### On Your PC (Desktop Host)

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| **.NET SDK** | 10.0+ | `dotnet --version` |
| **GitHub Copilot CLI** | Latest | `copilot --version` |
| **Git** | Any | `git --version` |
| **Android SDK** (if building APK) | API 24+ | `dotnet workload list` should show `android` |

Install .NET MAUI workloads:
```bash
dotnet workload install maui-android
# On macOS, also:
dotnet workload install maui-maccatalyst maui-ios
```

### On Your Android Phone

- Android 7.0+ (API 24)
- The PolyPilot APK (you'll build this — see [Building the Android Client](build-android.md))

### Copilot CLI Authentication

The desktop app needs an authenticated Copilot CLI:

```bash
# Authenticate with GitHub (one-time)
# Auth commands may vary by CLI version. Check `copilot --help` for available subcommands.
copilot auth login

# Verify
copilot auth status
```

> **Note:** The Copilot CLI runs as a background server in Persistent mode. It must be authenticated before the desktop app launches.

## Network Requirements

### Home (LAN) Scenario

| What | Port | Direction |
|------|------|-----------|
| WsBridge server | **4322** (TCP) | Phone → PC |

Both devices must be on the **same WiFi network**. Your PC's local IP (e.g., `192.168.1.x`) must be reachable from the phone.

### Internet (Mobile Data) Scenario

Choose one:

| Method | Requirements |
|--------|-------------|
| **DevTunnel** (Microsoft) | `devtunnel` CLI installed + Microsoft/GitHub account |
| **ngrok** | `ngrok` CLI installed + ngrok account |
| **Tailscale** | Tailscale installed on both PC and phone |
| **Port forwarding** | Router admin access — **not recommended** (security risk) |

#### Installing DevTunnel CLI

```bash
# macOS
brew install --cask devtunnel

# Linux (curl)
curl -sL https://aka.ms/DevTunnelCliInstall | bash

# Windows
winget install Microsoft.devtunnel
```

Then authenticate:
```bash
devtunnel user login -g   # -g for GitHub auth (used by PolyPilot)
```

## Hardware

- **PC:** Any machine that can run .NET 10. ~4 GB RAM free for Copilot + PolyPilot.
- **Phone:** Android 7+ with camera (for QR code scanning) and WiFi/mobile data.

## Firewall Notes

On Linux, ensure port 4322 is open for LAN:
```bash
# UFW
sudo ufw allow 4322/tcp

# firewalld
sudo firewall-cmd --add-port=4322/tcp --permanent && sudo firewall-cmd --reload

# iptables
sudo iptables -A INPUT -p tcp --dport 4322 -j ACCEPT
```

On macOS, the first time PolyPilot starts the bridge server, macOS will prompt "Allow incoming connections?" — click **Allow**.

On Windows, you may need to add a firewall exception:
```powershell
New-NetFirewallRule -DisplayName "PolyPilot Bridge" -Direction Inbound -LocalPort 4322 -Protocol TCP -Action Allow
```

---

**Next:** [Building & Running the Desktop Server](build-desktop.md) →
