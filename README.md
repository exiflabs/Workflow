# Workflow

A starter scaffold for an advisor / builder / QA agent workflow in Claude Code.

## First-time setup

Open this folder in Claude Code, then run:

```
/onboard
```

The `/onboard` command walks through prerequisites, graphify install, git setup, model selection, and remote configuration.

If `/onboard` outputs "Restart required", quit Claude Code (Cmd+Q on macOS), reopen the folder, and run `/onboard` again. The agent system loads on session start; it can't be activated mid-session.

## What's in here

- `.claude/agents/` — advisor, builder, qa agent definitions
- `.claude/commands/` — slash commands (`/onboard`, `/whoami`, `/set-model`, `/yolo`)
- `.claude/agent-memory/` — persistent per-agent memory (committed)
- `.workflow/tasks/` — one markdown file per task; the source of truth for all in-progress and completed work
- `CLAUDE.md` — project-wide conventions and working principles
- `src/` — your project code (initially empty)

## Workflow at a glance

1. User describes a task to the advisor
2. Advisor presents a plan, gets approval
3. Advisor writes a task file at `.workflow/tasks/<slug>.md`
4. Advisor spawns a builder, which executes in `src/`
5. Builder reports back; advisor records the implementation in the task file and commits
6. For logic-bearing changes, QA can independently verify

See `.claude/agents/advisor.md` for the canonical per-task workflow.

## Useful commands after onboarding

- `/whoami` — confirm which agent is active in the current session
- `/set-model <agent> <model>` — change a model (`opus` / `sonnet` / `haiku` / `inherit`)
- `/yolo <request>` — skip the approval gate for a single task
