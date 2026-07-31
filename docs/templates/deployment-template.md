# Deployment Template

This document specifies the standard release and deployment scaffold every
Zodize product uses. The authoritative CI/CD pipeline definition (for
Zodize's own engineering environments, used to build and verify a release
before it is packaged for a buyer) is in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md); the real
deployment topology a buyer's instance runs on is defined in
[`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer).
This document is the concrete checklist and sequence between them: how a
product goes from a tagged commit in Zodize's own repository to a
self-hosted source-code package a non-technical buyer installs on their own
shared/VPS hosting. A product customizes environment-specific configuration
values and its own migration files; it does not customize the environment
tiers, the release packaging sequence, or the installer health-check gate.

## No Zodize-operated production environment

Zodize does not run any product as a hosted service, and there is no
Zodize-operated "production" tier serving buyer traffic — see
[`single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md).
Every buyer's live instance runs entirely on hosting the buyer provisions
and controls; Zodize has no runtime access to it beyond what the buyer
explicitly grants for support. What Zodize's own pipeline is responsible for
is producing a correct, tested, installable release package — not operating
it.

## Zodize's own environment tiers (pre-release only)

Every product MUST maintain exactly two environment tiers inside Zodize's
own engineering environment, both used only to build and verify a release
before it is packaged:

| Tier | Purpose | Data | Access |
|---|---|---|---|
| `local` | Individual engineer development. | Synthetic (Seeder-generated), never a copy of any buyer's data. | Engineer's own machine. |
| `staging` | Pre-release verification and demo environment for internal review, running the exact codebase that will be packaged as a release. | Synthetic or demo data seeded per the Demo Standard in [`../../README.md`](../../README.md); MUST NOT contain real buyer data, since Zodize has no standing access to any buyer's database. | Zodize engineering + internal stakeholders. |

A product MUST NOT package a release from anything other than a commit that
has passed `staging` verification in the same pipeline run; skipping this
requires an explicit, logged break-glass process defined in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md), never a
silent unverified release.

## Required environment variables checklist

