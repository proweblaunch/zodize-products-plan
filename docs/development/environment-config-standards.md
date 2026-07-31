# Environment & Configuration Standards

## Environment tiers

Every product runs in four tiers: `local` (developer machine), `ci`
(ephemeral, test suite only), `staging` (production-like, pre-release
verification), `production`. Configuration differences between tiers live
exclusively in environment variables, never in branched code
(`if app()->environment('staging')` in application logic is prohibited
outside of a small, explicit allowlist such as debug-toolbar registration).

## Configuration source of truth

- All configuration is read through Laravel's `config/*.php` files, which in
  turn read `env()` — `env()` is called only inside `config/` files, never
  directly in application code, so configuration is cacheable
  (`config:cache`) without surprises.
- Every environment variable a product depends on is documented in that
  product's `.env.example` with a comment explaining its purpose and, where
  applicable, its expected format/valid values.

## Secrets management

- Secrets (API keys, database credentials, signing keys) are never committed
  to the repository, including in `.env.example` (which contains only
  placeholder/dummy values clearly marked as such).
- Production secrets are managed through the deployment platform's secret
  store (per [deployment-template.md](../templates/deployment-template.md)),
  injected at deploy/runtime, rotated on a documented schedule, and rotated
  immediately on suspected compromise per
  [security-standards.md](../security/security-standards.md).

## Required baseline environment variables

Every product's `.env.example` includes, at minimum: `APP_ENV`, `APP_URL`,
`APP_KEY`, `DB_*`, `REDIS_*`, `QUEUE_CONNECTION`, `MAIL_*`,
`BROADCAST_CONNECTION`, and `SENTRY_DSN`-equivalent for error tracking
([monitoring-observability.md](../quality/monitoring-observability.md)).
`DB_*` is the only configuration a buyer sets at the file level — everything
else (branding, gateways, wallet, KYC, referrals, plans, i18n, CMS) is
configured after install from the admin panel, per
[admin-configuration-baseline.md](../standards/admin-configuration-baseline.md)
and
[overview.md](../architecture/overview.md#the-business-model-this-architecture-serves).

## Feature-tier configuration

Plan/tier-gated features (see
[product-philosophy.md](./product-philosophy.md#second-layer-feature-catalog))
are controlled through the feature flag system
([feature-flags.md](./feature-flags.md)), never through environment
variables — environment variables configure the deployment, feature flags
configure product behavior within that one deployment.

## Validation on boot

The application validates required configuration on boot (a missing
required secret in a non-local environment fails startup loudly, not a
runtime `null` error deep in a request). This is implemented as a startup
health check consumed by the deployment pipeline's readiness gate
([ci-cd-standards.md](../quality/ci-cd-standards.md)).
