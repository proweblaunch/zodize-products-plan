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
   in-memory/ephemeral test database, including the cross-tenant isolation
   suite required by
   [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md#cross-tenant-data-leakage-prevention).
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

- Production deploys use a **rolling deployment** as the default: new
  application nodes are brought up running the new release, health-checked
  (see
  [`monitoring-observability.md`](./monitoring-observability.md#health-check-endpoint-standard)),
  and added to the load balancer while old nodes are drained and removed
  one batch at a time — no full-fleet cutover in one step.
- Financial-grade products MUST use **blue-green deployment** instead: the
  new release is deployed to a fully separate, pre-warmed environment,
  validated with automated smoke tests against it, and traffic is cut over
  at the load balancer only after validation passes — giving an instant
  full rollback path (repoint the load balancer back to blue) rather than a
  gradual rolling rollback.
- Database migrations MUST be backward-compatible with the previous release
  for the duration of a rolling/blue-green window (additive changes first,
  destructive changes — column drops, renames — in a subsequent release
  after the old code path is fully retired). This "expand and contract"
  pattern is mandatory for any product with more than one running
  application version during deploy.

## Rollback requirement

- Every deploy pipeline MUST support a one-command rollback to the
  immediately prior release artifact, without requiring a new build.
- Rollback MUST be exercised (not just theoretically available) at least
  once per quarter as part of the restore-testing discipline referenced in
  [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#restore-testing-cadence),
  so the team has verified, current muscle memory for it.
- A deploy that trips an automated alert threshold (error rate spike,
  latency regression) within 15 minutes of going live MUST trigger an
  automatic rollback for financial-grade products, and a paged on-call
  decision for all other products, per
  [`monitoring-observability.md`](./monitoring-observability.md#alerting-thresholds-and-on-call-escalation).

## Feature flag gating for risky deploys

- Any change that is behaviorally risky (new payment flow, new
  authentication path, a rewrite of a high-traffic module) MUST ship behind
  a feature flag defaulted off in production, enabled first for an internal
  Zodize test tenant, then a small percentage rollout, then general
  availability — never a direct 100% cutover for risky changes.
- Feature flags are managed centrally through ZodiCore's flag service
  (tenant-scoped and percentage-rollout-capable) so a flag can be disabled
  instantly without a redeploy if an issue surfaces.
- A flag guarding a launched, stable feature MUST be removed within one
  release cycle of reaching 100% rollout, per the flag hygiene rule in
  [`definition-of-done.md`](./definition-of-done.md#feature-flags).

## Related standards

- [`definition-of-done.md`](./definition-of-done.md)
- [`definition-of-production-ready.md`](./definition-of-production-ready.md)
- [`monitoring-observability.md`](./monitoring-observability.md)
- [`../security/security-standards.md`](../security/security-standards.md)
- [`../architecture/overview.md`](../architecture/overview.md)
