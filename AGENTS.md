# Workflow

This project uses an **advisor / builder / QA** workflow that is **vendor-equal**: the same process runs on Claude Code, Codex, Grok, and any harness that loads project rules + subagents.

## Roles

| Role | Who it is | Writes | Talks to user |
|------|-----------|--------|----------------|
| **Advisor** | Always the **root session** | `workflow/tasks/`, root config, agent memory | Yes |
| **Builder** | Spawned subagent | `src/` (code + tests) only | No — reports to advisor |
| **QA** | Spawned subagent | `src/tests/` only | No — reports to advisor |

- Advisor never executes code or modifies `src/`.
- Builder never writes task vault files or converses with the user.
- QA never modifies production code.

Role contracts (must follow exactly):

- Advisor → `workflow/roles/advisor.md`
- Builder → `workflow/roles/builder.md`
- QA → `workflow/roles/qa.md`

Platform adapters under `.claude/agents/`, `.codex/agents/`, and `.grok/agents/` only wire tools/models; they do not redefine the process.

## Root session = advisor

On every vendor, the **root task is the advisor**. Builder and QA are always spawned subagents.

- Do **not** require vendor-specific frontmatter (e.g. `name: advisor`) to act as advisor.
- Do **not** require a restart ritual to “activate” the advisor.
- If you are clearly running as builder or QA (subagent), do not take over advisor duties — return a report only.

## Directory structure

```
src/                        # Project code
src/tests/                  # Tests
workflow/tasks/             # One MD file per task (slug-named)
workflow/roles/             # Shared role contracts (source of truth)
workflow/agent-memory/      # Shared persistent memory per role
workflow/models.md          # Recommended model IDs per vendor
graphify-out/               # Knowledge graph (regenerated; usually gitignored)
.agents/skills/             # Shared skills (onboard, whoami, yolo, set-model, …)
.claude/agents/             # Claude adapters
.codex/agents/              # Codex adapters (builder + qa)
.grok/agents/               # Grok adapters
```

`CLAUDE.md` is a symlink to this file so Claude-compatible loaders get the same rules.

## Task files

One file per task: `workflow/tasks/<slug>.md`. Rounds are sequential sections. Commits identify rounds (`git log --grep=<slug>`).

**Frontmatter (required):**

- `title` — human-readable description
- `slug` — matches the filename
- `tags` — topical keywords (graphify / search)
- `related` — list of related task slugs
- `status` — `in-progress` or `complete`

**Each round subsections:**

- `### User Asked` — verbatim original request
- `### Discussion` — decision points (omit if none)
- `### Plan` — approved plan with steps and verification
- `### Implementation` — what was built (advisor fills after builder)

Round headers use minute-precision local time: `## Round N — 2026-05-01 14:23`.

**Commit convention:** one commit per round: `<slug>: brief description of this round`

## Per-task sequence (canonical)

1. Plan in chat ending with **Should I proceed?** (unless yolo bypass).
2. On approval, write/update the task file **first** (frontmatter + round + User Asked + Plan).
3. Spawn builder with the task file path; builder follows `workflow/roles/builder.md`.
4. Write `### Implementation` from the builder report.
5. Commit task file + `src/` (and related paths) **together**.
6. Summarize; offer QA for logic-bearing work.
7. On task complete: mark `status: complete`, summary, memory update; ask about Graphify semantic update (never automatic).

Full advisor detail: `workflow/roles/advisor.md`.

## Working principles

### 1. Think before coding

- State assumptions. If uncertain, ask.
- Multiple interpretations → present them; don’t pick silently.
- Prefer the simpler approach when it works.

### 2. Simplicity first

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked
- No abstractions for single-use code
- No unrequested configurability
- No error handling for impossible scenarios
- If 200 lines could be 50, rewrite it

### 3. Surgical changes

Every changed line traces to the request.

- Don’t improve adjacent code, comments, or formatting
- Don’t refactor what isn’t broken; match existing style
- Mention unrelated dead code — don’t delete it unless asked
- Remove imports/variables your change made unused

### 4. Goal-driven execution

Define testable success criteria; loop until verified.

- Vague asks → concrete verify steps
- Strong criteria enable independent loops

## Testing

- All tests live in `src/tests/`
- Test through public interfaces, not implementation details
- Tests are part of verification, not an afterthought
- Match existing project test conventions

**Write tests when the change involves logic:** compute/transform/validate, state, branching, APIs, sequencing/async, event handlers, error paths.

**Skip tests only for no-logic changes:** pure styling, static markup, copy, images, static config, docs.

**Animation timing is logic**, not pure visual — if JS controls *when* something happens, write tests.

## Coding standards

- Max 200 lines per file (excluding imports and docstring)
- Max 30 lines per function
- Max 3 levels of nesting
- One responsibility per file
- No ABC/Protocol unless 2+ implementations exist now
- No factory patterns — construct directly
- No registry / auto-discovery / decorator magic
- Every component explicitly imported and instantiated

## Architecture

- No shared mutable state between sibling modules
- Sibling modules expose functions; they don’t reach into each other’s internals
- Modules may inherit parent/global config and utilities
- Secrets in `.env` (gitignored); commit `.env.example` for required vars

## Context & knowledge graph (graphify)

Graphify provides background context at `graphify-out/`. When info conflicts: **active task file > completed task summaries > graphify**.

- Prefer `graphify query "<terms>"` for past work and codebase structure
- Read `graphify-out/GRAPH_REPORT.md` for architecture questions
- Git post-commit hook rebuilds the **code graph** (AST-only, free)
- For doc/MD re-index including `workflow/tasks/`, run semantic update only when the user opts in (`/graphify --update` or equivalent). After push/merge, advisor asks once — never automatic
- Task history: read `workflow/tasks/` directly when needed

Install skill + hook during onboard for every platform you use:

```bash
# package
uv tool install graphifyy || pipx install graphifyy || pip install graphifyy

# skill into harness config dirs (repeat platforms as needed)
graphify install --platform claude
graphify install --platform codex
graphify install --platform agents

graphify hook install
```

## Memory

Shared, committed memory (not vendor-locked):

- Advisor: `workflow/agent-memory/advisor/MEMORY.md`
- Builder: `workflow/agent-memory/builder/MEMORY.md`
- QA: `workflow/agent-memory/qa/MEMORY.md`

Keep each file under 25KB. Never create `MEMORY.md` at the project root.

## Models

Recommended flagship IDs: `workflow/models.md`. Default for all roles is **inherit** (session default) unless onboard or `/set-model` sets an override on the platform adapter.

## Skills

Shared skills live in `.agents/skills/`. Harnesses that also load `.claude/commands/` get thin wrappers for the same actions.

- `onboard` — first-time project setup
- `whoami` — which role is active
- `yolo` — skip approval gate for one task
- `set-model` — set role model override on adapters

## Project context

<!-- Add tech stack, domain rules, and environment notes below -->
