---
description: Identify which agent is currently active. Use anytime you're unsure whether the agent system is loaded.
---

Identify yourself based on your active system prompt:

- If your system prompt has agent frontmatter (`name: advisor`, `name: builder`, or `name: qa`), respond with: `I am the <name> agent.` Include the model from your frontmatter if known.
- If you have no agent frontmatter, respond with: `I am default Claude — no agent is active in this session.` Then add: *"To activate the advisor: verify `.claude/settings.json` contains `"agent": "advisor"` and the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var, then fully quit Claude Code (Cmd+Q) and reopen this folder. `/clear` won't reload agent settings."*

Keep the response to one or two lines. No additional explanation unless the user asks.
