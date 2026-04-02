# Twitch Channel Setup

> Connect your OpenClaw agent to a Twitch channel's chat.

**Time:** 20 minutes  
**Prerequisites:** Module 01 (Quickstart), a Twitch account

---

## Overview

The Twitch channel lets your AI agent participate in Twitch chat. It can respond to messages, answer viewer questions, moderate conversations, or provide stream-related information.

---

## Step 1: Get Your Twitch Credentials

### 1a. Create a Twitch Bot Account (Recommended)

Create a separate Twitch account for your bot, or use your existing account.

### 1b. Generate an OAuth Token

1. Go to the [Twitch Token Generator](https://twitchtokengenerator.com/) or use the Twitch CLI
2. Authorize with your bot account
3. Copy the **OAuth Access Token** (starts with `oauth:`)
4. Copy the **Client ID**

### 1c. Get Your Twitch User ID

To restrict who the bot responds to, you need your numeric Twitch user ID:

1. Go to [StreamWeasels User ID Converter](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
2. Enter your Twitch username
3. Copy the numeric ID

---

## Step 2: Configure the Channel

Add to your `openclaw.json`:

```json5
{
  channels: {
    twitch: {
      enabled: true,
      username: "my_bot_name",              // Bot's Twitch username
      accessToken: "oauth:abc123...",        // OAuth token
      clientId: "xyz789...",                 // Client ID from token generator
      channel: "your_channel_name",          // Which Twitch channel to join
      allowFrom: ["123456789"],              // Your Twitch user ID (recommended)
    },
  },
}
```

> **Security:** Use environment variables for secrets:
> ```json5
> {
>   channels: {
>     twitch: {
>       enabled: true,
>       username: "my_bot_name",
>       accessToken: { "$env": "TWITCH_OAUTH_TOKEN" },
>       clientId: { "$env": "TWITCH_CLIENT_ID" },
>       channel: "your_channel_name",
>       allowFrom: ["123456789"],
>     },
>   },
> }
> ```

---

## Step 3: Restart the Gateway

```bash
openclaw gateway restart
openclaw channels status --probe
```

---

## Access Control

### Restrict to Specific Users

The `allowFrom` array accepts Twitch **numeric user IDs** (not usernames):

```json5
{
  channels: {
    twitch: {
      allowFrom: ["123456789", "987654321"],  // Only these users can interact
    },
  },
}
```

### Open to All Chat Participants

Omit `allowFrom` or set it to `["*"]` — but be careful in busy streams.

---

## Use Cases

| Use Case | Description |
|----------|-------------|
| **Chat assistant** | Answer viewer questions about the stream |
| **Moderator** | Help with chat moderation tasks |
| **Knowledge base** | Provide info about your content/games |
| **Interactive AI** | Let viewers interact with your AI during streams |

---

## Verification Checklist

- [ ] Bot appears in your Twitch channel's chat
- [ ] Bot responds to messages from allowed users
- [ ] Bot ignores messages from non-allowed users (if `allowFrom` is set)
- [ ] Gateway logs show Twitch channel connected

---

## What's Next?

| Next Step | Module |
|-----------|--------|
| Add more channels | [Module 02: Channels](../) |
| Route Twitch to a specific agent | [Module 04: Multi-Agent](../../04-multi-agent/) |
| Schedule stream-related tasks | [Module 11: Scheduler](../../11-scheduler/) |
