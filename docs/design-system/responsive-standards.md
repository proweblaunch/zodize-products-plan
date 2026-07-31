# Responsive Standards

## Mobile-First vs. Desktop-First: An Explicit Decision

Zodize products fall into two distinct design contexts with **opposite**
responsive priorities. This is a deliberate, explicit decision, not an
inconsistency to be reconciled:

- **Dashboards, admin surfaces, and operational product UI are
  desktop-first.** Zodize products are professional tools — banking
  operations, trading desks, clinical workflows, fleet management — used
  primarily on desktop and laptop monitors during a workday. These screens
  are designed at the `xl` (1280px) breakpoint first, per
  [Spacing & Layout](./spacing-layout.md), and then adapted downward for
  `lg`/`md`/`sm`. A dashboard is never designed mobile-first and stretched
  up; the dense, multi-panel layouts these products require do not emerge
  naturally from a mobile layout scaled outward.
- **Marketing pages (product landing pages, the Zodize corporate site,
  pricing pages, documentation) are mobile-first.** These are
  discovery/evaluation surfaces visited from any device, frequently first
  encountered on mobile via search or social. They are designed at the
  `xs`/`sm` breakpoint first and scale up, per the responsive typography
  rules in [Typography](./typography.md).

Every screen in a Zodize product's `docs/products/<product>/SPEC.md` MUST
be classified as one or the other; there is no third "adaptive, no
priority" category. When in doubt: if the screen requires the user to be
authenticated to see it, it is desktop-first; if it is reachable
unauthenticated, it is mobile-first.

## Breakpoints

The full breakpoint table (widths, margins, grid behavior) is defined in
[Spacing & Layout](./spacing-layout.md) and is shared by both contexts —
only the design priority direction differs, not the pixel values.

## Desktop-First Responsive Patterns (Dashboards & Admin)

### Sidebar Navigation

| Breakpoint | Behavior |
|---|---|
| `xl` / `2xl` | Sidebar expanded (260px), always visible |
| `lg` | Sidebar collapses to icon-only (64px) by default; expandable via toggle, overlaying content rather than pushing it when expanded |
| `md` | Sidebar hidden behind a hamburger trigger in the top bar; opens as an overlay drawer (`--zdz-z-fixed`, full height, `--zdz-duration-page` slide-in per [Motion & Animation](./motion-animation.md)) |
| `sm` / `xs` | Same as `md`; additionally, the top bar itself compresses to icon-only actions with labels moved into an overflow menu |

### Data Tables

| Breakpoint | Behavior |
|---|---|
| `lg` and above | Full table, all columns visible, horizontal scroll only if the table's true content width exceeds the viewport (e.g. a 20-column ledger) |
| `md` | Table drops lowest-priority columns (defined per table in the product's SPEC.md column-priority list) rather than shrinking all columns proportionally; a "more columns hidden" indicator is shown |
| `sm` / `xs` | Table converts to a stacked card-per-row layout: the row's primary identifier becomes the card title, remaining fields render as label/value pairs beneath it. Full interaction requirements for this conversion are defined in [`../standards/table-standards.md`](../standards/table-standards.md) |

Tables never simply shrink font size to fit more columns on small screens —
this violates the minimum text size requirement in
[Accessibility](./accessibility.md) and produces illegible data-dense
UI. Column dropping and card-stacking are the only sanctioned strategies.

### Dashboard Widget Grids

| Breakpoint | Columns used |
|---|---|
| `xl` / `2xl` | Full 12-column grid; widgets typically span 3, 4, 6, or 12 columns |
| `lg` | 12-column grid retained; wide widgets (12-col) unaffected, narrow widgets (3-col) reflow to 6-col in pairs |
| `md` | Grid collapses to a single logical column-pair; widgets stack 2-wide at most |
| `sm` / `xs` | Full single-column stack, in the priority order defined by the dashboard's SPEC.md ("most operationally urgent widget first") |

### Filters & Toolbars

Dense filter bars (multi-select dropdowns, date range pickers, search) that
fit on one row at `lg` and above collapse into a single "Filters" button at
`md` and below, opening the full filter set in a bottom sheet/drawer rather
than wrapping to multiple cramped rows.

## Mobile-First Responsive Patterns (Marketing)

- Hero sections stack (headline, subhead, CTA, then supporting visual)
  vertically at `xs`/`sm`, moving to a side-by-side layout at `md` and
  above.
- Navigation is a hamburger-triggered full-screen overlay below `md`,
  becoming a horizontal nav bar at `md` and above.
- Multi-column feature/pricing sections (3–4 columns at `lg`+) stack to a
  single column below `md`, with no column ever dropped — marketing content
  is comprehensive at every breakpoint, unlike the column-dropping
  permitted for operational tables above.
- Long-form content (docs, legal pages) caps at the 720px reading measure
  defined in [Spacing & Layout](./spacing-layout.md) at every breakpoint
  above `sm`.

## Touch Targets

Regardless of context, every interactive element MUST maintain a minimum
44×44px touch target on any breakpoint below `md` (where touch input is the
primary or likely input method), even if its visual size is smaller — using
padding/hit-area expansion rather than enlarging the visible control. Full
requirement and rationale in [Accessibility](./accessibility.md).

## Orientation & Foldables

Zodize dashboards are not designed for phone-width operational use (a
banking operator does not run ZodiBank's back office on a phone), so no
dedicated phone-optimized dashboard layout is required beyond the
`sm`/`xs` stacking behavior above providing a functional, if not primary,
fallback. Marketing pages MUST be fully functional in both portrait and
landscape orientation at all breakpoints.
