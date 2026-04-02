# Module 11: Scheduler & Cron Jobs

> Automate your AI agent with scheduled tasks, recurring briefings, and timed reminders.

**Time:** 25 minutes  
**Prerequisites:** Module 01 (Quickstart)  
**You'll learn:** Cron jobs, one-shot reminders, delivery channels, heartbeat, and task monitoring

---

## Why Scheduling Matters

Your AI agent doesn't have to wait for you to talk first. With OpenClaw's built-in scheduler, your agent can:

- Deliver a **morning briefing** every day at 7 AM
- Send a **reminder** in 20 minutes
- Run a **weekly report** and post it to Slack
- **Monitor** a website on a recurring schedule
- Trigger actions based on **heartbeat** intervals

---

## Core Concepts

| Concept | What It Does |
|---------|-------------|
| **Cron job** | Recurring task on a schedule (cron expression) |
| **One-shot** | Single execution at a specific time or relative offset |
| **Delivery** | Where results go (Telegram, Slack, Discord, etc.) |
| **Session** | `main` (shared context) or `isolated` (clean each run) |
| **Heartbeat** | Periodic agent wake-up for background awareness |

---

## 1. Your First Cron Job

### One-Shot Reminder (Relative Time)

Set a reminder 20 minutes from now:

```bash
openclaw cron add \
  --name "Quick reminder" \
  --at "20m" \
  --session main \
  --system-event "Check battery status" \
  --wake now
```

### One-Shot Reminder (Absolute Time)

Schedule for a specific date/time:

```bash
openclaw cron add \
  --name "Deadline reminder" \
  --at "2026-04-15T09:00:00Z" \
  --session main \
  --system-event "Reminder: submit quarterly report" \
  --wake now \
  --delete-after-run
```

> **Tip:** Use `--delete-after-run` for one-time reminders that should clean up after themselves.

---

## 2. Recurring Jobs

### Daily Morning Briefing

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates from GitHub and email" \
  --model "opus" \
  --thinking high \
  --announce
```

### Weekly Report with Delivery

Send results to a Slack channel every Monday at 9 AM:

```bash
openclaw cron add \
  --name "Weekly report" \
  --cron "0 9 * * 1" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "Generate weekly project status report" \
  --model "opus" \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

### Deliver to Telegram

```bash
openclaw cron add \
  --name "Daily digest" \
  --cron "0 20 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "Summarize today's AI news highlights" \
  --announce \
  --channel telegram \
  --to "123456789"
```

### Cron Expression Reference

| Expression | Schedule |
|-----------|----------|
| `0 7 * * *` | Every day at 7:00 AM |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `*/30 * * * *` | Every 30 minutes |
| `0 0 1 * *` | First day of every month |
| `0 9,17 * * 1-5` | Weekdays at 9 AM and 5 PM |

---

## 3. Managing Cron Jobs

### List All Jobs

```bash
# Human-readable table
openclaw cron list --all

# Machine-readable JSON
openclaw cron list --all --json
```

### Check Scheduler Status

```bash
openclaw cron status
```

### View Run History

```bash
openclaw cron runs --id <job-id> --limit 50
```

### Edit a Job

```bash
openclaw cron edit <job-id> --message "Updated: summarize with bullet points" --model opus
```

### Enable / Disable

```bash
openclaw cron disable <job-id>
openclaw cron enable <job-id>
```

### Manual Execution

Test a job without waiting for the schedule:

```bash
openclaw cron run <job-id>

# Force execution even if already running
openclaw cron run <job-id> --force
```

### Remove a Job

```bash
openclaw cron rm <job-id>
```

---

## 4. Cron Configuration

Fine-tune the scheduler in `openclaw.json`:

```json5
{
  cron: {
    enabled: true,
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1,

    // Retry policy for failed one-shot jobs
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },

    // Session retention for isolated cron sessions
    sessionRetention: "24h",

    // Run log limits
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

| Key | Default | Description |
|-----|---------|-------------|
| `enabled` | `true` | Enable/disable the scheduler |
| `maxConcurrentRuns` | `1` | Max parallel cron executions |
| `retry.maxAttempts` | `3` | Retry count for failed jobs |
| `sessionRetention` | `"24h"` | How long to keep isolated session data |
| `runLog.maxBytes` | `"2mb"` | Max run log size |
| `runLog.keepLines` | `2000` | Max run log lines |

---

## 5. Heartbeat

Heartbeat is a separate mechanism that periodically wakes the agent in its **main session**. Unlike cron jobs, heartbeats don't create task records — they're lightweight check-ins.

### Configure Heartbeat

```json5
{
  agent: {
    heartbeat: {
      every: "30m",  // Wake every 30 minutes (0m = disabled)
    },
  },
}
```

### Heartbeat vs Cron

| | Heartbeat | Cron Job |
|-|-----------|----------|
| Session | Main (always) | Main or isolated |
| Creates task record | No | Yes |
| Has delivery | No | Yes (announce to channels) |
| Schedule | Fixed interval | Cron expression or one-shot |
| Use case | Background awareness | Scheduled actions |

> **When to use heartbeat:** Agent should periodically check for new messages, monitor state, or update awareness.  
> **When to use cron:** Agent should perform a specific action and optionally deliver results somewhere.

---

## 6. Cron via Tool Calls (JSON API)

Agents can also create cron jobs programmatically via tool calls:

```json
{
  "name": "Morning brief",
  "schedule": {
    "kind": "cron",
    "expr": "0 7 * * *",
    "tz": "America/Los_Angeles"
  },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

---

## 7. Tasks and ClawFlow

Every cron execution creates a **task record** that you can inspect:

- **Main-session** cron tasks default to `silent` notify policy
- **Isolated** cron tasks can announce to channels
- Tasks can trigger heartbeat wakes on completion

**ClawFlow** is the layer above tasks — it groups multiple task runs into a single job with a parent session context. See the [ClawFlow documentation](https://docs.openclaw.ai/automation/clawflow) for multi-step workflow orchestration.

---

## Practical Recipes

### Recipe A: Daily AI News Digest to Telegram

```bash
openclaw cron add \
  --name "AI news digest" \
  --cron "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --session isolated \
  --message "Search for today's top 5 AI news stories. Summarize each in 2 sentences. Include source links." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel telegram \
  --to "123456789"
```

### Recipe B: Hourly Website Monitor

```bash
openclaw cron add \
  --name "Site health check" \
  --cron "0 * * * *" \
  --session isolated \
  --message "Check https://mysite.com - verify it returns 200, measure response time, report any issues" \
  --announce \
  --channel slack \
  --to "channel:C_ALERTS"
```

### Recipe C: End-of-Day Summary

```bash
openclaw cron add \
  --name "EOD summary" \
  --cron "0 18 * * 1-5" \
  --tz "America/New_York" \
  --session main \
  --message "Summarize everything we discussed today. List action items and pending tasks." \
  --announce \
  --channel telegram \
  --to "123456789"
```

---

## Verification Checklist

- [ ] Create a one-shot reminder and verify it fires
- [ ] Create a recurring job and check `openclaw cron list`
- [ ] Run a job manually with `openclaw cron run`
- [ ] Verify delivery to your preferred channel
- [ ] Check run history with `openclaw cron runs`

---

## What's Next?

| Next Step | Module |
|-----------|--------|
| Connect mobile devices | [Module 12: Nodes](../12-nodes/) |
| Manage from browser | [Module 13: Control UI](../13-control-ui/) |
| Harden scheduled jobs | [Module 07: Security](../07-security/) |
