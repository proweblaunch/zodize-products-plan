# Design Tokens

This document is the consolidated reference for every design token used
across the Zodize design system, and the single place that defines how a
token's name maps to its implementation in the Laravel + Blade/Bootstrap
stack. Every
other document in `docs/design-system/` defines the *values*; this document
defines the **naming convention and the delivery mechanism**.

## Naming Convention

All Zodize design tokens use the `--zdz-` prefix, followed by a category,
followed by a scale step or semantic name:

```
--zdz-<category>-<name>[-<variant>]
```

Examples: `--zdz-color-surface-2`, `--zdz-space-6`, `--zdz-radius-md`,
`--zdz-shadow-modal`, `--zdz-text-h1`, `--zdz-z-modal`.

Category names are fixed: `color`, `space`, `radius`, `shadow`, `text`
(typography), `z` (z-index), `duration`, `ease` (the last two defined in
[Motion & Animation](./motion-animation.md)). A new category MUST NOT be
introduced without an ADR, per [`CONTRIBUTING.md`](../../CONTRIBUTING.md).

Product-specific accent tokens follow the same convention with an `accent`
name rather than a numbered scale step (e.g. `--zdz-color-accent-default`),
so that theming a product to its assigned hue (see
[Brand Standards](./brand-standards.md)) requires overriding only the
`--zdz-color-accent-*` group, never the neutral or semantic groups.

## Token Categories

### Color

Full value tables live in [Color System](./color-system.md). Token shape:

```json
{
  "color": {
    "surface": { "0": "#0B0D12", "1": "#12141C", "2": "#1A1D27", "3": "#242733", "4": "#2E323F" },
    "neutral": { "0": "#0B0D12", "50": "#12141C", "100": "#1A1D27", "200": "#242733", "300": "#2E323F", "400": "#3D4150", "500": "#565B6D", "600": "#767C90", "700": "#9CA1B5", "800": "#C7CAD9", "900": "#F1F2F7" },
    "accent": { "default": "var(--product-accent, #6366F1)", "hover": "...", "active": "...", "subtle": "...", "text": "..." },
    "success": "#22C55E", "warning": "#F59E0B", "danger": "#EF4444", "info": "#3B82F6",
    "text": { "primary": "...", "secondary": "...", "tertiary": "...", "disabled": "...", "inverse": "..." },
    "border": { "subtle": "...", "default": "...", "focus": "..." }
  }
}
```

### Spacing

Full value table lives in [Spacing & Layout](./spacing-layout.md). Token
shape: `{"space": {"1": "4px", "2": "8px", "3": "12px", "4": "16px", "5": "20px", "6": "24px", "8": "32px", "10": "40px", "12": "48px", "16": "64px", "20": "80px", "24": "96px"}}`

### Radius

Full value table lives in [Spacing & Layout](./spacing-layout.md). Token
shape: `{"radius": {"sm": "4px", "md": "6px", "lg": "10px", "xl": "14px", "full": "9999px"}}`

### Shadow

Shadows are used only for true overlay elevation (popovers, dropdowns,
modals) — dark theme's default surfaces are separated by lightness, not
shadow (see [Color System](./color-system.md) surface elevation table).

| Token | Value (dark theme) | Value (light theme) | Usage |
|---|---|---|---|
| `--zdz-shadow-sm` | `0 1px 2px rgba(0,0,0,0.4)` | `0 1px 2px rgba(16,24,40,0.06)` | Dropdown, small popover |
| `--zdz-shadow-md` | `0 4px 12px rgba(0,0,0,0.45)` | `0 4px 12px rgba(16,24,40,0.08)` | Popover, tooltip |
| `--zdz-shadow-lg` | `0 12px 32px rgba(0,0,0,0.5)` | `0 12px 32px rgba(16,24,40,0.12)` | Modal, dialog |
| `--zdz-shadow-focus` | `0 0 0 3px rgba(99,102,241,0.5)` (adjusted to product accent) | Same | Focus ring shadow companion, see [Accessibility](./accessibility.md) |

### Typography

Full value table lives in [Typography](./typography.md). Token shape:
`{"text": {"display": {"size": "3rem", "lineHeight": "1.1", "letterSpacing": "-0.02em", "weight": 700}, "h1": {...}, ...}}`

### Z-Index

A fixed, small stacking scale prevents the "z-index arms race" common in
large applications. Every stacking context in a Zodize product MUST use one
of these tokens — never an arbitrary integer.

