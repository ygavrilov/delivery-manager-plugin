---
name: knowledge-cakephp-architecture
type: knowledge
description: "CakePHP architecture principles and patterns as applied in this project. Covers Controller/Service/Table/Entity layering, naming conventions, response structure, enums, authentication, and logging. Consumed by cakephp-developer and solution-architect."
---

# CakePHP Architecture — Project Conventions

## Principles

1. **Migrations always use `up()` and `down()`** — Never `change()`.
2. **All schema changes go through migrations** — No manual SQL, no ALTER TABLE by hand.
3. **Migration names follow `ActionTableColumn`** — e.g. `AddApiTokenToUsersTable`.
4. **Every migration field has a `comment`** — No undocumented columns.
5. **Foreign keys and indexes are named** — `fk_{table}_{field}` and `idx_{table}_{field}`.
6. **Business logic lives in Services** — Tables hold queries and validation; controllers hold nothing but HTTP glue.
7. **Controllers are thin** — Check method, get identity, get data, call service, return response. Nothing else.
8. **API error handling is two-catch** — `InvalidArgumentException` → 400, `Exception` → 500. No other patterns.
9. **Response shape is fixed** — Success: `{success, message, data}`. Error: `{success, message, error_code}`.
10. **Authentication uses the CakePHP Authentication plugin** — Never check auth manually in a controller.
11. **Browser session auth uses cookies only** — No bearer tokens for browser-based sessions.
12. **Bake models after every migration** — Never let code drift from schema.

## Core Philosophy

- **KISS**: simplest approach first — no features, configurability, or abstractions beyond what was asked
- **DRY, but not prematurely**: extract only when something is used 2+ times; three similar lines beats a premature abstraction
- **No over-engineering**: no extra error handling, feature flags, docstrings, or comments beyond what logic requires

## Architecture Layers

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Controller | `src/Controller/` | HTTP method check, get identity, get data, call service, return response — nothing else |
| Service | `src/Service/` | Business logic, cross-table orchestration, throws `InvalidArgumentException` / `RuntimeException` |
| Table | `src/Model/Table/` | Validation, build rules, custom finders, associations, schema type mapping |
| Entity | `src/Model/Entity/` | `$_accessible`, accessor/mutator methods, entity-own helpers |
| Enum | `src/Model/Enum/` | String-backed PHP enums; `{Entity}{Column}` naming; UPPERCASE cases |

## Response Shape

- **Success**: `{ "success": true, "message": "...", "data": {...} }`
- **Error**: `{ "success": false, "message": "...", "error_code": "VALIDATION_ERROR|RUNTIME_ERROR|INTERNAL_ERROR" }`

## When to use this skill

Load when writing or reviewing any CakePHP backend code: controllers, services, tables, entities, enums, or migrations.

For full details — naming conventions table, HTTP status code map, auth/logging rules, migration template — see `./SKILL-DETAILS.md`.
