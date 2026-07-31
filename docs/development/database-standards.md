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
- `company_id` / `branch_id` (UUID, foreign key, indexed) on every table
  belonging to a product that models multi-company or multi-branch
  operation within its one deployment — see
  [localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).
  A product without multi-company/branch scope in its
  [`SPEC.md`](../products/) has no such column; there is no deployment-wide
  scoping column, because each deployed instance already belongs to exactly
  one business — see
  [single-tenant-deployment-model.md](../architecture/single-tenant-deployment-model.md).
- `created_at`, `updated_at` (timestamptz)
- `deleted_at` (timestamptz, nullable) on every table subject to soft delete
  per [rest-standards.md](./rest-standards.md#soft-delete-semantics) —
  which is every user-facing business entity by default

## Indexing

- Every foreign key column is indexed.
- Every column used in a `WHERE`, `ORDER BY`, or join in a shipped query path
  has a supporting index, verified by the query budget check in
  [performance-standards.md](../quality/performance-standards.md).
- Composite indexes are ordered by selectivity; on a product that models
  multi-company/multi-branch scoping, `company_id`/`branch_id` leads the
  composite index for lookups scoped to that context.
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
  key — scoped per deployment by default, or per company/branch on a
  product whose data model uses multi-company/multi-branch scoping (see
  [localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)).

## Schema change safety

Covered fully in
[migration-seeder-standards.md](./migration-seeder-standards.md); the
summary rule is additive-first, backward-compatible migrations, with
destructive changes split into an "expand, migrate data, contract" sequence
across separate deploys.

## Single-tenant at the schema level

Every product's schema models exactly one business's data, in one database,
with no `tenant_id` column and no tenant global scope anywhere — there is
nothing to scope against, because there is no second business's data in the
same running application. See
[single-tenant-deployment-model.md](../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).
A product whose spec requires multi-company/multi-branch operation within
that one deployment uses the `company_id`/`branch_id` scoping documented
above, per
[localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping) —
this is scoping within one business, not tenancy, and does not require the
cross-tenant isolation test category that a real multi-tenant system would.
