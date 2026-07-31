# Migration & Seeder Standards

## Migrations

- One logical change per migration file. Do not bundle unrelated schema
  changes into a single migration.
- Migrations are additive-first: adding a nullable column, adding a table, or
  adding an index are always safe to ship ahead of the application code that
  uses them.
- Destructive changes (dropping a column, renaming a column, tightening a
  constraint) follow the **expand → migrate → contract** pattern across at
  least two deploys:
  1. **Expand**: add the new column/table alongside the old one; application
     writes to both.
  2. **Migrate**: backfill data, flip reads to the new column, verified in
     production.
  3. **Contract**: a separate, later migration drops the old column, only
     after confirming no code path references it.
- Every migration has a working `down()` method except where a prior
  destructive migration has already made rollback meaningless (documented
  with a comment explaining why).
- Migrations never contain conditional logic based on environment (`if
  app()->environment('production')`) — schema must be identical across
  environments; use seeders/config for environment-specific data.
- Long-running migrations (backfills on large tables) are written as
  chunked, resumable jobs invoked from a migration or an Artisan command, not
  as a single unbounded `UPDATE`, to avoid locking production tables.

## Seeders

- Every product ships two seeder profiles: `DemoSeeder` (realistic,
  narratively coherent demo data — see the Demo Standard in
  [README.md](../../README.md): every dashboard must show meaningful data,
  every chart must render real trends, no `Lorem ipsum`) and
  `TestSeeder`/factories (minimal, deterministic data for automated tests).
- `DemoSeeder` output must be internally consistent: dates in chronological
  order, totals that sum correctly, relationships that make business sense
  (an invoice's customer exists, its line items sum to its total, its status
  history is plausible).
- Seeders are idempotent — re-running a seeder against a already-seeded
  database either no-ops or resets cleanly via `--fresh`, never duplicates
  data.
- Factories (`database/factories/`) exist for every model and are the
  building block both `DemoSeeder` and tests use — seeders compose
  factories with realistic relationships rather than hand-writing insert
  statements.

## Review requirements

A migration PR must state, in the PR description: whether it is
additive/destructive, whether it requires a backfill, and the expected lock
duration on the largest expected production table size. See
[pr-standards.md](./pr-standards.md).
