# Recipes

End-to-end, runnable scenarios. Each recipe takes you from zero to a working system in about 30 minutes. Every recipe includes complete configuration files you can copy, a verification checklist, and ideas for customization.

**Recipes are self-contained.** You do not need to complete the earlier cookbook modules first. Each recipe tells you exactly what to install and configure. If you want deeper understanding of any component, the recipe links back to the relevant module.

---

## Recipe Index

| # | Recipe | What You Build | Difficulty | Time |
|---|--------|----------------|------------|------|
| 1 | [Personal AI on Telegram](./personal-ai-on-telegram/) | A private AI assistant with memory, web search, and custom personality -- accessible from your phone | Beginner | 30 min |
| 2 | [Customer Support Bot](./customer-support-bot/) | A multi-language support agent with knowledge base, escalation rules, and conversation history | Intermediate | 30 min |
| 3 | [Code Review Team](./code-review-team/) | A ClawTeam swarm that reviews PRs from multiple angles (security, performance, style) | Advanced | 45 min |
| 4 | [Research Assistant](./research-assistant/) | An agent that researches topics, summarizes findings, and delivers results via chat | Intermediate | 30 min |

---

## Before You Start

Every recipe assumes:

- **Node.js 18+** is installed (`node --version`)
- **OpenClaw is installed** (`npm install -g openclaw && openclaw init`)
- **One API key** is configured in `~/.openclaw/.env`

If you have not done this yet, complete the [Quickstart](../01-quickstart/) first (15 minutes).

---

## How Recipes Are Structured

Each recipe follows the same format:

```
10-recipes/<recipe-name>/
  README.md            # Step-by-step guide with explanations
  config.json          # openclaw.json snippet to copy
  workspace/           # Workspace files (SOUL.md, IDENTITY.md, etc.)
    SOUL.md
    IDENTITY.md
    USER.md
    skills/            # Skill files (if the recipe uses them)
```

**To use a recipe:**

1. Read the README for the full walkthrough
2. Copy the workspace files to your OpenClaw workspace directory
3. Merge the config.json settings into your `openclaw.json`
4. Follow the verification checklist to confirm everything works

---

## Choosing a Recipe

**"I just want a personal AI on my phone."**
Start with [Personal AI on Telegram](./personal-ai-on-telegram/). It is the simplest recipe and gives you a fully private assistant with memory that learns about you over time.

**"I need to handle customer inquiries."**
Start with [Customer Support Bot](./customer-support-bot/). It covers multi-language support, knowledge base integration, and escalation to humans when the bot cannot help.

**"I want automated code reviews."**
The [Code Review Team](./code-review-team/) recipe sets up a multi-agent swarm where each agent reviews from a different angle (security, performance, style). Requires familiarity with multi-agent concepts from [Module 04](../04-multi-agent/).

**"I need help with research and summarization."**
The [Research Assistant](./research-assistant/) recipe creates an agent that searches the web, synthesizes findings, and delivers structured summaries.

---

## Combining Recipes

Recipes are composable. A single OpenClaw gateway can run multiple agents simultaneously. After completing two or more recipes, you will have multiple specialized agents, each on their own channel or bot handle, all sharing the same gateway.

See [Module 04: Multi-Agent](../04-multi-agent/) for how to configure agent routing.
