---
name: project-context-manager
argument-hint: "[project-name]"
description: Create and maintain a minimal Project Context document (.claude/PROJECT_CONTEXT.md); apply single-line corrections when errors repeat.
context: fork
color: green
---

## What this skill does

- Creates a new Project Context document (.claude/PROJECT_CONTEXT.md) from a short interview.
- Edits an existing Project Context document to fix recurring model mistakes.
- Keeps the document minimal and high-signal.

## When to use

- At project start, before creating role skills.
- After a wrong output repeats twice (add one corrective line).

## Required inputs (ask if missing)

1. Project name ($0)
2. Baseline / source of truth for parity (env or spec)

## Output

Return one of:

- **New document content** (ready to paste/save), or
- **Patch**: show exactly which lines to add/change/remove.

## Hard rules (anti-contamination)

- Every line must be an invariant that changes decisions.
- No checklists. No background. No nice-to-know.
- If you cannot confirm a fact, write it as a question placeholder (do not guess).
- Prefer a **single-line correction** over rewrites.

## Canonical template (keep it short)

### Objective

(one line)

### Scope boundary

(one line)

### Parity / Quality bar

(one line)

### Source of truth

(one line)

### Links

(list of links only)

## Procedure

1. Ask 3–7 questions max to fill the template.
2. Draft the Project Context using the canonical template.
3. Ask the user to confirm the draft.
4. On later invocations, accept an "observed error" description and add the smallest corrective line that prevents it. (See <attachments> above for file contents. You may not need to search or read the file again.)
