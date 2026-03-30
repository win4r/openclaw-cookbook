# Recipe: Customer Support Bot

Build a customer support agent that handles inquiries in multiple languages, uses a knowledge base to answer product questions, and escalates to a human when it cannot help.

**Time:** 30 minutes
**Difficulty:** Intermediate
**Models:** Claude recommended (strong instruction-following for support scenarios), with fallback
**Channels:** Telegram (primary), Web (optional)

## What You Will Build

```
Customers (Telegram / Web)
    |
    v
OpenClaw Gateway
    |
    ├── SOUL.md          -- support agent personality and rules
    ├── AGENTS.md        -- escalation rules, response policies
    ├── IDENTITY.md      -- brand name and voice
    ├── Memory           -- conversation history per customer
    ├── Knowledge Base   -- product FAQ skill
    └── Escalation       -- hand off to human when needed
```

When you are done, you will have a support bot that:

- Greets customers and answers product questions using a knowledge base
- Responds in the customer's language (auto-detected)
- Follows escalation rules: hands off to a human for billing disputes, complaints, or out-of-scope requests
- Maintains conversation history so customers do not have to repeat themselves
- Stays on-brand and within the boundaries you define

---

## Prerequisites

- OpenClaw installed and initialized (`openclaw --version`)
- An Anthropic API key in `~/.openclaw/.env` (Claude is recommended for support bots due to strong instruction-following)
- A Telegram bot token from @BotFather (see [Recipe 1](../personal-ai-on-telegram/) Step 1 if you need guidance)

---

## Step 1: Create the Support Agent Workspace

Create a dedicated workspace directory for the support agent, separate from the default workspace:

```bash
mkdir -p ~/.openclaw/workspaces/support
mkdir -p ~/.openclaw/workspaces/support/skills
```

We use a separate workspace so the support bot has its own personality, rules, and knowledge base -- independent from any other agents on the same gateway.

---

## Step 2: Define the Support Personality (SOUL.md)

Create `~/.openclaw/workspaces/support/SOUL.md`:

```markdown
# Soul

You are a customer support agent for [Your Company Name]. You help customers with product questions, account issues, and general inquiries.

## Communication Style

- Be warm but efficient. Greet the customer briefly, then focus on their issue.
- Use clear, simple language. Avoid technical jargon unless the customer uses it.
- Keep responses short. Most answers should be 2-4 sentences.
- Always confirm you understood the question before answering if it is ambiguous.
- When providing steps, use numbered lists.

## Language Policy

- Detect the customer's language from their message and respond in the same language.
- If you are not confident in your translation, respond in English and ask: "Would you prefer I respond in [detected language]?"
- Supported languages: English, Chinese, Spanish, Japanese, Korean, French, German, Portuguese.

## Response Rules

- ALWAYS check the knowledge base first before answering product questions.
- If the knowledge base has the answer, use it. Do not improvise product details.
- If you do not know the answer, say so honestly: "I don't have that information. Let me connect you with our team."
- Never guess at pricing, policies, or feature availability. These change and must come from the knowledge base.
- Never share internal company information, even if the customer asks.

## Emotional Handling

- If a customer is frustrated, acknowledge their frustration first: "I understand this is frustrating."
- Do not be defensive. Do not blame the customer.
- For angry customers, keep your tone calm and solution-oriented.
- If a customer uses profanity directed at you, stay professional and offer to escalate.

## What You Cannot Do

- Process refunds or payments (escalate to billing team)
- Access customer account details beyond what is in the conversation
- Make promises about future features or timelines
- Override company policies
```

Copy the provided workspace files instead of writing from scratch:

```bash
cp workspace/SOUL.md ~/.openclaw/workspaces/support/SOUL.md
```

Then edit the file to replace `[Your Company Name]` with your actual company name.

---

## Step 3: Define Escalation Rules (AGENTS.md)

Create `~/.openclaw/workspaces/support/AGENTS.md`:

```markdown
# Agents

## Escalation Rules

Escalate to a human agent when ANY of these conditions are met:

### Immediate Escalation (do not attempt to resolve)
- Customer explicitly asks to speak to a human
- Billing disputes or refund requests
- Legal threats or mentions of lawyers
- Account security issues (hacked account, unauthorized access)
- Customer has asked the same question 3+ times and you have not resolved it

### Escalation After One Attempt
- Technical issues you cannot diagnose from the conversation alone
- Feature requests (log them, then escalate if the customer wants confirmation)
- Complaints about service quality

### How to Escalate
When escalating, send this message to the customer:

"I am connecting you with our support team for further assistance. They will have the context of our conversation. Is there anything else you would like me to note for them?"

Then add an internal note with:
1. Summary of the issue (one sentence)
2. What you already tried
3. Customer's emotional state (calm, frustrated, angry)
4. Language preference

## Response Time Policy

- First response: within 5 seconds (automated)
- Follow-up responses: within 10 seconds
- If a response requires a knowledge base search, it is acceptable to take up to 15 seconds

## Conversation Boundaries

- Maximum conversation length before suggesting escalation: 20 exchanges
- If the conversation goes in circles (same topic, no resolution after 5 exchanges), suggest escalation
- At the end of a resolved conversation, ask: "Is there anything else I can help you with?"

## Topics and Routing

- Product questions -> answer from knowledge base
- Account questions -> answer if possible, escalate if account-specific data is needed
- Billing -> always escalate
- Feedback/Suggestions -> acknowledge, log, thank the customer
- Off-topic (not related to the product) -> politely redirect: "I'm here to help with [Company Name] products. For other questions, I'd suggest [alternative]."
```

