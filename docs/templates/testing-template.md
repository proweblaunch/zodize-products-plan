# Testing Template

This document specifies the standard test suite scaffold every module must
ship, mirroring the `tests/` directory defined in
[module-template.md](./module-template.md). The authoritative testing
standards (frameworks, coverage thresholds, CI gating) are defined in
[`../development/testing-standards.md`](../development/testing-standards.md);
this document is the concrete per-module checklist. A product customizes
test cases specific to its domain logic; it does not customize which test
categories are mandatory or skip the authorization-denial case.

## Directory structure (recap)

```
Modules/{ModuleName}/tests/
  Unit/
    {Entity}ServiceTest.php
  Feature/
    {Entity}ControllerTest.php
  Browser/
    {Entity}FlowTest.php
database/factories/
  {Entity}Factory.php
database/seeders/
  {Entity}Seeder.php
```

## Unit tests (Services)

- Every public method on a Service class MUST have at least one unit test.
- Unit tests MUST mock/fake the Repository and any external
  integration — a unit test that hits the database or an external API is
  misclassified and belongs in Feature tests instead.
- Unit tests MUST cover: the success path, and every distinct branch in the
  method's business logic (e.g. an `if` that changes behavior based on
  tenant plan MUST have a test per branch).
- Unit tests MUST NOT assert on HTTP status codes or response JSON shape —
  that belongs to Feature tests.

## Feature tests (HTTP endpoints)

Every controller action MUST have a Feature test class covering, at
minimum, the following cases — this is a mandatory floor, not a menu:

1. **Happy path** — a properly authorized, properly formed request returns
   the expected success response in the standard envelope from
   [api-template.md](./api-template.md).
2. **Key edge cases** — specific to the endpoint, e.g. pagination boundary
   (empty result set, last page), duplicate-creation conflict, referencing a
   soft-deleted related record, tenant-scoping (a user from Tenant A MUST
   NOT be able to read/modify Tenant B's resource, and the test MUST assert
   a `404`, not a `403`, to avoid confirming the resource's existence to an
   unauthorized tenant).
3. **Authorization denial case — mandatory** — a request from an
   authenticated user who lacks the required permission (see
   [permission-template.md](./permission-template.md)) MUST be tested and
   MUST assert a `403` in the standard error format. A Feature test suite
   missing this case fails the module-level review gate in
   [`../checklists/pr-checklist.md`](../checklists/pr-checklist.md)
   regardless of how much other coverage exists.
4. **Validation failure case** — a malformed request MUST assert a `422`
   with field-level `details` per [api-template.md](./api-template.md).

## Browser/e2e tests (critical flows)

- A module MUST ship a Browser test for any flow classified as
  "critical" — authentication (see
  [authentication-template.md](./authentication-template.md)), any flow
  that moves money or commits an irreversible action (send, void, delete),
  and the module's primary create-to-view happy path.
- Browser tests MUST run against a real (test-environment) browser via the
  framework specified in
  [`../development/testing-standards.md`](../development/testing-standards.md),
  not simulate DOM interaction at the unit level.
- Browser tests MUST assert on user-visible outcomes (text rendered, redirect
  occurred, toast shown), not on internal state — they are a proxy for what
  a real user would observe.

## Factory and seeder requirements

- Every model MUST have a corresponding Factory producing a valid,
  persistable instance with sensible random defaults — no Feature or
  Browser test may construct test data by hand-assembling arrays passed
  directly to `Model::create()`.
- A Factory MUST support states for meaningfully distinct variants (e.g.
  `Invoice::factory()->overdue()`, `Invoice::factory()->voided()`) rather
  than every test manually overriding fields to reach that state.
- A module's Seeder MUST populate enough realistic demo data to make the
  module's UI meaningfully explorable in a local/staging environment
  (referenced by [deployment-template.md](./deployment-template.md) for
  non-production environments); Seeders MUST NOT run automatically against
  production.

## Scaffold checklist per module

- [ ] `tests/Unit/` covers every Service public method's branches.
- [ ] `tests/Feature/` covers happy path, key edge cases, authorization
      denial, and validation failure for every controller action.
- [ ] `tests/Browser/` covers every critical flow in the module.
- [ ] A Factory exists for every model, with states for distinct variants.
- [ ] A Seeder exists producing realistic non-production demo data.
- [ ] All tests pass in CI per
      [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md)
      before merge.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the base `TestCase` class with tenant/auth test helpers,
the Browser test driver configuration, and shared Factory traits (e.g.
tenant-scoping helpers).

A product customizes: every test file and Factory/Seeder listed above for
its own modules. A product MUST NOT reduce the mandatory case list above to
ship faster — an incomplete Feature test suite is a release blocker per
[`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md).
