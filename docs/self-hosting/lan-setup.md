# LAN Setup — Direct Sharing Over WiFi

> Connect your phone to PolyPilot on your PC over your home WiFi. Simplest setup, no internet required.

## How It Works

```
┌───────────────┐    ws://192.168.1.X:4322    ┌───────────────┐
│  Desktop PC   │ ◄──────────────────────────► │ Android Phone │
│  WsBridge:4322│          Local WiFi          │  Remote Mode  │
└───────────────┘                              └───────────────┘
```

The desktop runs a WebSocket server on port **4322**. Your phone connects directly via your PC's local IP address. Both devices must be on the same WiFi network.

## Step 1: Enable Direct Sharing on Desktop

1. Open PolyPilot on your PC.
2. Go to **Settings** (gear icon).
3. Scroll to the **Direct Sharing** section.
4. Set a **Server Password** — this protects the bridge from unauthorized access.
5. Click **Enable Direct Sharing**.

The bridge server starts on port 4322. You'll see:
- A green status indicator
- Your **LAN URL** (e.g., `http://192.168.1.100:4322`)
- A **QR code** for quick phone setup

## Step 2: Connect From Your Phone

### Option A: QR Code (Recommended)

1. On your phone, open PolyPilot → Settings.
2. Tap the **QR scan** button (camera icon).
3. Scan the QR code shown on your desktop.
4. The URL and password auto-fill.
5. Tap **Save & Reconnect**.

### Option B: Manual Entry

1. On your phone, open PolyPilot → Settings.
2. Enter the **Remote URL**: `http://<your-pc-ip>:4322`
3. Enter the **Remote Token**: your server password.
4. Tap **Save & Reconnect**.

To find your PC's IP:
```bash
# macOS
ipconfig getifaddr en0

# Linux
hostname -I | awk '{print $1}'

# Windows
ipconfig | findstr "IPv4"
```

## Step 3: Verify Connection

On your phone you should see:
- The dashboard with all sessions from your desktop
- A "Connected" status indicator
- Live-streaming chat messages when agents are working

Try sending a prompt from your phone — it should execute on the desktop and stream back in real-time.

## Authentication

Direct Sharing uses **password-based auth**:
- The password is sent via the `X-Tunnel-Authorization: tunnel <password>` header (primary method)
- Or as a `token` query parameter on the WebSocket URL (fallback)
- Loopback connections (localhost) are auto-trusted without auth
- If no password is set, **all connections are allowed** — no authentication is enforced

> **Security:** Always set a password before enabling Direct Sharing. Without one, anyone on your WiFi **can** connect with zero authentication.

## Limitations

- **WiFi only** — phone must be on the same network as your PC
- **No internet access** — won't work from outside your home
- **IP changes** — if your PC's IP changes (DHCP), you'll need to re-enter it
- **Firewall** — port 4322 must be open (see [Prerequisites](prerequisites.md#firewall-notes))

For internet access from anywhere, add a tunnel — see [Internet Tunnel](internet-tunnel.md) or go straight to [Dual-Mode Setup](dual-mode-setup.md) for the best of both worlds.

---

**Next:** [Internet Tunnel](internet-tunnel.md) →
