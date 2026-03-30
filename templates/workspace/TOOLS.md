# TOOLS.md

<!--
  TOOLS.md provides tool-specific instructions and preferences.
  This file tells the agent HOW to use its tools effectively.
  Only include tools that are enabled for this agent.
-->

## exec (Shell Commands)

<!-- Rules for executing shell commands via the exec tool. -->

- Always use absolute paths.
- Quote arguments that may contain spaces.
- Prefer non-destructive commands. If a command modifies state, explain what it will do first.
- When running long commands, set an appropriate timeout.
- Check exit codes; do not assume success.

## web_search

<!-- Rules for web search tool usage. -->

- Use web_search when the user asks about current events, recent releases, or anything outside your training data.
- Include the source URL when citing search results.
- Prefer official documentation over blog posts.

## file operations (read, write, edit)

<!-- Rules for file system tools. -->

- Read before writing. Never overwrite a file without reading its current contents.
- Use edit (patch) for small changes; use write only for new files or full rewrites.
- Do not create files unless necessary.

## memory (memory_store / memory_recall)

<!-- Rules for long-term memory tools, if enabled. -->

- Use memory_recall at the start of conversations to check for relevant context.
- Use memory_store to save important facts, decisions, and user preferences.
- Set importance >= 0.8 for corrections and key decisions.
- Do not store transient or trivial information.

## MCP Tools

<!--
  Add instructions for any MCP (Model Context Protocol) tools connected
  to this agent. Example:

  ### database-mcp
  - Always use parameterized queries.
  - Never run DDL statements without confirmation.
  - Limit SELECT results to 100 rows unless asked otherwise.
-->

## Custom Skills

<!--
  Reference any skills defined in the workspace/skills/ directory.
  Example:

  ### deploy.md
  - Use when the user says "deploy" or "ship".
  - Always run tests before deploying.
-->
