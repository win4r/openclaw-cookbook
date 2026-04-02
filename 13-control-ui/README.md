# Module 13: Control UI (Web Dashboard)

> Manage your entire OpenClaw instance from the browser.

**Time:** 15 minutes  
**Prerequisites:** Module 01 (Quickstart)  
**You'll learn:** Dashboard setup, chat interface, channel management, cron UI, config editing, log monitoring, and remote access

---

## What Is the Control UI?

The Control UI is a browser-based dashboard served by the Gateway. It gives you a visual interface for everything you'd normally do via CLI — chat with your agent, manage channels, edit config, monitor logs, and more.

Built with **Vite + Lit**, it communicates with the Gateway via WebSocket on the same port.

```
Browser ──── http://127.0.0.1:18789/ ────► Gateway
         (WebSocket on same port)
```

---

## 1. Enable the Control UI

The Control UI is enabled by default. To configure it:

```json5
{
  gateway: {
    controlUi: {
      enabled: true,
      basePath: "/openclaw",  // optional: serve at /openclaw instead of /
    },
  },
}
```

Then open:

```bash
openclaw dashboard
# or navigate to http://127.0.0.1:18789/
```

---

## 2. Features Overview

### Chat & Agent Interaction

- **Real-time streaming** — watch tool calls and outputs as they happen
- **Live output cards** — structured display of agent actions
- **Session overrides** — toggle thinking level, fast mode, verbose mode per-session
- **Stop/abort** — cancel running operations with partial output retention
- **Agent event monitoring** — see what your agent is doing in real time

### Channel Management

- **All channels in one place** — WhatsApp, Telegram, Discord, Slack, and plugin channels
- **QR login** — scan QR codes for WhatsApp directly in the UI
- **Per-channel configuration** — toggle settings without editing JSON

### Cron & Scheduling

- **Create cron jobs** visually
- **Edit schedules** and prompts
- **Manual execution** — trigger any job with one click
- **Delivery configuration** — set output channels per job

### Administration

- **Skill management** — view status, update API keys
- **Node presence** — see connected devices and their capabilities
- **Execution approvals** — manage allowlists for Gateway and node exec
- **System health** — model snapshots, resource usage

### Config & Debug

- **Live config editor** — JSON editing with validation and concurrent-edit protection
- **SecretRef resolution** — see which secrets are loaded and valid
- **Log tailing** — real-time Gateway logs with filtering
- **Package updates** — update OpenClaw and restart from the UI

---

## 3. Access & Security

### Local Access

Local connections auto-approve — just open `http://127.0.0.1:18789/`.

### Remote Access (Pairing)

Remote connections require **one-time pairing approval**:

1. Open the Control UI from a remote browser
2. The Gateway prompts for approval
3. Approve via CLI or from a local session

### Authentication Methods

| Method | How |
|--------|-----|
| **Token** | Pass `?token=<gateway-token>` or set in headers |
| **Password** | Configure in Gateway settings |
| **Tailscale** | Identity via Tailscale headers (recommended for remote) |

### Tailscale Serve (Recommended for Remote Access)

```bash
# Install and authenticate Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

This gives you HTTPS access with Tailscale identity — no exposed ports, no passwords.

### Tailnet Binding (Alternative)

```json5
{
  gateway: {
    bind: "tailscale",  // bind to Tailscale IP only
    token: "<gateway-token>",
  },
}
```

---

## 4. Deployment Options

| Mode | URL | Best For |
|------|-----|----------|
| **Local** | `http://127.0.0.1:18789/` | Development, personal use |
| **Tailscale Serve** | `https://<machine>.tail12345.ts.net/` | Remote access (recommended) |
| **Tailnet binding** | `http://<tailscale-ip>:18789/` | Team access within tailnet |
| **Reverse proxy** | Your domain with TLS | Production deployment |

---

## 5. Localization

The Control UI supports multiple languages:

- English
- Simplified Chinese (简体中文)
- Traditional Chinese (繁體中文)
- Portuguese (Brazil)
- German
- Spanish

Language is auto-detected from your browser settings.

---

## 6. Development & Customization

### Build Production UI

```bash
pnpm ui:build
```

### Local Development

Run the dev server pointing to a remote Gateway:

```bash
pnpm ui:dev
# Then open with ?gatewayUrl=ws://<gateway-host>:18789
```

---

## Verification Checklist

- [ ] Open the Control UI at `http://127.0.0.1:18789/`
- [ ] Send a message and verify streaming response
- [ ] View channel status in the Channels panel
- [ ] Create a test cron job from the UI
- [ ] Open the live log viewer and filter by keyword
- [ ] Edit a config value and verify validation

---

## What's Next?

| Next Step | Module |
|-----------|--------|
| Schedule automated tasks | [Module 11: Scheduler](../11-scheduler/) |
| Connect mobile devices | [Module 12: Nodes](../12-nodes/) |
| Secure remote access | [Module 07: Security](../07-security/) |
| Production deployment | [Module 09: Production](../09-production/) |
