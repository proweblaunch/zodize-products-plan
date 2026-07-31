# ZodiBusiness — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; exhaustive ER diagrams and a full endpoint catalog
> are queued — see [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiBusiness is the small-and-midsize-business operating system: CRM,
inventory, quoting, invoicing, and core accounting in one self-hosted
application, so an SMB owner runs their whole operation without stitching
together a CRM tool, a spreadsheet-based inventory count, and a separate
accounting package that never quite reconciles with each other.

## 2. Purpose

Most SMB "ERP" tools force a choice: a CRM that bolts on weak invoicing, or
an accounting package that bolts on weak CRM. ZodiBusiness exists because a
lead, a quote, an invoice, a payment, and a journal entry are the same
business event viewed from five angles — they belong in one data model, not
five integrated-but-separate SaaS subscriptions with sync lag between them.

## 3. Target Market

SMBs with 5–250 employees and $500K–$50M in annual revenue across services,
light manufacturing, wholesale distribution, and professional services, who
have outgrown spreadsheets and QuickBooks-plus-a-CRM-app but don't need
full enterprise ERP (SAP/Oracle NetSuite) complexity or cost.

## 4. Industries

General SMB — professional services, wholesale/distribution, light
manufacturing, field services, and consulting firms are the primary
verticals; the product is intentionally horizontal within the SMB segment
per [product-philosophy.md](../../development/product-philosophy.md), with
industry-specific depth left to dedicated Zodize verticals (e.g.
[ZodiBuild](../ZodiBuild/SPEC.md) for construction) where it exists.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| SMB accounting | QuickBooks Online, Xero | Native CRM and inventory in the same ledger, not a third-party app integration |
| SMB CRM | HubSpot CRM (free/starter tiers), Zoho CRM | Deal-to-invoice flow is one workflow, not a CRM-to-accounting handoff via Zapier |
| Inventory for SMB | inFlow, Cin7 (entry tiers) | Shared stock ledger with the same quoting/invoicing engine, not a separate app |
| Quote-to-cash | PandaDoc + QuickBooks combo | One approval chain and audit trail from quote through paid invoice |
| Vendor/PO management | Bill.com (AP side only) | Purchase orders and vendor bills live in the same chart of accounts as sales |

## 6. Personas

- **Owner/Principal** — the SMB owner overseeing sales pipeline, cash flow,
  and overall business health.
- **Sales Rep** — manages leads, deals, and quotes through the pipeline.
- **Operations/Inventory Manager** — manages stock levels, warehouses, and
  purchase orders.
- **Bookkeeper/Accountant** — manages the chart of accounts, reconciles
  transactions, and closes books monthly.
- **Customer** — receives quotes and invoices; may have a self-service
  portal to view/pay invoices.
- **Vendor** — receives purchase orders; not a system user, tracked as a
  record.

## 7. User Journeys

1. **Lead to deal to quote**: Sales Rep logs a new lead → qualifies it into a
   deal in the pipeline → moves it through configurable pipeline stages →
   generates a quote from the deal with line items pulled from the product
   catalog → sends it to the customer for e-signature/acceptance.
2. **Quote to invoice to payment**: Customer accepts the quote → it converts
   to a sales order → an invoice is generated (partially or fully, per
   milestone billing) → customer pays via the payment link → the payment
   auto-posts to the ledger as a debit to cash and credit to accounts
   receivable, closing the invoice.
3. **Inventory-aware quoting**: Sales Rep builds a quote for a physical
   product → the system checks available stock across warehouses in
   real-time → if insufficient, flags a backorder risk and suggests the
   nearest warehouse with stock or a reorder-triggering purchase order.
4. **Purchase order to vendor bill**: Inventory Manager creates a purchase
   order for a low-stock item → sends it to the vendor → receives the goods,
   incrementing stock at the receiving warehouse → the vendor's bill is
   matched against the PO (three-way match: PO, receipt, bill) → Bookkeeper
   approves and schedules payment.
5. **Month-end close**: Bookkeeper reviews the trial balance → reconciles
   bank transactions against ledger entries → posts adjusting journal
   entries → locks the accounting period, after which no journal entry
   before that date can be posted or edited without an explicit unlock
   permission and audit trail.

## 8. Business Goals

- Replace the CRM-plus-accounting-plus-spreadsheet stack with one system of
  record, reducing manual reconciliation time.
- Shorten the average lead-to-cash cycle by removing handoff friction between
  sales and billing.
- Give SMB owners a real-time cash-position view instead of a
  once-a-month bookkeeper report.

## 9. Functional Requirements

- CRM: leads, contacts, companies, deals/opportunities with configurable
  pipeline stages, activity logging (calls, emails, notes, tasks).
- Inventory: multi-warehouse stock tracking, stock transfers, reorder
  points, and lot/serial tracking for applicable industries.
- Quote-to-invoice flow: quotes with line items and discounts, e-signature
  acceptance, conversion to sales orders and invoices, partial/milestone
  invoicing.
- Expense tracking: employee expense submission, receipt capture, approval
  routing, reimbursement.
- Core accounting: configurable chart of accounts, double-entry journal
  engine, accounts receivable/payable aging, bank reconciliation, trial
  balance, basic financial statements (P&L, balance sheet).
- Purchase orders: PO creation, vendor approval routing, receiving against a
  PO, three-way match against the vendor bill.
- Vendor management: vendor records, contact/terms, purchase history,
  1099-equivalent reporting readiness.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains for expenses/POs/period-close, saved pipeline views,
  custom fields on deals/invoices/vendors, CSV import/export wizard for
  contacts and inventory, mass actions on invoice lists, full audit
  history and version history per record, soft delete + restore.

## 10. Non-Functional Requirements

Inherits the baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md).
ZodiBusiness-specific additions:

