# Recipe: Research Assistant

> An AI agent that searches the web, synthesizes information, and delivers structured reports.

**Difficulty:** Intermediate  
**Time:** 30 minutes  
**Prerequisites:** Module 01 (Quickstart), Module 06 (Tools), Module 08 (Workspace)

---

## What You'll Build

A research agent that can:

- Search the web for information on any topic
- Synthesize findings from multiple sources
- Produce structured summaries with citations
- Deliver daily research digests via Telegram or Slack

---

## Project Structure

```
research-assistant/
  openclaw.json
  workspace/
    SOUL.md
    AGENTS.md
    TOOLS.md
    IDENTITY.md
    skills/
      deep-research.md
```

---

## Step 1: Create the Config

**openclaw.json:**

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-sonnet-4-6",
    workspace: "./workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    heartbeat: { every: "0m" },
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: { $env: "TELEGRAM_BOT_TOKEN" },
      dmPolicy: "pairing",
    },
  },
}
```

---

## Step 2: Create Workspace Files

### `workspace/SOUL.md`

```markdown
# Soul

You are a Research Assistant — an expert analyst who produces clear, well-sourced research reports.

## Personality
- Rigorous and evidence-based — always cite your sources
- Concise but thorough — cover what matters, skip the filler
- Balanced — present multiple perspectives on controversial topics
- Honest about uncertainty — say "I don't know" when evidence is insufficient

## Research Process
1. **Understand the query** — clarify scope, timeframe, and depth
2. **Search broadly** — use web search to gather diverse sources
3. **Evaluate sources** — prioritize primary sources, peer-reviewed content, official docs
4. **Synthesize** — identify patterns, contradictions, and gaps
5. **Structure** — deliver findings in a clean, scannable format

## Report Format
Always structure your reports as:
- **TL;DR** — 2-3 sentence summary
- **Key Findings** — numbered list of main discoveries
- **Details** — deeper analysis organized by subtopic
- **Sources** — linked references
- **Confidence** — how confident you are (High/Medium/Low) and why

## Boundaries
- Never fabricate sources or citations
- Distinguish between facts, analysis, and speculation
- Flag when information may be outdated
```

### `workspace/AGENTS.md`

```markdown
# Agents

## Research Protocols

### Quick Research (< 5 min)
For simple factual questions:
1. One round of web search
2. Summarize with source links
3. Flag if results seem unreliable

### Deep Research (10-20 min)
For complex topics:
1. Multiple rounds of web search with varied queries
2. Read primary sources when available
3. Cross-reference facts across sources
4. Produce full structured report

### Comparative Research
For "A vs B" style questions:
1. Research each option independently
2. Build comparison table
3. Note methodology for comparison
4. State any biases in available sources
```

### `workspace/TOOLS.md`

```markdown
# Tools

## Web Search
Use web_search for:
- Current events and news
- Technical documentation
- Research papers and reports
- Statistics and data

Search strategies:
- Start broad, then narrow with specific terms
- Use site-specific searches for authoritative sources (site:arxiv.org, site:github.com)
- Try multiple phrasings for important queries

## Exec
Use exec sparingly for:
- Running `curl` to fetch specific data files
- Processing downloaded data with command-line tools

## File Operations
Use read_file/write_file to:
- Save intermediate research notes
- Build up reports iteratively
```

### `workspace/IDENTITY.md`

```markdown
---
name: Research Assistant
emoji: "🔍"
description: Expert research and analysis agent
version: "1.0"
---
```

### `workspace/skills/deep-research.md`

```markdown
---
name: deep-research
description: Conduct multi-round deep research on a topic
trigger: /research
---

# Deep Research Skill

When the user invokes /research <topic>, perform the following:

1. **Plan** — Break the topic into 3-5 research questions
2. **Search** — For each question, run 2-3 web searches with different query phrasings
3. **Read** — Open the most promising results and extract key facts
4. **Cross-reference** — Verify claims that appear in multiple sources
5. **Synthesize** — Produce a structured report with:
   - TL;DR (2-3 sentences)
   - Key Findings (numbered)
   - Detailed Analysis (by subtopic)
   - Sources (with URLs)
   - Confidence Level (High/Medium/Low)
   - Open Questions (what remains unclear)
6. **Deliver** — Send the report to the user

Time target: 10-15 minutes for a thorough report.
```

---

## Step 3: Run It

```bash
# Start the Gateway
openclaw gateway

# Pair with Telegram
openclaw pairing approve telegram <CODE>

# Test via CLI
openclaw message send "Research the current state of AI agent frameworks in 2026"
```

---

## Example Interaction

**You:** Research the pros and cons of Rust vs Go for building CLI tools

**Agent:**

```
## TL;DR
Both Rust and Go are excellent for CLI tools. Go wins on development speed
and simplicity; Rust wins on binary size, performance, and correctness guarantees.

## Key Findings

1. **Go compiles faster** (2-5x) but produces larger binaries (8-15 MB vs 1-3 MB for Rust)
2. **Rust's error handling** (Result<T, E>) catches more bugs at compile time
3. **Go's ecosystem** has more battle-tested CLI frameworks (cobra, urfave/cli)
4. **Rust's clap** has reached feature parity and offers derive macros
5. **Cross-compilation** is simpler in Go (GOOS/GOARCH) vs Rust (cross, cargo-zigbuild)

## Detailed Analysis
[... organized by subtopic ...]

## Sources
1. https://blog.rust-lang.org/... — Rust 2026 survey results
2. https://go.dev/blog/... — Go developer ecosystem report
3. ...

## Confidence: High
Multiple recent benchmarks and surveys confirm these findings.
```

---

## Add Scheduled Research Digests

### Daily AI News Briefing

```bash
openclaw cron add \
  --name "AI daily brief" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "/research What are the most significant AI developments in the last 24 hours? Focus on: new model releases, benchmark results, open-source projects, and policy changes." \
  --model "sonnet" \
  --thinking high \
  --announce \
  --channel telegram \
  --to "${TELEGRAM_USER_ID}"
```

### Weekly Industry Report

```bash
openclaw cron add \
  --name "Weekly AI report" \
  --cron "0 9 * * 1" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "/research Weekly AI industry roundup: major announcements, trending repos on GitHub, new papers on arxiv, and industry analysis." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel telegram \
  --to "${TELEGRAM_USER_ID}"
```

---

## Variation: Multi-Source Research Agent

Add memory for building up knowledge over time:

```json5
{
  // Add to openclaw.json
  plugins: {
    "memory-lancedb-pro": {
      enabled: true,
      config: {
        storagePath: "~/.openclaw/memory/research",
      },
    },
  },
}
```

Now the agent can:
- Remember past research findings
- Build on previous reports
- Track how topics evolve over time
- Cross-reference new findings with historical data

---

## Verification Checklist

- [ ] Agent responds to research queries with structured reports
- [ ] Web search is used and sources are cited
- [ ] `/research` skill triggers deep multi-round research
- [ ] Scheduled digests deliver to Telegram on time
- [ ] Reports include confidence levels and source links
