# Components

This document defines the core, atomic component inventory shared by every
Zodize product: the visual and behavioral contract for buttons, inputs,
selects, badges, avatars, tooltips, tabs, breadcrumbs, pagination, empty
states, and skeleton loaders. Higher-level composite patterns — tables,
modals, forms as a whole, navigation shells, notifications, dashboards — are
governed by [`../standards/`](../standards) (see cross-references at the end
of each section below) and are out of scope here.

Every component MUST implement the full state matrix and accessibility
requirements listed for it. A component missing a required state (e.g. no
visible focus ring, no loading state on an async button) is not production
ready and fails the checklists in [`../checklists/`](../checklists).

Sizing, radius, and spacing values reference [Spacing & Layout](./spacing-layout.md)
and [Design Tokens](./design-tokens.md). Color values reference
[Color System](./color-system.md). Motion values reference
[Motion & Animation](./motion-animation.md).

## Buttons

### Variants

| Variant | Background | Text | Usage |
|---|---|---|---|
| `primary` | `--color-accent-default` | `--color-text-inverse` | One per screen/section — the single primary action |
| `secondary` | `--color-surface-3` | `--color-text-primary` | Secondary actions alongside a primary |
| `outline` | Transparent, `--color-border-default` border | `--color-text-primary` | Tertiary actions, toolbars |
| `ghost` | Transparent | `--color-text-secondary` | Low-emphasis actions, icon-only toolbar buttons |
| `danger` | `--color-danger` | `--color-text-inverse` | Destructive actions only (delete, revoke, terminate) |

### Sizes

| Size | Height | Horizontal padding | Text | Icon size |
|---|---|---|---|---|
| `sm` | 32px | `--zdz-space-3` (12px) | `--zdz-text-body-sm` | 16px |
| `md` (default) | 40px | `--zdz-space-4` (16px) | `--zdz-text-body-sm` | 18px |
| `lg` | 48px | `--zdz-space-5` (20px) | `--zdz-text-body` | 20px |

### States

| State | Requirement |
|---|---|
| Default | As specified per variant above |
| Hover | Background shifts to the `-hover` token (accent) or `--color-surface-4` (secondary/outline/ghost); cursor `pointer` |
| Focus | 2px `--color-border-focus` ring with 2px offset, visible via `:focus-visible` only (see [Accessibility](./accessibility.md)) |
| Active/pressed | Background shifts to the `-active` token; no scale/transform effect (see [Motion & Animation](./motion-animation.md) on restrained interaction) |
| Disabled | 40% opacity, `cursor: not-allowed`, no hover/active response, `aria-disabled="true"` |
| Loading | Label replaced by a spinner sized to the button's icon size; button retains its width (no layout shift); `aria-busy="true"`; button is not interactive while loading |

Icon-only buttons MUST include an `aria-label` describing the action and
MUST use the `ghost` or `outline` variant at a minimum touch target of 40×40px
regardless of visual size, per [Accessibility](./accessibility.md).

## Inputs (text, number, textarea)

| Property | Value |
|---|---|
| Height (single-line) | 40px (`md`), 32px (`sm`, used only in dense filter bars) |
| Padding | `--zdz-space-3` horizontal, vertical centered |
| Border | 1px `--color-border-default`, `--zdz-radius-md` |
| Background | `--color-surface-1` |
| Text | `--zdz-text-body-sm` |

### States

| State | Requirement |
|---|---|
| Default | As above |
| Hover | Border shifts to `--color-neutral-500` |
| Focus | Border shifts to `--color-border-focus`; 2px focus ring per [Accessibility](./accessibility.md) |
| Filled | No visual distinction beyond content; placeholder disappears |
| Disabled | 40% opacity, `--color-surface-0` background, `cursor: not-allowed` |
| Error | Border and helper text use `--color-danger`; an `aria-describedby` MUST point to the error message; `aria-invalid="true"` |
| Read-only | Border becomes `--color-border-subtle`, no focus ring on click (still focusable for copy) |

Field-level and form-level accessibility (label association, required-field
convention, error summary) is governed by
[`../standards/form-standards.md`](../standards/form-standards.md); this
section defines only the input component's own visual/state contract.

## Selects (single and multi)

Selects share the input component's box model (height, border, radius,
states table above) with these additions:

- A chevron-down icon (16px, `--color-text-tertiary`) is right-aligned,
  16px from the edge.
- The open dropdown panel uses `--color-surface-3`, `--zdz-shadow-md`,
  `--zdz-z-dropdown`, and MUST NOT exceed 320px height before scrolling
  internally.
- Multi-select selections render as removable chips inside the control once
  1 or more values are selected; beyond 3 chips, collapse to "N selected."
- Keyboard: arrow keys move the active option, `Enter`/`Space` selects,
  `Escape` closes without changing selection, typing filters/jumps to a
  matching option. The listbox uses `role="listbox"` / `role="option"` per
  [Accessibility](./accessibility.md).

## Badges

| Property | Value |
|---|---|
| Height | 20px |
| Padding | `--zdz-space-2` horizontal |
| Text | `--zdz-text-caption`, uppercase, weight 500 |
| Radius | `--zdz-radius-sm` |

