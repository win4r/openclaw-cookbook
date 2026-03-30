# Troubleshooting

Common issues and their fixes, organized by symptom.

---

## Gateway Won't Start

### Port already in use

**Symptom:** `Error: listen EADDRINUSE :::18789`

**Fix:**
```bash
# Find what's using the port
lsof -i :18789

# Kill it, or change the port
# In .env:
OPENCLAW_PORT=18790

# Or in openclaw.json:
# "gateway": { "port": 18790 }
```

### Missing dependencies

**Symptom:** `Error: Cannot find module 'xxx'`

**Fix:**
```bash
npm install
# or if using a global install
npm install -g openclaw
```

### Bad configuration

**Symptom:** `SyntaxError: Unexpected token` or `Invalid configuration`

**Fix:**
- JSON does not allow trailing commas. Check for `,"key": "value"}` patterns.
- JSON does not allow comments natively. If using `//` comments, verify OpenClaw's comment-stripping loader is active, or remove comments.
- Validate your config:
```bash
node -e "JSON.parse(require('fs').readFileSync('openclaw.json','utf8'))"
```

### Missing .env file

**Symptom:** Model API calls fail immediately, or `${VAR_NAME}` appears literally in logs.

**Fix:**
```bash
cp templates/.env.example .env
# Fill in your API keys
```

---

## Bot Not Responding

### Plugin not enabled

**Symptom:** Bot is online but ignores all messages.

**Fix:** Check openclaw.json:
```jsonc
"channels": {
  "telegram": {
    "enabled": true,  // Must be true, not false
    "accounts": {
      "main-bot": { "botToken": "${TELEGRAM_BOT_TOKEN}", "dmPolicy": "pairing" }
    }
  }
},
"bindings": [
  { "agentId": "main", "match": { "channel": "telegram", "accountId": "main-bot" } }
]
```

### Wrong bot token

**Symptom:** `401 Unauthorized` in logs, or bot appears offline.

**Fix:**
1. Get a fresh token from @BotFather on Telegram.
2. Update `.env`:
```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```
3. Restart the gateway.

### Pairing required

**Symptom:** Bot is online, receives messages (visible in debug logs), but does not reply.

**Fix:** The user needs to be paired/approved:
1. User sends `/start` to the bot on Telegram.
2. In your OpenClaw terminal, approve the pending pairing request.
3. For groups, the group itself must be approved.

### Agent not routed

**Symptom:** Plugin is enabled but logs show `No agent found for channel`.

**Fix:** Verify the binding's `agentId` matches an agent `id`:
```jsonc
// Binding references agent "main"
"bindings": [{ "agentId": "main", "match": { "channel": "telegram", "accountId": "main-bot" } }]

// Agent list must contain id "main"
"agents": { "list": [{ "id": "main", ... }] }
```

---

## Model Errors

### Invalid API key

**Symptom:** `401 Unauthorized` or `Invalid API Key` from model provider.

