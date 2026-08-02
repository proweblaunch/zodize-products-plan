# Coding Standards — Frontend (Blade / Bootstrap / jQuery)

## Baseline

- Server-rendered Blade templates are the default. There is no SPA/client
  router and no client-side framework (React/Vue/etc.) anywhere in the
  stack — this is the classic ViserLab pattern already used by
  qfsfountains, Pay Secure (ZodiBank), and Ultimate POS (ZodiCore): a
  reusable Laravel core with a consistent admin panel, settings, payment-
  gateway, and helper structure across products.
- Bootstrap 5 is the CSS/component framework. jQuery is the interactivity
  layer (AJAX calls, DataTables, modals, form handlers, Select2, etc.) —
  the same libraries already present in every base codebase.
- Font Awesome for iconography.
- Zodize's own design tokens/theme (colors, typography — see
  [design-tokens.md](../design-system/design-tokens.md)) apply on top of
  this stack as a Bootstrap 5 theme layer (custom SCSS variables/overrides
  and a shared `zodize-theme.css`), not as a component library replacing
  Bootstrap.
- Node + NPM compile frontend assets (SCSS → CSS, JS bundling/minification)
  via each base codebase's existing Laravel Mix or Vite config — whichever
  the base already uses; do not introduce a new build tool.
- Linting/formatting: PHP-CS-Fixer/Pint for Blade-adjacent PHP, ESLint for
  any plain JS, enforced in CI; no style debates in review.

## Template structure

- One Blade file per view/partial, snake_case or kebab-case filenames
  matching the base codebase's existing convention (check the product's
  base before introducing a new naming style).
- Views are organized as: `layouts/` (shared page shells — header, footer,
  sidebar), `partials/` (reusable fragments included across views), and
  per-module view directories mirroring the module's own namespace (e.g.
  `resources/views/admin/banking/` for a banking module's admin screens).
- A Blade file over ~200 lines is a signal to extract a `@include`d
  partial or a Blade component — not a hard rule, a smell to justify.
- Shared UI (buttons, cards, status badges, empty states) are Blade
  components (`<x-component-name>`) under `resources/views/components/`
  when a base codebase already has this pattern; otherwise match whatever
  the base's own reusable-partial convention already is.

## JavaScript

- Shared client-side logic (AJAX calls, table filters, form handlers) is
  extracted into named functions in a page- or module-scoped `.js` file
  under `public/js/` or `resources/js/`, not inlined per-page unless it's
  a handful of lines specific to that one view.
- Every AJAX call handles loading/error/success states consistently (a
  spinner or disabled-button state while in flight, a toast/alert on
  error) — never a silent failure.
- DataTables (or the base codebase's existing table library) is the
  default for any list view over ~20 rows, giving pagination/search/sort
  for free instead of hand-rolling it.

## State and data

- No client-side global store. Session/user context lives server-side
  (Laravel session, the authenticated `Auth::user()`/`Auth::guard()`
  instance) and is read directly in Blade/controllers — there is no
  client-side equivalent to keep in sync.
- Cross-request UI state (e.g. a wizard's in-progress steps) uses the
  Laravel session or a database-backed draft record, not client storage,
  unless the base codebase already has an established pattern for it.

## Accessibility requirements (enforced, not optional)

Every interactive element must meet
[accessibility.md](../design-system/accessibility.md): correct semantic
element or `role`, full keyboard operability, visible focus state, and
`aria-*` attributes matching element state (`aria-expanded`,
`aria-selected`, `aria-busy` while an AJAX call is in flight, etc.) —
Bootstrap's own components (modals, dropdowns, tabs) already handle most
of this correctly out of the box; don't override their built-in ARIA
wiring without a reason.

## Forms

Forms use Laravel's built-in validation (`$request->validate()` /
FormRequest classes) with Blade's `@error` directive rendering field-level
errors matching [form-standards.md](../standards/form-standards.md):
server-side validation on submit, with jQuery-based inline validation
(e.g. jQuery Validate, or the base codebase's existing pattern) as a
progressive-enhancement layer, not a replacement for server-side checks.

## Performance

- Paginate any list over the row-count budget in
  [performance-standards.md](../quality/performance-standards.md) using
  Laravel's built-in paginator rather than loading a full table client-side.
- Images use responsive `srcset`/lazy loading (`loading="lazy"`) by default.
- Avoid N+1 queries in any view — eager-load relationships used in a loop.

## Testing requirement

- Feature/HTTP tests (Pest/PHPUnit) covering controller behavior for every
  CRUD flow — the equivalent of component tests in this stack is testing
  the controller and the rendered view's key assertions (status code,
  visible text, DB state after submit), not isolated JS unit tests.
- Critical user flows (auth, checkout/billing, primary CRUD flows per
  product) have Playwright end-to-end coverage, per
  [testing-standards.md](./testing-standards.md), and per protocol rule 8
  in [`BUILD_STATE.md`](../../BUILD_STATE.md) — live-browser verification
  with a screenshot before calling any visible-surface feature "done."

## Internationalization

All user-facing strings go through Laravel's `__()`/`@lang()` translation
helpers and `resources/lang/` files — no hardcoded UI strings — per
[localization-i18n.md](../standards/localization-i18n.md).
