# Workflow

This project uses an advisor/builder agent workflow.

- **Advisor** — converses with the user, plans tasks, writes prompt files. Never executes code or modifies `src/`
- **Builder** — executes tasks in `src/`, writes response files. Never converses with the user directly

## Directory Structure

```
src/                     # All project code lives here
src/tests/               # All tests live here
workflow/tasks/         # One MD file per task, named by slug
graphify-out/            # Knowledge graph (auto-updated on commit)
.claude/agents/          # Agent definitions
.claude/agent-memory/    # Persistent agent memory (committed to version control)
.claude/skills/          # Optional skills (e.g., TDD if installed)
```

## Task Files

One file per task, named by kebab-case slug (e.g., `fix-login-bug.md`). Rounds of back-and-forth are sequential sections within the file. Commit IDs serve as identifiers — no separate version numbering.

**Frontmatter (required):**
- `title` — human-readable description
- `slug` — matches the filename
- `tags` — topical keywords (used by graphify queries)
- `related` — list of related task slugs
- `status` — `in-progress` or `complete`

**Each round must contain these subsections:**
- `### User Asked` — verbatim original request
- `### Discussion` — bullet summary of decision points (omit if no clarification was needed)
- `### Plan` — approved plan with steps and verification
- `### Implementation` — what was built, including verification (filled by advisor after builder finishes)

Round headers include a minute-precision timestamp: `## Round N — 2026-05-01 14:23`. Commits link back to rounds via the slug convention — `git log --grep=<slug>` enumerates all commits for a task, and the round-header timestamp matches the commit time.

## Commit Convention

Commit per round. Format: `<slug>: brief description of this round`

---

# Working Principles

## 1. Think Before Coding
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. If something is unclear, stop and ask.

## 2. Simplicity First
Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked, no abstractions for single-use code
- No flexibility or configurability that wasn't requested
- No error handling for impossible scenarios
- If 200 lines could be 50, rewrite it. Ask: *"Would a senior engineer say this is overcomplicated?"*

## 3. Surgical Changes
Every changed line should trace directly to the user's request.
- Don't improve adjacent code, comments, or formatting
- Don't refactor things that aren't broken; match existing style
- Mention unrelated dead code — don't delete it
- Remove imports/variables your changes made unused
- Never remove pre-existing dead code without being asked

## 4. Goal-Driven Execution
Define testable success criteria, loop until verified.
- Transform vague asks: "Add validation" → "Write tests for invalid inputs, then make them pass"
- State steps with explicit verification for each
- Strong criteria let you loop independently; weak ones ("make it work") cause constant clarification

---

# Testing

- All tests live in `src/tests/`
- Test through public interfaces, not implementation details — tests should survive refactors
- Tests are part of verification, not an afterthought — they run before a task is declared done
- Match existing test conventions in the project (framework, style)

## When to write tests

**Write tests when the change involves logic, including:**
- Functions that compute, transform, or validate values
- State transitions, conditionals, branching
- API/endpoint behavior
- Sequencing or timing logic (e.g., delays, transitions, async flows)
- Event handlers that perform actions
- Error handling paths
- Anything with `if`/`else`, loops, or async control flow

**Skip tests only when the change has no logic:**
- Pure styling (CSS color, font, spacing, layout — no JS interaction)
- Static HTML markup with no scripted behavior
- Copy/text/string changes
- Image swaps
- Static configuration values
- Documentation updates

**Animation timing is logic, not visual.** A 3-second delay or sequenced fade involves testable behavior. Visual = appearance only. If JS controls *when* something happens, write tests.

---

# Coding Standards

- Max 200 lines per file (excluding imports and docstring)
- Max 30 lines per function
- Max 3 levels of nesting
- One responsibility per file — if the filename needs "and" or "manager", split it
- No ABC/Protocol unless 2+ implementations exist right now
- No factory patterns — construct directly
- No registry, auto-discovery, or decorator-based magic
- Every component explicitly imported and instantiated

---

# Architecture

- No shared mutable state between sibling modules
- Sibling modules are deaf to each other — they expose functions, never reach into each other's internals
- Modules may inherit from parent/global modules (config, env, shared utilities)
- Values live in `.env` (gitignored) and root config files — child modules inherit, never hardcode
- Commit a `.env.example` template to document required variables

---

# Context & Knowledge Graph

Graphify provides background context at `graphify-out/`. Priority when info conflicts: active task file > completed task summaries > graphify.

- Read `graphify-out/GRAPH_REPORT.md` before answering architecture questions
- Git post-commit hook auto-rebuilds the **code graph** (AST-only, free)
- For doc/MD indexing including `workflow/tasks/`, `/graphify --update` is run only when the user explicitly opts in (uses API calls). After a push to origin or merge, the advisor will ask "Would you like to update Graphify?" — yes runs it, no/ignore skips it. Never automatic.
- For task history, read `workflow/tasks/` files directly — graphify isn't needed for that

---

# Project Context

<!-- Add your project-specific context below: tech stack, architecture, environment setup, constraints, domain rules -->
