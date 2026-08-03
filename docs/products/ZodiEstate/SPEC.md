# ZodiEstate — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth). See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for spec status
> definitions.

## 1. Vision

ZodiEstate is the operating system for property management companies and
real estate portfolio owners: one system of record for every property, unit,
lease, tenant, work order, and dollar of security deposit, so that a
portfolio manager running 50 units or 50,000 units can run rent collection,
maintenance, trust accounting, and owner reporting without stitching
together a spreadsheet, a screening vendor portal, and a separate accounting
package.

## 2. Purpose

Property managers are legally and financially exposed on two fronts every
day: fair housing compliance in how they list and screen applicants, and
trust accounting compliance in how they hold tenant security deposits and
owner funds. Generic ERP or spreadsheet-based operations cannot enforce
either — ZodiEstate exists to make trust-account segregation, audit-ready
fair housing screening records, and maintenance/turnover operations a single
enforced system instead of manual discipline.

## 3. Target Market

Residential and mixed-use property management companies managing 20 to
50,000+ units across single-family, multifamily, and small commercial
portfolios; owner-operators who self-manage a portfolio; and third-party
property management firms managing units on behalf of external owners who
require owner statements and disbursements.

## 4. Industries

Residential real estate management, commercial property management (retail
and small office), community association management (HOA/condo adjacent
use), and student/off-campus housing operators.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Property/lease management + tenant portal | AppFolio, Buildium | Built on the same audited, single-tenant base engine (RBAC, audit logging, wallet/ledger) as every other Zodize product, sold as source code the property manager owns and hosts, instead of a siloed hosted vertical stack |
| Trust accounting for deposits/owner funds | Buildium Trust Accounting, RealPage | Double-entry trust ledger enforced at the data-model level, not a bolt-on report |
| Maintenance/work order management | Property Meld, Rentvine | Unified with leasing and accounting rather than a separate integrated tool |
| Owner/investor reporting | RealPage, Yardi Voyager (enterprise tier) | Yardi-grade owner statements available to mid-market operators, not just enterprise accounts |
| Tenant screening compliance | TransUnion SmartMove, RentPrep | Fair housing-aware screening workflow with adverse-action documentation built into the applicant record |

## 6. Personas

- **Property Manager / Portfolio Manager** — oversees a set of properties,
  approves leases, reviews maintenance escalations, owns owner relationships.
- **Leasing Agent** — manages listings, applicant screening, and lease
  signing for assigned properties.
- **Maintenance Coordinator / Technician** — triages, assigns, and closes
  work orders; logs parts/labor.
- **Accounting/Trust Administrator** — reconciles trust accounts, runs owner
  disbursements, closes the books monthly.
- **Property Owner (external)** — views owner statements, disbursement
  history, and property performance via the owner portal.
- **Tenant** — pays rent, submits maintenance requests, views lease and
  ledger via the tenant portal.
- **Applicant** — applies to a listing, completes screening, e-signs a lease
  once approved.

## 7. User Journeys

1. **Listing to lease-signed**: Leasing Agent publishes a vacant unit listing
   with fair-housing-compliant language (screened against protected-class
   terms per §21) → applicant applies and pays application fee → screening
   report (credit, criminal, eviction history) returns via integrated
   screening vendor → agent applies the property's documented, consistently
   applied screening criteria → approval or adverse-action notice generated
   → lease drafted from a template, e-signed by applicant and manager →
   move-in inspection scheduled → unit status flips to Occupied.
