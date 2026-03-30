# Recipe: Your Personal AI on Telegram

Build a private AI assistant that lives on Telegram. It remembers your conversations, searches the web when you ask, and has a personality you define. Only you can talk to it.

**Time:** 30 minutes
**Difficulty:** Beginner
**Models:** Any (Claude, GPT, Gemini, etc.)
**Channel:** Telegram

## What You Will Build

```
Your Phone (Telegram)
    |
    v
@your_bot on Telegram
    |
    v
OpenClaw Gateway (localhost:18789)
    |
    ├── SOUL.md      -- custom personality
    ├── USER.md      -- what the agent knows about you
    ├── IDENTITY.md  -- name and appearance
    ├── Memory       -- LanceDB long-term memory
    └── Web Search   -- real-time information
```

When you are done, you will have a Telegram bot that:

- Responds to your messages with a personality you defined
- Remembers facts from past conversations and recalls them when relevant
- Searches the web when it needs current information
- Is locked down so only you can use it

---

## Prerequisites

- OpenClaw installed and initialized (`openclaw --version` should print a version)
- At least one API key configured in `~/.openclaw/.env`
- A Telegram account (the free app on your phone)

If you have not installed OpenClaw yet, complete the [Quickstart](../../01-quickstart/) first.

---

## Step 1: Create Your Telegram Bot

Open Telegram on your phone and search for `@BotFather`. Start a conversation and send:

```
/newbot
```

BotFather will ask you two things:

1. **Display name** for your bot (e.g., `My AI Assistant`). This is what users see in the chat header.
2. **Username** for your bot (e.g., `my_ai_assistant_bot`). This must end in `bot` and be globally unique on Telegram.

After you provide both, BotFather responds with a **bot token** that looks like this:

```
7123456789:AAG-some-random-string-here
```

Copy this token. You will need it in the next step.

**Important:** Keep this token secret. Anyone with this token can control your bot.

### Optional: Customize the Bot Profile

While still talking to BotFather, you can set a profile picture and description:

```
/setuserpic
/setdescription
/setabouttext
```

These are cosmetic -- they do not affect how the bot works.

---

## Step 2: Configure OpenClaw with the Bot Token

Enable the Telegram plugin and set the bot token:

```bash
# Enable the Telegram plugin
openclaw plugin enable telegram

# Set the bot token
openclaw config set telegram.bots.main.token "7123456789:AAG-some-random-string-here"

# Bind this bot to the default agent
openclaw config set telegram.bots.main.agent "default"
```

Verify the configuration was saved:

```bash
openclaw config get telegram
```

You should see your token (partially masked) and the agent binding.

---

## Step 3: Create a Custom Personality (SOUL.md)

The SOUL.md file defines your agent's personality, values, communication style, and behavioral boundaries. This is the most important workspace file -- it shapes every response.

Copy the provided example to your workspace:

```bash
cp workspace/SOUL.md ~/.openclaw/workspace/SOUL.md
```

Or create your own at `~/.openclaw/workspace/SOUL.md`. Here is what the example contains:

```markdown
# Soul

You are a personal AI assistant. You are direct, helpful, and low-ceremony.

## Communication Style
- Be concise. Default to short answers unless I ask for detail.
- Skip greetings and pleasantries. Get to the point.
- Use plain language. Avoid jargon unless I use it first.
- When you are uncertain, say so. Do not hedge with filler phrases.

## Values
- Accuracy over speed. If you are not sure, say "I don't know" or search the web.
- My time is valuable. Do not pad responses with obvious context.
- Remember what I have told you. If I mentioned a preference before, honor it.

## What You Can Do
- Answer questions using your knowledge and memory of our conversations.
- Search the web for current information when your knowledge is insufficient.
- Remember facts I tell you and recall them when relevant.
- Help me think through decisions, draft messages, plan tasks.

## What You Should Not Do
- Do not make up facts. If you do not know, say so or search.
- Do not give unsolicited advice. Answer what I asked.
- Do not repeat my question back to me before answering.
```

The key principle: **write SOUL.md the way you would brief a new personal assistant on day one.** Be specific about what you want and what you do not want.

See the [Workspace module](../../08-workspace/) for a full guide to SOUL.md authoring.

---

## Step 4: Create USER.md (What the Agent Knows About You)

