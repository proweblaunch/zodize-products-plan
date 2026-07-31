# Security Standards

> Master security standard for every Zodize product. Product specifications in
> `docs/products/<product>/SPEC.md` MAY add stricter requirements on top of this
> document (for example, ZodiBank, ZodiTrade, ZodiXchange, and ZodiMed operate
> under PCI-DSS-equivalent, and HIPAA-equivalent obligations respectively) but
> MUST NOT weaken any requirement below.

## Scope

This document is the floor. It applies to every Laravel service and Vue
frontend in every Zodize product — including `ZodiCore`, itself one of the
twenty sellable products, not a shared platform — and to every plugin
distributed through the marketplace (see
[`../architecture/marketplace-architecture.md`](../architecture/marketplace-architecture.md)).
It is written against the shared base codebase and single-tenant deployment
model described in
[`../architecture/overview.md`](../architecture/overview.md) and
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md).

## OWASP Top 10 mapping

Every product MUST demonstrate mitigation of each of the following before it
is declared production-ready (see
[`../quality/definition-of-production-ready.md`](../quality/definition-of-production-ready.md)):

| OWASP Top 10 (2021) | Required mitigation on this stack |
|---|---|
| A01 Broken Access Control | Every controller action MUST be gated by a Laravel Policy or Gate (see [`authentication-authorization.md`](./authentication-authorization.md)). No route may rely on hiding a UI element as its only control. All tenant-scoped queries MUST pass through the global tenant scope described in [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md). |
| A02 Cryptographic Failures | TLS 1.2+ in transit (TLS 1.3 preferred), AES-256 at rest for any column classified `confidential` or `restricted` under [`data-protection-privacy.md`](./data-protection-privacy.md). No custom cryptography — use Laravel's `encrypter` (AES-256-CBC/GCM via `APP_KEY`) or the framework's `Hash` facade (bcrypt/argon2id) exclusively. |
| A03 Injection | All database access MUST go through Eloquent or the query builder with bound parameters. Raw SQL (`DB::raw`, `whereRaw`) requires a code comment justifying why the builder cannot express the query and MUST NOT interpolate user input. Blade/Vue templates MUST rely on default output escaping; `{!! !!}` and `v-html` require a documented sanitization step. |
| A04 Insecure Design | New modules MUST document their threat model (who can act on what, in what tenant scope) in the product SPEC before implementation, per [`../development/`](../development/) design standards. |
| A05 Security Misconfiguration | `APP_DEBUG=false` and `APP_ENV=production` MUST be enforced in production config, verified by a CI check that fails the build if either is violated in a deploy artifact. Default framework error pages MUST NOT leak stack traces to end users. |
| A06 Vulnerable and Outdated Components | Composer and NPM dependency audits are mandatory CI gates (see below). |
| A07 Identification and Authentication Failures | See [`authentication-authorization.md`](./authentication-authorization.md) for password policy, MFA, and session standards. |
| A08 Software and Data Integrity Failures | CI artifacts MUST be built from a pinned lockfile (`composer.lock`, `package-lock.json`); no `composer update`/`npm update` in a deploy pipeline. Plugin packages MUST be signature-verified per [`../architecture/plugin-architecture.md`](../architecture/plugin-architecture.md). |
| A09 Security Logging and Monitoring Failures | See [`audit-logging.md`](./audit-logging.md) and [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md). |
| A10 Server-Side Request Forgery | Any feature that fetches a user-supplied URL (webhooks, integrations, avatar-by-URL) MUST validate against an allowlist of schemes (`https` only) and MUST reject requests to RFC 1918 private ranges, `169.254.169.254`, and `localhost` at the application layer before the HTTP client resolves DNS. |

## Secure coding baseline

- All input validation MUST use Laravel Form Requests, never ad-hoc
  in-controller checks. See [`../development/`](../development/) for the
  coding standard this inherits from.
- Mass assignment MUST be controlled via explicit `$fillable` (never
  `$guarded = []`) on every Eloquent model.
- File uploads MUST be validated by MIME type and magic-byte sniffing (not
  extension alone), stored outside the public webroot, and served through a
  controller that re-checks authorization on every request — never via a
  directly guessable public path.
- Every outbound API response MUST set `Content-Type` explicitly; JSON
  endpoints MUST NOT be renderable as HTML.
- Third-party JavaScript MUST be loaded with Subresource Integrity (`integrity`
  attribute) or bundled at build time; no unpinned CDN `<script>` tags in
  production Vue builds.

