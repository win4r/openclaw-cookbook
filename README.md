# 🦞 OpenClaw Cookbook

**Your AI agent gateway, running on your machine, talking to your users, using any model you choose.**

OpenClaw is a self-hosted, multi-channel AI agent gateway. This cookbook gives you production-ready recipes to go from zero to a fully deployed AI agent system — with real configs you can copy, real docker-compose files you can run, and real scenarios you can ship.

> **This is not documentation.** The [official docs](https://github.com/anthropics/openclaw) cover every flag and option. This cookbook covers *how to actually build things* — the patterns, the gotchas, the recipes that work in production.

[中文版](./README_zh.md)

---

## What You Can Build

```
┌─────────────────────────────────────────────────────────┐
│                    Your Users                           │
│   Telegram  ·  WhatsApp  ·  Discord  ·  Slack  · Web   │
└─────────────┬───────────────────────────────┬───────────┘
              │                               │
              ▼                               ▼
┌─────────────────────────────────────────────────────────┐
│                   OpenClaw Gateway                      │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │  Agent A  │  │  Agent B  │  │  Agent C  │  Routing   │
│  │ (Support) │  │ (Coding)  │  │(Research) │  & Auth    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘             │
│       │              │              │                    │
│  ┌────┴──────────────┴──────────────┴────┐             │
│  │         Model Provider Pool           │             │
│  │  Claude · GPT · Gemini · DeepSeek     │             │
│  │  MiniMax · Kimi · Ollama (local)      │             │
│  └───────────────────────────────────────┘             │
│                                                         │
│  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Memory  │  │  Tools  │  │  Skills  │  │ Plugins │ │
│  │(LanceDB)│  │  (exec) │  │  (.md)   │  │  (npm)  │ │
│  └─────────┘  └─────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

**One gateway. Multiple agents. Any model. Every channel.**

---

## Cookbook Modules

| # | Module | What You'll Build | Time |
|---|--------|-------------------|------|
| **01** | [Quickstart](./01-quickstart/) | Running OpenClaw instance with one command | 15 min |
| **02** | [Channels](./02-channels/) | Telegram bot, Discord bot, Slack app, WhatsApp | 30 min each |
| **03** | [Model Providers](./03-model-providers/) | Multi-model setup with fallbacks and routing | 20 min |
| **04** | [Multi-Agent](./04-multi-agent/) | Specialized agents + ClawTeam swarm orchestration | 45 min |
| **05** | [Memory](./05-memory/) | LanceDB long-term memory with semantic recall | 30 min |
| **06** | [Tools](./06-tools/) | Custom tools, exec approvals, MCP integration | 40 min |
| **07** | [Security](./07-security/) | SOUL.md hardening, auth, exec allowlists | 30 min |
| **08** | [Workspace](./08-workspace/) | AGENTS.md, TOOLS.md, IDENTITY.md — persona programming | 30 min |
| **09** | [Production](./09-production/) | Monitoring, logging, multi-user isolation, cost control | 45 min |
| **10** | [Recipes](./10-recipes/) | End-to-end runnable scenarios | 30 min each |

**Total: ~6 hours to master everything.** Or jump straight to a [recipe](#recipes) and learn by building.

---

## Quick Taste (5 minutes)

### Install

```bash
# Install OpenClaw
npm install -g openclaw

# Verify
openclaw --version
```

### Configure Your First Agent

```bash
# Initialize workspace
openclaw setup

# Add your model provider key
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> ~/.openclaw/.env

# Start the gateway
openclaw gateway
```

### Talk to It

```bash
# CLI mode
openclaw message send "What can you do?"

# Or connect a Telegram bot (see 02-channels/)
openclaw plugins enable telegram
openclaw config set telegram.bots.main.token "YOUR_BOT_TOKEN"
openclaw gateway --restart
```

That's it. Your private AI agent is running. Now make it useful — pick a module above or jump to a recipe below.

---

## Recipes

Complete, runnable scenarios. Each recipe includes all configs, a docker-compose file (where applicable), and a verification checklist.

| Recipe | Description | Models Used | Channels |
|--------|-------------|-------------|----------|
| [Personal AI on Telegram](./10-recipes/personal-ai-on-telegram/) | Your own AI assistant with memory, web search, and personality | Any | Telegram |
| [Customer Support Bot](./10-recipes/customer-support-bot/) | Multi-language support agent with escalation rules and knowledge base | Claude + fallback | Telegram, Web |
| [Code Review Team](./10-recipes/code-review-team/) | ClawTeam swarm that reviews PRs from multiple angles | Claude + GPT | GitHub + Slack |
| [Research Assistant](./10-recipes/research-assistant/) | Agent that researches topics, summarizes findings, delivers via chat | GPT + Gemini | Telegram, Discord |

---

## Why OpenClaw

| | Cloud AI APIs | ChatGPT/Claude Apps | **OpenClaw** |
|---|---|---|---|
| **Data stays on your machine** | No | No | Yes |
| **Use any model** | One at a time | No | Yes, with fallbacks |
| **Multi-channel** | Build it yourself | No | Built-in |
| **Multiple specialized agents** | Build it yourself | No | Built-in |
| **Long-term memory** | Build it yourself | Limited | LanceDB + semantic search |
| **Custom tools** | Build it yourself | Limited | Exec + MCP + plugins |
| **Cost** | Per-token | $20/mo per app | Your API keys only |

---

## Key Concepts

Before diving in, here are the building blocks:

### Workspace — Your Agent's Brain

```
~/.openclaw/workspace/
├── SOUL.md          # Personality and values
├── AGENTS.md        # Operational rules and strategies
├── TOOLS.md         # Tool-specific instructions
├── IDENTITY.md      # Name, emoji, avatar
├── USER.md          # What the agent knows about you
├── MEMORY.md        # Curated long-term notes
├── BOOT.md          # Startup health checks
├── HEARTBEAT.md     # Periodic health checks (optional)
├── skills/          # Reusable skill definitions
└── checklists/      # Pre-flight safety checks
```

Each markdown file shapes how your agent thinks and behaves. This is **persona programming** — you define your agent's character through prose, not code.

### Agents — Specialized Personalities

One gateway can host multiple agents, each with its own workspace, model, and channel bindings:

```
Agent: "Support"     → Claude      → Telegram Bot @support_bot
Agent: "Coder"       → GPT-5      → Telegram Bot @code_bot
Agent: "Researcher"  → Gemini      → Discord Bot
```

### Plugins — Extend Everything

```bash
# Built-in
openclaw plugins enable telegram
openclaw plugins enable memory-lancedb-pro

# Community (npm)
npm install -g @tencent-weixin/openclaw-weixin
openclaw plugins enable openclaw-weixin
```

---

## Project Structure

```
openclaw-cookbook/
├── 01-quickstart/           # Zero to running in 15 minutes
├── 02-channels/             # Connect to messaging platforms
│   ├── telegram-bot/
│   ├── whatsapp-business/
│   ├── discord-bot/
│   └── slack-app/
├── 03-model-providers/      # Configure LLM backends
├── 04-multi-agent/          # Multiple agents + ClawTeam swarm
├── 05-memory/               # LanceDB long-term memory
├── 06-tools/                # Custom tools and MCP
├── 07-security/             # Hardening and access control
├── 08-workspace/            # Persona programming with markdown
├── 09-production/           # Deploy, monitor, scale
├── 10-recipes/              # End-to-end runnable scenarios
├── templates/               # Copy-paste config files
└── troubleshooting/         # Common issues and fixes
```

---

## Prerequisites

- **Node.js** 18+ (recommend 22 LTS)
- **One API key** from any supported provider (Anthropic, OpenAI, MiniMax, etc.)
- **5 minutes** for quickstart, or ~6 hours for the full cookbook

Optional:
- **Telegram account** (for the most popular channel integration)
- **tmux** (for ClawTeam multi-agent swarm)
- **Python 3.10+** (for ClawTeam only)

---

## Contributing

Found a bug? Have a recipe idea? PRs welcome.

- **Add a recipe**: Create a new directory under `10-recipes/` with a README, configs, and verification steps
- **Fix an error**: Open an issue or PR — accuracy matters more than coverage
- **Translate**: We're building bilingual (EN/ZH) — translations of any module are welcome

---

## License

MIT - see [LICENSE](./LICENSE)

---

Built by [CortexReach](https://github.com/CortexReach). Powered by [OpenClaw](https://github.com/anthropics/openclaw).
