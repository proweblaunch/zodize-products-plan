# Testing Standards

## Test pyramid

| Layer | Tool | Scope | Required coverage |
|---|---|---|---|
| Unit | Pest/PHPUnit | Services, single-responsibility classes | Every Service method's happy path + every distinct business-rule branch |
| Feature/API | Pest/PHPUnit + Laravel HTTP testing | Full HTTP request → response, including auth/authorization, and rendered-view assertions | Every endpoint: happy path, validation failure, authorization denial (mandatory), not-found |
| End-to-end | Playwright | Full browser, real backend (test deployment/database) | Every critical path in the product spec's "User Journeys" section |
| Contract | Schemathesis/Dredd-class tool | API responses vs. OpenAPI spec | Every endpoint, run in CI |

## Non-negotiable test cases

Every mutating API endpoint's feature test suite must include, at minimum:

1. Success case with a fully authorized, valid request.
2. Validation failure (422) with an invalid payload.
3. Authorization denial (403) for a role that lacks the required permission
   — see [rbac-permissions.md](../security/rbac-permissions.md). This case
   is mandatory precisely because it is the one most often skipped and the
   one most often exploited.
4. Not-found (404) for a nonexistent or soft-deleted (without
   `with_trashed`) resource.
5. On a product with multi-company/multi-branch scoping (see
   [localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)):
   a request authenticated as a staff member of one branch cannot read or
   mutate another branch's resource, even by guessing a valid UUID, unless
   they hold cross-branch permission.

There is no cross-tenant isolation test category — every product's schema
already belongs to exactly one business, so there is no second tenant's data
in the same running application to leak into. See
[single-tenant-deployment-model.md](../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).

## Test data

Tests use model factories (see
[migration-seeder-standards.md](./migration-seeder-standards.md#seeders)),
never hand-rolled fixtures duplicating factory logic. Tests never depend on
seeded demo data — the two are separate, non-overlapping datasets.

## Isolation and determinism

- Every test runs inside a database transaction rolled back after the test
  (`RefreshDatabase`/`DatabaseTransactions`).
- Time-dependent tests use `Carbon::setTestNow()` / a frozen clock, never
  `sleep()`.
- External HTTP calls are faked (`Http::fake()`) in unit/feature tests; only
  a dedicated, explicitly-tagged integration test suite hits real third-party
  sandboxes, run separately from the main CI gate.
- Flaky tests are treated as broken tests: quarantined immediately (tagged
  and excluded from the required gate) and fixed within one sprint, not
  silently retried until green.

## Coverage requirement

Minimum line coverage: 80% for Services and Policies, 70% overall per
module, enforced in CI per [ci-cd-standards.md](../quality/ci-cd-standards.md).
Coverage percentage is a floor, not a target — the mandatory test cases above
matter more than the number.

## Accessibility and visual testing

- Automated accessibility checks (axe-core) run against every page in the
  Playwright e2e suite; a new WCAG AA violation fails the build. See
  [accessibility.md](../design-system/accessibility.md).
- Visual regression testing (screenshot diffing) runs on the shared
  `base/` component library on every PR that touches it.

## Performance testing

Load testing against the budgets in
[performance-standards.md](../quality/performance-standards.md) is required
before a product reaches Production Ready status
([definition-of-production-ready.md](../quality/definition-of-production-ready.md)),
and re-run before any major release that touches a high-traffic endpoint.
