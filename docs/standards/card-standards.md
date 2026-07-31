# Card Standards

Cards are the fundamental content container in every Zodize product —
dashboard widgets, list items in card view, summary panels on detail pages,
and pricing/plan displays are all built from the same base card component
defined here. Visual tokens (border radius, shadow, border color) come from
[`../design-system/components.md`](../design-system/components.md); this
document defines structure, variants, and interaction states.

## Anatomy

Every card is composed of up to four regions, all optional except body, in
this fixed vertical order:

```
┌───────────────────────────────────────────┐
│ Header  (title, optional subtitle, actions)│
├───────────────────────────────────────────┤
│ Body     (the card's primary content)      │
├───────────────────────────────────────────┤
│ Footer  (metadata, timestamp, link)        │
└───────────────────────────────────────────┘
     [Overflow actions menu, top-right of header]
```

- **Header**: contains a title (required if the header is present at all),
  an optional one-line subtitle/description, and an optional actions area
  on the right — either a single icon button (e.g. refresh), an overflow
  "⋯" menu, or both. A card used as a dashboard widget MUST have a header
  with at minimum a title.
- **Body**: the only required region. Padding is a fixed 20px on desktop,
  16px on mobile, per the spacing scale. Body content varies by variant
  (below).
- **Footer**: used for secondary metadata that supports but doesn't belong
  in the primary content — a "last updated" timestamp (see
  [`dashboard-standards.md`](./dashboard-standards.md#refresh-and-real-time-data-requirements)),
  a count ("Showing 5 of 42"), or a "View all →" link. Visually separated
  from the body by either a top border or increased whitespace, never both.
- **Actions**: primary in-card actions (e.g. "Approve" / "Reject" on an
  approval-request card) render as buttons at the bottom of the body or in a
  dedicated action row above the footer — never in the header, which is
  reserved for meta-actions (refresh, overflow menu) rather than
  content-level actions.

## Variants

- **Stat card**: body is a KPI stat tile per
  [`dashboard-standards.md`](./dashboard-standards.md#kpi-stat-tiles). No
  header title is required if the stat's own label serves that purpose;
  used standalone on dashboards.
- **List card**: body is a compact vertical list of up to 5-8 items (each
  item: icon/avatar, primary text, secondary text, optional trailing value
  or badge), with a footer "View all →" link if the underlying collection
  has more items than shown. Used for "Recent activity", "Top accounts",
  "Pending approvals" widgets.
- **Chart card**: body is a chart per
  [`chart-standards.md`](./chart-standards.md), header includes a time-range
  selector (e.g. "7d / 30d / 90d" segmented control) as a header action when
  the chart supports range switching.
- **Entity card**: represents a single record in a card-grid view (used as
  an alternative to a table row when browsing visual/spatial records — e.g.
  hotel rooms, product catalog items, floor plan units). Body includes a
  thumbnail/image region (fixed aspect ratio per
  [`../design-system/spacing-layout.md`](../design-system/spacing-layout.md)),
  a title, 1-3 key attributes, and a status badge. Entity cards are
  clickable in their entirety (the whole card is the click target, not just
  the title) and MUST show a visible hover state (below).

## Hover and interactive states

A card is "interactive" if clicking anywhere on it (outside of nested
controls like buttons or menus) navigates or opens something — this applies
to entity cards and list-card items, but NOT to stat cards or chart cards
used purely as dashboard display (those only navigate via an explicit link
or the stat tile's own click behavior per
[`dashboard-standards.md`](./dashboard-standards.md#kpi-stat-tiles)).

- **Default**: base border and shadow per the design system's elevation
  token `elevation-1`.
- **Hover** (interactive cards only): elevation increases to `elevation-2`,
  border color shifts to the design system's primary-tint, cursor becomes
  `pointer`, transition duration 150ms ease-out. Non-interactive cards MUST
  NOT show a hover elevation change — an unclickable card that visually
  reacts to hover misleads the user.
- **Focus** (keyboard navigation to an interactive card): a visible focus
  ring per [`../design-system/accessibility.md`](../design-system/accessibility.md),
  identical treatment to hover elevation plus the ring.
- **Active/pressed**: elevation drops back to `elevation-1` momentarily
  (100ms) to give tactile click feedback before navigation occurs.
- **Selected** (used in card-grid multi-select, e.g. bulk-selecting entity
  cards): a persistent 2px primary-color border and a checkmark badge in the
  top-left corner, independent of hover/focus state.
- **Disabled**: reduced opacity (per design system disabled token), cursor
  `not-allowed`, no hover elevation change, and MUST include a tooltip or
  adjacent text explaining why (per Principle 1 in
  [`ux-principles.md`](./ux-principles.md)).
- **Draggable** (dashboard edit mode): shows a drag-handle icon in the
  header on hover, per
  [`dashboard-standards.md`](./dashboard-standards.md#customizable-dashboards).

## Skeleton loading state

Every card MUST render a skeleton placeholder while its data is loading —
never a blank card, spinner-only card, or a card that pops in with a layout
shift.

- The skeleton MUST match the target card's final layout dimensions exactly
  (same header height, same body region heights) so no content jump occurs
  when real data replaces it.
- Skeleton regions render as pulsing gray blocks (per
  [`../design-system/motion-animation.md`](../design-system/motion-animation.md)
  for the pulse animation timing/easing) shaped to their content type: a
  short wide block for a title, a large block for a value, a thin block for
  a subtitle, and repeated row-shaped blocks for list cards (matching the
  expected item count or a default of 4).
- Minimum skeleton display time is 200ms even if data returns faster, to
  avoid a flash-of-skeleton that reads as flicker; there is no maximum —
  the skeleton persists until data arrives or an error state (below)
  replaces it.
- **Error state**: if the card's data fails to load, the skeleton is
  replaced with an inline error message ("Couldn't load this widget") and a
  "Retry" button, keeping the card's chrome (header, footer) intact rather
  than collapsing the whole card away — a failed widget MUST NOT change the
  dashboard's overall grid layout.