- Financial reports (trial balance, P&L) must generate for a 12-month range
  in under 5 seconds for a deployment with up to 500,000 journal lines.
- Ledger posting is transactionally atomic — a payment that touches both
  accounts receivable and cash must never leave the books unbalanced, even
  under partial failure.

## 11. Architecture

ZodiBusiness — the first product in the build order
([ROADMAP.md](../../../ROADMAP.md)) — is built by cloning the sanitized base
codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md).
Because ZodiBusiness's own shape (users, wallet, plans, accounting-adjacent
records) is the closest fit to the base codebase as audited, this clone is
the reference example every other product's genericization pass follows: the
banking-specific `loans`/`dps`/`fdr`/`branches`/`branch_staff`/
`other_banks`/`beneficiaries`/`airtime` tables are stripped (none of them
serve an SMB CRM/inventory/accounting product), and the `branch_staff` guard
is dropped by default, per
[product-genericization-checklist.md](../../architecture/product-genericization-checklist.md#step-2--strip-banking-domain-specific-tables-models-and-controllers).
ZodiBusiness inherits the base engine's wallet/ledger, payment gateways
(§22), RBAC/auth, KYC, i18n, and admin configuration surface unmodified —
see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
and layers its own CRM, Inventory, Sales, Expenses, Accounting, and
Procurement modules (§13) on top, per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#layering-a-products-domain-modules-onto-the-sanitized-base).

CRM, inventory, and accounting are distinct modules within the same modular
monolith and the same one database of the buyer's single deployment —
deliberately avoiding a microservice split between "CRM" and "accounting"
because the quote-to-cash flow requires strict transactional consistency
between them (a paid invoice and its journal entry must commit together in
one Laravel database transaction), per
[overview.md](../../architecture/overview.md#modular-monolith-one-codebase-per-product).
There is no shared tenant boundary and no ZodiCore platform dependency: each
ZodiBusiness deployment is one SMB's standalone, self-hosted instance, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Multi-company/multi-branch support — letting a single SMB operate multiple
legal entities with consolidated and per-entity reporting — is scoping
within this one deployment, via the `company_id`/`branch_id` model in §14,
per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping),
never tenancy.

## 12. Technology

Laravel (PHP) per the base codebase's stack (Laravel 11, PHP ^8.3, Vite 5) —
see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
following
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md);
MySQL/MariaDB + Redis (where the buyer's hosting supports it, with a file/DB
cache fallback) per
[database-standards.md](../../development/database-standards.md); a
dedicated append-only `journal_entries` ledger table enforced at the
database layer to be immutable once posted (corrections via reversing entry,
never edited in place), mirroring the inherited wallet engine's own
append-only `Transaction` pattern documented in
[wallet-system.md](../../standards/wallet-system.md).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| CRM | Leads, Contacts/Companies, Pipeline/Deals, Activities, Tasks |
| Inventory | Stock Ledger, Warehouses, Transfers, Reorder Rules, Lot/Serial Tracking |
| Sales | Quotes, Sales Orders, Invoices, Payment Links, Recurring Invoices |
| Expenses | Expense Submission, Receipt Capture, Approval Routing, Reimbursement |
| Accounting | Chart of Accounts, Journal Engine, AR/AP Aging, Bank Reconciliation, Period Close |
| Procurement | Purchase Orders, Vendor Management, Receiving, Three-Way Match |
| Reporting | Financial Statements, Sales Pipeline Reports, Inventory Valuation |

