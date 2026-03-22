---
name: workflow-progress-tracker
type: workflow
description: Template and update rules for maintaining PROGRESS.md. Updated by agents reporting task completion — no file inference.
user-invocable: false
allowed-tools: Read, Write
---

## Output Contract

- Always write to `PROGRESS.md` at project root
- Never modify `WBD.md`
- Use `WBD.md` as the single source of truth for task definitions
- Status indicators: `[ ]` pending · `[🚧]` in progress · `[x]` done

## When to use this skill

Load when initialising a new PROGRESS.md or updating task status after an agent reports completion.

For the PROGRESS.md template and detailed update procedure, see `./SKILL-DETAILS.md`.
