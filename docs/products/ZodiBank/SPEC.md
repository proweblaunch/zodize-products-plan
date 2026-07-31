# ZodiBank — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

ZodiBank is a standalone, self-hosted Laravel application built by cloning
the sanitized [base codebase](../../architecture/base-codebase-strategy.md)
and layering banking-domain modules on top. Unlike every other product in
the catalog, ZodiBank does **not** run the
[genericization checklist's](../../architecture/product-genericization-checklist.md)
loan/DPS/FDR removal step — it keeps and extends the base engine's inherited
loan, DPS, and FDR tables as its own core banking modules, rather than
stripping them. It does not depend on any other Zodize product or on a
central "ZodiCore" platform for identity, billing, notifications, or
tenancy — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`ZodiCore` is itself just another standalone product in the catalog (a
general-purpose back-office/ERP starter), not a platform ZodiBank runs on.

## 1. Vision

ZodiBank is the core banking platform for digital-first banks, credit
unions, and embedded-finance programs that need a real ledger, real card
issuing, and real regulatory posture — not a fintech prototype. It gives a
chartered institution or a sponsor-bank program the deposit account,
transaction, and compliance infrastructure to launch a checking/savings
product in months, on a platform that is audited the way core banking
software is actually audited.

## 2. Purpose

Standing up core banking today means choosing between decades-old cores
(Fiserv, FIS, Jack Henry) that are slow to integrate, or piecing together a
ledger vendor, a card issuer processor, a KYC vendor, and a payments rail
integration by hand. ZodiBank exists to give a digital bank or credit union
one coherent, self-hosted system — general ledger, deposit accounts, card
issuing, ACH/wire movement, and regulatory reporting — built on a base
codebase whose wallet/ledger, RBAC, KYC, and audit engine already work,
instead of building its own from scratch.

## 3. Target Market

Digital-first banks and neobanks, credit unions modernizing off a legacy
core, embedded-finance programs (fintechs partnering with a sponsor bank),
and regional community banks running a digital-only deposit product
alongside their legacy core. Buyers are typically a Head of Banking
Operations, Chief Compliance Officer, or CTO evaluating a build-vs-buy
decision against a legacy core conversion.

## 4. Industries

Banking, credit unions, embedded finance / banking-as-a-service (BaaS)
programs.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Core banking ledger | Mambu, Thought Machine (Vault), Temenos | Ships with RBAC/audit/wallet-ledger already built into the inherited base codebase, no separate identity integration project |
| Digital account origination | nCino, Narmi, Alkami | Deposit onboarding and KYC/AML wired directly into the base engine's KYC and audit model from day one |
| Card issuing/processing | Marqeta, Galileo, i2c | Card issuing module shares one permission and audit model with the rest of the bank's operations, not a bolted-on processor console |
| BaaS/sponsor-bank platforms | Unit, Treasury Prime, Synctera | Each buyer runs their own fully independent, self-hosted deployment, so a program layers custom compliance rules on its own codebase without forking anyone else's ledger code |
| Regulatory/call reporting | Ncontracts, Abrigo | Call-report-style extracts generated directly from the same ledger of record, not a reconciled shadow system |

## 6. Personas

- **Bank Operations Manager** — runs daily account operations: exceptions,
  holds, NSF review, dispute intake.
- **Compliance/BSA Officer** — owns KYC/AML onboarding rules, SAR/CTR
  workflows, and regulatory reporting sign-off.
- **Card Program Manager** — configures card products, controls, and
  dispute/chargeback handling.
- **Accountholder (retail/business)** — opens and uses a checking/savings
  account and cards via the bank's own branded digital banking experience.
- **Treasury/Finance Analyst** — reconciles the general ledger, reviews
  interest accrual, and produces financial statements.
- **Fraud Analyst** — reviews fraud rule alerts and disposes of cases.
- **Buyer's own IT/support staff** — the only support layer this deployment
  has; there is no Zodize-operated support console, since each deployment is
  the buyer's own standalone codebase (see
  [admin-template.md](../../templates/admin-template.md)).

