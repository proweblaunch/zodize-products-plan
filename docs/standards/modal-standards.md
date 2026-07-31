# Modal Standards

This document defines the sizes, use cases, and interaction rules for
modals, drawers, and confirmation dialogs — the three overlay patterns used
across every Zodize product — plus the decision rule for when a task belongs
in an overlay at all versus its own page. Visual tokens (overlay scrim
opacity, border radius, shadow) come from
[`../design-system/components.md`](../design-system/components.md).

## Sizes

Modals come in four fixed sizes; a product MUST NOT introduce arbitrary
pixel widths outside these:

| Size | Max width | Typical use |
|---|---|---|
| `sm` | 480px | Confirmation dialogs, single-field prompts, the command palette. |
| `md` | 640px | Short forms (2-5 fields), simple create dialogs. |
| `lg` | 960px | Multi-field forms, forms with a two-column layout. |
| `xl` | 1200px | Complex forms with embedded tables/previews, comparison views. |

- Modal height is always content-driven up to a maximum of 85% of the
  viewport height; beyond that, the modal body (not the header/footer)
  becomes independently scrollable while the header and footer action bar
  remain fixed/visible.
- Modals are centered horizontally and vertically in the viewport, with a
  scrim (per the design system's overlay token) behind them.

## Modal vs. drawer vs. full page

Use this decision order, in priority:

1. **Full page** (its own route, per
   [`page-layout-standards.md`](./page-layout-standards.md#form-page-template)):
   use when the task has 6+ fields, is a multi-step wizard, needs to be
   bookmarkable/deep-linkable, or the user reasonably expects to spend more
   than ~2 minutes on it (e.g. onboarding a new employee record with
   documents, creating a full product listing).
2. **Drawer** (slides in from the right edge, full viewport height, width
   400-560px depending on content): use for quick create/edit tasks that
   benefit from keeping the underlying list or detail page visible/in
   context behind it (e.g. quickly editing one row's details without losing
   your place in a filtered table), and for the mobile sidebar navigation
   overlay (per
   [`navigation-standards.md`](./navigation-standards.md#mobile-navigation-collapse)).
   Drawers use the same header/body/footer structure as modals but anchor
   to the right edge and animate in via a 250ms ease-out slide, with the
   scrim behind them.
3. **Modal**: use for short, focused, single-purpose interactions that
   interrupt the current flow on purpose — confirmations, single-field
   prompts, short forms under 5 fields, and previews. If a "modal" form
   grows past `lg` size or accumulates more than 5 fields, split it into a
   drawer or full page instead — modals must not become the depth-1200px
   task screens page-layout-standards reserves for full pages.

## Confirmation dialogs for destructive actions

Confirmation dialogs are the primary safety mechanism required by Principle
6 in [`ux-principles.md`](./ux-principles.md), and use a fixed three-tier
model based on blast radius:

### Destructive action tiers

- **Tier 1 — Low risk, single record, reversible via undo**: no modal at
  all. Perform the action immediately and show a toast with an "Undo"
  action (per
  [`notification-standards.md`](./notification-standards.md#toast-notifications)).
  Example: archiving a single draft.
- **Tier 2 — Moderate risk, single record or small batch, not easily
  undone**: a `sm` confirmation modal. MUST restate exactly what will
  happen using the specific record's identifying name (never a generic
  "Are you sure?"), e.g. "Delete invoice INV-1042? This cannot be undone."
  Two buttons: a neutral "Cancel" (left, default-focused) and a danger-
  colored primary action labeled with the specific verb (e.g. "Delete
  invoice", never a bare "Confirm" or "OK"). Example: deleting one contact,
  deactivating one user.
- **Tier 3 — High risk: bulk operations, financial transfers, permission
  or company/branch-wide changes, or permanent data loss at scale**: a `sm`
  confirmation modal that additionally REQUIRES the user to type a specific
  confirmation phrase into a text input before the primary action button
  becomes enabled. The phrase is either the exact record identifier/name
  (for a single high-value record, e.g. type the account number to close a
  bank account) or the literal word matching the action (e.g. type
  "DELETE" to bulk-delete 340 records). The dialog MUST show the exact
  count and, where feasible, list a truncated sample of affected items
  (first 5 names/IDs + "and 335 more"). The primary button remains visibly
  disabled (not just inert) until the typed value matches exactly,
  case-sensitive.

A product's `SPEC.md` MUST classify each of its destructive actions into one
of these three tiers; an undocumented destructive action defaults to Tier 2
until explicitly classified.

## Focus trap and escape-key behavior

- On open, focus MUST move to the modal's first focusable element (or the
  modal container itself if it has no form fields), and MUST be trapped
  inside the modal — `Tab`/`Shift+Tab` cycle only through elements inside
  the modal, never escaping to the underlying page.
- `Escape` MUST close the modal, UNLESS the modal contains a dirty
  (unsaved-changes) form, in which case `Escape` triggers the same
  "Discard changes?" Tier-2-style confirmation used for the Cancel button,
  rather than silently closing.
- On close (by any method), focus MUST return to the element that triggered
  the modal's opening, so keyboard users are not left disoriented.
- Clicking the scrim closes the modal under the same dirty-check rule as
  `Escape`, EXCEPT for Tier 3 confirmation dialogs and any modal explicitly
  marked non-dismissible (e.g. a mandatory terms-acceptance modal), where
  scrim-click and `Escape` are both disabled and only the explicit
  Cancel/Close button works.

## Stacking rules

- At most ONE modal may be open at a time. Triggering a second modal from
  within an open modal (e.g. a confirmation dialog for an action taken
  inside a form modal) MUST replace the first modal's content in place
  (the confirmation renders in the same overlay container, same size class
  or smaller) rather than stacking a second overlay/scrim on top of the
  first.
- A drawer MAY open a `sm` confirmation modal on top of it (this is the one
  permitted exception to strict single-overlay stacking) since a drawer is
  anchored to the edge and a small centered confirmation does not visually
  compound scrims in a disorienting way; a modal MUST NOT open a drawer, and
  a drawer MUST NOT open a second drawer.
- Closing the top-most overlay always returns focus and view state to
  exactly what was underneath it — no overlay close action may also
  dismiss anything beneath it.
