# Channels

Channels connect your OpenClaw agents to messaging platforms. Each channel is a plugin that translates platform-specific messages into OpenClaw's internal format, and sends agent responses back to the user.

```
User on Telegram  ──>  telegram plugin  ──>  OpenClaw Gateway  ──>  Agent
User on Discord   ──>  discord plugin   ──>  OpenClaw Gateway  ──>  Agent
User on Slack     ──>  slack plugin     ──>  OpenClaw Gateway  ──>  Agent
User on WhatsApp  ──>  whatsapp plugin  ──>  OpenClaw Gateway  ──>  Agent
```

One gateway can run multiple channels simultaneously. Each channel can bind to a different agent, or multiple channels can share the same agent.

---

## Channel Comparison

| Feature | Telegram | Discord | Slack | WhatsApp | Twitch | iMessage |
|---------|----------|---------|-------|----------|--------|----------|
| **Setup difficulty** | Easy | Moderate | Moderate | Hard | Easy | Moderate |
| **Multiple bots** | Yes | Yes | Yes | No (one number) | No | No |
| **Group support** | Yes, with allowlist | Yes, with role-based | Yes, with channel-based | Yes, with group ID | Channel chat | Group chats |
| **DM access control** | Pairing system | Role-based | Workspace membership | Phone number | User ID allowlist | Phone allowlist |
| **Streaming** | Yes | Yes | No | No | No | No |
| **Rich formatting** | Markdown subset | Full Markdown | Block Kit / mrkdwn | Limited | Plain text | iMessage format |
| **File/image support** | Yes | Yes | Yes | Yes | No | Yes |
| **Free tier** | Unlimited bots | Unlimited bots | Free plan limits | Business API required | Free | Free (macOS) |
| **Plugin maturity** | Stable | Beta | Beta | Beta | Beta | Beta |
| **Best for** | Personal AI, small teams | Communities, gaming | Workplaces, enterprises | Customer-facing | Streamers | Apple ecosystem |

---

## Which Channel Should I Start With?

**Start with Telegram** if you:
- Want the fastest path from zero to working bot
- Need a personal AI assistant accessible from your phone
- Want DM and group support with fine-grained access control
- Care about streaming (seeing partial responses as they generate)

**Start with Discord** if you:
- Run a community server and want an AI bot in it
- Need role-based access control (e.g., only @premium members can talk to the bot)
- Want rich embeds and formatting in responses

**Start with Slack** if you:
- Want an AI agent inside your company workspace
- Need integration with existing Slack workflows
- Require enterprise-grade auth (SSO, SCIM)

**Start with WhatsApp** if you:
- Build a customer-facing product where users contact you via WhatsApp
- Need to reach users who only use WhatsApp (common in many regions)
- Are willing to deal with Meta's Business API approval process

**Start with Twitch** if you:
- Stream on Twitch and want an AI assistant in your chat
- Want to engage viewers with an interactive AI during streams

**Start with iMessage** if you:
- Live in the Apple ecosystem and want AI accessible from any Apple device
- Want a seamless experience across iPhone, iPad, and Mac

---

## How Channels Work

### Enabling a Channel

Every channel is an OpenClaw plugin. The pattern is the same for all:

```bash
# Enable the plugin
openclaw plugins enable <channel>

# Configure credentials
openclaw config set <channel>.<setting> "<value>"

# Restart to pick up changes
openclaw gateway --restart
```

### Configuration

Channel config lives in `openclaw.json` under `channels.<channel>`:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "streaming": "partial",
      "accounts": {
        "main-bot": {
          "botToken": "${TELEGRAM_BOT_TOKEN}",
          "dmPolicy": "pairing",
          "groupPolicy": "allowlist"
        }
      }
    },
    "discord": {
      "enabled": true,
      "accounts": {
        "main-bot": {
          "botToken": "${DISCORD_BOT_TOKEN}"
        }
      }
    }
  },
  "bindings": [
    { "agentId": "default", "match": { "channel": "telegram", "accountId": "main-bot" } },
    { "agentId": "default", "match": { "channel": "discord", "accountId": "main-bot" } }
  ]
}
```

### Binding Accounts to Agents

Each account within a channel can be bound to a different agent via the top-level `bindings` array:

```json
"bindings": [
  { "agentId": "support", "match": { "channel": "telegram", "accountId": "support-bot" } },
  { "agentId": "community", "match": { "channel": "discord", "accountId": "main-bot" } }
]
```

This lets you expose different agent personalities on different platforms or different bot handles.

### State and Data

Channel plugins store their runtime state in `~/.openclaw/<channel>/`. This includes session data, pairing records, and message caches. This directory is created automatically when the plugin starts.

---

## Guides

| Channel | Guide | Time |
|---------|-------|------|
| Telegram | [telegram-bot/](./telegram-bot/) | 20 min |
| Discord | [discord-bot/](./discord-bot/) | 30 min |
| Slack | [slack-app/](./slack-app/) | 30 min |
| WhatsApp | [whatsapp-business/](./whatsapp-business/) | 45 min |
| Twitch | [twitch/](./twitch/) | 20 min |
| iMessage | [imessage/](./imessage/) | 15 min |

### Plugin Channels (Community)

Additional channels available via plugins: **Mattermost**, **Matrix**, **Microsoft Teams**, **Nostr**, **Signal**, **IRC**. Check the [OpenClaw plugin registry](https://docs.openclaw.ai) for setup guides.

---

## Running Multiple Channels

You can enable all channels simultaneously:

```bash
openclaw plugins enable telegram
openclaw plugins enable discord
openclaw plugins enable slack
openclaw plugins enable whatsapp
openclaw gateway --restart
```

All channels share the same gateway, agents, memory, and tools. A user messaging on Telegram and another on Discord can talk to the same agent with the same long-term memory.

---

## Next Steps

- [Telegram Bot Setup](./telegram-bot/) -- recommended first channel
- [Model Providers](../03-model-providers/) -- configure which LLM powers your agent
- [Workspace](../08-workspace/) -- customize your agent's personality
