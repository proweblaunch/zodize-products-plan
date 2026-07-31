# Chart Standards

This document defines which chart type to use for a given data shape, how
charts use design-system color tokens, the tooltip standard, chart-specific
loading/empty/error states, accessibility requirements, and real-time update
behavior. Charts are most commonly embedded in a chart card (see
[`card-standards.md`](./card-standards.md#variants)) on a dashboard (see
[`dashboard-standards.md`](./dashboard-standards.md)).

## Approved chart types per data shape

A product team MUST select a chart type from this table based on the shape
of the underlying data — inventing a novel chart type or using a chart type
outside its approved shape requires an ADR per
[`../../CONTRIBUTING.md`](../../CONTRIBUTING.md).

| Data shape | Chart type | Notes |
|---|---|---|
| Single metric over time, one series | Line chart | Default for trends (revenue over time). |
| Single metric over time, multiple series to compare | Multi-line chart | Max 5 series before switching to small multiples. |
| Cumulative or volume over time | Area chart | Use for stacked values (e.g. revenue by channel over time) with semi-transparent fill. |
| Discrete categories, comparing magnitude | Bar chart (vertical) | Default for category comparison (revenue by branch). |
| Discrete categories, long labels or many categories (>8) | Bar chart (horizontal) | Avoids label truncation/rotation. |
| Part-to-whole, 2-5 categories | Donut chart | Never a 3D pie. Center of donut MAY show a total/key value. |
| Part-to-whole, >5 categories | Stacked bar chart | Pie/donut with more than 5 slices is prohibited — segments become unreadable. |
| Two dimensions with intensity/density (e.g. day-of-week × hour-of-day occupancy) | Heatmap | Color intensity uses a sequential scale, never the categorical palette. |
| Sequential process with drop-off (e.g. signup funnel, sales pipeline) | Funnel chart | Each stage shows absolute count and conversion % from the previous stage. |
| Sparkline (compact trend indicator inside a stat tile) | Sparkline | See [`dashboard-standards.md`](./dashboard-standards.md#kpi-stat-tiles); axis-less, single series only. |

Pie charts (as opposed to donut) are not used anywhere in Zodize products —
donut is the standard for part-to-whole visualization because the center
space can carry the total value.

## Color usage

- Chart colors MUST be drawn from the categorical, sequential, and
  diverging chart-color scales defined in
  [`../design-system/color-system.md`](../design-system/color-system.md) —
  charts MUST NOT hardcode hex values or introduce chart-specific colors
  outside that token set.
- **Categorical** data (distinct series/categories with no inherent order,
  e.g. revenue by product line) uses the categorical scale, assigned in a
  fixed, consistent order across a session so the same category always gets
  the same color within one product (e.g. "ZodiBank Checking" is always the
  same blue on every chart in the app).
- **Sequential** data (a single variable's intensity, e.g. heatmap
  occupancy) uses the sequential scale from the design system, low-to-high.
- **Diverging** data (a variable with a meaningful zero/midpoint, e.g.
  profit/loss, sentiment score) uses the diverging scale, centered on the
  neutral midpoint color at the zero value.
- Status-semantic values (success, warning, danger states within a chart,
  e.g. red for "over budget" bars) use the design system's semantic colors
  (success/warning/danger tokens), not the categorical scale, so meaning is
  consistent with the rest of the product's status badges.
- Every chart with 2+ series MUST render a legend; single-series charts omit
  the legend and rely on the card title/axis labels instead.

## Tooltip standard

- Hovering (desktop) or tapping (touch) a data point MUST show a tooltip
  within 50ms, positioned to avoid clipping at the viewport/card edge
  (flips above/below or left/right as needed).
- Tooltip content, in fixed order: the category/date label (bold), then one
  row per series at that point (color swatch, series name, formatted
  value), then, for time-series comparisons, the delta vs. the previous
  point or comparison period if the chart has one configured.
- Values in tooltips MUST use the same locale-aware formatting as the rest
  of the product, per
  [`localization-i18n.md`](./localization-i18n.md#number-and-date-formatting)
  — currency values show the currency symbol/code, percentages show one
  decimal place by default.
- On touch devices, tapping a point pins the tooltip open until the user
  taps elsewhere on the chart or outside it; a hover-only tooltip that
  disappears on touch-release is prohibited since it makes the value
  unreadable on mobile.

## Empty, loading, and error states

- **Loading**: chart cards show the skeleton state defined in
  [`card-standards.md`](./card-standards.md#skeleton-loading-state), shaped
  as a flat placeholder bar/line matching the chart's approximate final
  dimensions, not a generic spinner.
- **Empty** (no data for the selected range): the chart area is replaced
  with a centered message ("No data for this period") and, if the product
  supports it, a suggestion to widen the date range ("Try the last 90 days
  instead"). This follows the same never-silently-blank rule as
  [`dashboard-standards.md`](./dashboard-standards.md#default-dashboard--never-empty) —
  an empty chart card MUST explain why, never render as a bare blank box.
- **Error** (data fetch failed): identical treatment to the card error state
  in [`card-standards.md`](./card-standards.md#skeleton-loading-state) — an
  inline message and Retry button, chart chrome (title, range selector)
  intact.
- **Partial data** (e.g. real-time chart where today's data is incomplete):
  the current, still-accumulating period MUST be visually distinguished
  (dashed line segment or reduced-opacity bar) with a "Partial" label in the
  tooltip for that point, so users do not misread an in-progress period as a
  completed downturn.

## Accessibility

- Every chart MUST have an accessible text alternative: a visually-hidden
  (`sr-only`) data table containing the same series/category/value data,
  associated with the chart via `aria-describedby`, so screen reader users
  get the underlying data rather than being told only "chart image."
- Charts with a "View data" toggle (recommended for any chart card with
  more than one series) render that same data as a visible, standard data
  table per [`table-standards.md`](./table-standards.md) when toggled,
  serving both accessibility and power-user needs from one implementation.
- Color MUST NOT be the only means of distinguishing series — line charts
  use distinct dash patterns per series when more than 3 series are shown
  simultaneously, and bar/donut charts include a legend with text labels
  (never color-swatch-only legends).
- All chart color combinations MUST meet the contrast requirements in
  [`../design-system/accessibility.md`](../design-system/accessibility.md)
  against the chart's background in both light and dark theme, per
  [`../design-system/dark-theme.md`](../design-system/dark-theme.md).

## Real-time and live chart updates

- Charts backed by real-time data (per
  [`dashboard-standards.md`](./dashboard-standards.md#refresh-and-real-time-data-requirements))
  MUST animate new data points in smoothly (data enters from the trailing
  edge, existing points shift left) rather than snapping/redrawing the
  entire chart, using the transition timing defined in
  [`../design-system/motion-animation.md`](../design-system/motion-animation.md).
- A live-updating chart MUST show a small pulsing "Live" indicator badge in
  its header when actively receiving real-time updates, and MUST pause
  automatic updates (with a "Updates paused — N new" banner and a manual
  "Resume" action) whenever the user is actively interacting with it
  (hovering a tooltip, has opened the "View data" table), so live data
  doesn't yank the view out from under an in-progress inspection.
- Charts with a time-range selector (7d/30d/90d/custom) reset to a static
  (non-live) rendering when a historical, non-"live" range is selected —
  real-time push behavior only applies to the current-period/live default
  view.