## 7. User Journeys

1. **Deposit account opening**: prospect applies online → identity
   document capture and KYC checks run (see §21 Integrations) → risk
   scoring determines auto-approve, manual review, or decline → on
   approval, a checking account and general-ledger sub-account are created
   atomically → welcome package (account number, routing number, first
   card offer) sent via the inherited notification dispatcher (§17).
2. **Card issuance and activation**: accountholder requests a debit card →
   card issuing module provisions a card record (PAN handling per §27) →
   physical card ships or virtual card is available instantly → accountholder
   activates via the digital banking app → card state transitions
   `pending → active` with an audit entry.
3. **ACH transfer with hold and NSF handling**: accountholder initiates an
   outbound ACH transfer → available-balance check runs against ledger
   balance minus holds → if funds are insufficient, the transfer either
   returns NSF (with configurable fee) or triggers linked-account overdraft
   sweep depending on the account's overdraft opt-in state → transfer
   settles on the ACH network's standard timeline and the ledger posts the
   settled entry.
4. **Fraud alert triage**: a card transaction trips a fraud rule (e.g.
   velocity, geography mismatch) → transaction is held pending review →
   Fraud Analyst sees the case in the fraud queue with the triggering
   rule(s) shown → analyst approves, declines, or escalates → decision is
   audit-logged and, if declined, the accountholder is notified per the
   dispute journey.
5. **Regulatory call-report extract**: end of quarter, Compliance/BSA
   Officer runs the call-report-style extract → the system aggregates
   ledger balances by regulatory account category → extract is generated as
   a reviewable, exportable report before submission to the examiner/
   regulator, with the run itself audit-logged.

## 8. Business Goals

- Let a digital bank launch a compliant checking/savings + debit card
  product without building a ledger or KYC pipeline from scratch.
- Keep every dollar movement traceable to an immutable, append-only ledger
  entry that reconciles to the penny.
- Reduce time-to-regulatory-exam-readiness by generating call-report-style
  extracts directly from the system of record.

## 9. Functional Requirements

