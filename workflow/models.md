# Recommended models

Defaults are **inherit** (use whatever model the root session selected). Override per role only when you want a deliberate split.

Use free-form model IDs your harness accepts. Names below are the intended flagship tier for this scaffold — adjust the exact API string to match your client’s model picker.

| Vendor | Advisor | Builder | QA | Notes |
|--------|---------|---------|-----|--------|
| **Anthropic / Claude** | Opus 4.8 | Sonnet 5 | Sonnet 5 | Planning on Opus; implementation/verify on Sonnet is a good cost split |
| **OpenAI / Codex** | 5.6 (strongest available) | 5.6 or Fable 5 | 5.6 | Prefer the newest 5.6 family for advisor/QA; Fable 5 is fine for focused builder work |
| **xAI / Grok** | Grok 4.5 | Grok 4.5 | Grok 4.5 | Match the session default unless you pin a variant |
| **Any** | `inherit` | `inherit` | `inherit` | Safest multi-vendor default |

## Where overrides live

| Harness | Files |
|---------|--------|
| Claude Code | `.claude/agents/{advisor,builder,qa}.md` → `model:` frontmatter |
| Codex | `.codex/agents/{builder,qa}.toml` → `model = "..."` (advisor = root session model) |
| Grok | `.grok/agents/{advisor,builder,qa}.md` → `model:` frontmatter |

Change via the `set-model` skill or by editing those files. Restart or re-spawn subagents so overrides take effect.

## set-model usage

```
/set-model <agent> <model-id>
```

- **agent:** `advisor` | `builder` | `qa`
- **model-id:** any string your harness accepts, or `inherit`

Examples: `opus-4.8`, `sonnet-5`, `fable-5`, `grok-4.5`, `gpt-5.6`, `inherit`.
