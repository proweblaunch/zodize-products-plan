# Navigation Standards

Every Zodize product MUST use the same navigation shell: a collapsible left
sidebar, a top bar, contextual breadcrumbs, and a global command palette.
This document defines their exact structure and behavior. Visual treatment
(color, spacing, icon set) is defined in
[`../design-system/components.md`](../design-system/components.md) and
[`../design-system/spacing-layout.md`](../design-system/spacing-layout.md);
this document defines interaction and information architecture.

## Sidebar

The sidebar is the primary navigation surface and MUST be present on every
authenticated screen except full-screen focus modes (see
[`page-layout-standards.md`](./page-layout-standards.md#full-screen-mode)).

- **Structure**: logo/product switcher at top, a scrollable list of
  navigation sections below, a pinned footer area (user avatar, help link,
  collapse toggle) at bottom.
- **Sections**: top-level nav items MUST be grouped into labeled sections
  (e.g. "Workspace", "Records", "Reports", "Settings") when a product has
  more than 7 top-level items. Section labels are non-interactive, uppercase,
  small text per the design system's typography scale.
- **Items**: every nav item MUST render an icon plus a text label. Icon-only
  sidebars are prohibited outside the collapsed state (below) because they
  fail Principle 1 (clarity) in [`ux-principles.md`](./ux-principles.md).
- **Active state**: the nav item matching the current route MUST show a
  distinct background fill, a left accent bar in the product's primary
  color, and a bolded label. Exactly one top-level item (and its parent
  section, if nested) is active at a time — never zero, never more than one.
- **Nesting**: a maximum of two levels of nesting is permitted (section →
  item → sub-item). Sub-items appear as an indented, collapsible list under
  their parent when the parent is active or manually expanded. A chevron
  icon indicates expand/collapse state and rotates 90° on expand with a
  150ms ease-out transition.
- **Badges**: nav items MAY show a numeric or dot badge for unread/pending
  counts (e.g. "Approvals (4)"). Counts above 99 MUST render as "99+".
  Badges use the notification data model in
  [`notification-standards.md`](./notification-standards.md#data-model).
- **Collapse**: the sidebar MUST be collapsible to an icon-only rail
  (64px wide) via a toggle button pinned to the sidebar footer. In collapsed
  state, each icon MUST show its label in a tooltip on hover/focus after
  400ms. Collapse state persists per-user across sessions (stored in user
  preferences, not local-only). Collapsing MUST NOT collapse expanded
  sub-navigation state — it is restored when the sidebar re-expands.
- **Resize**: the sidebar is not user-resizable; its expanded width is fixed
  per the design system (default 260px) to keep navigation muscle memory
  consistent across products.

## Top bar

The top bar sits above the content region, to the right of the sidebar (or
full-width above it in collapsed/mobile layouts), and is always visible
(does not scroll with content).

- **Left**: breadcrumb trail (see below) or, on the dashboard/home route, the
  page title.
- **Center or left-of-actions**: global search input. Clicking or focusing it
  opens the command palette (see below) rather than a separate search UI —
  Zodize does not maintain two competing search entry points.
- **Right, in fixed order**: company/branch switcher (if the product
  supports multi-company/multi-branch scoping per
  [`localization-i18n.md`](./localization-i18n.md#multi-company--multi-branch-data-scoping)
  and the user has access to more than one company or branch) → notification
  bell → user menu.
  - **Company/branch switcher**: renders the current company/branch's name
    and logo/initials. Clicking opens a dropdown listing every company/
    branch the user has access to, each with logo, name, and the user's role
    at that branch, plus an "All branches" option (for users with
    cross-branch permission) and a "Manage branches" link at the bottom.
    Switching triggers a full context reload (not a partial SPA swap) to
    guarantee no cross-branch data leakage in client state. There is no
    organization/tenant switcher — every deployment belongs to exactly one
    buyer's business (see
    [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)),
    and a product with no multi-branch concept omits this switcher entirely.
  - **Notification bell**: shows an unread-count badge and opens the
    notification center panel on click, per
    [`notification-standards.md`](./notification-standards.md#notification-center).
  - **User menu**: avatar (or initials fallback) opens a dropdown with
    profile link, theme toggle, help/docs link, keyboard shortcuts link, and
    sign out, in that fixed order, with sign out visually separated by a
    divider.

## Breadcrumbs

Breadcrumbs render the hierarchical path from the top-level nav section to
the current record, and are REQUIRED on every detail page and every page
nested more than one level deep.

- Format: `Section / Sub-section / Current page`, with `/` rendered as a
  chevron icon, not a literal slash character.
- Every segment except the last MUST be a clickable link back to that level.
  The last segment (current page) is plain text, not a link, and MUST match
  the page's `<h1>` title.
- On a record detail page, the record's identifying name/number replaces a
  generic label (e.g. `Accounts / ACC-10293`, not `Accounts / Detail`).
- On mobile widths (below the breakpoint in
  [`../design-system/responsive-standards.md`](../design-system/responsive-standards.md)),
  breadcrumbs collapse to a single "← Back to Accounts" link showing only the
  immediate parent, to conserve horizontal space.

## Command palette (⌘K)

The command palette is a REQUIRED feature of every Zodize product, not an
optional enhancement. It is the fastest path to any destination, action, or
record, and is the primary interface for Principle 4 (keyboard-first) in
[`ux-principles.md`](./ux-principles.md).

- **Invocation**: `Cmd+K` (macOS) / `Ctrl+K` (Windows/Linux) opens the
  palette from anywhere in the authenticated app. Clicking the top bar
  search input also opens it. `Escape` closes it.
- **Structure**: a centered modal overlay (per
  [`modal-standards.md`](./modal-standards.md#sizes), size `sm`, max-width
  560px) with a text input pinned at top and a scrollable, categorized
  results list below.
- **Empty query state**: shows recent items (last 5 records the user
  opened) and a fixed list of top-level navigation shortcuts.
- **Result categories**, in fixed order: Actions (e.g. "Create invoice"),
  Navigation (pages), Records (search results from the product's primary
  entities), Settings. Each category shows a maximum of 5 results with a
  "View all N results" row if truncated.
- **Search behavior**: debounced at 200ms, queries the product's global
  search endpoint, and MUST return results within 300ms at the p95 for a
  deployment with up to 100k records (see
  [`../development/`](../development) API performance standards). A loading
  spinner replaces the category headers while a query is in flight; results
  from the previous query are not shown stale.
- **Keyboard interaction**: `↑`/`↓` moves selection across all results
  regardless of category, wrapping is disabled (stops at first/last), `Enter`
  activates the selected result, and number keys `1`-`9` are not bound to
  avoid conflicting with typed queries.
- **No-results state**: shows a "No results for '<query>'" message with a
  "Search everywhere" fallback link that runs a full-text search across all
  entity types instead of the default weighted/recent results.

## Mobile navigation collapse

Below the tablet breakpoint defined in
[`../design-system/responsive-standards.md`](../design-system/responsive-standards.md):

- The sidebar MUST be hidden by default and accessible via a hamburger icon
  in the top bar's far left position, replacing the breadcrumb trail. Tapping
  it slides the sidebar in as a full-height overlay drawer (see
  [`modal-standards.md`](./modal-standards.md#drawers)) with a scrim behind
  it; tapping the scrim or a nav item closes it.
- The top bar collapses to: hamburger, page title (centered or left-aligned
  per product), notification bell, user avatar. The company/branch switcher
  and search input move inside the sidebar drawer's header on mobile rather
  than competing for top bar space.
- The command palette remains available via the search entry in the sidebar
  drawer; a dedicated on-screen keyboard shortcut is not applicable on
  touch, so a visible "Search" row with a magnifying-glass icon MUST be the
  first item in the drawer.
- Bottom tab bars are NOT part of the standard Zodize navigation pattern —
  all products use the drawer pattern for consistency, even on mobile web,
  unless a product's `SPEC.md` documents a native-app-specific exception.
