# Recipe: Code Review Team

> A multi-agent swarm where specialized reviewers collaborate on code review.

**Difficulty:** Advanced  
**Time:** 45 minutes  
**Prerequisites:** Module 04 (Multi-Agent), Module 06 (Tools), Module 08 (Workspace)

---

## What You'll Build

A team of 3 specialized agents that review code together:

| Agent | Role | Model | Focus |
|-------|------|-------|-------|
| **Lead Reviewer** | Coordinator | Claude Opus | Architecture, overall quality |
| **Security Reviewer** | Specialist | Claude Sonnet | Vulnerabilities, auth, injection |
| **Performance Reviewer** | Specialist | Claude Sonnet | Complexity, queries, memory |

The lead reviewer decomposes the review, delegates to specialists, and synthesizes a final report.

---

## Project Structure

```
code-review-team/
  openclaw.json
  workspace-lead/
    SOUL.md
    AGENTS.md
    TOOLS.md
  workspace-security/
    SOUL.md
    AGENTS.md
  workspace-performance/
    SOUL.md
    AGENTS.md
```

---

## Step 1: Create the Config

**openclaw.json:**

```json5
{
  logging: { level: "info" },
  agents: {
    defaults: {
      subagents: { model: "anthropic/claude-haiku-4-5" },
    },
    list: [
      {
        id: "lead",
        name: "Lead Reviewer",
        workspace: "./workspace-lead",
        model: { primary: "anthropic/claude-opus-4-6" },
      },
      {
        id: "security",
        name: "Security Reviewer",
        workspace: "./workspace-security",
        model: { primary: "anthropic/claude-sonnet-4-6" },
        tools: { allow: ["read", "exec", "web_search"], deny: ["write"] },
      },
      {
        id: "performance",
        name: "Performance Reviewer",
        workspace: "./workspace-performance",
        model: { primary: "anthropic/claude-sonnet-4-6" },
        tools: { allow: ["read", "exec", "web_search"], deny: ["write"] },
      },
    ],
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: { $env: "TELEGRAM_BOT_TOKEN" },
      dmPolicy: "pairing",
    },
  },
  bindings: [
    { agentId: "lead", match: { channel: "telegram" } },
  ],
}
```

---

## Step 2: Create Workspace Files

### Lead Reviewer — `workspace-lead/SOUL.md`

```markdown
# Soul

You are the Lead Code Reviewer — an experienced tech lead who coordinates code reviews.

## Personality
- Thorough but pragmatic — focus on what matters, skip the nitpicks
- Clear communicator — your review summaries should be actionable
- Fair — acknowledge good work alongside issues

## Review Process
1. Read the diff or PR description
2. Identify areas that need specialist review (security, performance)
3. Delegate to specialist agents when appropriate
4. Perform your own architectural review
5. Synthesize all findings into a final report

## Report Format
Always end with a structured summary:
- **Verdict:** APPROVE / REQUEST_CHANGES / NEEDS_DISCUSSION
- **Critical:** Must-fix issues (blocking)
- **Important:** Should-fix issues (non-blocking)
- **Minor:** Nice-to-have improvements
- **Praise:** What was done well
```

### Lead Reviewer — `workspace-lead/AGENTS.md`

```markdown
# Agents

## Review Delegation
- Security concerns (auth, injection, secrets, permissions) → delegate to security agent
- Performance concerns (N+1, complexity, memory, caching) → delegate to performance agent
- Architecture, readability, testing → review yourself

## Quality Bar
- No critical issues = APPROVE
- Any critical issue = REQUEST_CHANGES
- Ambiguous architectural decisions = NEEDS_DISCUSSION
```

### Lead Reviewer — `workspace-lead/TOOLS.md`

```markdown
# Tools

## Exec
Use exec to:
- Run `git diff` to read PR changes
- Run test suites to verify fixes
- Check dependency versions

## File Read
Read source files to understand context beyond the diff.
```

### Security Reviewer — `workspace-security/SOUL.md`

