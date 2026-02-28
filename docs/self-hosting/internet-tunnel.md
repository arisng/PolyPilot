# Internet Tunnel — Access From Anywhere

> Reach your desktop PolyPilot from mobile data, coffee shops, or anywhere with internet.

## How It Works

```
┌───────────────┐     wss://xyz.devtunnels.ms     ┌───────────────┐
│  Desktop PC   │ ◄──┬───────────────────────────► │ Android Phone │
│  WsBridge:4322│    │   DevTunnel relay cloud     │  Remote Mode  │
└───────────────┘    │                              └───────────────┘
                     │  DevTunnel CLI (port forward)
                     └─ localhost:4322 → public URL
```

A tunnel service creates a public URL that routes traffic to your desktop's port 4322. Your phone connects to the public URL — the tunnel relays everything to your PC.

## Option A: Microsoft DevTunnel (Built-In)

PolyPilot has **native DevTunnel integration** — you can start/stop tunnels from the Settings page.

### Setup

1. Install the DevTunnel CLI (see [Prerequisites](prerequisites.md)).
2. Authenticate: `devtunnel user login -g`
3. Open PolyPilot on your desktop → **Settings**.
4. Scroll to the **DevTunnel** section.
5. Click **Start Tunnel**.

PolyPilot will:
- Run `devtunnel host -p 4322` (new tunnel) or `devtunnel host <tunnel-id>` (reuse existing)
- Issue a scoped access token via `devtunnel token <id> --scopes connect`
- Parse the tunnel URL from stdout
- Display a **QR code** with the tunnel URL + access token
- Set the bridge server's access token for auth

### Connect From Phone

1. On your phone, open PolyPilot → Settings.
2. Scan the **QR code** on the desktop, or manually enter:
   - **Remote URL**: the `.devtunnels.ms` URL
   - **Remote Token**: the tunnel access token
3. Tap **Save & Reconnect**.

### Auto-Start Tunnel

When you start a tunnel, PolyPilot automatically saves the tunnel ID and enables auto-start for next launch. Stopping the tunnel disables auto-start. The setting persists in `~/.polypilot/settings.json`:
```json
{
  "AutoStartTunnel": true,
  "TunnelId": "your-tunnel-id",
  "DirectSharingEnabled": true
}
```

> **Note:** Direct Sharing must also be enabled — the tunnel forwards to the bridge server, which must be running.

## Option B: ngrok

If you prefer ngrok:

```bash
# Install
brew install ngrok  # or download from ngrok.com

# Authenticate (one-time)
ngrok config add-authtoken YOUR_TOKEN

# Start tunnel
ngrok http 4322
```

Copy the `https://xxxx.ngrok-free.app` URL and enter it manually on your phone.

> **Note:** ngrok URLs change on restart unless you have a paid plan with reserved domains. DevTunnel is free and integrated — prefer it.

## Option C: Cloudflare Tunnel

```bash
# Install cloudflared
brew install cloudflare/cloudflare/cloudflared

# Quick tunnel (no account needed)
cloudflared tunnel --url http://localhost:4322
```

Copy the `https://xxxx.trycloudflare.com` URL.

## Authentication Over Tunnels

| Method | Auth Flow |
|--------|-----------|
| **DevTunnel** | PolyPilot issues a scoped access token via `devtunnel token --scopes connect` and includes it in the QR code. The bridge server validates this token. Loopback connections (from DevTunnel's localhost proxy) are also auto-trusted as a separate mechanism. |
| **ngrok/Cloudflare** | Use the **Server Password** for auth. Set it in Settings → Direct Sharing → Server Password. Enter the same password as the Remote Token on your phone. |

## Tunnel Comparison

| Feature | DevTunnel | ngrok (free) | Cloudflare |
|---------|-----------|-------------|------------|
| Integrated in UI | **Yes** | No | No |
| QR code generation | **Yes** | No | No |
| Persistent URL | With tunnel ID | No (paid only) | No |
| Auto-start | **Yes** | Manual | Manual |
| Auth | Built-in token | Token or password | Password |
| WebSocket support | **Yes** | **Yes** | **Yes** |
| Cost | Free | Free tier limited | Free |

## Keeping the Tunnel Alive

### DevTunnel

The built-in integration manages the tunnel process lifecycle. If the tunnel dies, PolyPilot shows an error in the Settings page. Click **Start Tunnel** again.

For unattended operation, you can run the tunnel independently:

```bash
# Create a persistent tunnel (omit --allow-anonymous to require token auth)
devtunnel create
devtunnel port create -p 4322

# Host it (stays alive until killed)
devtunnel host
```

### systemd (Linux) for Auto-Restart

```ini
# ~/.config/systemd/user/polypilot-tunnel.service
[Unit]
Description=PolyPilot DevTunnel
After=network-online.target

[Service]
ExecStart=/usr/local/bin/devtunnel host -p 4322
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

```bash
systemctl --user enable --now polypilot-tunnel.service
```

---

**Next:** [Dual-Mode Setup](dual-mode-setup.md) →
