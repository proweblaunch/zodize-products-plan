# ZodiCore — Product Specification

> Status: **Reference-depth**. ZodiCore is the platform every other Zodize
> product is built on. This specification is the template other product
> specs are measured against — see
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

Companion documents: [DATA_MODEL.md](./DATA_MODEL.md) (full ER model),
[API_REFERENCE.md](./API_REFERENCE.md) (endpoint catalog).

## 1. Vision

ZodiCore is the operating platform for every Zodize product: one identity
system, one billing engine, one notification fabric, one permission model,
one plugin runtime, one audit trail — built once to an enterprise-grade
standard so that every vertical product (ZodiBank, ZodiMed, ZodiCommerce,
and the rest of the [catalog](../../../PRODUCT_CATALOG.md)) inherits trust,
extensibility, and operability from day one instead of re-earning it
product by product.

## 2. Purpose

Without ZodiCore, each of the twenty Zodize products would reimplement
login, billing, RBAC, and notifications with twenty different bugs. ZodiCore
exists to make "which product" an implementation detail on top of a shared,
hardened core — the same reason AWS has IAM once, not once per service.

## 3. Target Market

Internal: every Zodize product team. External surface: enterprise IT/security
teams evaluating any Zodize product will, in practice, be evaluating
ZodiCore's identity, audit, and compliance posture, since it underlies all of
them. ZodiCore itself is never sold standalone.

## 4. Industries

Cross-industry — ZodiCore is industry-agnostic by design; all industry
specificity lives in vertical product modules layered on top.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Identity/SSO/RBAC | Auth0, WorkOS, AWS Cognito | Native, first-party across every product — no per-product IAM integration project |
| Multi-tenant billing | Stripe Billing, Chargebee | Deeply integrated with per-tenant entitlement flags and usage metering across 20 verticals |
| Plugin/extension runtime | Salesforce AppExchange platform layer, Shopify app platform | Consistent hook model across every vertical, not just one product |
| Notifications fabric | Courier, Novu | Owns the full fan-out (in-app/email/SMS/push/webhook) from one domain event |
| Audit/compliance | Vanta-adjacent audit trail tooling | Built into the data model, not bolted on for a SOC 2 audit |

## 6. Personas

- **Tenant Owner** — the customer-org admin who provisions the account,
  manages billing, and assigns Admin roles.
- **Tenant Admin** — configures roles, integrations, and plugins for their
  organization.
- **End User** — uses a vertical product's features under a role Tenant Admin
  assigned; interacts with ZodiCore only implicitly (login, notifications).
- **Developer/Integrator** — a third party or internal team building against
  the API/SDK/webhooks/plugin system.
- **Zodize Support/Ops** — uses impersonation and admin tooling to assist
  tenants, fully audited.
- **Zodize Platform Engineer** — builds and operates ZodiCore itself.

## 7. User Journeys