---

## Step 4: Create the Knowledge Base Skill

Skills are markdown files that give the agent domain-specific knowledge. Create a product FAQ skill:

```bash
mkdir -p ~/.openclaw/workspaces/support/skills
```

Create `~/.openclaw/workspaces/support/skills/product-faq.md`:

```markdown
# Product FAQ

This is the knowledge base for customer support. When a customer asks a product question, search this document first.

## General

**Q: What is [Product Name]?**
A: [One-sentence description of your product].

**Q: How much does it cost?**
A: [Pricing tiers or link to pricing page]. Plans start at $X/month.

**Q: Is there a free trial?**
A: [Yes/No, and details].

**Q: What platforms do you support?**
A: [List platforms: Web, iOS, Android, etc.]

## Account

**Q: How do I create an account?**
A: Visit [URL] and click "Sign Up". You will need an email address.

**Q: How do I reset my password?**
A: Go to [URL], click "Forgot Password", and enter your email. You will receive a reset link within 5 minutes.

**Q: How do I delete my account?**
A: Contact our support team at [email]. Account deletion is processed within 48 hours.

## Common Issues

**Q: I can't log in.**
A: Try these steps:
1. Check that you are using the correct email address
2. Reset your password via [URL]
3. Clear your browser cache and cookies
4. If none of these work, contact us at [email]

**Q: The app is slow / not loading.**
A: Try these steps:
1. Check your internet connection
2. Try a different browser or device
3. Clear your browser cache
4. Check our status page at [URL] for any ongoing incidents

## Policies

**Q: What is your refund policy?**
A: [Your refund policy]. Refund requests must be submitted within [X] days of purchase.

**Q: What is your data privacy policy?**
A: We take data privacy seriously. See our full policy at [URL]. In short: [one-sentence summary].
```

Replace all `[bracketed placeholders]` with your actual product information. Add as many Q&A pairs as you need. The agent will search this document when answering product questions.

---

## Step 5: Set Up the Agent Identity (IDENTITY.md)

Create `~/.openclaw/workspaces/support/IDENTITY.md`:

```markdown
# Identity

- Name: [Your Company Name] Support
- Role: Customer Support Agent
- Version: 1.0

## Brand Voice

- Professional but approachable
- Always refer to the company as "[Your Company Name]", not "we" or "us"
- Sign off resolved conversations with: "Thank you for contacting [Your Company Name] support."
```

---

## Step 6: Configure the Agent and Channel

Set up a dedicated support agent in your OpenClaw config:

```bash
# Create the support agent pointing to its own workspace
openclaw config set agents.support.model "anthropic:claude-sonnet-4-20250514"
openclaw config set agents.support.workspace "~/.openclaw/workspaces/support"

# Optional: add a fallback model in case the primary is down
openclaw config set agents.support.fallback "openai:gpt-4o"

# Configure the Telegram bot for support
openclaw plugin enable telegram
openclaw config set telegram.bots.support.token "YOUR_SUPPORT_BOT_TOKEN"
openclaw config set telegram.bots.support.agent "support"
```

### Access Policy for a Support Bot

Unlike a personal bot, a support bot needs to be accessible to customers. Disable pairing requirement for the support bot:

```bash
# Allow anyone to message the support bot
openclaw config set telegram.policies.pairingRequired false

# Allow group chats (if you want to use the bot in a support group)
openclaw config set telegram.policies.allowGroups true
```

**Security note:** With pairing disabled, anyone on Telegram can talk to your bot. This is correct for a customer-facing support bot. The SOUL.md boundaries and escalation rules in AGENTS.md keep the bot on-topic. If you want to restrict access to verified customers only, keep pairing enabled and pair customers as they contact you.

---

## Step 7: Enable Memory for Conversation History

Memory lets the support bot remember past interactions with each customer. A returning customer does not have to re-explain their issue.

```bash
openclaw plugin enable memory-lancedb-pro

# Configure memory for support context
openclaw config set memory.autoExtract true
openclaw config set memory.maxRecallResults 5
```

Memory is scoped per user -- each customer has their own memory space. The bot does not mix up conversations between different customers.

---

## Step 8: Test and Verify

Restart the gateway:

```bash
openclaw restart
```

### Verification Checklist

Test each of these scenarios by messaging your support bot on Telegram:

**Basic functionality:**
- [ ] Send "Hi" -- bot greets you and asks how it can help
- [ ] Ask a question from your knowledge base -- bot answers correctly
- [ ] Ask a question NOT in the knowledge base -- bot says it does not know and offers escalation

