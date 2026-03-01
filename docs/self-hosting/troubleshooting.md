# Troubleshooting

> Common issues and their fixes.

## Connection Issues

### Phone Can't Connect (LAN)

**Symptoms:** "Connection failed" or timeout on phone.

| Check | Fix |
|-------|-----|
| Same WiFi network? | Both phone and PC must be on the same network. Check your phone's WiFi settings. |
| Correct IP? | Run `hostname -I` (Linux) or `ipconfig getifaddr en0` (macOS) on your PC. IPs change on DHCP. |
| Bridge running? | Desktop → Settings → Direct Sharing should show a green indicator. |
| Firewall blocking? | Open port 4322 — see [Prerequisites](prerequisites.md#firewall-notes). |
| Password correct? | The "LAN Token" on phone must match "Server Password" on desktop exactly. |

**Quick diagnostic:**
```bash
# From another machine on the same network, test if port 4322 is reachable
curl -v http://<your-pc-ip>:4322/
# Should return 200 OK with body "WsBridge OK" — the port IS open and bridge is running
# If connection refused → firewall or bridge not running
```

### WebSocket Auth Fails (`401 expected 101`)

**Symptoms:**
- Phone shows: `The server returned status code 401 when status code 101 was expected`
- HTTP health check may still return `200 WsBridge OK`

**What this means:**
- Network path is usually fine.
- Authentication failed during the WebSocket upgrade.

**Most common cause:**
- Desktop and phone are not using the same active token state (stale settings on one side), or LAN credentials were entered into `Remote` fields.

**Fix sequence (works reliably):**
1. On desktop: Settings → Direct Sharing → set (or re-set) **Server Password**.
2. Click **Save & Reconnect** (forces bridge to reload active auth state).
3. On phone: enter **LAN URL** and **LAN Token** (not Remote fields) for LAN-only use.
4. Tap **Save & Reconnect** on phone.
5. If still failing, re-scan the desktop QR code to refresh URL/token pairs together.

**Why this is confusing:**
- `curl http://<pc-ip>:4322/` can return `200` while WebSocket auth still fails.
- `200` verifies reachability, not WebSocket authorization.

### Phone Can't Connect (Tunnel)

**Symptoms:** Timeout or auth error when connecting via DevTunnel/ngrok.

| Check | Fix |
|-------|-----|
| Tunnel running? | Desktop → Settings → DevTunnel should show "Running" with a URL. |
| URL correct? | Ensure the full `https://xxx.devtunnels.ms` URL is entered on phone. |
| Token correct? | Scan the QR code again to refresh all credentials. |
| DevTunnel auth expired? | Run `devtunnel user login` on desktop. |
| Bridge running on desktop? | Direct Sharing must be enabled for the tunnel to have something to forward to. |

### Phone Connects but Sees No Sessions

The bridge is connected but the session list is empty:

1. Verify desktop has running sessions (check the dashboard).
2. On phone, pull-to-refresh or tap the dashboard reload.
3. Check that both Direct Sharing AND Copilot are connected on desktop.
4. If you just enabled Direct Sharing, restart the app — `WsBridgeServer.SetCopilotService()` must be called.

### Phone Shows "Disconnected" After Working

**Auto-reconnect:** The bridge client retries automatically. Wait 5-10 seconds.

If persistent:
- **LAN:** Your PC's IP may have changed. Re-scan the QR code.
- **Tunnel:** The tunnel may have died. Check desktop Settings → restart tunnel.
- **Sleep/idle:** Your PC may have gone to sleep. Disable sleep in OS power settings.

## Desktop Issues

### "Failed to start persistent server"

PolyPilot falls back to Embedded mode and shows a yellow banner.

**Causes:**
- Copilot CLI not installed or not in PATH
- Copilot auth expired
- Port conflict on the persistent server port

**Fixes:**
```bash
# Verify Copilot CLI
which copilot && copilot auth status

# Kill any stale server
kill $(cat ~/.polypilot/server.pid 2>/dev/null) 2>/dev/null

# Check port availability
lsof -i :4321  # Copilot server port
lsof -i :4322  # Bridge server port
```

### Bridge Server Won't Start

`WsBridgeServer.Start()` tries `http://+:4322/` first (all interfaces), then falls back to `http://localhost:4322/`.

If both fail:
```bash
# Check what's using port 4322
lsof -i :4322
# Kill it or change the bridge port (currently hardcoded to 4322)

# On Linux, binding to :80-1024 requires root. 4322 should be fine.
# If `http://+:` fails, check for non-admin httplistener restrictions.
```

### Sessions Disappear After Restart

Sessions are persisted in:
- `~/.polypilot/active-sessions.json` — index of active sessions
- `~/.copilot/session-state/<guid>/events.jsonl` — session event logs

Check:
```bash
# Active session index
cat ~/.polypilot/active-sessions.json | python3 -m json.tool

# Session directories
ls ~/.copilot/session-state/
```

If `active-sessions.json` is empty but directories exist, the merge-based persistence may have been interrupted. Restart PolyPilot — it will attempt restoration.

## Build Issues

### `dotnet build -f net10.0-android` Fails

```bash
# Ensure workloads are installed
dotnet workload install maui-android

# Check SDK version (must be 10.0+)
dotnet --version

# Clean and retry
dotnet clean
dotnet build -f net10.0-android
```

### APK Install Fails on Device

```bash
# Check device connection
adb devices

# If devices list is empty:
# 1. Enable USB debugging on phone (Settings → Developer options)
# 2. Trust the computer when prompted

# Use -t:Install not adb install
dotnet build -f net10.0-android -t:Install
```

### Mac Catalyst Build Fails

```bash
dotnet workload install maui-maccatalyst

# If signing issues:
cd PolyPilot && dotnet build -f net10.0-maccatalyst
# The sandbox is disabled in Entitlements.plist — required for spawning CLI processes
```

## Network Diagnostics

### Test WebSocket Connectivity

```bash
# Install websocat for WebSocket testing
# macOS: brew install websocat
# Linux: cargo install websocat

# Test LAN connection (header auth — how the app authenticates)
websocat -H "X-Tunnel-Authorization: tunnel YOUR_PASSWORD" ws://192.168.1.100:4322

# Alternative: query param auth (also accepted by server)
websocat ws://192.168.1.100:4322?token=YOUR_PASSWORD

# Test tunnel connection
websocat -H "X-Tunnel-Authorization: tunnel YOUR_TOKEN" wss://your-tunnel.devtunnels.ms
```

### Check Bridge Server Logs

Watch the console output on desktop:
```
[WsBridge] Listening on port 4322 (state-sync mode)
[WsBridge] Client abc123 connected (1 total)
[WsBridge] Rejected unauthenticated WebSocket connection
```

### Connectivity Change Events (Android/iOS)

The smart URL resolver logs network transitions (mobile client only — desktop doesn't use smart URL switching):
```
[SmartURL] Network changed: WiFi, Cellular
[SmartURL] Lost WiFi while on LAN — aborting connection to re-resolve via tunnel
[SmartURL] Gained WiFi while on tunnel — aborting connection to re-resolve via LAN
```

## Performance

### High Latency on Phone

| Cause | Fix |
|-------|-----|
| Using tunnel when on same WiFi | Set up [Dual-Mode](dual-mode-setup.md) — auto-uses LAN |
| DevTunnel relay far from your region | Try [Tailscale](tailscale-setup.md) for direct connections |
| Large conversation history | The bridge limits history sync to last 200 messages per turn |
| Many concurrent sessions | Reduce active session count or upgrade PC |

### Phone App Drains Battery

The WebSocket connection stays alive in background. To reduce battery:
- Close PolyPilot when not needed
- Use push notifications (if enabled) instead of keeping the app open
- On Android, exclude PolyPilot from battery optimization for reliable background connection

## Log Files

| File | Location | Contents |
|------|----------|----------|
| Event diagnostics | `~/.polypilot/event-diagnostics.log` | SDK event flow, IsProcessing transitions |
| Crash log | `~/.polypilot/crash.log` | Unhandled exceptions |
| Console output | Terminal where app runs | Bridge server, tunnel, Copilot logs |

---

**Back to:** [README](README.md)
