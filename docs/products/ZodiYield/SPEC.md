# ZodiYield — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

Built on [ZodiCore](../ZodiCore/SPEC.md) — ZodiYield does not reimplement
identity, tenancy, billing, notifications, RBAC, plugins, or audit logging;
it consumes those services and adds lending-domain modules on top.

## 1. Vision

ZodiYield is lending, credit, and yield product management for fintech
lenders, credit unions, and embedded-lending programs that need real
origination, real underwriting integration, and real amortization — not a
loan-tracking spreadsheet with a UI. It gives a lender the origination,
servicing, collections, and disclosure infrastructure to run a compliant
consumer or commercial lending product from application to payoff.

## 2. Purpose

Launching a lending product today means integrating a loan origination
system, a credit bureau/underwriting decision engine, a servicing platform
for amortization and payment processing, and a collections workflow
separately, while staying compliant with disclosure requirements throughout.
ZodiYield exists to give a lender one coherent system — origination,
underwriting integration, servicing, collections, and regulatory
disclosures — built on ZodiCore's identity and audit backbone, so
engineering effort goes into credit policy and product design, not
plumbing.

## 3. Target Market

Fintech consumer and small-business lenders, credit unions modernizing
their lending stack, embedded-lending programs (e.g. point-of-sale
financing partners), and yield/deposit-product providers offering
interest-bearing accounts alongside lending. Buyers are typically a Head
of Lending Operations, Chief Credit Officer, or fintech CTO evaluating a
build-vs-buy decision against a legacy loan servicing platform.

## 4. Industries

Fintech lending, consumer credit, small-business lending, credit unions.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Loan origination system | nCino, Blend, LendingPad | Ships with tenant/RBAC/audit already built via ZodiCore, faster to stand up a compliant LOS |
| Loan servicing/amortization | Shaw Systems, LoanPro, Nortridge | Servicing schedule and payment ledger share one audit trail with origination, not a handoff between systems |
| Underwriting/decisioning | Zest AI, Upstart's underwriting layer, Provenir | Underwriting integration is provider-abstracted so a lender can swap or A/B test decision engines |
| Collections/delinquency management | Katabat, TrueAccord, CollectMax | Collections workflow operates directly on the same loan ledger, not a reconciled downstream export |
| Regulatory disclosure generation | MeridianLink compliance modules, Ncontracts | APR/TILA-style disclosures generated from the actual loan terms at origination, not a separately maintained template |

## 6. Personas

- **Head of Lending Operations** — oversees origination pipeline,
  underwriting exceptions, and servicing operations.
- **Chief Credit Officer/Underwriter** — sets credit policy and reviews
  underwriting decisions and exceptions.
- **Collections Manager/Agent** — manages the delinquency workflow and
  works collections queues.
- **Borrower** — applies for a loan, reviews disclosures, and manages
  repayment via a self-service portal.
- **Compliance Officer** — reviews disclosure accuracy and regulatory
  filing obligations.
