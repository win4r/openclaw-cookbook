# 09 - Production Deployment

## Overview

Running OpenClaw in production means making it reliable, secure, and observable. This guide covers the operational concerns beyond basic setup.

## Process Management

OpenClaw is a long-running Node.js process. Use a process manager to keep it alive.

### systemd (Linux)

Create `/etc/systemd/system/openclaw.service`:

```ini
[Unit]
Description=OpenClaw AI Gateway
After=network.target

[Service]
Type=simple
User=openclaw
Group=openclaw
WorkingDirectory=/opt/openclaw
ExecStart=/usr/bin/node /opt/openclaw/server.js
Restart=always
RestartSec=5
EnvironmentFile=/opt/openclaw/.env

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/openclaw/data

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable openclaw
sudo systemctl start openclaw
sudo systemctl status openclaw

# View logs
journalctl -u openclaw -f
```

### pm2 (cross-platform)

```bash
# Install pm2
npm install -g pm2

# Start OpenClaw
pm2 start server.js --name openclaw --env production

# Auto-restart on crash
pm2 startup
pm2 save

# Monitor
pm2 monit
pm2 logs openclaw
```

pm2 ecosystem file (`ecosystem.config.js`):

```javascript
module.exports = {
  apps: [{
    name: "openclaw",
    script: "server.js",
    cwd: "/opt/openclaw",
    env: {
      NODE_ENV: "production"
    },
    max_memory_restart: "1G",
    log_date_format: "YYYY-MM-DD HH:mm:ss",
    error_file: "/var/log/openclaw/error.log",
    out_file: "/var/log/openclaw/out.log",
    merge_logs: true
  }]
};
```

### launchd (macOS)

Create `~/Library/LaunchAgents/com.openclaw.gateway.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw.gateway</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/node</string>
        <string>/Users/you/openclaw/server.js</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/Users/you/openclaw</string>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/you/openclaw/logs/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/you/openclaw/logs/stderr.log</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>NODE_ENV</key>
        <string>production</string>
    </dict>
</dict>
</plist>
```

```bash
launchctl load ~/Library/LaunchAgents/com.openclaw.gateway.plist
launchctl list | grep openclaw
```

## Logging and Monitoring

### Log levels

Set via environment variable:

```bash
OPENCLAW_LOG_LEVEL=info  # debug, info, warn, error
```

### Structured logging

OpenClaw outputs JSON-structured logs in production. Key fields:

```json
{
  "timestamp": "2026-03-30T10:15:30Z",
  "level": "info",
  "agent": "main",
  "channel": "telegram",
  "event": "message_received",
  "userId": "user_123",
  "latencyMs": 2340
}
```

### Log aggregation

Forward logs to your monitoring stack:

```bash
# pm2 + journalctl to a log file, then ship with your agent (Datadog, Fluentd, etc.)
OPENCLAW_LOG_FILE=/var/log/openclaw/app.log
```

### Key metrics to monitor

- **Response latency**: time from message received to response sent
- **Error rate**: 4xx/5xx responses, tool failures, model API errors
- **Token usage**: per-agent, per-model, per-day
- **Memory (system)**: Node.js heap usage, LanceDB size
- **Uptime**: process restarts, crash frequency

## Multi-User Isolation

In multi-user mode, each user gets isolated conversation state:

```jsonc
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "${OPENCLAW_GATEWAY_TOKEN}"
    }
  }
}
```

Isolation boundaries:
- Each user has separate conversation history.
- Memory (LanceDB) can be scoped per-user or shared, depending on config.
- Exec commands run as the same OS user (the OpenClaw process user). For stronger isolation, use containerized agents.

**Warning:** Multi-user mode isolates conversations, not system access. All users share the same exec tool and file system. If users should not have access to each other's data, run separate OpenClaw instances.

## Cost Tracking and Control

LLM API costs can accumulate quickly. Strategies for control:

### Model tiering

Route simple requests to cheap models:

```jsonc
{
  "agents": {
    "list": [
      {
        "id": "general",
        "model": { "primary": "anthropic/claude-haiku-4-5" }
      },
      {
        "id": "complex",
        "model": { "primary": "anthropic/claude-opus-4-6" }
      }
    ]
  }
}
```

