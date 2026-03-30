# Discord Bot

Connect your OpenClaw agent to Discord. The bot can respond in server channels, DMs, and threads. Discord's rich embed support and role-based permissions make it well suited for community bots.

**Time:** 30 minutes

**Plugin maturity:** Beta

---

## Prerequisites

- OpenClaw installed and running (`openclaw start` works)
- A Discord account
- Admin access to a Discord server (or create a new one for testing)
- At least one model provider configured

---

## Step 1: Create a Discord Application

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
2. Click "New Application" and give it a name
3. Go to the "Bot" section in the left sidebar
4. Click "Add Bot" and confirm
5. Under "Token", click "Copy" to copy your bot token. Save it -- you cannot view it again without regenerating.
6. Under "Privileged Gateway Intents", enable:
   - **Message Content Intent** (required to read user messages)
   - **Server Members Intent** (optional, needed for role-based access)

### Bot Permissions

Under "OAuth2 > URL Generator":
1. Select the "bot" scope
2. Select these permissions:
   - Send Messages
   - Send Messages in Threads
   - Read Message History
   - Embed Links
   - Attach Files
   - Use Slash Commands
3. Copy the generated URL and open it in your browser to invite the bot to your server

---

## Step 2: Enable the Discord Plugin

```bash
openclaw plugin enable discord
```

---

## Step 3: Configure the Bot Token

```bash
openclaw config set discord.bots.main.token "MTIzNDU2Nzg5MDEy..."
```

---

## Step 4: Bind the Bot to an Agent

```bash
openclaw config set discord.bots.main.agent "default"
```

---

## Step 5: Set Access Policies

### Channel Allowlist

Restrict which Discord channels the bot responds in:

```bash
openclaw config set discord.policies.channelAllowlist '[1234567890123456789, 9876543210987654321]'
```

To find a channel ID: enable Developer Mode in Discord settings (App Settings > Advanced > Developer Mode), then right-click a channel and select "Copy Channel ID".

If not set, the bot responds in all channels it has access to.

### Role Requirement

Require users to have a specific Discord role to interact with the bot:

```bash
openclaw config set discord.policies.requiredRole "AI Access"
```

Users without this role are ignored. This is useful for premium tiers or internal-only bots.

---

## Step 6: Restart and Verify

```bash
openclaw restart
openclaw logs | grep discord
```

You should see:
```
[discord] Bot connected as MyAIAssistant#1234
```

---

## Step 7: Test It

1. Go to a channel the bot has access to
2. Mention the bot or use a slash command: `@MyAIAssistant What can you do?`
3. The bot should respond with the agent's reply

---

## Sample Configuration

```json
{
  "plugins": {
    "discord": {
      "bots": {
        "main": {
          "token": "MTIzNDU2Nzg5MDEy...",
          "agent": "default"
        }
      },
      "policies": {
        "channelAllowlist": [1234567890123456789],
        "requiredRole": null
      }
    }
  }
}
```

---

## Advanced: Multiple Bots

Run multiple Discord bots, each bound to a different agent. Each bot requires a separate Discord application:

```bash
openclaw config set discord.bots.support.token "TOKEN_A"
openclaw config set discord.bots.support.agent "support-agent"

openclaw config set discord.bots.creative.token "TOKEN_B"
openclaw config set discord.bots.creative.agent "creative-agent"

openclaw restart
```

---

## Advanced: Thread Support

When a user starts a thread on the bot's reply, the bot continues the conversation within that thread. This keeps long conversations organized and avoids flooding the main channel.

Thread behavior is automatic -- no configuration needed. Each thread maintains its own conversation context.

---

## Streaming

Discord supports streaming -- the bot edits its message progressively as the agent generates a response. This is enabled by default.

---

## State Directory

```
~/.openclaw/discord/
├── sessions/            # Per-user session state
└── cache/               # Message cache for context
```

---

## Troubleshooting

### Bot not responding

1. **Check the token.** Run `openclaw config get discord.bots.main.token`. If you regenerated the token in the Developer Portal, the old one is invalid.
2. **Check intents.** The "Message Content Intent" must be enabled in the Developer Portal under Bot > Privileged Gateway Intents. Without it, the bot receives no message text.
3. **Check permissions.** The bot needs "Send Messages" and "Read Message History" in the channel. Check the channel's permission overwrites.
4. **Check logs.** `openclaw logs | grep discord` -- look for `401` (bad token) or `DISALLOWED_INTENTS` (missing intents).
5. **Restart.** Config changes require `openclaw restart`.

### Bot is online but ignores messages

1. **Channel allowlist.** If `channelAllowlist` is set, the bot only responds in listed channels.
2. **Role requirement.** If `requiredRole` is set, only users with that role get responses.
3. **Mention required.** By default, the bot only responds to messages that mention it. Send `@BotName your message`.

### Rate limits

Discord rate-limits bots aggressively. If the bot responds to many messages simultaneously, some may be delayed. OpenClaw queues messages to respect Discord's rate limits, but heavy usage in large servers may cause noticeable delays.

### Embeds not rendering

If the bot's responses show as plain text instead of rich embeds, check that the bot has the "Embed Links" permission in the channel.

---

## Next Steps

- [Telegram Bot](../telegram-bot/) -- the most mature channel
- [Slack App](../slack-app/) -- for workplace integration
- [Workspace](../../08-workspace/) -- customize your agent's personality
