# Tailscale Setup — Zero-Config VPN

> If both your PC and phone have Tailscale, you can skip tunnels entirely. Tailscale creates a secure overlay network that works from anywhere.

## How It Works

```
┌───────────────┐      Tailscale mesh (WireGuard)      ┌───────────────┐
│  Desktop PC   │ ◄──────────────────────────────────► │ Android Phone │
│  100.x.y.z    │     ws://mypc.tail12345.ts.net:4322  │  100.x.y.w    │
│  WsBridge:4322│                                       │  Remote Mode  │
└───────────────┘                                       └───────────────┘
```

Tailscale provides:
- Direct WireGuard connections (fast, encrypted)
- Stable `100.x.y.z` IPs that work from anywhere
- Magic DNS names (e.g., `mypc.tail12345.ts.net`)
- No port forwarding or tunnels needed

## Setup

### 1. Install Tailscale

| Platform | Install |
|----------|---------|
| macOS | `brew install tailscale` or [Tailscale app](https://tailscale.com/download) |
| Linux | `curl -fsSL https://tailscale.com/install.sh \| sh` |
| Windows | [Tailscale installer](https://tailscale.com/download/windows) |
| Android | [Google Play Store](https://play.google.com/store/apps/details?id=com.tailscale.ipn) |

### 2. Join the Same Tailnet

Sign in on both devices with the same Tailscale account. Verify:

```bash
# On your PC
tailscale status
```

Both devices should appear in the output.

### 3. Enable Direct Sharing on Desktop

1. Open PolyPilot → **Settings**.
2. Set a **Server Password**.
3. Click **Enable Direct Sharing**.

PolyPilot auto-detects Tailscale via `TailscaleService` and:
- Discovers your Tailscale IP (`100.x.y.z`)
- Discovers your Magic DNS name (e.g., `mypc.tail12345.ts.net`)
- Uses the Tailscale address in the QR code / LAN URL

### 4. Connect From Phone

The QR code (or LAN URL) will show your Tailscale address:
```
http://mypc.tail12345.ts.net:4322
```

On your phone:
1. Make sure Tailscale is connected (VPN toggled on).
2. Scan the QR code or enter the URL manually.
3. Enter the server password as the token.
4. **Save & Reconnect**.

## Why Tailscale?

| Feature | Tailscale | DevTunnel | LAN Only |
|---------|-----------|-----------|----------|
| Works from home | ✅ | ✅ | ✅ |
| Works from outside | ✅ | ✅ | ❌ |
| No relay server | Mostly direct ✅ (DERP relay fallback) | ❌ (relayed) | ✅ |
| Latency | **Lowest** | Higher | Low |
| Encryption | WireGuard | TLS | None (HTTP) |
| Stable URL | ✅ (MagicDNS) | With tunnel ID | ❌ (DHCP) |
| No extra process | ✅ | Needs devtunnel | ✅ |

## PolyPilot Tailscale Auto-Detection

PolyPilot's `TailscaleService` automatically detects Tailscale on startup:

1. **Unix socket API** (`/var/run/tailscale/tailscaled.sock`) — tried first on macOS/Linux
2. **CLI fallback** (`tailscale status --json`) — used if socket unavailable

Detected values:
- `TailscaleIp`: your `100.x.y.z` address
- `MagicDnsName`: your `hostname.tailnet.ts.net` name

These are used in QR code generation and LAN URL display.

## Combining With Dual-Mode

You can use Tailscale as the LAN URL and a DevTunnel as fallback for devices not on Tailscale. On your phone, set:

| Field | Value |
|-------|-------|
| LAN URL | `http://mypc.tail12345.ts.net:4322` |
| LAN Token | Server password |
| Remote URL | `https://xyz.devtunnels.ms` |
| Remote Token | Tunnel access token |

The phone prefers the Tailscale URL on WiFi and falls back to the tunnel when Tailscale is unreachable.

---

**Next:** [Squad Management](squad-management.md) →
