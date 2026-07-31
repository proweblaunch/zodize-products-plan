# ZodiCore — Product Specification

> Status: **Reference-depth**. ZodiCore is an ordinary, standalone Zodize
> product — a general-purpose back-office/ERP starter, closest in shape to
> the shared base codebase itself. This specification is the template other
> product specs are measured against for depth and rigor, not because
> ZodiCore is architecturally special — see
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

Companion documents: [DATA_MODEL.md](./DATA_MODEL.md) (full ER model),
[API_REFERENCE.md](./API_REFERENCE.md) (endpoint catalog).

## 1. Vision

ZodiCore is the general-purpose back-office and light operations product in
the Zodize catalog: one deployable, self-hosted Laravel application giving a
small or mid-size business a working admin console — settings/branding,
staff accounts with granular roles, a wallet/ledger, configurable payment
gateways, a referral program, subscription-style plans, KYC, multilingual
support, and a page builder — plus a generic task/project tracker and a
flexible records module for whatever structured data the business needs to
track that doesn't warrant its own vertical product. It is the product
closest in shape to the shared Zodize base codebase
([base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)),
built for buyers who want a solid operational back office without a
hospitality, fleet, agriculture, or construction-specific domain layered on
top.

## 2. Purpose

Not every business needs ZodiHotel's room inventory or ZodiFleet's
telematics ingestion. A large segment of Zodize's buyers — consultancies,
agencies, cooperatives, small service businesses, internal ops teams —
need exactly what the base engine already provides (accounts, money
movement, staff permissions, multilingual content) plus a lightweight way
to track tasks/projects and arbitrary structured records, without paying
for or maintaining domain modules they will never use. ZodiCore exists to
be that product: the leanest, most direct expression of the base codebase's
own value, sold on its own merits.

## 3. Target Market

Small and mid-size service businesses, consultancies, agencies, cooperatives,
and internal operations teams that need an admin back office, wallet/
payments, staff roles, and lightweight task/records tracking, but have no
need for a vertical (hospitality, fleet, agriculture, construction, medical,
etc.) domain layer. Also a common starting point for a buyer evaluating
Zodize's product quality before purchasing a vertical product, since
ZodiCore best demonstrates the shared base engine with the least
domain-specific noise.

## 4. Industries

Cross-industry — ZodiCore is deliberately industry-agnostic. Buyers with a
specific vertical need are better served by that vertical's own product
(e.g. [ZodiHotel](../ZodiHotel/SPEC.md), [ZodiFleet](../ZodiFleet/SPEC.md));
ZodiCore is for buyers whose operations don't require one.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Back-office/admin starter | Laravel Nova, Filament-based admin kits | Ships a complete business back office (wallet, gateways, KYC, referrals, i18n) out of the box, not just a CRUD scaffold |
| Simple task/project tracker | Trello, basic Asana tier | Native to the same back office as the wallet and staff roles — one login, one system, not a separate SaaS subscription |
| Generic records/database tool | Airtable, simple Notion database use | Self-hosted, buyer-owned data with the same RBAC and audit trail as every other module, not a third-party-hosted spreadsheet |
| White-label back-office SaaS kits | SaaS boilerplates (various) | Sold once as source code the buyer owns forever, not a recurring-fee boilerplate license |

## 6. Personas

- **Business Owner/Admin** — configures the deployment (branding, gateways,
  roles, plans), oversees wallet activity and staff, the primary buyer.
- **Staff Member** — uses assigned modules (tasks, records) under a role the
  Admin configured; day-to-day user of the product.
- **Finance/Bookkeeping Staff** — reviews wallet transactions, deposits,
  and withdrawals; manages payment gateway configuration.
- **Referral Partner** — an end user who refers other users and earns
  commission per the configured referral program.
- **End Customer** (where ZodiCore is used customer-facing rather than
  purely internal) — registers, holds a wallet balance, submits KYC,
  interacts with the deployment as an ordinary product user.

## 7. User Journeys

