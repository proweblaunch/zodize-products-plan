# Spacing & Layout

Zodize products favor information density over decorative whitespace (see
[Brand Standards](./brand-standards.md)). Every layout MUST be built from the
spacing scale and grid defined here; arbitrary pixel values in application
code are a defect.

## Base Spacing Scale

The scale is built on a **4px base unit**, with named tokens for the values
actually used in product UI. Do not interpolate values between steps.

| Token | Value | Typical usage |
|---|---|---|
| `--zdz-space-0` | 0px | Reset |
| `--zdz-space-1` | 4px | Icon-to-label gap, tight inline spacing |
| `--zdz-space-2` | 8px | Default gap between related inline elements, input internal padding (vertical) |
| `--zdz-space-3` | 12px | Input internal padding (horizontal), compact list item padding |
| `--zdz-space-4` | 16px | Default gap between form fields, card internal padding (compact) |
| `--zdz-space-5` | 20px | Card internal padding (default) |
| `--zdz-space-6` | 24px | Gap between cards/widgets, section internal padding |
| `--zdz-space-8` | 32px | Gap between major page sections |
| `--zdz-space-10` | 40px | Page top padding on dashboards |
| `--zdz-space-12` | 48px | Marketing section padding (mobile) |
| `--zdz-space-16` | 64px | Marketing section padding (desktop) |
| `--zdz-space-20` | 80px | Marketing hero vertical padding |
| `--zdz-space-24` | 96px | Marketing section separation (large) |

Numbering follows the token's pixel value ÷ 4 (i.e. `--zdz-space-6` = 24px),
so gaps in the scale (e.g. no `-7`) are intentional — do not add
intermediate tokens; compose from adjacent ones instead.

### Application Rules

- Dashboards and operational UI (the default context) use `--zdz-space-2`
  through `--zdz-space-8` almost exclusively. Values above
  `--zdz-space-10` are reserved for marketing pages.
- Component internal padding uses the `-2` through `-5` range; see
  [Components](./components.md) for the exact padding of each component
  size variant.
- Gaps in a flex/grid layout MUST use a spacing token via `gap`, never
  margin-based spacing hacks (no "owl selector" margin resets).

## Grid System

Zodize uses a **12-column grid** for all application and marketing layouts.

| Property | Value |
|---|---|
| Columns | 12 |
| Gutter (column gap) | `--zdz-space-6` (24px) on `lg` and above; `--zdz-space-4` (16px) on `md` and below |
| Margin (edge padding) | See breakpoint table below |

Dashboard layouts commonly span widgets across 3, 4, 6, or 12 columns (e.g.
a 4-up KPI row is four 3-column cards; a primary chart + side panel layout
is 8 columns + 4 columns). Column spans MUST always be expressed as a
multiple of the 12-column grid — no fractional or pixel-based widget widths.

## Breakpoints

| Name | Min width | Typical device | Grid margin | Columns typically visible |
|---|---|---|---|---|
| `xs` | 0px | Small mobile | 16px | Single column, grid collapses |
| `sm` | 640px | Large mobile | 20px | Single column, grid collapses |
| `md` | 768px | Tablet | 24px | Full 12-column grid begins |
| `lg` | 1024px | Small laptop | 32px | Full 12-column grid |
| `xl` | 1280px | Desktop (primary target for dashboards) | 40px | Full 12-column grid, sidebar + content |
| `2xl` | 1536px | Large desktop / ultrawide | 48px | Full 12-column grid, content max-width caps further growth |

Breakpoints are `min-width` based (mobile-up in CSS mechanics) regardless of
whether a given page is mobile-first or desktop-first in design priority —
see [Responsive Standards](./responsive-standards.md) for that distinction.
`xl` (1280px) is the primary design target for every dashboard and admin
screen: designs are authored at `xl` first, then adapted down.

## Container Max-Widths

| Context | Max width | Notes |
|---|---|---|
| Dashboard / application content area | `2560px` soft cap via `2xl` container, but content typically fills available width inside the app shell | Application shells are fluid within the sidebar/content split, not centered-and-capped like marketing pages |
| Marketing page content | `1280px` | Centered, with the breakpoint margins above applied outside the max-width |
| Marketing narrow content (article/legal text) | `720px` | Long-form reading measure, centered |
| Modal (default) | `560px` | See [Components](./components.md) and [`../standards/modal-standards.md`](../standards/modal-standards.md) |
| Modal (large) | `800px` | Multi-step or data-heavy modals |

## Page Margin Rules

- The application shell (sidebar + top bar + content) uses the breakpoint
  margin values above as the **content area's inner padding**, not as
  margin around the shell itself — the shell always spans the full
  viewport width.
- Cards and panels within the content area are separated by
  `--zdz-space-6` (24px) vertically and horizontally at `lg` and above,
  dropping to `--zdz-space-4` (16px) at `md` and below.
- The sidebar (see [Responsive Standards](./responsive-standards.md) for its
  collapse behavior) has a fixed width of `260px` expanded and `64px`
  collapsed-to-icons; it is not part of the 12-column content grid.
- Marketing pages apply the breakpoint margin as true outer margin around
  the centered, max-width-capped content container.
- Vertical rhythm between stacked sections on a page MUST use a spacing
  token, never an ad hoc value — page-level section separation defaults to
  `--zdz-space-8` in dashboards and `--zdz-space-16`/`--zdz-space-20` in
  marketing contexts.

## Border Radius Scale

Radius is part of the layout system because it interacts directly with
spacing (padding must accommodate radius without optical crowding).

| Token | Value | Usage |
|---|---|---|
| `--zdz-radius-sm` | 4px | Badges, tags, small chips |
| `--zdz-radius-md` | 6px | Buttons, inputs, default controls |
| `--zdz-radius-lg` | 10px | Cards, panels |
| `--zdz-radius-xl` | 14px | Modals, large containers |
| `--zdz-radius-full` | 9999px | Avatars, pills, status dots |

Radius tokens are catalogued alongside spacing in
[Design Tokens](./design-tokens.md).
