# Vue.js Architecture — Full Reference

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

## API Call Pattern

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

## State Management

- **No Pinia/Vuex** unless the project has explicitly adopted it (check PROJECT_CONTEXT.md)
- Use `ref()` and `reactive()` for local component state
- Use composables for shared state: `const { user, logout } = useAuth()`
- Pass state down via props; communicate up via emits

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

## Router

- History mode (no hash)
- Route guards for authentication: redirect to `/login` on 401
- Named routes for programmatic navigation
- Lazy-loaded views: `component: () => import('../views/FooView.vue')`