### Usage limits

Set per-user or per-agent limits:
- Daily message count caps
- Monthly token budget per agent
- Rate limiting on the gateway

### Cost monitoring

Track costs by:
1. Logging token counts per request (input + output tokens)
2. Multiplying by model pricing
3. Aggregating by agent, user, and time period

Example log query pattern:
```
agent=general model=haiku input_tokens=500 output_tokens=1200 cost_usd=0.0004
```

## Backup and Restore

### What to back up

| Data | Location | Priority |
|------|----------|----------|
| Configuration | `openclaw.json`, `.env` | Critical |
| Workspace files | `workspace/` | Critical |
| LanceDB data | `~/.openclaw/memory/lancedb-pro/` | High |
| Conversation history | `data/conversations/` | Medium |
| Logs | `logs/` | Low |

### Backup script

```bash
#!/bin/bash
BACKUP_DIR="/backups/openclaw/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Config and workspace
cp openclaw.json "$BACKUP_DIR/"
cp -r workspace/ "$BACKUP_DIR/workspace/"
cp .env "$BACKUP_DIR/.env"

# LanceDB (stop writes during backup for consistency)
cp -r ~/.openclaw/memory/lancedb-pro/ "$BACKUP_DIR/memory-lancedb/"

# Compress
tar -czf "$BACKUP_DIR.tar.gz" -C /backups/openclaw "$(date +%Y%m%d)"
rm -rf "$BACKUP_DIR"

echo "Backup saved to $BACKUP_DIR.tar.gz"
```

### Restore

```bash
tar -xzf /backups/openclaw/20260330.tar.gz -C /opt/openclaw/
# Restart the gateway
sudo systemctl restart openclaw
```

## Gateway Security in Production

### Reverse proxy

Never expose the OpenClaw port directly. Use nginx or Caddy as a reverse proxy with TLS:

```nginx
server {
    listen 443 ssl http2;
    server_name openclaw.example.com;

    ssl_certificate /etc/letsencrypt/live/openclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support (if used)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Firewall

```bash
# Only allow localhost access to the OpenClaw port
sudo ufw allow from 127.0.0.1 to any port 18789
sudo ufw deny 18789
```

### Run as a non-root user

Create a dedicated user:

```bash
sudo useradd -r -s /bin/false openclaw
sudo chown -R openclaw:openclaw /opt/openclaw
```

## Health Checks

### HTTP health endpoint

OpenClaw exposes a health check endpoint:

```bash
curl http://localhost:18789/health
# {"status":"ok","uptime":3600,"agents":["main"],"plugins":["telegram","memory-lancedb-pro"]}
```

### Monitoring script

```bash
#!/bin/bash
HEALTH=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:18789/health)
if [ "$HEALTH" != "200" ]; then
    echo "OpenClaw health check failed: HTTP $HEALTH"
    # Send alert (email, Slack, PagerDuty, etc.)
    # Optionally restart
    sudo systemctl restart openclaw
fi
```

Add to crontab for periodic checks:

```bash
# Check every 5 minutes
*/5 * * * * /opt/openclaw/scripts/healthcheck.sh >> /var/log/openclaw/healthcheck.log 2>&1
```

### Readiness indicators

Before considering the gateway "ready":
- HTTP health endpoint returns 200
- At least one agent is loaded
- All enabled plugins are connected (Telegram bot polling active, etc.)
- LLM provider API keys are valid (test with a minimal completion)

## Production Checklist

- [ ] Process manager configured (systemd/pm2/launchd)
- [ ] Gateway auth enabled with `"mode": "token"` and a strong token
- [ ] Reverse proxy with TLS
- [ ] Firewall restricting direct port access
- [ ] Running as non-root user
- [ ] Exec approvals in allowlist mode, default deny
- [ ] Log level set to info (not debug)
- [ ] Log rotation configured
- [ ] Backup schedule in place
- [ ] Health check monitoring active
- [ ] Cost alerts configured
- [ ] SOUL.md hardened (prompt leak, jailbreak prevention)
- [ ] `.env` excluded from version control
