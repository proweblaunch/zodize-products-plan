# Caching, Queues & Events

> The runtime infrastructure standard every Zodize product implements
> within its own single deployment, underpinning the request lifecycle in
> [`overview.md`](./overview.md#request-lifecycle) and the per-product
> extension hook system in
> [`plugin-architecture.md`](./plugin-architecture.md#hook-and-event-points).

## Caching strategy

- **Redis is the preferred cache, session, and queue driver where the
  buyer's hosting supports it.** Because the reference deployment target is
  shared/VPS hosting a non-technical buyer controls (see
  [`overview.md`](./overview.md#deployment-topology-per-product-per-buyer)),
  a product MUST also support the `database` (or `file`) cache/session/queue
  driver as a documented fallback for buyers on hosting without Redis
  available, with a documented performance caveat rather than a hard
  requirement on Redis.
- Every product operates on exactly one business's data, so there is no
  cross-business cache key namespacing to worry about — a cache key is
  simply `{product}:{cache-key}`. Cache tagging (where the chosen driver
  supports it — `database`/`file` drivers do not support tags, Redis does)
  is used for grouped invalidation by resource type (e.g. all cached
  `invoices` list queries carry an `invoices` tag, so writing a new invoice
  invalidates `Cache::tags(['invoices'])->flush()`).
- **Invalidation pattern**: cache-aside (read-through) is the default —
  application code checks cache, falls back to the database on miss, and
  writes the result back with a TTL. Write paths invalidate the specific
  tags affected synchronously within the same request as the write, never
  relying on TTL expiry alone for correctness-sensitive data (e.g. a
  permission change per
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md) MUST
  invalidate the affected user's permission cache immediately, not wait out
  a TTL).
- Default TTLs: 5 minutes for list/aggregate query caches, 1 hour for
  rarely-changing reference data (e.g. the deployment's enabled
  gateway/plugin list per
  [`admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)),
  no caching for any `restricted`-classified data per
  [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-classification-levels)
  unless the cache store itself is encrypted at rest and access-scoped
  identically to the source data.

## Queue standard

- Every product defines exactly four named queues, processed by separate
  worker pools (where the buyer's hosting supports a persistent queue
  worker process — see below) so a backlog in one never starves another:

| Queue | Priority | Example jobs |
|---|---|---|
| `high` | Processed first, smallest worker-to-job latency target (under 5s median pickup) | Payment capture confirmation, MFA/OTP delivery, password reset email |
| `default` | Standard priority | Domain event side-effects, report generation, PDF export |
| `low` | Best-effort, may lag under load | Analytics aggregation, bulk data cleanup, non-urgent housekeeping |
| `notifications` | Dedicated pool so a notification-sending burst never delays `high`/`default` work | Email/SMS/push dispatch, webhook fan-out (see below) |

- **Hosting reality**: many buyers are on shared hosting without a
  persistent process supervisor. Every product MUST support the database
  queue driver processed via the scheduler (`schedule:run` triggering a
  short-lived `queue:work --stop-when-empty` pass each minute) as the
  default-safe configuration, with true persistent `queue:work` workers
  documented as the recommended upgrade for buyers on a VPS that supports
  long-running processes. A product's job code MUST NOT assume sub-minute
  processing latency is guaranteed.
- **Retry/backoff policy**: every job MUST declare `$tries` (default 3) and
  use exponential backoff (`backoff()` returning `[10, 60, 300]` seconds as
  the default progression) rather than immediate retry. Idempotency is
  mandatory for any job that is retried — jobs MUST be written so a
  duplicate execution (from a retry or an at-least-once delivery replay)
  does not double-charge, double-send, or double-post a ledger entry (see
  [`../standards/wallet-system.md`](../standards/wallet-system.md), which
  enforces this via the append-only `Transaction` pattern).
- **Failed job handling**: jobs exhausting `$tries` land in the
  `failed_jobs` table and MUST trigger an in-admin-panel alert (per
  [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md))
  for any `high` or `notifications` queue failure, visible to the buyer's
  own admin staff — there is no Zodize-operated on-call channel to page,
  since Zodize does not operate the buyer's deployment. Failed jobs are
  retained 30 days for manual inspection/replay from the admin panel, then
  pruned. A failed job affecting a financial transaction MUST also raise an
  audit log entry per
  [`../security/audit-logging.md`](../security/audit-logging.md).

## Event-driven architecture

- Every meaningful state change is modeled as a **domain event** (a
  Laravel Event class named in the past tense, e.g. `InvoicePaid`,
  `PatientAdmitted`, `TradeExecuted`), dispatched from the service/action
  class that performs the write, inside the same database transaction where
  practical so the event is never fired for a write that later rolls back.
- Domain events are the sanctioned mechanism for cross-module communication
  within the modular monolith (see
  [`overview.md`](./overview.md#modular-monolith-one-codebase-per-product)) —
  a module MUST NOT call another module's internals directly to react to
  its state changes; it registers a listener on the emitting module's
  public event instead.
- **Event listeners** default to queued (`ShouldQueue`), landing on the
  `default` queue unless the listener's own latency profile justifies
  `notifications` or `high`. Synchronous listeners are permitted only for
  in-request invariant enforcement (e.g. recalculating a derived total)
  that the response itself depends on, or as the fallback path when the
  deployment has no persistent queue worker (above).
- **Webhook fan-out**: outbound webhooks a buyer configures for their own
  integrations (and per-product plugin `hooks` subscriptions per
  [`plugin-architecture.md`](./plugin-architecture.md#hook-and-event-points))
  are driven off the same domain events via a generic `WebhookDispatcher`
  listener on the `notifications` queue. Each webhook delivery is signed
  (HMAC-SHA256 over the payload using a per-endpoint secret, sent as an
  `X-Zodize-Signature` header) and retried up to 5 times with exponential
  backoff; an endpoint failing 20 consecutive deliveries is auto-disabled
  and the buyer's admin is notified in-panel.

## Broadcasting standard

- Real-time UI updates (live dashboards, notification badges) are an
  **optional enhancement, not a requirement**, per
  [`overview.md`](./overview.md#deployment-topology-per-product-per-buyer) —
  the reference deployment target is shared/VPS hosting that may not support
  a persistent WebSocket process. Every product MUST provide a polling
  fallback (short-interval AJAX refresh) for any UI surface that would
  otherwise depend on a live socket connection.
- A product MAY additionally support Laravel's broadcasting layer over
  **Laravel Reverb** (self-hosted, Pusher-protocol compatible) or a managed
  Pusher-compatible service, for buyers on a VPS that supports it, without
  that support being load-bearing for the product's core functionality.
- Where broadcasting is enabled, every channel is a **private, authorized
  channel**, using a channel authorization callback that re-runs the same
  Policy check the equivalent HTTP endpoint would — a WebSocket subscription
  MUST NOT be a side channel that bypasses the authorization model in
  [`../security/authentication-authorization.md`](../security/authentication-authorization.md).
- Broadcast payloads follow the same data-classification rule as caching: a
  `restricted`-classified field is never broadcast in a payload the frontend
  doesn't already have a right to fetch via the authorized REST endpoint —
  broadcast a change notification (resource ID + event type) and let the
  client re-fetch.

## Related standards

- [`overview.md`](./overview.md)
- [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md)
- [`plugin-architecture.md`](./plugin-architecture.md)
- [`../standards/wallet-system.md`](../standards/wallet-system.md)
- [`../security/audit-logging.md`](../security/audit-logging.md)
- [`../quality/performance-standards.md`](../quality/performance-standards.md)
- [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)
