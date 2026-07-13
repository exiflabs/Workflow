---
name: set-model
description: Change the model used by advisor, builder, or qa. Usage set-model <agent> <model-id>. Model may be any harness ID or inherit.
---

# Set model

Parse arguments: `<agent> <model-id>`.

- **agent:** `advisor` | `builder` | `qa`
- **model-id:** free-form string the harness accepts, or `inherit`

If missing/invalid, ask which agent and model. See `workflow/models.md` for recommended flagships (Opus 4.8, Sonnet 5, 5.6 family, Fable 5, Grok 4.5).

## Where to write

Update **all adapters that exist** for that agent so vendors stay aligned, unless the user names one harness only.

| Agent | Claude | Codex | Grok |
|-------|--------|-------|------|
| advisor | `.claude/agents/advisor.md` `model:` | root session only (tell user to pick model in Codex UI) | `.grok/agents/advisor.md` `model:` |
| builder | `.claude/agents/builder.md` | `.codex/agents/builder.toml` `model = "..."` | `.grok/agents/builder.md` |
| qa | `.claude/agents/qa.md` | `.codex/agents/qa.toml` `model = "..."` | `.grok/agents/qa.md` |

For `inherit`:

- Markdown frontmatter: `model: inherit`
- Codex TOML: set `model = "inherit"` or remove the `model` key if the client rejects inherit — prefer `model = "inherit"` and note if the client ignores it

## After edit

Confirm paths changed. Remind: re-spawn subagents / new session so the override applies.

If multiple agents in one request, update each.
