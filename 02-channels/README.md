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

| Feature | Telegram | Discord | Slack | WhatsApp |
|---------|----------|---------|-------|----------|
| **Setup difficulty** | Easy | Moderate | Moderate | Hard |
| **Multiple bots per channel** | Yes | Yes | Yes (one app, multiple bots) | No (one number) |
| **Group support** | Yes, with allowlist | Yes, with role-based | Yes, with channel-based | Yes, with group ID |
| **DM access control** | Pairing system | Role-based | Workspace membership | Phone number |
| **Streaming (partial responses)** | Yes | Yes | No (Slack API limitation) | No |
| **Rich formatting** | Markdown subset | Full Markdown | Block Kit / mrkdwn | Limited |
| **File/image support** | Yes | Yes | Yes | Yes |
| **Free tier** | Unlimited bots | Unlimited bots | Free plan limits | Business API required |
| **Plugin maturity** | Stable | Beta | Beta | Alpha |
| **Best for** | Personal AI, small teams | Communities, gaming | Workplaces, enterprises | Customer-facing |

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

---

## How Channels Work

### Enabling a Channel

Every channel is an OpenClaw plugin. The pattern is the same for all:

```bash
# Enable the plugin
openclaw plugin enable <channel>

# Configure credentials
openclaw config set <channel>.<setting> "<value>"

# Restart to pick up changes
openclaw restart
```

### Configuration

Channel config lives in `openclaw.json` under `plugins.<channel>`:

```json
{
  "plugins": {
    "telegram": {
      "bots": {
        "main": {
          "token": "123456:ABC-DEF...",
          "agent": "default"
        }
      },
      "policies": {
        "pairingRequired": true
      }
    },
    "discord": {
      "bots": {
        "main": {
          "token": "MTIz...",
          "agent": "default"
        }
      }
    }
  }
}
```

### Binding Bots to Agents

Each bot within a channel can be bound to a different agent:

```bash
# Telegram bot @support_bot uses the "support" agent
openclaw config set telegram.bots.support.agent "support"

# Discord bot uses the "community" agent
openclaw config set discord.bots.main.agent "community"
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

---

## Running Multiple Channels

You can enable all channels simultaneously:

```bash
openclaw plugin enable telegram
openclaw plugin enable discord
openclaw plugin enable slack
openclaw plugin enable whatsapp
openclaw restart
```

All channels share the same gateway, agents, memory, and tools. A user messaging on Telegram and another on Discord can talk to the same agent with the same long-term memory.

---

## Next Steps

- [Telegram Bot Setup](./telegram-bot/) -- recommended first channel
- [Model Providers](../03-model-providers/) -- configure which LLM powers your agent
- [Workspace](../08-workspace/) -- customize your agent's personality
