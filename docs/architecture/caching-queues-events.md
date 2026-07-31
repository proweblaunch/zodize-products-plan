# Caching, Queues & Events

> The shared runtime infrastructure standard for every Zodize product,
> underpinning the request lifecycle in
> [`overview.md`](./overview.md#request-lifecycle) and the plugin hook system
> in [`plugin-architecture.md`](./plugin-architecture.md#hook-and-event-points).

## Caching strategy

- **Redis is the standard cache, session, and queue driver** for every
  product — no product may use the `file` or `database` cache/session
  driver in production; local `array` driver is permitted only in the test
  environment.
- Every cache key MUST be namespaced by tenant:
  `{product}:{tenant_id}:{cache-key}` (or use Laravel's cache tagging with a
  `tenant:{tenant_id}` tag), so that a cache flush or invalidation for one
  tenant can never affect another, and so tenant deprovisioning (see
  [`multi-tenancy.md`](./multi-tenancy.md#tenant-provisioning-and-deprovisioning-lifecycle))
  can flush a tenant's cache footprint precisely via tag.
- Cache tagging is used for grouped invalidation: e.g. all cached queries
  for a tenant's `invoices` list carry both a `tenant:{id}` tag and an
  `invoices` tag, so writing a new invoice invalidates
  `Cache::tags(["tenant:{$id}", 'invoices'])->flush()` without a broad
  tenant-wide flush.
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
  rarely-changing reference data (e.g. a tenant's enabled plugin list,
  custom domain lookup per
  [`multi-tenancy.md`](./multi-tenancy.md#tenant-identification)), no
  caching for any `restricted`-classified data per
  [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-classification-levels)
  unless the cache store itself is encrypted at rest and access-scoped
  identically to the source data.

## Queue standard

- Every product defines exactly four named queues, processed by separate
  worker pools so a backlog in one never starves another:

| Queue | Priority | Example jobs |
|---|---|---|
| `high` | Processed first, smallest worker-to-job latency target (under 5s median pickup) | Payment capture confirmation, MFA/OTP delivery, password reset email |
| `default` | Standard priority | Domain event side-effects, report generation, PDF export |
| `low` | Best-effort, may lag under load | Analytics aggregation, bulk data cleanup, non-urgent housekeeping |
| `notifications` | Dedicated pool so a notification-sending burst never delays `high`/`default` work | Email/SMS/push dispatch, webhook fan-out (see below) |

- **Retry/backoff policy**: every job MUST declare `$tries` (default 3) and
  use exponential backoff (`backoff()` returning `[10, 60, 300]` seconds as
  the default progression) rather than immediate retry, to avoid
  thundering-herd retries against a degraded downstream dependency.
  Idempotency is mandatory for any job that is retried — jobs MUST be
  written so a duplicate execution (from a retry or an at-least-once
  delivery replay) does not double-charge, double-send, or double-post a
  ledger entry (financial-grade products enforce this via an idempotency
  key column on the target write).
- **Failed job handling**: jobs exhausting `$tries` land in the
  `failed_jobs` table and MUST trigger an alert to the on-call channel per
  [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)
  for any `high` or `notifications` queue failure; `default`/`low` failures
  surface on a daily digest. Failed jobs are retained 30 days for manual
  inspection/replay, then pruned. A failed job affecting a financial
  transaction MUST also raise an audit log entry per
  [`../security/audit-logging.md`](../security/audit-logging.md).

## Event-driven architecture

- Every meaningful state change is modeled as a **domain event** (a
  Laravel Event class named in the past tense, e.g. `InvoicePaid`,
  `PatientAdmitted`, `TradeExecuted`), dispatched from the service/action
  class that performs the write, inside the same database transaction where
  practical so the event is never fired for a write that later rolls back.
- Domain events are the sanctioned mechanism for cross-module communication
  within the modular monolith (see
  [`overview.md`](./overview.md#modular-monolith-not-microservices)) — a
  module MUST NOT call another module's internals directly to react to its
  state changes; it registers a listener on the emitting module's public
  event instead.
- **Event listeners** default to queued (`ShouldQueue`), landing on the
  `default` queue unless the listener's own latency profile justifies
  `notifications` or `high`. Synchronous listeners are permitted only for
  in-request invariant enforcement (e.g. recalculating a derived total)
  that the response itself depends on.
- **Webhook fan-out**: tenant-configured outbound webhooks (and plugin
  `hooks` subscriptions per
  [`plugin-architecture.md`](./plugin-architecture.md#hook-and-event-points))
  are driven off the same domain events via a generic `WebhookDispatcher`
  listener on the `notifications` queue. Each webhook delivery is signed
  (HMAC-SHA256 over the payload using a per-tenant-per-endpoint secret,
  sent as an `X-Zodize-Signature` header) and retried up to 5 times with
  exponential backoff; an endpoint failing 20 consecutive deliveries is
  auto-disabled and the tenant Admin is notified.

## Broadcasting standard

- Real-time UI updates (live dashboards, notification badges, collaborative
  editing indicators) use Laravel's broadcasting layer over **Laravel
  Reverb** as the standard self-hosted WebSocket server (Pusher-protocol
  compatible, so a product may substitute a managed Pusher-compatible
  service where self-hosting Reverb is not viable for a given deployment,
  without changing application code).
- Every broadcast channel is a **private, tenant-scoped channel**
  (`private-tenant.{tenant_id}.{resource}`), authorized via a channel
  authorization callback that re-runs the same Policy check the equivalent
  HTTP endpoint would — a WebSocket subscription MUST NOT be a side channel
  that bypasses the authorization model in
  [`../security/authentication-authorization.md`](../security/authentication-authorization.md).
- Broadcast payloads follow the same data-classification rule as caching:
  a `restricted`-classified field is never broadcast in a payload the
  frontend doesn't already have a right to fetch via the authorized REST
  endpoint — broadcast a change notification (resource ID + event type) and
  let the client re-fetch, rather than pushing raw sensitive data over the
  socket, where the re-fetch is the only place a full authorization check
  is guaranteed to run.

## Related standards

- [`overview.md`](./overview.md)
- [`multi-tenancy.md`](./multi-tenancy.md)
- [`plugin-architecture.md`](./plugin-architecture.md)
- [`../security/audit-logging.md`](../security/audit-logging.md)
- [`../quality/performance-standards.md`](../quality/performance-standards.md)
- [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)
