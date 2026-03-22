# SDLC — Full Phase Reference

## Phase 0 — Scoping

**Status**: Covered

Establish what the project is, what it will cost, and whether to proceed. Happens before the project is formally started — output is a proposal or estimate for the client.

### Deliverables

#### Proposal / Estimate
- Scope defined at feature level (not task level)
- Effort estimated
- RACI established
- Client accepts before Phase 1 begins

### Agents & Skills
- `business-analyst` — clarifies client needs
- `rfq-knowledge` skill — RFQ structure checklist and minimal RACI

---

## Phase 1 — Requirements

**Status**: Covered

Understand what is being built, for whom, and why. Produces the document that all subsequent phases work from.

### Deliverables

#### PRD (Product Requirements Document)
For user-facing features or new products.
- Problem and users clearly stated
- Scope boundary explicit (in / out)
- Requirements listed
- Success criteria defined
- Open questions resolved before Phase 2

#### RFD (Request for Development)
For technical changes, integrations, or system improvements.
- Problem statement clear
- Proposed change defined
- Constraints listed
- Acceptance criteria defined
- Open questions resolved before Phase 2

### Agents & Skills
- `business-analyst` — gathers and clarifies requirements, maps stakeholders
- `requirements-gathering` skill — interview process
- `stakeholder-mapping` skill — identifies who is affected and how
- `process-analysis` skill — maps existing business processes
- `prd` skill — PRD template
- `rfd` skill — RFD template

---

## Phase 2 — Architecture

**Status**: Covered

Define how the system will be built. Evaluate options, make decisions, document them. No code is written in this phase.

### Deliverables

#### ADD (Architecture Decision Document)
- Problem statement is one clear paragraph
- 2–3 options evaluated with trade-offs
- One option recommended with justification
- All affected layers defined (DB, backend, frontend, infra, SEO/analytics)
- API contracts specified for new or changed endpoints
- Risks and mitigations listed
- Open questions flagged for stakeholder input

### Agents & Skills
- `solution-architect` — designs solutions, produces ADDs
- `database-architect` — designs schema, indexing strategy, migration plan
- `solution-architecture` skill — process for producing ADDs

---

## Phase 3 — Planning

**Status**: Covered

Break the approved architecture into an executable work breakdown. Identify the critical path and where work can run in parallel.

### Deliverables

#### WBD.md (Work Breakdown Document)
- Foundation block defined with all infrastructure prerequisites
- Every feature broken into vertical slices (DB → API → Admin UI → Public SSR)
- Each task has one owner: `backend`, `frontend`, or `devops`
- Each task has explicit `blocked by` and `blocks` dependencies
- Critical path identified
- Parallelism map produced (what backend and frontend can do simultaneously)
- Nothing outside Phase 1 scope appears in the WBD

### Agents & Skills
- `delivery-manager` — orchestrates the breakdown
- `solution-architect` — consulted on layer boundaries and dependencies
- `work-breakdown` skill — full process for producing WBD.md

---

## Phase 4 — Specification

**Status**: ⚠️ Not covered — skill and process to be defined

Translate the WBD into execution-ready context. Fill in all concrete details that agents need to actually build: domain names, folder structure, port ranges, container names, database credentials pattern, environment file locations. Without this phase, agents have to guess at specifics.

### Deliverables

#### PROJECT_CONTEXT.md (fully populated)
- Project name and domain confirmed
- VPS connection details present (`user@ip`, key path, workdir)
- Port range allocated and documented
- All container names defined
- Database names and user pattern defined
- Environment file paths listed
- All open questions from WBD resolved

### Agents & Skills
- `project-context-manager` — creates and updates PROJECT_CONTEXT.md
- `project-context` skill — schema for PROJECT_CONTEXT.md
- **Specification interview skill** — ⚠️ to be created

---

## Phase 5 — Development

**Status**: Covered (execution orchestration skill to be defined)

Agents execute WBD tasks in critical-path order. Progress is tracked in parallel. Each task is dispatched to the appropriate agent with PROJECT_CONTEXT.md and the task description as context.

Owner mapping from WBD to agents:
- `backend` → `cakephp-developer`
- `frontend` → `vue-developer`
- `devops` → `devops`
- `database` → `database-architect` (design) + `cakephp-developer` (migration)

Progress tracked in parallel by `progress-tracker` skill — agents report completion, PROGRESS.md is updated after each task.

### Deliverables

#### Working feature code (per WBD task)
- Task matches the definition in WBD.md
- Follows patterns from the relevant expertise skill (`cakephp-architecture`, `vps-patterns`)
- Migrations run without error
- No regressions in previously completed tasks

#### PROGRESS.md (maintained throughout)
- Every WBD task has a status: `[ ]`, `[🚧]`, or `[x]`
- Updated after every agent completion report
- `Last updated` date current

### Agents & Skills
- `cakephp-developer` — backend features, migrations, services, controllers
- `vue-developer` — admin SPA views, components, forms, routing
- `devops` — infrastructure, nginx, SSL, deploy scripts, Docker
- `database-architect` — schema and index design before migration is written
- `cakephp-architecture` skill — CakePHP patterns consumed by cakephp-developer
- `vps-patterns` skill — VPS conventions consumed by devops
- `progress-tracker` skill — PROGRESS.md maintenance
- **WBD execution skill** — ⚠️ to be created (orchestration logic for dispatching tasks in order)

---

## Phase 6 — Testing

**Status**: ⚠️ Not covered — agent and skill to be defined

Verify that what was built matches acceptance criteria from Phase 1 and behaves correctly end-to-end.

### Deliverables

#### Test report
- All features from WBD verified against Phase 1 acceptance criteria
- Critical user flows tested (happy path + key failure cases)
- No critical or high-severity bugs open
- Performance acceptable (page load, API response times)

### Agents & Skills
- **QA agent** — ⚠️ to be created
- **Testing skill** — ⚠️ to be created

---

## Phase 7 — Deployment

**Status**: Partially covered — formal deployment checklist to be defined

Ship the built and tested application to production.

### Deliverables

#### Production deployment
- Application accessible at the production domain
- SSL certificate valid
- All environment variables set correctly in production
- Database migrations applied on production
- Monitoring and backup active
- Deployment script and cron update-checker in place

### Agents & Skills
- `devops` — executes deployment (nginx, SSL, Docker, cron)
- `vps-patterns` skill — deployment patterns and new project checklist
- `vps-ssh-setup` skill — one-time VPS access setup
- **Deployment checklist skill** — ⚠️ to be created (formal go-live gate)

---

## Phase 8 — Maintenance

**Status**: ⚠️ Not covered — process to be defined

Handle change requests, bug reports, and operational issues after go-live. Each non-trivial change re-enters the SDLC at the appropriate phase (bug fix may start at Phase 5; new feature starts at Phase 1).

### Deliverables

#### Change implemented or incident resolved
- Root cause identified (for bugs)
- Change scoped and approved before implementation
- Deployed and verified using the same deployment process as Phase 7

### Agents & Skills
- `devops` — operational incidents, infra changes
- All developer agents — code changes
- **Maintenance triage skill** — ⚠️ to be created (determines re-entry point in SDLC)