## 14. Core Data Model

| Entity | Key columns |
|---|---|
| `leads` | id, tenant_id, name, source, status, owner_id, created_at |
| `deals` | id, tenant_id, contact_id, pipeline_stage, value, expected_close_date |
| `contacts` | id, tenant_id, company_id, name, email, phone |
| `products` | id, tenant_id, sku, name, unit_cost, unit_price, is_stocked |
| `warehouses` | id, tenant_id, name, address |
| `stock_levels` | id, product_id, warehouse_id, quantity_on_hand, reorder_point |
| `quotes` | id, tenant_id, deal_id, status, total, accepted_at |
| `invoices` | id, tenant_id, customer_id, quote_id, status, due_date, balance_due |
| `invoice_items` | id, invoice_id, product_id, quantity, unit_price |
| `expenses` | id, tenant_id, submitted_by, amount, category, status, approved_by |
| `chart_of_accounts` | id, tenant_id, code, name, type (asset/liability/equity/revenue/expense) |
| `journal_entries` | id, tenant_id, entry_date, reference, posted_at, period_id |
| `journal_lines` | id, journal_entry_id, account_id, debit, credit |
| `purchase_orders` | id, tenant_id, vendor_id, status, total, expected_at |
| `vendors` | id, tenant_id, name, terms, payment_method |

## 15. Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/leads` | List leads with source/status filters |
| POST | `/api/v1/deals` | Create a deal in the pipeline |
| PATCH | `/api/v1/deals/{id}/stage` | Move a deal to a new pipeline stage |
| POST | `/api/v1/quotes` | Create a quote from a deal |
| POST | `/api/v1/quotes/{id}/accept` | Record customer acceptance, convert to sales order |
| POST | `/api/v1/invoices` | Generate an invoice from a sales order or quote |
| GET | `/api/v1/invoices/{id}` | Invoice detail including payment status |
| POST | `/api/v1/invoices/{id}/payments` | Record a payment against an invoice |
| GET | `/api/v1/inventory/{product_id}` | Stock levels by warehouse |
| POST | `/api/v1/inventory/transfers` | Transfer stock between warehouses |
| POST | `/api/v1/expenses` | Submit an expense for approval |
| POST | `/api/v1/expenses/{id}/approve` | Approve or reject an expense |
| GET | `/api/v1/accounting/chart-of-accounts` | List the chart of accounts |
| POST | `/api/v1/accounting/journal-entries` | Post a manual journal entry |
| GET | `/api/v1/accounting/trial-balance` | Trial balance for a date range |
| GET | `/api/v1/accounting/profit-and-loss` | P&L statement for a date range |
| POST | `/api/v1/purchase-orders` | Create a purchase order |
| POST | `/api/v1/purchase-orders/{id}/receive` | Record goods receipt against a PO |
| POST | `/api/v1/vendor-bills` | Record a vendor bill for three-way match |
| GET | `/api/v1/reports/ar-aging` | Accounts receivable aging report |

