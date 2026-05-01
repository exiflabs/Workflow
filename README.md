# Workflow

A scaffold for serious work with Claude Code: an **advisor / builder / QA** agent team backed by a persistent task vault, an AST knowledge graph, and a deterministic per-task workflow that survives compaction, session restarts, and switching between machines.

If a single `CLAUDE.md` isn't enough structure for your project, but BMAD / Spec-Kit / GSD-style frameworks own too much of the process — this is a middle path. Three small agents, a folder of Markdown task files, a few slash commands. That's it.

## Quickstart (60 seconds)

1. Clone the scaffold into a new, empty directory:

```bash
git clone https://github.com/exiflabs/Workflow.git my-project
cd my-project
```

2. Open Claude Code in that directory (a fresh session — not a `/clear` of an existing one):

```bash
claude
```

3. In the Claude Code session, run:

```
/onboard
```

The onboarding command:

- Verifies prerequisites (`git`, `gh`, `uv` / `pipx` / `pip`)
- Installs [`graphify`](https://github.com/safishamsi/graphify) and its post-commit hook
- Resets the scaffold's git history so your project starts clean
- Optionally installs the [TDD skill](https://github.com/mattpocock/skills/tree/main/skills/engineering/tdd) by Matt Pocock
- Asks which model each agent should use (advisor / builder / QA)
- Configures a remote (new GitHub repo, existing repo, or local-only)
- Makes the initial commit and pushes if a remote is set

If the first `/onboard` outputs "Restart required", quit Claude Code (Cmd+Q on macOS, or fully exit), reopen the folder, and run `/onboard` again. The agent system bootstraps on session start; `/clear` won't reload it.

## Why This Exists

Most coding-agent setups are either **too loose** (a `CLAUDE.md` and good intentions, easy to drift) or **too rigid** (fully scripted frameworks that fight you when reality diverges from their happy path). This scaffold targets the failure modes that show up in the middle.

### #1: Context Loss Between Sessions

> "Without ceremony, you don't have a process — you have a hope."
>
> Anonymous, every project that's ever lost its history

**The Problem.** You finish a task, close Claude Code, come back tomorrow. The model has no memory of what you discussed, what you decided not to do, or what didn't work. You re-explain.

**The Fix.** Every task has a Markdown file at `.workflow/tasks/<slug>.md`. Each round of back-and-forth is a section within that file: `User Asked` (verbatim), `Discussion`, `Plan`, `Implementation`. Closing and reopening a session is safe — the advisor reads any in-progress task file at startup and resumes from there. Task files are committed to version control, so they travel with the repo.

### #2: The Agent Did Whatever It Wanted

> "Always take small, deliberate steps. Never take on a task that's too big."
>
> David Thomas & Andrew Hunt, *The Pragmatic Programmer*

**The Problem.** You ask for a thing. The agent gives you the thing plus three improvements you didn't ask for, plus a refactor you'll spend an hour reading. Or it skips planning entirely and starts writing code.

**The Fix.** A strict approval gate between planning and execution. The advisor presents a plan in chat ending with "Should I proceed?". No `Write`, `Edit`, `Bash`, or `Agent` calls fire until you say yes. A `/yolo` bypass exists for one-off small changes when the gate is friction you don't want.

### #3: Plans Drift Mid-Build

**The Problem.** The agent ships code that doesn't match the plan. Later, you can't tell what was decided vs. what was added on the fly.

**The Fix.** The task file is written to disk **before** the builder is spawned. The builder reads the plan, executes within it, and reports back as a structured Agent tool response. The advisor records the implementation as the `### Implementation` subsection of the same task file, in the same commit as the source changes. Plan and code never separate.

### #4: No Independent Verification

**The Problem.** The agent writes the code AND writes the tests AND tells you it works. There's no second pair of eyes between the implementer and the verifier.

**The Fix.** A separate QA agent. After any logic-bearing round, the advisor offers QA verification. QA reads the task file, runs the tests, inspects the code the builder produced, and returns a `PASS` / `PASS WITH NOTES` / `ISSUES FOUND` verdict with severity-tagged findings. QA can edit test files but never production code — keeping the role honest.

### #5: The Agent Forgets The Project It's Working On

> "A program is shaped by being read as much as by being written."
>
> Donald Knuth, *Literate Programming*

**The Problem.** Each Claude Code session is a blank slate. The agent doesn't know which functions exist, which modules call which, or what the project's domain language is.

**The Fix.** Every commit triggers [graphify](https://github.com/safishamsi/graphify), which extracts an AST graph of the codebase into `graphify-out/`. Agents can run `graphify query "<terms>"` to pull relevant project context before starting work. This is free (AST-only, no LLM). At task-completion checkpoints, the advisor will *ask* if you want to also run `/graphify --update` for an LLM-powered re-index of your Markdown — opt-in, never automatic.

## How The Agents Divide Work

| Role | Writes to | Reads from | Talks to |
|------|-----------|------------|----------|
| **Advisor** | `.workflow/tasks/`, root config files, agent memory | Everything | The user |
| **Builder** | `src/` (code + tests) | Task file, source code, graphify | Advisor (via Agent tool reports) |
| **QA** | `src/tests/` only | Task file, source code, graphify | Advisor (via Agent tool reports) |

The advisor never touches `src/`. The builder never writes to the task vault. QA never modifies production code. Each agent has one job and a clear boundary.

## Per-Task Workflow

Every task follows the same shape:

1. User describes the work to the advisor
2. Advisor presents a plan in chat ending with "Should I proceed?"
3. User approves
4. Advisor writes `.workflow/tasks/<slug>.md` with frontmatter, a timestamped round header, `User Asked` (verbatim), and the approved `Plan`
5. Advisor spawns the builder via the Agent tool, passing the task file path
6. Builder reads the file, executes in `src/`, returns a structured report
7. Advisor records the report as the `### Implementation` subsection
8. Advisor commits the task file + `src/` changes together (one commit per round)
9. Advisor offers QA verification for logic-bearing changes
10. On task completion, advisor asks whether to run `/graphify --update`

The full canonical sequence lives in `.claude/agents/advisor.md`.

## Reference

### Agents

- **advisor** — converses with the user, plans tasks, writes the task vault, coordinates builders, runs QA. Never executes code in `src/`.
- **builder** — executes plans in `src/`. Reads the task file, does the work, reports back. Never converses with the user.
- **qa** — independently verifies builder output. Runs tests, inspects code, returns a verdict with severity-tagged findings. Can edit tests; never touches production code.

### Slash commands

- **`/onboard`** — first-time project setup. Idempotent.
- **`/whoami`** — print which agent is active in this session. Returns "default Claude" if the agent system didn't load.
- **`/set-model`** — change a per-agent model. Usage: `/set-model advisor opus`. Models: `opus`, `sonnet`, `haiku`, `inherit`.
- **`/yolo`** — skip the approval gate for a single task.
- **`/graphify`** — query the project knowledge graph (added by `graphify install`). `/graphify --update` re-indexes Markdown using an LLM (uses API calls).

### Files and folders

- **`.workflow/tasks/`** — one Markdown file per task, slug-named (`fix-login-bug.md`). The source of truth for project history.
- **`.claude/agent-memory/`** — per-agent persistent memory, committed to version control.
- **`.claude/skills/tdd/`** — optional TDD red-green-refactor skill (installed during onboarding if you opt in).
- **`CLAUDE.md`** — project-wide working principles, coding standards, architecture rules. Edit this to add your project's context, tech stack, and domain vocabulary.
- **`src/`** — your project code (initially empty).
- **`graphify-out/`** — knowledge graph artifacts (created on first commit that touches code files).

## Acknowledgements

- The optional TDD skill is from Matt Pocock's [Skills For Real Engineers](https://github.com/mattpocock/skills). When enabled, the builder follows red-green-refactor with vertical slicing on every behavior change. Highly recommended.
- [`graphify`](https://github.com/safishamsi/graphify) provides the codebase knowledge graph that agents query for context.
