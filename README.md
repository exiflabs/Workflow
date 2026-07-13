# Workflow

A scaffold for serious agentic work that is **vendor-equal**: the same **advisor / builder / QA** process on **Claude Code**, **Codex**, **Grok**, and any harness that loads project rules and can spawn subagents.

If a single rules file isn’t enough structure, but heavy frameworks own too much of the process — this is a middle path. Three roles, a folder of Markdown task files, shared skills. That’s it.

## Quickstart

1. Clone into a new empty directory:

```bash
git clone https://github.com/exiflabs/Workflow.git my-project
cd my-project
```

2. Open the project in **your** coding agent (Claude Code, Codex, Grok, …) as a **root** session in this folder.

3. Run onboard:
   - Claude: `/onboard`
   - Skills-based harnesses: invoke the `onboard` skill (`.agents/skills/onboard/`)
   - Or ask: “run onboard”

Onboarding:

- Checks prerequisites (`git`, `gh`, Python installer)
- Installs [graphify](https://github.com/safishamsi/graphify) and multi-platform skill hooks
- Installs the post-commit AST graph hook
- Resets scaffold git history for a clean project start
- Optionally installs [TDD skill](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd)
- Optional model pins (see `workflow/models.md`)
- Remote setup + initial commit

The **root session is always the advisor**. Builder and QA are spawned subagents. No vendor-specific restart ritual is required.

## Why this exists

### Context loss between sessions

**Fix:** every task is `workflow/tasks/<slug>.md` with User Asked / Plan / Implementation per round. Committed with the repo.

### Agent freelancing

**Fix:** approval gate — plan ends with **Should I proceed?** No builder spawn until you affirm (unless `yolo`).

### Plan drift

**Fix:** task file is written **before** the builder runs. Implementation is recorded in the same commit as code.

### No second pair of eyes

**Fix:** separate QA subagent. Verdicts: `PASS` / `PASS WITH NOTES` / `ISSUES FOUND`.

### Blank-slate sessions

**Fix:** graphify AST graph on commit (`graphify-out/`); query with `graphify query "…"`. Semantic re-index of docs is opt-in only.

## Vendor layout (equal)

| Concern | Location |
|---------|----------|
| Project rules | `AGENTS.md` (canonical). `CLAUDE.md` → symlink to the same file |
| Role contracts | `workflow/roles/{advisor,builder,qa}.md` |
| Memory | `workflow/agent-memory/{advisor,builder,qa}/MEMORY.md` |
| Models guide | `workflow/models.md` |
| Shared skills | `.agents/skills/` |
| Claude adapters | `.claude/agents/`, `.claude/commands/` |
| Codex adapters | `.codex/agents/{builder,qa}.toml` (advisor = root + `AGENTS.md`) |
| Grok adapters | `.grok/agents/` |

Process and role behavior live in **one** place. Platform folders only wire tools/models.

## How roles divide work

| Role | Writes | Reads | Talks to |
|------|--------|-------|----------|
| **Advisor** (root) | `workflow/tasks/`, config, memory | Everything | User |
| **Builder** | `src/` | Task file, code, graphify | Advisor (report) |
| **QA** | `src/tests/` only | Task file, code, graphify | Advisor (report) |

## Per-task flow

1. User describes work to the advisor  
2. Plan in chat → **Should I proceed?**  
3. On yes: write `workflow/tasks/<slug>.md` for this round  
4. Spawn builder with that path  
5. Record Implementation; commit task file + `src/` together  
6. Offer QA for logic-bearing changes  
7. On complete: mark done; ask about Graphify semantic update (never auto)

Canonical detail: `workflow/roles/advisor.md`.

## Skills / commands

| Action | Shared skill | Claude slash |
|--------|--------------|--------------|
| First-time setup | `onboard` | `/onboard` |
| Active role | `whoami` | `/whoami` |
| Pin model | `set-model` | `/set-model <agent> <id>` |
| Skip gate once | `yolo` | `/yolo` |
| Knowledge graph | graphify skill (installed) | `/graphify` |

Recommended models (Opus 4.8, Sonnet 5, 5.6 family, Fable 5, Grok 4.5): `workflow/models.md`. Default is **inherit**.

## Graphify

```bash
uv tool install graphifyy   # or pipx / pip
graphify install --platform claude
graphify install --platform codex
graphify install --platform agents
graphify hook install
```

- **Free:** post-commit AST rebuild of code into `graphify-out/`
- **Opt-in (API cost):** semantic update for Markdown/docs when you agree

## Obsidian

Open the project folder as a vault. Task files and `related:` frontmatter give backlinks and graph view. `.obsidian/` is gitignored.

## Acknowledgements

- Optional TDD skill: [Matt Pocock / skills](https://github.com/mattpocock/skills)
- Knowledge graph: [graphify](https://github.com/safishamsi/graphify)
