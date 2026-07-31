# Dashboard Standards

Every Zodize product has a dashboard as its default landing page after
login. This document defines the widget grid system, the KPI stat tile
standard, the mandatory non-empty default state, the customization
requirement, and refresh/real-time expectations. Widget visual styling
follows [`card-standards.md`](./card-standards.md); the dashboard is
composed of cards laid out on the grid defined here.

## Widget grid system

- Dashboards use a 12-column responsive grid, matching the grid defined in
  [`../design-system/spacing-layout.md`](../design-system/spacing-layout.md).
  Each widget occupies a column span of 3, 4, 6, or 12 (never an arbitrary
  span), and a row-height unit of 1 (compact card, e.g. a stat tile) or 2
  (tall card, e.g. a chart or list).
- Standard widget sizes and their typical use:
  - `span-3, height-1` — single KPI stat tile (4 fit per row on desktop).
  - `span-4, height-2` — chart card or short list card (3 per row).
  - `span-6, height-2` — wide chart card or comparison card (2 per row).
  - `span-12, height-2` — full-width table/list widget or timeline.
- Widgets reflow to a single column below the tablet breakpoint per
  [`../design-system/responsive-standards.md`](../design-system/responsive-standards.md),
  in the order defined by their `order` property (see customization below),
  not the raw grid position, so reordering behaves predictably on mobile.
- Grid gap is a fixed 16px horizontal and vertical; widgets MUST NOT define
  custom margins to simulate a different gap.

## KPI stat tiles

A KPI stat tile is the primary building block of a Zodize dashboard and MUST
follow this exact anatomy:

```
Label (e.g. "Monthly Active Accounts")
Value (large numeral, e.g. "12,483")
Delta (e.g. "▲ 4.2% vs last period", colored per direction)
Sparkline (optional but REQUIRED when the underlying data has a
           time-series representation)
```

- **Label**: describes the metric and its time window if not obvious
  (e.g. "New Signups (30d)"). MUST NOT be abbreviated to the point of
  ambiguity.
- **Value**: formatted per the user's locale (see
  [`localization-i18n.md`](./localization-i18n.md#number-and-date-formatting)),
  large enough to be scannable (design system's `display-sm` or `display-md`
  type scale).
- **Delta**: REQUIRED whenever a comparison period exists (previous period,
  previous year, target). Positive deltas render in the design system's
  success color, negative in the danger color — UNLESS the metric is one
  where a decrease is good (e.g. churn rate, error rate, average handle
  time), in which case the color mapping inverts and the widget config MUST
  set an explicit `invertColor` flag rather than relying on the sign alone.
  A delta of exactly 0% renders in neutral/muted color with no arrow.
- **Sparkline**: a minimal, axis-less line or bar chart showing the trend
  over the comparison window, rendered per
  [`chart-standards.md`](./chart-standards.md#sparklines). Clicking a stat
  tile with a sparkline navigates to the full chart/report view for that
  metric.
- Stat tiles MUST render a skeleton state (per
  [`card-standards.md`](./card-standards.md#skeleton-loading-state)) while
  the metric is loading, and MUST NOT show a stale value from a previous
  session during load.

## Default dashboard — never empty

A dashboard showing no data on first login is a Definition-of-Done defect.
Every product MUST define a default dashboard configuration that is never
empty, resolved in this priority order:

1. **Real data, if any exists** — if the business already has data (e.g. an
   existing account with transactions), the dashboard renders real KPIs and
   charts immediately.
2. **Guided onboarding state** — if the deployment is new and has no data
   yet for a given widget, that widget renders an onboarding variant of itself:
   the same card chrome and layout position, but with a zero-state
   illustration, one sentence explaining what will appear here, and a
   direct call-to-action button to create the first relevant record (e.g. a
   "Revenue" chart widget with no transactions shows "Once you record your
   first sale, your revenue trend appears here." and a "Record a sale"
   button). This is NOT the same as the table/list empty state in
   [`table-standards.md`](./table-standards.md#empty-state) — it must be
   proactive and instructional, not just an absence notice.
3. **Sample/demo data mode** — for products where onboarding CTAs alone
   leave the dashboard feeling too sparse (e.g. an analytics-heavy product
   like ZodiCapital), the product MAY offer an explicit, clearly labeled
   "Preview with sample data" toggle that populates the dashboard with
   realistic, obviously-fake data (e.g. company name "Acme Holdings",
   watermarked with a persistent "Sample data" badge on every affected
   widget). Sample data MUST NEVER be shown without this explicit,
   dismissible label, and MUST NEVER be mixed silently with real data.

A product's `SPEC.md` MUST document which of tiers 2 or 3 (or both) it
implements for its default dashboard, and list the exact widgets in the
default layout.

## Customizable dashboards

Every product's dashboard MUST support user-level customization:

- **Add/remove widgets**: an "Edit dashboard" mode (toggled via a button in
  the page header) reveals a "＋ Add widget" tile and a remove control
  (small "×" in the top-right corner of each widget) on every existing
  widget. The widget picker opens as a drawer (per
  [`modal-standards.md`](./modal-standards.md#drawers)) categorized by data
  domain, with a live preview thumbnail per widget type.
- **Reorder**: in edit mode, widgets are draggable via a drag handle that
  appears on hover in the widget's header; drop targets snap to the grid.
  Reordering MUST be keyboard-accessible as an alternative: a "Move" entry
  in each widget's overflow menu opens a submenu with "Move up / Move down /
  Move to position…".
- **Resize**: widgets that support multiple valid spans (see grid sizes
  above) expose a resize handle on their bottom-right corner in edit mode,
  constrained to the defined span values (no free-form resize).
- **Persistence**: layout changes save automatically per user, per
  dashboard (a user may have a distinct layout on each product's dashboard),
  and MUST NOT require an explicit "Save layout" action — leaving edit mode
  persists it, with an "Reset to default" option available from the
  dashboard's overflow menu for recovery.
- **Role-based defaults**: a product MAY define different default widget
  sets per role (e.g. a branch manager's default dashboard differs from a
  teller's), but a user's own customization always overrides the role
  default once they have made a change.

## Refresh and real-time data requirements

- Every dashboard widget MUST display a "last updated" relative timestamp
  (e.g. "Updated 2 minutes ago") in its footer, per
  [`card-standards.md`](./card-standards.md#anatomy).
- Widgets backed by data that changes frequently (transaction volume, active
  sessions, queue depth) MUST update via the real-time broadcast channel
  defined in
  [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md)
  rather than polling, with a visual pulse/highlight animation (per
  [`../design-system/motion-animation.md`](../design-system/motion-animation.md))
  on the value when it changes, lasting 600ms, to draw attention without
  being distracting.
- Widgets backed by data that is expensive to compute (aggregate reports,
  cross-branch rollups per
  [`localization-i18n.md`](./localization-i18n.md#multi-company--multi-branch-data-scoping))
  MAY use a polling refresh at a minimum interval of
  60 seconds instead of real-time push, but MUST still show the last-updated
  timestamp and MUST offer a manual "Refresh" icon button in the widget
  header for the user to force an update on demand.
- A manual "Refresh all" action MUST be available in the dashboard page
  header, showing a brief loading state on every widget simultaneously.