USER.md seeds the agent with facts about you so it does not start from zero. This is different from memory -- memory is learned over time, USER.md is pre-loaded knowledge.

Copy the provided example:

```bash
cp workspace/USER.md ~/.openclaw/workspace/USER.md
```

Then edit `~/.openclaw/workspace/USER.md` with your actual information. The template looks like this:

```markdown
# User Profile

## Basics
- Name: [Your name]
- Location: [City, timezone]
- Languages: [Languages you speak]

## Work
- Role: [What you do]
- Tech stack: [If relevant]
- Work hours: [When you are typically available]

## Preferences
- Communication: [e.g., "concise, no fluff"]
- Topics of interest: [What you frequently ask about]
- Tools you use: [So the agent can give contextual advice]
```

You can add as much or as little as you want. The agent uses this as background context, not as a script.

---

## Step 5: Create IDENTITY.md

IDENTITY.md defines the agent's name and other metadata that appears in conversation.

Copy the provided example:

```bash
cp workspace/IDENTITY.md ~/.openclaw/workspace/IDENTITY.md
```

Or create `~/.openclaw/workspace/IDENTITY.md`:

```markdown
# Identity

- Name: Atlas
- Role: Personal AI Assistant
- Created by: [Your name]
```

This is minimal by design. The agent uses this when it needs to refer to itself by name.

---

## Step 6: Enable Memory (LanceDB)

Memory lets the agent remember facts across conversations. Without it, every conversation starts fresh.

```bash
# Enable the memory plugin
openclaw plugin enable memory-lancedb-pro

# Verify it is enabled
openclaw plugin list
```

Memory uses LanceDB for vector storage and Jina embeddings for semantic search. No external database or API key is needed -- it runs locally.

How it works:

1. During conversation, the agent extracts important facts from your messages
2. Facts are embedded and stored in a local LanceDB database at `~/.openclaw/memory/`
3. On each new message, the agent searches memory for relevant past facts
4. Retrieved facts are injected into the context so the agent can reference them

You can configure memory behavior in the config:

```bash
# Optional: tune memory settings
openclaw config set memory.autoExtract true
openclaw config set memory.maxRecallResults 10
```

To verify memory is working after setup:

```bash
# Tell your bot a fact (via Telegram or CLI)
openclaw chat "My favorite programming language is Rust."

# Later, ask about it
openclaw chat "What's my favorite programming language?"
# Should respond: Rust
```

---

## Step 7: Enable Web Search

Web search lets the agent find current information instead of relying only on its training data.

```bash
# Enable web search tool
openclaw config set tools.web_search.enabled true
```

No additional API key is needed -- OpenClaw uses a built-in search provider. The agent decides when to search based on the question. You can also explicitly ask: "Search the web for..." to force a search.

---

## Step 8: Set Access Policy (Lock It Down)

By default, the Telegram plugin uses a pairing system. Only users who are paired with the bot can talk to it. This means strangers who find your bot's username cannot use it.

Configure the access policy:

```bash
# Require pairing (default, but be explicit)
openclaw config set telegram.policies.pairingRequired true

# Optional: disable group chat (DMs only)
openclaw config set telegram.policies.allowGroups false
```

### How Pairing Works

1. Someone sends a message to your bot on Telegram
2. The bot responds asking them to request pairing
3. You (the admin) see the pairing request in your OpenClaw terminal
4. You approve or deny via `openclaw telegram:pair approve <user_id>`

Since this is a personal bot, the first user you pair will be yourself. After that, you can leave pairing required and simply never approve anyone else.

### Quick Pairing (Just You)

The fastest way to pair yourself:

```bash
# Restart the gateway to pick up all changes
openclaw restart

# Open Telegram and send any message to your bot
# You will see a pairing request in the terminal

# Approve it
openclaw telegram:pair approve <your_telegram_user_id>
```

After approval, your bot responds to your messages. All other users are blocked.

---

## Step 9: Test and Verify

Restart the gateway to pick up all configuration changes:

```bash
openclaw restart
```

### Verification Checklist

Run through each of these to confirm your setup works end to end:

