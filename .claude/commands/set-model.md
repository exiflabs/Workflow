---
description: Change the model used by an agent. Usage /set-model <agent> <model>. Agents are advisor, builder, qa. Models are opus, sonnet, haiku, or inherit.
---

The user wants to change the model for one or more agents.

Parse `$ARGUMENTS` for an agent name and model:
- Agent must be one of: `advisor`, `builder`, `qa`
- Model must be one of: `opus`, `sonnet`, `haiku`, `inherit`

If arguments are missing or invalid, ask the user which agent and which model they want to set, presenting them as menu options.

Once you have valid agent + model:

1. Read the agent's definition file at `.claude/agents/<agent>.md`
2. Update the `model:` field in the YAML frontmatter to the new value
3. Confirm the change to the user
4. Remind them: "Restart any active sessions for this agent to pick up the new model."

If the user wants to change multiple agents at once (e.g., "set advisor and builder to opus"), apply the change to each agent file in turn.
