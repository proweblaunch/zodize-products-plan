# Versioning & Release Standards

## Semantic versioning for APIs and SDKs

APIs and SDKs follow semantic versioning (`MAJOR.MINOR.PATCH`):
`MAJOR` for breaking changes (see
[api-standards.md](./api-standards.md#versioning)), `MINOR` for backward-
compatible additions, `PATCH` for backward-compatible fixes.

## Calendar versioning for product releases

Product-level releases (the deployed application as a whole) use calendar
versioning: `YYYY.MM.PATCH` (e.g. `2026.8.1`), because customers reason
about "what did we get this month," not semantic diffing of an internal
monolith. This is distinct from and does not need to track API semantic
versions.

## Release cadence

- Continuously deployed products (the default — see
  [ci-cd-standards.md](../quality/ci-cd-standards.md)) ship to production
  multiple times per day behind feature flags
  ([feature-flags.md](./feature-flags.md)); the calendar version is a
  labeling convenience over a period of continuous deploys, not a discrete
  "release event" requiring a freeze.
- Regulated products with a formal change-control requirement (see
  [ZodiBank](../products/ZodiBank/SPEC.md),
  [ZodiMed](../products/ZodiMed/SPEC.md)) may use scheduled release trains
  with a documented freeze/approval window — the exception is called out
  explicitly in that product's spec, not assumed.

## Changelog requirements

Every product maintains a customer-facing changelog
(`docs/products/<product>/CHANGELOG.md` once implementation begins),
distinct from this repository's own [`CHANGELOG.md`](../../CHANGELOG.md).
Entries are written for the customer, not the engineer: "Invoices can now be
partially paid," not "refactored InvoiceService::pay()."

## Deprecation policy

Any deprecation (an API version, an SDK major version, a feature being
sunset) is announced a minimum of 90 days ahead via the changelog, an
in-product banner for affected tenants, and (for API/SDK deprecation) the
`Sunset` HTTP header per [api-standards.md](./api-standards.md#versioning).

## Release notes template

See [release-template.md](../templates/release-template.md) for the
customer-facing release notes structure and the internal rollout-stage
checklist (internal → beta tenants → GA).