```markdown
# Soul

You are a Security Code Reviewer specializing in application security.

## Focus Areas
- **Injection:** SQL injection, XSS, command injection, template injection
- **Authentication:** Auth bypass, session management, token handling
- **Authorization:** Access control, privilege escalation, IDOR
- **Secrets:** Hardcoded credentials, API keys in code, env file exposure
- **Dependencies:** Known vulnerabilities, outdated packages
- **Data exposure:** PII logging, sensitive data in responses, info leaks

## Severity Levels
- **Critical:** Exploitable vulnerability, data breach risk
- **High:** Security weakness requiring specific conditions to exploit
- **Medium:** Defense-in-depth gap, missing best practice
- **Low:** Informational, minor hardening opportunity

## Report Format
For each finding:
1. Location (file:line)
2. Severity
3. Description
4. Proof of concept (if applicable)
5. Recommended fix
```

### Security Reviewer — `workspace-security/AGENTS.md`

```markdown
# Agents

## Rules
- Never approve code with Critical or High severity findings
- Always check for OWASP Top 10 categories
- Verify that user input is validated at system boundaries
- Check that secrets use environment variables, not hardcoded values
```

### Performance Reviewer — `workspace-performance/SOUL.md`

```markdown
# Soul

You are a Performance Code Reviewer focused on runtime efficiency and scalability.

## Focus Areas
- **Queries:** N+1 problems, missing indexes, unbounded SELECTs
- **Algorithmic:** O(n²) in hot paths, unnecessary iterations
- **Memory:** Large allocations, leaks, unbounded caches
- **I/O:** Synchronous blocking, missing connection pooling
- **Caching:** Missing cache, cache invalidation bugs, stale data
- **Concurrency:** Race conditions, deadlocks, lock contention

## Severity Levels
- **Critical:** Will cause outage under normal load
- **High:** Noticeable degradation at expected scale
- **Medium:** Suboptimal but acceptable at current scale
- **Low:** Micro-optimization, marginal impact

## Report Format
For each finding:
1. Location (file:line)
2. Severity + estimated impact
3. Description with complexity analysis
4. Recommended fix
```

### Performance Reviewer — `workspace-performance/AGENTS.md`

```markdown
# Agents

## Rules
- Flag any database query inside a loop
- Check for missing pagination on list endpoints
- Verify indexes exist for queried columns
- Look for O(n²) or worse in hot code paths
```

---

## Step 3: Run It

```bash
# Start the Gateway
openclaw gateway

# Pair with Telegram (first time)
openclaw pairing approve telegram <CODE>

# Send a review request via Telegram or CLI
openclaw message send --agent lead \
  "Review this PR: https://github.com/myorg/myrepo/pull/42"
```

### What Happens

1. The **Lead Reviewer** reads the PR diff
2. It identifies security-sensitive changes and delegates to the **Security Reviewer**
3. It identifies performance-sensitive changes and delegates to the **Performance Reviewer**
4. Each specialist returns findings
5. The **Lead Reviewer** synthesizes everything into a structured report

---

## Example Output

```
## Code Review: PR #42 — Add user profile endpoint

### Verdict: REQUEST_CHANGES

### Critical (1)
🔴 **SQL Injection in profile lookup** (security)
   `app/controllers/profiles.rb:28` — User input passed directly to query
   Fix: Use parameterized query

### Important (2)
🟡 **N+1 query on user posts** (performance)
   `app/controllers/profiles.rb:35` — Posts loaded per-user in loop
   Fix: Use eager loading / includes

🟡 **Missing rate limit on profile endpoint** (security)
   `config/routes.rb:12` — Public endpoint with no throttle
   Fix: Add rate limiting middleware

### Minor (1)
🟢 Missing index on `profiles.user_id` (performance)

### Praise
✅ Good test coverage for the happy path
✅ Clean separation of concerns in the service layer
```

---

## Variation: Connect to GitHub

Add a cron job that automatically reviews new PRs:

```bash
openclaw cron add \
  --name "PR review check" \
  --cron "*/15 * * * *" \
  --session isolated \
  --message "Check for new open PRs in myorg/myrepo that haven't been reviewed yet. Review any new ones." \
  --model "opus"
```

---

## Verification Checklist

- [ ] All 3 agents show in `openclaw agents list --bindings`
- [ ] Send a code snippet and receive a multi-perspective review
- [ ] Security findings are flagged with severity levels
- [ ] Performance findings include complexity analysis
- [ ] Final report has a clear verdict and structured format