1. **Install to first login**: buyer uploads the codebase to their hosting,
   imports the SQL dump, sets database credentials in `.env`, and logs into
   the seeded default Admin account → completes the General Settings wizard
   (site name, logo, currency, timezone) → the deployment is live with zero
   further code involvement.
2. **Staff onboarding**: Admin creates a role with a scoped permission set
   (e.g. "Bookkeeper" limited to wallet/withdrawal screens) → invites a
   staff member by email → staff member sets a password and logs in, seeing
   only the modules their role grants.
3. **Wallet activity**: an end user deposits via a configured payment
   gateway → the gateway webhook credits their balance, producing a
   `Transaction` row → user later requests a withdrawal via a configured
   withdrawal method → Admin reviews and approves the withdrawal, which
   debits the balance and produces its own `Transaction`.
4. **Task tracking**: a staff member creates a project, adds tasks with
   assignees and due dates, and moves tasks across a status board →
   teammates comment and attach files → the Admin reviews a project's
   completion status from a dashboard widget.
5. **Generic records use**: an Admin defines a custom record type (e.g.
   "Vendor," "Equipment Asset," "Client Contract") with an admin-defined
   field schema → staff create and search records of that type → no code
   change is required to add a new record type or field.
6. **Referral commission flow**: an existing user shares their referral
   link → a new user signs up via that link → on the new user's first
   deposit, a referral commission is credited to the referrer's wallet per
   the Admin-configured commission percentage.

## 8. Business Goals

- Give a buyer with no vertical-specific need a complete, credible business
  back office at a fraction of a custom-build cost.
- Demonstrate the shared base engine's quality as a standalone product,
  since ZodiCore is the product with the fewest domain-specific modules
  layered on top of it.
- Provide a lightweight operations layer (tasks, records) sufficient for a
  small team without requiring a separate project-management SaaS
  subscription.

## 9. Functional Requirements

- Everything in the inherited base engine, unmodified: general settings/
  branding, wallet/ledger, payment gateways, withdrawal methods, referral
  program, plans, KYC, language/i18n, cron engine, extensions, CMS/page
  builder, roles & permissions, notifications — see
  [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)
  for the full inventory.
- **Task/project tracker**: projects containing tasks with assignee, due
  date, priority, and status (a configurable Kanban-style board); comments
  and file attachments per task; a per-project completion dashboard widget.
- **Generic records module**: admin-defined record types with a dynamic
  field schema (text, number, date, dropdown, file — form-builder driven,
  reusing the same schema pattern as the inherited KYC form builder),
  per-type list/detail views, and search/filter across records of a type.
