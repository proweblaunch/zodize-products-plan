# Pull Request Checklist

This is the standard checklist every pull request against a Zodize product
or this handbook MUST satisfy before merge. It complements
[`../development/pr-standards.md`](../development/pr-standards.md) (how a PR
should be structured and sized) and
[`../development/code-review-standards.md`](../development/code-review-standards.md)
(how a reviewer should evaluate it). A reviewer MUST NOT approve a PR with
unchecked items unless the item is genuinely not applicable, in which case
the author MUST state why in the PR description rather than silently
skipping it.

## Scope and structure

- [ ] The PR does one coherent thing, per
      [`../development/pr-standards.md`](../development/pr-standards.md);
      unrelated changes are split into separate PRs.
- [ ] The PR description explains the "why," not just the "what."
- [ ] If the PR touches a template in [`../templates/`](../templates), the
      description notes which existing product specs, if any, now diverge
      from the template and need a follow-up, per
      [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md).
- [ ] If the PR is a breaking change to an existing standard, an ADR exists
      in [`../decisions/`](../decisions) per
      [`../decisions/adr-template.md`](../decisions/adr-template.md) and is
      linked from this PR.

## Code quality

- [ ] Code follows [`../development/coding-standards-php-laravel.md`](../development/coding-standards-php-laravel.md) and [`../development/coding-standards-vue.md`](../development/coding-standards-vue.md)
      (Laravel and Vue conventions).
- [ ] New modules follow the directory structure and layer responsibilities
      in [`../templates/module-template.md`](../templates/module-template.md).
- [ ] No business logic in Controllers or Models beyond what
      [`../templates/module-template.md`](../templates/module-template.md)
      permits.
- [ ] No dead code, commented-out code, or debug statements left in.
- [ ] No placeholder text, TODO without a tracked follow-up, or "coming
      soon" left outside an explicit Open Questions/Roadmap section.

## API and data

- [ ] Any new or changed endpoint follows
      [`../templates/api-template.md`](../templates/api-template.md) —
      standard envelope, standard error format, pagination, allow-listed
      filter/sort fields.
- [ ] Any new or changed table follows
      [`../development/database-standards.md`](../development/database-standards.md)
      and, if tenant-scoped, carries `tenant_id` per
      [`../templates/database-template.md`](../templates/database-template.md).
- [ ] Any new permission is registered per
      [`../templates/permission-template.md`](../templates/permission-template.md),
      not checked ad hoc.

## Testing

- [ ] Unit, Feature, and (where applicable) Browser tests are included per
      [`../templates/testing-template.md`](../templates/testing-template.md).
- [ ] Feature tests include the mandatory authorization-denial case for any
      new or changed endpoint.
- [ ] All tests pass in CI per
      [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md).
- [ ] No test was skipped, marked flaky-and-ignored, or deleted to make CI
      pass without a documented reason.

## Security and accessibility

- [ ] No secret, credential, or PII is introduced into source control, logs,
      or client-side bundles.
- [ ] Any sensitive action added writes an audit log entry per
      [`../security/audit-logging.md`](../security/audit-logging.md).
- [ ] Any new UI is checked against
      [`../checklists/accessibility-checklist.md`](./accessibility-checklist.md)
      for the specific flow it introduces (full-checklist re-run is not
      required per PR; the new surface area is).

## Documentation

- [ ] Relevant documentation (`docs/products/<product>/SPEC.md`, guides
      under [`../../docs/`](../../docs), or this handbook) is updated in the
      same PR, not deferred.
- [ ] `CHANGELOG.md` is updated under `[Unreleased]` per
      [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md).
- [ ] Any new customer-facing endpoint is reflected in the OpenAPI
      generation source per
      [`../templates/documentation-template.md`](../templates/documentation-template.md).

## Review

- [ ] At least one reviewer other than the author has approved.
- [ ] All reviewer comments are resolved or explicitly deferred with a
      tracked follow-up, not silently dismissed.