2. **Rent collection with a late payment**: Tenant portal shows rent due →
   tenant pays via ACH or card → payment posts to the tenant ledger and, for
   security deposits, the trust ledger specifically → on day N past due (per
   the lease's configured grace period and jurisdiction rules), a late fee
   is assessed automatically and a notice is sent → if unresolved, the
   Property Manager can generate a pay-or-quit notice packet for the
   applicable jurisdiction.
3. **Maintenance work order lifecycle**: Tenant submits a request with
   photos via the portal → Maintenance Coordinator triages priority
   (emergency/urgent/routine) → assigns an in-house technician or dispatches
   a vetted vendor → technician logs status, parts, and labor time from a
   mobile-friendly view → tenant is notified on status changes → completion
   triggers a tenant satisfaction survey and, if vendor-billed, an
   owner-chargeable expense entry.
4. **Move-out and deposit disposition**: Tenant gives notice → move-out
   inspection scheduled and compared against move-in inspection/photos →
   itemized deductions (if any) are documented with cost justification →
   remaining deposit balance disbursed from the trust account within the
   jurisdiction's statutory deadline → disposition letter and ledger sent to
   the former tenant, and the unit enters the turnover/make-ready workflow.
5. **Monthly owner statement and disbursement**: Trust Administrator closes
   the accounting period → system reconciles trust and operating accounts →
   owner statement generated per property (income, expenses, management fee,
   net disbursement) → Owner reviews the statement and disbursement history
   in the owner portal → funds disbursed to the owner's bank account.

## 8. Business Goals

- Give mid-market property managers enterprise-grade (Yardi/RealPage-class)
  trust accounting and owner reporting without enterprise-grade
  implementation cost.
- Reduce fair housing exposure by making compliant screening criteria and
  adverse-action documentation the default path, not an optional add-on.
- Cut average work-order resolution time and increase tenant renewal rate
  through a responsive maintenance and communication loop.
- Make the owner portal a retention lever for management companies competing
  for owner accounts, not just a reporting afterthought.

## 9. Functional Requirements

- Property and unit inventory: property, building, unit, amenity, and unit
  type management across residential and small commercial types.
- Listing syndication-ready listing management with fair-housing-screened
  content.
- Applicant management: application intake, fee collection, integrated
  screening (credit/criminal/eviction), documented approval criteria,
  adverse-action notice generation.
- Lease lifecycle: templated lease generation, e-signature, renewals,
  amendments, month-to-month conversion, early termination.
- Tenant portal: rent payment (ACH/card), ledger view, maintenance requests,
  document access, lease renewal actions.
- Rent roll, recurring charges, late fee assessment rules (jurisdiction and
  lease-configurable), NSF handling.
- Trust accounting: segregated trust ledger for security deposits and
  owner-designated funds, separate from the operating account, with
  reconciliation workflows.
- Owner portal: property performance dashboards, statements, disbursement
  history, document library.
- Maintenance/work order management: request intake, triage, vendor and
  in-house technician assignment, parts/labor logging, vendor insurance
  tracking.
- Vacancy and turnover tracking: days-vacant metrics, make-ready checklists,
  move-in/move-out inspection with photo documentation.
- Owner statements and 1099 preparation support for owner and vendor
  payments.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiEstate-specific additions:

- Trust ledger writes are append-only and must never net-zero incorrectly —
  a trust/operating account mixing bug is a compliance-severity defect, not
  a standard bug-severity one.
- Tenant and owner portal pages (payment, statement view) p95 < 400ms given
  their high-frequency, non-technical user base.
- Rent roll and owner statement generation for a 1,000-unit portfolio
  completes in under 30 seconds.

## 11. Architecture

ZodiEstate is a standalone, self-hosted Laravel application, built by
cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
The clone strips the banking-specific tables that don't apply to property
management — `loans`/`loan_plans`, `dps`/`dps_plans`, `fdr`/`fdr_plans`,
`other_banks`, `beneficiaries`, `airtime_operators`/`airtime_configs` — and
keeps the `branches`/`branch_staff` guard, re-purposed rather than dropped:
a property management firm's field staff (leasing agents, maintenance
technicians assigned to specific properties) are modeled as
branch-scoped staff per
[product-genericization-checklist.md § Step 4](../../architecture/product-genericization-checklist.md#step-4--confirm-guard-configuration-matches-the-products-needs),
with "branch" mapped to "property" or "portfolio region" in ZodiEstate's own
admin navigation.

ZodiEstate inherits the base engine's admin settings/branding, double-entry
wallet ledger, 30+ payment gateway integrations, referral engine, KYC,
i18n, cron, extension toggles, and CMS/page builder as-is — see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps).
On top of that inherited engine, ZodiEstate adds its own domain modules
(Properties, Leasing, Trust Accounting, Maintenance, Owner Portal, Tenant
Relations) as new, clearly bounded Laravel modules per
[`module-template.md`](../../templates/module-template.md), each
registering its own permissions into the inherited `Role`/`Permission`
system (never a parallel RBAC) per
[`permission-template.md`](../../templates/permission-template.md).

