# Notification Standards

This document defines the three in-app notification surfaces (toast,
notification center, banner), the shared notification data model, real-time
delivery expectations, and per-user notification preferences. Cross-channel
delivery (email/SMS/push fanout from the same underlying event) is defined
in [`email-sms-standards.md`](./email-sms-standards.md), which this
document's data model feeds.

## Toast vs. notification center vs. banner

Zodize products have exactly three in-app notification surfaces, each with
a distinct purpose. A product MUST NOT invent a fourth.

| Surface | Persistence | Scope | Use for |
|---|---|---|---|
| Toast | Transient, auto-dismisses | Reaction to the current user's own just-taken action | Immediate feedback: "Invoice created", "Undo" prompts, action confirmations, action failures. |
| Notification center | Persistent until read/dismissed | Events relevant to the user, possibly triggered by someone/something else | Assignment, mentions, approvals needed, background job completion, another user's activity on a shared record. |
| Banner | Persistent until resolved or dismissed, system-wide | Conditions affecting the whole business or the whole product, not one user's action | Maintenance windows, licensing/plan issues, degraded service, required actions blocking the product (e.g. "Verify your email to continue"). |

### Toast (transient)

- Appears in a fixed position (bottom-right on desktop, bottom-center on
  mobile, above any sticky footer action bar) and stacks vertically (newest
  on top) when multiple toasts fire in quick succession, max 3 visible
  simultaneously with older ones queued.
- Auto-dismisses after 5 seconds for informational/success toasts, 8
  seconds for toasts with an action button (e.g. "Undo"), and does NOT
  auto-dismiss for error toasts — the user must dismiss those explicitly,
  since an error the user didn't get to read is worse than a transient one
  they did.
- Types: success (checkmark icon, success color accent), error (alert icon,
  danger color accent), info (info icon, neutral accent), warning (warning
  icon, warning color accent). Each MUST show a one-line message; an
  optional single action button (e.g. "Undo", "View") and a manual close
  "×" are permitted, but a toast MUST NOT contain a form or multiple
  actions — that belongs in the notification center or a modal.

### Notification center (persistent)

- Accessed via the bell icon in the top bar, per
  [`navigation-standards.md`](./navigation-standards.md#top-bar), opening a
  right-anchored panel (drawer-like, per
  [`modal-standards.md`](./modal-standards.md#modal-vs-drawer-vs-full-page))
  listing notifications newest-first, grouped by day ("Today", "Yesterday",
  "Earlier").
- Each entry shows: an icon or avatar representing the source, the message
  text, a relative timestamp, an unread indicator (filled dot), and,
  where applicable, a direct action link that navigates to the relevant
  record.
- Tabs at the top of the panel: "All" and "Unread" (default: whichever the
  user last selected, persisted per user). A "Mark all as read" action sits
  in the panel header.
- Clicking a notification marks it read and navigates to its action link in
  one action; a separate small "✓" control on hover/focus marks it read
  without navigating, for triaging without leaving the current page.
- The bell icon's badge shows the unread count (99+ cap, per
  [`navigation-standards.md`](./navigation-standards.md#sidebar)) and MUST
  update in real time (below) without requiring a page refresh.

### Banner (system-wide)

- Renders as a full-width strip at the very top of the content region,
  above the page header, pushing content down rather than overlaying it
  (never obscures the top bar or sidebar).
- At most ONE banner is visible at a time; if multiple system conditions
  are active, they are prioritized (blocking/required-action banners over
  degraded-service banners over informational banners) and only the
  highest-priority one shows, with a small "1 of 3" indicator and
  next/previous arrows if more than one is queued and none is dismissible
  without action.
- Blocking banners (e.g. "Your trial has expired — add a payment method to
  continue") MUST NOT have a dismiss control; they persist until the
  underlying condition is resolved. Non-blocking informational banners
  (e.g. scheduled maintenance notice) show a dismiss "×" that hides them
  for that user until the next occurrence of that specific banner ID.

## Notification data model

Every notification, regardless of which surface(s) it renders on, is backed
by one record with this shape:

- `id` — unique identifier.
- `recipient_user_id` — the user this notification is for (notifications
  are always scoped to one user, never broadcast without fanout to
  individual recipient rows).
- `category` — a fixed enum per product domain (e.g. `approval_required`,
  `mention`, `assignment`, `system`, `record_updated`, `job_completed`).
  Categories drive both iconography and the per-user preference granularity
  described below.
- `priority` — `low` | `normal` | `high` | `critical`. Priority determines
  default surface routing: `critical` MAY also trigger a banner or a toast
  in addition to the notification center entry; `low`/`normal` populate the
  notification center only, without an additional toast, to avoid
  over-interrupting the user for routine events.
- `title` and `body` — the rendered message; `title` is required, `body` is
  optional supporting detail shown when the notification is expanded.
- `action_url` — optional deep link to the relevant record/screen.
- `read_at` — nullable timestamp; null means unread.
- `created_at` — used for ordering and relative-time display.
- `source_event_id` — foreign key to the originating domain event, per the
  shared notification-event architecture in
  [`email-sms-standards.md`](./email-sms-standards.md#the-shared-notification-event-architecture),
  so an in-app notification, an email, and an SMS can all trace back to the
  single event that triggered them.

## Real-time delivery

- Notification-center entries and bell-badge counts MUST be delivered via
  the real-time broadcast channel defined in
  [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md)
  (WebSocket/broadcast), not polling — a new notification MUST appear
  without the user refreshing the page, within 2 seconds of the triggering
  event under normal load.
- If the real-time connection drops, the client MUST reconcile on
  reconnect by fetching any notifications created since the last known
  timestamp, rather than silently missing them.
- `critical` priority notifications that also trigger a toast MUST fire the
  toast through the same real-time channel, not a separate polling
  mechanism, to guarantee the toast and the notification-center entry never
  desynchronize.

## Notification preferences

- Every user has a preferences screen (linked from the user menu, per
  [`navigation-standards.md`](./navigation-standards.md#top-bar)) listing
  every notification `category` for the product, with independent toggles
  per delivery channel: In-app (notification center — cannot be fully
  disabled for `critical` priority categories, only muted for
  `low`/`normal`), Email, SMS, Push (per
  [`email-sms-standards.md`](./email-sms-standards.md)).
- Preferences default to: In-app ON for all categories, Email ON for
  `normal`+ priority categories, SMS ON only for categories explicitly
  flagged as `critical`-eligible (e.g. security alerts, OTP — see
  [`email-sms-standards.md`](./email-sms-standards.md#sms-standards)), Push
  ON if the user has an active push-capable session.
- A user MUST NOT be able to fully disable `critical` in-app notifications
  (e.g. account security alerts) — the toggle for those categories is
  restricted to channel selection (email/SMS/push on or off) while the
  in-app entry always persists in the notification center, since it carries
  no interruption cost by itself.
