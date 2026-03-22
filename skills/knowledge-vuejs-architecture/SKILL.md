---
name: knowledge-vuejs-architecture
type: knowledge
description: "Vue.js architecture principles and patterns as applied in this project. Covers Composition API, Pinia stores, PrimeVue component usage, nested view structure, routing, API integration, and naming conventions. Consumed by vue-developer and solution-architect."
---

# Vue.js Architecture — Project Conventions

## Principles

1. **Pinia for all shared state** — Every piece of shared/global state lives in a Pinia store. No composables-as-stores, no prop-drilling for cross-component state.
2. **Stores own business logic** — Stores hold API calls, derived state, and actions. Components call store methods; they do not contain business logic.
3. **Split stores by feature** — One store per feature domain (e.g. `useAuthStore`, `useOrdersStore`, `useProductsStore`). Never a single monolithic store.
4. **PrimeVue for UI components** — Use PrimeVue components first. Solve UI problems with what PrimeVue provides before building custom components.
5. **Nested view structure** — Route-level views are nested under layout wrappers. Layouts handle chrome (sidebar, header); views handle content.
6. **Composition API only** — Always `<script setup>`. Never Options API.
7. **One responsibility per component** — Components receive props and emit events. They do not own business logic or make direct API calls.
8. **KISS** — Simplest approach that works. No abstractions without proven reuse (2+ uses).
9. **DRY, but not prematurely** — Extract to a composable only when the same stateless logic appears in 2+ places.
10. **Convention over invention** — Follow existing file and naming patterns before creating new ones.

## Core Philosophy

- **KISS**: simplest approach first — no features, configurability, or abstractions beyond what was asked
- **DRY, but not prematurely**: extract only when something is used 2+ times; three similar lines beats a premature abstraction
- **No over-engineering**: no loading skeletons, animations, or custom components when PrimeVue has a solution

## Architecture Layers

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Layouts | `src/layouts/` | App chrome (nav, sidebar, header); wrap route views |
| Views | `src/views/` | Route-level component; orchestrates stores and child components; no direct API calls |
| Components | `src/components/` | Presentational UI; receive props, emit events; one responsibility per component |
| Stores | `src/stores/` | Business logic, API calls, shared state; one store per feature domain |
| Composables | `src/composables/` | Shared stateless logic used in 2+ components (utilities, not state) |
| Router | `src/router/` | History mode; named routes; auth guards; lazy-loaded views |

## Key Rules

- Always use `<script setup>` (Composition API) — never Options API
- All HTTP via Axios; base URL from `import.meta.env.VITE_API_URL`; auth via session cookie
- Pinia stores own all API calls and cross-component state
- Every API call must handle: loading state, error display, success feedback
- Use PrimeVue components before building custom UI

## When to use this skill

Load when writing or reviewing any Vue.js frontend code: views, components, stores, composables, router, or API integration.

For full details — naming conventions table, component structure template, store pattern, API call pattern, form handling, error handling, build setup — see `./SKILL-DETAILS.md`.
