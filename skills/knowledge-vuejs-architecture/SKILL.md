---
name: knowledge-vuejs-architecture
type: knowledge
description: "Vue.js architecture principles and patterns as applied in this project. Covers Composition API, component structure, routing, API integration, state management, and naming conventions. Consumed by vue-developer and solution-architect."
---

# Vue.js Architecture — Project Conventions

## Core Philosophy

### KISS — Keep It Simple

- Always start with the simplest possible approach
- Don't add abstractions, composables, or utilities beyond what was asked
- A simple component that works is better than an elegant one that's over-built

### DRY — But Not Prematurely

- Extract to a composable ONLY when the same logic appears in 2 or more components
- Three similar lines in different components is better than a premature composable
- Don't create helpers or utilities for one-time use

### No Over-Engineering

- Only make changes directly requested or clearly necessary
- Don't add loading skeletons, animations, or UX improvements unless specified
- Don't add component libraries — plain HTML and project CSS only
- Only add comments where logic isn't self-evident

## Architecture: What Goes Where

### Views (`src/views/`)

Views handle:
1. Route-level component — one per page
2. Fetch data on mount via composables or direct Axios
3. Pass data down to child components via props
4. Handle page-level loading and error states

Views MUST NOT contain reusable UI logic or complex business calculations.

### Components (`src/components/`)

Components handle:
1. Presentational UI — receive data via props, emit events upward
2. Form inputs, tables, modals, navigation elements
3. One responsibility per component — if it does two things, split it

Components MUST NOT fetch data directly (unless a self-contained widget).

### Composables (`src/composables/`)

Composables handle:
1. Shared stateful logic used in 2+ components (auth state, form validation)
2. Wrap Axios calls that are reused across multiple views
3. Return reactive refs and methods — no template logic

Only create a composable when the same logic is genuinely reused.

### Router (`src/router/`)

- History mode (no hash)
- Route guards for authentication: redirect to `/login` on 401
- Named routes for programmatic navigation
- Lazy-loaded views: `component: () => import('../views/FooView.vue')`

### API Integration

- All HTTP calls use Axios
- Base URL configured via `import.meta.env.VITE_API_URL`
- Auth via session cookie — no manual token headers
- Every API call must handle: loading state, error display, success feedback

```js
const loading = ref(false)
const error = ref(null)

async function fetchData() {
  loading.value = true
  error.value = null
  try {
    const response = await axios.get('/api/resource')
    data.value = response.data.data
  } catch (err) {
    error.value = err.response?.data?.message ?? 'Request failed'
  } finally {
    loading.value = false
  }
}
```

## Component Structure

Always use `<script setup>` syntax (Composition API only — never Options API):

```vue
<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const props = defineProps({ id: Number })
const emit = defineEmits(['saved'])

const data = ref(null)
const loading = ref(false)
const error = ref(null)

onMounted(() => fetchData())

async function fetchData() { /* ... */ }
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else-if="error" class="error">{{ error }}</div>
  <div v-else><!-- content --></div>
</template>
```

## Naming Conventions

| Element       | Convention  | Example                              |
| ------------- | ----------- | ------------------------------------ |
| Components    | PascalCase  | `ProductForm.vue`, `OrderTable.vue`  |
| Views         | PascalCase  | `ProductsView.vue`, `LoginView.vue`  |
| Composables   | camelCase   | `useAuth.js`, `useProducts.js`       |
| Props         | camelCase   | `productId`, `isLoading`             |
| Emits         | kebab-case  | `update:modelValue`, `item-deleted`  |
| CSS classes   | kebab-case  | `.product-form`, `.error-message`    |
| Env vars      | SCREAMING   | `VITE_API_URL`, `VITE_APP_TITLE`     |

## State Management

- **No Pinia/Vuex** unless the project has explicitly adopted it (check PROJECT_CONTEXT.md)
- Use `ref()` and `reactive()` for local component state
- Use composables for shared state: `const { user, logout } = useAuth()`
- Pass state down via props; communicate up via emits

## Form Handling

- Plain HTML `<form>` elements — no form libraries
- `v-model` on inputs for two-way binding
- Validate on submit, not on every keystroke (unless specified)
- Disable submit button while `loading.value === true`

```vue
<form @submit.prevent="handleSubmit">
  <input v-model="form.name" type="text" required />
  <button type="submit" :disabled="loading">Save</button>
</form>
```

## Sort Order Controls

For items with manual sort order:
- Up/Down arrow buttons or drag-and-drop (check PROJECT_CONTEXT.md for which pattern to use)
- PATCH the order immediately on move, show optimistic UI update
- Revert on API error

## Error Handling

- Always show user-facing error messages — never silently swallow errors
- Map API `message` field to UI: `error.value = err.response?.data?.message`
- Use consistent error display pattern across all views (check existing components first)

## Build & Environment

- Build tool: Vite
- Dev server: `npm run dev`
- Build: `npm run build` (outputs to `dist/`)
- Env vars: `.env` file with `VITE_` prefix for client-side exposure
