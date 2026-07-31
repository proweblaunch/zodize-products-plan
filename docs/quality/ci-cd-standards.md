# CI/CD Standards

> The pipeline every Zodize product's repository MUST implement, enforcing
> [`definition-of-done.md`](./definition-of-done.md) automatically rather
> than by reviewer memory.

## Pipeline stages

Every product's CI pipeline runs the following stages, in order, on every
pull request. A stage MUST fail the build (not just warn) when it detects a
violation:

1. **Lint** — Pint (PHP) and ESLint/Prettier (Vue/TypeScript) in check mode.
   No auto-fix commits in CI; a failing lint stage means the author fixes it
   locally and pushes again.
2. **Static analysis** — PHPStan/Larastan at the repository's configured
   level (minimum level 6, financial-grade and healthcare-grade products
   MUST run at level 8) and `vue-tsc`/TypeScript strict-mode checking for
   the frontend. No new baseline-ignored errors permitted without a review
   comment justifying the suppression.
3. **Unit tests** — fast, isolated tests (no database, no HTTP) with a
   required minimum coverage threshold of 70% for new code (measured as
   diff coverage, not whole-repository coverage, so legacy modules don't
   block unrelated PRs).
4. **Feature tests** — Laravel feature tests hitting real routes against an
   in-memory/ephemeral test database, including the non-negotiable test
   cases (authorization denial, not-found, and — on a product with
   multi-company/multi-branch scoping — cross-branch isolation) required by
   [`../development/testing-standards.md`](../development/testing-standards.md#non-negotiable-test-cases).
5. **Browser/E2E tests** — Playwright (or Cypress) tests covering the
   product's critical user journeys (login, core CRUD workflow, checkout/
   payment where applicable), run against a fully built frontend and a
   seeded backend. Run on every PR touching frontend code or shared API
   contracts; may be scheduled nightly-only for PRs touching only
   documentation or isolated backend modules with no UI surface, at the
   product's discretion.
6. **Security scan** — dependency audit (`composer audit`, `npm audit`) and
   secret scanning per
   [`../security/security-standards.md`](../security/security-standards.md#dependency-scanning),
   plus static application security testing (SAST) for the injection and
   SSRF patterns listed in
   [`../security/security-standards.md`](../security/security-standards.md#owasp-top-10-mapping).
7. **Build** — production asset build (Vite build for the Vue SPA,
   `composer install --no-dev --optimize-autoloader` for the backend),
   producing the deployable artifact. The build MUST fail on any warning
   the framework treats as fatal in production mode.
8. **Deploy** — to the PR's preview environment (below) automatically; to
   staging on merge to `main`; to production only via the explicit release
   process.

## Required status checks before merge

A pull request MUST NOT be mergeable unless all of the following are green,
enforced by branch protection on `main`:

- Lint, static analysis, unit tests, feature tests, security scan, and build
  (stages 1-3, 4, 6, 7 above).
- Browser/E2E tests, when triggered per the rule in stage 5.
- At least one human approving review, per
  [`definition-of-done.md`](./definition-of-done.md#review).
- No unresolved review conversation threads.
- Branch is up to date with `main` (no stale-base merges) — CI re-runs
  automatically on rebase.

Direct pushes to `main` are disabled for every product repository; every
change lands through a reviewed, CI-passing pull request, with no
exceptions for hotfixes — hotfixes get an expedited review, not a bypassed
one.

## Branch environments (preview deploys)

- Every open pull request MUST get an automatically provisioned preview
  environment (a full application stack — web, queue worker, isolated
  database seeded with representative fixture data) reachable at a unique
  URL, torn down automatically when the PR is closed or merged.
- Preview environments use the same container image/build process as
  production to catch environment-specific issues before merge, not a
  divergent "dev mode" build.
- Secrets used in preview environments MUST be preview-scoped, never
  production secrets or production data, per
  [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#pii-handling).

## Deployment strategy

Zodize does not operate production infrastructure for any product — every
buyer runs their own shared/VPS hosting and applies each release themselves
(see
[`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer)
and
[`single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#licensing-and-update-model)).
What this pipeline's "Deploy" stage produces and promotes is:

- **Zodize's own internal staging/demo environment** — a single running
  instance per product, used for QA and for the always-live Demo Standard
  ([`../../README.md`](../../README.md)). This environment uses a **rolling
  restart** (new code deployed, health-checked per
  [`monitoring-observability.md`](./monitoring-observability.md#health-check-endpoint-standard),
  then traffic resumed) — there is no customer traffic to protect here, so a
  full-fleet restart is acceptable.
- **The versioned, downloadable release artifact** the buyer applies to
  their own single hosting instance, per
  [`versioning-releases.md`](../development/versioning-releases.md). Zodize
  never deploys this artifact on the buyer's behalf.
- Database migrations MUST be safe for a buyer to run directly against their
  own single production database with minimal downtime: additive changes
  first, destructive changes — column drops, renames — in a subsequent
  release only after the old code path is fully retired. This "expand and
  contract" pattern matters more here than in a rolling-fleet deploy,
  because a self-hosted buyer applying an update has no second node to mask
  an incompatible in-between state during the update window.

## Rollback requirement

- Zodize's internal staging/demo pipeline MUST support a one-command
  rollback to the immediately prior release artifact, without requiring a
  new build. Rollback MUST be exercised (not just theoretically available)
  at least once per quarter as part of the restore-testing discipline
  referenced in
  [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#restore-testing-cadence),
  so the team has verified, current muscle memory for it.
- A deploy to Zodize's internal staging/demo environment that trips an
  automated alert threshold (error rate spike, latency regression) within
  15 minutes of going live MUST trigger an automatic rollback for
  financial-grade products, and a paged on-call decision for all other
  products, per
  [`monitoring-observability.md`](./monitoring-observability.md#alerting-thresholds-and-on-call-escalation).
- Every product release MUST ship a documented rollback procedure the buyer
  can follow without developer assistance on their own hosting: reinstall
  the prior release's codebase archive and, if a migration ran, restore the
  pre-migration database backup per
  [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md).

## Feature flag gating for risky deploys

- Any change that is behaviorally risky (new payment flow, new
  authentication path, a rewrite of a high-traffic module) MUST ship behind
  a feature flag defaulted off, validated first in Zodize's internal
  staging/demo environment, then included in the next release as an
  opt-in (admin-toggleable) capability before it defaults on in a
  subsequent release — never a direct default-on cutover for risky changes.
- Feature flags are implemented within each product's own codebase per
  [`../development/feature-flags.md`](../development/feature-flags.md) —
  global to the deployment, or scoped per company/branch on a product with
  multi-company/multi-branch operation — so a flag can be disabled from the
  admin panel or a config toggle without a new code release if an issue
  surfaces after a buyer applies an update.
- A flag guarding a launched, stable feature MUST be removed within one
  release cycle of reaching 100% rollout, per the flag hygiene rule in
  [`definition-of-done.md`](./definition-of-done.md#feature-flags).

## Related standards

- [`definition-of-done.md`](./definition-of-done.md)
- [`definition-of-production-ready.md`](./definition-of-production-ready.md)
- [`monitoring-observability.md`](./monitoring-observability.md)
- [`../security/security-standards.md`](../security/security-standards.md)
- [`../architecture/overview.md`](../architecture/overview.md)
