---
description: Initialize Pi session context for Kanri — load guidance docs, establish working context, prepare for tasks
argument-hint: "[task description]"
---

# Session Initialization: Kanri

Task argument: "$@"

## Instructions

### Step 1 — Load guidance docs

Read both files if they exist:
- `PROJECT_OVERVIEW.md`
- `EDITING_RULES.md`

If either file is missing, note it and suggest running `/analyse` to regenerate them.

### Step 2 — If task argument is empty (just `/init`)

Do NOT inspect the repo further unless guidance is missing or clearly stale.

Summarize concisely:
- App architecture (one paragraph)
- Important directories (bullet list)
- Frontend/backend boundary
- Editing constraints and forbidden areas
- Validation commands to run before committing

After summarizing, state that session context is loaded and that future task prompts should follow the loaded guidance without rereading the docs (unless the user asks, the task is risky, or context appears stale).

### Step 3 — If task argument is non-empty (`/init <task description>`)

In addition to Step 2:

a) Identify which sections of PROJECT_OVERVIEW.md and EDITING_RULES.md are relevant to the stated task.
b) Inspect only the likely relevant source files (use `read`, not `bash find` or `rg` for searching — prefer targeted reads).
c) Summarize task-specific context (relevant files, affected areas, risk level).
d) Propose a short plan (3-5 steps, bullet points).
e) End with "Shall I proceed with this plan?" and wait for user confirmation before making any edits.

### Constraints

- Do NOT modify any files during this command.
- Do NOT run expensive commands (no `find`, no `rg` on the whole repo, no builds).
- Keep the summary concise — do not reprint full document contents.
- After initialization, assume subsequent task prompts follow the loaded guidance. Do not reread the docs before every task unless the user asks, the task is risky, or the context appears stale.
