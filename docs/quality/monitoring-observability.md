# Monitoring & Observability

> The required monitoring stack, dashboards, and alerting policy for every
> Zodize product. This is the operational instrumentation that the go-live
> gate in
> [`definition-of-production-ready.md`](./definition-of-production-ready.md#4-monitoring-and-alerting-configured)
> checks for, and the data source the DR runbook in
> [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#disaster-recovery-runbook-requirement)
> relies on for disaster detection.

## Required monitoring stack

Every product MUST run all four of the following, with no substitutions
that drop coverage:

- **Application Performance Monitoring (APM)**: distributed tracing across
  the request lifecycle described in
  [`../architecture/overview.md`](../architecture/overview.md#request-lifecycle) —
  HTTP request, database queries, cache calls, queue job dispatch and
  execution, external API calls — tagged with `request_id` so a single slow
  request can be traced end to end.
- **Error tracking** (Sentry-equivalent): every unhandled exception and
  every explicitly logged `error`/`critical` log entry is captured with full
  stack trace, request context, and authenticated user ID — with PII
  redaction applied per
  [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#pii-handling)
  before the event leaves the application (no raw request bodies containing
  password/card/SSN fields sent to the error tracker).
- **Uptime monitoring**: external, synthetic checks against the health check
  endpoint (below) and at least one critical user-facing page per product,
  polled from multiple geographic regions at 1-minute intervals, independent
  of the product's own infrastructure (so an infrastructure-wide outage is
  still detected and paged).
- **Log aggregation**: structured (JSON) application logs shipped centrally
  from every process type (web, queue worker, scheduler) with `request_id`
  and `user_id` fields on every log line where applicable, retained a
  minimum of 30 days searchable, 1 year in cold
  storage for financial-grade and healthcare-grade products to support
  incident investigation alongside the audit log in
  [`../security/audit-logging.md`](../security/audit-logging.md).

## Required dashboards

Every product MUST maintain, at minimum, the following dashboards, visible
to the engineering team and reviewed as part of the weekly operational
review:

| Dashboard | Contents |
|---|---|
| Error rate | 5xx rate and unhandled exception rate, overall and broken down by endpoint/module, with week-over-week comparison. |
| Latency | p50/p95/p99 response time per endpoint group (read/write/search per [`performance-standards.md`](./performance-standards.md#api-response-time-targets)), plotted against the target budget lines so a regression is visually obvious. |
| Queue depth | Per-queue (`high`/`default`/`low`/`notifications` per [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md#queue-standard)) pending job count and oldest-pending-job age, plus failed job count. |
| Database connections | Active connection count against the pool ceiling, slow query rate (>100ms per [`performance-standards.md`](./performance-standards.md#database-query-budget)), replication lag where applicable. |
| Infrastructure health | CPU/memory/disk per node class (web, queue worker), load balancer target health, cache (Redis) memory usage and eviction rate. |

Financial-grade products additionally maintain a **transaction integrity
dashboard** tracking ledger balance reconciliation status and audit log
hash-chain verification results per
[`../security/audit-logging.md`](../security/audit-logging.md#immutability-requirement).

## Alerting thresholds and on-call escalation

| Signal | Threshold | Severity |
|---|---|---|
| 5xx error rate | > 1% of requests over a 5-minute window | Page on-call immediately |
| API p95 latency | Exceeds the target in [`performance-standards.md`](./performance-standards.md#api-response-time-targets) for 10 consecutive minutes | Page on-call immediately |
| Health check failure | 2 consecutive failed checks from any monitoring region | Page on-call immediately |
| Queue depth (`high`) | Oldest pending job age > 2 minutes | Page on-call immediately |
| Queue depth (`default`/`low`) | Oldest pending job age > 30 minutes | Notify on-call channel, page if sustained past 1 hour |
| Failed job spike | > 10 failed jobs in 5 minutes on `high` or `notifications` | Page on-call immediately |
| Database connection pool | > 90% utilized for 5 minutes | Page on-call immediately |
| Disk usage | > 85% on any node | Notify on-call channel |
| Backup failure | Any scheduled backup job fails or does not complete within its expected window | Page on-call immediately |
| Certificate expiry | TLS certificate within 14 days of expiry | Notify on-call channel |
| Audit log hash-chain break (financial-grade only) | Any chain verification failure per [`../security/audit-logging.md`](../security/audit-logging.md#immutability-requirement) | Page on-call and Security lead immediately |

**Escalation policy**: a page unacknowledged within 5 minutes escalates to
the secondary on-call; unacknowledged within 15 minutes escalates to the
engineering lead for the product. Every paged incident MUST be triaged
within 15 minutes (acknowledge, assess severity, begin mitigation or
declare a disaster per
[`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#disaster-recovery-runbook-requirement)
if warranted). Every incident that pages on-call MUST have a written
post-incident summary within 3 business days, regardless of whether it
escalated further.

## Health check endpoint standard

- Every product MUST expose `GET /health` (unauthenticated, excluded from
  rate limiting) returning `200 OK` with a JSON body reporting the status of
  each critical dependency:

```json
{
  "status": "ok",
  "checks": {
    "database": "ok",
    "redis": "ok",
    "queue": "ok"
  },
  "version": "2026.07.14-abc1234"
}
```

- `status` MUST be `"degraded"` (still `200`, so the load balancer keeps the
  node in rotation for a non-fatal issue) or `"down"` (returns `503`,
  removing the node from load balancer rotation) based on the worst
  individual check result.
- The `database` check MUST perform a real lightweight query (not just
  "connection object exists"); the `redis` check MUST perform a real
  `PING`; the `queue` check MUST verify the queue connection is reachable,
  not that workers are running (worker liveness is covered by the queue
  depth dashboard/alert above, not the HTTP health check, to keep the
  endpoint fast).
- `version` MUST reflect the actual deployed build identifier (git SHA or
  release tag) so on-call can immediately confirm which release is live
  during an incident, without a separate lookup.
- The health check endpoint itself MUST respond in under 100ms; it is
  polled every 10-30 seconds by the load balancer and every 60 seconds by
  external uptime monitoring, and MUST NOT itself become a load bottleneck.

## Related standards

- [`performance-standards.md`](./performance-standards.md)
- [`ci-cd-standards.md`](./ci-cd-standards.md)
- [`definition-of-production-ready.md`](./definition-of-production-ready.md)
- [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md)
- [`../security/audit-logging.md`](../security/audit-logging.md)
- [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md)
