# Workflow: Specification (Phase 4)

## Inputs Required

1. **PRD or RFD** — scope, goals, constraints
2. **ADD** — architecture decisions, tech stack, data model, API endpoints
3. **WBD.md** — tasks, owner assignments, dependencies

## Steps

### Step 1 — Read all three inputs
PRD/RFD, ADD, WBD.md. Extract confirmed facts — what is already decided (stack, domain, architecture decisions).

### Step 2 — Identify gaps
What is NOT in the documents: ports, credentials pattern, VPS details, env paths.

### Step 3 — Run specification interview
Ask only about gaps. Max 8 questions. Use table below.

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

### Step 4 — Populate PROJECT_CONTEXT.md
Use schema from `core-project-context`. One-liners only, no prose.

### Step 5 — Verify completeness
Check all Phase 4 acceptance criteria below.

### Step 6 — Write the file
Save to `.claude/PROJECT_CONTEXT.md`.

## Phase 4 Acceptance Criteria

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

## Rules

- **No guessing**: if a fact is not confirmed, ask — never fill in a plausible value
- **Schema fidelity**: follow `core-project-context` schema exactly
- **One-liners only**: no prose, no paragraphs in the output file
- **Confirmed only**: Stack and Constraints must only contain explicitly decided facts
- **Minimal questions**: extract as much as possible from documents before interviewing
