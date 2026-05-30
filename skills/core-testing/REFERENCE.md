# Workflow: Testing (Phase 6)

## Trigger

Invoked after a feature is marked implementation-complete. Invoker must provide:
- Acceptance criteria source (PRD, RFD, or user story)
- Code reference (branch, commit, or changed files list)
- Test commands (from `PROJECT_CONTEXT.md`)

## Inputs

| Input | Source | Required |
|-------|--------|----------|
| Acceptance criteria | PRD / RFD / user story | Yes |
| Changed files / branch | Implementation task or git | Yes |
| Test commands | `PROJECT_CONTEXT.md` | Yes |
| Existing test suite location | `PROJECT_CONTEXT.md` | Yes |

## Steps

### Step 1 — Load context
1. Read `.claude/PROJECT_CONTEXT.md` — note stack, test commands, test file locations.
2. Read the acceptance criteria document.
3. List all changed files. If branch given: `git diff main...<branch> --name-only`.

### Step 2 — Map acceptance criteria to code
For each criterion: identify responsible code path, check if existing test covers it. Mark: `covered`, `partial`, or `missing`.

### Step 3 — Run existing test suite
Run project test command (e.g. `docker exec <php_container> vendor/bin/phpunit`, `npm test`). Capture: total, passed, failed, skipped, full failure output.

### Step 4 — Write missing tests
For each criterion marked `partial` or `missing`: write minimal test, follow existing conventions, run and verify pass.

### Step 5 — Behavior validation
For criteria not coverable by unit tests: describe manual/script validation, record actual vs expected.

### Step 6 — Defect recording

```
DEFECT-<n>
- Criterion: [exact acceptance criterion text]
- Expected: [what should happen]
- Actual: [what happened]
- Steps to reproduce: [numbered steps or failing test name]
- Severity: critical | high | medium | low
```

### Step 7 — Produce test report

```markdown
## Test Report

**Feature**: [feature name]
**Date**: [date]
**Verdict**: PASS | FAIL

### Coverage Summary

| Criterion | Status | Test |
|-----------|--------|------|
| [criterion 1] | pass/fail/skip | [test name or n/a] |

### Test Suite Results

- Total: X | Passed: X | Failed: X | Skipped: X

### Defects

[DEFECT-n list, or "None"]

### New Tests Added

[List new test files/methods, or "None"]
```

## Pass Criteria

All must be true:
1. All acceptance criteria are `covered`
2. No failing tests (existing or new)
3. No open `critical` or `high` defects

Any condition fails → verdict FAIL.
