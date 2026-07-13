---
name: onboard
description: First-time project setup after cloning this workflow scaffold. Use when the user invokes onboard or asks to initialize the project.
---

# Onboard

Run setup from the **root session** (advisor). This skill is vendor-equal: Claude, Codex, Grok, or any harness loading `.agents/skills/`.

## 1. Role check

- If you are clearly a **builder** or **QA** subagent: stop and tell the user to run onboard from the root task.
- Otherwise continue as advisor. Do **not** require `name: advisor` frontmatter or a restart.

Verify scaffold files:

```bash
test -f AGENTS.md
test -L CLAUDE.md || test -f CLAUDE.md
test -f workflow/roles/advisor.md
test -f workflow/roles/builder.md
test -f workflow/roles/qa.md
```

If any fail, stop and report what is missing.

## 2. Prerequisites

```bash
git --version
gh --version
uv --version || pipx --version || pip --version
```

If git or a Python installer is missing, stop and tell the user what to install. `gh` is optional until a GitHub remote is chosen.

## 3. Install graphify

If `graphify` is missing:

```bash
uv tool install graphifyy || pipx install graphifyy || pip install graphifyy
```

Install the skill into harnesses the user cares about (ask which, or install all common ones):

```bash
graphify install --platform claude
graphify install --platform codex
graphify install --platform agents
```

`agents` covers AGENTS.md-oriented tools. Add other platforms from `graphify install --help` if needed.

**Verify:** `command -v graphify`

## 4. Version control

Ask:

> How would you like to set up version control?
>
> 1. Create a new GitHub repository
> 2. Connect to an existing GitHub repository
> 3. Local git only (no remote)

Then confirm:

> Onboarding will remove this scaffold’s `.git` directory and start a clean history. Continue? (y/n)

Stop if not confirmed.

## 5. Reset git and install hook

```bash
rm -rf .git
git init
graphify hook install
```

**Verify:** `.git/` exists and `.git/hooks/post-commit` exists (or hook status succeeds).

## 6. Optional TDD skill

Ask whether to install Matt Pocock’s TDD skill into `.agents/skills/tdd/`.

If yes:

```bash
mkdir -p .agents/skills/tdd
curl -fsSL https://raw.githubusercontent.com/mattpocock/skills/main/skills/engineering/tdd/SKILL.md -o .agents/skills/tdd/SKILL.md
```

**Verify:** `.agents/skills/tdd/SKILL.md` exists.

## 7. Models

Point at `workflow/models.md`. Ask whether to pin role models or leave **inherit** (recommended default).

If pinning, use free-form IDs the user’s harness accepts (examples: Opus 4.8, Sonnet 5, Fable 5, newest 5.6, Grok 4.5). Write overrides into the adapters for the harnesses they use:

- Claude: `.claude/agents/*.md` → `model:`
- Codex: `.codex/agents/{builder,qa}.toml` → `model = "..."` (advisor = root session)
- Grok: `.grok/agents/*.md` → `model:`

## 8. Configure remote

**New GitHub repo:** ask name, then `gh repo create <name> --private --source=. --remote=origin` (after initial commit prep as needed).

**Existing remote:** `git ls-remote <url>`. Empty → add origin. Auto-init only → pull with `--allow-unrelated-histories` carefully. Substantive history → stop and warn; do not auto-merge scaffolds.

**Local only:** skip.

## 9. Initial commit

```bash
git add .
git commit -m "initial commit: workflow starter scaffold"
```

If remote configured:

```bash
git branch -M main
git push -u origin main
```

## 10. Done

Output:

> **Setup complete. Root session is the advisor.**
>
> **Useful skills/commands:**
> - `whoami` / `/whoami` — which role is active
> - `set-model` / `/set-model <agent> <model>` — pin a role model (or `inherit`)
> - `yolo` / `/yolo` — skip approval for one task
>
> Edit `AGENTS.md` (and the `CLAUDE.md` symlink follows) for project-specific context, or start your first task.
>
> Graphify: free AST updates on commit via the hook; semantic `/graphify --update` only when you opt in.
