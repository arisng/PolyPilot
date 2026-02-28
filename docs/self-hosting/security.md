# Security Hardening

> Protect your self-hosted PolyPilot from unauthorized access.

## Threat Model

When you expose PolyPilot remotely, you're giving network access to:
- All your Copilot sessions (read + write)
- The ability to send prompts to AI agents running on your PC
- Tool execution on your machine (file reads, terminal commands, etc.)

**This is powerful. Protect it.**

## Authentication Layers

### Layer 1: Server Password (Always Set This)

The WsBridge server supports password-based auth:

1. **Desktop** → Settings → Direct Sharing → **Server Password**
2. Set a strong, unique password
3. This password is required for any non-localhost connection

Authentication flow:
```
Phone connects → Header: X-Tunnel-Authorization: tunnel YOUR_PASSWORD
                 (or fallback: ws://host:4322?token=YOUR_PASSWORD)
```

> The header method is how the PolyPilot client actually authenticates. The query string is accepted as a fallback but exposes the password in server logs.

If neither a server password nor an access token is configured, **all connections are allowed** (no authentication). Always set a password before enabling Direct Sharing.

### Layer 2: DevTunnel Access Token

When using DevTunnel, an additional access token is generated:
- The tunnel CLI produces this token automatically
- It's included in the QR code
- DevTunnel infrastructure validates it before proxying

> **Defense in depth:** The tunnel URL is protected by the DevTunnel access token (validated by Microsoft's tunnel infrastructure). Direct LAN connections are protected by your server password. These are separate layers — tunnel connections are authenticated by DevTunnel before reaching your bridge, while LAN connections authenticate directly with your server password.

### Layer 3: Network-Level Isolation

| Method | Protection Level |
|--------|-----------------|
| **Tailscale** | ★★★★★ — WireGuard encrypted, only your devices |
| **LAN-only** | ★★★★ — limited to your WiFi network |
| **DevTunnel** | ★★★ — TLS encrypted, token-authenticated |
| **ngrok** | ★★★ — TLS encrypted, but URL is guessable |
| **Port forwarding** | ★ — **Avoid.** Exposes directly to internet |

## Best Practices

### DO ✅

- **Always set a server password** before enabling Direct Sharing
- **Use Tailscale** if possible — best security-to-convenience ratio
- **Rotate passwords** periodically in `~/.polypilot/settings.json`
- **Review DevTunnel access tokens** — PolyPilot issues connect-scoped tokens via `devtunnel token --scopes connect`
- **Keep your Copilot CLI up to date** (`copilot update`)
- **Monitor active connections** via PolyPilot's bridge status indicator

### DON'T ❌

- **Don't expose port 4322 directly to the internet** (use tunnels/VPN instead)
- **Don't use simple passwords** — anyone who can guess it gets full agent control
- **Don't share QR codes publicly** — they contain your URL + password
- **Don't leave Auto-Start Tunnel on** if you don't need constant access
- **Don't disable the firewall** — open only port 4322 specifically

## Loopback Trust

The bridge server trusts loopback connections (`127.0.0.1`, `::1`) without authentication. This is by design:

- DevTunnel proxies to localhost → appears as loopback → auto-trusted
- DevTunnel's own auth happens at the tunnel infrastructure level
- Local development/testing doesn't require a password
- The bridge also exposes a `/token` endpoint on localhost that returns the current access token — useful for local tooling integration

This means **if someone can access localhost on your machine, they can connect without a password.** This is the same trust model as any local service.

## Settings File Security

Connection settings with passwords are stored in plaintext at:
```
~/.polypilot/settings.json
```

This file contains:
- `ServerPassword` — the bridge server password
- `RemoteToken` — tunnel access token
- `LanToken` — LAN connection token

**Protect this file:**
```bash
chmod 600 ~/.polypilot/settings.json
```

## Audit Checklist

Run through this before going "always-on":

- [ ] Server password is set and strong (12+ chars, random)
- [ ] Firewall only exposes port 4322 to local network (not internet)
- [ ] DevTunnel (if used) requires authentication
- [ ] `~/.polypilot/settings.json` has `600` permissions
- [ ] Copilot CLI is authenticated and up to date
- [ ] Tailscale (if used) has ACLs configured for your devices
- [ ] No other services are listening on port 4322
- [ ] QR code screenshots are not saved/shared publicly

---

**Next:** [Troubleshooting](troubleshooting.md) →
