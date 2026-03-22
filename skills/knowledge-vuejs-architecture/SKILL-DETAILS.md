# Vue.js Architecture — Full Reference

## Naming Conventions

| Element       | Convention  | Example                              |
| ------------- | ----------- | ------------------------------------ |
| Components    | PascalCase  | `ProductForm.vue`, `OrderTable.vue`  |
| Views         | PascalCase  | `ProductsView.vue`, `LoginView.vue`  |
| Layouts       | PascalCase  | `AppLayout.vue`, `AuthLayout.vue`    |
| Stores        | camelCase   | `useAuthStore.js`, `useOrdersStore.js` |
| Composables   | camelCase   | `useFormatDate.js`, `useDebounce.js` |
| Props         | camelCase   | `productId`, `isLoading`             |
| Emits         | kebab-case  | `update:modelValue`, `item-deleted`  |
| CSS classes   | kebab-case  | `.product-form`, `.error-message`    |
| Env vars      | SCREAMING   | `VITE_API_URL`, `VITE_APP_TITLE`     |

## Nested View Structure

Route-level views nest under layout components. Layouts handle app chrome; views handle content.

```
AppLayout.vue          ← sidebar, header, navigation
└── ProductsView.vue   ← route content
    ├── ProductTable.vue
    └── ProductForm.vue
```

Router example:
```js
{
  path: '/admin',
  component: AppLayout,
  children: [
    { path: 'products', name: 'products', component: () => import('../views/ProductsView.vue') },
    { path: 'orders',   name: 'orders',   component: () => import('../views/OrdersView.vue') }
  ]
}
```

## Pinia Store Pattern

One store per feature domain. Stores own API calls, state, and actions.

```js
// src/stores/useProductsStore.js
import { defineStore } from 'pinia'
import { ref } from 'vue'
import axios from 'axios'

export const useProductsStore = defineStore('products', () => {
  const items = ref([])
  const loading = ref(false)
  const error = ref(null)

  async function fetchAll() {
    loading.value = true
    error.value = null
    try {
      const res = await axios.get('/api/products')
      items.value = res.data.data
    } catch (err) {
      error.value = err.response?.data?.message ?? 'Failed to load products'
    } finally {
      loading.value = false
    }
  }

  async function save(product) {
    loading.value = true
    error.value = null
    try {
      const res = product.id
        ? await axios.patch(`/api/products/${product.id}`, product)
        : await axios.post('/api/products', product)
      return res.data.data
    } catch (err) {
      error.value = err.response?.data?.message ?? 'Save failed'
      throw err
    } finally {
      loading.value = false
    }
  }

  return { items, loading, error, fetchAll, save }
})
```

## Component Structure

Always use `<script setup>` syntax. Components call stores; they do not own API logic.

```vue
<script setup>
import { onMounted } from 'vue'
import { useProductsStore } from '@/stores/useProductsStore'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'

const store = useProductsStore()
onMounted(() => store.fetchAll())
</script>

<template>
  <div v-if="store.loading"><ProgressSpinner /></div>
  <div v-else-if="store.error" class="error">{{ store.error }}</div>
  <DataTable v-else :value="store.items">
    <Column field="name" header="Name" />
  </DataTable>
</template>
```

## PrimeVue Usage

- Use PrimeVue components as the default for all UI — tables, forms, dialogs, buttons, dropdowns, calendars.
- Check PrimeVue docs first when you need a UI element. Only build a custom component if PrimeVue has no equivalent.
- Register components locally (import in `<script setup>`) not globally unless the component is used everywhere.

Common PrimeVue components for admin interfaces:

| Use Case       | Component               |
| -------------- | ----------------------- |
| Data table     | `DataTable` + `Column`  |
| Form input     | `InputText`, `Dropdown` |
| Button         | `Button`                |
| Confirm delete | `ConfirmDialog`         |
| Modal          | `Dialog`                |
| Notification   | `Toast`                 |
| Loading        | `ProgressSpinner`       |
| File upload    | `FileUpload`            |

## Form Handling

- Use PrimeVue form inputs (`InputText`, `Dropdown`, `Calendar`, etc.)
- `v-model` on inputs for two-way binding
- Validate on submit, not on every keystroke (unless specified)
- Disable submit button while `store.loading === true`

```vue
<form @submit.prevent="handleSubmit">
  <InputText v-model="form.name" required />
  <Button type="submit" :disabled="store.loading" label="Save" />
</form>
```

## State Management Rules

- **Pinia stores** for all shared, cross-component, or persisted state
- **`ref()` / `reactive()`** for local component state only (UI toggles, local form state)
- **Composables** for shared stateless utilities (formatting, debounce) — not for state

## Error Handling

- Always show user-facing error messages — never silently swallow errors
- Map API `message` field to store error: `error.value = err.response?.data?.message`
- Use PrimeVue `Toast` for transient notifications; inline error messages for form errors
- Consistent error display pattern across all views (check existing views first)

## API Call Pattern

All API calls live in stores, not components.

```js
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
- Nested routes under layout components
