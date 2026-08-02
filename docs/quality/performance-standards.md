# Performance Standards

> Concrete performance budgets every Zodize product MUST meet, verified by
> the load testing requirement in
> [`definition-of-production-ready.md`](./definition-of-production-ready.md#3-load-testing-passed-at-defined-targets)
> and enforced continuously by the monitoring stack in
> [`monitoring-observability.md`](./monitoring-observability.md).

## API response time targets

Measured server-side, from request received to response sent, excluding
client network latency, under normal production load:

| Operation type | p95 target | p99 target |
|---|---|---|
| Read (`GET` list/show endpoints) | < 200ms | < 500ms |
| Write (`POST`/`PUT`/`PATCH`/`DELETE`) | < 500ms | < 1000ms |
| Search/filter endpoints over indexed columns | < 300ms | < 700ms |
| Report/export generation (synchronous, small) | < 1000ms | < 2000ms |
| Any operation exceeding these targets | MUST be moved to a background job (see below) | — |

Financial-grade products (ZodiBank, ZodiTrade, ZodiXchange) tighten the read
target to p95 < 150ms for any endpoint on the primary trading/transaction
path, given latency-sensitive financial workflows; this MUST be documented
explicitly in the product's own performance section if it applies to
additional endpoints beyond this default.

## Frontend Core Web Vitals targets

Measured via field data (real user monitoring) at the 75th percentile,
consistent with Google's "Good" thresholds, for every Zodize product's
authenticated application shell and public marketing pages alike:

| Metric | Target |
|---|---|
| Largest Contentful Paint (LCP) | < 2.5s |
| Interaction to Next Paint (INP) | < 200ms |
| Cumulative Layout Shift (CLS) | < 0.1 |
| Time to First Byte (TTFB) | < 600ms |

- Compiled JS/CSS bundles MUST be split per page/module at minimum (no
  single monolithic bundle loaded on every page); any bundle exceeding
  250KB gzipped MUST be justified in the PR description or split further.
- Plugin-contributed frontend assets are lazy-loaded only when the plugin is
  enabled for the deployment, per
  [`../architecture/plugin-architecture.md`](../architecture/plugin-architecture.md#migrations-routes-and-views),
  and MUST NOT count against the base application's bundle budget.
- Images MUST be served in a modern format (WebP/AVIF with fallback) and
  responsively sized; any image above 200KB in a critical rendering path
  requires an explicit justification comment.

## Database query budget

- **N+1 prevention is mandatory**: any Eloquent relationship accessed in a
  loop MUST be eager-loaded (`with()`/`load()`). CI's static analysis stage
  (see [`ci-cd-standards.md`](./ci-cd-standards.md#pipeline-stages)) MUST
  run an N+1 detection tool (e.g. a query-count assertion package) in the
  feature test suite; a feature test that triggers an N+1 pattern fails the
  build.
- **Query count budget per request**: a single request MUST NOT execute more
  than 20 database queries for a read endpoint or 30 for a write endpoint
  under normal conditions (excluding queries inside dispatched background
  jobs, which are budgeted separately). Feature tests for high-traffic
  endpoints MUST assert an explicit query count ceiling using Laravel's
  query-log assertions, so a regression is caught in CI rather than in
  production.
- Every `company_id`/`branch_id`-filtered query on a table expected to
  exceed 100,000 rows per company/branch (on a product with multi-company/
  multi-branch scoping) MUST have a covering composite index per
  [`../development/database-standards.md`](../development/database-standards.md#single-tenant-at-the-schema-level);
  the database standards in [`../development/`](../development/) define the
  exact indexing conventions this inherits from.
- Slow query logging MUST be enabled in production with a 100ms threshold,
  feeding the dashboards in
  [`monitoring-observability.md`](./monitoring-observability.md#required-dashboards).

## Pagination requirement

- Any list endpoint or UI list view capable of returning more than 50 items
  MUST be paginated — cursor-based pagination is the default for
  high-volume, frequently-appended resources (e.g. audit logs per
  [`../security/audit-logging.md`](../security/audit-logging.md#audit-log--activity-timeline-ui),
  transaction feeds), offset-based (Laravel's standard `paginate()`) is
  acceptable for smaller, stable resource lists.
- Default page size is 25; maximum client-requestable page size is 100. A
  request for a larger page size MUST be clamped server-side, not rejected
  silently with an unbounded result.
- Every paginated API response MUST include a consistent envelope (`data`,
  `meta.current_page`/`meta.next_cursor`, `meta.total` where computable
  without a full table scan, `links.next`) per the API standard in
  [`../development/`](../development/).
- Infinite-scroll and "load more" UI patterns MUST use the same underlying
  paginated endpoint as the paged view — no separate unpaginated "give me
  everything" endpoint is permitted for any list-shaped resource.

## Background job threshold

- Any operation whose synchronous execution time is expected to exceed 2
  seconds under normal conditions (report generation, bulk export, image/
  video processing, third-party API calls with unpredictable latency, bulk
  email/notification sends) MUST be dispatched as a queued job per
  [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md#queue-standard)
  rather than run inline in the HTTP request-response cycle.
- The triggering endpoint MUST return immediately (`202 Accepted` with a job/
  export ID) and the frontend MUST poll or subscribe (via the broadcasting
  standard in
  [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md#broadcasting-standard))
  for completion rather than holding the HTTP connection open.
- Exceeding the 2-second threshold in production for an endpoint not
  already backgrounded is treated as a performance defect and MUST be
  filed and prioritized, not silently tolerated because "it's rare."

## Related standards

- [`monitoring-observability.md`](./monitoring-observability.md)
- [`ci-cd-standards.md`](./ci-cd-standards.md)
- [`definition-of-production-ready.md`](./definition-of-production-ready.md)
- [`../architecture/caching-queues-events.md`](../architecture/caching-queues-events.md)
- [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)
