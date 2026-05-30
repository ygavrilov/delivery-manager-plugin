# PROJECT_CONTEXT.md

## File Location

`.claude/PROJECT_CONTEXT.md`

## Schema

```
# [Project Name]
[One-line purpose]

## Phase
[Discovery | Planning | Development | Release | Maintenance]

## Stack
[Confirmed technologies only, one per line]

## Agents & Skills
[List what is configured in .claude/ for this project]

## Key Paths
[Critical files or directories]

## Constraints & Decisions
[Hard limits and decisions already made]

## Open Questions
[Unresolved items blocking progress]
```

## Rules

- One-liners only — no prose
- Remove resolved items from Open Questions immediately
- Phase must always reflect current state
- Stack: only what is confirmed, never speculative
