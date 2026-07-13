---
name: yolo
description: Skip the approval gate for the next task. State the plan briefly and execute immediately.
---

# Yolo

Skip the approval gate for this task only.

1. Briefly state the plan (slug, steps) so the user can see it.
2. Immediately write/update `workflow/tasks/<slug>.md` for this round.
3. Spawn the builder (and continue the normal post-builder sequence from `workflow/roles/advisor.md`).

Do not wait for **Should I proceed?**

User request follows any arguments passed with the skill invocation.
