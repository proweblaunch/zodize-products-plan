# Table Standards

The data table is the most-used component across Zodize products — nearly
every list-page template (see
[`page-layout-standards.md`](./page-layout-standards.md#list-page-template))
is built around one. This document is the complete behavioral spec for the
standard Zodize data table. Visual tokens (row height, borders, colors) come
from [`../design-system/components.md`](../design-system/components.md).

## Sorting

- Any column marked sortable in the table's configuration shows a sort
  affordance (chevron icon) in its header on hover, and a solid up/down
  chevron when active.
- Clicking a sortable header cycles: unsorted → ascending → descending →
  unsorted. Only ONE column is sortable at a time by default; multi-column
  sort (shift-click to add a secondary sort key) is supported only for
  tables explicitly configured for it, indicated by a numbered badge (`1`,
  `2`) next to each active sort chevron.
- Sort state MUST persist in the URL query string (e.g. `?sort=created_at&dir=desc`)
  so a sorted view is shareable and survives refresh, and MUST be included
  in saved views (below).
- Sorting a column with a server-side (paginated) dataset MUST trigger a new
  server request, not a client-side re-sort of the currently loaded page.

## Multi-column filtering

- A filter bar sits above the table (per
  [`page-layout-standards.md`](./page-layout-standards.md#list-page-template))
  with a primary text search input plus a "＋ Add filter" control that opens
  a dropdown of filterable columns.
- Each active filter renders as a removable chip (`Status: Active ×`) in the
  filter bar. Chips for the same field combine with OR logic internally
  (e.g. `Status: Active, Pending` means either); different fields combine
  with AND logic.
- Filter operators depend on the column's data type: text columns get
  contains/equals/starts-with; date columns get a date-range picker with
  presets (Today, Last 7 days, Last 30 days, Custom range); numeric/currency
  columns get equals/greater-than/less-than/between; enum/status columns get
  a multi-select checklist.
- Filters combine with the free-text search (AND logic) and MUST persist in
  the URL query string identically to sort state.
- A "Clear all filters" text link appears in the filter bar whenever at
  least one filter is active.

## Column visibility, resize, and reorder

- Every table with more than 6 columns MUST provide a column-visibility
  toggle, accessed via a "Columns" button (icon: sliders) at the right edge
  of the filter bar, opening a checklist of all available columns with
  show/hide toggles. At least one column plus the identifying column (e.g.
  name/ID) MUST remain visible at all times — the identifying column's
  toggle is disabled.
- Columns are resizable by dragging the border between header cells; the
  cursor changes to `col-resize` on hover over the 8px hit area at each
  border. Minimum column width is 80px. Resize state persists per saved
  view (below).
- Columns are reorderable by dragging a header cell by a drag handle that
  appears on hover in desktop pointer environments; the identifying column
  is pinned first and not reorderable.
- Visibility, width, and order preferences persist per user, per table,
  independent of saved views, as the table's "current" state, with saved
  views layering named, shareable snapshots on top (below).

## Row selection and bulk actions

- Tables that support bulk operations show a checkbox in the leftmost cell
  of every row plus a header checkbox that selects/deselects all rows on
  the current page. A tri-state header checkbox (checked / unchecked /
  indeterminate) reflects partial-page selection.
- Selecting at least one row replaces the filter bar with a bulk action bar
  (fixed height swap, per
  [`page-layout-standards.md`](./page-layout-standards.md#list-page-template))
  showing the selected count ("14 selected"), a "Clear selection" link, and
  the available bulk actions as buttons (e.g. "Export", "Archive",
  "Delete").
- When all rows on the current page are selected, a banner offers "Select
  all 1,284 matching records" to extend selection beyond the current page
  for server-side bulk operations; this MUST show the exact total count,
  not "all".
- Bulk destructive actions (delete, deactivate) MUST route through the
  confirmation-dialog tiers in
  [`modal-standards.md`](./modal-standards.md#destructive-action-tiers),
  restating the exact count and, where feasible, a sample of affected
  record names/IDs.

## Pagination

- Tables with 100 rows or fewer in total MAY load and paginate entirely
  client-side.
- Tables with MORE than 100 rows MUST use server-side pagination: each page
  change issues a new request with the current sort/filter/page-size state,
  and the table shows a loading skeleton (below) for the new page rather
  than blocking the whole page.
- Page size options are 25 / 50 / 100 (default 25), selectable via a control
  in the pagination footer; page size preference persists per user, per
  table.
- The pagination footer shows the current range and total ("Showing 1-25 of
  1,284"), Previous/Next buttons, and direct page-number links for up to 5
  pages around the current page, with first/last jump controls for large
  result sets.

## Inline row actions

- Each row MAY show inline actions in its rightmost cell: for 1-2 actions,
  render as visible icon buttons that appear on row hover (and are always
  visible on touch devices); for 3+ actions, collapse into a single
  overflow "⋯" menu that is always visible (not hover-only, since touch
  devices have no hover).
- The overflow menu's first item is the most common action, destructive
  items (delete, deactivate) are visually separated at the bottom with a
  divider and rendered in the danger color.
- Inline actions MUST NOT trigger row selection or row-click navigation —
  clicks on the actions cell always stop propagation.

## Empty states

Two distinct empty states MUST be distinguished:

- **Zero records overall** (the business has never created any): shows a centered
  illustration, a one-sentence explanation, and a primary "Create <entity>"
  button — matching the onboarding pattern in
  [`dashboard-standards.md`](./dashboard-standards.md#default-dashboard--never-empty).
- **Zero results from filters/search** (records exist but none match):
  shows a lighter-weight inline message ("No results match your filters")
  with a "Clear filters" link — no illustration, no creation CTA, since
  records do exist.

## Loading skeleton

- Initial table load renders skeleton rows (default 8) matching the
  configured columns' approximate content width, using the same pulse
  animation as [`card-standards.md`](./card-standards.md#skeleton-loading-state).
- Subsequent loads (pagination, sort, filter change) show a thin
  indeterminate progress bar at the top edge of the table plus a dimmed
  (60% opacity) overlay on the existing rows, rather than replacing rows
  with skeletons, so the user retains visual context of what they were
  looking at. This MUST resolve within the same interaction cycle; there is
  no separate "loading more" state distinct from a full page-change load.

## Export

- Every table that supports bulk actions MUST also support export, exposed
  either as a bulk action (exports only selected rows) or a "Export" button
  in the filter bar (exports the current filtered/sorted view).
- Supported formats: CSV (always) and XLSX (required whenever the export
  exceeds 3 columns or includes formatted values like currency/dates that
  benefit from native spreadsheet typing).
- Exports of more than 5,000 rows MUST run as an asynchronous background job
  (per [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md))
  with a progress notification delivered via the notification center (see
  [`notification-standards.md`](./notification-standards.md#notification-center))
  and a download link that expires after 24 hours, rather than blocking the
  browser tab.

## Saved views

- Users MAY save the combination of filters, sort, column visibility,
  width, and order as a named view, accessed via a "Views" dropdown at the
  left of the filter bar (default entry: "All <entities>", not deletable).
- Saved views are personal by default; a view MAY be marked "Shared with
  team" (visible to all users with access to the table) by a user with
  edit permission on that view, per the RBAC model in
  `docs/security/`.
- The active view's name shows in the Views dropdown trigger; unsaved
  changes to a loaded view show a dot indicator and offer "Save" (overwrite)
  or "Save as new view" actions.

## Sticky header

- The column header row MUST remain visible (sticky, `position: sticky`)
  while scrolling through table rows, per
  [`page-layout-standards.md`](./page-layout-standards.md#page-header-pattern)'s
  single-sticky-layer rule.
- On horizontal scroll (wide tables with many columns), the identifying
  column MUST remain pinned/frozen to the left edge so the user always
  knows which record a row belongs to while scrolling right.

## Row density

- Users may toggle row density via a control next to the Columns button:
  **Comfortable** (default, 48px row height), **Compact** (36px row
  height, reduced cell padding, smaller avatar/icon sizes). Density
  preference persists per user across all tables in the product (a global
  preference, not per-table), since it reflects a user's general
  scanning-speed preference.

## Keyboard interaction

- Once a row has focus (via `Tab` or by clicking a non-interactive cell),
  `↑`/`↓` moves focus between rows, `Space` toggles that row's selection
  checkbox, and `Enter` activates the row's primary action (typically
  navigating to its detail page), satisfying Principle 4 in
  [`ux-principles.md`](./ux-principles.md).

## Responsive behavior

- On mobile widths, tables with more than 4 columns MUST switch to a
  stacked card layout — one card per row (using the list-card variant in
  [`card-standards.md`](./card-standards.md#variants)) showing the
  identifying field prominently and up to 3 secondary fields as
  label/value pairs — rather than forcing horizontal scroll, which is
  reserved for desktop/tablet widths where the full table remains usable
  inside an `overflow-x: auto` container.
