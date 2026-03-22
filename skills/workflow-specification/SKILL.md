---
name: workflow-specification
type: workflow
description: Phase 4 workflow — translate PRD + ADD + WBD into a fully-populated PROJECT_CONTEXT.md ready for agent execution. Triggered by delivery-manager after work breakdown is approved.
---

## Purpose

Produce a fully-populated `PROJECT_CONTEXT.md` that gives every agent unambiguous execution context — no guessing at domains, ports, paths, or container names.

## When to Use

- Phase 3 (Planning) is complete: WBD.md is approved
- Before any development agent is dispatched
- When PROJECT_CONTEXT.md exists but is incomplete or stale after a new ADD

## Inputs (required before starting)

1. **PRD or RFD** — source of scope, goals, and constraints
2. **ADD** — architecture decisions, tech stack, data model, API endpoints
3. **WBD.md** — tasks, owner assignments, dependencies

## Process

1. **Read all three inputs** — PRD/RFD, ADD, WBD.md
2. **Extract confirmed facts** — what is already decided (stack, domain, architecture decisions)
3. **Identify gaps** — what is NOT in the documents (ports, credentials pattern, VPS details, env paths)
4. **Run specification interview** — ask only about gaps (max 8 questions)
5. **Populate PROJECT_CONTEXT.md** — use the schema from `knowledge-project-context`
6. **Verify completeness** — check Phase 4 acceptance criteria below
7. **Write the file** — save to `.claude/PROJECT_CONTEXT.md`

## Specification Interview — Questions to Ask

Ask only about items not already answered by the input documents:

| Topic | Question |
|-------|----------|
| Domain | What is the production domain? |
| VPS | SSH access: `user@ip`, key path, working directory on VPS |
| Port range | Which port range is allocated for this project on the VPS? |
| Container names | What prefix or naming convention for Docker containers? |
| Database | Database name and user pattern (e.g. `project_db`, `project_user`) |
| Env files | Where are `.env` files located (local and production paths)? |
| Agents | Which agents are installed in `.claude/` for this project? |
| Open questions | Are there any open questions from the WBD that still need resolution? |

## Phase 4 Acceptance Criteria

PROJECT_CONTEXT.md is complete when all of the following are true:

- [ ] Project name and one-line purpose present
- [ ] Phase set to `Development`
- [ ] Stack lists all confirmed technologies from ADD
- [ ] Agents & Skills lists what is configured in `.claude/`
- [ ] Key Paths lists all critical directories and files
- [ ] Constraints & Decisions includes all hard decisions from ADD
- [ ] VPS connection details present (`user@ip`, key path, workdir)
- [ ] Port range allocated and documented
- [ ] All container names defined
- [ ] Database name and user pattern defined
- [ ] Environment file paths listed (local and production)
- [ ] All open questions from WBD resolved or explicitly flagged with owner
- [ ] Open Questions section is empty or contains only items pending external answer

## Principles

- **No guessing**: if a fact is not confirmed, ask — never fill in a plausible value
- **Schema fidelity**: follow the schema from `knowledge-project-context` exactly
- **One-liners only**: no prose, no paragraphs in the output file
- **Confirmed only**: Stack and Constraints must only contain facts that were explicitly decided
- **Minimal questions**: extract as much as possible from documents before interviewing