## 16. Events

`lead.created`, `deal.stage_changed`, `quote.accepted`, `invoice.issued`,
`invoice.paid`, `invoice.overdue`, `expense.submitted`, `expense.approved`,
`expense.rejected`, `journal_entry.posted`, `period.closed`,
`purchase_order.created`, `purchase_order.received`, `stock.low`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `quote.accepted` | ✔ (Sales Rep) | ✔ | — | — |
| `invoice.issued` (to customer) | — | ✔ | — | — |
| `invoice.overdue` | ✔ (Owner/Bookkeeper) | ✔ | — | — |
| `expense.submitted` | ✔ (approver) | ✔ | — | — |
| `stock.low` | ✔ (Inventory Manager) | ✔ | — | — |
| `period.closed` | ✔ (Bookkeeper, Owner) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Extends ZodiCore's default roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles))
with: `deals.manage`, `inventory.adjust`, `invoices.issue`,
`payments.record`, `expenses.approve`, `journal_entries.post`,
`period.close`, `purchase_orders.approve`. `journal_entries.post` and
`period.close` are restricted to the `Owner` and a dedicated `Accountant`
role by default — no `Member`-level role can post to the ledger.

## 19. Workflows & Approval Chains

- **Expense approval**: submitted expenses route to the submitter's manager
  (or Owner if unset); expenses above a configurable threshold require a
  second approval from `Accountant` before reimbursement is scheduled.
- **Purchase order approval**: POs above a configurable dollar threshold
  require Owner or Accountant approval before being sent to the vendor.
- **Period close**: closing an accounting period requires Accountant role;
  reopening a closed period requires Owner-level approval and is fully
  audit-logged, per [audit-logging.md](../../security/audit-logging.md).
- **Three-way match exception**: if a vendor bill doesn't match its PO and
  receipt within tolerance, it routes to Accountant for manual resolution
  instead of auto-posting.

## 20. Audit Logs

Every deal-stage change, invoice issuance/edit, journal entry post/reversal,
expense approval decision, and period close/reopen is recorded to ZodiCore's
shared audit log with actor, before/after state, and IP/device context, per
[audit-logging.md](../../security/audit-logging.md). Journal entries are
themselves append-only, so the audit log and the ledger are mutually
reinforcing records.

## 21. Reports & Analytics & Dashboards

Sales pipeline funnel and win-rate by rep, AR/AP aging, cash-flow forecast
(based on open invoices and scheduled bill payments), inventory valuation
(FIFO/weighted-average configurable per tenant), P&L and balance sheet, and
an owner-facing "business health" dashboard combining pipeline, cash
position, and overdue invoices. Dashboard-builder and scheduled-report
capability per [dashboard-standards.md](../../standards/dashboard-standards.md).

## 22. Integrations

