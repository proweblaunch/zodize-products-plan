# Release Template

This document specifies the standard release process scaffold every Zodize
product follows, from versioning through customer-facing communication. It
complements [deployment-template.md](./deployment-template.md), which covers
the mechanics of getting a build into production; this document covers what
constitutes a "release" and how it is communicated and rolled out. A product
customizes its own release cadence and feature content; it does not
customize the versioning scheme, changelog format, or the requirement for a
rollback plan.

## Semantic versioning policy

Every product MUST version its API and its customer-facing release notes
using semantic versioning (`MAJOR.MINOR.PATCH`):

- `MAJOR` — a breaking API change (see the versioning rules in
  [api-template.md](./api-template.md)) or a change that requires customer
  action to avoid disruption.
- `MINOR` — new customer-visible functionality, backward-compatible.
- `PATCH` — bug fixes, backward-compatible, no new functionality.

Internal application deploys (see [deployment-template.md](./deployment-template.md))
MAY happen far more frequently than versioned releases — not every deploy is
a release. A "release" is a version bump accompanied by customer-facing
release notes; routine deploys of internal refactors or non-customer-visible
fixes do not require one.

## Changelog format

Every product MUST maintain a `CHANGELOG.md` (surfaced to customers via
[documentation-template.md](./documentation-template.md)'s Changelog
section) following this structure:

```markdown
## [1.4.0] - 2026-07-31

### Added
- Short, customer-facing description of new functionality.

### Changed
- Short, customer-facing description of behavior changes.

### Fixed
- Short, customer-facing description of bugs fixed.

### Deprecated
- Functionality now deprecated, with removal timeline.
```

- Entries MUST be written for the customer reading them, not copied from
  commit messages or internal ticket titles.
- Every entry MUST correspond to a real, shipped, customer-visible change —
  internal refactors, dependency bumps, and test additions MUST NOT appear
  in the customer-facing changelog (they may still be tracked in internal
  commit history).
- `Unreleased` changes accumulate at the top of the file until the next
  version is cut, then move under a dated version heading.

## Release notes template for customers

Every `MINOR` or `MAJOR` release MUST ship customer-facing release notes
using this structure, published alongside the changelog entry:

```markdown
# {Product} {version} — {one-line theme}

## What's new
- Feature-level summary, written for the end user, not the engineer.

## Improvements
- Smaller enhancements bundled into this release.

## Fixes
- Notable bug fixes worth customer awareness (not every patch-level fix
  needs a customer-facing line item).

## Breaking changes (MAJOR releases only)
- What changed, why, and the concrete action a customer must take, with a
  link to a migration guide if action is non-trivial.
```

`PATCH` releases MAY skip full release notes and rely on the changelog
entry alone, unless the patch fixes a customer-impacting incident, in which
case a short notes entry is required regardless of version tier.

## Feature flag rollout stages

Because each deployment is an independent, buyer-owned instance with its own
single `feature_flags` row per flag (see
[database-template.md](./database-template.md)), a rollout stage is not a
runtime toggle that varies across many live customers of one running
instance — it governs which **release channel** a feature ships in, using
the flag's local `is_enabled` value inside a given release plus
[admin-template.md](./admin-template.md)'s feature flag management UI for a
buyer who wants to switch it off locally after installing that release.
Every customer-facing feature of non-trivial risk (anything touching
wallet/ledger behavior, a data migration, or a core workflow) MUST progress
through the following stages before it ships in the default release
channel:

1. **Internal** — the flag ships `is_enabled` false by default, exercised
   only in Zodize's own `staging` environment (see
   [deployment-template.md](./deployment-template.md#zodizes-own-environment-tiers-pre-release-only))
   against demo data. Used to validate the feature before any buyer sees it.
2. **Beta** — shipped in an explicit, clearly labeled beta release channel
   that an opted-in list of beta buyers download and install themselves,
   having agreed to early access and a feedback channel. Minimum soak time
   before promotion is defined per-product in
   `docs/products/<product>/SPEC.md`; there is no fixed handbook-wide
   minimum because risk varies by feature.
3. **General availability (GA)** — the flag ships `is_enabled` true by
   default in the standard release channel every buyer downloads/updates
   into, with the buyer's own admin panel retaining the ability to switch it
   off locally for their instance if they need to stay on prior behavior
   temporarily.

A feature MUST NOT skip the Beta stage for any change classified as
non-trivial risk above. A feature MAY skip directly to GA only for low-risk,
purely additive, easily reversible changes.

## Rollback plan requirement

Every release MUST have a documented rollback plan before it ships, covering:

- The mechanism to disable the feature (flag rollback to a prior stage, or
  full deploy rollback per [deployment-template.md](./deployment-template.md)).
- Whether any database migration in the release is safely reversible per the
  expand/contract policy in [deployment-template.md](./deployment-template.md);
  if not, the rollback plan MUST explicitly state the forward-fix approach
  instead of a database rollback.
- Who is notified and how, if a rollback is executed (see the incident
  process referenced in [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md)).

A release without a documented rollback plan fails the production readiness
gate in [`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md).

## What the base codebase provides vs. what a product customizes

The base codebase and shared component library (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md))
provide: the feature flag storage/evaluation mechanism, the changelog-to-docs
sync used by [documentation-template.md](./documentation-template.md), and
the release-notes publishing pipeline — all running inside the product's own
codebase.

A product customizes: its own changelog content, release notes content,
rollout stage durations, and which features are classified as non-trivial
risk. A product MUST NOT publish a `MAJOR` or `MINOR` release without both
an updated `CHANGELOG.md` entry and a documented rollback plan.
