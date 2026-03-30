# USER.md

<!--
  USER.md stores information about the user(s) this agent serves.
  The agent reads this to personalize interactions and avoid repeated questions.

  In single-user mode, fill this in directly.
  In multi-user mode, this serves as a template; per-user context
  may be handled differently depending on your setup.

  IMPORTANT: Do not store sensitive personal data (passwords, SSN, etc.)
  in this file. It is read by the LLM on every turn.
-->

## About the User

<!--
  Basic context about who the user is.
  Example: "Senior developer working on a SaaS platform."
-->

## Preferences

<!--
  Communication and workflow preferences.
  Examples:
  - Prefers concise answers over detailed explanations
  - Wants code examples in Python unless specified otherwise
  - Prefers dark mode terminal commands
  - Uses vim keybindings
-->

## Projects

<!--
  Key projects the user works on. Helps the agent provide relevant context.
  Example:
  - my-api: Node.js REST API, deployed on AWS ECS
  - my-frontend: Next.js app, deployed on Vercel
-->

## Common Requests

<!--
  Patterns the user frequently asks for. Lets the agent skip clarification.
  Example:
  - "deploy" means run the staging deploy script
  - "check logs" means tail the last 50 lines of the app log
  - "PR" means create a pull request with the standard template
-->

## Notes

<!--
  Anything else the agent should know about the user.
  Updated over time as the agent learns more.
-->