**Fix:**
```bash
# Test the key directly
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "content-type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

If the key is invalid, regenerate it from the provider's dashboard.

### Rate limited

**Symptom:** `429 Too Many Requests` or `Rate limit exceeded`.

**Fix:**
- Wait and retry (automatic in most cases).
- Configure a fallback model:
```jsonc
"model": {
  "primary": "anthropic/claude-sonnet-4-6",
  "fallbacks": ["openai/gpt-5.4"]
}
```
- Reduce request frequency (longer conversations = fewer API calls).
- Upgrade your API tier with the provider.

### Model not found

**Symptom:** `404 Not Found` or `Model not found: xxx`.

**Fix:**
- Check the exact model ID. Model names change across versions.
- Verify the model is listed in your provider config:
```jsonc
"anthropic": {
  "api": "anthropic",
  "models": [{ "id": "claude-sonnet-4-6", "name": "Claude Sonnet 4.6" }]
}
```
- Check if your API key has access to that model (some models require specific tier access).

### Context length exceeded

**Symptom:** `400 Bad Request` with message about token limit.

**Fix:**
- Reduce workspace file sizes (see [Workspace guide](../08-workspace/README.md) for token budgeting).
- Start new conversations more frequently.
- Use a model with a larger context window.

---

## Memory Not Working

### Plugin not enabled

**Symptom:** Agent never calls `memory_store` or `memory_recall`.

**Fix:** Enable the plugin:
```jsonc
"plugins": {
  "allow": ["memory-lancedb-pro"],
  "entries": {
    "memory-lancedb-pro": {
      "enabled": true,
      "config": {
        "embedding": {
          "provider": "openai-compatible",
          "apiKey": "${JINA_API_KEY}",
          "model": "jina-embeddings-v5-text-small",
          "baseURL": "https://api.jina.ai/v1",
          "dimensions": 1024
        },
        "dbPath": "~/.openclaw/memory/lancedb-pro",
        "autoCapture": true,
        "autoRecall": true
      }
    }
  }
}
```

### Jina API key invalid or missing

**Symptom:** `memory_store` fails with embedding error.

**Fix:**
```bash
# Test the key
curl https://api.jina.ai/v1/embeddings \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input":["test"],"model":"jina-embeddings-v5-text-small"}'
```

If invalid, get a new key at https://jina.ai/.

### Empty recall results

**Symptom:** `memory_recall` always returns nothing.

**Fix:**
- Verify something has been stored first. An empty database returns no results.
- Try a broader query.
- Check that `dbPath` exists and contains data files:
```bash
ls -la ~/.openclaw/memory/lancedb-pro/
```

### Agent not using memory tools

**Symptom:** Memory plugin is enabled but the agent never calls memory tools.

**Fix:** Add explicit instructions in TOOLS.md:
```markdown
## memory
- Call memory_recall at the start of each conversation.
- Call memory_store when the user shares preferences, corrections, or important facts.
```

Also check BOOT.md -- add a recall step to the startup routine.

---

## Permission Denied

### Exec command blocked

**Symptom:** Agent reports "command not allowed" or similar.

**Fix:** Check `exec-approvals.json`:
```bash
cat workspace/exec-approvals.json
```

If the command should be allowed, add a rule:
```json
{ "pattern": "your-command *", "action": "allow" }
```

If the mode is `allowlist` with `"default": "deny"`, every allowed command must have an explicit rule.

### Gateway auth failure

**Symptom:** `403 Forbidden` when making API requests.

**Fix:**
- Verify the auth token matches between client and server.
- Check that the `Authorization: Bearer <token>` header is included in requests.
- In single mode, auth should not be required. If it is failing, check that `mode` is set correctly.

### File system permission

**Symptom:** `EACCES: permission denied` when reading/writing files.

**Fix:**
```bash
# Check ownership
ls -la /opt/openclaw/

# Fix ownership
sudo chown -R openclaw:openclaw /opt/openclaw/data/
```

Ensure the OpenClaw process user has read/write access to:
- The workspace directory
- The data directory (LanceDB, conversations)
- The logs directory

---

## Performance Issues

### High response latency

**Symptom:** Agent takes 10+ seconds to respond.

**Causes and fixes:**

1. **Large workspace files.** Reduce total workspace to under 4,000 tokens.
2. **Long conversation history.** Start new conversations periodically.
3. **Slow model.** Opus is slower than Sonnet, which is slower than Haiku. Match model to task complexity.
4. **Tool chain delays.** Each tool call adds a round trip. Minimize unnecessary tool use.
5. **Network latency to API providers.** Check with:
```bash
time curl -s -o /dev/null https://api.anthropic.com/v1/messages
```

### High memory usage

**Symptom:** Node.js process consuming excessive RAM.

**Fixes:**
- Set a memory limit: `NODE_OPTIONS=--max-old-space-size=1024`
- Large LanceDB indexes consume memory. If the index grows beyond 100K entries, consider pruning old low-importance memories.
- Restart the process periodically (pm2 handles this with `max_memory_restart`).

### LanceDB slow queries

**Symptom:** `memory_recall` takes several seconds.

**Fixes:**
- Reduce `embedding.dimensions` from 1024 to 512.
- Ensure `dbPath` is on a local SSD, not a network mount.
- Prune old entries to keep the index lean.

---

## Quick Diagnostic Commands

```bash
# Is OpenClaw running?
pgrep -f openclaw || echo "Not running"

# Check the port
lsof -i :18789

# Health check
curl -s http://localhost:18789/health | python3 -m json.tool

# Recent logs (pm2)
pm2 logs openclaw --lines 50

# Recent logs (systemd)
journalctl -u openclaw --since "5 minutes ago"

# Test Anthropic API key
curl -s https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-6","max_tokens":5,"messages":[{"role":"user","content":"ping"}]}' \
  | python3 -m json.tool

# Test Jina API key
curl -s https://api.jina.ai/v1/embeddings \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"input":["test"],"model":"jina-embeddings-v5-text-small"}' \
  | python3 -m json.tool

# Workspace token estimate
wc -c workspace/*.md 2>/dev/null | tail -1 | awk '{print int($1/4), "estimated tokens"}'

# LanceDB size
du -sh ~/.openclaw/memory/lancedb-pro/ 2>/dev/null || echo "No LanceDB data found"
```
