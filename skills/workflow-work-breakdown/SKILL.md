---
name: workflow-work-breakdown
type: workflow
description: Process for breaking down project scope into a Foundation block and Feature vertical slices with critical path and parallelism analysis. Use after architecture is approved and before implementation begins.
---

## Purpose

Produce a Work Breakdown Document (WBD) that organises all implementation work into:
1. A **Foundation block** — infrastructure and scaffolding prerequisites
2. **Feature blocks** — vertical slices from DB through to UI
3. A **critical path** — the sequence where any delay delays the project
4. A **parallelism map** — what backend and frontend can work on simultaneously

## When to Use

- Architecture is approved (PROJECT_CONTEXT.md is complete and signed off)
- Before implementation begins
- When producing a fixed-price proposal (WBD becomes the scope line items)

## Inputs (required before starting)

1. **PROJECT_CONTEXT.md** — approved scope, data model, API endpoints, site structure
2. **Team roles** — who owns backend (CakePHP) and who owns frontend (Vue.js)

## Process

### Step 1 — Read project context

- Agent: delivery-manager
- Read `PROJECT_CONTEXT.md`
- Extract: Phase 1 scope, out-of-scope items, data model, API endpoints, admin routes, public routes, stack constraints

### Step 2 — Define Foundation block

- Agent: solution-architect or delivery-manager
- List all infrastructure and scaffolding tasks that must exist before any feature work can begin
- Foundation always includes:
  - VPS provisioning and Docker Compose setup
  - Nginx routing config (reverse proxy, SSL, path routing rules)
  - Base CakePHP application scaffold (app skeleton, routing prefix, auth plugin, base layout)
  - Base Vue 3 + Vite admin scaffold (router, auth guard, Axios instance, login view)
  - MariaDB container + baseline migration (empty schema, admin user seed)
  - Deployment process (SSH + docker compose workflow or equivalent)
- Mark each Foundation task with its owner (backend / frontend / devops / shared)
- Foundation block is always on the critical path — nothing else starts until it is done

### Step 3 — Identify features from scope

- Agent: delivery-manager
- List every discrete user-facing capability from Phase 1 scope
- Each feature must be independently deliverable and testable
- Use the admin panel sections and public pages as the primary source
- Example features: Product Catalog Management, FAQ Management, Store Management, Settings & SEO Globals, Public Product Pages (SSR), Public Where-to-Buy Page (SSR), FAQ Page (SSR), Home Page (SSR), Sitemap & robots.txt, Analytics Setup

### Step 4 — Break each feature into layer tasks

- Agent: solution-architect + developer agents (as needed)
- For each feature, produce tasks per affected layer. Only include layers that apply:
  - **DB** — migration (table creation, seed data if needed)
  - **API** — CakePHP endpoints (list, get, create, update, delete, reorder as applicable)
  - **Admin UI** — Vue 3 view and components (list view, form, sort controls)
  - **Public SSR** — CakePHP PHP template page (layout, data fetch, Schema markup)
- Assign owner: `backend` (CakePHP developer) or `frontend` (Vue developer)
- Mark dependencies: what this task is **blocked by** and what it **blocks**

### Step 5 — Identify critical path

- Agent: delivery-manager
- The critical path is the longest sequence of dependent tasks
- Standard critical path for this stack:
  ```
  Foundation → DB migrations → CakePHP API endpoints → [Admin UI | Public SSR pages]
  ```
- Within features: DB migration must precede API, API must precede Admin UI and Public SSR
- Across features: features with shared DB dependencies are serialised; independent features are parallel
- Mark each task as `critical` (on the critical path) or `parallel` (can be done concurrently)

### Step 6 — Produce parallelism map

- Agent: delivery-manager
- Show which tasks backend and frontend can work on simultaneously
- General rule: frontend can begin scaffolding and static views immediately; API-dependent views wait for API tasks
- Identify the earliest point frontend can start feature work (typically: Foundation backend done + DB migration done + API contract agreed)

### Step 7 — Produce Work Breakdown Document

- Agent: delivery-manager
- Compile all output into the WBD format below
- Review against PROJECT_CONTEXT.md scope to ensure nothing is missing and nothing out-of-scope is included

## Output: Work Breakdown Document

```
# WBD: [Project Name]

## Foundation

| Task | Owner | Blocks |
|------|-------|--------|
| VPS provisioning + Docker Compose setup | backend | everything |
| Nginx routing config (reverse proxy, SSL, /api/, /admin/, /uploads/) | backend | everything |
| Base CakePHP scaffold (skeleton, routing prefix, auth plugin, base layout + GTM) | backend | all API tasks, all SSR tasks |
| Base Vue 3 + Vite admin scaffold (router, auth guard, Axios instance, login view) | frontend | all Admin UI tasks |
| MariaDB container + baseline migration + admin user seed | backend | all DB tasks |
| Deployment process (SSH + docker compose) | backend | go-live |

---

## Features

### [Feature Name]

| Layer | Task | Owner | Blocked by | Blocks |
|-------|------|-------|------------|--------|
| DB | Migration: [table name] | backend | Foundation | API tasks for this feature |
| API | [GET/POST/PUT/DELETE] /api/... | backend | DB migration | Admin UI, Public SSR |
| Admin UI | [View name] — [description] | frontend | API | — |
| Public SSR | [Page name] — [description] | backend | DB migration | — |

*Repeat for each feature.*

---

## Critical Path

```
Foundation
  └→ [Feature A] DB migration
       └→ [Feature A] API endpoints
            ├→ [Feature A] Admin UI
            └→ [Feature A] Public SSR page
  └→ [Feature B] DB migration
       └→ ...
```

Mark: `[CRITICAL]` for tasks on the longest dependency chain.

---

## Parallelism Map

| Phase | Backend (CakePHP) | Frontend (Vue) |
|-------|-------------------|----------------|
| Foundation | Docker, Nginx, CakePHP scaffold, DB baseline | Vue scaffold, login view, static layout |
| Feature development | DB migrations → API endpoints | Admin UI views (after API contract agreed) |
| Public pages | SSR page templates | — |
| Integration | API testing, SEO/Schema | Admin UI wiring, route guards |

```

## Principles

- **Foundation first**: no feature work starts before Foundation is complete (or explicitly agreed to be parallel with risk)
- **Vertical slices**: each feature is a complete slice from DB to UI — avoids half-built features at end of project
- **Critical path over everything**: identify it early, protect it; delay on critical path = project delay
- **Scope fence**: if a task is not in Phase 1 scope in PROJECT_CONTEXT.md, it does not appear in the WBD
- **Owner clarity**: every task has one owner — backend or frontend, never ambiguous

## RACI

- Foundation definition
  - R: solution-architect / delivery-manager
  - A: delivery-manager
  - C: cakephp-developer, vue-developer
  - I: stakeholders
- Feature breakdown
  - R: delivery-manager
  - A: delivery-manager
  - C: solution-architect, cakephp-developer, vue-developer
  - I: stakeholders
- Critical path identification
  - R: delivery-manager
  - A: delivery-manager
  - C: solution-architect
  - I: stakeholders
