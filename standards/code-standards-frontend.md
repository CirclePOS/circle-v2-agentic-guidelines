# Code Standards — Frontend

## Hotwire (New Backoffice Pages)

**New and revamped backoffice pages use Hotwire — not Vue. POS stays on Vue (Capacitor dependency).**

| Scenario | Use |
|---|---|
| New backoffice page | Hotwire (ViewComponent + Stimulus + Turbo) |
| Existing Vue page — small fix | Fix in Vue, do not rewrite |
| Existing Vue page — significant rework | Migrate to Hotwire if scope allows |
| POS terminal | Vue 3 (stays on Vue) |

### Technology Stack

| Layer | Technology |
|---|---|
| UI components | ViewComponent — all inherit `BaseComponent`; logic in `.rb`, display only in `.html.erb` |
| JavaScript | Stimulus controllers in `app/javascript/controllers/<domain>/` |
| Partial updates | Turbo Frames + Turbo Streams |
| Styles | Tailwind CSS + DaisyUI (`data-theme="circle-design-system"`) |
| Layout | `backoffice_hotwire` layout (`app/views/layouts/backoffice_hotwire.html.erb`) |

### ViewComponent Rules
- Keyword-arg `initialize` with `attr_reader`
- `ComponentName.with_collection(records)` for collections — no manual loops in ERB
- Use `render?` for conditional rendering

### Stimulus Rules
- Filename determines `data-controller` id (path separators → `--`, underscores → `-`)
- Use `static targets` for DOM refs, `static values` for server data
- `this.dispatch()` for cross-controller events
- Arrow functions for event listener callbacks: `handleSubmitEnd = (event) => {}`

### Hotwire Controller Pattern
```ruby
render ComponentClass.new(...), layout: "backoffice_hotwire"
```

### Conventions
- Gate new pages behind `current_site.feature?(:name, :backoffice_hotwire_layout)` until QA signs off
- Name routes with `_hotwire` suffix until full Vue cutover: `as: :promotions_hotwire`

---

## Vue 3 Standards (POS + Legacy Backoffice Fixes)

### Component Rules
- `<script setup lang="ts">` on all components — Options API is legacy; do not use for new code
- Props: typed interface + `withDefaults(defineProps<Props>(), {...})`
- Use `defineModel()` for v-model bindings
- Form submission logic in a `use<Feature>FormSubmit` composable — not inline in the component

### TypeScript Rules
- No `any`, no `@ts-ignore` — use `unknown` with narrowing
- `interface` for object shapes, `type` for unions

### Feature Folder Requirements
Every feature folder requires:
- `queries.ts` — TanStack Query key factory + `useQuery`/`useMutation`
- `schema.ts` — Zod schema + `z.infer<>` type — **never write a separate interface for form data**

### Pinia
- Setup function style: `defineStore('id', () => { const x = ref(); return { x } })`
- Granular `ref()` per variable

### API Rules (`@repo/api`)
- Default export object with named methods
- `AxiosPromise<T>` return types
- Use `request` from `@repo/utils` — **never import axios directly in feature code**

### i18n
- Error messages via `t('validation.key')` — no hardcoded English strings

---

## Vue3 Monorepo Structure (`app/vue3/`)

### Apps
- `apps/backoffice` — Web backoffice (Element Plus, Vue Router, Pinia, TanStack Query, Vite)
- `apps/pos` — POS terminal; web + native mobile via Capacitor (iOS/Android)

### Shared Packages (referenced as `@repo/*`)
- `packages/api` — Per-domain axios API modules (e.g. `saleOrders/`, `accounts/`)
- `packages/components` — Shared Vue components
- `packages/composables` — Shared composables (`useHandleAxiosError`, `useGlobalSiteFeature`, etc.)
- `packages/lang` — vue-i18n translations
- `packages/utils` — Axios request service (`request.ts`), platform detection, session helpers

### API URL Config
- All API calls target `<VITE_API_URL>api/v1/`
- Development: `VITE_API_URL=http://localhost:3008/`
- Production/CI: `VITE_API_URL=/`

### Authentication
- Web: CSRF tokens from Rails session
- Native mobile: Bearer token stored in Capacitor Preferences

---

## Frontend Commands Reference

```bash
# From app/vue3/
npm run dev                   # all apps in parallel
npm run dev:backoffice        # backoffice only (:1233)
npm run dev:pos               # pos only (:5173)
npm run build                 # build all apps for production
npm run build:backoffice
npm run build:pos:mobile      # Capacitor production
npm run build:pos:mobile:dev  # Capacitor dev
npm run cap:sync              # sync Capacitor native projects after build
npx eslint .                  # lint
npx eslint . --fix            # auto-fix lint
npm run lint:ts               # TypeScript type-check all apps

# Type-check specific app
cd app/vue3/apps/backoffice && npx vue-tsc --noEmit
cd app/vue3/apps/pos && npx vue-tsc --noEmit
```