## Dependency scanning

Every product's CI pipeline MUST run, on every pull request and on a nightly
schedule against `main`:

- `composer audit` — fails the build on any advisory of severity `high` or
  `critical`.
- `npm audit --audit-level=high` — same threshold for the Vue frontend.

A dependency with no available fix MUST be recorded as an accepted risk in the
product's `docs/products/<product>/SPEC.md` under a `## Open Questions`
section with an owner and a re-review date no more than 30 days out. See
[`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md) for where
these gates sit in the pipeline.

## Secrets management

- Secrets (API keys, database credentials, `APP_KEY`, OAuth client secrets,
  webhook signing secrets) MUST NEVER be committed to a repository, in any
  form — not in `.env.example` with a real value, not in a config comment,
  not in a test fixture.
- Every environment MUST source secrets from the deployment platform's secret
  store (environment variables injected at deploy time, or a managed vault
  such as AWS Secrets Manager / HashiCorp Vault for financial-grade products).
  Local development uses `.env`, which MUST be listed in `.gitignore` at the
  repository root.
- Secrets MUST be rotated on a fixed schedule: 90 days for standard products,
  30 days for financial-grade products (ZodiBank, ZodiTrade, ZodiXchange,
  ZodiCapital, ZodiYield), and immediately on suspected compromise or
  personnel offboarding with access.
- CI MUST run a secret-scanning check (e.g., gitleaks-equivalent pattern
  scanning) on every push and block merge on a match.

## TLS requirements

- TLS 1.2 is the minimum accepted protocol version for any public endpoint;
  TLS 1.0/1.1 and all SSL versions MUST be disabled at the load balancer.
  Financial-grade products MUST prefer TLS 1.3 and disable TLS 1.2 cipher
  suites that do not provide forward secrecy.
- HSTS MUST be enabled with `max-age=31536000; includeSubDomains; preload` on
  every product domain.
- Internal service-to-service traffic (API to queue workers, API to
  ZodiCore) MUST also be encrypted in transit; plaintext internal HTTP is not
  permitted even inside a private network.

## Rate limiting

- Every public API endpoint MUST be behind Laravel's rate limiter. Default
  budget: 60 requests/minute per authenticated user for read endpoints, 20
  requests/minute per authenticated user for write endpoints, 5 requests/
  minute per IP for unauthenticated endpoints (login, password reset,
  registration).
- Financial-grade products MUST additionally rate-limit per API token at the
  scope level (see [`authentication-authorization.md`](./authentication-authorization.md#api-token-scoping))
  and MUST return `429 Too Many Requests` with a `Retry-After` header on
  every throttled response.
- Login and MFA-verification endpoints MUST implement exponential backoff
  per account after 5 consecutive failures, in addition to the IP-based
  limit, to resist credential stuffing.

## Vulnerability disclosure

- Every product MUST publish a `security.txt` file at
  `/.well-known/security.txt` per RFC 9116, containing a `Contact:` field
  (a monitored `security@<product-domain>` mailbox), an `Expires:` field no
  more than 12 months out, and a link to the disclosure policy.
- The disclosure policy MUST commit to acknowledging a report within 2
  business days and MUST NOT threaten legal action against good-faith
  researchers who follow the policy.
- Confirmed vulnerabilities MUST be triaged within 24 hours for
  financial-grade and healthcare-grade products (critical/high severity) and
  within 5 business days for all other products.

## Penetration testing cadence

- Financial-grade products (ZodiBank, ZodiTrade, ZodiXchange, ZodiCapital,
  ZodiYield) and healthcare-grade products (ZodiMed) MUST undergo a
  third-party penetration test at least every 6 months and after any
  material architecture change (new payment rail, new external integration
  with data access).
- All other products MUST undergo a third-party penetration test at least
  annually.
- Every finding rated `critical` or `high` MUST be remediated or have a
  documented compensating control before the product's next production
  release; findings MUST be tracked to closure, not just filed.
- Results MUST be summarized in the product's production-readiness review
  per [`../quality/definition-of-production-ready.md`](../quality/definition-of-production-ready.md).

## Related standards

- [`authentication-authorization.md`](./authentication-authorization.md)
- [`rbac-permissions.md`](./rbac-permissions.md)
- [`audit-logging.md`](./audit-logging.md)
- [`data-protection-privacy.md`](./data-protection-privacy.md)
- [`backup-disaster-recovery.md`](./backup-disaster-recovery.md)
- [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md)
