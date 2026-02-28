# Building & Running the Desktop Server

> Your PC is the brain — it runs Copilot sessions and serves state to your phone.

## Clone & Build

```bash
git clone https://github.com/<your-fork>/PolyPilot.git
cd PolyPilot
```

### macOS (Mac Catalyst)

```bash
cd PolyPilot
dotnet build -f net10.0-maccatalyst
```

To build and launch:
```bash
./relaunch.sh
```

> `relaunch.sh` does a full build, copies to staging, kills any old instance (freeing port 4322), and launches the new one. **Always use this** after code changes.

### Windows

```bash
cd PolyPilot
dotnet build -f net10.0-windows10.0.19041.0
dotnet run -f net10.0-windows10.0.19041.0
```

### Linux

Linux cannot run the MAUI desktop app natively (no `net10.0-linux` TFM in MAUI). Your options:

| Option | How |
|--------|-----|
| **Build Android target on Linux** | `dotnet build -f net10.0-android` — then deploy to an Android emulator or device on the same machine |
| **Run in a Windows/macOS VM** | Use a VM or WSL2 with GUI passthrough |
| **Use the Android APK as both host and client** | See note below |

> **Linux workaround:** You can build the Android APK on Linux and run it in an Android emulator. The emulator's PolyPilot instance runs Copilot locally, and you connect from your phone to the emulator's bridge.

## First Launch

1. Launch PolyPilot.
2. Go to **Settings** (gear icon).
3. Under **Connection Mode**, select **Persistent** (default on desktop).
4. Click **Save & Reconnect**.
5. Verify the status bar shows "Connected" with a green indicator.

## Verify Copilot Is Working

1. Click **+ New Session** on the dashboard.
2. Type a test prompt: "Hello, what model are you?"
3. You should see a streaming response.

If it fails, check:
- `copilot auth status` — must be authenticated
- `~/.polypilot/crash.log` — for error details
- `~/.polypilot/server.pid` — the persistent server PID file

## Connection Mode Reference

| Mode | Description | When to use |
|------|-------------|-------------|
| **Persistent** | App spawns a detached `copilot --headless` server (PID tracked in `~/.polypilot/server.pid`). Survives app restarts. | **Default for desktop.** Use this. |
| **Embedded** | SDK spawns copilot via stdio; dies with the app. | Fallback if Persistent fails. |
| **Demo** | Local mock responses, no network. | Testing UI without Copilot. |
| **Remote** | Connects to another PolyPilot's bridge server. | Designed for mobile. Available on desktop but typically unnecessary. |

## Persistent Mode Details

The persistent server:
- Runs as a background process (`copilot --headless`)
- Survives app restarts and relaunches
- PID stored at `~/.polypilot/server.pid`
- Sessions stored at `~/.copilot/session-state/<guid>/events.jsonl`
- Active session index at `~/.polypilot/active-sessions.json`

If the persistent server dies unexpectedly:
```bash
# Check if it's running
ps aux | grep "copilot.*headless"

# The app will auto-restart it on next launch
# Or manually restart (must specify port to match app expectations):
copilot --headless --port 4321 &
```

---

**Next:** [Building the Android Client](build-android.md) →
