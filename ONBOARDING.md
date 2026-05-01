# Onboarding

This file sets up the project for first use. Run it once by opening Claude Code in the project directory and saying:

> Run the ONBOARDING.md

Onboarding has two phases. Phase 1 is a one-step identity check. If the advisor agent isn't active yet, it tells you to restart Claude Code, then halts. Phase 2 (all real install work) only runs once the advisor is active, so you can trust every step.

---

## Phase 1 — Identity Check (any agent)

**Before doing anything else, check your identity by inspecting your own system prompt frontmatter.**

There are exactly two valid outcomes:

### Outcome A — You are the advisor

Your frontmatter contains `name: advisor`. Skip directly to Phase 2.

### Outcome B — You are NOT the advisor (e.g., default Claude, or any other agent)

The user asked you to run this file. The correct way to run this file when you are not the advisor is to output the restart message below and end your response. **Stopping here IS executing the user's instruction correctly.** Do not interpret "run the file" as license to skip Phase 1.

Output **exactly** the following block and then end your response. Nothing else — no preamble, no analysis, no offer of alternatives, no questions:

> **Restart required.**
>
> The advisor agent isn't active yet. It needs to load before onboarding can run safely.
>
> Quit Claude Code now (Cmd+Q on macOS, or fully exit). Reopen this folder, then send the message:
>
> ```
> Run the ONBOARDING.md
> ```
>
> The agent system only initializes on a fresh session — `/clear` won't reload it.

### Hard rules for Outcome B

- **Do not** offer to proceed as default Claude.
- **Do not** ask the user whether they'd like to skip Phase 1.
- **Do not** present the situation as a choice. There is no choice — the design requires the advisor for Phase 2.
- **Do not** add explanations beyond the block above. The user already knows what's happening; the block is sufficient.
- **Do not** continue to Phase 2 under any circumstance, including if the user pushes back. If the user insists on bypassing, repeat the same block and stop again.

Phase 2 is invisible to you in Outcome B. Treat the rest of this file as if it does not exist for this response.

---

## Phase 2 — Setup (advisor only)

Execute these in order. After each step that creates or modifies a file, verify the result before continuing. If any verification fails, STOP and report the error to the user.

### 1. Verify Prerequisites

Check that the following are installed:

```bash
git --version
gh --version
uv --version || pipx --version || pip --version
```

If any are missing, stop and tell the user to install them before continuing.

### 2. Install Graphify

```bash
uv tool install graphifyy && graphify install
```

If `uv` isn't available, fall back to `pipx install graphifyy && graphify install` or `pip install graphifyy && graphify install`.

**Verify:** `command -v graphify` returns a path. If not, stop.

### 3. Ask About Git Setup

Ask the user:

> How would you like to set up version control?
>
> 1. Create a new GitHub repository
> 2. Connect to an existing GitHub repository
> 3. Local git only (no remote)

Wait for the user's response.

### 4. Reset Git History

Remove the starter repo's git history:

```bash
rm -rf .git
git init
```

**Verify:** `.git/` directory exists. If not, stop.

### 5. Install Graphify Git Hook

```bash
graphify hook install
```

**Verify:** `.git/hooks/post-commit` exists. If not, stop.

### 6. Ask About Optional TDD Skill

Ask the user:

> Install the TDD (test-driven development) skill? It teaches the builder to use a red-green-refactor loop with vertical slicing. Recommended for projects where behavior changes get tested.
>
> Source: [skills/engineering/tdd by Matt Pocock](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd)
>
> (y/n)

If yes:
```bash
mkdir -p .claude/skills/tdd
curl -fsSL https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/tdd/SKILL.md -o .claude/skills/tdd/SKILL.md
```

Then add `skills: [tdd]` to the frontmatter of `.claude/agents/builder.md` and `.claude/agents/qa.md` if not already present.

**Verify:** `.claude/skills/tdd/SKILL.md` exists. If not, stop.

If user said no, skip this step.

### 7. Configure Agent Models

For each of the three agents, ask which model to use:

> **Advisor model** — handles planning and conversation. Recommended: opus (best reasoning) or sonnet (balanced).
> 1. opus (recommended)
> 2. sonnet
> 3. haiku
> 4. inherit (uses session default)

> **Builder model** — executes code. Recommended: sonnet (balanced) or opus (for complex work).
> 1. opus
> 2. sonnet (recommended)
> 3. haiku
> 4. inherit

> **QA model** — verifies builder work. Recommended: sonnet (catches issues) or opus (deep analysis).
> 1. opus
> 2. sonnet (recommended)
> 3. haiku
> 4. inherit

After collecting all three answers, edit the `model:` field in each agent file (`.claude/agents/advisor.md`, `builder.md`, `qa.md`).

### 8. Configure Remote (Based on Step 3 Choice)

**If new GitHub repo:**

Ask for the repository name. Then run:
```bash
gh repo create <name> --private --source=. --remote=origin
```

**If existing GitHub repo:**

Ask for the repository URL. Then run:
```bash
git remote add origin <url>
git pull origin main --allow-unrelated-histories
```

**If local only:**

Skip remote setup.

### 9. Initial Commit

```bash
git add .
git commit -m "initial commit: workflow starter scaffold"
```

If a remote was configured, push:

```bash
git branch -M main
git push -u origin main
```

### 10. Remove This File

```bash
rm ONBOARDING.md
git add ONBOARDING.md
git commit -m "remove onboarding file"
git push 2>/dev/null || true
```

### 11. Final Message — Reference Card

Output exactly this to the user:

> **Setup complete. Advisor is active and ready.**
>
> **Useful commands:**
> - `/whoami` — confirms which agent is active. Run anytime you're unsure.
> - `/set-model <agent> <model>` — change the model used by advisor, builder, or qa. Models: `opus`, `sonnet`, `haiku`, `inherit`.
>
> **If `/whoami` ever returns "default Claude":** the agent system didn't load. Verify `.claude/settings.json` still has `"agent": "advisor"` and the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var, then fully quit Claude Code (Cmd+Q) and reopen. `/clear` alone won't reload agent settings.
>
> Edit `CLAUDE.md` to add project-specific context (tech stack, conventions, domain rules), or start your first task.

---

## Notes

- Phase 1's identity check is non-negotiable. Default Claude must NOT proceed past Phase 1.
- If any verification step in Phase 2 fails, stop immediately and report — don't try to continue.
- The `rm -rf .git` step is the only destructive operation. It's intentional and safe because this is a starter scaffold.
