# Coding Standards — Frontend (Blade, Bootstrap 5, jQuery)

> Supersedes the retired `coding-standards-vue.md`. Zodize products are
> **not** Vue/SPA applications — see
> [`../architecture/overview.md`](../architecture/overview.md) and
> [`../standards/frontend-standard.md`](../standards/frontend-standard.md).
> The real, confirmed stack (verified directly against the qfsfountains
> base, Pay Secure/ZodiBank, and Ultimate POS/ZodiCore codebases on the
> build server) is the classic ViserLab pattern: **Laravel (latest, PHP
> 8.x) + Blade templates + Bootstrap 5 + jQuery**, not a Vue/TypeScript SPA
> layer.

## Baseline

- Server-rendered Blade views, not a client-side SPA framework. Laravel
  controllers return Blade views (or Blade partials for AJAX-refreshed
  fragments); there is no separate Vue/React build producing the primary
  UI.
- **Bootstrap 5** is the CSS framework for structure, grid, and components
  (modals, dropdowns, tabs, cards, forms) across every admin panel and
  customer-facing portal, matching the existing qfsfountains/Pay
  Secure/Ultimate POS admin conventions.
- **jQuery** is the frontend interactivity layer — AJAX form submission,
  DataTables-driven listing tables, Select2 dropdowns, modal wiring,
  client-side validation hooks — following the same pattern already used
  across the inherited base codebase's `assets/admin/js/` and
  `assets/js/` directories.
- **Font Awesome** is the icon set, used the same way across every
  product's admin sidebar, buttons, and status badges.
- Assets are compiled with **Node + NPM** (Laravel Mix or Vite, whichever
  the inherited base codebase already uses) and shipped pre-built as part
  of the source archive per
  [`../architecture/overview.md`](../architecture/overview.md) — the buyer
  never runs a Node build step.
- Zodize's own design tokens/theme (colors, typography sourced from
  zodize.com — see [`../design-system/`](../design-system/)) are applied
  **on top of** this Bootstrap 5 + jQuery stack as a styling layer (CSS
  custom properties overriding Bootstrap's Sass variables, plus a
  Zodize-branded component skin). This is a theming/styling pass, not a
  framework swap — no product replaces Bootstrap with Tailwind or replaces
  Blade+jQuery with a JS framework to adopt the Zodize look.

## File & component structure

- Blade views organized the same way as the inherited base: `views/admin/`
  for the back office, `views/` (or a portal-specific subfolder) for
  customer-facing pages, `views/components/` for reusable Blade
  components, `views/layouts/` for shared shells (sidebar, topbar,
  footer).
- Reusable UI fragments are Blade components (`<x-component-name>`), not
  Vue SFCs. A fragment used in 2+ places is extracted into
  `views/components/`, matching the existing `x-zodize.*` shared library
  convention documented in
  [`../standards/frontend-standard.md`](../standards/frontend-standard.md).
- Page-specific JS lives alongside the page's Blade view or in
  `assets/admin/js/<module>.js` / `assets/js/<module>.js`, loaded via the
  layout's script stack (`@push('scripts')` / `@stack('scripts')`), not
  bundled per-component.

## Forms & validation

- Server-side validation via Laravel Form Requests is authoritative.
  Client-side validation (jQuery Validate or equivalent, matching whatever
  the base codebase already uses) is a UX layer only, never a substitute
  for server-side checks.
- AJAX form submissions use jQuery's `$.ajax`/`$.post`, matching the
  existing base codebase pattern for admin CRUD screens (submit without a
  full page reload where the base already does this, otherwise a standard
  POST + redirect).

## Tables & listings

- Data-heavy listing tables use **DataTables** (jQuery plugin), matching
  the existing admin panel convention across the inherited base for
  server-side pagination, search, and sorting on large tables.

## Accessibility

Every interactive Blade component still meets
[`../design-system/accessibility.md`](../design-system/accessibility.md):
correct semantic element/`role`, full keyboard operability, visible focus
state, and `aria-*` attributes matching component state — Bootstrap 5's own
components already provide most of this correctly out of the box; don't
regress it with custom markup that drops the built-in `aria-*` wiring.

## Internationalization

All user-facing strings go through Laravel's own `__()`/`trans()` i18n
layer and Blade `@lang` directives — no hardcoded UI strings — per
[`../standards/localization-i18n.md`](../standards/localization-i18n.md).
This is unchanged from the retired Vue-based draft; only the rendering
layer (Blade vs. Vue templates) changes.
