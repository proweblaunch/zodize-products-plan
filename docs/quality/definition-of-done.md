# Definition of Done

> The checklist a single feature, change, or pull request MUST satisfy before
> merge. This is the per-PR bar; the much higher per-product bar is
> [`definition-of-production-ready.md`](./definition-of-production-ready.md).
> Every item below is a required gate, not a suggestion — a PR missing any
> item is not done, regardless of how good the code is.

## Code

- [ ] Code follows the coding standards in [`../development/`](../development/)
      (naming, structure, module boundaries per
      [`../architecture/overview.md`](../architecture/overview.md#modular-monolith-not-microservices)).
- [ ] No new Composer or NPM dependency was added without a stated reason in
      the PR description; the dependency audit in
      [`../security/security-standards.md`](../security/security-standards.md#dependency-scanning)
      passes with no new `high`/`critical` advisories.
- [ ] Every new or changed controller action is gated by a Policy/Gate check
      per [`../security/authentication-authorization.md`](../security/authentication-authorization.md#authorization-policy-based-pattern) —
      no bare route added without an authorization check.
- [ ] Every new tenant-owned Eloquent model uses the tenant global scope per
      [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md#global-query-scopes-for-tenant-isolation)
      and ships a cross-tenant isolation test (see Tests below).
- [ ] Any new sensitive action (auth event, permission change, data export,
      destructive action, financial transaction) is wired into the audit log
      per [`../security/audit-logging.md`](../security/audit-logging.md#what-must-be-audited).

## Tests

- [ ] Unit and feature tests cover the change, including the failure/edge
      paths, not only the happy path.
- [ ] If the change touches a tenant-owned resource, a cross-tenant
      isolation test exists per
      [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md#cross-tenant-data-leakage-prevention).
- [ ] All existing tests pass locally and in CI; no test was skipped or
      commented out to get the PR green.
- [ ] New API endpoints have a feature test asserting the exact response
      shape (status code, JSON structure, pagination envelope where
      applicable per
      [`performance-standards.md`](./performance-standards.md#pagination-requirement)).

## Lint, types, and static analysis

- [ ] No new PHPStan/Larastan or ESLint errors introduced; the CI static
      analysis stage in [`ci-cd-standards.md`](./ci-cd-standards.md) passes
      at the repository's configured level with zero new baseline entries.
- [ ] No new PHP or TypeScript type errors; strict types are declared on any
      new PHP file (`declare(strict_types=1);`) and new Vue components use
      `<script setup lang="ts">` typed props.
- [ ] Code formatting (Pint for PHP, Prettier/ESLint for Vue/TS) is applied;
      no manually-formatted diff noise.

## Accessibility

- [ ] Any new or changed UI is checked against
      [`../design-system/`](../design-system/) accessibility guidance:
      keyboard navigability, visible focus states, correct heading
      hierarchy, form labels programmatically associated with inputs, and
      color contrast meeting WCAG 2.1 AA (4.5:1 for normal text, 3:1 for
      large text and UI components).
- [ ] Interactive elements have accessible names (not icon-only buttons
      without an `aria-label`); any new modal/dialog traps focus and is
      dismissible via `Escape`.
- [ ] Any new data visualization or status indicator does not rely on color
      alone to convey meaning.

## Documentation

- [ ] Any new API endpoint is documented per
      [`../development/`](../development/) API documentation standard
      (request/response shape, auth requirement, rate limit tier).
- [ ] Any new permission is added to the product's permissions matrix in
      `docs/products/<product>/SPEC.md` per
      [`../security/rbac-permissions.md`](../security/rbac-permissions.md#default-system-roles).
- [ ] `CHANGELOG.md` (or the product's own changelog, for product-repo PRs)
      is updated with a user-facing summary of the change.
- [ ] Inline code comments explain *why*, not *what*, and only where the
      code's intent is not otherwise obvious.

## Review

- [ ] At least one other engineer (or, for handbook changes, another
      contributor per [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)) has
      reviewed and approved the PR.
- [ ] Every review comment is resolved or explicitly acknowledged with a
      reply before merge — no unaddressed open threads.
- [ ] The PR description states what changed and why, and links the
      relevant ticket/issue.

## Feature flags

- [ ] Any feature that is not fully complete, or that changes behavior a
      tenant is not yet ready for, is gated behind a feature flag with a
      documented default (off for incomplete work, gradually enabled per
      the rollout plan in
      [`ci-cd-standards.md`](./ci-cd-standards.md#deployment-strategy)).
- [ ] The flag has a named owner and a removal plan — flags are not left
      permanently in the codebase once a feature is fully rolled out.

## CI

- [ ] All required status checks defined in
      [`ci-cd-standards.md`](./ci-cd-standards.md#required-status-checks-before-merge)
      are green.
- [ ] The change deploys cleanly to the PR's preview environment (see
      [`ci-cd-standards.md`](./ci-cd-standards.md#branch-environments)) with
      no manual intervention required.

## Related standards

- [`definition-of-production-ready.md`](./definition-of-production-ready.md)
- [`ci-cd-standards.md`](./ci-cd-standards.md)
- [`performance-standards.md`](./performance-standards.md)
- [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
