# Coding Standards — PHP & Laravel

## Baseline

- PHP: latest stable LTS-equivalent version supported by the current Laravel
  LTS release at time of product start; pin the exact version in
  `composer.json` `"platform"` and in CI.
- Framework: Laravel, latest LTS release. Upgrades to a new LTS happen as a
  tracked, tested migration, never silently.
- Style: PSR-12, enforced by Laravel Pint in CI (`pint --test` must pass
  before merge). No style debates in code review — Pint is the arbiter.
- Static analysis: PHPStan (or Larastan) at level 8 minimum, run in CI. New
  code must not introduce new baseline suppressions without a documented
  reason inline.

## Architecture within a module

Every domain module follows [module-template.md](../templates/module-template.md):
`Models/`, `Policies/`, `Http/Controllers/`, `Http/Requests/`, `Services/`,
`Repositories/`, `Events/`, `Listeners/`, `Notifications/`, `database/migrations/`,
`database/factories/`, `routes/`, `tests/`.

- **Controllers** are thin. A controller method validates (via a Form
  Request), calls exactly one Service method, and returns a Resource/response.
  No business logic, no query building, in a controller.
- **Services** hold business logic and orchestration. A Service method
  represents one use case (e.g. `InvoiceService::issue()`), not a CRUD dump.
- **Repositories** are used for any model with non-trivial query complexity
  or where swappable persistence is plausible; see
  [repository-pattern.md](./repository-pattern.md) for exactly when a
  repository is required vs. when direct Eloquent use on a thin model is
  acceptable.
- **Form Requests** own all validation and authorization-via-`authorize()`
  for their route. Never validate manually inside a controller.
- **Policies** own all model-level authorization decisions, registered
  against the RBAC permissions defined in
  [rbac-permissions.md](../security/rbac-permissions.md). Controllers call
  `$this->authorize()`; they never inline permission checks.
- **Events/Listeners** are used for any side effect that is not the primary
  purpose of the action (sending a notification, writing an audit log entry,
  firing a webhook). Prefer queued listeners for anything not required to
  complete synchronously.

## Dependency injection

Constructor injection only. Do not resolve dependencies via the `app()`
helper or facades inside a class that has a constructor, except for
first-party Laravel facades explicitly allowed for readability
(`Auth`, `Cache`, `Queue`). Never use facades in Services where a mock is
needed for isolated unit testing — inject the underlying contract instead.

## Eloquent conventions

- Every model declares `$fillable` explicitly (never `$guarded = []`).
- Every tenant-scoped model uses a global scope for tenant isolation — see
  [multi-tenancy.md](../architecture/multi-tenancy.md). Never rely on every
  query manually adding a `where('tenant_id', ...)`.
- Relationships are typed with return type declarations
  (`public function invoices(): HasMany`).
- No business logic in Eloquent model `boot()` beyond simple, obvious
  defaults (e.g. UUID generation). Anything more belongs in a Service via an
  event.
- Mass assignment only through validated Form Request data, never raw
  `$request->all()`.

## Query performance

- N+1 queries are a defect, not a style preference. Eager-load relationships
  used in any list/detail view. CI must run with Laravel's
  `Model::preventLazyLoading()` enabled in the testing environment so an N+1
  fails the test suite, not just a code review.
- Any endpoint returning a list must paginate; see
  [table-standards.md](../standards/table-standards.md) and
  [performance-standards.md](../quality/performance-standards.md) for exact
  budgets.

## Error handling

- Domain-level failures throw typed exceptions extending a product's base
  `DomainException`, caught centrally in the exception handler and mapped to
  the standard API error envelope in [api-standards.md](./api-standards.md).
- Never use exceptions for expected control flow (e.g. "not found" in a
  loop). Never silently swallow an exception — log with context or rethrow.

## Jobs and queues

- Every queued Job is idempotent — re-running it with the same input must not
  double-apply side effects (e.g. double-charge, double-send). Use unique job
  IDs or DB-level idempotency keys where the side effect is external.
- Jobs declare `$tries`, `$backoff`, and a `failed()` handler that logs and,
  for user-impacting jobs, notifies. See
  [caching-queues-events.md](../architecture/caching-queues-events.md).

## Configuration

- No hardcoded config values in code. All environment-dependent values go
  through `config/*.php`, sourced from environment variables per
  [environment-config-standards.md](./environment-config-standards.md).

## Testing requirement

Every Service method and every API endpoint has coverage per
[testing-standards.md](./testing-standards.md); PHPUnit/Pest is the standard
test runner (Pest syntax preferred for new products).
