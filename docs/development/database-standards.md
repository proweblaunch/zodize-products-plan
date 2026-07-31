# Database Standards

## Engine and baseline

- Primary datastore: PostgreSQL (latest stable major version) for all new
  products. MySQL-compatible products may exist only where an ADR documents
  a specific reason (e.g. a required managed-hosting constraint).
- Redis for cache, session, and queue backing per
  [caching-queues-events.md](../architecture/caching-queues-events.md).
- Search: a dedicated search engine (Meilisearch/Typesense-class) for any
  product requiring [global search](../products/ZodiCore/SPEC.md) beyond
  simple `LIKE` queries — never full-text search bolted onto the primary
  OLTP database at scale.

## Naming conventions

- Tables: plural, snake_case (`purchase_orders`).
- Columns: snake_case, foreign keys as `<singular_table>_id`
  (`customer_id`), booleans prefixed `is_`/`has_` (`is_active`).
- Primary keys: UUID v7 (time-ordered) stored as `uuid`/`binary(16)`,
  exposed externally as-is — never expose sequential integer IDs externally
  per [rest-standards.md](./rest-standards.md#resource-representation).
  An internal auto-increment `bigint` may additionally exist purely for
  clustering/index locality if a product's scale justifies it, but is never
  the externally-facing identifier.

## Required columns on every table

- `id` (UUID, primary key)
- `tenant_id` (UUID, foreign key, indexed, not null) on every tenant-scoped
  table — see [multi-tenancy.md](../architecture/multi-tenancy.md)
- `created_at`, `updated_at` (timestamptz)
- `deleted_at` (timestamptz, nullable) on every table subject to soft delete
  per [rest-standards.md](./rest-standards.md#soft-delete-semantics) —
  which is every user-facing business entity by default

## Indexing

- Every foreign key column is indexed.
- Every column used in a `WHERE`, `ORDER BY`, or join in a shipped query path
  has a supporting index, verified by the query budget check in
  [performance-standards.md](../quality/performance-standards.md).
- Composite indexes are ordered by selectivity, `tenant_id` first for
  tenant-scoped lookups.
- Partial indexes are used for common filtered queries (e.g.
  `WHERE deleted_at IS NULL`).

## Constraints over convention

- Foreign keys are enforced at the database level, not only in application
  code.
- `NOT NULL` and `CHECK` constraints enforce invariants the application layer
  also validates — defense in depth, not either/or.
- Enum-like columns use a `CHECK` constraint or native enum type, never a
  free-text column validated only in PHP.

## Money, dates, and identifiers

- Monetary amounts: integer minor units (`bigint`) plus a `currency` column
  (ISO 4217) — never `decimal`/`float` for currency.
- Timestamps: always `timestamptz`, stored in UTC, converted to the viewing
  user's locale at the presentation layer only.
- Natural business identifiers (invoice numbers, order numbers) are a
  separate human-readable sequence column, distinct from the UUID primary
  key, scoped per tenant.

## Schema change safety

Covered fully in
[migration-seeder-standards.md](./migration-seeder-standards.md); the
summary rule is additive-first, backward-compatible migrations, with
destructive changes split into an "expand, migrate data, contract" sequence
across separate deploys.

## Multi-tenancy at the schema level

Single database, `tenant_id`-scoped rows is the default model (see
[multi-tenancy.md](../architecture/multi-tenancy.md)); every Eloquent model
on a tenant-scoped table uses a mandatory global scope, and cross-tenant
query leakage is covered by an automated test suite that runs against every
tenant-scoped table (see [testing-standards.md](./testing-standards.md)).
