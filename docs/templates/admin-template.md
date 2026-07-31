# Admin (Back-Office) Template

Every Zodize product ships an internal administration surface, separate from
the customer-facing application, used by Zodize staff to operate the
product. This document specifies its scaffold. A product customizes which
domain-specific admin tools it adds; it does not customize the mandatory
sections below or weaken the impersonation audit requirement.

## Directory structure

```
app/
  Http/Controllers/Admin/
    TenantController.php
    ImpersonationController.php
    FeatureFlagController.php
    SystemHealthController.php
    SupportToolsController.php
  Policies/Admin/
    AdminAccessPolicy.php
resources/js/
  pages/Admin/
    Tenants/Index.vue
    Tenants/Show.vue
    FeatureFlags/Index.vue
    SystemHealth.vue
    SupportTools/Index.vue
  components/Admin/
    ImpersonationBanner.vue
    AdminNav.vue
routes/
  admin.php
```

The admin surface MUST be served behind a distinct route prefix (`/admin/**`)
and MUST require a Zodize-staff role that no tenant-provisioned user can
hold, per [`../security/rbac-permissions.md`](../security/rbac-permissions.md).

## Tenant management

`Tenants/Index.vue` MUST provide: search/filter by tenant name, plan, status
(trial/active/suspended/churned), and creation date; and bulk-safe actions
(suspend, reinstate, extend trial) that require a confirmation step.

`Tenants/Show.vue` MUST provide, at minimum: tenant metadata, plan and
billing status (read-only mirror of ZodiCore billing, not a duplicate
billing UI), user list for that tenant, feature flag overrides scoped to
that tenant, and a link into impersonation.

## User impersonation

Impersonation MUST follow this exact flow:

1. Admin selects a tenant user from `Tenants/Show.vue` and provides a reason
   (free-text, required, minimum 10 characters) before impersonation starts.
2. The system MUST write an audit log entry at the moment impersonation
   **starts**, containing the admin's identity, the impersonated user's
   identity, the tenant, the stated reason, and a timestamp, per
   [`../security/audit-logging.md`](../security/audit-logging.md). This
   entry is mandatory and non-optional — impersonation MUST NOT be possible
   through any code path that skips it.
3. `ImpersonationBanner.vue` MUST render on every page for the duration of
   the impersonated session, unmissable (fixed position, high-contrast),
   showing "Impersonating {user} — {admin} — Exit" at all times.
4. The system MUST write a second, corresponding audit log entry when
   impersonation **ends**, whether ended explicitly or via session timeout,
   including the duration.
5. Impersonated sessions MUST be time-boxed (maximum 60 minutes) and MUST
   auto-terminate, re-entering as the admin's own identity.
6. An impersonated session MUST NOT be able to change the impersonated
   user's password, MFA enrollment, or billing details — read and
   support-relevant-write access only, enforced by `AdminAccessPolicy`.

## Feature flag management

`FeatureFlags/Index.vue` MUST provide: a list of all registered flags (see
[permission-template.md](./permission-template.md) for the registration
pattern flags follow), current global default, and per-tenant overrides.
Toggling a flag MUST write an audit log entry. Flag changes MUST take effect
without a deploy. This UI is the operator surface for the rollout stages
defined in [release-template.md](./release-template.md).

## System health

`SystemHealth.vue` MUST surface, at minimum: queue depth and failure rate,
scheduled job last-run status, third-party integration status (payment
processor, email provider, SMS provider), and current deploy
version/commit. It MUST pull from the same health-check endpoints used by
the deployment gate in [deployment-template.md](./deployment-template.md),
not a separate parallel health implementation.

## Support tools

`SupportTools/Index.vue` MUST provide, at minimum: a tenant/user lookup by
email, the ability to manually trigger a password-reset email, the ability
to manually resend a stuck notification (per
[`../architecture/`](../architecture) notification system), and a read-only
view of a user's recent audit log entries. Any action taken from support
tools that mutates tenant data MUST itself produce an audit log entry.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: `AdminAccessPolicy`, `ImpersonationController` and its
mandatory audit hooks, `ImpersonationBanner.vue`, `FeatureFlagController`,
and the system health endpoint contract.

A product customizes: domain-specific entries in `SupportTools/Index.vue`
and any product-specific tenant metadata surfaced in `Tenants/Show.vue`. A
product MUST NOT implement its own impersonation mechanism outside
`ImpersonationController` — doing so bypasses the mandatory audit trail and
is treated as a security defect.
