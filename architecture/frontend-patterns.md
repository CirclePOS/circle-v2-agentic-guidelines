# Frontend Patterns

> **Note:** This document covers **Vue 3 (legacy) patterns** for maintenance and bug fixes on existing Vue layouts.
>
> **For new backoffice UI:** Use **Hotwire** (Rails views + Turbo/Stimulus). See `code-standards-hotwire.md` for Hotwire patterns, ViewComponent structure, and Stimulus controller standards.

---

## Vue 3 Patterns (Legacy)

### Turborepo Structure (`app/vue3/`)

```
app/vue3/
├── apps/
│   ├── backoffice/        # Web backoffice — Element Plus, Vue Router, Pinia, TanStack Query, Vite
│   └── pos/               # POS terminal — web + Capacitor (iOS/Android)
└── packages/
    ├── api/               # Per-domain axios API modules (e.g. saleOrders/, accounts/)
    ├── components/        # Shared Vue components
    ├── composables/       # Shared composables
    ├── lang/              # vue-i18n translations
    └── utils/             # request.ts, platform detection, session helpers
```

All packages referenced as `@repo/<name>`.

---

### Feature Folder Pattern (Vue 3)

Every feature folder should contain:

```
<feature>/
├── index.vue              # Entry component
├── queries.ts             # TanStack Query key factory + useQuery/useMutation hooks
├── schema.ts              # Zod schema + z.infer<> types
└── use<Feature>FormSubmit.ts   # Form submission composable
```

### `queries.ts` pattern
```typescript
export const productKeys = {
  all: ['products'] as const,
  list: (params: SearchParams) => [...productKeys.all, 'list', params] as const,
}

export function useProducts(params: SearchParams) {
  return useQuery({
    queryKey: productKeys.list(params),
    queryFn: () => productApi.search(params),
  })
}
```

### `schema.ts` pattern
```typescript
export const productSchema = z.object({
  name: z.string().min(1),
  price: z.number().positive(),
})

export type ProductForm = z.infer<typeof productSchema>
// Never write a separate interface for form data — use z.infer<>
```

---

### API Module Pattern (`@repo/api`, Vue 3)

```typescript
// packages/api/src/products/index.ts
import { request } from '@repo/utils'
import type { AxiosPromise } from 'axios'

const productApi = {
  search(params: SearchParams): AxiosPromise<ProductSearchResponse> {
    return request.get('/api/v1/products', { params })
  },
  create(data: CreateProductParams): AxiosPromise<Product> {
    return request.post('/api/v1/products', data)
  },
}

export default productApi
```

Rules:
- Default export object with named methods
- `AxiosPromise<T>` return types
- Use `request` from `@repo/utils` — never import axios directly in feature code

---

### Authentication Flow (Vue 3)

| Context | Method |
|---------|--------|
| Web (backoffice) | CSRF tokens from Rails session |
| Native mobile (POS) | Bearer token stored in Capacitor Preferences |

---

### Pinia Store Pattern (Vue 3)

```typescript
export const useProductStore = defineStore('products', () => {
  const items = ref<Product[]>([])
  const isLoading = ref(false)

  function setItems(data: Product[]) {
    items.value = data
  }

  return { items, isLoading, setItems }
})
```

- Setup function style — not options API
- Granular `ref()` per variable, not one big reactive object

---

### Shared Composables (`@repo/composables`, Vue 3)

Common composables available across apps:
- `useHandleAxiosError` — standardised error handling
- `useGlobalSiteFeature` — feature flag checks
- Add new cross-app composables here — not duplicated per app
