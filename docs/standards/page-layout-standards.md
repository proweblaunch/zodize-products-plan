# Page Layout Standards

This document defines the page shell every Zodize screen renders inside, the
standard page header pattern, content-width rules, and the three layout
templates (list, detail, form) that every product screen must map to. It
complements [`navigation-standards.md`](./navigation-standards.md) (which
defines the persistent chrome) and
[`../design-system/spacing-layout.md`](../design-system/spacing-layout.md)
(which defines the underlying grid and spacing tokens used below).

## Page shell regions

Every authenticated screen is composed of four regions, in this fixed
arrangement:

1. **Sidebar** — persistent left navigation, per
   [`navigation-standards.md`](./navigation-standards.md#sidebar). Fixed
   width, full viewport height, does not scroll with content.
2. **Top bar** — persistent header above content, per
   [`navigation-standards.md`](./navigation-standards.md#top-bar). Fixed
   height (56px), does not scroll with content.
3. **Content region** — the only region that scrolls vertically. Contains
   the page header (below) and the page body. Horizontal overflow within
   this region is prohibited except inside explicitly scrollable
   sub-containers (e.g. wide tables, per
   [`table-standards.md`](./table-standards.md#responsive-behavior)).
4. **Footer** — OPTIONAL, used only for legal/compliance text (copyright,
   version number, support links) on a small number of screens (billing,
   legal, public-facing pages). Most authenticated app screens do not render
   a footer; the content region ends when its content ends, it does not pad
   out to fill the viewport.

### Full-screen mode

A small set of screens (onboarding wizards, document editors, kiosk/POS
checkout flows) MAY suppress the sidebar and top bar entirely in favor of a
minimal top strip containing only a back/close control, a page title, and
essential actions. Full-screen mode MUST be documented as such in the
product's `SPEC.md` for the specific route it applies to — it is an
exception, not a default.

## Page header pattern

Every page in the content region MUST begin with a page header, a
structurally consistent block regardless of product or page type:

```
[Title]                                    [Secondary actions] [Primary action]
[Description text, optional]
[Tabs, optional]
```

- **Title**: an `<h1>`, matches the last breadcrumb segment exactly (see
  [`navigation-standards.md`](./navigation-standards.md#breadcrumbs)).
  Required on every page.
- **Description**: a single line (truncated with a "more" expansion if
  longer) of supporting context under the title. Optional — omit rather than
  fill with filler text.
- **Primary action**: at most ONE primary (filled, high-emphasis) button,
  right-aligned at the top of the header, representing the single most
  common action a user takes on this page (e.g. "New invoice" on an
  invoices list). A page MUST NOT have zero or multiple primary actions.
- **Secondary actions**: zero to three lower-emphasis actions (outlined or
  ghost buttons, or a single overflow "⋯" menu once there are more than 3)
  positioned immediately left of the primary action.
- **Tabs**: OPTIONAL, used to switch between views of the same object (e.g.
  "Details / Activity / Documents" on a record) or between filtered subsets
  of a list (e.g. "All / Active / Archived"). Tabs render as an underlined
  tab strip spanning the header's full width, directly below the
  title/description block. Tabs MUST update the URL (query param or path
  segment) so a tab is deep-linkable and survives a page refresh.

The page header is sticky (remains visible) while the page body scrolls,
UNLESS the page body is itself a data table with its own sticky header (see
[`table-standards.md`](./table-standards.md#sticky-header)), in which case
only one sticky layer is active at a time to avoid stacked sticky headers
consuming excessive vertical space — the page header scrolls away and the
table's own header becomes sticky.

## Content width rules

- Content MUST be constrained to a maximum width appropriate to its type,
  centered within the content region, with responsive gutters defined in
  [`../design-system/spacing-layout.md`](../design-system/spacing-layout.md):
  - **List/table pages**: full available width (tables benefit from more
    horizontal room for columns) up to no hard cap.
  - **Detail pages**: max-width 1280px, to keep multi-column detail layouts
    (see below) readable without excessive line length in text blocks.
  - **Form pages**: max-width 720px for single-column forms, 960px for
    forms using the two-column field layout in
    [`form-standards.md`](./form-standards.md#field-layout). Forms are never
    full-width — long input lines and long labels reduce scan speed.
  - **Dashboard pages**: full available width, laid out on the widget grid
    in [`dashboard-standards.md`](./dashboard-standards.md#widget-grid).
- Content padding from the content region edge is a fixed 24px on desktop,
  16px on mobile, per the spacing scale — pages MUST NOT define ad hoc
  padding values.

## List-page template

Used for any screen whose primary purpose is browsing/searching a
collection of records (accounts, invoices, guests, tickets).

```
Page header (title="Accounts", primary action="New account")
Filter/search bar (search input, filter chips, saved views, column toggle)
Data table (see table-standards.md), full width
Pagination footer
```

- The filter/search bar sits directly under the page header with no
  intervening whitespace beyond the standard section gap (24px), and MUST
  persist filter state in the URL query string so a filtered view is
  shareable and survives refresh.
- Bulk action bar (row-selection toolbar) replaces the filter/search bar
  when one or more rows are selected, per
  [`table-standards.md`](./table-standards.md#bulk-actions), and MUST NOT
  push the table down (fixed height swap, not an insertion).
- Empty state (zero records, vs. zero results from a filter) is defined per
  [`table-standards.md`](./table-standards.md#empty-state).

## Detail-page template

Used for any screen showing a single record in full.

```
Page header (title=record name/number, tabs for related sections)
Two-column body (desktop):
  Left/main column (66%): primary content for the active tab
  Right column (33%): summary metadata card, related-records widgets,
                       activity feed
```

- On tablet and mobile widths, the two-column body collapses to a single
  column with the right-column content moved below the main column, per
  [`../design-system/responsive-standards.md`](../design-system/responsive-standards.md).
- The right column is NOT tab-dependent — it stays constant while the left
  column's content changes with the active tab, giving the user persistent
  at-a-glance context (status, owner, key dates) regardless of which tab
  they are viewing.
- Destructive actions on a record (delete, deactivate, archive) live in an
  overflow menu in the page header's secondary actions, never as a primary
  action, per Principle 6 in [`ux-principles.md`](./ux-principles.md).

## Form-page template

Used for create/edit screens that are NOT presented in a modal (see
[`modal-standards.md`](./modal-standards.md#modal-vs-drawer-vs-full-page)
for when a form belongs in a modal, drawer, or its own page).

```
Page header (title="New invoice" / "Edit invoice #1029", secondary
             action="Cancel", primary action="Save")
Single or two-column form body (see form-standards.md)
Sticky footer action bar (mobile only, mirrors header actions)
```

- Form pages use "Cancel" (returns to the previous list/detail page,
  discarding unsaved changes with a confirmation if the form is dirty) and
  "Save" (or a more specific verb like "Create account") as the only two
  header-level actions — no overflow menu on a create/edit form page.
- Long forms split across logical sections use in-page anchored section
  headers with a sticky in-page mini-nav on the right for forms exceeding
  4 sections, rather than paginating, unless the form is explicitly a
  multi-step wizard (see
  [`form-standards.md`](./form-standards.md#multi-step-wizard)).

## Roadmap

- A fourth template for kanban/board-style pages (used by pipeline and
  workflow-heavy products) is planned but not yet specified; until written,
  such screens should follow the list-page template with the data table
  replaced by a board component documented in the consuming product's
  `SPEC.md`.
