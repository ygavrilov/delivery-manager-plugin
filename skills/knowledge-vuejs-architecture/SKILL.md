---
name: knowledge-vuejs-architecture
type: knowledge
description: "Vue.js architecture principles and patterns as applied in this project. Covers Composition API, component structure, routing, API integration, state management, and naming conventions. Consumed by vue-developer and solution-architect."
---

# Vue.js Architecture — Project Conventions

## Core Philosophy

- **KISS**: simplest approach first — no abstractions, composables, or utilities beyond what was asked
- **DRY, but not prematurely**: extract to a composable only when the same logic appears in 2+ components
- **No over-engineering**: no loading skeletons, animations, or component libraries — plain HTML and project CSS only

## Architecture Layers

| Layer | Location | Responsibility |
|-------|----------|----------------|
| Views | `src/views/` | Route-level component; fetch data on mount; pass to children; handle loading/error |
| Components | `src/components/` | Presentational UI; receive props, emit events; one responsibility per component |
| Composables | `src/composables/` | Shared stateful logic used in 2+ components; return reactive refs and methods |
| Router | `src/router/` | History mode; named routes; auth guards; lazy-loaded views |

## Key Rules

- Always use `<script setup>` (Composition API) — never Options API
- All HTTP via Axios; base URL from `import.meta.env.VITE_API_URL`; auth via session cookie
- No Pinia/Vuex unless PROJECT_CONTEXT.md says otherwise — use composables for shared state
- Every API call must handle: loading state, error display, success feedback

## When to use this skill

Load when writing or reviewing any Vue.js frontend code: views, components, composables, router, or API integration.

For full details — naming conventions table, component structure template, API call pattern, form handling, sort order, error handling, build setup — see `./SKILL-DETAILS.md`.
