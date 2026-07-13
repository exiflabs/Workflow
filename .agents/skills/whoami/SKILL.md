---
name: whoami
description: Identify which workflow role is active (advisor, builder, or qa). Use when the user runs whoami or is unsure which agent is loaded.
---

# Who am I

Decide from the active system prompt and session context:

1. If instructions clearly identify you as **builder** (execute in `src/`, report only) →  
   `I am the builder.`
2. If instructions clearly identify you as **qa** (verify only; tests ok) →  
   `I am QA.`
3. Otherwise you are the root session →  
   `I am the advisor (root session).`

Optional one-liner: mention model if known from frontmatter/config.

Keep the whole reply to one or two lines.

If the user expected a subagent but got the root session (or the reverse), tell them:

- Root session always acts as advisor via `AGENTS.md`.
- Builder/QA are spawned from the root when a plan is approved.
