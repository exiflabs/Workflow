---
title: Universalize workflow scaffold for equal multi-vendor support
slug: universalize-workflow-scaffold
tags: [scaffold, multi-vendor, graphify, agents, memory]
related: []
status: complete
---

## Round 1 [commit: 3986e38] — 2026-07-12 22:16

### User Asked

Take a look at the code in this repo. It's somewhat out of date and only works with Claude. Graphify has a newer version. Get up to date and make it more universal for Grok and Codex (and equal for every vendor/model). Design decisions: equal for every vendor; shared memory; CLAUDE.md symlink to AGENTS.md; no hardware-comms carryover; focus on latest models (Sonnet 5, 5.6, Opus 4.8, Fable 5, Grok 4.5).

### Discussion

- Root session = advisor on all vendors; builder/QA always subagents
- Shared role contracts under `workflow/roles/`; thin platform adapters only
- Shared memory under `workflow/agent-memory/`
- Canonical rules in `AGENTS.md`; `CLAUDE.md` → symlink
- Models free-form + `workflow/models.md`; default inherit
- Graphify multi-platform install in onboard

### Plan

| Step | Action | Verify |
|------|--------|--------|
| 1 | AGENTS.md + CLAUDE.md symlink + models.md | Symlink resolves; content multi-vendor |
| 2 | Shared roles + agent-memory | Paths exist; old .claude/agent-memory removed |
| 3 | .claude / .codex / .grok adapters | Thin wrappers to roles |
| 4 | .agents/skills + Claude command wrappers | onboard/whoami/yolo/set-model |
| 5 | README + ignores | Vendor-equal quickstart |
| 6 | Task file + commit | One commit for this round |

### Implementation

- Added `AGENTS.md` as canonical multi-vendor process; `CLAUDE.md` → `AGENTS.md` symlink
- Added `workflow/roles/{advisor,builder,qa}.md` and `workflow/models.md`
- Shared memory at `workflow/agent-memory/{advisor,builder,qa}/MEMORY.md`; removed `.claude/agent-memory`
- Thin adapters: `.claude/agents/*`, `.codex/agents/{builder,qa}.toml`, `.grok/agents/*`
- Shared skills under `.agents/skills/{onboard,whoami,yolo,set-model}/`; Claude commands delegate to them
- Updated `.claude/settings.json` permissions for new paths
- Rewrote `README.md`; updated `.graphifyignore`
- Onboard is vendor-equal (no Claude restart/frontmatter gate); graphify multi-platform install

**Verification:** layout listing + `readlink CLAUDE.md` → `AGENTS.md`

**Assumptions:** Codex inherits session model when `model` is commented out; harnesses load `.agents/skills/` and/or platform agent dirs as documented.

**Blockers / follow-ups:** Users should `pipx upgrade graphifyy` (or equivalent) locally to 0.9.13+; not pinned inside this scaffold. Exact API model ID strings vary by client — documented as human names in `workflow/models.md`.

## Summary

Scaffold is vendor-equal: shared `AGENTS.md` + role contracts + memory; thin Claude/Codex/Grok adapters; shared skills; graphify multi-platform onboard; flagship model guidance only. `CLAUDE.md` symlinks to `AGENTS.md`.
