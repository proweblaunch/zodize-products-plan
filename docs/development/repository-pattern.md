# Repository Pattern

## When a repository is required

Use a Repository class when a model has **any** of:

- Query complexity beyond simple `find`/`where` chains reused across more
  than one Service (e.g. a search/filter query composed from many optional
  parameters — see [table-standards.md](../standards/table-standards.md)).
- Multiple possible data sources or a plausible future one (e.g. a resource
  that may be read from a search index for listing but Eloquent for
  mutation).
- Complex permission-scoped or company/branch-scoped visibility rules that
  must be applied consistently everywhere the model is queried.

## When direct Eloquent is acceptable

Simple CRUD models with no cross-cutting query complexity (e.g. a lookup
table, a settings row) may be queried directly via Eloquent inside a Service,
without a dedicated repository — introducing one would be needless
indirection. See [engineering-principles.md](./engineering-principles.md) on
avoiding premature abstraction.

## Interface and implementation

- Every repository is defined as an interface in `Repositories/Contracts/`
  and bound to a concrete Eloquent implementation in a service provider —
  Services depend on the interface, never the concrete class, so tests can
  substitute an in-memory fake.
- Method names describe intent, not SQL: `findOverdueInvoicesForBranch()`,
  not `queryInvoicesWhereStatusAndDueDate()`.
- Repositories return domain models or typed DTOs/Collections — never raw
  query builder instances or arrays — so callers are insulated from the
  underlying query mechanism.

## Company/branch scoping responsibility

On a product with multi-company/multi-branch scoping, a repository method
never accepts an implicit "current company/branch" from global state
silently — that scoping is applied via the model's global scope (see
[database-standards.md](./database-standards.md#single-tenant-at-the-schema-level)
and
[localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)),
and repository tests explicitly assert that a user without access to a given
branch cannot read its records through the repository.

## What repositories do not do

- No business logic. A repository answers "what data matches these
  criteria" or "persist this," never "should this invoice be void-able" —
  that belongs in a Service.
- No authorization checks. Authorization is a Policy's job
  ([coding-standards-php-laravel.md](./coding-standards-php-laravel.md)); a
  repository trusts the caller has already authorized the request.

## Testing

Repository implementations are tested against a real database (feature-level
test, using the project's test database) rather than mocked, since their
entire value is correct query behavior; Services that depend on a repository
interface are unit-tested against an in-memory fake implementation of that
interface. See [testing-standards.md](./testing-standards.md).