- **Zodize Support/Ops** — as defined in [ZodiCore §6](../ZodiCore/SPEC.md#6-personas).

## 7. User Journeys

1. **Loan application and underwriting**: borrower applies for a loan
   product → application data and consent for a credit pull are captured →
   underwriting decision engine integration returns a credit score and
   decision recommendation (see §21) → application resolves to
   auto-approve, manual underwriting review, or decline → on approval, loan
   terms and required disclosures (APR, finance charge, payment schedule)
   are generated for borrower review.
2. **Disclosure acknowledgment and disbursement**: borrower reviews and
   electronically acknowledges TILA-style disclosures → loan agreement is
   executed → disbursement is initiated to the borrower's designated
   account → the amortization schedule is generated and the loan enters
   `active` servicing status.
3. **Scheduled payment and amortization**: a scheduled payment date
   arrives → the system attempts to collect the payment via the borrower's
   configured payment method → on success, principal/interest is
   allocated per the amortization schedule and the loan balance updates;
   on failure, the payment enters a retry/grace-period flow feeding the
   delinquency workflow.
4. **Delinquency and collections**: a payment remains unresolved past the
   grace period → the loan transitions through configurable delinquency
   buckets (e.g. 30/60/90 days past due) → a collections case opens and
   routes to a Collections Agent → agent works the case (contact borrower,
   negotiate a payment plan, or escalate) → resolution (cured, payment
   plan, charge-off) is recorded against the loan.
5. **Loan payoff and closure**: borrower pays the loan in full (scheduled
   final payment or early payoff) → the system calculates the payoff
   amount including any accrued interest → on receipt, the loan transitions
   to `paid_off` status → a payoff confirmation and, where applicable, a
   final disclosure/tax document is generated for the borrower.

## 8. Business Goals

- Let a lender launch a compliant lending product without building
  origination, servicing, and collections systems from scratch.
- Keep every loan's terms, payment history, and disclosure record
  traceable end-to-end for regulatory examination and borrower dispute
  resolution.
- Reduce delinquency loss through a configurable, auditable collections
  workflow rather than ad hoc agent processes.

## 9. Functional Requirements

- Loan origination: multi-product application intake (installment,
  revolving/line-of-credit, yield-bearing deposit products), configurable
  application forms per product.
- Underwriting/credit scoring integration: provider-abstracted credit
  pull and decision-engine integration, with configurable auto-approve/
  manual-review/decline thresholds.
- Amortization schedules: standard amortizing schedules plus configurable
  interest-only, balloon, and variable-rate schedule types.
- Disbursement: single or staged disbursement to a borrower's designated
  account, with disbursement status tracking.
- Collections/delinquency workflows: configurable delinquency bucket
  definitions, case queueing, agent assignment, and resolution tracking.
- Interest rate products: fixed-rate, variable-rate (indexed to a
  configurable benchmark), and yield-bearing product configuration.
- Regulatory disclosures: APR/finance-charge calculation and TILA-style
  disclosure generation at origination, with disclosure acknowledgment
  tracking.
- Payment processing: scheduled payment collection, retry logic, payment
  method management, and manual payment recording.
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (underwriting exception approval, charge-off approval),
  rule engine (delinquency bucket transitions, collections escalation),
  saved filters and global search over loans/borrowers, custom fields on
  loan/borrower records, full audit history, soft delete with restore on
  non-ledger entities, mass actions (bulk payment retry), import/export
  wizards for loan portfolio data, command palette, scheduled/report-
  builder reporting, system health dashboard, white-labeling of the
  borrower portal.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiYield-specific additions:

- Underwriting decision round-trip (application submit to decision
  returned, excluding third-party decision-engine latency) targets p95 <
  2s.
- Amortization schedule generation for a new loan targets p95 < 1s
  regardless of term length.
- 99.9% uptime target for the borrower portal and payment processing
  services, matching the general product baseline.

## 11. Architecture

ZodiYield is a Laravel application consuming ZodiCore's identity, tenancy,
billing, notification, RBAC, plugin, and audit packages exactly as
described in [ZodiCore §11](../ZodiCore/SPEC.md#11-architecture). It adds a
`zodize/lending-servicing` domain package providing the loan ledger,
amortization engine, and payment-allocation logic, plus a `zodize/
collections` package for the delinquency workflow, both registering into
ZodiCore's audit log and RBAC policy registry. Underwriting/credit-bureau
integration runs behind an `UnderwritingProviderContract` so a lender can
swap or A/B test decision engines without touching loan ledger logic, and
payment processing runs as a queued worker (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
isolated from the request path for retry resilience.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL for loan/payment/amortization records + Redis for delinquency-
bucket and collections-queue caching per
[database-standards.md](../../development/database-standards.md);
underwriting, credit bureau, and payment-processing integrations are
provider-abstracted (see §22).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Origination | Application Intake, Product Configuration, Disclosure Generation |
| Underwriting | Credit Pull Integration, Decision Engine Integration, Manual Review Queue |
| Servicing | Amortization Engine, Disbursement, Payment Processing, Balance Ledger |
| Collections | Delinquency Buckets, Case Queue, Agent Assignment, Resolution Tracking |
| Interest Rate Products | Fixed-Rate, Variable-Rate/Indexed, Yield-Bearing Configuration |
| Compliance/Disclosures | APR/Finance Charge Calculation, TILA-Style Disclosure, Acknowledgment Tracking |
| Borrower Portal | Statement Access, Payment Method Management, Payoff Requests |

## 14. Core Data Model

The 11 entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)).

| Entity | Key columns |
|---|---|
| `loan_products` | id, tenant_id, name, rate_type (fixed/variable/yield), term_options, apr_calc_method |
| `loan_applications` | id, tenant_id, borrower_id, product_id, requested_amount, status, submitted_at |
| `underwriting_decisions` | id, application_id, provider, credit_score, decision, decision_reason, decided_at |
| `loans` | id, application_id, principal_amount, interest_rate, term_months, status, originated_at |
| `amortization_schedules` | id, loan_id, installment_number, due_date, principal_due, interest_due, balance_after |
| `payments` | id, loan_id, amount, payment_method_id, status, applied_principal, applied_interest, processed_at |
| `disclosures` | id, loan_id, disclosure_type, apr, finance_charge, acknowledged_at |
| `delinquency_records` | id, loan_id, bucket, days_past_due, entered_at, cured_at |
| `collections_cases` | id, delinquency_record_id, assigned_agent_id, status, resolution, resolved_at |
| `disbursements` | id, loan_id, amount, destination_account_ref, status, disbursed_at |
| `borrowers` | id, tenant_id, legal_name, identity_verification_status, created_at |

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/loan-applications` | Submit a loan application |
| POST | `/api/v1/loan-applications/{id}/underwrite` | Trigger underwriting decision |
| POST | `/api/v1/loan-applications/{id}/manual-review` | Route an application to manual review |
| POST | `/api/v1/loan-applications/{id}/approve` | Approve an application and originate the loan |
| GET | `/api/v1/loans/{id}` | Fetch loan detail including current balance |
| GET | `/api/v1/loans/{id}/amortization-schedule` | Fetch the amortization schedule |
| POST | `/api/v1/loans/{id}/disbursements` | Initiate disbursement |
| POST | `/api/v1/loans/{id}/payments` | Record/process a payment |
| GET | `/api/v1/loans/{id}/payments` | List payment history |
| POST | `/api/v1/loans/{id}/payoff-quote` | Generate a payoff quote |
| POST | `/api/v1/loans/{id}/payoff` | Process full payoff |
| GET | `/api/v1/loans/{id}/disclosures` | List disclosures for a loan |
| POST | `/api/v1/disclosures/{id}/acknowledge` | Acknowledge a disclosure |
| GET | `/api/v1/collections/cases` | List collections case queue |
| POST | `/api/v1/collections/cases/{id}/assign` | Assign a collections case to an agent |
| POST | `/api/v1/collections/cases/{id}/resolve` | Resolve a collections case |
| GET | `/api/v1/loans/{id}/delinquency-history` | Fetch delinquency bucket history |
| GET | `/api/v1/reports/portfolio-performance` | Generate portfolio performance report |
| GET | `/api/v1/portal/loans` | Borrower-scoped: fetch own loans |
| POST | `/api/v1/portal/payment-methods` | Borrower-scoped: add a payment method |

## 16. Events

Domain events registered on ZodiCore's event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`application.submitted`, `application.underwritten`,
`application.approved`, `application.declined`, `loan.originated`,
`loan.disbursed`, `disclosure.generated`, `disclosure.acknowledged`,
`payment.scheduled`, `payment.succeeded`, `payment.failed`,
`delinquency.bucket_entered`, `delinquency.cured`,
`collections_case.opened`, `collections_case.resolved`, `loan.charged_off`,
`loan.paid_off`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `application.approved` | ✔ | ✔ | ✔ | ✔ |
| `application.declined` | ✔ | ✔ (adverse action notice) | — | — |
| `disclosure.generated` | ✔ | ✔ | — | — |
| `payment.failed` | ✔ | ✔ | ✔ | ✔ |
| `delinquency.bucket_entered` | ✔ (to borrower and collections queue) | ✔ | ✔ | — |
| `loan.paid_off` | ✔ | ✔ (payoff confirmation) | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Extends [ZodiCore's default roles](../../security/rbac-permissions.md#default-system-roles)
with lending-specific roles: `Head of Lending Operations`,
`Chief Credit Officer/Underwriter`, `Collections Manager`,
`Collections Agent`, `Compliance Officer`, `Borrower` (portal-scoped
external role). Key permissions: `applications.underwrite`,
`applications.approve`, `applications.decline`, `loans.disburse`,
`loans.charge_off` (restricted to Chief Credit Officer/Compliance),
`collections.assign_case`, `collections.resolve_case`,
`disclosures.regenerate`. Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **Underwriting exception approval**: an application that falls outside
  automated decision-engine thresholds routes to manual review; approval
  by an Underwriter is required before origination, and the underwriter's
  rationale is captured and audit-logged.
- **Charge-off approval**: writing off a delinquent loan as uncollectible
  requires Chief Credit Officer or Compliance Officer approval and is
  never triggered automatically by the collections workflow alone.
- **Disclosure regeneration**: any change to loan terms before
  disbursement requires disclosures to regenerate and be re-acknowledged
  by the borrower before disbursement can proceed.
- **Payment plan approval**: a collections agent proposing a modified
  payment plan for a delinquent borrower requires Collections Manager
  approval before the plan supersedes the original amortization schedule.

## 20. Audit Logs

Every underwriting decision, loan status transition, payment application,
disclosure generation/acknowledgment, delinquency bucket transition, and
collections case resolution writes an immutable audit entry via ZodiCore's
audit log ([audit-logging.md](../../security/audit-logging.md)), capturing
actor (including the underwriting provider as a system actor where
applicable), timestamp, and before/after state. Amortization schedules and
payment allocations are never edited in place — corrections occur via a
new schedule version or offsetting payment entry referencing the original.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: origination volume, approval rate, portfolio
  balance, delinquency rate by bucket, collections case backlog — per
  [dashboard-standards.md](../../standards/dashboard-standards.md).
- Portfolio performance: vintage analysis, charge-off rate, roll-rate
  (bucket-to-bucket transition) reporting.
- Compliance: disclosure acknowledgment audit, adverse-action notice log,
  APR accuracy verification report.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Credit bureaus**: consumer/commercial credit pull integration (e.g.
  Experian/Equifax/TransUnion-class bureaus) behind a
  `CreditBureauContract`.
- **Underwriting/decisioning engines**: automated decision-engine
  integration (e.g. Zest AI/Provenir-class vendors) behind an
  `UnderwritingProviderContract`, allowing a lender to swap or A/B test
  engines.
- **Payment processing/ACH rails**: shares the payment-rail integration
  category described in
  [ZodiBank §22](../ZodiBank/SPEC.md#22-integrations) for disbursement and
  scheduled payment collection.
- **KYC/AML and identity verification**: same identity-verification
  integration category as [ZodiBank §22](../ZodiBank/SPEC.md#22-integrations)
  for borrower onboarding.
- **Collections/skip-tracing vendors**: optional integration for
  charged-off account placement with third-party collection agencies.

## 23. AI Features

- Underwriting decision explanation: for manual-review applications,
  summarizes in plain language which application/credit-bureau signals
  drove the automated recommendation, to speed underwriter review.
- Collections prioritization assistant: ranks open collections cases by
  estimated recovery likelihood and suggested next action, always leaving
  final contact/negotiation decisions to the Collections Agent.
- Portfolio risk anomaly detection, layered on top of ZodiCore's audit-log
  anomaly detection ([ZodiCore §23](../ZodiCore/SPEC.md#23-ai-features)),
  tuned for unusual delinquency-rate shifts within a product or vintage
  cohort.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: daily scheduled-payment collection run, payment retry
  processing, delinquency-bucket re-evaluation, disclosure-acknowledgment
  reminder, variable-rate index recalculation.
- CLI commands (Artisan): `yield:run-scheduled-payments`,
  `yield:retry-failed-payments`, `yield:reevaluate-delinquency`,
  `yield:recalculate-variable-rates` — each requires the same
  authorization context as its API equivalent, per
  [ZodiCore §24](../ZodiCore/SPEC.md#24-automation-scheduled-jobs-cron-jobs-cli-commands).

## 25. Seed/Demo Data

`YieldDemoSeeder` provisions a demo tenant with two or three loan products
(fixed installment, variable-rate line of credit, yield-bearing deposit),
200+ synthetic borrowers and loans spanning current, delinquent, charged-
off, and paid-off states, 12 months of payment and amortization history,
and a populated collections case queue with resolved and open cases — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the nightly scheduled-payment collection run
processes at least 50,000 due payments within a 2-hour batch window without
manual intervention.

## 27. Security Requirements

Financial products carry Zodize's highest security/compliance bar. Full
baseline from [security-standards.md](../../security/security-standards.md)
applies, plus:

- **PCI-DSS-equivalent handling** for all stored payment method data
  (bank account/card-on-file for scheduled payments), tokenized at
  ingestion and never stored or logged in plaintext.
- **SOC2-equivalent controls**: change management and access review apply
  to origination, underwriting, and servicing modules with the same rigor
  as the ZodiCore platform baseline.
- **KYC/AML** required before an application can be underwritten, shared
  with the identity-verification category in
  [ZodiBank §22](../ZodiBank/SPEC.md#22-integrations).
- **Immutable audit trails**: underwriting, disbursement, payment, and
  collections audit entries are append-only, matching §20.
- **MFA is mandatory, not optional**, for every internal user role
  (Lending Operations, Underwriter, Collections, Compliance), and enforced
  for borrower portal access to payment method management specifically,
  per [authentication-authorization.md](../../security/authentication-authorization.md).
- **Regulatory disclosure accuracy**: APR/finance-charge calculations are
  covered by a dedicated regression suite (§28) because a calculation
  error here is a compliance violation, not merely a display bug.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated amortization/APR calculation regression suite validated against
known reference schedules across fixed, variable, interest-only, and
balloon products, and a delinquency-bucket transition test suite covering
every configured bucket boundary.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Amortization/APR calculation and disclosure-generation logic changes
require a documented compliance review in addition to the standard PR
review per [pr-standards.md](../../development/pr-standards.md), given the
direct regulatory exposure of a calculation error.

## 30. Acceptance Criteria

- A borrower can apply, receive an underwriting decision, review and
  acknowledge disclosures, and receive disbursement entirely through the
  API for the auto-approve path.
- Amortization and APR calculations match reference values for a defined
  regression fixture set across every supported rate type.
- A missed payment correctly transitions the loan through configured
  delinquency buckets and opens a collections case without manual
  intervention.
- A loan can be paid off in full, with the payoff amount matching the
  reference calculation and the loan transitioning to `paid_off` status.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiYield additionally requires sign-off from a compliance stakeholder that
disclosure generation, adverse-action notices, and underwriting fairness
have been validated against the lender's actual regulatory jurisdiction
before go-live.

## 32. Future Roadmap

- Secured lending support (collateral tracking, lien perfection status).
- Loan participation/syndication support for lenders selling portions of
  originated loans.
- Expanded variable-rate product support with additional benchmark index
  options.

## 33. Known Risks

- Underwriting provider dependency: decision quality and latency depend on
  the integrated credit bureau/decision-engine vendor — mitigated by the
  `UnderwritingProviderContract` abstraction, but remains an external
  dependency outside Zodize's direct control.
- Regulatory divergence: disclosure requirements (TILA-style APR
  disclosure) vary by jurisdiction and loan type; the current
  implementation targets US consumer-lending-style disclosure and will
  need per-jurisdiction variants as the product expands internationally.

## 34. Future Improvements

- Configurable regulatory disclosure templates per jurisdiction beyond the
  current US-style baseline.
- Machine-learning-assisted collections next-best-action recommendations
  beyond the current prioritization assistant.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram and migration set for the loan/servicing/collections schema
(companion `DATA_MODEL.md`), a complete endpoint catalog including the full
borrower portal surface (companion `API_REFERENCE.md`) matching
[ZodiCore's structure](../ZodiCore/SPEC.md), and a full regulatory
disclosure catalog covering jurisdictions beyond the US TILA-style baseline.
Changes follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).
