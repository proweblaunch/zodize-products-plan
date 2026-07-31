# Deployment Template

This document specifies the standard deployment scaffold every Zodize
product uses. The authoritative CI/CD pipeline definition is in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md); this
document is the concrete checklist and sequence a product's deploy pipeline
implements. A product customizes environment-specific configuration values;
it does not customize the environment tiers, the zero-downtime sequence, or
the health-check gate.

## Environment tiers

Every product MUST maintain exactly three environment tiers:

| Tier | Purpose | Data | Access |
|---|---|---|---|
| `local` | Individual engineer development. | Synthetic (Seeder-generated), never a copy of production data. | Engineer's own machine. |
| `staging` | Pre-production verification, demo environment for internal review. | Synthetic or anonymized; MUST NOT contain real customer data. | Zodize engineering + internal stakeholders. |
| `production` | Live customer traffic. | Real tenant data. | Restricted per [`../security/`](../security) access controls. |

A product MUST NOT deploy directly to `production` without first passing
through `staging` in the same pipeline run; skipping tiers requires an
explicit, logged break-glass process defined in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md), never a
silent manual deploy.

## Required environment variables checklist

Every product's deployment MUST validate presence of, at minimum, the
following categories before a deploy is allowed to proceed — a missing
required variable MUST fail the deploy at the validation step, not at
runtime:

- [ ] `APP_ENV`, `APP_KEY`, `APP_URL` — framework identity.
- [ ] `DB_*` — database connection (host, port, database, credentials via
      secret store, never committed).
- [ ] `REDIS_*` or equivalent cache/queue backing store connection.
- [ ] `MAIL_*` — transactional email provider credentials.
- [ ] `SMS_*` — SMS provider credentials (if the product uses SMS
      notifications).
- [ ] `QUEUE_CONNECTION` — queue driver for background jobs.
- [ ] Payment processor credentials (ZodiCore billing integration), scoped
      per environment (test-mode keys in `local`/`staging`, live keys only
      in `production`).
- [ ] `SENTRY_DSN` or equivalent error-tracking endpoint, per
      [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md)
      monitoring requirements.
- [ ] Feature-flag service connection (if externally hosted) or confirmation
      that the in-database `feature_flags` table from
      [database-template.md](./database-template.md) is used.
- [ ] Any product-specific third-party integration credentials, each
      individually listed in the product's own
      `docs/products/<product>/SPEC.md`.

Secrets MUST be sourced from the secrets manager defined in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md) and MUST
NOT appear in any committed `.env` file, CI log, or deployment manifest in
plaintext.

## Zero-downtime deploy sequence

Every production deploy MUST follow this sequence, in order:

1. Build the release artifact from a tagged commit (see
   [release-template.md](./release-template.md) for versioning).
2. Run the automated test suite per
   [testing-template.md](./testing-template.md); a failing suite MUST block
   the deploy.
3. Deploy the new artifact to a parallel set of application instances
   without routing production traffic to them yet.
4. Run pending database migrations (see migration policy below) against the
   production database before traffic cutover.
5. Run the health check gate (see below) against the new instances.
6. Shift traffic to the new instances incrementally (rolling or blue-green,
   per [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md)),
   monitoring error rate and latency at each increment.
7. Keep the previous release's instances warm and ready for immediate
   rollback until the new release has served production traffic without
   elevated error rate for the monitoring window defined in
   [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md).
8. Decommission the previous release's instances only after that window
   passes.

A deploy MUST NOT terminate the previous release's instances before the new
release has passed the health check gate and served real traffic
successfully — this is what makes the deploy zero-downtime and immediately
reversible.

## Migration-run policy during deploy

- Migrations MUST be backward-compatible with the previous release's code
  for the duration of the rolling deploy — i.e., a migration that drops a
  column or renames it in a way the old code depends on MUST be split into
  a multi-deploy expand/contract sequence (add new, dual-write, backfill,
  switch reads, remove old in a later deploy), never a single breaking
  migration deployed atomically with code that depends on it.
- Migrations MUST run before traffic is routed to the new code (step 4
  above), never after.
- Destructive migrations (dropping a table or column) MUST only run in a
  deploy where no code in that release or the immediately prior release
  reads from it.

## Health check gate

Before traffic cutover, the new release's instances MUST pass a health
check endpoint that verifies, at minimum: database connectivity, cache/queue
backing store connectivity, and successful boot of the application
container. This is the same endpoint surfaced in
[admin-template.md](./admin-template.md)'s System Health page — a product
MUST NOT maintain a separate health-check implementation for deploy gating
versus operator visibility. A deploy MUST automatically roll back if the
health check fails or times out, without requiring manual intervention to
trigger the rollback.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the CI/CD pipeline templates, the secrets manager
integration, the health check endpoint contract, and the rolling/blue-green
traffic-shifting mechanism.

A product customizes: its own environment variable values (not the
checklist categories), its own migration files, and product-specific
health-check sub-checks it registers into the shared health endpoint. A
product MUST NOT bypass the health check gate or the staging tier for any
production deploy.
