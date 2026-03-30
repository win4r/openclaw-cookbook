# Telegram Bot

Connect your OpenClaw agent to Telegram. Users can message your bot in DMs or add it to group chats. Telegram is the most mature channel plugin -- it supports streaming, pairing-based DM access control, group allowlists, and multiple bots per gateway.

**Time:** 20 minutes

---

## Prerequisites

- OpenClaw installed and running (`openclaw start` works)
- A Telegram account
- At least one model provider configured (e.g., `ANTHROPIC_API_KEY` in `~/.openclaw/.env`)

---

## Step 1: Create a Bot via BotFather

1. Open Telegram and search for `@BotFather`
2. Send `/newbot`
3. Choose a display name (e.g., "My AI Assistant")
4. Choose a username (must end in `bot`, e.g., `my_ai_assistant_bot`)
5. BotFather gives you a token like `7123456789:AAH1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6`
6. Save this token -- you need it in the next step

Optional BotFather settings (send these commands to BotFather):
- `/setdescription` -- what users see before starting a conversation
- `/setabouttext` -- the bot's "About" section
- `/setuserpic` -- upload an avatar for the bot
- `/setcommands` -- set the bot's command menu (e.g., `/start`, `/help`)

---

## Step 2: Enable the Telegram Plugin

```bash
openclaw plugin enable telegram
```

This registers the Telegram channel. No restart needed yet.

---

## Step 3: Configure the Bot Token

```bash
openclaw config set telegram.bots.main.token "7123456789:AAH1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6"
```

This writes the token to `openclaw.json` under `plugins.telegram.bots.main.token`.

---

## Step 4: Bind the Bot to an Agent

By default, the bot uses the `default` agent. To use a specific agent:

```bash
openclaw config set telegram.bots.main.agent "my-agent"
```

If you only have one agent and have not configured `AGENTS.md` beyond the default, skip this step.

---

## Step 5: Restart and Verify

```bash
openclaw restart
```

Check the logs for a successful Telegram connection:

```bash
openclaw logs | grep telegram
```

You should see a line like:
```
[telegram] Bot @my_ai_assistant_bot connected (polling)
```

---

## Step 6: Test It

1. Open Telegram and search for your bot's username
2. Send `/start` or any message
3. The bot should respond using your configured agent and model

If the bot does not respond, see [Troubleshooting](#troubleshooting) below.

---

## Access Policies

### DM Access: Pairing

By default, anyone who finds your bot can message it. To restrict DM access, enable pairing:

```bash
openclaw config set telegram.policies.pairingRequired true
openclaw restart
```

With pairing enabled:
- A new user sends a message to your bot
- The bot replies with a pairing request and a code
- You approve the pairing from the OpenClaw CLI or control UI
- Only approved users can continue chatting

To approve a pending pairing:

```bash
openclaw telegram approve <user_id>
```

To list pending pairings:

```bash
openclaw telegram pairings
```

Pairing records are stored in `~/.openclaw/telegram/pairings.json`.

### Group Access: Allowlist

To control which Telegram groups the bot responds in:

```bash
# Add a group to the allowlist (use the group's chat ID)
openclaw config set telegram.policies.groupAllowlist '[-1001234567890, -1009876543210]'
openclaw restart
```

To find a group's chat ID:
1. Add the bot to the group
2. Send a message in the group
3. Check `openclaw logs` -- the log shows the chat ID for each incoming message

If `groupAllowlist` is not set, the bot responds in all groups it is added to. Once set, it only responds in listed groups.

---

## Sample Configuration

Complete `openclaw.json` example with Telegram configured:

```json
{
  "plugins": {
    "telegram": {
      "bots": {
        "main": {
          "token": "7123456789:AAH1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6",
          "agent": "default"
        }
      },
      "policies": {
        "pairingRequired": true,
        "groupAllowlist": [-1001234567890]
      }
    }
  }
}
```

---

## Advanced: Multiple Bots

You can run multiple Telegram bots, each bound to a different agent:

```bash
# Support bot
openclaw config set telegram.bots.support.token "TOKEN_A"
openclaw config set telegram.bots.support.agent "support-agent"

# Code review bot
openclaw config set telegram.bots.coder.token "TOKEN_B"
openclaw config set telegram.bots.coder.agent "code-agent"

openclaw restart
```

This produces a config like:

```json
{
  "plugins": {
    "telegram": {
      "bots": {
        "support": {
          "token": "TOKEN_A",
          "agent": "support-agent"
        },
        "coder": {
          "token": "TOKEN_B",
          "agent": "code-agent"
        }
      },
      "policies": {
        "pairingRequired": true
      }
    }
  }
}
```

Each bot is a separate Telegram account (created via BotFather), but they all run inside the same OpenClaw gateway and share memory, tools, and plugins.

---

## Streaming

Telegram supports partial streaming -- the bot edits its message in real time as the agent generates a response. This is enabled by default. The user sees text appear progressively rather than waiting for the full response.

Streaming uses Telegram's `editMessage` API. There is a rate limit (~30 edits per minute per chat), so OpenClaw batches edits to stay within the limit.

---

## State Directory

Telegram plugin state lives in `~/.openclaw/telegram/`:

```
~/.openclaw/telegram/
├── pairings.json        # Approved and pending pairing records
├── sessions/            # Per-user session state
└── cache/               # Message cache for context
```

This directory is created automatically. You can delete it to reset all Telegram state (pairings, sessions).

---

## Troubleshooting

### Bot not responding

1. **Check the token.** Run `openclaw config get telegram.bots.main.token` and verify it matches BotFather.
2. **Check the logs.** Run `openclaw logs | grep telegram` and look for errors. Common errors:
   - `401 Unauthorized` -- token is wrong or revoked. Get a new one from BotFather.
   - `409 Conflict` -- another process is polling with the same token. Stop any other bots using this token (including BotFather's built-in test).
3. **Check the plugin is enabled.** Run `openclaw plugin list` and confirm telegram is listed.
4. **Restart.** Run `openclaw restart` -- config changes require a restart.

### Pairing issues

1. **User says they never got a pairing prompt.** Check that `pairingRequired` is `true` in config. If it was just enabled, restart.
2. **Approved user still cannot chat.** Check `~/.openclaw/telegram/pairings.json` for their user ID. If their entry shows `"status": "pending"`, approve it with `openclaw telegram approve <user_id>`.
3. **Cannot find user ID.** Ask the user to send any message to the bot, then check `openclaw logs` -- the user ID appears in the log entry.

### Group permissions

1. **Bot does not respond in a group.** Check that the group's chat ID is in `groupAllowlist`. Group IDs are negative numbers (e.g., `-1001234567890`).
2. **Bot responds to every message in a group.** By default, bots in Telegram groups only receive messages that mention them (`@bot_username`) or start with a command (`/`). To receive all messages, disable privacy mode via BotFather: send `/setprivacy` and choose `Disable`.
3. **Bot was added to a group but shows no log entry.** Make sure the bot is still a member of the group. Some group admins may have removed it.

### Message formatting issues

Telegram supports a subset of Markdown. If agent responses appear with raw formatting characters, check that your agent's system prompt does not produce unsupported syntax (e.g., tables render poorly on Telegram).

---

## Next Steps

- [Discord Bot](../discord-bot/) -- add another channel
- [Workspace](../../08-workspace/) -- customize your agent's personality
- [Memory](../../05-memory/) -- give your agent long-term memory
- [Personal AI on Telegram recipe](../../10-recipes/personal-ai-on-telegram/) -- full end-to-end setup
