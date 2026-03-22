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

## Process (steps summary)

1. Read project context — extract scope, data model, endpoints, routes
2. Define Foundation block — infra/scaffolding tasks that must exist before any feature work
3. Identify features from scope — each must be independently deliverable
4. Break each feature into layer tasks (DB → API → Admin UI → Public SSR)
5. Identify critical path — longest sequence of dependent tasks
6. Produce parallelism map — what backend/frontend can do simultaneously
7. Compile into WBD format

## Principles

- **Foundation first**: no feature work starts before Foundation is complete
- **Vertical slices**: each feature is a complete slice from DB to UI
- **Critical path over everything**: identify it early, protect it
- **Scope fence**: only Phase 1 scope appears in the WBD
- **Owner clarity**: every task has one owner — backend or frontend

For the full step-by-step instructions, WBD output template, and RACI, see `./SKILL-DETAILS.md`.