- **Plugin/extension marketplace** (optional per deployment): an
  admin-installable plugin runtime per
  [plugin-architecture.md](../../architecture/plugin-architecture.md) and
  [marketplace-architecture.md](../../architecture/marketplace-architecture.md),
  letting a buyer extend their deployment with third-party or
  Zodize-published plugins without a code change — see
  [§22 Integrations](#22-integrations).
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (e.g. high-value withdrawal approval), automation rules,
  saved filters on records/tasks, custom fields, full audit history,
  soft-delete/restore, mass actions, command palette, report builder.

## 10. Non-Functional Requirements

Baseline from [performance-standards.md](../../quality/performance-standards.md)
and [security-standards.md](../../security/security-standards.md) applies.
ZodiCore carries no additional non-functional requirement beyond that
baseline — unlike a vertical product with a domain-specific latency or
throughput constraint (e.g. ZodiHotel's booking-engine SLA), ZodiCore's own
new modules (tasks, records) are ordinary CRUD workloads.

## 11. Architecture

ZodiCore is a standalone, self-hosted Laravel application, sold as source
code to one buyer and deployed entirely within that buyer's own hosting
account, exactly like every other Zodize product — there is no shared
platform service and no other Zodize product it depends on at runtime
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
Earlier drafts of this handbook described ZodiCore as "the platform every
other Zodize product is built on," providing shared identity, billing,
notifications, and tenancy that every other product called over the
network. **That model is retired and MUST NOT be assumed anywhere in this
handbook.** ZodiCore does not run as a service other products call; it does
not provision, suspend, or manage any other product's deployment; and no
other product has a runtime dependency on it.

ZodiCore is built the same way every other product is: clone the sanitized
base codebase and run the
[genericization checklist](../../architecture/product-genericization-checklist.md).
Because ZodiCore's own domain modules (task tracker, generic records) are
intentionally light, this genericization pass is the largest part of
ZodiCore's build — strip the banking-specific tables (`loans`, `dps`,
`fdr`, `branches`/`branch_staff`, `other_banks`, `beneficiaries`,
`airtime_operators`), rename hardcoded identity strings, and confirm the
inherited engine (wallet/ledger, payment gateways, RBAC/auth, KYC, i18n,
admin configuration, CMS) is presented as ZodiCore's own product surface —
see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
ZodiCore keeps the `web` and `admin` guards from the base and does not
re-add the `branch_staff`-equivalent guard by default, since a plain
back-office deployment has no property/depot/branch-scoped staff concept
built in; a buyer needing multi-company scoping uses the `company_id`
scoping layer per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
the same way any other product does. There is no `tenant_id` anywhere in
ZodiCore's schema.

Where ZodiCore ships the optional plugin/marketplace system (per
[plugin-architecture.md](../../architecture/plugin-architecture.md)), that
system extends *this one deployment only* — a plugin installed into one
buyer's ZodiCore instance has no visibility into, or effect on, any other
buyer's separately purchased and deployed ZodiCore instance. There is no
Zodize-hosted cross-deployment plugin registry that phones home at runtime;
the marketplace is a catalog a buyer's admin panel fetches listings from at
install time, per
[marketplace-architecture.md](../../architecture/marketplace-architecture.md).

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
MySQL/MariaDB + optional Redis cache per
[database-standards.md](../../development/database-standards.md), matching
the base codebase's inherited stack rather than introducing a different
database engine than every other product.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| General Settings & Branding | Site Identity, Currency/Timezone, Social Login (inherited) |
| Wallet & Ledger | Balances, Transactions, Deposits, Withdrawals, Balance Transfers (inherited) |
| Payment Gateways | Gateway Configuration, Currency Conversion (inherited) |
| Referral Program | Multi-level Commissions, Trigger Configuration (inherited) |
| Plans | Plan CRUD, Features/Limits (inherited, genericized) |
| KYC | Form Builder, Submission Review (inherited) |
| Language / i18n | Language Management, Translation Editing (inherited) |
| Roles & Permissions | Role Builder, Permission Assignment (inherited) |
| CMS / Page Builder | Sections, SEO, Policy Pages, Sitemap (inherited) |
| Task Tracker | Projects, Tasks, Boards, Comments, Attachments |
| Generic Records | Record Type Builder, Dynamic Fields, List/Detail Views, Search |
| Plugins & Marketplace (optional) | Plugin Runtime, Manifest/Permission Scoping, Marketplace Listing |
| Reporting | Wallet/Transaction Dashboards, Task Completion, Report Builder |

## 14. Database Design

See [DATA_MODEL.md](./DATA_MODEL.md) for the full entity list, columns, and
ER diagram. Core entities inherited from the base engine as-is: `users`,
`transactions`, `balance_transfers`, `general_settings`, `roles`,
`permissions`, `role_user`, `gateways`, `gateway_currencies`,
`withdraw_methods`, `withdrawals`, `plans`, `forms` (KYC schema), `kyc_data`
(on `users`), `languages`, `frontends`, `pages`. ZodiCore's own domain
tables: `projects`, `tasks`, `task_comments`, `task_attachments`,
`record_types`, `records`, and, where the plugin system is enabled,
`plugins`, `installed_plugins`, `webhooks`, `webhook_deliveries`. There is
no `tenants` table and no subscriptions-as-a-service billing schema — see
[DATA_MODEL.md](./DATA_MODEL.md#why-there-is-no-tenants-table) for why.

## 15. API Endpoints

See [API_REFERENCE.md](./API_REFERENCE.md) for the full endpoint catalog
(all conforming to [api-standards.md](../../development/api-standards.md)
and [rest-standards.md](../../development/rest-standards.md)). The surface
is the single-business API every ZodiCore deployment exposes to its own
frontend and to integrations the buyer configures — there is no
tenant-scoped or cross-deployment endpoint category.

## 16. Events

Domain events (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`user.registered`, `user.login_succeeded`, `user.login_failed`,
`deposit.completed`, `withdrawal.requested`, `withdrawal.approved`,
`referral.commission_earned`, `role.assigned`, `role.revoked`,
`kyc.submitted`, `kyc.approved`, `kyc.rejected`, `project.created`,
`task.assigned`, `task.completed`, `record.created`, `record.updated`,
`plugin.installed`, `plugin.uninstalled`, `webhook.delivery_failed`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `user.login_failed` (3+ in 10 min) | ✔ | ✔ (security alert) | ✔ (if phone on file) | ✔ |
| `deposit.completed` | ✔ | ✔ | — | — |
| `withdrawal.approved` | ✔ | ✔ | — | ✔ |
| `kyc.rejected` | ✔ | ✔ | — | — |
| `role.assigned` | ✔ | ✔ | — | — |
| `task.assigned` | ✔ (assignee) | — | — | ✔ |
| `webhook.delivery_failed` (endpoint disabled) | ✔ (admin) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Default admin roles per
[rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles),
assigned entirely from the admin panel. ZodiCore-specific permissions:
`wallet.manage`, `withdrawals.approve`, `gateways.manage`, `plans.manage`,
`kyc.review`, `roles.manage`, `projects.manage`, `tasks.manage`,
`record_types.manage`, `records.manage`, `plugins.manage` (where the
plugin system is enabled), `webhooks.manage`. A Staff role by default can
manage tasks/records assigned to them but cannot approve withdrawals or
manage gateway credentials without an elevated role.

## 19. Workflows & Approval Chains

- **Withdrawal approval**: every withdrawal request enters a pending state
  and requires Admin (or a role holding `withdrawals.approve`) review
  before the debit and payout are finalized, per
  [wallet-system.md](../../standards/wallet-system.md).
- **KYC review**: a submitted KYC form requires an Admin decision
  (approve/reject with a reason) before the user's KYC status changes.
- **Plugin install approval**: installing a plugin requesting sensitive
  scopes (e.g. wallet access) requires the installing Admin to explicitly
  confirm the requested permission list, matching
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).

## 20. Payment Gateways, Wallet, Accounting, Taxes, Invoices

ZodiCore inherits the base engine's wallet and gateway system as-is and
adds no extension to it — see
[wallet-system.md](../../standards/wallet-system.md) and
[payment-gateways.md](../../standards/payment-gateways.md) for the full
mechanics (append-only `Transaction` ledger with post-balance snapshots,
gateway-layer currency conversion, admin-configured withdrawal methods).
ZodiCore remains single-base-currency by default, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-currency-standard);
it has no regulatory requirement forcing a true multi-currency wallet, so
it does not extend the base engine the way ZodiBank or ZodiXchange do.
There is no platform-level billing of the buyer by Zodize — Zodize sells
the source code once, not a recurring subscription — see
[single-tenant-deployment-model.md §Licensing and update model](../../architecture/single-tenant-deployment-model.md#licensing-and-update-model).
The `plans` entity is the generic, buyer-configured plan pattern (e.g. a
membership tier or service package the buyer's own end users purchase),
never a Zodize-to-buyer billing relationship.

## 21. Documents, Exports, Imports

Withdrawal receipts and KYC-related documents are stored per
[data-protection-privacy.md](../../security/data-protection-privacy.md). An
Admin can export their own deployment's data (users, transactions, records)
for their own backup/portability purposes, per
[backup-disaster-recovery.md](../../security/backup-disaster-recovery.md).
Bulk record import via CSV with a mapping wizard is available for the
Generic Records module, per the
[import-wizard requirements](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Search, Filters, Global Search

ZodiCore's admin panel provides search/filter across its own modules
(users, transactions, tasks, records) per
[navigation-standards.md](../../standards/navigation-standards.md). This is
ZodiCore's own in-product search — not a cross-product or cross-deployment
search service, since no such thing exists in this architecture.

## 23. AI Features

- **Records field suggestion**: AI-assisted suggestion of a record type's
  field schema from a short description (e.g. "track vendor contracts"),
  always reviewed by the Admin before the record type is created.
- **Task triage**: AI-assisted priority/due-date suggestions for newly
  created tasks based on similar past tasks, never auto-applied without
  confirmation.

## 24. Automation, Scheduled Jobs, Cron Jobs, CLI Commands

- Scheduled jobs: withdrawal reminder sweep for pending approvals, KYC
  submission reminder, task due-date reminders, webhook delivery retries
  (where the plugin system is enabled).
- CLI commands (Artisan): `core:withdrawals:sweep`, `core:kyc:remind`,
  `core:tasks:remind`, `core:webhooks:retry` — each requires the same
  authorization context as its API equivalent, no CLI bypass of RBAC.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a realistic single-business deployment: a populated
`general_settings` row, a handful of staff accounts across different roles,
6 months of wallet transaction history (deposits, withdrawals, referral
commissions), a KYC form with sample approved/pending/rejected submissions,
3 demo projects with tasks in varying status, 2 demo record types with
sample records, and — where the plugin system is enabled — one installed
sample plugin — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the wallet transaction history view paginates
efficiently for an account with tens of thousands of transactions, and the
generic records list view remains responsive (p95 < 1s) for a record type
with tens of thousands of records, per
[performance-standards.md](../../quality/performance-standards.md).

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiCore carries no elevated security requirement beyond that
baseline — unlike the earlier "platform" framing, a security incident in
one buyer's ZodiCore deployment has no blast radius beyond that one
deployment, since there is no shared identity or billing service other
products depend on.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md) applies,
including the authorization-denial and not-found test cases required of
every product. ZodiCore has no cross-tenant isolation test category — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model)
for why that test category does not apply to any Zodize product.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md), same
as every other Zodize product — a single shared/VPS hosting target, one
database, one codebase directory. ZodiCore has no elevated deployment
requirement beyond that baseline.

## 30. Acceptance Criteria

- A buyer can go from uploading the codebase to a fully configured admin
  panel (branding, one payment gateway, one withdrawal method) with zero
  code edits.
- A deposit via a configured payment gateway correctly credits the user's
  wallet balance and produces a `Transaction` record.
- A staff member assigned a scoped role sees only the modules and actions
  their role grants.
- A task or generic record created by one staff member is visible to every
  other staff member with the appropriate permission, entirely within the
  one deployment.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).
