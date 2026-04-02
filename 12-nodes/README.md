# Module 12: Nodes (iOS, Android & macOS)

> Connect your phone, tablet, or Mac as a peripheral to your AI agent.

**Time:** 30 minutes  
**Prerequisites:** Module 01 (Quickstart)  
**You'll learn:** Device pairing, Canvas display, camera/screen capture, device commands, location, and remote node hosting

---

## What Are Nodes?

A **node** is a companion device that connects to the OpenClaw Gateway via WebSocket. Nodes are **peripherals**, not gateways — they expose device capabilities (camera, screen, canvas, sensors) to your AI agent.

```
┌─────────────┐     WebSocket      ┌──────────────┐
│  iOS / Android  │ ──────────────► │   Gateway    │
│  macOS Menubar  │   role: "node"  │  :18789      │
└─────────────┘                    └──────────────┘
```

### What Nodes Can Do

| Capability | iOS | Android | macOS |
|-----------|-----|---------|-------|
| Canvas (WebView display) | Yes | Yes | Yes |
| Camera (photo/video) | Yes | Yes | Yes |
| Screen recording | Yes | Yes | Yes |
| Location | Yes | Yes | - |
| SMS | - | Yes | - |
| Contacts | - | Yes | - |
| Calendar | - | Yes | - |
| Notifications | - | Yes | - |
| Call logs | - | Yes | - |
| Shell commands | - | - | Yes |
| Voice | Yes | Yes | Yes |

---

## 1. Device Pairing

Before a node can interact with the Gateway, it must be paired.

### Step 1: Start the Node

**iOS/Android:** Install the OpenClaw app and enter your Gateway address.

**macOS Menubar:** The macOS app auto-connects as a node when launched.

**Headless Node (remote machine):**

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Step 2: Approve the Pairing

The Gateway creates a pairing request that must be approved:

```bash
# List pending pairing requests
openclaw devices list

# Approve a specific request
openclaw devices approve <requestId>

# Check node status
openclaw nodes status
```

A node shows as **paired** once its device pairing role includes the node designation.

---

## 2. Canvas & Display

The Canvas is a WebView on the node that your agent can control — display web pages, render HTML, execute JavaScript, and capture snapshots.

### Display a URL

```bash
openclaw nodes canvas present --node <id> --url "https://example.com"
```

### Navigate

```bash
openclaw nodes canvas navigate --node <id> --url "https://dashboard.myapp.com"
```

### Execute JavaScript

```bash
openclaw nodes canvas eval --node <id> --script "document.title"
```

### Take a Snapshot

```bash
openclaw nodes canvas snapshot --node <id> --format png
```

### A2UI Overlays

Control A2UI v0.8 JSONL overlays on the Canvas:

```bash
openclaw nodes canvas a2ui push --node <id> --data '{"type":"notification","text":"Hello"}'
openclaw nodes canvas a2ui reset --node <id>
```

---

## 3. Camera & Multimedia

### Take a Photo

```bash
# Front camera (selfie)
openclaw nodes camera snap --node <id> --facing front

# Back camera
openclaw nodes camera snap --node <id> --facing back
```

### Record a Video Clip

```bash
openclaw nodes camera clip --node <id> --duration 10s
```

### Screen Recording

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 10
```

> **Important:** Camera and screen recording require the node app to be in the **foreground**. Background calls return `NODE_BACKGROUND_UNAVAILABLE`.

---

## 4. Android Device Commands

Android nodes expose rich device command families:

### Device Info & Health

```bash
openclaw nodes invoke --node <id> --command device.status
openclaw nodes invoke --node <id> --command device.info
openclaw nodes invoke --node <id> --command device.health
openclaw nodes invoke --node <id> --command device.permissions
```

### SMS

```bash
# Send a text message
openclaw nodes invoke --node <id> --command sms.send \
  --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'

# Search messages
openclaw nodes invoke --node <id> --command sms.search \
  --params '{"query":"meeting"}'
```

### Contacts

```bash
openclaw nodes invoke --node <id> --command contacts.search \
  --params '{"query":"John"}'

