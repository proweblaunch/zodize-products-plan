# ZodiLaw — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth). See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for spec status
> definitions.

## 1. Vision

ZodiLaw is a legal practice management platform that runs a firm's matters,
billable time, trust (IOLTA) accounting, documents, and court deadlines as
one system of record, so a firm never has to choose between a time-and-
billing tool, a document management system, and a separate calendar/
deadline tracker that don't share a conflict check or an audit trail.

## 2. Purpose

Legal practice carries two categories of risk that generic practice
software does not model correctly: missing a statute-of-limitations or
court deadline (malpractice exposure), and mishandling client trust funds
(bar discipline and disbarment exposure). ZodiLaw exists to make deadline
calculation with escalating alerts, IOLTA-compliant trust accounting, and
conflict-of-interest checking default, enforced behaviors of the system
rather than practices a firm must separately discipline itself to follow.

## 3. Target Market

Solo practitioners, small and mid-size law firms (litigation, transactional,
and mixed practice), and in-house legal departments that need matter
management, billing, trust accounting, and document management without
enterprise legal-suite implementation cost.

## 4. Industries

Legal services across practice areas: litigation, family law, real estate/
transactional, estate planning, personal injury, and corporate/in-house
legal departments.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Matter management + billing | Clio, MyCase | Built on ZodiCore's shared audit/RBAC core instead of a standalone legal-only stack |
| Trust/IOLTA accounting | Clio Manage Trust Accounting, LeanLaw | Trust ledger enforced as a segregated, append-only structure at the data-model level, matching bar trust-accounting rules |
| Document management with version control | NetDocuments, iManage | Version history and access control share the same engine as ZodiCore's platform-wide version history capability |
| Conflict-of-interest checking | Intapp Conflicts, Clio's basic conflict search | Conflict search spans every matter, party, and related-entity field firm-wide, not just party-name matching |
| Court deadline/calendar with SOL tracking | CalendarRules, Clio Court Rules | Deadline rule engine surfaces escalating alerts tied to matter risk level, not a single reminder |

## 6. Personas

- **Managing Partner/Firm Administrator** — oversees firm-wide matters,
  billing rates, trust compliance, and conflict policy.
- **Attorney** — manages assigned matters, tracks billable time, drafts and
  reviews documents, monitors deadlines.
- **Paralegal/Legal Assistant** — supports matter work, drafts documents,
  tracks deadlines under attorney supervision.
- **Billing/Trust Administrator** — manages invoicing, trust account
  reconciliation, and IOLTA compliance.
- **Client** — views matter status, documents, invoices, and makes payments
  via the client portal.

## 7. User Journeys

1. **Intake with conflict check**: Prospective client intake form captures
   the client, adverse parties, and related entities → firm-wide conflict
   search runs across every existing matter's parties and related-entity
   fields → any hit surfaces to the Managing Partner for a documented
   waiver-or-decline decision before the matter is opened → cleared intake
   converts to a new matter with the attorney(s) of record assigned.
2. **Time tracking to invoice**: Attorney/paralegal logs billable time
   against a matter (timer or manual entry) with a narrative description →
   entries accumulate against the matter's billing arrangement (hourly,
   flat fee, or contingency) → Billing Administrator reviews and edits
   pre-bill entries → invoice is generated, applying any trust retainer
   balance first → invoice is sent to the client via the portal → payment
   received posts to the operating account (fees) and, if applicable, draws
   down the trust retainer per §19.
