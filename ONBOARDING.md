# Onboarding

This file sets up the project for first use. Run it once by opening Claude Code in the project directory and saying:

> Run the ONBOARDING.md

Claude will execute the steps below, ask you a few questions, and delete this file when finished.

---

## Steps for Claude to Execute

Execute these in order. Pause for user input where indicated.

### 1. Verify Prerequisites

Check that the following are installed. If any are missing, stop and tell the user to install them before continuing.

```bash
git --version
gh --version
uv --version || pipx --version || pip --version
```

### 2. Install Graphify

Install graphify and its Claude Code integration:

```bash
uv tool install graphifyy && graphify install
```

If `uv` isn't available, fall back to `pipx install graphifyy && graphify install` or `pip install graphifyy && graphify install`.

### 3. Ask About Git Setup

Ask the user:

> How would you like to set up version control?
>
> 1. Create a new GitHub repository
> 2. Connect to an existing GitHub repository
> 3. Local git only (no remote)

Wait for the user's response.

### 4. Reset Git History

Remove the starter repo's git history and start fresh:

```bash
rm -rf .git
git init
```

### 5. Install Graphify Git Hook

Now that git is initialized, install the hook:

```bash
graphify hook install
```

This auto-rebuilds the knowledge graph on every commit.

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

Then add `skills: [tdd]` to the frontmatter of `.claude/agents/builder.md` and `.claude/agents/qa.md` (if not already present).

If no, skip this step.

### 7. Configure Agent Models

For each of the three agents, ask the user which model to use. Present options:

> **Advisor model** — handles planning and conversation. Recommended: opus (best reasoning) or sonnet (balanced).
> 1. opus (recommended)
> 2. sonnet
> 3. haiku
> 4. inherit (uses session default)

Wait for response, then ask the same for **Builder** and **QA**:

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

After collecting all three answers, edit the `model:` field in each agent's frontmatter (`.claude/agents/advisor.md`, `builder.md`, `qa.md`) to match the user's selection. Map their answer to the alias: `opus`, `sonnet`, `haiku`, or `inherit`.

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

**Do not delete ONBOARDING.md yet.** Deletion happens in the post-restart verification step below.

### 10. Restart Required — Final Message to User

Output exactly this to the chat:

> **Setup is 90% done. One more step.**
>
> Quit Claude Code now (Cmd+Q on macOS, or fully exit). Reopen this folder, then send the message:
>
> ```
> /verify-onboarding
> ```
>
> This finalizes setup by confirming the advisor agent is active. The `agent` and `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` settings only take effect on a fresh session — `/clear` won't pick them up.
>
> ONBOARDING.md will be removed automatically once verification succeeds.

---

## Notes

- If any step fails, stop and report the error to the user. Don't continue.
- Don't skip the prerequisite check — missing tools cause silent failures later.
- The `rm -rf .git` step is the only destructive operation. It's intentional and safe because this is a starter scaffold.
