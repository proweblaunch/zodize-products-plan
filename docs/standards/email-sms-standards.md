# Email, SMS, and Push Standards

This document defines the standards for the three out-of-band delivery
channels — transactional email, SMS, and push notification — and the shared
notification-event architecture that fans a single trigger out to all
channels, including the in-app surfaces defined in
[`notification-standards.md`](./notification-standards.md).

## Transactional email standards

- Every transactional email MUST use the single shared branded template
  defined in
  [`../design-system/components.md`](../design-system/components.md)'s
  email section: a header with the sending product's logo/wordmark, a
  single-column body constrained to 600px, the message content, a primary
  call-to-action button (when applicable) using the design system's
  primary button styling adapted for email-client-safe CSS, and a fixed
  footer with the sending product name, company postal address (required
  for CAN-SPAM/GDPR compliance), and the unsubscribe/preferences link.
- A product MUST NOT design a one-off email template per email type —
  every transactional email (welcome, password reset, invoice receipt,
  approval request, alert) uses the same shell with only the body content
  and CTA changing.
- Every email MUST be sent with BOTH an HTML part and a plain-text
  fallback part (`multipart/alternative`), generated from the same source
  content so they never drift out of sync. The plain-text version omits
  styling but preserves all information and the CTA URL as a plain link.
- Subject lines MUST be specific and non-generic ("Your ZodiBank statement
  for June 2026 is ready", not "Notification from ZodiBank") so the email
  is scannable in an inbox without opening it.
- **Unsubscribe / preference link**: every email classified as anything
  other than strictly required transactional (security alerts, password
  resets, OTP, legal notices) MUST include a one-click unsubscribe or
  "Manage email preferences" link in the footer, per CAN-SPAM/GDPR/CASL
  requirements. Purely transactional, account-critical emails (password
  reset, security alert, receipt for a completed action) are exempt from
  the unsubscribe requirement but MUST still link to the preferences
  center for adjusting non-critical categories.
- Emails MUST render correctly (no broken layout) in the major email
  clients Zodize supports: Gmail (web/app), Outlook (desktop/web),
  Apple Mail. Table-based layout with inlined CSS is REQUIRED for the
  shared template given Outlook's rendering constraints.

## SMS standards

- SMS is reserved for **high-priority, time-sensitive, and security-related**
  use cases only: one-time passcodes (OTP)/two-factor codes, critical
  security alerts (new device login, password changed), and operationally
  urgent alerts the receiving product's `SPEC.md` explicitly designates as
  SMS-eligible (e.g. a fraud-hold alert in ZodiBank, a same-day appointment
  reminder in ZodiMed). SMS MUST NOT be used for marketing, general
  updates, or any `low`/`normal` priority notification category per the
  priority model in
  [`notification-standards.md`](./notification-standards.md#notification-data-model).
- **Opt-in requirement**: a user MUST explicitly provide and verify a phone
  number and opt in to SMS before any SMS is sent, except OTP/2FA codes
  sent as part of a flow the user actively initiated (e.g. they entered
  their phone number specifically to receive a login code) — that
  transactional exchange does not require separate marketing-style opt-in
  since the user directly triggered it.
- **Character limits**: messages MUST fit within a single SMS segment
  (160 characters for GSM-7 encoding, 70 for messages containing non-GSM-7
  characters such as most non-Latin scripts or emoji) whenever possible.
  Messages that cannot be shortened to fit MAY concatenate to a maximum of
  2 segments; beyond that, the message MUST be shortened to a summary plus
  a short link to the full detail in-app, since long concatenated SMS is
  both costly and frequently mis-delivered by carriers.
- Every SMS MUST identify the sending product by name in the first segment
  (e.g. "ZodiBank: Your code is 482913...") since recipients receive SMS
  from many senders and have no visual branding cue.
- Marketing/promotional SMS (where a product's `SPEC.md` explicitly enables
  it) MUST include opt-out instructions ("Reply STOP to unsubscribe") in
  every message and MUST honor STOP replies within the carrier-mandated
  window, handled by the shared notification-event architecture's SMS
  channel adapter (below).

## Push notification standards

- Push notifications follow the same priority-gated model as SMS: `high`
  and `critical` priority categories are push-eligible by default; `low`/
  `normal` categories are push-eligible only if the user has explicitly
  enabled push for that category in their notification preferences (per
  [`notification-standards.md`](./notification-standards.md#notification-preferences)).
- A push notification's payload MUST include a deep link (`action_url` from
  the shared notification data model) so tapping it opens the relevant
  record directly rather than the product's generic home screen.
- Push MUST degrade gracefully when the user has no push-capable session
  registered (no mobile app installed / browser push not granted) — the
  notification-event fanout (below) does not fail or retry indefinitely in
  that case, it simply skips the push channel for that recipient.

## The shared notification-event architecture

Zodize does not send in-app, email, SMS, and push notifications as four
independently-triggered actions. Every notification originates from a
single **domain event** (e.g. `invoice.overdue`, `approval.requested`,
`login.new_device`) published to the event/queue infrastructure defined in
[`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md).
A single **notification dispatcher** service consumes that event and fans
it out to every eligible channel:

```
Domain event published
        │
        ▼
Notification dispatcher
  ├─ resolve recipient(s) + their per-category channel preferences
  ├─ resolve category + priority from the event type
  └─ for each eligible channel, enqueue a channel-specific job:
        ├─ In-app  → writes the notification-center record + real-time
        │            broadcast (notification-standards.md#data-model)
        ├─ Email   → renders the shared template, sends via the email
        │            provider adapter
        ├─ SMS     → renders the SMS copy, sends via the SMS provider
        │            adapter (only if priority/opt-in eligible)
        └─ Push    → renders the push payload, sends via the push
                     provider adapter (only if eligible + registered)
```

- This architecture MUST be implemented once, centrally, per
  [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md),
  and consumed by every product — a product MUST NOT implement its own
  parallel email-sending or SMS-sending path outside the dispatcher, since
  that would bypass the shared preference model, the unsubscribe/opt-out
  handling, and the `source_event_id` traceability that ties all four
  channel outputs back to one originating event.
- Each channel job is independently retryable and independently failable —
  a failed email send MUST NOT block or delay the in-app notification, and
  vice versa. Failures are logged against the `source_event_id` for
  observability, per the incident/observability standards in
  `docs/quality/`.
- A product's `SPEC.md` MUST enumerate its domain events that trigger
  notifications, each event's default category/priority, and which
  channels it is eligible for (some events are in-app-only by design, e.g.
  "comment added to a record you're watching," which does not warrant
  email or SMS by default).

## Roadmap

- A self-service template preview/test-send tool for product teams to
  validate the shared email template with real content before launch is
  planned but not yet built; until then, teams MUST manually verify
  rendering in the supported clients listed above before shipping a new
  email type.