1. **Tenant signup → first login**: prospect signs up → tenant provisioned
   (see [DATA_MODEL.md](./DATA_MODEL.md#tenants)) → verification email sent
   → Owner completes profile → default roles seeded → guided onboarding
   checklist widget appears on the dashboard (per
   [dashboard-standards.md](../../standards/dashboard-standards.md)).
2. **Inviting a teammate**: Owner/Admin invites by email with a role →
   invitee receives branded email → accepts → account created and scoped to
   the tenant → audit log entry recorded.
3. **Upgrading plan**: Owner hits an entitlement-flag-gated feature → sees an
   upgrade prompt → completes checkout via the billing module → entitlement
   flags update in real time, no redeploy.
4. **Developer integration**: Developer creates a scoped API token in
   Settings → Developer → reads the OpenAPI-generated docs → registers a
   webhook endpoint → verifies signature using the SDK helper → goes live.
5. **Support impersonation**: Support agent requests time-boxed impersonation
   of a tenant with the tenant's consent flag set → session banner clearly
   marks impersonation → every action is audit-logged against the real
   support agent's identity, not the impersonated user's.
6. **Plugin install**: Tenant Admin browses the marketplace → installs a
   plugin → reviews the requested permission scope → plugin activates,
   registers its hooks, runs its migrations in an isolated schema namespace.

## 8. Business Goals

- Reduce time-to-production-ready for a new vertical product by providing
  identity, billing, notifications, and permissions "for free."
- Give every Zodize product SOC 2-ready audit logging from day one.
- Enable a marketplace revenue line across the whole product portfolio
  through one shared plugin/extension runtime.

## 9. Functional Requirements

- Tenant provisioning, suspension, and deprovisioning lifecycle.
- Authentication: email/password, MFA (TOTP + WebAuthn), SSO (SAML 2.0,
  OIDC), magic link, session/device management — see
  [authentication-authorization.md](../../security/authentication-authorization.md).
- Authorization: full RBAC with custom role builder — see
  [rbac-permissions.md](../../security/rbac-permissions.md).
- Billing: subscription plans, usage-based add-ons, invoicing, dunning,
  payment method management.
- Notifications: in-app, email, SMS, push, webhook fan-out from one event.
- Plugin runtime: install/enable/disable/uninstall, permission scoping,
  marketplace listing.
- Audit log and activity timeline across every tenant action.
- Global search across all installed modules' indexed entities.
- Feature flag and entitlement management, exposed to every product.
- Admin console: tenant management, impersonation, system health, support
  tools — see [admin-template.md](../../templates/admin-template.md).
- Developer portal: API tokens, webhook management, OpenAPI docs, SDK
  downloads.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiCore-specific additions:

- Identity/auth endpoints: p95 < 150ms (stricter than the general API budget,
  since every request across every product depends on a fast auth check).
- 99.95% uptime SLA target for the auth and entitlement-check services
  specifically, since their failure cascades to every product.
- Horizontal scalability of the notification fan-out queue to handle
  platform-wide traffic spikes (e.g. a mass password-reset event) without
  degrading per-tenant delivery latency.

## 11. Architecture

ZodiCore is a modular monolith (see
[architecture/overview.md](../../architecture/overview.md)) exposing its
capabilities to vertical products via: (a) shared PHP packages
(`zodize/core-identity`, `zodize/core-billing`, `zodize/core-notifications`,
`zodize/core-permissions`, `zodize/core-plugins`) consumed as Composer
dependencies within each product's Laravel app, and (b) a platform API for
cross-product operations (e.g. the admin console managing tenants across
products). Vertical products are not separate runtime services calling
ZodiCore over the network for every request — identity/permission checks
happen in-process via the shared package for latency and reliability, while
platform-wide operations (billing, marketplace, cross-product admin) go
through ZodiCore's own API.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); Stripe
(or Stripe-compatible) as the primary payment gateway integration for
platform billing (see §20); SAML/OIDC via a standards-compliant library, not
a hand-rolled implementation.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Identity | Authentication, MFA, SSO, Session/Device Management, Impersonation |
| Tenancy | Tenant Provisioning, Multi-Company/Branch Scoping, Domain Management |
| Permissions | RBAC Engine, Custom Role Builder, Policy Registry |
| Billing | Subscriptions, Invoicing, Payment Methods, Dunning, Usage Metering |
| Notifications | In-App, Email, SMS, Push, Webhook Fan-out, Preferences |
| Plugins & Marketplace | Plugin Runtime, Manifest/Permission Scoping, Marketplace Listing, Reviews |
| Audit | Audit Log, Activity Timeline, Version History |
| Search | Global Search Index, Saved Filters |
| Admin Console | Tenant Management, Impersonation Tooling, System Health, Feature Flags |
| Developer Portal | API Tokens, Webhook Management, Docs, SDKs |

## 14. Database Design

See [DATA_MODEL.md](./DATA_MODEL.md) for the full entity list, columns, and
ER diagram. Core entities: `tenants`, `companies`, `branches`, `users`,
`roles`, `permissions`, `role_user`, `subscriptions`, `plans`, `invoices`,
`payment_methods`, `notifications`, `notification_preferences`,
`audit_logs`, `plugins`, `tenant_plugins`, `api_tokens`, `webhooks`,
`webhook_deliveries`, `feature_flags`, `tenant_feature_flags`.

## 15. API Endpoints

See [API_REFERENCE.md](./API_REFERENCE.md) for the full endpoint catalog
(all conforming to [api-standards.md](../../development/api-standards.md)
and [rest-standards.md](../../development/rest-standards.md)).

## 16. Events

Domain events (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`tenant.provisioned`, `tenant.suspended`, `user.invited`,
`user.registered`, `user.login_succeeded`, `user.login_failed`,
`user.mfa_enrolled`, `role.assigned`, `role.revoked`,
`subscription.created`, `subscription.upgraded`, `subscription.canceled`,
`invoice.issued`, `invoice.paid`, `invoice.payment_failed`,
`plugin.installed`, `plugin.uninstalled`, `webhook.delivery_failed`,
`impersonation.started`, `impersonation.ended`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `user.invited` | — | ✔ (invite) | — | — |
| `user.login_failed` (3+ in 10 min) | ✔ | ✔ (security alert) | ✔ (if MFA phone on file) | ✔ |
| `subscription.payment_failed` | ✔ | ✔ | — | ✔ |
| `role.assigned` | ✔ | ✔ | — | — |
| `webhook.delivery_failed` (endpoint disabled) | ✔ | ✔ | — | — |
| `impersonation.started` | ✔ (to tenant Owner) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Default system roles: `Owner`, `Admin`, `Manager`, `Member`, `Viewer`,
`Billing`, `Support/Impersonator` — per
[rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles).
ZodiCore-specific permissions: `tenant.manage`, `billing.manage`,
`roles.manage`, `plugins.manage`, `api_tokens.manage`,
`webhooks.manage`, `impersonation.perform` (Zodize-internal role only,
never assignable within a tenant).

## 19. Workflows & Approval Chains

- **Plan downgrade approval**: if a downgrade would exceed the new plan's
  seat/usage limits, the downgrade enters a pending state requiring the
  Owner to resolve the overage before it takes effect.
- **Plugin install approval**: installing a plugin requesting sensitive
  scopes (e.g. financial data access) requires re-confirmation with the
  scope list explicitly displayed, matching
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **Impersonation approval**: impersonation requires either tenant
  consent-flag opt-in or a support-ticket-linked, time-boxed grant, always
  logged per §21.

## 20. Payment Gateways, Wallet, Accounting, Taxes, Invoices

- **Payment gateways**: Stripe as primary; gateway abstraction layer
  (`PaymentGatewayContract`) so vertical products with region-specific needs
  (e.g. ZodiBank, ZodiXchange) can register additional gateways without
  touching core billing logic.
- **Wallet**: platform-level tenant wallet for prepaid usage credits
  (metered add-ons draw down the wallet before falling back to invoiced
  billing).
- **Accounting**: billing events post to an internal ledger (double-entry,
  append-only) that reconciles against Stripe; this ledger is the system of
  record for revenue reporting, not the payment gateway's dashboard.
- **Taxes**: tax calculation via a tax-engine integration (rate lookup by
  tenant billing address), tax-inclusive/exclusive display per locale per
  [localization-i18n.md](../../standards/localization-i18n.md).
- **Invoices**: generated per billing cycle, PDF export, itemized by plan +
  usage add-ons, immutable once issued (corrections via credit note, never
  edited in place).

## 21. Documents, Exports, Imports

Invoices and receipts are generated as PDFs stored in tenant-scoped document
storage. Tenant data export (GDPR-style full export) is available to Owners
per [data-protection-privacy.md](../../security/data-protection-privacy.md).
Bulk user import via CSV with a mapping wizard, per
[import-wizard requirements](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Search, Filters, Global Search

ZodiCore provides the shared global search infrastructure (index, query API,
`⌘K` command palette integration per
[navigation-standards.md](../../standards/navigation-standards.md)) that
every vertical product registers its own entities into via a
`SearchableContract` — global search is a platform capability, not
reimplemented per product.

## 23. AI Features

- AI-assisted support: an in-product assistant that can answer "how do I...
  " questions grounded in that product's documentation, and take permitted
  actions on the user's behalf via the same permission-scoped API every
  human integration uses (never a backdoor bypassing RBAC).
- AI-assisted admin: anomaly detection on audit logs (e.g. unusual bulk
  export activity) surfaced to Tenant Admins and Zodize Support.

## 24. Automation, Scheduled Jobs, Cron Jobs, CLI Commands

- Scheduled jobs: subscription renewal processing, dunning retries, webhook
  delivery retries, audit log retention pruning, trial-expiration reminders.
- CLI commands (Artisan): `tenant:provision`, `tenant:suspend`,
  `plugin:publish`, `flags:sync`, `billing:reconcile` — every CLI command
  requires the same authorization context as its API equivalent when run
  against production data (no CLI bypass of RBAC for scripted actions).

## 25. Seed Data, Demo Data

`DemoSeeder` provisions 3 demo tenants with realistic org structures (varied
company/branch counts), populated billing history (12 months, including at
least one failed-then-recovered payment), populated notification history,
and a populated audit log spanning role changes, logins, and a plugin
install — per [migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: tenant provisioning completes end-to-end (DB +
default roles + welcome notification) in under 3 seconds p95.

## 27. Security Requirements

Full baseline from [docs/security/](../../security/security-standards.md)
applies, plus: ZodiCore's identity service is the single point where a
compromised credential is rotated and immediately invalidates sessions
platform-wide; penetration testing of the identity/billing modules occurs
quarterly, higher frequency than the general annual baseline.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated cross-tenant isolation test suite that runs against every
ZodiCore-owned table as part of the required CI gate, since a leak here
compromises every product.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
ZodiCore's identity/billing services are deployed with a stricter
zero-downtime requirement (no maintenance-window auth outages) than
individual vertical product deploys.

## 30. Acceptance Criteria

- A new tenant can be provisioned, invite a teammate, assign a role, and
  have that teammate log in — end to end — with zero manual intervention.
- A tenant can upgrade/downgrade plan and see entitlement flags reflect the
  change without a deploy.
- A registered webhook receives a correctly signed, replay-protected
  delivery for every subscribed event type within the SLA in
  [webhook-standards.md](../../development/webhook-standards.md#delivery-guarantees).
- Impersonation is impossible without either tenant consent or a logged,
  time-boxed grant, and always produces an audit trail.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiCore additionally requires sign-off that every other in-flight product
spec's identity/billing/notification assumptions have been validated against
the actual ZodiCore API before ZodiCore itself is marked Production Ready.

## 32. Future Roadmap

- Expand SSO support to SCIM-based automated provisioning/deprovisioning.
- Usage-based billing rate cards configurable per-tenant for enterprise
  custom contracts.
- Plugin marketplace revenue analytics dashboard for third-party developers.

## 33. Known Risks

- Shared-core failure blast radius: a defect in ZodiCore's auth path affects
  every product simultaneously — mitigated by the stricter testing/SLA
  requirements above, but this remains the platform's single largest risk
  and the primary justification for its elevated release-control bar.
- Plugin runtime security: third-party plugins are a persistent attack
  surface — mitigated by the marketplace review pipeline in
  [marketplace-architecture.md](../../architecture/marketplace-architecture.md),
  but sandboxing depth should be re-evaluated as the marketplace scales.

## 34. Future Improvements

- Field-level encryption key rotation without downtime.
- Per-tenant data residency selection for regulated customers.

## Roadmap (spec depth)

This spec is reference-depth and considered complete. Companion documents
([DATA_MODEL.md](./DATA_MODEL.md), [API_REFERENCE.md](./API_REFERENCE.md))
will expand as implementation surfaces additional entities/endpoints; changes
there follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).
