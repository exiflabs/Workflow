---
description: First-time project setup. Run once after cloning the scaffold.
---

# Onboard

Run setup for this project. Two phases — identity check first, then install steps.

## Phase 1 — Identity check

Inspect your own system prompt frontmatter.

If `name: advisor` is in your frontmatter → continue to Phase 2.

If your frontmatter does NOT contain `name: advisor` → output the block below as your entire response and stop. Do not add anything before, after, or alongside it.

> **Restart required.**
>
> The advisor agent isn't active yet. It needs to load before onboarding can run safely.
>
> Quit Claude Code now (Cmd+Q on macOS, or fully exit). Reopen this folder, then run:
>
> ```
> /onboard
> ```
>
> The agent system only initializes on a fresh session — `/clear` won't reload it.

## Phase 2 — Setup (advisor only)

Execute these in order. After each file-creating step, verify the result before continuing. If any verification fails, stop and report.

### 1. Verify prerequisites

```bash
git --version
gh --version
uv --version || pipx --version || pip --version
```

If any are missing, stop and tell the user what to install.

### 2. Install graphify

```bash
uv tool install graphifyy && graphify install
```

Fall back to `pipx install graphifyy && graphify install` or `pip install graphifyy && graphify install` if `uv` isn't available.

**Verify:** `command -v graphify` returns a path.

### 3. Ask about git setup

> How would you like to set up version control?
>
> 1. Create a new GitHub repository
> 2. Connect to an existing GitHub repository
> 3. Local git only (no remote)

Wait for the user's response.

### 4. Reset git history

```bash
rm -rf .git
git init
```

**Verify:** `.git/` directory exists.

### 5. Install graphify git hook

```bash
graphify hook install
```

**Verify:** `.git/hooks/post-commit` exists.

### 6. Ask about optional TDD skill

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

**Verify:** `.claude/skills/tdd/SKILL.md` exists.

If no, skip.

### 7. Configure agent models

For each agent, ask which model to use:

> **Advisor model** — planning and conversation. Recommended: opus (best reasoning) or sonnet (balanced).
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

Edit the `model:` field in `.claude/agents/advisor.md`, `builder.md`, `qa.md` to match.

### 8. Configure remote (based on Step 3)

**New GitHub repo:** ask for repo name, then:

```bash
gh repo create <name> --private --source=. --remote=origin
```

**Existing GitHub repo:** ask for URL. Then check whether the remote is empty before merging anything:

```bash
git ls-remote <url> 2>/dev/null
```

- If `git ls-remote` returns no refs (empty repo): safe to push directly.
  ```bash
  git remote add origin <url>
  ```
  No pull needed; Step 9 will push as the first commit.

- If `git ls-remote` returns refs but the only files are README/LICENSE-style (auto-init from GitHub): pull is safe.
  ```bash
  git remote add origin <url>
  git pull origin main --allow-unrelated-histories
  ```

- If the remote has substantive history (more than a single auto-init commit, or any source files), STOP and warn the user:

  > **The remote `<url>` already has unrelated history.** Pulling it will merge two scaffolds and produce a polluted history (duplicate "initial commit", orphaned files, conflicting agent definitions).
  >
  > Options:
  > 1. Delete the existing repo and create a new one (recommend).
  > 2. Push this scaffold to a different repo URL.
  > 3. Force-overwrite the remote with this scaffold (destroys all existing remote content): only do this if you're certain the remote content can be discarded.
  >
  > Tell me which option you want, or paste a different URL.

  Wait for the user's choice before proceeding. Do not auto-merge.

**Local only:** skip.

### 9. Initial commit

```bash
git add .
git commit -m "initial commit: workflow starter scaffold"
```

If a remote was configured:

```bash
git branch -M main
git push -u origin main
```

### 10. Remove the legacy ONBOARDING.md if present

```bash
[ -f ONBOARDING.md ] && rm ONBOARDING.md && git add ONBOARDING.md && git commit -m "remove legacy onboarding file" && git push 2>/dev/null
```

### 11. Final reference card

Output exactly:

> **Setup complete. Advisor is active and ready.**
>
> **Useful commands:**
> - `/whoami` — confirms which agent is active. Run anytime you're unsure.
> - `/set-model <agent> <model>` — change the model used by advisor, builder, or qa. Models: `opus`, `sonnet`, `haiku`, `inherit`.
>
> **If `/whoami` ever returns "default Claude":** the agent system didn't load. Verify `.claude/settings.json` still has `"agent": "advisor"` and the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` env var, then fully quit Claude Code (Cmd+Q) and reopen. `/clear` alone won't reload agent settings.
>
> Edit `CLAUDE.md` to add project-specific context (tech stack, conventions, domain rules), or start your first task.