The trust ledger (security deposits, owner-designated funds) is
purpose-built rather than reusing the inherited single wallet-balance
engine directly: it is implemented as a dedicated `TrustLedgerContract`
that enforces double-entry, append-only posting fully segregated from the
operating ledger at the data-model level — see §27. This is a deliberate
domain-specific extension on top of the inherited wallet/ledger pattern
documented in
[wallet-system.md](../../standards/wallet-system.md#what-a-products-domain-modules-do-with-this-engine),
not a reimplementation of it: ordinary operating-account transactions
(management fees, vendor payments) still post through the inherited
`Transaction` model, while client/owner trust funds post exclusively
through `TrustLedgerContract` so the two can never commingle in code.

Each ZodiEstate deployment belongs to exactly one property management
company or owner-operator, running on their own hosting with their own
database — there is no shared platform, no `tenant_id` scoping, and no
runtime dependency on any other Zodize product, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
A firm managing units on behalf of multiple external owners models that
via the `owners` entity (§14), not via multi-tenancy; a firm structured as
multiple legal entities (e.g. separate LLCs per property) uses the
`company_id` multi-company scoping pattern from
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
if its own operations require it.

## 12. Technology

Laravel 11 + PHP ^8.3 per the inherited base codebase
([coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)),
with Blade + Bootstrap 5 + jQuery for new module UI per
[coding-standards-laravel-frontend.md](../../development/coding-standards-laravel-frontend.md);
MySQL/MariaDB per the base codebase's inherited schema, matching
[database-standards.md](../../development/database-standards.md); ACH/card
processing via the inherited payment gateway abstraction (see
[payment-gateways.md](../../standards/payment-gateways.md)); e-signature and
tenant/applicant screening are third-party integrations (§22).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Properties | Property/Building/Unit Inventory, Amenities, Unit Types |
| Leasing | Listings, Applications, Screening, Lease Templates, E-Signature, Renewals |
| Rent & Billing | Rent Roll, Recurring Charges, Late Fees, NSF Handling, Tenant Payments |
| Trust Accounting | Security Deposit Ledger, Owner Fund Ledger, Reconciliation, Disbursements |
| Maintenance | Work Orders, Vendor Management, Technician Dispatch, Preventive Maintenance |
| Turnover | Move-In/Move-Out Inspections, Make-Ready Checklists, Vacancy Tracking |
| Owner Relations | Owner Portal, Owner Statements, 1099 Support |
| Tenant Relations | Tenant Portal, Communications, Renewal Offers |

## 14. Core Data Model

Full ER diagram queued (§ Roadmap). Core entities:

Every entity below belongs to the one property management company that owns
this deployment — there is no `tenant_id` column anywhere in this schema,
per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`company_id` appears only where a firm operating multiple legal entities
needs it, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).