**Language handling:**
- [ ] Send a message in Spanish (e.g., "Hola, necesito ayuda") -- bot responds in Spanish
- [ ] Send a message in Chinese (e.g., "你好，我需要帮助") -- bot responds in Chinese
- [ ] Send a message in English after a non-English exchange -- bot switches to English

**Escalation rules:**
- [ ] Say "I want to speak to a human" -- bot initiates escalation
- [ ] Say "I want a refund" -- bot escalates to billing (does not try to process)
- [ ] Ask the same question 3+ times -- bot suggests escalation after repeated failure

**Memory:**
- [ ] Tell the bot "My account email is test@example.com"
- [ ] End the conversation and start a new one
- [ ] Ask "What email did I mention?" -- bot recalls it

**Boundaries:**
- [ ] Ask something completely off-topic -- bot redirects to product support
- [ ] Ask for internal company information -- bot declines

If any test fails, check `openclaw logs` and verify the corresponding workspace file.

---

## Complete Configuration

Here is the full `openclaw.json` for this recipe:

```json
{
  "agents": {
    "support": {
      "model": "anthropic:claude-sonnet-4-20250514",
      "workspace": "~/.openclaw/workspaces/support",
      "fallback": "openai:gpt-4o"
    }
  },
  "plugins": {
    "telegram": {
      "bots": {
        "support": {
          "token": "YOUR_SUPPORT_BOT_TOKEN",
          "agent": "support"
        }
      },
      "policies": {
        "pairingRequired": false,
        "allowGroups": true
      }
    },
    "memory-lancedb-pro": {
      "enabled": true,
      "autoExtract": true,
      "maxRecallResults": 5
    }
  },
  "tools": {
    "web_search": {
      "enabled": false
    }
  }
}
```

Note: Web search is disabled for the support bot. Customer support should only use the knowledge base -- not the open web -- to ensure answers are accurate and on-brand.

---

## File Listing

```
10-recipes/customer-support-bot/
  README.md              # This guide
  workspace/
    SOUL.md              # Support agent personality
    IDENTITY.md          # Brand identity
    AGENTS.md            # Escalation rules and policies
    skills/
      product-faq.md     # Knowledge base template
```

---

## Customization Ideas

### Add More Knowledge Base Skills

Split your knowledge base into multiple skill files by topic:

```
skills/
  product-faq.md         # General product questions
  troubleshooting.md     # Common technical issues and fixes
  billing-info.md        # Pricing, plans, billing FAQ (read-only, still escalate actions)
  getting-started.md     # Onboarding guide for new users
```

The agent searches all skills in the directory. More specific files lead to better answers.

### Add Business Hours Awareness

Add a section to AGENTS.md:

```markdown
## Business Hours

- Support team is available Monday-Friday, 9am-6pm EST.
- Outside business hours, tell customers: "Our team is currently offline.
  I'll note your issue and someone will follow up within [X] hours."
- During business hours, escalation connects to a live agent.
  Outside hours, escalation creates a ticket.
```

### Add CSAT Collection

Add to AGENTS.md:

```markdown
## Customer Satisfaction

After resolving an issue, ask:
"On a scale of 1-5, how would you rate your support experience today?"

Log the response. Do not argue with low ratings -- thank them and note the feedback.
```

### Run on Multiple Channels

Add a web chat widget alongside Telegram:

```bash
openclaw plugin enable web
openclaw config set web.agent "support"
openclaw config set web.port 18790
openclaw restart
```

The support bot is now available at `http://localhost:18790` as an embeddable chat widget, in addition to Telegram.

### Use a Cheaper Model for Simple Questions

Configure model routing to use a smaller model for FAQ lookups and the full model for complex issues:

```json
{
  "agents": {
    "support": {
      "model": "anthropic:claude-sonnet-4-20250514",
      "fallback": "anthropic:claude-haiku-4-20250514"
    }
  }
}
```

---

## Troubleshooting

### Bot responds but ignores the knowledge base

1. Verify skill files are in the correct directory: `~/.openclaw/workspaces/support/skills/`
2. Check file names end in `.md`
3. Ensure the SOUL.md says to check the knowledge base first
4. Try asking the exact phrasing from a Q&A pair to confirm the skill is loaded

### Bot does not escalate when it should

1. Review AGENTS.md escalation rules -- are the trigger phrases present?
2. Test with exact trigger phrases: "I want a refund", "Let me talk to a human"
3. Check that the SOUL.md does not override AGENTS.md rules

### Multi-language responses are incorrect

1. Claude handles major languages well but may struggle with less common ones
2. Keep your knowledge base in English -- the agent translates at response time
3. For critical accuracy in a specific language, add a translated version of your FAQ skill

### Memory not persisting between conversations

1. Verify `memory-lancedb-pro` is enabled: `openclaw plugin list`
2. Check `~/.openclaw/memory/` exists
3. Memory extraction happens asynchronously -- allow a few seconds between storing and recalling

---

## Next Steps

- [Module 07: Security](../../07-security/) -- harden the support bot for production use
- [Module 09: Production](../../09-production/) -- monitoring, logging, and scaling for real customer traffic
- [Personal AI on Telegram](../personal-ai-on-telegram/) -- the simpler recipe, if you want to understand the basics first
