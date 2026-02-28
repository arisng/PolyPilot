# Dual-Mode Setup — LAN + Internet With Auto-Switching

> **Recommended setup.** The phone auto-detects whether it's on home WiFi (uses LAN for speed) or mobile data (uses tunnel for access). Zero manual switching.

## How It Works

```
                        ┌─ WiFi detected? ──► ws://192.168.1.X:4322 (LAN, fast)
┌───────────────┐       │
│ Android Phone │───────┤  Smart URL
│  Remote Mode  │       │  Resolution
└───────────────┘       │
                        └─ Mobile data? ────► wss://xyz.devtunnels.ms (tunnel, anywhere)
```

PolyPilot's mobile app monitors network connectivity changes. When both a LAN URL and a tunnel URL are configured:

- **On WiFi:** connects via LAN (lower latency, no tunnel overhead)
- **On mobile data:** connects via tunnel (internet access)
- **Network switch:** automatically reconnects to the appropriate URL

## Setup (One-Time)

### On Your Desktop

1. **Enable Direct Sharing** with a password (see [LAN Setup](lan-setup.md)).
2. **Start a DevTunnel** (see [Internet Tunnel](internet-tunnel.md)).
3. Both should be running simultaneously.
4. The QR code now contains **both** URLs:

```json
{
  "url": "https://xyz.devtunnels.ms",
  "token": "tunnel-access-token",
  "lanUrl": "http://192.168.1.100:4322",
  "lanToken": "your-server-password"
}
```

### On Your Phone

1. Scan the QR code — **both** URLs are imported automatically.
2. Tap **Save & Reconnect**.
3. Done. The app handles switching.

### Manual Configuration

If entering manually on the phone:

| Field | Value | Purpose |
|-------|-------|---------|
| **Remote URL** | `https://xyz.devtunnels.ms` | Internet tunnel URL |
| **Remote Token** | Tunnel access token | Auth for tunnel |
| **LAN URL** | `http://192.168.1.100:4322` | Local WiFi URL |
| **LAN Token** | Your server password | Auth for LAN |

Both URLs (Remote URL and LAN URL) must be filled for auto-switching. Tokens are needed for authentication but the switching logic works without them.

## How Auto-Switching Works

The smart URL resolution logic (`WsBridgeClient.cs` for URL probing, `CopilotService.Bridge.cs` for network-change detection):

1. On **WiFi connected**: probes LAN URL (2-second HTTP timeout) → falls back to tunnel if LAN unreachable
2. On **WiFi lost** (mobile data): skips LAN probe entirely → connects via tunnel
3. On **WiFi regained**: disconnects tunnel → reconnects via LAN
4. Network changes are debounced (1-second timer) to avoid rapid reconnection flapping

The app registers a `ConnectivityChanged` event handler on Android/iOS that triggers URL re-resolution.

## Verifying Dual-Mode

### Test LAN Mode

1. Connect your phone to home WiFi.
2. Open PolyPilot — should connect via LAN.
3. Check the connection indicator — it should show the LAN URL.

### Test Tunnel Mode

1. Disconnect your phone from WiFi (use mobile data).
2. PolyPilot should auto-reconnect via the tunnel within a few seconds.
3. Verify you can still see sessions and send prompts.

### Test Switching

1. Start on WiFi → verify LAN connection.
2. Turn off WiFi → verify tunnel reconnection.
3. Turn WiFi back on → verify LAN reconnection.

## When to Use This

| Scenario | Setup |
|----------|-------|
| Only use at home | [LAN Setup](lan-setup.md) is simpler |
| Only use outside | [Internet Tunnel](internet-tunnel.md) is simpler |
| **Use both home & outside** | **This guide** (recommended) |
| Have Tailscale on all devices | [Tailscale Setup](tailscale-setup.md) — skip dual-mode |

---

**Next:** [Tailscale Setup](tailscale-setup.md) →