3. **Trust retainer lifecycle**: Client trust retainer is collected and
   deposited into the IOLTA trust account, ledgered against the specific
   client/matter (never commingled with firm operating funds or other
   clients' funds) → as invoices are approved for payment from the retainer,
   funds are transferred from trust to the operating account in the exact
   invoiced amount, with a corresponding trust ledger entry → trust account
   reconciliation runs monthly, comparing the trust ledger's client-by-
   client balances against the bank balance, required to match to the cent
   before the period closes.
4. **Court deadline calculation and escalating alerts**: A triggering event
   (e.g. date of service, filing date) is entered against a matter → the
   deadline rule engine calculates dependent deadlines (e.g. answer due,
   discovery cutoff, statute of limitations) per the applicable
   jurisdiction's rule set → alerts escalate as the deadline approaches
   (e.g. 30/14/7/1 day) and escalate further in urgency and recipient
   (assigned attorney, then supervising partner) if unacknowledged →
   missing a deadline without an acknowledgment trail is treated as a
   near-miss requiring firm-wide review.
5. **Document version control and client portal sharing**: Attorney drafts
   a document against a matter → each save creates a new version with full
   history retained, never overwriting a prior version → attorney marks a
   version as the one to share → client views/downloads the shared version
   via the client portal, scoped strictly to their own matter's documents,
   with privileged internal work product never exposed to the portal by
   default.

## 8. Business Goals

- Eliminate the deadline-miss failure mode that is the single largest
  source of legal malpractice claims industry-wide.
- Make IOLTA-compliant trust accounting the default behavior so firms pass
  bar trust audits without a separate accounting system.
- Reduce unbilled/unrecorded time leakage through low-friction time capture
  integrated with matter work.
- Give firms enterprise-grade (iManage/NetDocuments-class) document version
  control without enterprise DMS implementation cost.

## 9. Functional Requirements

- Matter/case management: matter intake, conflict checking, matter status,
  assigned timekeepers, related parties.
- Time tracking and billing: timer and manual time entry, billing
  arrangements (hourly/flat/contingency), pre-bill review, invoice
  generation, payment application.
- Trust/IOLTA accounting: client/matter-segregated trust ledger, retainer
  deposit and drawdown, monthly three-way reconciliation (ledger, bank,
  client balances).
- Document management: version-controlled document storage per matter,
  templates, client-portal sharing controls distinguishing privileged
  work product from shareable documents.
- Conflict-of-interest checking: firm-wide search across parties and
  related entities at intake and ongoing matter changes.
- Court deadline/calendar: triggering-event-driven deadline calculation,
  jurisdiction-aware rule sets, escalating alerting, acknowledgment
  tracking.
- Client portal: matter status, shared documents, invoices, trust balance
  (where applicable to jurisdiction disclosure norms), online payment.
- Conflict/ethical wall enforcement: restricting specific timekeepers from
  a matter where a conflict waiver requires it.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiLaw-specific additions:

- Conflict search across a firm's full matter/party history returns in
  under 3 seconds p95, since it sits in the critical path of every new
  matter intake and cannot be skipped for speed.
- Trust ledger writes are append-only and reconciliation reports are
  computed, never manually overridden, to preserve audit integrity.
- Deadline calculation and alert dispatch is treated as a high-priority
  queue with delivery confirmation, not best-effort — a missed alert
  delivery is itself an incident.

## 11. Architecture

ZodiLaw is a tenant application on [ZodiCore](../ZodiCore/SPEC.md),
consuming `zodize/core-identity`, `zodize/core-billing`,
`zodize/core-notifications`, `zodize/core-permissions`, and
`zodize/core-plugins` per ZodiCore's
[Architecture](../ZodiCore/SPEC.md#11-architecture) section. ZodiLaw adds a
`PrivilegeSegregationContract` layered on ZodiCore's RBAC engine that scopes
matter access to assigned timekeepers and enforces ethical-wall exceptions
per matter, distinct from standard role-based permission — a user with a
firm-wide "Attorney" role does not automatically see every matter, only
matters they are assigned to or explicitly granted access to. Trust ledger
writes go through a dedicated `TrustLedgerContract` enforcing double-entry,
append-only, client/matter-segregated posting, independent of ZodiCore's
platform billing ledger (ZodiLaw's trust ledger represents client funds
under fiduciary duty, not Zodize's own revenue). Multi-office firms map onto
ZodiCore's tenant/company/branch hierarchy per
[multi-tenancy.md](../../architecture/multi-tenancy.md).

## 12. Technology

Laravel + Vue per the shared stack
([coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md),
[coding-standards-vue.md](../../development/coding-standards-vue.md));
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); document
storage with full version history per tenant-scoped document storage
conventions; payment processing (invoice payment, trust deposits) via
ZodiCore's payment gateway abstraction
(§20 of [ZodiCore's SPEC.md](../ZodiCore/SPEC.md#20-payment-gateways-wallet-accounting-taxes-invoices)),
with trust deposits routed to a dedicated trust bank account distinct from
the firm's operating account at the integration level.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Intake & Conflicts | Intake Forms, Conflict Search, Waiver/Decline Decisions, Matter Opening |
| Matter Management | Matter Records, Parties, Timekeeper Assignment, Ethical Walls |
| Time & Billing | Time Entry, Billing Arrangements, Pre-Bill Review, Invoicing, Payment Application |
| Trust Accounting | IOLTA Ledger, Retainer Deposit/Drawdown, Three-Way Reconciliation |
| Documents | Version-Controlled Storage, Templates, Client-Portal Sharing Controls |
| Calendar & Deadlines | Deadline Rule Engine, Jurisdiction Rule Sets, Escalating Alerts, Acknowledgment Tracking |
| Client Portal | Matter Status, Shared Documents, Invoices, Payments |

## 14. Core Data Model

Full ER diagram queued (§ Roadmap). Core entities:

| Entity | Key columns |
|---|---|
| `matters` | id, tenant_id, client_id, matter_number, status, practice_area, opened_at |
| `parties` | id, matter_id, party_type (client/adverse/related), name, entity_type |
| `conflict_checks` | id, intake_id, matched_matter_id, matched_party_id, resolution (cleared/waived/declined) |
| `timekeeper_assignments` | id, matter_id, user_id, role, ethical_wall_flag |
| `time_entries` | id, matter_id, timekeeper_id, minutes, rate, narrative, billed_status |
| `billing_arrangements` | id, matter_id, arrangement_type (hourly/flat/contingency), rate_or_fee |
| `invoices` | id, matter_id, client_id, period_start, period_end, amount, status |
| `trust_ledger_entries` | id, tenant_id, client_id, matter_id, entry_type, amount, posted_at |
| `trust_reconciliations` | id, tenant_id, period, bank_balance, ledger_balance, variance, closed_by |
| `documents` | id, matter_id, title, current_version_id, privilege_status (privileged/shareable) |
| `document_versions` | id, document_id, version_number, storage_key, created_by, created_at |
| `deadlines` | id, matter_id, deadline_type, triggering_event_date, due_date, jurisdiction_rule_id |
| `deadline_alerts` | id, deadline_id, escalation_level, sent_at, acknowledged_at, acknowledged_by |

## 15. Key API Endpoints

Full endpoint catalog queued (§ Roadmap). Key routes, all conforming to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md):

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/intake/conflict-check` | Run firm-wide conflict search |
| POST | `/api/v1/matters` | Open a matter from cleared intake |
| POST | `/api/v1/matters/{matter}/timekeepers` | Assign a timekeeper / set ethical wall |
| POST | `/api/v1/matters/{matter}/time-entries` | Log a time entry |
| GET | `/api/v1/matters/{matter}/pre-bill` | Retrieve pre-bill draft |
| POST | `/api/v1/matters/{matter}/invoices` | Generate an invoice from approved time entries |
| POST | `/api/v1/invoices/{invoice}/payments` | Apply a payment (operating and/or trust drawdown) |
| POST | `/api/v1/matters/{matter}/trust-deposits` | Record a trust retainer deposit |
| GET | `/api/v1/trust-ledger/{client}/balance` | Client trust balance |
| POST | `/api/v1/trust-ledger/reconciliations` | Run/close a monthly trust reconciliation |
| POST | `/api/v1/matters/{matter}/documents` | Upload a document |
| POST | `/api/v1/documents/{document}/versions` | Create a new document version |
| PATCH | `/api/v1/documents/{document}/share-status` | Mark a version shareable to client portal |
| POST | `/api/v1/matters/{matter}/deadlines` | Create a deadline (calculates dependent deadlines) |
| POST | `/api/v1/deadlines/{deadline}/alerts/{alert}/acknowledge` | Acknowledge a deadline alert |
| GET | `/api/v1/portal/client/matters/{matter}` | Client portal matter status view |
| GET | `/api/v1/reports/unbilled-time` | Unbilled time report |
| GET | `/api/v1/reports/trust-variance` | Trust reconciliation variance report |

## 16. Events

`intake.conflict_checked`, `matter.opened`, `time_entry.logged`,
`invoice.generated`, `invoice.paid`, `trust.deposited`, `trust.drawn_down`,
`trust_reconciliation.closed`, `trust_reconciliation.variance_detected`,
`document.version_created`, `document.shared`, `deadline.calculated`,
`deadline.alert_sent`, `deadline.acknowledged`, `deadline.missed`. See
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `intake.conflict_checked` (hit found) | ✔ (to Managing Partner) | ✔ | — | ✔ |
| `deadline.alert_sent` (30/14 day) | ✔ | ✔ | — | — |
| `deadline.alert_sent` (7/1 day, escalated) | ✔ | ✔ | ✔ | ✔ |
| `deadline.missed` (unacknowledged) | ✔ (to supervising partner) | ✔ | ✔ | ✔ |
| `invoice.generated` | ✔ (to client) | ✔ | — | — |
| `trust_reconciliation.variance_detected` | ✔ (to Trust Administrator) | ✔ | — | — |
| `document.shared` | ✔ (to client) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits ZodiCore's default system roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles))
plus ZodiLaw-specific roles: `Managing Partner`, `Attorney`,
`Paralegal/Legal Assistant`, `Billing/Trust Administrator`, `Client`
(portal-only, scoped to own matters). Key permissions: `matters.open`
(requires cleared or waived conflict check), `conflicts.waive` (Managing
Partner only), `trust_ledger.post` (Billing/Trust Administrator only, never
Attorney or Paralegal directly), `trust_reconciliation.close`,
`documents.mark_shareable`, `deadlines.acknowledge`. Matter-level access is
additionally scoped by `PrivilegeSegregationContract` (§11) regardless of
firm-wide role.

## 19. Workflows & Approval Chains

- **Conflict waiver approval**: a conflict hit at intake requires a
  documented Managing Partner decision (waive with informed consent on
  file, or decline the matter) before the matter can open; the decision and
  its basis are permanently attached to the matter record.
- **Pre-bill review and edit approval**: time entries can be adjusted
  (write-down, narrative correction) only during pre-bill review by the
  Billing Administrator or the timekeeper's supervising attorney, with the
  original entry retained in history — never silently overwritten.
- **Trust drawdown authorization**: moving funds from trust to operating
  requires an approved, invoiced amount — the system will not allow a trust
  drawdown that exceeds the specific client's trust balance, enforcing the
  "never use client A's funds for client B's invoice" trust accounting
  rule at the data layer.
- **Deadline escalation chain**: an unacknowledged deadline alert escalates
  from the assigned attorney to the supervising partner automatically at
  the next escalation threshold, per §7 journey 4.

## 20. Audit Logs

Every trust ledger entry, conflict check result and waiver decision,
document version, and deadline acknowledgment is immutably audit-logged per
[audit-logging.md](../../security/audit-logging.md), including the actor
and, for privileged-matter access, a record sufficient to demonstrate
privilege segregation was maintained.

## 21. Reports & Analytics & Dashboards

- Unbilled/unrecorded time and realization-rate reports.
- Trust reconciliation and variance reporting.
- Matter profitability by billing arrangement.
- Upcoming and missed-deadline dashboards, firm-wide and per attorney.
- Conflict check audit history for bar compliance review.
- Report builder and scheduled reports per the
  [Second-Layer Feature Catalog](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Court e-filing systems**: jurisdiction e-filing integration for
  applicable practice areas (e.g. PACER-adjacent federal court systems,
  state e-filing portals) to auto-populate filing-triggered deadlines.
- **Payment processing**: invoice and trust deposit processing via
  ZodiCore's payment gateway abstraction, with trust deposits routed to a
  segregated trust bank account.
- **Accounting export**: QuickBooks-compatible export for firm operating
  books maintained outside ZodiLaw's trust ledger.
- **Legal research platforms**: outbound linking/citation integration with
  legal research providers (Westlaw/Lexis-class) from matter and document
  records.
- **E-signature**: for engagement letters and client-facing documents.

## 23. AI Features

- AI-assisted deadline extraction: reads uploaded court documents (e.g. a
  scheduling order) and proposes triggering-event dates and calculated
  deadlines for attorney confirmation before they become binding calendar
  entries.
- AI-assisted document drafting from matter templates, with all output
  requiring attorney review before client-portal sharing.
- AI-assisted time entry narrative suggestions from calendar/document
  activity, with the timekeeper confirming before the entry is billable.
- Anomaly detection on trust ledger activity, extending ZodiCore's audit
  anomaly detection (§23 of [ZodiCore's SPEC.md](../ZodiCore/SPEC.md#23-ai-features)).

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: deadline alert escalation dispatch, monthly trust
  reconciliation run, invoice generation batch, unbilled-time reminder
  digest, engagement/matter statute-of-limitations sweep.
- CLI commands: `law:calculate-deadlines`, `law:run-trust-reconciliation`,
  `law:generate-invoices`, `law:sweep-sol-alerts` — each requiring the same
  authorization context as its API equivalent.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a demo firm with 3 attorneys and 2 paralegals across
20 demo matters spanning litigation, transactional, and contingency
arrangements; 12 months of time entry and invoicing history; a populated
trust ledger with at least one full deposit-to-drawdown-to-reconciliation
cycle; a document library with multi-version history; and a deadline
calendar including one escalated-and-acknowledged alert and one
demonstration of the conflict-check waiver flow — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10. Additionally: deadline calculation for a new triggering event
(computing all dependent deadlines under the applicable jurisdiction rule
set) completes in under 2 seconds p95.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiLaw-specific requirements:

- **Attorney-client privilege data segregation**: matter access is scoped
  to assigned timekeepers via `PrivilegeSegregationContract` (§11); a
  firm-wide role does not imply matter access, and ethical-wall flags block
  specific timekeepers from a matter even if their general role would
  otherwise permit it. Privileged documents are never exposed to the client
  portal unless explicitly marked shareable (§9).
- **Trust accounting compliance**: the trust ledger is modeled and
  reconciled fully segregated from the operating ledger at the data-model
  level, matching IOLTA/bar trust-accounting rules that prohibit
  commingling; a drawdown exceeding a specific client's trust balance is
  rejected at the data layer, not merely flagged.
- **Conflict-check completeness**: conflict search covers parties, related
  entities, and — where captured — beneficial-owner/affiliate fields across
  every matter in the firm's history, since an incomplete search is itself
  a professional-responsibility risk.
- **Client data confidentiality**: client PII and matter content are
  encrypted at rest per
  [data-protection-privacy.md](../../security/data-protection-privacy.md);
  cross-tenant isolation is tested per ZodiCore's cross-tenant isolation
  suite, critical here because opposing firms may both be Zodize tenants.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md);
additionally a dedicated trust ledger integrity suite (mirroring
ZodiEstate's, adapted for client/matter segregation rather than
deposit/owner segregation) and a privilege-segregation access-control suite
verifying no code path can return a matter's data to a non-assigned,
non-walled-in timekeeper, both run as required CI gates.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).

## 30. Acceptance Criteria

- A matter cannot be opened without a completed conflict check that is
  either clear or has a documented waiver/decline decision attached.
- Time entries flow from entry through pre-bill review to an invoice with a
  full, unaltered history of any pre-bill adjustments.
- A trust drawdown can never exceed the specific client's trust balance,
  and monthly reconciliation correctly identifies any variance between
  ledger and bank balances.
- A deadline's escalating alerts fire on schedule and an unacknowledged
  alert correctly escalates to the supervising partner.
- A timekeeper not assigned to a matter (or subject to its ethical wall)
  cannot access that matter's documents or time entries through any API
  path.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).

## 32. Future Roadmap

- Jurisdiction rule-set marketplace/expansion beyond the initial covered
  jurisdictions.
- Contingency-fee matter cost-advance tracking module.
- Court e-filing status tracking integrated directly into the deadline
  calendar.

## 33. Known Risks

- Deadline-calculation defects carry direct malpractice exposure for
  tenants — mitigated by the escalating-alert design and acknowledgment
  tracking in §7/§19, but jurisdiction rule-set accuracy requires ongoing
  legal-content maintenance, not just software correctness.
- Trust accounting defects carry bar-discipline and disbarment exposure for
  tenants — mitigated by the dedicated integrity test suite in §28, but
  this remains the module requiring the highest engineering scrutiny in the
  product, matching ZodiEstate's equivalent risk posture.

## 34. Future Improvements

- Configurable jurisdiction rule-set builder for firms practicing in
  jurisdictions outside the initial covered set.
- Contingency-fee settlement disbursement statement generation.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), full endpoint
catalog (companion `API_REFERENCE.md`), full jurisdiction deadline rule-set
library, and a complete report catalog beyond the summary list in §21.
