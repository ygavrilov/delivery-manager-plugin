---
name: workflow-testing
description: Phase 6 QA workflow — structured test run from acceptance criteria through test execution to test report.
---

## Trigger

Invoked by `delivery-manager` (or `project-manager`) after a feature is marked implementation-complete. The invoker must provide:

- **Acceptance criteria source**: PRD, RFD, or user story with explicit criteria
- **Code reference**: branch, commit, or list of changed files
- **Test commands**: how to run the project's test suite (from `PROJECT_CONTEXT.md`)

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
2. Read the acceptance criteria document (PRD, RFD, or user story).
3. List all changed files in scope. If a branch was given, use `git diff main...<branch> --name-only`.

### Step 2 — Map acceptance criteria to code

For each acceptance criterion:
- Identify the code path(s) responsible for satisfying it.
- Note whether an existing test already covers it.
- Mark: `covered`, `partial`, or `missing`.

### Step 3 — Run existing test suite

Run the project test command (e.g. `docker exec <php_container> vendor/bin/phpunit`, `npm test`).

Capture:
- Total tests, passed, failed, skipped
- Full output of any failures

### Step 4 — Write missing tests

For each criterion marked `partial` or `missing`:
- Write the minimal test that validates the criterion.
- Follow existing test file conventions (file location, naming, base class).
- Run the new tests and verify they pass.

### Step 5 — Behavior validation

For criteria not easily covered by unit tests (UI flows, integration behavior):
- Describe the manual or script-based validation performed.
- Record actual vs expected outcome.

### Step 6 — Defect recording

For each failure or unmet criterion, record:

```
DEFECT-<n>
- Criterion: [exact acceptance criterion text]
- Expected: [what should happen]
- Actual: [what happened]
- Steps to reproduce: [numbered steps or failing test name]
- Severity: critical | high | medium | low
```

### Step 7 — Produce test report

Output a structured report (post as issue comment or document):

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

[List defects using DEFECT-n format, or "None" if verdict is PASS]

### New Tests Added

[List new test files/methods added, or "None"]
```

## Output

- Test report posted as a comment on the feature issue (or as `document/test-report`)
- Verdict: **PASS** (hand off to done) or **FAIL** (reassign to developer with defect list)

## Pass Criteria

All of:
1. All acceptance criteria are `covered`
2. No failing tests (existing or new)
3. No open `critical` or `high` defects

If any condition fails, verdict is FAIL.