- **Payment processing**: Stripe, PayPal, ACH via the `PaymentGatewayContract`
  abstraction from [ZodiCore §20](../ZodiCore/SPEC.md#20-payment-gateways-wallet-accounting-taxes-invoices).
- **Bank feeds**: Plaid-class bank-connection integration for automatic
  transaction import into reconciliation.
- **Tax filing prep**: export formats compatible with common SMB tax-prep
  workflows (1099 vendor summaries, sales tax liability reports).
- **E-signature**: DocuSign/HelloSign-class integration for quote acceptance.
- **Email/calendar sync**: Google Workspace/Microsoft 365 for CRM activity
  logging.

## 23. AI Features

- AI-assisted deal scoring: surfaces a likelihood-to-close signal on the
  pipeline view based on activity recency and deal attributes.
- Expense receipt OCR: auto-extracts vendor, amount, and date from a
  photographed receipt into the expense form as an editable draft.
- Cash-flow forecasting assistant: projects near-term cash position from
  open invoices and recurring bills, flagging projected shortfalls.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: recurring invoice generation, overdue-invoice reminder
  emails, low-stock reorder-point sweep, bank-feed transaction import,
  nightly trial-balance snapshot.
- CLI commands: `business:generate-recurring-invoices`,
  `business:close-period {period_id}`, `business:reconcile-bank {account_id}`,
  `business:export-financials`.

## 25. Seed/Demo Data

`DemoSeeder` provisions a demo SMB tenant with a populated pipeline (leads
through won/lost deals), a 2-warehouse inventory with realistic stock
levels, 12 months of invoice/payment history including at least one overdue
invoice, a standard chart of accounts with populated journal history, and 3
demo vendors with open and closed purchase orders, per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the pipeline kanban view must remain responsive with
5,000+ open deals via server-side pagination, and invoice PDF generation
completes in under 2 seconds p95.

## 27. Security Requirements

Full baseline from
[security-standards.md](../../security/security-standards.md) applies.
Financial data (bank connection tokens, payment references) is encrypted at
rest per [data-protection-privacy.md](../../security/data-protection-privacy.md);
journal entry mutation is blocked at the database layer once posted, not
just at the application layer, to prevent a compromised admin session from
silently altering historical books.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated ledger-balance test suite asserting every posted journal entry
sums debits to credits, and a three-way-match test suite covering
over-receipt, under-receipt, and price-variance scenarios.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). The
accounting module's journal posting endpoint is included in the
zero-downtime deployment path — no maintenance window may block invoice
payment posting during business hours.

## 30. Acceptance Criteria

- A lead can move end-to-end through the pipeline to a paid invoice with the
  correct journal entries posted automatically, with zero manual ledger
  entry.
- A purchase order's three-way match correctly flags a vendor bill that
  doesn't match the PO/receipt within tolerance, routing it for manual
  review instead of silently posting.
- A closed accounting period cannot accept a new or edited journal entry
  without an explicit, audited reopen action.
- The trial balance is always balanced (total debits equal total credits)
  for any date range, verified by an automated reconciliation check.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiBusiness additionally requires sign-off that the ledger-balance
invariant test suite (§28) passes against a full year of seeded demo
transaction volume before go-live.

## 32. Future Roadmap

- Multi-currency consolidated reporting across subsidiary entities.
- Payroll module integration (or first-party payroll processing).
- Bank-feed-driven auto-categorization rules with a learning suggestion
  model for recurring vendor transactions.

## 33. Known Risks

- Ledger correctness is existential: a bug in the journal engine that
  unbalances the books undermines every financial report — mitigated by the
  dedicated ledger-balance test suite (§28) and append-only enforcement, but
  this remains the module warranting the highest review bar in the product.
- SMB users often lack a dedicated bookkeeper, increasing the risk of
  misconfigured chart-of-accounts or misapplied journal entries — mitigated
  by seeded standard chart-of-accounts templates, but ongoing UX investment
  in guided reconciliation is needed as the user base scales downmarket.

## 34. Future Improvements

- Guided chart-of-accounts setup wizard tailored to industry (services vs.
  distribution vs. light manufacturing).
- Automated bank-transaction-to-journal-entry matching with confidence
  scoring before requiring bookkeeper confirmation.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram covering multi-entity consolidation and lot/serial tracking tables,
the complete endpoint catalog (bulk journal import, full financial statement
endpoint set), and a dedicated `DATA_MODEL.md`/`API_REFERENCE.md` pair
matching [ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.
