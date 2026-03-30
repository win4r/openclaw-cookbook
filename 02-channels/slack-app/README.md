# Slack App

Connect your OpenClaw agent to a Slack workspace. Users interact with the bot via DMs, mentions in channels, or slash commands. Slack integration is well suited for workplace AI assistants where access is controlled by workspace membership.

**Time:** 30 minutes

**Plugin maturity:** Beta

---

## Prerequisites

- OpenClaw installed and running (`openclaw gateway` works)
- Admin access to a Slack workspace (or create a free one for testing)
- A publicly accessible URL for your OpenClaw instance (Slack uses webhooks, not polling). If running locally, use a tunnel like `ngrok` or `cloudflared`.
- At least one model provider configured

---

## Step 1: Create a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click "Create New App" and choose "From scratch"
3. Enter an app name (e.g., "OpenClaw AI") and select your workspace
4. You are now on the app's settings page

### Configure OAuth Scopes

Go to "OAuth & Permissions" and add these **Bot Token Scopes**:

- `app_mentions:read` -- receive mentions
- `chat:write` -- send messages
- `im:history` -- read DM history
- `im:read` -- receive DMs
- `im:write` -- send DMs
- `channels:history` -- read channel history (for context)
- `files:read` -- read uploaded files
- `files:write` -- send files

### Enable Event Subscriptions

Go to "Event Subscriptions":
1. Toggle "Enable Events" to On
2. Set the Request URL to: `https://<your-openclaw-host>/webhooks/slack`
3. Under "Subscribe to bot events", add:
   - `message.im` -- DM messages
   - `app_mention` -- mentions in channels
   - `message.channels` -- channel messages (optional, if you want the bot to observe all messages)
4. Click "Save Changes"

Slack will send a verification challenge to your URL. OpenClaw handles this automatically if the plugin is enabled and running.

### Enable Interactivity (Optional)

If you want the bot to use interactive elements (buttons, menus), go to "Interactivity & Shortcuts":
1. Toggle "Interactivity" to On
2. Set the Request URL to: `https://<your-openclaw-host>/webhooks/slack/interactive`
3. Click "Save Changes"

### Install the App

Go to "Install App" and click "Install to Workspace". Authorize the requested permissions. After installation, copy the **Bot User OAuth Token** (starts with `xoxb-`).

---

## Step 2: Enable the Slack Plugin

```bash
openclaw plugins enable slack
```

---

## Step 3: Configure Credentials

```bash
# Bot token (from Install App page)
openclaw config set channels.slack.accounts.main-app.botToken "xoxb-your-bot-token-here"

# Signing secret (from Basic Information > App Credentials)
openclaw config set channels.slack.accounts.main-app.signingSecret "a1b2c3d4e5f6g7h8i9j0..."
```

The signing secret verifies that incoming webhooks are genuinely from Slack.

---

## Step 4: Bind to an Agent

Add a binding in `openclaw.json`:

```json
"bindings": [
  { "agentId": "default", "match": { "channel": "slack", "accountId": "main-app" } }
]
```

---

## Step 5: Set Access Policies

### Channel Allowlist

Restrict which Slack channels the bot responds in:

```bash
openclaw config set channels.slack.policies.channelAllowlist '["C0123456789", "C9876543210"]'
```

To find a channel ID: right-click the channel name in Slack, select "View channel details", and scroll to the bottom.

### User Allowlist

Restrict which users can interact with the bot:

```bash
openclaw config set channels.slack.policies.userAllowlist '["U0123456789", "U9876543210"]'
```

If neither allowlist is set, the bot responds to all users in all channels it has access to.

---

## Step 6: Restart and Verify

```bash
openclaw gateway --restart
openclaw logs | grep slack
```

You should see:
```
[slack] Listening for events on /webhooks/slack
```

Verify the webhook is reachable by going to your Slack app's "Event Subscriptions" page -- the URL should show a green checkmark.

---

## Step 7: Test It

