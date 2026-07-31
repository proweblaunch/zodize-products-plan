# Authentication Template

This document specifies the concrete pages, routes, and states every Zodize
product ships for authentication. The underlying rules — password policy,
MFA requirements, session lifetime, SSO protocol support — are defined in
[`../security/authentication-authorization.md`](../security/authentication-authorization.md);
this document is the scaffold that implements those rules. A product
customizes branding on these pages; it does not customize which pages,
routes, or states exist, and it does not customize the underlying auth rules
without an ADR.

## Directory structure

```
app/
  Http/Controllers/Auth/
    RegisterController.php
    LoginController.php
    MfaController.php
    PasswordResetController.php
    EmailVerificationController.php
    SsoController.php
    SessionController.php
resources/js/
  pages/Auth/
    Register.vue
    Login.vue
    MfaEnroll.vue
    MfaChallenge.vue
    ForgotPassword.vue
    ResetPassword.vue
    VerifyEmail.vue
    SsoRedirect.vue
    Sessions.vue          # active session/device management
routes/
  auth.php
```

## Required flows, routes, and states

Every flow below MUST implement all four states — **default/loading, error,
success, and empty (where applicable)** — before it ships. A flow missing
the error or loading state is not production-ready per
[`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md).

### 1. Registration (`/register`)

- Fields: name, email, password, password confirmation, terms-of-service
  acceptance checkbox (required, linking to the marketing site's
  `/legal/terms-of-service`).
- States: `idle` → `submitting` (loading) → `success` (redirect to email
  verification pending screen) or `error` (field-level validation errors,
  plus a form-level error for server/rate-limit failures).
- MUST enforce the password policy from
  [`../security/authentication-authorization.md`](../security/authentication-authorization.md)
  client-side (real-time strength feedback) and server-side (authoritative).

### 2. Login (`/login`)

- Fields: email, password, "remember me" checkbox, link to `/forgot-password`.
- States: `idle` → `submitting` → `success` (redirect to MFA challenge if
  enrolled, else to dashboard per [dashboard-template.md](./dashboard-template.md))
  or `error` (invalid credentials — MUST use a generic message that does not
  disclose whether the email exists; rate-limit lockout — MUST show a
  countdown, not a hard dead-end).

### 3. MFA enrollment (`/settings/security/mfa/enroll`)

- States: `not-enrolled` (prompt with a "why" explainer and a skip option
  only where MFA is not mandated for the account's role/product tier),
  `enrolling` (QR code + manual entry key + verification code input),
  `verifying` (loading), `success` (recovery codes displayed exactly once,
  with a mandatory "I have saved these" confirmation before proceeding), and
  `error` (invalid code).

### 4. MFA challenge (`/login/mfa`)

- States: `awaiting-code`, `submitting`, `error` (invalid code, with attempt
  counter), `success` (redirect to originally requested route), and a
  `use-recovery-code` alternate path.

### 5. Password reset (`/forgot-password`, `/reset-password/{token}`)

- `/forgot-password` states: `idle` → `submitting` → `success` (generic
  "if that email exists, a reset link was sent" message regardless of
  outcome, to avoid account enumeration).
- `/reset-password/{token}` states: `idle` (token valid), `token-invalid`
  (expired/used token — explicit distinct state, not a generic error),
  `submitting`, `success` (auto-redirect to login with a confirmation
  banner), `error` (password policy violation).

### 6. Email verification (`/verify-email`)

- States: `pending` (instructs the user to check email, with a
  resend-with-cooldown action), `verifying` (token click-through, loading),
  `success`, `error` (expired/invalid token, with a resend action).
- Unverified accounts MUST be restricted per the access rules in
  [`../security/authentication-authorization.md`](../security/authentication-authorization.md);
  this template only owns the page states, not the authorization rule.

### 7. Social login entry point (`/login/redirect/{provider}`, `/login/callback/{provider}`)

Every product inherits the base codebase's existing social login
integration (Google/Facebook/LinkedIn via `laravel/socialite`, configured
per
[`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#general-settings--branding))
rather than implementing enterprise SSO/SAML from scratch — there is no
multi-tenant identity provider to resolve against, since each deployment
belongs to exactly one business (see
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)).

- `/login/redirect/{provider}`: `resolving` (loading) → redirect to the
  configured OAuth provider, or `error` (that provider is not enabled by
  the buyer's admin, with a fallback link to standard login).
- `/login/callback/{provider}`: `processing` (loading) → `success` (redirect
  to dashboard) or `error` (provider rejected the login, or the account is
  not provisioned — MUST show actionable next steps, not a raw provider
  error).

### 8. Session and device management (`/settings/security/sessions`)

- Lists all active sessions with device, approximate location (derived from
  IP, never stored as precise geolocation), and last-active timestamp.
- States: `loading`, `loaded` (list, current session marked distinctly and
  non-revocable from itself), `empty` (cannot occur for the viewing session
  itself but MUST be handled defensively), `revoking` (per-row loading),
  `revoked` (row removed with confirmation toast).
- MUST include a "revoke all other sessions" bulk action.

## What the inherited base codebase provides vs. what a product customizes

The base codebase (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md))
provides: every controller listed above, the MFA TOTP/recovery-code
implementation, the social login OAuth handlers (Google/Facebook/LinkedIn
via `laravel/socialite`), rate limiting, and the session store, all running
within the product's own three auth guards (`web`, `admin`, `branch_staff`)
per [`admin-template.md#auth-guards`](./admin-template.md#auth-guards).

A product customizes: page copy, logo/branding on the auth layout, and
which social login providers are enabled and configured by the buyer from
the admin panel. A product MUST NOT implement its own registration/login
controller from scratch, and MUST NOT call out to another Zodize product or
any Zodize-operated identity service — each product's authentication is
fully self-contained within its own codebase per
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#no-shared-platform-service).
