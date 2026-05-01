---
description: Finalize onboarding after restart. Confirms the advisor agent is active, prints a reference card, and removes ONBOARDING.md.
---

This command finalizes setup after the user restarted Claude Code following ONBOARDING.md.

## Steps

### 1. Verify the advisor is active

Confirm both:
- `.claude/settings.json` contains `"agent": "advisor"`
- Your own system prompt has frontmatter `name: advisor`

If you are NOT the advisor (no `name: advisor` frontmatter), STOP. Output the recovery message below and do nothing else:

> **Verification failed — the agent system didn't load.**
>
> Check that `.claude/settings.json` contains:
> - `"agent": "advisor"`
> - `"env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }`
>
> Then fully quit Claude Code (Cmd+Q) and reopen the folder. `/clear` alone won't reload agent settings.
>
> Once reopened, run `/verify-onboarding` again.

### 2. On success, print the reference card

Output exactly this:

> **Setup verified. Advisor is active.**
>
> **Useful commands:**
> - `/whoami` — confirms which agent is active. Run anytime you're unsure.
> - `/set-model <agent> <model>` — change the model used by advisor, builder, or qa. Models: `opus`, `sonnet`, `haiku`, `inherit`.
>
> **If `/whoami` ever returns "default Claude":** the agent system didn't load. Verify `.claude/settings.json` still has `"agent": "advisor"` and the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var, then fully quit Claude Code (Cmd+Q) and reopen. `/clear` alone won't reload agent settings.
>
> Edit `CLAUDE.md` to add project-specific context (tech stack, conventions, domain rules), or start your first task.

### 3. Remove ONBOARDING.md and commit

```bash
rm ONBOARDING.md
git add ONBOARDING.md
git commit -m "remove onboarding file"
git push 2>/dev/null || true
```
