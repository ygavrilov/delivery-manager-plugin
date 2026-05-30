# Architecture Decision Rules

## How to Think Through a Decision

1. **Understand context** — read PROJECT_CONTEXT.md, identify current state
2. **Clarify requirements** — what problem, what constraints, what is in/out
3. **Identify affected layers** — which parts of the stack are involved
4. **Evaluate 2–3 options** — with explicit trade-offs (complexity, performance, maintainability)
5. **Recommend one** — the simplest that meets all requirements (KISS)
6. **Define integration points** — API contracts, data shapes, how layers communicate
7. **Identify risks** — dependencies, failure modes, rollback path
8. **Document** — produce ADD following `workflow-solution-architecture` format

## Domain Knowledge Map

| Domain | What to assess |
|--------|---------------|
| Backend | MVC layering, service boundaries, REST API design, authentication, session handling |
| Frontend | SPA vs SSR vs SSG, component hierarchy, state management, routing |
| Database | Normalization, indexing strategy, migration safety, ORM conventions |
| Infrastructure | Containerization, reverse proxy, caching layers, deployment model |
| SEO | Technical SEO, structured data, Core Web Vitals, crawlability, rendering strategy |
| Performance | Asset optimization, caching, rendering strategy, DB query efficiency |
| Analytics | Tag management, event taxonomy, consent handling, data layer design |

Apply knowledge through the lens of the actual project stack (PROJECT_CONTEXT.md) — not from assumptions.

## ADD Scope Rules

- One decision per ADD — do not bundle unrelated decisions
- Document the decision AND the rejected alternatives with reasons
- State assumptions explicitly; any assumption that breaks invalidates the decision
- ADD is final when the orchestrator approves it — do not revise without a new decision record

## Principles

- **KISS**: always recommend the simplest viable option; reject complexity that doesn't serve a stated requirement
- **Fewer moving parts**: fewer components = fewer failure modes
- **DRY but not premature**: identify true duplication; do not abstract for hypothetical reuse
- **Convention first**: check if a pattern already exists in the project before proposing a new one