| Entity | Key columns |
|---|---|
| `properties` | id, company_id (nullable — multi-entity firms only), name, address, property_type, owner_id |
| `units` | id, property_id, unit_number, bedrooms, bathrooms, sqft, status (vacant/occupied/make_ready) |
| `owners` | id, name, payout_bank_account_id, management_fee_pct |
| `applicants` | id, listing_id, applicant_user_id, screening_status, decision, adverse_action_sent_at |
| `leases` | id, unit_id, tenant_user_id, start_date, end_date, rent_amount, deposit_amount, status |
| `lease_charges` | id, lease_id, charge_type, amount, due_date, recurrence_rule |
| `tenant_ledger_entries` | id, lease_id, entry_type (charge/payment/fee/credit), amount, posted_at |
| `trust_ledger_entries` | id, property_id, lease_id, entry_type, amount, account (deposit/owner_funds), posted_at |
| `disbursements` | id, owner_id or lease_id (former tenant), amount, method, disbursed_at, statutory_deadline |
| `work_orders` | id, unit_id, priority, category, status, assigned_to_type (staff/vendor), created_by |
| `vendors` | id, name, insurance_expires_at, trade_categories |
| `inspections` | id, unit_id, lease_id, inspection_type (move_in/move_out/routine), photos, condition_notes |
| `owner_statements` | id, owner_id, property_id, period_start, period_end, net_disbursed, pdf_document_id |

Here `tenant_user_id` and `tenant_ledger_entries` use "tenant" in its
real-estate sense (the renter occupying a unit) — unrelated to the
deprecated SaaS-tenancy model above; this is standard domain vocabulary for
a property management product and is retained as-is.

## 15. Key API Endpoints