openclaw nodes invoke --node <id> --command contacts.add \
  --params '{"name":"Jane Doe","phone":"+15555550456"}'
```

### Calendar

```bash
openclaw nodes invoke --node <id> --command calendar.events \
  --params '{"days":7}'

openclaw nodes invoke --node <id> --command calendar.add \
  --params '{"title":"Team standup","start":"2026-04-02T10:00:00","duration":"30m"}'
```

### Notifications

```bash
openclaw nodes invoke --node <id> --command notifications.list
openclaw nodes invoke --node <id> --command notifications.actions
```

### Photos & Call Logs

```bash
openclaw nodes invoke --node <id> --command photos.latest
openclaw nodes invoke --node <id> --command callLog.search
```

### Motion & Activity (sensor-dependent)

```bash
openclaw nodes invoke --node <id> --command motion.activity
openclaw nodes invoke --node <id> --command motion.pedometer
```

---

## 5. Location Services

Location is **off by default** and requires explicit system permissions on the device.

```bash
openclaw nodes location get --node <id> --accuracy precise
```

Your agent can use location data for:
- Weather-aware morning briefings
- Location-based reminders ("remind me when I get to the office")
- Travel time estimates

---

## 6. Remote Shell Execution via Nodes

macOS nodes and headless node hosts can execute shell commands on behalf of the agent.

### Configure Default Exec Target

```bash
# Route exec commands to a specific node
openclaw config set tools.exec.host node
openclaw config set tools.exec.node "<id-or-name>"
```

### Exec Allowlists

Control which commands a node can execute:

```bash
openclaw approvals allowlist add --node <id> "/usr/bin/uname"
openclaw approvals allowlist add --node <id> "/usr/local/bin/docker"
```

Approvals are stored in `~/.openclaw/exec-approvals.json` on the node host.

> **Security:** Environment variables in exec requests are restricted to: `TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`.

---

## 7. Remote Node Hosting

Run a node on a separate machine from the Gateway — useful for build servers, GPU boxes, or always-on desktops.

### Foreground Mode

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Server"
```

### Service Mode (persistent)

```bash
openclaw node install --host <gateway-host> --port 18789
```

### Through SSH Tunnel

If your Gateway is bound to loopback:

```bash
# Open tunnel
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Connect node through tunnel
export OPENCLAW_GATEWAY_TOKEN="<token>"
openclaw node run --host 127.0.0.1 --port 18790
```

---

## 8. macOS Menubar App

The macOS app connects to the Gateway as a node automatically. It provides:

- Quick access to chat
- Node status in the menu bar
- Automatic SSH tunnel for remote Gateway connectivity
- All `openclaw nodes ...` commands work against the Mac

---

## Architecture Overview

```
┌──────────────────────────────────────────────────┐
│                   Gateway :18789                 │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │ Agent    │  │ Channels │  │ Node Manager │   │
│  │ Runtime  │  │ (TG/WA)  │  │              │   │
│  └────┬─────┘  └──────────┘  └──────┬───────┘   │
│       │                             │            │
│       │      tool: node.invoke      │            │
│       └─────────────────────────────┘            │
└──────────────────────────────────────────────────┘
        ▲              ▲              ▲
        │ WS           │ WS           │ WS
   ┌────┴────┐   ┌─────┴─────┐  ┌────┴─────┐
   │  iPhone │   │  Android  │  │   Mac    │
   │  Node   │   │  Node     │  │  Menubar │
   └─────────┘   └───────────┘  └──────────┘
```

---

## Verification Checklist

- [ ] Pair a device and verify it shows in `openclaw nodes status`
- [ ] Take a Canvas snapshot from the paired device
- [ ] Capture a photo using `camera snap`
- [ ] (Android) Send a test SMS via `sms.send`
- [ ] (Android) Query upcoming calendar events
- [ ] Get device location with `location get`

---

## What's Next?

| Next Step | Module |
|-----------|--------|
| Manage nodes from browser | [Module 13: Control UI](../13-control-ui/) |
| Schedule node actions | [Module 11: Scheduler](../11-scheduler/) |
| Secure node exec | [Module 07: Security](../07-security/) |