Badge color follows the semantic tokens in [Color System](./color-system.md):
`neutral` (default status), `success`, `warning`, `danger`, `info`, or
`accent` (for product-specific labels, e.g. "New"). Badges are
non-interactive by default; an interactive/removable badge (used for filter
chips) adds an 12px close icon with its own 24×24px hit target and an
`aria-label="Remove <value>"`.

## Avatars

| Size | Dimension | Usage |
|---|---|---|
| `xs` | 20px | Inline mentions, dense table rows |
| `sm` | 24px | Table rows, comment threads |
| `md` (default) | 32px | Nav bar, list items |
| `lg` | 40px | Profile headers, card headers |
| `xl` | 64px | Full profile pages |

Avatars are circular (`--zdz-radius-full`). Fallback for missing images is
initials (1–2 characters, uppercase) on a deterministic background color
selected from the chart categorical palette (see
[Color System](./color-system.md)) keyed by user ID, never a random color
per render. Avatars representing a system/automation actor use a fixed
neutral icon glyph rather than initials.

## Tooltips

- Trigger: hover (200ms delay in) or keyboard focus (no delay). Dismiss on
  mouse-out, blur, or `Escape`.
- Background `--color-surface-4`, `--zdz-shadow-sm`, `--zdz-radius-md`,
  max-width 240px, `--zdz-text-small`.
- `role="tooltip"` with the trigger referencing it via `aria-describedby`.
  Tooltips MUST NOT contain interactive content or be the only means of
  conveying essential information — critical info must also exist in
  visible UI or an accessible label per [Accessibility](./accessibility.md).

## Tabs

- Underline style: active tab has a 2px bottom border in
  `--color-accent-default`; inactive tabs use `--color-text-secondary`.
- Tab list uses `role="tablist"`, each tab `role="tab"` with
  `aria-selected`, panels `role="tabpanel"`. Arrow-key navigation moves
  focus between tabs; `Tab` key moves focus into the active panel.
- Tabs MUST NOT be used for more than 6 items in the primary horizontal
  form; beyond that, use a select or side navigation instead.

## Breadcrumbs

- `--zdz-text-body-sm`, `--color-text-tertiary` for all but the final
  (current page) crumb, which uses `--color-text-primary` and is not a link.
- Separator is a 14px chevron icon, not a literal "/" character.
- Rendered as an ordered navigation landmark: `<nav aria-label="Breadcrumb">`
  wrapping an `<ol>`.
- On overflow (deep hierarchy), collapse middle items behind an ellipsis
  button that expands the full path on click/focus.

## Pagination

- Default pattern is page-number pagination for tables (see
  [`../standards/table-standards.md`](../standards/table-standards.md) for
  when to use cursor/infinite-scroll instead).
- Buttons follow the `ghost` button variant at `sm` size; the active page
  uses `--color-accent-subtle` background and `--color-accent-text`.
- Previous/Next controls are always present and disable (per the Button
  disabled state) at the first/last page rather than disappearing.
- The control announces the current state via a visually-hidden live region
  ("Page 3 of 12") for screen reader users, per
  [Accessibility](./accessibility.md).

## Empty States

Every list, table, or dashboard widget that can be empty MUST define an
explicit empty state — a blank white/dark space is a defect.

- Structure: a single-color line-style illustration (see
  [Icons & Illustrations](./icons-illustrations.md)), a one-line title
  (`--zdz-text-h5`), an optional one-line description
  (`--zdz-text-body-sm`, `--color-text-tertiary`), and, where a next action
  exists, a single primary or secondary button.
- Empty states MUST distinguish "no data exists yet" from "no data matches
  your filter" — the latter includes a "Clear filters" action rather than a
  generic "Create new" CTA.

## Skeleton Loaders

- Skeletons use `--color-surface-2` base with a subtle shimmer sweep to
  `--color-surface-3`, animated per the "standard" duration in
  [Motion & Animation](./motion-animation.md), and MUST respect
  `prefers-reduced-motion` by falling back to a static pulse-free block.
- Skeleton shapes MUST mirror the real content's layout (text-line widths,
  avatar circles, card blocks) so the transition to loaded content does not
  cause a layout shift.
- Skeletons carry `aria-busy="true"` on their container and are hidden from
  the accessibility tree (`aria-hidden="true"` on the skeleton nodes
  themselves) so screen readers announce loading state once, not per
  placeholder shape.
- Skeletons MUST be replaced by real content or an explicit error/empty
  state within a bounded time; a skeleton that can spin indefinitely on
  failure is a defect — pair every skeleton with an error-state fallback.

## Related Standards

Composite and higher-level patterns built from these primitives are
specified in [`../standards/`](../standards):

- [`../standards/table-standards.md`](../standards/table-standards.md) — data tables, sorting, filtering, row actions
- [`../standards/modal-standards.md`](../standards/modal-standards.md) — modals, drawers, dialogs
- [`../standards/form-standards.md`](../standards/form-standards.md) — form layout, validation, multi-step forms
- [`../standards/navigation-standards.md`](../standards/navigation-standards.md) — sidebar, top nav, app switcher
- [`../standards/notification-standards.md`](../standards/notification-standards.md) — toasts, in-app notifications, alerts
- [`../standards/dashboard-standards.md`](../standards/dashboard-standards.md) — widget layout, KPI cards, chart composition