- [ ] `openclaw status` shows the gateway is running
- [ ] `openclaw plugin list` shows `telegram` and `memory-lancedb-pro` as enabled
- [ ] Send "Hello" to your bot on Telegram -- it responds
- [ ] The response reflects the personality from SOUL.md (concise, direct, etc.)
- [ ] Send "My birthday is March 15th" -- the bot acknowledges
- [ ] Later, send "When is my birthday?" -- the bot recalls "March 15th" from memory
- [ ] Send "What is the weather in Tokyo right now?" -- the bot uses web search
- [ ] Have a friend try to message your bot -- they should be blocked or asked to pair

If any step fails, check `openclaw logs` for error details.

---

## Complete Configuration

Here is the full `openclaw.json` snippet for this recipe. You can also find it in [`config.json`](./config.json).

```json
{
  "agents": {
    "default": {
      "model": "anthropic:claude-sonnet-4-20250514",
      "workspace": "~/.openclaw/workspace"
    }
  },
  "plugins": {
    "telegram": {
      "bots": {
        "main": {
          "token": "YOUR_BOT_TOKEN_HERE",
          "agent": "default"
        }
      },
      "policies": {
        "pairingRequired": true,
        "allowGroups": false
      }
    },
    "memory-lancedb-pro": {
      "enabled": true,
      "autoExtract": true,
      "maxRecallResults": 10
    }
  },
  "tools": {
    "web_search": {
      "enabled": true
    }
  }
}
```

---

## File Listing

```
10-recipes/personal-ai-on-telegram/
  README.md              # This guide
  config.json            # openclaw.json snippet
  workspace/
    SOUL.md              # Personality definition
    IDENTITY.md          # Agent name and metadata
    USER.md              # User profile template
```

---

## Customization Ideas

Once the basic setup works, try these modifications:

### Change the Model

Swap the model provider without changing anything else:

```bash
# Use GPT instead of Claude
openclaw config set agents.default.model "openai:gpt-4o"

# Use a local Ollama model (free, fully private)
openclaw config set agents.default.model "ollama:llama3"
```

### Add Skills

Create reusable capabilities as markdown files in `~/.openclaw/workspace/skills/`:

```bash
mkdir -p ~/.openclaw/workspace/skills
```

Example: a translation skill at `~/.openclaw/workspace/skills/translate.md`:

```markdown
# Translate

When the user asks to translate text:
1. Detect the source language if not specified
2. Translate to the requested target language
3. Provide the translation only, no explanations unless asked
4. For ambiguous words, pick the most common translation and note alternatives
```

### Add More Tools

Enable shell command execution for power-user tasks:

```bash
openclaw config set tools.exec.enabled true
openclaw config set tools.exec.allowlist '["ls", "date", "curl", "python3"]'
```

**Warning:** The exec tool runs real commands on your machine. Always use an allowlist to restrict what commands the agent can run.

### Connect to Additional Channels

Your agent can be available on Telegram and Discord simultaneously:

```bash
openclaw plugin enable discord
openclaw config set discord.bots.main.token "YOUR_DISCORD_TOKEN"
openclaw config set discord.bots.main.agent "default"
openclaw restart
```

Both channels share the same agent, memory, and personality.

---

## Troubleshooting

### Bot does not respond on Telegram

1. Check `openclaw status` -- the gateway must be running
2. Check `openclaw logs` for errors
3. Verify your bot token is correct: `openclaw config get telegram.bots.main.token`
4. Make sure you have paired yourself (see Step 8)

### Memory does not recall facts

1. Check that `memory-lancedb-pro` is enabled: `openclaw plugin list`
2. After telling the bot a fact, wait a few seconds before asking about it (embedding takes a moment)
3. Check `~/.openclaw/memory/` exists and contains data files
4. Try `openclaw chat "Recall everything you know about me"` to see what is stored

### Web search returns no results

1. Check that `tools.web_search.enabled` is true: `openclaw config get tools.web_search`
2. Ensure your machine has internet access
3. Some queries work better with explicit phrasing: "Search for X" instead of just asking

### Agent personality does not match SOUL.md

1. Check the file is at `~/.openclaw/workspace/SOUL.md` (not a subdirectory)
2. The gateway reads workspace files on each request -- no restart needed for SOUL.md changes
3. Check `openclaw config get agents.default.workspace` points to the right directory

---

## Next Steps

- [Customer Support Bot](../customer-support-bot/) -- build a support agent with escalation rules
- [Module 05: Memory](../../05-memory/) -- deep dive into memory configuration and tuning
- [Module 08: Workspace](../../08-workspace/) -- advanced persona programming techniques
