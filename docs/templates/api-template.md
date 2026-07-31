# API Template

This document is the concrete scaffold and checklist for the API surface
every Zodize product exposes. The authoritative rules for API design are in
[`../development/api-standards.md`](../development/api-standards.md); this
document specifies what a module's controllers must produce to comply. A
product customizes its own endpoints and resources; it does not customize
versioning scheme, envelope shape, error format, pagination format, or query
param conventions.

## Versioning

- Every endpoint MUST be served under `/api/v{n}` (e.g. `/api/v1/invoices`).
  Unversioned API routes MUST NOT exist.
- A breaking change (removed field, changed type, changed required-ness,
  removed endpoint) MUST ship as a new version, never as a silent change to
  an existing version.
- A product MUST support at least the current version and the immediately
  prior version simultaneously for a minimum deprecation window defined in
  [`../development/api-standards.md`](../development/api-standards.md).

## Authentication

- All non-public endpoints require a Bearer token issued via the
  `api_tokens` table (see [database-template.md](./database-template.md)),
  Sanctum-style: `Authorization: Bearer {token}`.
- Tokens are scoped by `abilities` (permission slugs); a controller action
  MUST declare which ability it requires and the framework MUST reject a
  request whose token lacks it with a `403` in the standard error format.
- Public/unauthenticated endpoints MUST be explicitly allow-listed in
  `routes/api.php`, never the default.

## Standard response envelope

Every successful response MUST use this envelope:

```json
{
  "data": { },
  "meta": { }
}
```

- `data` — the resource or array of resources.
- `meta` — optional; present for paginated responses (see below) or when an
  endpoint returns supplementary information (e.g. rate-limit headroom).
- A single-resource response MUST NOT wrap `data` in an extra named key
  (e.g. `{"data": {"invoice": {...}}}` is incorrect — it is `{"data": {...}}`).

## Standard error format

Every error response MUST use this envelope, with an appropriate HTTP status
code:

```json
{
  "error": {
    "code": "invoice_not_found",
    "message": "The requested invoice could not be found.",
    "details": {}
  }
}
```

- `code` — a stable, machine-readable snake_case identifier; MUST NOT change
  across releases once published (it is part of the public contract).
- `message` — a human-readable, non-internal description; MUST NOT leak
  stack traces, SQL, or file paths.
- `details` — optional structured validation errors, keyed by field for
  `422` responses (e.g. `{"email": ["The email field is required."]}`).
- Every module's exception handler MUST translate framework exceptions
  (`ValidationException`, `AuthorizationException`, `ModelNotFoundException`)
  into this format; a raw Laravel error page or default JSON exception
  response reaching an API client is a defect.

## Pagination

- List endpoints MUST use cursor or page-based pagination (page-based is the
  default unless the resource has real-time insertion under load, per
  [`../development/api-standards.md`](../development/api-standards.md)) and
  MUST return pagination metadata under `meta`:

```json
{
  "data": [ ],
  "meta": {
    "current_page": 1,
    "per_page": 25,
    "total": 142,
    "last_page": 6
  }
}
```

- Default `per_page` is 25; maximum allowed `per_page` is 100. A request for
  more MUST be clamped, not rejected.
- List endpoints MUST NOT return unbounded results with no pagination as a
  "convenience" default.

## Filtering and sorting query param convention

- Filtering: `?filter[field]=value` (exact match) or `?filter[field][op]=value`
  for operators (`gte`, `lte`, `like`) — a controller MUST allow-list which
  fields are filterable; filtering on an unlisted field returns a `400` in
  the standard error format, not a silent no-op.
- Sorting: `?sort=field` (ascending) or `?sort=-field` (descending), with
  `?sort=field1,-field2` for multi-field sort. Unsortable-field requests
  return a `400`.
- Sparse fieldsets (optional, where supported): `?fields=field1,field2`.
- Relationship inclusion (optional, where supported): `?include=relation1,relation2`,
  never eager-loaded by default for expensive relations.

## Scaffold checklist per endpoint

Every new endpoint MUST satisfy this checklist before merge:

- [ ] Served under `/api/v{n}`.
- [ ] Requires an explicit ability, or is explicitly allow-listed as public.
- [ ] Success responses use the standard envelope.
- [ ] Error responses use the standard error format for every failure mode
      (validation, not found, unauthorized, forbidden, rate-limited).
- [ ] List endpoints are paginated with standard `meta`.
- [ ] Filterable/sortable fields are allow-listed, not open.
- [ ] Documented in the OpenAPI spec per
      [documentation-template.md](./documentation-template.md) (auto-generated,
      not hand-written).
- [ ] Covered by feature tests per [testing-template.md](./testing-template.md),
      including an authorization-denial case.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the base API middleware stack (auth, rate limiting,
tenant resolution), the standard envelope/error response formatters, the
pagination trait, and the filter/sort query parser.

A product customizes: its own resource endpoints, filterable/sortable field
allow-lists per resource, and resource-specific response shapes within the
`data` key. A product MUST NOT introduce a different envelope or error
format for any endpoint.