Full endpoint catalog queued (§ Roadmap). Key routes, all conforming to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md):

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/properties` | List properties in this deployment |
| POST | `/api/v1/properties/{property}/units` | Create a unit |
| GET | `/api/v1/listings` | Public/portal listing search |
| POST | `/api/v1/listings/{listing}/applications` | Submit rental application |
| POST | `/api/v1/applications/{application}/screening` | Trigger screening report |
| POST | `/api/v1/applications/{application}/decision` | Record approve/deny + adverse action |
| POST | `/api/v1/leases` | Create lease from template |
| POST | `/api/v1/leases/{lease}/sign` | Record e-signature event |
| GET | `/api/v1/leases/{lease}/ledger` | Tenant ledger for a lease |
| POST | `/api/v1/leases/{lease}/payments` | Record/process a rent payment |
| POST | `/api/v1/leases/{lease}/late-fees/assess` | Manually trigger late fee assessment |
| POST | `/api/v1/work-orders` | Create a maintenance work order |
| PATCH | `/api/v1/work-orders/{workOrder}/assign` | Assign to staff or vendor |
| PATCH | `/api/v1/work-orders/{workOrder}/status` | Update work order status |
| POST | `/api/v1/inspections` | Create move-in/move-out inspection |
| GET | `/api/v1/trust-ledger/reconciliation` | Trust account reconciliation report |
| POST | `/api/v1/disbursements` | Create an owner or deposit disbursement |
| GET | `/api/v1/owners/{owner}/statements` | Owner statement history |
| GET | `/api/v1/portal/tenant/dashboard` | Tenant portal dashboard data |
| GET | `/api/v1/portal/owner/dashboard` | Owner portal dashboard data |
| POST | `/api/v1/vendors` | Create vendor record |
| GET | `/api/v1/reports/vacancy` | Vacancy/turnover report |

## 16. Events

`unit.listed`, `application.submitted`, `application.screened`,
`application.decided`, `lease.signed`, `lease.renewed`,
`lease.terminated`, `payment.received`, `payment.failed`,
`late_fee.assessed`, `work_order.created`, `work_order.assigned`,
`work_order.completed`, `inspection.completed`, `deposit.disbursed`,
`owner_statement.generated`, `disbursement.completed`. See
[caching-queues-events.md](../../architecture/caching-queues-events.md) for
the event/queue architecture these are built on.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `application.decided` (denial) | — | ✔ (adverse action notice) | — | — |
| `lease.signed` | ✔ | ✔ | — | — |
| `payment.received` | ✔ | ✔ (receipt) | — | — |
| Rent due reminder (T-3 days) | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `late_fee.assessed` | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `work_order.created` (to coordinator) | ✔ | ✔ | — | ✔ |
| `work_order.status_changed` (to tenant) | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `owner_statement.generated` | ✔ | ✔ | — | — |
| Lease renewal offer (T-60 days) | ✔ | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine and `admin` guard per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie),
plus a re-purposed `branch_staff` guard (§11) for property-scoped field
staff. ZodiEstate's own roles: `Property Manager`, `Leasing Agent`,
`Maintenance Coordinator`, `Trust Administrator`, `Owner` (portal-only,
scoped to their own properties), `Tenant` (portal-only, scoped to their own
lease). Key permissions: `properties.manage`, `leases.approve`,
`trust_ledger.post` (restricted to Trust Administrator and Owner-tier roles
— never Leasing Agent), `disbursements.execute`, `work_orders.assign`,
`screening.run`, all registered into the inherited permission system per
[permission-template.md](../../templates/permission-template.md). Trust
ledger posting permissions are enforced with a stricter approval
requirement than standard CRUD per §19.

## 19. Workflows & Approval Chains

- **Lease approval**: applications exceeding a configured risk threshold (or
  denied against stated criteria) require Property Manager sign-off before
  an adverse-action notice is sent, ensuring criteria were applied
  consistently.
- **Trust disbursement approval**: any disbursement from the trust ledger
  (deposit return, owner payout) requires a second approver distinct from
  the initiator when the amount exceeds an admin-configured threshold —
  a maker-checker control matching standard trust accounting practice.
- **Large expense/work order approval**: vendor-billed work orders above a
  configured dollar threshold require Property Manager approval before the
  vendor is dispatched.
- **Move-out deposit deduction dispute**: a tenant-disputed deduction routes
  to the Property Manager for review before disbursement finalizes.

## 20. Audit Logs

Every trust ledger entry, disbursement, lease decision, and screening
decision is immutably audit-logged per
[audit-logging.md](../../security/audit-logging.md), including the actor,
before/after state, and — for adverse actions — the specific criteria cited,
supporting fair housing compliance review.

## 21. Reports & Analytics & Dashboards

- Rent roll, delinquency aging, and collections reports.
- Vacancy rate, days-on-market, and turnover-cost-per-unit dashboards.
- Trust account reconciliation and owner statement reports.
- Maintenance SLA/resolution-time and vendor performance dashboards.
- Fair housing screening consistency report (approval/denial rate by
  criteria applied, supporting self-audit).
- Report builder and scheduled reports per the
  [Second-Layer Feature Catalog](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Tenant screening**: credit, criminal, and eviction history providers
  (TransUnion SmartMove-class, integrated via a vendor abstraction so
  tenants can substitute providers).
- **Payments**: ACH/card processing via the inherited base codebase's
  payment gateway abstraction (§12); integrated with tenant ledger and trust
  ledger posting.
- **E-signature**: DocuSign/Adobe Sign-class provider for lease execution.
- **Listing syndication**: outbound feed to listing aggregators.
- **Accounting export**: QuickBooks/Xero-compatible export for firms that
  maintain a separate corporate books system alongside ZodiEstate's trust
  ledger.
- **Insurance verification**: vendor certificate-of-insurance tracking
  integration.

## 23. AI Features

- AI-assisted maintenance triage: suggests priority and trade category from
  the tenant's free-text request and photos, with the Maintenance
  Coordinator confirming before dispatch.
- AI-drafted listing descriptions with an automated fair-housing language
  screen that flags and blocks protected-class-referencing terms before
  publish.
- Anomaly detection on trust ledger activity (unusual disbursement patterns)
  surfaced to the Trust Administrator, built on ZodiEstate's own audit log
  data per [audit-logging.md](../../security/audit-logging.md).

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: recurring rent charge generation, late fee assessment,
  rent due reminders, lease renewal offer generation, statutory deposit
  disbursement deadline alerts, monthly owner statement generation.
- CLI commands: `estate:generate-rent-roll`, `estate:assess-late-fees`,
  `estate:close-accounting-period`, `estate:reconcile-trust-account` — each
  requiring the same authorization context as its API equivalent.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a demo portfolio of 3 properties (40 units total)
spanning occupied, vacant, and make-ready states; 12 months of rent roll and
trust ledger history including one late payment and one deposit disposition;
a populated maintenance work order history across all statuses; and two
demo owner accounts with generated owner statements — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10. Additionally: applicant screening report retrieval completes
within the vendor's SLA with a user-visible pending state, never a blocking
synchronous wait beyond 5 seconds.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiEstate-specific requirements:

- **Fair housing compliance**: listing content and screening criteria are
  validated against protected-class references before publish; adverse
  action notices are generated and retained per fair housing recordkeeping
  requirements; screening criteria must be documented once per property/
  portfolio and applied consistently — the system prevents ad hoc,
  per-applicant criteria changes without a logged justification.
- **Trust accounting segregation**: trust funds (security deposits,
  owner-designated funds) are modeled and reconciled in a ledger fully
  segregated from the operating/revenue ledger at the data-model level, not
  merely by report filter — mirroring statutory trust accounting
  requirements that prohibit commingling. Reconciliation discrepancies
  block period close and alert the Trust Administrator.
- **PII handling**: applicant screening data (credit, criminal history) is
  field-level encrypted at rest and access-logged, per
  [data-protection-privacy.md](../../security/data-protection-privacy.md),
  with retention limited to the jurisdiction's required window.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md);
additionally a dedicated trust ledger integrity test suite verifying every
posting sequence nets to zero within its account grouping and that no code
path can post a trust entry to the operating ledger or vice versa.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).

## 30. Acceptance Criteria

- An applicant can apply, be screened, and receive a decision — with an
  adverse-action notice generated automatically on denial — end to end.
- A tenant can pay rent, see the payment reflected in their ledger, and
  trigger no false late fee when paid within the grace period.
- A security deposit can be collected, held in the trust ledger separate
  from operating funds, and disbursed with an itemized disposition on
  move-out.
- An owner statement reconciles exactly against the trust and operating
  ledgers for the period, with no unexplained variance.
- A maintenance request submitted by a tenant reaches an assigned technician
  or vendor and the tenant is notified at each status change.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).

## 32. Future Roadmap

- HOA/community association-specific assessment and violation tracking
  module.
- Automated rent benchmarking against local market comparables.
- Renters insurance verification and requirement enforcement integration.

## 33. Known Risks

- Trust accounting defects carry regulatory and licensure risk for
  customers (property managers can lose their license for commingling) —
  mitigated by the dedicated integrity test suite in §28, but this remains
  the module requiring the highest engineering scrutiny in the product.
- Fair housing screening criteria vary by jurisdiction; a criteria template
  that is compliant in one jurisdiction may not be in another — mitigated
  by jurisdiction-aware criteria templates, tracked for expansion.

## 34. Future Improvements

- Configurable per-jurisdiction late fee and notice rule packs beyond the
  initial set.
- Predictive turnover/vacancy risk scoring per unit.

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture (§11), Core Data Model
(§14), and Permissions & Roles (§18) sections were revised to the
standalone, self-hosted, single-tenant model in
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md);
no product-domain content (vision, personas, journeys, workflows) changed.
Queued for Deep-depth expansion: full ER diagram and migration set
(companion `DATA_MODEL.md`), full endpoint catalog (companion
`API_REFERENCE.md`), complete jurisdiction-specific late-fee and notice rule
library, and a full report catalog beyond the summary list in §21.