| Token | Value | Usage |
|---|---|---|
| `--zdz-z-base` | 0 | Default document flow |
| `--zdz-z-sticky` | 10 | Sticky table headers, sticky page headers |
| `--zdz-z-dropdown` | 20 | Dropdowns, select menus, autocomplete panels |
| `--zdz-z-fixed` | 30 | Fixed sidebar, fixed top nav |
| `--zdz-z-overlay` | 40 | Modal/drawer backdrop |
| `--zdz-z-modal` | 50 | Modal, drawer, dialog content |
| `--zdz-z-popover` | 60 | Popover/tooltip rendered above a modal (e.g. a select inside a modal) |
| `--zdz-z-toast` | 70 | Toast/notification stack — always above everything else |

### Duration & Easing

Defined in full in [Motion & Animation](./motion-animation.md); included in
the same `--zdz-` token family (`--zdz-duration-*`, `--zdz-ease-*`) so that
motion values are consumable through the same pipeline as every other
token.

## Mapping to Bootstrap SCSS Variables / CSS Variables in Laravel + Blade

Zodize products are built on Laravel (backend) with Blade views styled by
Bootstrap 5 and jQuery for interactivity. Tokens flow through a single
pipeline so that a change to a value in this handbook has exactly one place
to update in code:

1. **Source of truth in code**: tokens are defined once as CSS custom
   properties in `resources/css/tokens.css` (or the shared design-system
   package consumed by every product), scoped to `:root` for defaults and
   overridden per-product via a `[data-product="zodibank"]` (etc.) attribute
   for the accent group only:

   ```css
   :root {
     --zdz-color-surface-0: #0B0D12;
     --zdz-color-surface-1: #12141C;
     --zdz-color-accent-default: #6366F1; /* platform default */
     --zdz-space-6: 24px;
     --zdz-radius-md: 6px;
     --zdz-z-modal: 50;
   }
   [data-product="zodibank"] {
     --zdz-color-accent-default: #2563EB; /* deep blue, from SPEC.md */
   }
   ```

2. **Bootstrap SCSS variable overrides**: a product's `resources/sass/_zodize-theme.scss`
   (compiled via the base codebase's existing Laravel Mix/Vite pipeline)
   maps Bootstrap's own SCSS variables onto the `--zdz-*` custom
   properties before Bootstrap's own `_variables.scss`/`_maps.scss` are
   imported, never by hardcoding hex/px values, so Bootstrap's component
   classes and raw CSS stay perfectly in sync:

   ```scss
   // resources/sass/_zodize-theme.scss
   $primary:    var(--zdz-color-accent-default);
   $body-bg:    var(--zdz-color-surface-0);
   $border-radius: var(--zdz-radius-md);
   $spacer:     var(--zdz-space-6);
   $zindex-modal: var(--zdz-z-modal);

   @import "~bootstrap/scss/bootstrap";
   ```

3. **Blade views/components** consume tokens exclusively through Bootstrap
   utility/component classes (`bg-body`, `rounded`, `btn-primary`) driven by
   the SCSS variable mapping above, or, where no Bootstrap utility exists
   for the property, directly via `var(--zdz-*)` in a scoped `<style>`
   block or a small component-specific SCSS partial. Raw hex values, raw px
   values, and raw integer z-indexes MUST NOT appear in view/component code
   — this is enforced in code review and MAY be enforced via a stylelint
   rule (`declaration-property-value-disallowed-list`) in each product's CI
   pipeline; see [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md).
4. **Light/dark theme switching** is handled by swapping the `--zdz-color-*`
   variable block based on a `data-theme="light"` attribute on `<html>`,
   never by duplicating component logic — see [Dark Theme](./dark-theme.md)
   for the full switching mechanism.
5. **Product theming** is handled by the `data-product` attribute shown
   above, set once at the application shell root from the deployment's own
   product identity (fixed per deployment, since each deployment is one
   product for one buyer — see
   [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)),
   overriding only the `accent` token group. No other token group is ever
   overridden per product.

## Open Questions

- Whether tokens will eventually be generated from a single JSON/YAML
  source via Style Dictionary (rather than hand-maintained CSS) is not yet
  decided. If adopted, it changes the build pipeline but not the token
  names or values defined in this handbook, and does not require an ADR
  unless it changes a value.