1. In Slack, send a DM to the bot
2. Or mention the bot in a channel: `@OpenClaw AI What can you do?`
3. The bot should respond

---

## Sample Configuration

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "accounts": {
        "main-app": {
          "botToken": "${SLACK_BOT_TOKEN}",
          "signingSecret": "${SLACK_SIGNING_SECRET}"
        }
      }
    }
  },
  "bindings": [
    { "agentId": "default", "match": { "channel": "slack", "accountId": "main-app" } }
  ]
}
```

---

## Important: Webhooks Require a Public URL

Unlike Telegram and Discord (which use polling), Slack uses webhooks. This means your OpenClaw instance must be reachable from the internet.

**For local development**, use a tunnel:

```bash
# Using ngrok
ngrok http 18789

# Using cloudflared
cloudflared tunnel --url http://localhost:18789
```

Then set the tunnel's public URL as your Slack app's Request URL. Update it each time the tunnel restarts (unless you use a fixed subdomain).

**For production**, deploy OpenClaw behind a reverse proxy (nginx, Caddy) with a proper domain and TLS certificate. See [Production](../../09-production/).

---

## Limitations

- **No streaming.** Slack's API does not support progressive message editing the way Telegram and Discord do. The bot sends the complete response as a single message. For long responses, the bot may send a "thinking" indicator and then replace it with the final message.
- **3-second response window.** Slack requires an HTTP 200 response within 3 seconds of receiving an event. OpenClaw sends an immediate acknowledgment and processes the message asynchronously, so this is handled automatically.
- **Message formatting.** Slack uses its own `mrkdwn` format, not standard Markdown. Most basic formatting (bold, italic, code blocks, links) translates correctly. Tables and some advanced Markdown features do not render as expected.

---

## Advanced: Slash Commands

To add slash commands (e.g., `/ask What is the weather?`):

1. In your Slack app settings, go to "Slash Commands"
2. Create a new command (e.g., `/ask`)
3. Set the Request URL to: `https://<your-openclaw-host>/webhooks/slack/commands`
4. Save and reinstall the app to your workspace

```bash
openclaw config set channels.slack.commands.enabled true
openclaw gateway --restart
```

---

## State Directory

```
~/.openclaw/slack/
├── sessions/            # Per-user session state
└── cache/               # Message cache for context
```

---

## Troubleshooting

### Bot not responding

1. **Check webhook connectivity.** In Slack app settings, "Event Subscriptions" should show a green checkmark next to the URL. If it shows an error, your URL is not reachable.
2. **Check the token.** Run `openclaw config get channels.slack.accounts.main-app.botToken`. Verify it starts with `xoxb-`.
3. **Check the signing secret.** A wrong signing secret causes all incoming events to be rejected. Check `openclaw logs` for `Invalid signature` errors.
4. **Check event subscriptions.** Make sure `message.im` and/or `app_mention` are listed under "Subscribe to bot events".
5. **Restart.** Config changes require `openclaw gateway --restart`.

### Bot only responds in DMs, not channels

The bot must be invited to a channel before it can respond there. Type `/invite @OpenClaw AI` in the channel. Also check that `app_mention` is in your event subscriptions.

### Duplicate responses

If the bot sends the same response multiple times, you may have duplicate event subscriptions or multiple OpenClaw instances polling the same Slack app. Check `openclaw logs` for duplicate event IDs.

### "This app isn't responding" error in Slack

This means Slack did not get a 200 response within 3 seconds. Check that OpenClaw is running and the webhook URL is correct. If using a tunnel, verify the tunnel is active.

### Token revoked after reinstall

When you reinstall a Slack app to a workspace, the old token is revoked and a new one is issued. Copy the new token from the "Install App" page and update your config.

---

## Next Steps

- [Telegram Bot](../telegram-bot/) -- faster setup, no webhook needed
- [Discord Bot](../discord-bot/) -- community-focused
- [Security](../../07-security/) -- harden your agent for workplace use
- [Production](../../09-production/) -- deploy with a proper domain
