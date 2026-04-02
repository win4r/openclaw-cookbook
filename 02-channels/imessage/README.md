# iMessage Channel Setup

> Connect your OpenClaw agent to iMessage on macOS.

**Time:** 15 minutes  
**Prerequisites:** Module 01 (Quickstart), macOS with Messages app configured

---

## Overview

The iMessage channel connects your AI agent to Apple's Messages app. Your agent can send and receive iMessages, making it accessible from any Apple device signed into the same Apple ID.

> **Note:** iMessage integration requires the Gateway to run on macOS (or have access to a macOS node with Messages configured).

---

## Step 1: Configure the Channel

Add to your `openclaw.json`:

```json5
{
  channels: {
    imessage: {
      enabled: true,
      allowFrom: ["+15555550123"],  // Your phone number(s)
    },
  },
}
```

---

## Step 2: Grant Permissions

The Gateway needs access to the Messages database on macOS. Grant **Full Disk Access** to the terminal or process running OpenClaw:

1. Open **System Settings** > **Privacy & Security** > **Full Disk Access**
2. Add your terminal app (Terminal, iTerm2) or the OpenClaw process

---

## Step 3: Restart and Test

```bash
openclaw gateway restart
openclaw channels status --probe
```

Send an iMessage to test the connection.

---

## Access Control

### Restrict by Phone Number

```json5
{
  channels: {
    imessage: {
      allowFrom: ["+15555550123", "+15555550456"],
    },
  },
}
```

### Group Messages

```json5
{
  channels: {
    imessage: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
}
```

---

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Personal assistant** | Quick AI access from any Apple device |
| **Family bot** | Shared AI assistant in a family group chat |
| **Cross-device** | Same AI on iPhone, iPad, and Mac |

---

## Limitations

- Requires macOS (Messages.app dependency)
- No support for RCS or SMS — iMessage only
- Group chat support depends on macOS version

---

## Verification Checklist

- [ ] Gateway shows iMessage channel connected
- [ ] Bot responds to iMessages from allowed numbers
- [ ] Messages appear on all synced Apple devices

---

## What's Next?

| Next Step | Module |
|-----------|--------|
| Add more channels | [Module 02: Channels](../) |
| Connect iOS as a node | [Module 12: Nodes](../../12-nodes/) |
| Multi-agent routing | [Module 04: Multi-Agent](../../04-multi-agent/) |
