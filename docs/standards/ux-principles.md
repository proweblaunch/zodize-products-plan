# UX Principles

These are the principles every Zodize product's interface must be judged
against. They are not aspirational — they are the tiebreaker whenever a
designer, engineer, or AI coding agent has to choose between two valid
implementations. When a specific pattern document (navigation, forms,
tables, etc.) is silent on a case, resolve it against these principles, in
the order they resolve conflicts as described in
["Resolving conflicts between principles"](#resolving-conflicts-between-principles) below.

## 1. Clarity over cleverness

Every screen MUST communicate what it is, what state it is in, and what the
user can do next, without requiring the user to hover, guess, or read a
tooltip to find out. A novel interaction pattern that is not documented
elsewhere in this handbook MUST NOT ship — if a product needs one, propose it
as an addition to [`components.md`](../design-system/components.md) first.

- Icon-only controls MUST have a visible text label unless the icon is one of
  the universally recognized set (search, close, more/kebab, notification
  bell) AND the control has an accessible `aria-label` and a tooltip on
  hover/focus with a minimum 400ms show delay.
- Ambiguous state (is this button disabled because I lack permission, or
  because a required field is empty?) MUST be resolved with a visible
  explanation — a tooltip on the disabled control, or inline helper text —
  never silence.

## 2. Speed as a feature

Zodize products are used by operators who repeat the same task dozens of
times a day (a bank teller opening account records, a hotel front desk
checking in guests). Perceived speed is a first-class requirement, not a
performance nice-to-have.

- Any action that triggers a network request MUST show a loading indicator
  within 100ms if the request has not resolved, per
  [`../design-system/motion-animation.md`](../design-system/motion-animation.md).
- Optimistic UI updates are REQUIRED for reversible, low-risk actions
  (toggling a switch, starring an item, reordering a list) — the UI updates
  immediately and reconciles with the server response, rolling back with a
  toast on failure. Optimistic updates MUST NOT be used for destructive or
  financial actions (see Principle 6).
- Every list, table, and search MUST support keyboard-driven navigation
  (see Principle 4) because keyboard interaction is faster than mouse
  interaction for repeat users.

## 3. Progressive disclosure

Screens MUST show the information and controls relevant to the user's
current task by default, and defer secondary detail behind an explicit
action (an expand toggle, a "Show advanced" link, a detail drawer, a second
tab). A screen that shows every field a record could ever have, all at once,
is a defect.

- Forms MUST group optional/advanced fields under a collapsed
  "Advanced settings" or "More options" disclosure, collapsed by default,
  per [`form-standards.md`](./form-standards.md#field-layout).
- Detail pages MUST lead with a summary (identity, status, key metrics) and
  push exhaustive history/audit data to a secondary tab, per
  [`page-layout-standards.md`](./page-layout-standards.md#detail-page-template).
- Dashboards MUST default to the 4-8 most decision-relevant widgets for the
  user's role, not every metric the system can compute, per
  [`dashboard-standards.md`](./dashboard-standards.md).

## 4. Keyboard-first power users

Every Zodize product is enterprise software used for hours a day by trained
staff, not a consumer app used casually. Mouse-only interaction is treated as
an accessibility and productivity defect, not just an accessibility one.

- Every product MUST implement the global command palette (`Cmd/Ctrl+K`) per
  [`navigation-standards.md`](./navigation-standards.md#command-palette).
- Every interactive control reachable by mouse MUST be reachable by `Tab`,
  operable by `Enter`/`Space`, and have a visible focus ring per
  [`../design-system/accessibility.md`](../design-system/accessibility.md).
- Data tables MUST support arrow-key row navigation and `Space` to toggle row
  selection once a row has focus, per
  [`table-standards.md`](./table-standards.md#keyboard-interaction).
- Modals and drawers MUST close on `Escape`, and forms inside them MUST
  submit on `Cmd/Ctrl+Enter`.

## 5. Consistency across products

A user who has used ZodiPOS MUST be able to sit down at ZodiHotel and
recognize the navigation shell, the table interactions, the modal behavior,
and the notification patterns without relearning them. Consistency is
enforced structurally: every product consumes the same navigation shell (
[`navigation-standards.md`](./navigation-standards.md)), the same page
templates ([`page-layout-standards.md`](./page-layout-standards.md)), and the
same component set ([`../design-system/components.md`](../design-system/components.md)).

- A product MUST NOT introduce a one-off table, modal, or form pattern for a
  single screen when an existing standard component covers the case.
- A product-specific deviation from a standard in this directory is only
  permitted when documented as an exception in that product's
  `docs/products/<product>/SPEC.md`, with the reason recorded — silent
  deviation is a defect caught in review, per
  [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md).

## 6. Forgiving and reversible actions

Users make mistakes; the interface's job is to make mistakes cheap to
recover from, not to prevent all mistakes through excessive friction.

- Non-destructive state changes (archiving, deactivating, unassigning) MUST
  be reversible from the UI without contacting support, and MUST show an
  "Undo" affordance in the confirming toast for at least 6 seconds, per
  [`notification-standards.md`](./notification-standards.md#toast-notifications).
- Destructive, irreversible actions (permanent delete, financial transfer,
  bulk data change affecting >1 record) MUST require an explicit
  confirmation dialog, per
  [`modal-standards.md`](./modal-standards.md#confirmation-dialogs), and MUST
  NOT use optimistic UI.
- Multi-step forms and long single-page forms MUST autosave drafts so a
  closed tab or session timeout does not destroy work, per
  [`form-standards.md`](./form-standards.md#autosave-and-drafts).

## Resolving conflicts between principles

The most common real conflict is **speed vs. safety**: optimistic,
frictionless interaction (Principle 2) against forgiving, reversible
interaction (Principle 6). Zodize resolves this with a fixed rule, not
per-product judgment:

1. Classify the action's **blast radius** first: does it affect one record
   the user owns, or multiple records, other users' data, money, or
   compliance state?
2. **Low blast radius, reversible** (toggle a preference, star an item,
   rename a draft): optimize for speed. Apply immediately, no confirmation,
   offer Undo.
3. **Low blast radius, irreversible** (send a single email, permanently
   delete one draft): optimize for safety with the lightest possible
   friction — a single confirmation dialog, no typed confirmation.
4. **High blast radius, or financial, or irreversible at scale** (bulk
   delete, funds transfer, permission grant, tenant-wide setting change):
   optimize for safety with maximum friction — a confirmation dialog that
   restates exactly what will change and, for the highest-risk cases,
   requires the user to type a confirmation phrase. See
   [`modal-standards.md`](./modal-standards.md#destructive-action-tiers) for
   the exact tiering.

Speed never wins over safety once an action crosses into "high blast
radius." Clarity (Principle 1) always wins over both — an action that is
fast and reversible but not understandable to the user is still a defect.

## Roadmap

- A shared usability-testing protocol for validating these principles against
  real operator workflows per product vertical is planned but not yet
  written; until it exists, product teams MUST self-review against this
  document during design review.
