# Authentication & Authorization

> Every Zodize product implements its own authentication and authorization
> in its own codebase, inherited from the base codebase's existing engine —
> the `web`/`admin`/`branch_staff` guards, password hashing, and session
> handling already built into the base every product clones. There is no
> shared identity service any product calls at runtime; each deployment is
> fully self-contained. Product specs MUST NOT implement a parallel
> authentication system; they build on this inherited one. See
> [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
> and
> [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md).

## Password policy

- Minimum length: 12 characters. No maximum below 128.
- Passwords MUST be checked against a breached-password list (a k-anonymity
  API such as the Have I Been Pwned range API, or an equivalent bundled
  corpus) at registration and at password change, and rejected if found.
- No forced periodic rotation for end users (rotation-on-schedule is
  deprecated NIST guidance and increases weak-password reuse). Financial-grade
  and healthcare-grade products MAY require rotation only in direct response
  to a confirmed breach, never on a calendar.
- Passwords MUST be hashed with `bcrypt` (Laravel default, cost factor 12) or
  `argon2id` where available. Plaintext or reversibly-encrypted password
  storage is prohibited under any circumstance.
- Account lockout: after 10 failed attempts within 15 minutes, the account is
  locked for 15 minutes and the owner is notified by email. This is
  independent of the IP-based rate limit in
  [`security-standards.md`](./security-standards.md#rate-limiting).

## Multi-factor authentication (MFA)

| Role / product tier | MFA requirement |
|---|---|
| Any user with an Admin, Owner, or Billing role (see [`rbac-permissions.md`](./rbac-permissions.md)) | Mandatory. Enforced at login; account cannot reach an authenticated session without a verified second factor. |
| End users on financial-grade products (ZodiBank, ZodiTrade, ZodiXchange, ZodiCapital, ZodiYield) | Mandatory for all users, all roles. No opt-out. |
| End users on healthcare-grade products (ZodiMed) accessing PHI-equivalent records | Mandatory for all users with access to patient records. |
| End users on all other products | Optional, encouraged via an in-app prompt after first login and a recurring reminder until enabled. |

- Supported factors: TOTP (RFC 6238, compatible with standard authenticator
  apps) as the baseline every product MUST support; WebAuthn/FIDO2 security
  keys and platform authenticators SHOULD be offered as an additional option;
  SMS OTP MAY be offered only as a fallback recovery factor, never as the
  primary factor, because it is not phishing-resistant.
- Each account MUST be issued 10 single-use recovery codes at MFA enrollment,
  regenerable on demand, invalidating the previous set.
- Disabling MFA on an account that requires it (Admin/Owner/Billing role, or
  any role on a financial-grade product) MUST be blocked by policy, not just
  by UI — the underlying authorization check MUST reject the request
  server-side.

## Session management

- Authentication uses short-lived signed tokens (Laravel Sanctum for
  first-party SPA/API sessions, Passport-issued OAuth2 tokens for
  third-party integrations).
- Access token expiry: 15 minutes for financial-grade products, 60 minutes
  for all other products.
- Refresh tokens are long-lived (30 days), stored as httpOnly, `Secure`,
  `SameSite=Lax` cookies (never in `localStorage`), and are rotated on every
  use (refresh token rotation) — a used refresh token is immediately revoked
  and reuse triggers revocation of the entire token family plus a security
  alert to the account owner.
- Idle session timeout: 15 minutes of inactivity for financial-grade and
  healthcare-grade products, 8 hours for all other products, after which the
  refresh token is invalidated and re-authentication is required.
- Every product MUST expose a "Devices & Sessions" settings screen listing
  active sessions with device type, approximate location (derived from IP,
  never precise geolocation without consent), last-active timestamp, and a
  per-session "Revoke" action plus a "Revoke all other sessions" action. This
  UI is a hard requirement, not an enhancement — see
  [`../quality/definition-of-production-ready.md`](../quality/definition-of-production-ready.md).
- Logging in from a new device MUST trigger an email notification to the
  account owner containing device, approximate location, and a "This wasn't
  me" revocation link.

## SSO / SAML / OAuth2

- Every product MUST support OAuth2 "Sign in with Google" and "Sign in with
  Facebook"/"Sign in with LinkedIn" as first-party social login options via
  the base codebase's already-integrated `laravel/socialite` credentials
  screen (see
  [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
  and
  [`admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#general-settings--branding)),
  configured once by the buyer from the admin panel — excluded on
  financial-grade products, which require password+MFA only, given
  heightened identity-assurance requirements.
- A product whose target market expects enterprise SSO (OAuth2/OpenID
  Connect or SAML 2.0 SP-initiated) MAY add it as a scoped extension of the
  inherited auth engine, documented in that product's own
  [`SPEC.md`](../products/): SP metadata is generated once for the
  deployment, IdP metadata is uploaded by the business's own Admin/Owner from
  the admin panel, and just-in-time (JIT) user provisioning MUST map IdP
  group claims to the product's own roles per
  [`rbac-permissions.md`](./rbac-permissions.md). This is not inherited from
  the base codebase by default and is built once per product that needs it,
  not assumed to already exist.
- When the business enables an "SSO required" setting, password-based login
  for that deployment MUST be disabled at the application layer, not just
  hidden from the UI.

## Passwordless / magic-link authentication

- Every product MUST support email magic-link login as an alternative to
  password login: a single-use, time-limited (15 minutes) signed URL sent to
  the verified account email.
- Magic links MUST NOT themselves satisfy an MFA requirement — an account
  that requires MFA still completes a second factor after following the
  link.
- Magic-link requests are rate-limited to 3 per email address per 15 minutes
  to prevent email-bombing.

## Authorization: policy-based pattern

- Every model that is user-visible MUST have a corresponding Laravel Policy
  registered in the service provider; controllers MUST call `$this->authorize()`
  or the `can` middleware before performing any read or write — no controller
  branches on `if ($user->role === 'admin')` directly.
- Policies MUST check, in order: (1) the actor's effective permission for
  the resource/action pair per
  [`rbac-permissions.md`](./rbac-permissions.md), (2) any resource-level
  ownership or scoping rule (e.g., a Manager can only edit invoices in their
  own branch, on a product with multi-branch scoping — see
  [`localization-i18n.md`](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)).
  Failing any check returns `403`, never a silent empty result.
- Gates are reserved for cross-cutting, non-model checks (e.g., "can access
  the billing settings area"); Policies are used for every Eloquent model.
- Every authorization decision that denies access to a sensitive action MUST
  be captured by the audit log per
  [`audit-logging.md`](./audit-logging.md).

## API token scoping

- Personal access tokens and third-party API integration tokens are issued
  via Sanctum/Passport with explicit scopes following the same
  `resource.action` naming convention as permissions (e.g. `invoices.read`,
  `invoices.write`). A token MUST NOT be issued with a broader scope than the
  issuing user's own effective permissions.
- Tokens MUST support an optional expiry date, set by the issuing user, with
  a maximum lifetime of 1 year for standard products and 90 days for
  financial-grade products.
- Every token MUST be revocable individually from the "Devices & Sessions" /
  API tokens settings screen, and revocation MUST take effect within 60
  seconds (cache-invalidated, not just database-flagged).
- Webhook-signing and server-to-server integration credentials are scoped
  per-integration and MUST NOT reuse a human user's personal token.

## Related standards

- [`rbac-permissions.md`](./rbac-permissions.md)
- [`audit-logging.md`](./audit-logging.md)
- [`security-standards.md`](./security-standards.md)
- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
- [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)