- Deposit account products: checking, savings, and configurable sub-account
  types (e.g. goal-based savings buckets), admin-configurable per the
  genericized `Plan` pattern (see
  [base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#genericizing-the-plan-pattern)).
- Double-entry general ledger: every account movement posts as a balanced
  debit/credit pair to an append-only ledger; no update-in-place on posted
  entries, only reversing entries.
- Card issuing and lifecycle management: virtual and physical debit cards,
  PIN management, card controls (merchant category block, spend limits,
  travel notices), lost/stolen reissue.
- ACH origination and receipt, and outbound/inbound wire transfer
  initiation with dual-control approval above a configurable threshold.
- Interest accrual: daily accrual, configurable APY tiers per product,
  monthly capitalization, and 1099-INT-style annual summaries.
- Statement generation: monthly e-statements, PDF export, on-demand
  statement regeneration for a closed period.
- KYC/AML onboarding: identity verification, sanctions/watchlist screening,
  risk scoring, and ongoing periodic re-screening.
- Fraud detection rules engine: configurable velocity, geography, device,
  and amount-based rules with a review queue and disposition workflow.
- Regulatory reporting: call-report-style balance/category extracts, SAR/CTR
  case tracking, and exportable audit-ready packages.
- Overdraft/NSF handling: configurable per-product overdraft opt-in,
  linked-account sweep, NSF fee assessment, and return-item processing.
- Dispute/chargeback intake and status tracking for card transactions.
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (wire/ACH dual control), rule engine (fraud rules),
  saved filters and global search over accounts/transactions, custom fields
  on account/customer records, full audit history, soft delete with
  restore on non-ledger entities, mass actions (e.g. bulk statement
  regeneration), import/export wizards for account and transaction data,
  command palette, scheduled/report-builder reporting, system health
  dashboard, white-labeling of the digital banking experience.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiBank-specific additions:

- Ledger posting is transactionally atomic: a transfer either posts both
  legs or neither, with no partial-post state ever visible to a read.
- Balance-check and authorization-decision endpoints target p95 < 200ms,
  since card authorization has a hard network response-time budget.
- 99.99% uptime target for account balance and card authorization services,
  higher than the general product baseline, because an outage there
  directly blocks accountholders' spend.

## 11. Architecture

ZodiBank is built by cloning the sanitized
[base codebase](../../architecture/base-codebase-strategy.md) — a single,
independent Laravel application the buyer deploys entirely on their own
shared/VPS hosting, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
It inherits the base engine's wallet/ledger, payment gateways, RBAC/auth
(`Role`/`Permission` models), KYC, i18n, admin configuration surface, and
audit logging as-is (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps)).

ZodiBank is the one product in the catalog that does **not** run
[Step 2 of the genericization checklist](../../architecture/product-genericization-checklist.md#step-2--strip-banking-domain-specific-tables-models-and-controllers):
it keeps the base engine's `loans`/`loan_plans`, `dps`/`dps_plans`, and
`fdr`/`fdr_plans` tables and controllers, and re-adds them as its own
first-class domain modules — Deposit Accounts, General Ledger, Card
Issuing, Payments, Onboarding & Compliance, Fraud, Regulatory Reporting, and
Overdraft/NSF — built on top of the genericized `Plan` model per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#genericizing-the-plan-pattern),
rather than being stripped and rebuilt from zero. These domain modules
consume the inherited wallet/ledger engine (extended per §14's
multi-currency note), RBAC, KYC, and audit log the same way any other
application code does — they never fork or duplicate an inherited
controller/model to add banking behavior. Card authorization and ACH/wire
processing run as dedicated queued workers (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
isolated from the request path, with a synchronous fallback for buyers on
hosting without persistent worker support, so a processor outage degrades
gracefully instead of blocking the core app. ZodiBank has no runtime
dependency on any other Zodize product or on a Zodize-operated central
service; the only external dependencies are the third-party KYC, card
processor, and payment-rail integrations the buyer's own institution
configures (§22).

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL (ledger tables use append-only, indexed-by-account-and-date
partitioning) + Redis per
[database-standards.md](../../development/database-standards.md); card
issuing/processing and ACH/wire rail integrations are provider-abstracted
behind contracts (see §21) so no processor is hard-coded into ledger logic.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Deposit Accounts | Account Products, Sub-Accounts/Buckets, Holds, Interest Accrual, Statements |
| General Ledger | Chart of Accounts, Posting Engine, Reconciliation, Period Close |
| Card Issuing | Card Products, Card Lifecycle, PIN/Controls, Dispute/Chargeback |
| Payments | ACH Origination/Receipt, Wire Transfer, Dual-Control Approval |
| Onboarding & Compliance | KYC/AML Verification, Watchlist Screening, Risk Scoring, Periodic Re-screening |
| Fraud | Rule Engine, Review Queue, Case Disposition |
| Regulatory Reporting | Call-Report Extracts, SAR/CTR Case Tracking, Exam Packages |
| Overdraft/NSF | Overdraft Opt-In, Linked-Account Sweep, NSF Fees, Return-Item Processing |

## 14. Core Data Model

The 12 entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)).

| Entity | Key columns |
|---|---|
| `deposit_accounts` | id, tenant_id, customer_id, product_type, account_number, routing_number, status, opened_at, overdraft_opt_in |
| `ledger_entries` | id, account_id, entry_type (debit/credit), amount, currency, posted_at, batch_id, reversal_of_entry_id |
| `ledger_batches` | id, tenant_id, source_type, source_reference, status, created_at |
| `customers` | id, tenant_id, legal_name, dob, tax_id_hash, kyc_status, risk_score, screened_at |
| `cards` | id, account_id, customer_id, card_type (virtual/physical), pan_token, last4, status, expires_at |
| `card_transactions` | id, card_id, merchant_name, mcc, amount, currency, auth_status, fraud_flagged, settled_at |
| `payments` | id, account_id, rail (ach/wire), direction, amount, status, dual_control_approved_by, initiated_at |
| `holds` | id, account_id, amount, reason, expires_at, released_at |
| `interest_accruals` | id, account_id, accrual_date, rate_apy, amount, capitalized_at |
| `statements` | id, account_id, period_start, period_end, pdf_document_id, generated_at |
| `fraud_cases` | id, transaction_id, rule_triggered, status, disposition, reviewed_by, reviewed_at |
| `regulatory_reports` | id, tenant_id, report_type, period, status, generated_by, exported_at |

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/accounts` | Open a deposit account |
| GET | `/api/v1/accounts/{id}` | Fetch account detail including live balance |
| GET | `/api/v1/accounts/{id}/transactions` | List ledger transactions for an account |
| POST | `/api/v1/accounts/{id}/holds` | Place a hold on funds |
| POST | `/api/v1/customers/{id}/kyc/verify` | Trigger/re-trigger KYC verification |
| POST | `/api/v1/cards` | Issue a card against an account |
| POST | `/api/v1/cards/{id}/activate` | Activate a card |
| PATCH | `/api/v1/cards/{id}/controls` | Update card spend controls |
| POST | `/api/v1/cards/{id}/report-lost` | Report a card lost/stolen and trigger reissue |
| POST | `/api/v1/payments/ach` | Initiate an ACH transfer |
| POST | `/api/v1/payments/wire` | Initiate a wire transfer |
| POST | `/api/v1/payments/{id}/approve` | Dual-control approval of a pending payment |
| GET | `/api/v1/accounts/{id}/statements` | List statements for an account |
| GET | `/api/v1/statements/{id}/pdf` | Download a statement PDF |
| GET | `/api/v1/fraud-cases` | List fraud review queue |
| POST | `/api/v1/fraud-cases/{id}/disposition` | Resolve a fraud case |
| POST | `/api/v1/disputes` | File a card transaction dispute |
| GET | `/api/v1/regulatory-reports` | List generated regulatory reports |
| POST | `/api/v1/regulatory-reports/call-report` | Generate a call-report-style extract |
| GET | `/api/v1/ledger/reconciliation` | Reconciliation status of the general ledger |

## 16. Events

Domain events registered on ZodiCore's event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`account.opened`, `account.closed`, `kyc.approved`, `kyc.declined`,
`kyc.manual_review_required`, `ledger.entry_posted`, `ledger.entry_reversed`,
`card.issued`, `card.activated`, `card.blocked`, `card.reissued`,
`payment.initiated`, `payment.approved`, `payment.settled`,
`payment.returned`, `overdraft.triggered`, `nsf.fee_assessed`,
`fraud.case_opened`, `fraud.case_resolved`, `dispute.filed`,
`regulatory_report.generated`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `account.opened` | ✔ | ✔ (welcome packet) | — | — |
| `kyc.manual_review_required` | ✔ (to compliance queue) | — | — | — |
| `card.activated` | ✔ | ✔ | ✔ | ✔ |
| `payment.settled` (large amount) | ✔ | ✔ | ✔ | ✔ |
| `nsf.fee_assessed` | ✔ | ✔ | ✔ | ✔ |
| `fraud.case_opened` | ✔ (to fraud analyst) | — | ✔ (to accountholder, if card held) | ✔ |
| `dispute.filed` | ✔ | ✔ (confirmation) | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Extends [ZodiCore's default roles](../../security/rbac-permissions.md#default-system-roles)
with banking-specific roles: `Bank Operations Manager`, `Compliance Officer`,
`Card Program Manager`, `Fraud Analyst`, `Treasury Analyst`. Key
permissions: `accounts.open`, `accounts.close`, `ledger.post`,
`ledger.reverse` (restricted to Treasury Analyst/Compliance), `cards.issue`,
`cards.block`, `payments.initiate`, `payments.approve` (never the same
identity as `payments.initiate`, enforced per §19), `kyc.review`,
`fraud.disposition`, `regulatory_reports.generate`. Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **Dual-control payment approval**: any ACH/wire above a tenant-configured
  threshold requires approval from a user distinct from the initiator;
  self-approval is blocked at the policy layer, not just the UI.
- **KYC manual review escalation**: applications that fail automated
  risk-scoring thresholds route to a Compliance Officer queue before any
  account is opened; no account activates without an explicit approve/
  decline decision recorded.
- **Fraud case disposition**: a held transaction requires a Fraud Analyst
  decision (approve/decline/escalate) before funds move; escalated cases
  route to Compliance for SAR/CTR evaluation.
- **Ledger reversal approval**: reversing a posted ledger entry requires a
  Treasury Analyst or Compliance Officer role and always creates a new
  offsetting entry — original entries are never deleted or edited.

## 20. Audit Logs

Every ledger post, reversal, account status change, KYC decision, card
lifecycle transition, payment approval, and fraud disposition writes an
immutable audit entry via ZodiCore's audit log
([audit-logging.md](../../security/audit-logging.md)), capturing actor,
timestamp, before/after state, and (for payments) both the initiator and
approver identities. Ledger entries are additionally protected by database-
level append-only constraints so audit and ledger integrity cannot diverge.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: account growth, deposit balances, card activation
  rate, NSF/overdraft incidence, fraud case backlog — per
  [dashboard-standards.md](../../standards/dashboard-standards.md).
- Regulatory: call-report-style balance-by-category extracts, SAR/CTR case
  register, KYC exception report.
- Finance: daily/monthly ledger reconciliation report, interest expense
  report, statement generation audit.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **KYC/AML providers**: identity verification and document capture (e.g.
  Persona/Alloy/Socure-class vendors), sanctions/watchlist screening
  (OFAC-class screening providers), abstracted behind a
  `KycProviderContract` so tenants can swap vendors without ledger changes.
- **Card issuing/processing**: card issuer processor integration (Marqeta/
  Galileo/i2c-class) behind a `CardProcessorContract`.
- **Payment rails**: ACH origination/receipt via an ACH processor
  integration, wire transfer via a correspondent-bank/Fedwire-class
  integration, abstracted behind a `PaymentRailContract`.
- **Core banking / ledger reconciliation**: optional export connectors for
  institutions running ZodiBank alongside a legacy core during migration.
- **Credit bureau / identity verification**: for products offering
  overdraft lines or secured credit, integrates with credit bureau APIs.
- **Tax reporting**: 1099-INT-style annual summary export.

## 23. AI Features

- Fraud rule tuning assistant: surfaces false-positive/false-negative rate
  per rule to Fraud Analysts and suggests threshold adjustments, always
  requiring human confirmation before a rule changes.
- KYC risk-score explanation: for manual-review cases, summarizes in plain
  language which signals drove the risk score, to speed compliance review.
- Anomalous-ledger-activity detection layered on top of ZodiCore's
  audit-log anomaly detection ([ZodiCore §23](../ZodiCore/SPEC.md#23-ai-features)),
  tuned for banking-specific patterns (e.g. structuring-like transaction
  sequences) and routed to the Compliance queue.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: nightly interest accrual, monthly statement generation,
  periodic KYC re-screening, ACH/wire settlement polling, call-report
  extract generation on a regulatory calendar, dormant-account flagging.
- CLI commands (Artisan): `bank:accrue-interest`, `bank:generate-statements`,
  `bank:rescreen-customers`, `bank:reconcile-ledger`,
  `bank:generate-call-report` — each requires the same authorization context
  as its API equivalent, per [ZodiCore §24](../ZodiCore/SPEC.md#24-automation-scheduled-jobs-cron-jobs-cli-commands).

## 25. Seed/Demo Data

`BankDemoSeeder` provisions a demo tenant with realistic checking/savings
products, 200+ synthetic customers with varied KYC states, 12 months of
transaction and interest-accrual history, a populated fraud-case queue with
resolved and open cases, and a full quarter of regulatory-report history —
per [migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: card authorization decisions complete in under
300ms p95 end-to-end (fraud rule evaluation included), and ledger balance
reads are always strongly consistent (no eventual-consistency window on
balance display).

## 27. Security Requirements

Financial products carry Zodize's highest security/compliance bar. Full
baseline from [security-standards.md](../../security/security-standards.md)
applies, plus:

- **PCI-DSS-equivalent handling** for all card data: PAN is tokenized at
  ingestion, never stored in plaintext, and never logged; card data access
  is scoped to the minimum service boundary required (issuing/processing
  integration only).
- **SOC2-equivalent controls**: change management, access review cadence,
  and vendor risk management apply to every banking module, not only the
  ZodiCore platform baseline.
- **KYC/AML**: mandatory identity verification and sanctions screening
  before any account activates; periodic re-screening is enforced by
  scheduled job, not manual process.
- **Immutable audit trails**: ledger and compliance-decision audit entries
  are append-only at the database layer, matching §20.
- **MFA is mandatory, not optional**, for every human user role interacting
  with ZodiBank — enforced at the tenant policy level per
  [authentication-authorization.md](../../security/authentication-authorization.md),
  with no self-service opt-out available to Bank Operations, Compliance,
  Card Program, Fraud, or Treasury roles.
- Dual control on payment approval (§19) is a security control, not merely
  a workflow convenience, and cannot be disabled by a tenant admin.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated ledger-integrity test suite asserting every posted batch nets
to zero across debits/credits, and a dual-control-bypass regression suite
that must pass before any payments-module release.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Ledger-posting and card-authorization services deploy with the same
zero-downtime requirement as ZodiCore's identity/billing services
([ZodiCore §29](../ZodiCore/SPEC.md#29-deployment-requirements)), since an
authorization outage blocks accountholder spend in real time.

## 30. Acceptance Criteria

- A prospective customer can complete deposit account opening end-to-end
  (application → KYC → account creation → welcome notification) with no
  manual intervention for the auto-approve path.
- Every ledger-affecting action produces a balanced, immutable ledger entry
  that reconciles to zero at the batch level.
- A payment above the dual-control threshold cannot settle without a
  distinct approver, verified by an automated regression test.
- A card can be issued, activated, controlled, and reported lost/reissued
  entirely through the API without direct database access.
- A call-report-style regulatory extract can be generated for any closed
  period and matches the underlying ledger balances exactly.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiBank additionally requires sign-off from a compliance stakeholder that
KYC/AML, dual-control payment approval, and regulatory reporting have been
validated against the tenant's actual chartering/sponsor-bank requirements
before go-live.

## 32. Future Roadmap

- Real-time payments (FedNow/RTP-class rail) support alongside ACH/wire.
- Configurable multi-tier interest products (relationship-based APY
  boosts).
- Business banking module: ACH batch origination, multi-user account
  entitlements, positive pay.

## 33. Known Risks

- Processor dependency: card authorization latency is bounded by the
  third-party processor's response time — mitigated by the
  `CardProcessorContract` abstraction, but a processor outage remains an
  external dependency outside Zodize's direct control.
- Regulatory divergence: call-report and SAR/CTR formats vary by
  jurisdiction and regulator; the current extract format targets US-style
  reporting and will need per-jurisdiction variants as the product expands.

## 34. Future Improvements

- Configurable ledger chart-of-accounts templates per regulatory
  jurisdiction.
- Expanded fraud rule engine with machine-learning-scored risk in addition
  to deterministic rules.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram and migration set for the ledger and compliance schema (companion
`DATA_MODEL.md`), a complete endpoint catalog (companion `API_REFERENCE.md`)
matching [ZodiCore's structure](../ZodiCore/SPEC.md), and a full regulatory
report catalog covering additional jurisdictions beyond the US call-report
format. Changes follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).
