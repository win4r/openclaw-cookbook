# 06 - Tools

## Overview

OpenClaw agents can use tools to interact with the outside world: run shell commands, search the web, read/write files, store memories, and connect to external services via MCP.

Tools are the bridge between "thinking" and "doing." Configuring them correctly is the difference between a helpful agent and a dangerous one.

## Exec Tool (Shell Commands)

The exec tool lets the agent run shell commands on the host machine. This is powerful and requires careful configuration.

### Basic usage

When the agent decides it needs to run a command, it calls the exec tool:

```
exec({ "command": "ls -la /var/log/app/" })
exec({ "command": "git status" })
exec({ "command": "npm test" })
```

### Approval system

By default, exec requires user approval for each command. This is controlled by `exec-approvals.json` in the workspace directory.

## exec-approvals.json

This file defines which commands the agent can run without asking, which require approval, and which are blocked entirely.

### Structure

```json
{
  "mode": "allowlist",
  "rules": [
    {
      "pattern": "git *",
      "action": "allow",
      "description": "All git commands"
    },
    {
      "pattern": "npm test",
      "action": "allow",
      "description": "Run tests"
    },
    {
      "pattern": "ls *",
      "action": "allow",
      "description": "List files"
    },
    {
      "pattern": "cat *",
      "action": "allow",
      "description": "Read files"
    },
    {
      "pattern": "rm *",
      "action": "deny",
      "description": "Never allow deletion"
    }
  ],
  "default": "ask"
}
```

### Modes

| Mode | Behavior |
|------|----------|
| `allowlist` | Only commands matching an `allow` rule run without asking. Everything else requires approval or is denied. This is the recommended mode. |
| `permissive` | Commands run unless they match a `deny` rule. Faster but riskier. Only use for local dev. |

### Rule fields

| Field | Description |
|-------|-------------|
| `pattern` | Glob pattern matched against the command string. `*` matches anything. |
| `action` | `allow` (run without asking), `deny` (block entirely), `ask` (prompt for approval). |
| `description` | Human-readable note (not used by the system, but useful for maintenance). |

### Default action

The `default` field determines what happens when no rule matches:
- `"ask"` -- prompt the user for approval (recommended)
- `"allow"` -- run without asking (dangerous)
- `"deny"` -- block the command

### Recommended starter config

```json
{
  "mode": "allowlist",
  "rules": [
    { "pattern": "git status", "action": "allow" },
    { "pattern": "git diff *", "action": "allow" },
    { "pattern": "git log *", "action": "allow" },
    { "pattern": "ls *", "action": "allow" },
    { "pattern": "cat *", "action": "allow" },
    { "pattern": "head *", "action": "allow" },
    { "pattern": "tail *", "action": "allow" },
    { "pattern": "wc *", "action": "allow" },
    { "pattern": "npm test", "action": "allow" },
    { "pattern": "npm run lint", "action": "allow" },
    { "pattern": "rm *", "action": "deny" },
    { "pattern": "sudo *", "action": "deny" },
    { "pattern": "chmod *", "action": "deny" },
    { "pattern": "curl * | *", "action": "deny" }
  ],
  "default": "ask"
}
```

## MCP Integration

MCP (Model Context Protocol) lets agents connect to external tools and data sources over a standardized protocol.

### Configuring MCP servers

MCP servers are defined in `mcp.json` or within openclaw.json:

```jsonc
{
  "mcp": {
    "servers": {
      "database": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"],
        "description": "PostgreSQL access"
      },
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/documents"],
        "description": "Document filesystem access"
      }
    }
  }
}
```

### How agents use MCP tools

MCP tools appear to the agent just like built-in tools. The agent calls them by name, and OpenClaw handles the protocol translation.

```
# Agent sees a tool called "database.query" from the postgres MCP server
database.query({ "sql": "SELECT * FROM users LIMIT 10" })
```

### Finding MCP servers

Community MCP servers: https://github.com/modelcontextprotocol/servers

Common useful servers:
- `@modelcontextprotocol/server-postgres` -- PostgreSQL queries
- `@modelcontextprotocol/server-filesystem` -- scoped file access
- `@modelcontextprotocol/server-brave-search` -- web search via Brave
- `@modelcontextprotocol/server-github` -- GitHub API access

## Built-in Tools

OpenClaw provides several tools out of the box:

| Tool | Description |
|------|-------------|
| `exec` | Run shell commands (with approval system) |
| `web_search` | Search the web and return results |
| `read_file` | Read a file from the workspace |
| `write_file` | Write or overwrite a file |
| `edit_file` | Apply a patch to an existing file |
| `memory_store` | Store information in long-term memory (requires plugin) |
| `memory_recall` | Recall information from long-term memory (requires plugin) |

Tool availability depends on the agent's configuration and which plugins are enabled.

## Writing Custom Skills

Skills are markdown files in the `workspace/skills/` directory that teach the agent new capabilities.

### Skill file format (SKILL.md pattern)

```
workspace/
  skills/
    deploy.md
    code-review.md
    daily-standup.md
```

Each skill file follows this structure:

```markdown
# Deploy

## Trigger

Use this skill when the user says "deploy", "ship it", or "push to production".

## Steps

1. Run `npm test` and verify all tests pass.
2. Run `npm run build` and verify no errors.
3. Run `git push origin main`.
4. Run `./scripts/deploy.sh staging` and wait for success.
5. Verify the staging URL returns 200.
6. Run `./scripts/deploy.sh production`.
7. Report the deploy status.

## Rules

- Never deploy if tests fail.
- Always deploy to staging before production.
- If any step fails, stop and report the error.
```

### How skills work

The agent reads skill files as part of its workspace context. When a user request matches a skill's trigger, the agent follows the defined steps. Skills are not code -- they are structured instructions that guide the agent's behavior.

### Tips for writing skills

- Be specific about triggers so the agent knows when to activate the skill.
- Number the steps so the agent follows them in order.
- Include failure handling ("if X fails, do Y").
- Keep skills focused on one task. If a skill exceeds ~50 lines, split it.
