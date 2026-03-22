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

---

## PROGRESS.md Template

```markdown
# PROGRESS: [Project Name]

> Last updated: {DATE}

---

## Foundation

[Copy Foundation tasks verbatim from WBD.md, one checkbox per task]

---

## [Feature Name]

[Copy feature tasks verbatim from WBD.md, one checkbox per layer task]

---

[Repeat for each feature in WBD.md]
```

---

## Initialisation

When PROGRESS.md does not exist:

1. Read `WBD.md`
2. Copy every task into PROGRESS.md using the template above, all items set to `[ ]`
3. Set the project name and date

---

## Updating

When an agent reports work is done:

1. Read PROGRESS.md
2. Find the matching task by name
3. Advance its status: `[ ]` → `[🚧]` (started) or `[ ]` / `[🚧]` → `[x]` (done)
4. Update the `Last updated` date
5. Never demote a `[x]` item unless explicitly instructed

Trust the reporting agent — do not verify by reading files.
