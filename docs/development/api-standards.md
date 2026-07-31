# API Standards

This document is the authority for every API a Zodize product exposes. See
also [rest-standards.md](./rest-standards.md) (URL/resource conventions),
[webhook-standards.md](./webhook-standards.md), and
[sdk-standards.md](./sdk-standards.md).

## Versioning

- Every API is versioned in the URL path: `/api/v1/...`. Breaking changes
  require a new version; non-breaking additive changes (new optional field,
  new endpoint) ship within the current version.
- A version is supported for a minimum of 18 months after the next version
  ships, with deprecation communicated via the `Sunset` HTTP header and the
  developer portal changelog (see [documentation-template.md](../templates/documentation-template.md)).

## Authentication

- Bearer token authentication (Laravel Sanctum-style personal access tokens)
  for first-party SPA/mobile clients and third-party API integrations alike.
- Tokens are scoped (see [rbac-permissions.md](../security/rbac-permissions.md))
  — a token is issued with an explicit ability list, never blanket account
  access by default.
- OAuth2 authorization-code flow is additionally supported for third-party
  app integrations distributed via the marketplace
  ([marketplace-architecture.md](../architecture/marketplace-architecture.md)).

## Request/response envelope

Every successful response:

```json
{
  "data": { "...": "..." },
  "meta": { "...": "pagination or contextual metadata" }
}
```

Every error response, with a consistent HTTP status code:

```json
{
  "error": {
    "code": "invoice.already_paid",
    "message": "This invoice has already been paid and cannot be modified.",
    "details": { "field": "amount", "reason": "..." }
  }
}
```

- `error.code` is a stable, machine-readable, namespaced string —
  integrations key off `error.code`, never off `error.message` (which is
  human-readable and localized).
- Validation errors return HTTP 422 with `error.details` as a map of
  field → list of messages.

## Pagination

- Cursor-based pagination is the default for all list endpoints
  (`?cursor=...&limit=...`), returned with `meta.next_cursor`. Offset-based
  (`?page=`) is permitted only for admin/reporting endpoints where total
  count display is required and dataset size is bounded.
- Default `limit` is 25, maximum is 100. Requests above the maximum are
  clamped, not rejected.

## Filtering and sorting

- Filtering: `?filter[status]=paid&filter[created_after]=2026-01-01`.
- Sorting: `?sort=-created_at` (`-` prefix for descending), comma-separated
  for multi-column.
- Field selection (sparse fieldsets): `?fields=id,name,status` supported on
  all list/detail endpoints returning large objects.

## Idempotency

Any non-idempotent mutating endpoint that a client might reasonably retry
(payments, order creation) must accept an `Idempotency-Key` header; the
server persists the key/response pair and replays the original response for
a duplicate key within a 24-hour window.

## Rate limiting

- Every API token is rate-limited (default: 300 requests/minute, configurable
  per plan tier). Responses include `X-RateLimit-Limit`,
  `X-RateLimit-Remaining`, `X-RateLimit-Reset`. A 429 response includes
  `Retry-After`.

## Webhooks

Domain events fan out to registered webhooks; see
[webhook-standards.md](./webhook-standards.md) for payload format, signing,
and retry policy.

## Documentation

Every API is specified in OpenAPI 3.1, generated from route/request/resource
annotations where tooling allows, and published via
[documentation-template.md](../templates/documentation-template.md). An
endpoint that exists but isn't in the OpenAPI spec is considered
undocumented and fails the [Definition of Done](../quality/definition-of-done.md).

## Change safety

CI runs a contract diff against the previous OpenAPI spec on every PR;
removing a field, changing a type, or removing an endpoint fails the build
unless the PR explicitly bumps the API version and updates the deprecation
schedule.