Every product's `.env.example` (the file a buyer copies to `.env` and fills
in at install time — see
[`../architecture/overview.md`](../architecture/overview.md#the-business-model-this-architecture-serves))
MUST document, at minimum, the following categories. Per the "buyer never
edits code" model, the ONLY `.env` values a buyer is required to set at
install time are the database credentials; every other credential below is
either optional (with a documented safe default) or configured later from
the admin panel per
[`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md),
never required as a blocking `.env` edit:

- [ ] `APP_ENV`, `APP_KEY`, `APP_URL` — framework identity; `APP_URL` MUST
      be documented as "set this to your domain" in the install guide, not
      left as a placeholder the buyer must know to change.
- [ ] `DB_*` — database connection (host, port, database, credentials) —
      the one `.env` edit that is mandatory at install time, per the
      buyer's shared/VPS hosting control panel database setup.
- [ ] `REDIS_*` (optional) — if the buyer's hosting provides Redis; the
      product MUST fall back to file/database cache and the database queue
      driver when absent, per
      [`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer).
- [ ] `MAIL_*` — has a working default (PHP `mail()` or a documented free
      tier) but is normally reconfigured by the buyer from the admin panel's
      notification settings, not by editing `.env`, per
      [`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#notifications).
- [ ] `QUEUE_CONNECTION` — defaults to `database` for hosting without a
      persistent worker process; a buyer on a VPS with Supervisor/systemd
      access MAY switch to `redis`, never a requirement.
- [ ] Payment gateway credentials are NOT `.env` variables at all — every
      gateway is configured entirely from the admin panel per
      [`../standards/payment-gateways.md`](../standards/payment-gateways.md);
      a product that requires a gateway API key in `.env` fails this
      template.
- [ ] `SENTRY_DSN` or equivalent error-tracking endpoint (optional) — for
      Zodize's own `staging` verification only; a buyer's production
      instance has no Zodize-operated monitoring by default, per
      [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md).
- [ ] Confirmation that the in-database `feature_flags` table from
      [database-template.md](./database-template.md) is used — there is no
      externally hosted feature-flag service, since a buyer's instance has
      no network dependency on Zodize infrastructure.
- [ ] Any product-specific third-party integration credentials a buyer
      configures, each individually listed in the product's own
      `docs/products/<product>/SPEC.md`, with an explicit statement of
      whether it is a required `.env` value or an admin-panel setting.

Secrets used during Zodize's own `staging` verification MUST be sourced from
the secrets manager defined in
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md) and MUST
NOT appear in any committed `.env` file, CI log, or the shipped release
package in plaintext. A buyer's own production secrets (their database
password, their payment gateway keys) exist only on the buyer's own hosting
and are never transmitted to or visible from Zodize's systems.

## Release packaging sequence

Because there is no Zodize-operated production deploy, "deployment" for
Zodize's own pipeline means producing the artifact a buyer installs or
updates with — not shifting live traffic. Every release MUST follow this
sequence, in order:

1. Build the release artifact (the source-code archive, or an in-admin
   "update package") from a tagged commit (see
   [release-template.md](./release-template.md) for versioning).
2. Run the automated test suite per
   [testing-template.md](./testing-template.md); a failing suite MUST block
   packaging the release.
3. Deploy the tagged build to Zodize's own `staging` tier and run the
   installer health-check gate (below) against it, using fresh synthetic/demo
   data to simulate a buyer's first install.
4. Verify the in-admin "check for update" / update-package flow (where the
   product documents one, per
   [`single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#licensing-and-update-model))
   applies cleanly on top of the previous release's `staging` state.
5. Publish the verified artifact as the new release. A buyer applies it to
   their own hosting on their own schedule — Zodize does not push it to any
   running buyer instance.

## Migration-run policy for a buyer-applied update

- Migrations shipped in a release MUST be safe to run against a live
  instance of the immediately prior release's schema — a buyer applying an
  update runs `php artisan migrate` themselves (directly, or via an in-admin
  update flow that runs it for them) with their site briefly in maintenance
  mode, not as a zero-downtime rolling migration Zodize orchestrates.
- Destructive migrations (dropping a table or column) MUST only ship in a
  release where no code in that release or the immediately prior release
  reads from the dropped structure, since a buyer may delay applying an
  update by multiple releases.
- The product's install/update guide MUST tell the buyer to back up their
  database before running any update, per
  [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md) —
  this is the buyer's own responsibility, since Zodize has no standing
  access to take that backup for them.

## Installer / health-check gate

Zodize's own `staging` verification, and the buyer-facing installer/update
flow, MUST both use the same health check: it verifies, at minimum, database
connectivity, cache/queue backing store connectivity (or documented fallback
to file/database drivers), and successful boot of the application. This is
the same check surfaced in
[admin-template.md](./admin-template.md)'s System Health page — a product
MUST NOT maintain a separate health-check implementation for release
verification versus the buyer's own operator visibility. The buyer-facing
installer MUST show a clear, actionable error (missing PHP extension,
unwritable `config/`or storage directory, database connection failure) and
MUST NOT leave the buyer with a blank white-screen failure.

## What the base codebase / release tooling provides vs. what a product customizes

The base codebase and its release tooling (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md))
provide: the CI/CD pipeline templates for Zodize's own `staging`
verification, the health-check endpoint contract, the release-artifact
packaging script, and the in-admin update-package flow where a product
documents one.

A product customizes: its own environment variable values (not the
checklist categories), its own migration files, and product-specific
health-check sub-checks it registers into the shared health endpoint. A
product MUST NOT bypass the health check gate or `staging` verification for
any release, and MUST NOT require a buyer to run a CLI command or edit code
beyond the one mandatory `.env` database-credentials step to get the
product running.
