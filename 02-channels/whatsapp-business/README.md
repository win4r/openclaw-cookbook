# WhatsApp Business API

Connect your OpenClaw agent to WhatsApp via the Meta Cloud API. Users message a WhatsApp Business number and receive AI-powered responses. This is the most complex channel to set up due to Meta's verification and approval requirements.

**Time:** 45 minutes (setup) + days to weeks (Meta business verification)

**Plugin maturity:** Alpha

---

## Prerequisites

- OpenClaw installed and running (`openclaw gateway` works)
- A Meta Business account (create at [business.facebook.com](https://business.facebook.com))
- A phone number that is not already registered with WhatsApp (you cannot reuse a personal WhatsApp number)
- A publicly accessible URL for your OpenClaw instance (WhatsApp uses webhooks)
- Business verification completed with Meta (required for production, not needed for test numbers)
- At least one model provider configured

---

## Step 1: Set Up the WhatsApp Business Platform

### Create a Meta App

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Click "My Apps" then "Create App"
3. Select "Business" as the app type
4. Fill in the app name and contact email
5. Connect it to your Meta Business account

### Add WhatsApp to Your App

1. In your app dashboard, click "Add Product"
2. Select "WhatsApp" and click "Set Up"
3. You are now on the WhatsApp Getting Started page

### Get a Test Phone Number

Meta provides a free test phone number for development:
1. In the WhatsApp section, go to "API Setup"
2. You see a test phone number and a temporary access token
3. Copy the **Phone Number ID** and **Temporary Access Token**

The test number can only send messages to numbers you add to the "To" allowlist (up to 5 numbers during development).

### Get a Permanent Access Token

The temporary token expires in 24 hours. For a permanent token:
1. Go to "Business Settings" in your Meta Business account
2. Under "System Users", create a system user
3. Generate a token for this system user with the `whatsapp_business_messaging` permission
4. This token does not expire

---

## Step 2: Configure the Webhook

1. In your Meta app's WhatsApp section, go to "Configuration"
2. Under "Webhook", click "Edit"
3. Set the Callback URL to: `https://<your-openclaw-host>/webhooks/whatsapp`
4. Set the Verify Token to a random string you choose (e.g., `my-verify-token-abc123`)
5. Click "Verify and Save"
6. Subscribe to the `messages` webhook field

OpenClaw handles the verification challenge automatically if the plugin is enabled and the verify token matches.

---

## Step 3: Enable the WhatsApp Plugin

```bash
openclaw plugins enable whatsapp
```

---

## Step 4: Configure Credentials

```bash
# Access token (permanent, from system user)
openclaw config set channels.whatsapp.accounts.main.accessToken "EAAxxxxxxx..."

# Phone Number ID (from API Setup page)
openclaw config set channels.whatsapp.accounts.main.phoneNumberId "1234567890"

# Verify token (the random string you chose for the webhook)
openclaw config set channels.whatsapp.accounts.main.webhookVerifyToken "my-verify-token-abc123"
```

---

## Step 5: Bind to an Agent

Add a binding in `openclaw.json`:

```json
"bindings": [
  { "agentId": "default", "match": { "channel": "whatsapp", "accountId": "main" } }
]
```

---

## Step 6: Restart and Verify

```bash
openclaw gateway --restart
openclaw logs | grep whatsapp
```

You should see:
```
[whatsapp] Webhook listener active for phone 1234567890
```

---

## Step 7: Test It

1. Add your personal phone number to the test number's allowlist in the Meta developer console
2. Send a WhatsApp message to the test number
3. The bot should respond with the agent's reply

---

## Sample Configuration

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "accounts": {
        "main": {
          "accessToken": "${WHATSAPP_ACCESS_TOKEN}",
          "phoneNumberId": "${WHATSAPP_PHONE_NUMBER_ID}",
          "webhookVerifyToken": "${WHATSAPP_WEBHOOK_VERIFY_TOKEN}"
        }
      }
    }
  },
  "bindings": [
    { "agentId": "default", "match": { "channel": "whatsapp", "accountId": "main" } }
  ]
}
```

---

## Access Policies

### Number Allowlist

Restrict which phone numbers can interact with the bot:

```bash
openclaw config set channels.whatsapp.policies.numberAllowlist '["+1234567890", "+0987654321"]'
```

If not set, any number that messages the bot gets a response (subject to Meta's own restrictions).

### Group Support

WhatsApp groups are supported, but the bot only responds when mentioned or when a message starts with a configured trigger word:

```bash
openclaw config set channels.whatsapp.policies.groupTrigger "@bot"
```

---

## Moving to Production

### Business Verification

To use a real phone number (not the test number) and message any WhatsApp user:

1. Complete Meta Business Verification at [business.facebook.com](https://business.facebook.com) under Settings > Business Info > Verification
2. Submit your app for review in the developer console
3. Once approved, register your production phone number in the WhatsApp section of your app

This process takes days to weeks. Plan accordingly.

### Message Templates

WhatsApp requires pre-approved message templates for business-initiated conversations (messages you send first, not replies). User-initiated conversations (the user messages you first) allow free-form replies for 24 hours.

If your bot needs to send proactive messages, create templates in the Meta Business Manager under WhatsApp > Message Templates.

### Rate Limits

WhatsApp enforces tiered rate limits based on your account quality:
- New accounts: 250 messages per 24 hours
- Verified accounts: 1,000 - 100,000 messages per 24 hours depending on tier

Exceeding limits results in temporary blocks. Monitor your usage in the Meta developer console.

---

## Limitations

- **No streaming.** WhatsApp does not support progressive message editing. The bot sends the complete response as a single message.
- **Single phone number.** Unlike Telegram (where you can create multiple bots), WhatsApp ties a bot to one phone number. You cannot run multiple WhatsApp bots from the same gateway without multiple Meta apps and phone numbers.
- **24-hour window.** After a user sends a message, you have 24 hours to respond with free-form text. After that, you can only send pre-approved templates.
- **Media limitations.** WhatsApp has size limits for media (16 MB for most file types, 100 MB for video). Long responses may be truncated.
- **Formatting.** WhatsApp supports basic formatting: bold (`*text*`), italic (`_text_`), strikethrough (`~text~`), monospace (`` `text` ``). No tables, no headers, no links with custom text.

---

## State Directory

```
~/.openclaw/whatsapp/
├── sessions/            # Per-number session state
└── cache/               # Message cache for context
```

---

## Troubleshooting

### Bot not responding

1. **Check webhook status.** In the Meta developer console, go to WhatsApp > Configuration. The webhook URL should show as verified (green checkmark).
2. **Check the access token.** Run `openclaw config get channels.whatsapp.accounts.main.accessToken`. If using a temporary token, it may have expired. Generate a permanent one from a system user.
3. **Check the phone number ID.** This is the Phone Number ID from the API Setup page, not the phone number itself.
4. **Check webhook subscriptions.** Make sure you subscribed to the `messages` field in the webhook configuration.
5. **Check logs.** `openclaw logs | grep whatsapp` -- look for `403` (bad token), `400` (bad request), or `webhook verification failed`.

### "Unsupported message type" in logs

The bot received a message type it cannot process (e.g., a voice note, sticker, or location). Currently, the WhatsApp plugin processes text messages and images. Other message types are logged and ignored.

### Test number messages not arriving

1. Verify your personal number is in the test number's "To" allowlist in the Meta developer console.
2. Check that you are messaging the correct test number (shown in the API Setup page).
3. The test number may have a daily send limit of 250 messages.

### Webhook verification failing

1. Verify that `channels.whatsapp.accounts.main.webhookVerifyToken` in OpenClaw matches the verify token you entered in the Meta developer console.
2. Check that your OpenClaw instance is reachable from the internet at the webhook URL.
3. Check `openclaw logs` for the verification challenge -- it shows the expected and received tokens.

### Messages delayed

WhatsApp message delivery is not instant -- there can be delays of seconds to minutes depending on Meta's infrastructure. If delays are consistently over a minute, check your OpenClaw instance's response time and network connectivity.

---

## Next Steps

- [Telegram Bot](../telegram-bot/) -- much simpler setup, good for prototyping
- [Production](../../09-production/) -- deploy with proper domain and TLS
- [Security](../../07-security/) -- harden access control