ZodiCore follows the same checklist as every other product — there is no
elevated cross-product sign-off requirement, since no other product depends
on ZodiCore at runtime.

## 32. Future Roadmap

- Expand the generic Records module with relational linking between record
  types (e.g. linking a "Vendor" record to a "Contract" record).
- Task tracker recurring-task support.
- Deeper plugin manifest capabilities (scheduled-job hooks for plugins), if
  buyer demand for the marketplace justifies it.

## 33. Known Risks

- Feature overlap with vertical products: because ZodiCore's task/records
  modules are intentionally generic, a buyer with genuine vertical needs
  (e.g. real property or fleet management) may under-serve themselves by
  choosing ZodiCore instead of the matching vertical product — mitigated by
  clear positioning in [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md),
  not a product-level control.
- Plugin runtime security (where enabled): third-party plugins are a
  persistent attack surface within one buyer's deployment — mitigated by
  the marketplace review pipeline in
  [marketplace-architecture.md](../../architecture/marketplace-architecture.md).

## 34. Future Improvements

- Optional recurring/scheduled task templates for common operational
  checklists.
- Richer report builder templates specific to wallet/transaction analysis.

## Roadmap (spec depth)

This spec is reference-depth and considered complete. Companion documents
([DATA_MODEL.md](./DATA_MODEL.md), [API_REFERENCE.md](./API_REFERENCE.md))
will expand as implementation surfaces additional entities/endpoints;
changes there follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).
