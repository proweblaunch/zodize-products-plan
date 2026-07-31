# Coding Standards — Vue

## Baseline

- Vue 3, Composition API with `<script setup>` exclusively. Options API is
  not used in new code.
- TypeScript is required for all new components and modules — no `.vue`
  files with untyped `<script>`.
- State management: Pinia. No Vuex in new code.
- Routing: Vue Router with route-level code splitting (`defineAsyncComponent`
  / dynamic `import()`), lazy-loaded per module.
- Styling: Tailwind CSS utility classes bound to the design tokens in
  [design-tokens.md](../design-system/design-tokens.md) — never hardcoded
  hex colors or pixel values that bypass the token system.
- Linting/formatting: ESLint (Vue + TypeScript configs) and Prettier,
  enforced in CI; no style debates in review.

## Component structure

- One component per file, PascalCase filenames matching the component name.
- Components are organized as: `base/` (design-system primitives — buttons,
  inputs, cards, wrapping [components.md](../design-system/components.md)),
  `modules/<domain>/` (feature components scoped to one module), `layouts/`
  (page shells per [page-layout-standards.md](../standards/page-layout-standards.md)).
- A component over ~200 lines of template+script is a signal to extract a
  child component or a composable — not a hard rule, a smell to justify.
- Props are typed with `defineProps<T>()`; emits are typed with
  `defineEmits<T>()`. No untyped `props: ['value']` arrays.

## Composables

- Shared logic (data fetching, form state, permission checks) is extracted
  into `use*` composables under `composables/`, unit-testable in isolation
  from any component.
- Data-fetching composables wrap the API client (see
  [api-standards.md](./api-standards.md)) and expose `data`, `error`,
  `isLoading` consistently — every list/detail view in the product uses the
  same loading/error/empty pattern, never a bespoke one per screen.

## State management rules

- Local component state (`ref`/`reactive`) for anything not shared.
- Pinia store for: current user/session, current company/branch context on
  a product that models multi-company/multi-branch operation (see
  [localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)),
  permission set, feature flags, notification/toast queue, and any
  cross-route shared entity state.
- Server state (API data) is not duplicated into a global store unless it
  needs to be shared across unrelated routes; prefer a data-fetching
  composable with its own cache over stuffing API responses into Pinia.

## Accessibility requirements (enforced, not optional)

Every interactive component must meet
[accessibility.md](../design-system/accessibility.md): correct semantic
element or `role`, full keyboard operability, visible focus state, and
`aria-*` attributes matching component state (`aria-expanded`,
`aria-selected`, `aria-busy` while loading, etc.).

## Forms

All forms use the shared form composable/validation layer (schema-based
validation, e.g. Zod/Vuelidate-equivalent — pick one and use it consistently
across all products) matching [form-standards.md](../standards/form-standards.md):
inline validation on blur, submit-time full validation, and server-side
validation errors mapped back to the same field-level error slots as
client-side errors.

## Performance

- Route-level and heavy-component-level code splitting is mandatory for any
  bundle chunk over the budget in
  [performance-standards.md](../quality/performance-standards.md).
- Lists over 100 rows use virtualization (see
  [table-standards.md](../standards/table-standards.md)).
- Images use responsive `srcset`/lazy loading by default.

## Testing requirement

- Component tests with Vitest + Vue Testing Library for all `base/`
  components and any `modules/` component with non-trivial logic.
- Critical user flows (auth, checkout/billing, primary CRUD flows per
  product) have Playwright end-to-end coverage, per
  [testing-standards.md](./testing-standards.md).

## Internationalization

All user-facing strings go through the i18n layer (vue-i18n) — no hardcoded
UI strings — per [localization-i18n.md](../standards/localization-i18n.md).
