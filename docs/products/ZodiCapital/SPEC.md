# ZodiCapital — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

Unlike most other products in the catalog, ZodiCapital is **not** built by
cloning the sanitized [base codebase](../../architecture/base-codebase-strategy.md)
(the qfsfountains-sourced "ViserBank/ViserLab Core Engine") through the
standard [genericization checklist](../../architecture/product-genericization-checklist.md)
pipeline. A direct filesystem audit of the build server confirmed an
existing Laravel codebase at `/home/novavest/public_html/core/` — complete
with `app/`, `artisan`, `composer.json`, `bootstrap/`, `config/`,
`database/`, `resources/`, `routes/`, `storage/`, `vendor/`, and a
`.env`/`.env.example` — sitting alongside a `/home/novavest/public_html/assets/`
directory. That `assets/` + `core/` split is the same structural convention
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#directory-structure)
documents for the qfsfountains base, a strong signal this "novavest"
codebase is itself Zodize-lineage rather than an unrelated third codebase,
even though its `composer.json` still carries the generic `laravel/laravel`
boilerplate package name and its `README.md` is unmodified Laravel
boilerplate — neither file confirms a specific product identity on its own.
ZodiCapital is therefore based on and improved from novavest/core, similar
in spirit to how ZodiBank is built on "Pay Secure" instead of the
qfsfountains base (see [ZodiBank's SPEC.md §11](../ZodiBank/SPEC.md#11-architecture))
— a deliberate, documented exception to the standard "every product clones
qfsfountains" pipeline, not a contradiction of it. Unlike Pay Secure,
however, novavest is not a shipped/sold commercial product; it is an
existing internal build, so this is more accurately "extend an existing
internal codebase" than "Live — Extend Only" in the sense
[BUILD_STATE.md](../../../BUILD_STATE.md) uses for ZodiTrack. See
[Open Questions](#open-questions) for what a follow-up session must still
audit before building on top of it. ZodiCapital does not depend on any
other Zodize product or on a central "ZodiCore" platform for identity,
billing, notifications, or tenancy — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`ZodiCore` is itself just another standalone product in the catalog (a
general-purpose back-office/ERP starter), not a platform ZodiCapital runs
on.

**Shared foundation with ZodiYield**: [ZodiYield](../ZodiYield/SPEC.md) is
also based on and improved from the same novavest/core codebase. A session
working on either product's novavest-derived infrastructure (RBAC, KYC,
wallet/ledger, or any shared scaffolding novavest already provides) MUST
check [ZodiYield's SPEC.md §11](../ZodiYield/SPEC.md#11-architecture) and
this document's own audit findings first, so the two products don't
independently reinvent overlapping novavest infrastructure.

## 1. Vision

ZodiCapital is fund and investment portfolio management for private equity,
venture capital, and real asset fund managers who need real LP/GP fund
mechanics — capital calls, distributions, NAV, and IRR — not a generic CRM
repurposed as a fund back office. It gives a fund manager the fund
administration and investor relations infrastructure to run a fund's full
lifecycle from formation to wind-down on one platform.

## 2. Purpose

Fund managers today stitch together a fund administrator's portal, a
separate investor-relations CRM, a spreadsheet-based capital-call and
distribution waterfall, and a manually assembled quarterly reporting
package. ZodiCapital exists to give a fund manager one coherent, self-hosted
system — fund structure, capital accounts, calls/distributions, NAV,
performance reporting, and an investor portal — built on a base codebase
whose RBAC, KYC, and audit engine already work, so LPs get a real
self-service portal and the GP gets a real system of record instead of a
reconciled spreadsheet.

## 3. Target Market

Private equity and venture capital fund managers, real asset (real estate,
infrastructure) fund sponsors, fund-of-funds managers, and family offices
running SPV or co-investment vehicles. Buyers are typically a Fund
Controller/CFO, Head of Investor Relations, or a fund manager's COO
evaluating a build-vs-buy decision against a fund administrator's portal or
an in-house spreadsheet process.

## 4. Industries

Asset management, private equity, venture capital, real assets/real estate
funds.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Fund administration platform | Carta (fund admin), Allvue Systems, eFront | Ships with RBAC/audit already built into the inherited base codebase, faster to stand up compliant fund operations |
| Investor portal/CRM | Juniper Square, Dynamo Software | Investor portal shares one identity and permission model with fund accounting, not a separate synced system |
| Capital call/distribution workflow | Anduin, Carta subscription workflows | Capital calls, distributions, and NAV share one ledger of record with the investor portal |
| Performance reporting (IRR/MOIC) | PitchBook fund reporting tools, Preqin-adjacent analytics | Performance metrics computed directly from the cash-flow ledger, not a reconciled downstream export |
| Subscription/e-signature workflow | DocuSign + Carta workflow combination, AngelList fund tooling | E-signature and accredited-investor verification integrated into one onboarding flow, not stitched across vendors |

## 6. Personas

- **Fund Controller/CFO** — manages fund accounting, capital calls,
  distributions, and NAV calculation.
- **Head of Investor Relations** — manages LP communications, the investor
  portal, and subscription onboarding.
- **General Partner (GP)/Fund Manager** — reviews fund performance,
  approves capital calls and distributions.
- **Limited Partner (LP)/Investor** — views capital account statements,
  performance reporting, and responds to capital calls via the investor
  portal.
- **Compliance Officer** — manages accredited-investor verification and
  regulatory filings.
- **Buyer's own IT/support staff** — the only support layer this deployment
  has; there is no Zodize-operated support console, since each deployment is
  the buyer's own standalone codebase (see
  [admin-template.md](../../templates/admin-template.md)).

## 7. User Journeys

1. **Fund formation and investor onboarding**: GP creates a fund structure
   (LP/GP entity, commitment terms, fee/carry structure) → prospective LP
   is invited to the investor portal → LP completes subscription documents
   and accredited-investor verification (see §21) → e-signature workflow
   completes → LP's capital commitment is recorded and a capital account is
   created.
2. **Capital call**: Fund Controller initiates a capital call for a
   percentage of committed capital → the system calculates each LP's pro-
   rata call amount based on commitment percentage → GP approves the call
   → call notices are sent to LPs via the investor portal and notification
   channels → LP wire instructions and due dates are tracked until funded,
   with unfunded calls flagged for follow-up.
3. **Distribution and waterfall**: fund realizes an exit → Fund Controller
   models the distribution through the fund's configured waterfall (return
   of capital, preferred return, GP catch-up, carried interest split) →
   GP approves the distribution → per-LP distribution amounts are
   calculated and disbursed → each LP's capital account and cumulative
   distribution history update.
4. **Quarterly NAV and performance reporting**: Fund Controller runs the
   quarterly NAV calculation based on portfolio company valuations and cash
   position → IRR and MOIC are recalculated at the fund and per-LP level →
   the quarterly report package is generated and published to the investor
   portal, with LPs notified their statement is available.
5. **Accredited-investor re-verification**: as part of periodic compliance
   review, Compliance Officer triggers re-verification for LPs whose
   accreditation documentation is approaching expiry → LPs are prompted via
   the portal to re-submit → non-response beyond a configured grace period
   flags the LP's account for GP review.

## 8. Business Goals

- Let a fund manager run capital calls, distributions, and NAV reporting on
  one system of record instead of reconciling spreadsheets against a fund
  administrator's exports.
- Give LPs a real self-service portal for capital account statements and
  performance reporting, reducing IR team email volume.
- Reduce time-to-close for new fund formations by integrating subscription
  documents, e-signature, and accredited-investor verification into one
  onboarding flow.

## 9. Functional Requirements

- Fund structures: LP/GP entity modeling, commitment tracking, fee
  structure (management fee, hurdle rate) and carried-interest/waterfall
  configuration, support for multiple funds and co-investment vehicles
  within one deployment.
- Capital calls: pro-rata call calculation, call notice generation and
  delivery, funding status tracking, late-payment handling.
- Distributions: waterfall modeling (return of capital, preferred return,
  GP catch-up, carry split), per-LP distribution calculation and
  disbursement tracking.
- NAV calculation: portfolio company/asset valuation input, quarterly (or
  configurable cadence) NAV computation at fund and per-LP capital-account
  level.
- Investor portal: LP self-service access to capital account statements,
  documents, performance reporting, and capital call/distribution history.
- Performance reporting: IRR and MOIC computed at fund and per-LP level,
  both gross and net of fees/carry.
- Compliance/accredited-investor verification: verification workflow at
  subscription and periodic re-verification, with expiry tracking.
- Subscription documents/e-signature workflow: templated subscription
  agreement generation, e-signature capture, and document archival.
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (capital call/distribution approval), rule engine
  (re-verification scheduling), saved filters and global search over
  LPs/funds, custom fields on fund/LP records, full audit history, soft
  delete with restore on non-ledger entities, mass actions (bulk call
  notice send), import/export wizards for LP and commitment data, command
  palette, scheduled/report-builder reporting, system health dashboard,
  white-labeling of the investor portal.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiCapital-specific additions:

- Capital call/distribution calculation for a fund with up to 500 LPs
  completes in under 10 seconds, since GPs frequently run these
  calculations interactively during approval review.
- Investor portal document/statement load targets p95 < 1s, since LP trust
  in the platform is heavily shaped by portal responsiveness during
  reporting periods.
- 99.9% uptime target for the investor portal, matching the general
  product baseline (fund operations are not real-time-critical the way
  banking/trading are).

## 11. Architecture

ZodiCapital is based on and improved from the existing "novavest" Laravel
application at `/home/novavest/public_html/core/`, a single, independent
codebase the buyer deploys entirely on their own shared/VPS hosting, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)
— **not** a clone of the sanitized
[base codebase](../../architecture/base-codebase-strategy.md) run through
the [genericization checklist](../../architecture/product-genericization-checklist.md),
as most other products are. This was confirmed by a direct filesystem audit
of the build server: novavest/core has the standard Laravel skeleton
(`app/`, `artisan`, `composer.json`, `bootstrap/`, `config/`, `database/`,
`resources/`, `routes/`, `storage/`, `vendor/`) and a sibling
`novavest/assets/` directory, matching the same `assets/` + `core/` split
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#directory-structure)
documents for the qfsfountains base — meaning novavest is very likely
Zodize-lineage itself, not a from-scratch third codebase, even though
neither its `README.md` (unmodified Laravel boilerplate) nor its
`composer.json` (generic `laravel/laravel` package name) confirms a
specific prior product identity.

**What the audit has and has not established.** The audit confirmed
novavest/core's directory shape and Laravel/framework identity. It did
**not** deeply inspect `novavest/core/app/` or
`novavest/core/database/migrations/` against ZodiCapital's own fund-
management requirements (§9, §14) — whether novavest already has RBAC/auth
(`Role`/`Permission` models), KYC, wallet/ledger, or any fund-structure,
capital-call, distribution, NAV, or investor-portal functionality is
unknown until that audit runs. See [Open Questions](#open-questions). Do
**not** assume novavest lacks these systems and rebuild them from the
qfsfountains base's inherited-engine list — that would risk building
duplicate infrastructure on top of a codebase that may already have its
own. Equally, do not assume novavest already covers ZodiCapital's fund
mechanics without checking; the correct next step is a concrete gap list
(spec requirement → present/absent in novavest), the same audit-before-build
approach [BUILD_STATE.md](../../../BUILD_STATE.md) documents for ZodiTrack's
Live-Extend-Only status, adapted here for an existing internal codebase
rather than an already-shipped product.

Once that gap list exists, ZodiCapital's own domain modules — Fund
Structures, Capital Calls, Distributions, NAV & Valuation, Investor Portal,
Performance Reporting, Compliance, and Subscription & E-Signature — are
added as new, clearly bounded modules per
[module-template.md](../../templates/module-template.md), reusing whatever
RBAC/audit/KYC/wallet infrastructure the audit finds already present in
novavest rather than building parallel systems, and building fresh only
what the gap list shows is actually missing. The investor portal is a
restricted-scope view within the same single-business deployment the GP
manages, scoping each LP to only their own capital account and fund-level
public documents, never another LP's data, via whichever RBAC mechanism the
audit confirms novavest provides — this is ordinary row-level-authorization,
not tenant isolation, since there is exactly one fund manager's deployment.
ZodiCapital has no runtime dependency on any other Zodize product or on a
Zodize-operated central service; the only external dependencies are the
third-party e-signature, accreditation verification, and valuation-data
integrations the buyer's own fund manager configures (§22).

**Shared foundation with ZodiYield**: both ZodiCapital and
[ZodiYield](../ZodiYield/SPEC.md) are based on and improved from this same
novavest/core codebase. A future session auditing novavest's `app/` or
`database/migrations/` for one product SHOULD share findings with the
other's SPEC.md rather than re-running the same filesystem audit twice.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL for fund/capital-account/waterfall records + Redis for
performance-metric caching per
[database-standards.md](../../development/database-standards.md);
e-signature and accredited-investor verification integrated as
provider-abstracted services (see §22).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Fund Structures | Entity Modeling, Commitment Tracking, Fee/Carry Configuration |
| Capital Calls | Call Calculation, Notice Generation, Funding Status Tracking |
| Distributions | Waterfall Modeling, Per-LP Calculation, Disbursement Tracking |
| NAV & Valuation | Portfolio Valuation Input, NAV Calculation, Capital Account Rollforward |
| Investor Portal | Statement Access, Document Library, Performance Reporting View |
| Performance Reporting | IRR/MOIC Calculation, Fund/LP-Level Analytics, Benchmark Comparison |
| Compliance | Accredited-Investor Verification, Re-Verification Scheduling, Filing Tracking |
| Subscription & E-Signature | Document Templates, E-Signature Capture, Document Archival |

## 14. Core Data Model

The 11 entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)). There is no `tenant_id`
column anywhere in this model — each deployed instance belongs to exactly
one fund manager, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).
A fund manager running multiple funds within their one deployment models
that as multiple `funds` rows scoped by `fund_id` throughout, not as
tenancy.

**These entities describe what ZodiCapital needs, not what novavest already
has.** They were designed against ZodiCapital's own functional requirements
(§9), not against novavest's actual existing schema — novavest's
`database/migrations/` has not yet been audited (see
[Open Questions](#open-questions)). Before finalizing any new migration for
the tables below, a session with real access to
`novavest/core/database/migrations/` MUST check for overlap: novavest may
already have equivalent (or partially equivalent) fund, investor, capital-
account, or ledger tables under different names, in which case the correct
move is to extend or rename what exists rather than create a duplicate
table alongside it. Treat every table name below as a proposal pending that
audit, not as confirmed net-new schema.

| Entity | Key columns |
|---|---|
| `funds` | id, name, vintage_year, target_size, base_currency, fee_structure_json, waterfall_config_json |
| `investors` | id, legal_name, investor_type, accreditation_status, accreditation_expires_at |
| `commitments` | id, fund_id, investor_id, committed_amount, commitment_date, status |
| `capital_accounts` | id, commitment_id, called_to_date, distributed_to_date, current_nav, updated_at |
| `capital_calls` | id, fund_id, call_number, total_call_amount, due_date, status, approved_by |
| `capital_call_allocations` | id, capital_call_id, commitment_id, amount_due, amount_funded, funded_at |
| `distributions` | id, fund_id, distribution_number, total_amount, waterfall_tier, approved_by |
| `distribution_allocations` | id, distribution_id, commitment_id, amount, disbursed_at |
| `nav_valuations` | id, fund_id, valuation_date, nav_amount, methodology, approved_by |
| `subscription_documents` | id, commitment_id, document_id, esignature_status, signed_at |
| `performance_snapshots` | id, fund_id, as_of_date, irr, moic, dpi, tvpi |

**Multi-currency**: ZodiCapital operates single-base-currency by default —
each `funds` row carries its own `base_currency`, and every call/
distribution/NAV amount for that fund is recorded in that fund's base
currency, per
[ADR-0002](../../decisions/0002-single-currency-wallet-by-default.md). This
covers the common case of a single-currency fund calling capital from and
distributing to LPs in one currency. A fund manager whose LP base commits
and is called in multiple currencies (a genuinely global LP base, tracked
as future work in §32) MUST extend the pattern explicitly per
[wallet-system.md's Multi-currency gap](../../standards/wallet-system.md#multi-currency-gap)
rather than the base engine assuming it: a `capital_account_balances` table
(`id`, `capital_account_id`, `currency`, `called_to_date`,
`distributed_to_date`, `updated_at`) scoped per `capital_account_id`, with
its own append-only `capital_account_ledger_entries` table
(`id`, `capital_account_balance_id`, `amount` (signed), `currency`,
`trigger` (capital_call/distribution/fx_revaluation), `post_balance_snapshot`,
`reference_id`, `created_at`) mirroring the inherited `Transaction` model's
invariants.

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/funds` | Create a fund structure |
| POST | `/api/v1/funds/{id}/commitments` | Record an investor commitment |
| GET | `/api/v1/investors/{id}/capital-account` | Fetch an investor's capital account |
| POST | `/api/v1/funds/{id}/capital-calls` | Initiate a capital call |
| POST | `/api/v1/capital-calls/{id}/approve` | Approve a pending capital call |
| POST | `/api/v1/capital-calls/{id}/allocations/{allocation_id}/mark-funded` | Record LP funding |
| POST | `/api/v1/funds/{id}/distributions` | Model and initiate a distribution |
| POST | `/api/v1/distributions/{id}/approve` | Approve a pending distribution |
| POST | `/api/v1/funds/{id}/nav-valuations` | Record a NAV valuation |
| GET | `/api/v1/funds/{id}/performance` | Fetch fund-level IRR/MOIC/DPI/TVPI |
| GET | `/api/v1/investors/{id}/performance` | Fetch per-LP performance metrics |
| POST | `/api/v1/investors/{id}/verify-accreditation` | Trigger accreditation verification |
| GET | `/api/v1/investors/{id}/documents` | List an LP's documents |
| POST | `/api/v1/subscription-documents/{id}/send-for-signature` | Send subscription doc for e-signature |
| GET | `/api/v1/subscription-documents/{id}/status` | Check e-signature status |
| GET | `/api/v1/funds/{id}/capital-calls` | List capital call history |
| GET | `/api/v1/funds/{id}/distributions` | List distribution history |
| GET | `/api/v1/reports/quarterly-package/{fund_id}` | Generate quarterly report package |
| GET | `/api/v1/portal/statements` | LP-scoped: fetch own capital account statements |
| GET | `/api/v1/portal/documents` | LP-scoped: fetch own document library |

## 16. Events

Domain events registered on the inherited event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`fund.created`, `commitment.recorded`, `capital_call.initiated`,
`capital_call.approved`, `capital_call.funded`, `capital_call.overdue`,
`distribution.initiated`, `distribution.approved`, `distribution.disbursed`,
`nav_valuation.recorded`, `accreditation.verified`,
`accreditation.expiring`, `accreditation.expired`,
`subscription_document.signed`, `quarterly_package.published`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `capital_call.initiated` | ✔ (to LP) | ✔ (call notice) | — | — |
| `capital_call.overdue` | ✔ (to GP/IR) | ✔ (to LP, reminder) | — | — |
| `distribution.disbursed` | ✔ | ✔ (disbursement notice) | — | — |
| `accreditation.expiring` | ✔ | ✔ | — | — |
| `subscription_document.signed` | ✔ | ✔ (confirmation) | — | — |
| `quarterly_package.published` | ✔ (to LP) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine per
[admin-template.md](../../templates/admin-template.md), with fund-specific
roles registered on top of the
[default system roles](../../security/rbac-permissions.md#default-system-roles):
`Fund Controller/CFO`, `Investor Relations`,
`General Partner`, `Compliance Officer`, `Limited Partner` (portal-scoped
external role). Key permissions: `funds.manage`, `capital_calls.initiate`,
`capital_calls.approve`, `distributions.initiate`, `distributions.approve`,
`nav.record`, `accreditation.verify`, `portal.view_own_account` (LP role
default, cannot be escalated to view other LPs' data). Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **Capital call approval**: a capital call calculated by Fund Controller
  requires General Partner approval before notices send to LPs; the
  approver must be distinct from the initiator for funds above an
  admin-configured size threshold.
- **Distribution approval**: a modeled distribution waterfall requires
  General Partner approval before disbursement, with the full waterfall
  calculation shown for review, matching
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **NAV valuation approval**: a recorded NAV valuation requires Fund
  Controller sign-off before it becomes the fund's official NAV for
  performance-reporting purposes; prior valuations are retained, never
  overwritten.
- **Accreditation re-verification escalation**: an LP who does not
  re-verify within the grace period is flagged for GP review, which may
  result in a hold on future capital calls to that LP pending resolution.

## 20. Audit Logs

Every capital call, distribution, NAV valuation, accreditation decision,
and subscription document signature writes an immutable audit entry via
the inherited audit log ([audit-logging.md](../../security/audit-logging.md)),
capturing actor, timestamp, and before/after state. Capital account
balances are computed from the append-only call/distribution/valuation
ledger rather than stored as a mutable running total, so the full history
is always reconstructible and auditable.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: fund AUM, capital called-to-date vs. committed,
  distribution-to-date, open capital calls, accreditation status summary —
  per [dashboard-standards.md](../../standards/dashboard-standards.md).
- Performance: fund and per-LP IRR/MOIC/DPI/TVPI, benchmark comparison
  against vintage-year peer data where available.
- Investor relations: quarterly report package, capital call funding-rate
  report, LP communication history.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **E-signature providers**: subscription document signature workflow
  integration (e.g. DocuSign/Adobe Sign-class vendors) behind an
  `ESignatureProviderContract`.
- **Accredited-investor verification**: third-party verification services
  (income/net-worth/professional-certification verification vendors)
  behind an `AccreditationVerificationContract`.
- **Fund administration/accounting export**: optional export connectors for
  funds that retain a third-party fund administrator alongside ZodiCapital
  during a transition period.
- **Portfolio valuation data**: integration category for pulling comparable
  valuation data (e.g. PitchBook/Preqin-class data vendors) to support NAV
  and benchmark reporting.
- **Banking/payment rails**: wire transfer initiation for capital call
  funding and distribution disbursement, via the same inherited
  payment-gateway/withdrawal-method integration category described in
  [payment-gateways.md](../../standards/payment-gateways.md).

## 23. AI Features

- Waterfall scenario modeling assistant: given a hypothetical exit value,
  generates a plain-language summary of how the distribution waterfall
  would allocate proceeds across return-of-capital, preferred return, and
  carry, before the GP commits to an actual distribution.
- Quarterly report drafting assistant: generates a first-draft narrative
  summary of fund performance and portfolio company highlights for Fund
  Controller/IR review and edit, never auto-published without human
  approval.
- Anomaly detection on capital account activity, layered on top of the
  inherited audit log's anomaly detection, tuned for unusual
  call/distribution timing patterns.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: capital call overdue-status checks, accreditation
  expiry reminders, quarterly report package generation trigger,
  performance-snapshot recalculation on new NAV valuation.
- CLI commands (Artisan): `capital:calculate-call`,
  `capital:calculate-distribution`, `capital:recalculate-performance`,
  `capital:check-accreditation-expiry` — each requires the same
  authorization context as its API equivalent.

## 25. Seed/Demo Data

`CapitalDemoSeeder` provisions the demo deployment with two or three demo funds
across different vintage years, 30+ synthetic LPs with varied commitment
sizes and accreditation states, a full capital call and distribution
history spanning several years, quarterly NAV valuation history, and
signed subscription documents — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: IRR/MOIC recalculation across a fund's full
cash-flow history completes in under 5 seconds p95 even for funds with 10+
years of call/distribution history.

## 27. Security Requirements

Financial products carry Zodize's highest security/compliance bar. Full
baseline from [security-standards.md](../../security/security-standards.md)
applies, plus:

- **PCI-DSS-equivalent handling** for any stored payment/wire instruction
  data used in capital call funding, tokenized/masked and never logged in
  plaintext.
- **SOC2-equivalent controls**: change management and access review apply
  to fund accounting and investor portal modules with the same rigor as
  the inherited base engine.
- **KYC/AML and accredited-investor verification** required before a
  commitment can be recorded as active, with periodic re-verification
  enforced by scheduled job per §24, not manual process.
- **Immutable audit trails**: capital call, distribution, and NAV audit
  entries are append-only, matching §20.
- **MFA is mandatory, not optional**, for every internal user role
  (Fund Controller, Investor Relations, General Partner, Compliance) and
  strongly enforced for LP portal access given the sensitivity of capital
  account data, per
  [authentication-authorization.md](../../security/authentication-authorization.md).
- LP data isolation: an LP's portal session can never query another LP's
  capital account, enforced at the RBAC policy layer per §11 (ordinary
  row-level authorization within the one deployment, not tenant isolation),
  with a dedicated cross-LP isolation test per §28.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated waterfall-calculation regression suite validated against known
reference distribution scenarios, and a cross-LP data isolation test suite
asserting no LP portal session can read another LP's capital account or
documents.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Capital call and distribution calculation changes require a documented
finance-logic review in addition to the standard PR review per
[pr-standards.md](../../development/pr-standards.md), given the direct
financial impact of a calculation error.

## 30. Acceptance Criteria

- A fund can be created, an LP onboarded through subscription and
  accreditation verification, and a capital commitment recorded end-to-end
  with no manual data entry outside the platform.
- A capital call correctly calculates each LP's pro-rata share, requires
  GP approval, and tracks funding status to completion.
- A distribution correctly allocates proceeds through the fund's
  configured waterfall and updates each LP's capital account and
  cumulative distribution history.
- IRR and MOIC calculations match reference values for a defined regression
  fixture set at both fund and per-LP level.
- An LP portal user can view only their own capital account, documents,
  and performance data, verified by an automated isolation test.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiCapital additionally requires sign-off from a compliance stakeholder
that accredited-investor verification and subscription document workflows
have been validated against the fund's actual regulatory jurisdiction
before go-live.

## 32. Future Roadmap

- Secondary market/LP transfer support (recording and processing an LP
  interest transfer mid-fund).
- Multi-currency fund support for global LP bases.
- Co-investment vehicle and SPV-specific waterfall templates.

## 33. Known Risks

- Waterfall calculation complexity: fund waterfalls vary significantly by
  fund and can include multiple tiers, catch-up provisions, and clawback
  mechanics — mitigated by the configurable `waterfall_config_json` model
  and regression test suite, but remains the module's highest-complexity
  surface.
- Valuation subjectivity: NAV for illiquid private-fund assets depends on
  GP-provided valuations rather than a market price feed; ZodiCapital
  records and audits the valuation but does not independently verify its
  accuracy.

## 34. Future Improvements

- Configurable clawback provision modeling in the distribution waterfall.
- Automated benchmark comparison against third-party vintage-year
  performance data feeds.

## Open Questions

- **Novavest's exact existing feature set needs a deeper audit.** The
  filesystem audit backing §11 confirmed `/home/novavest/public_html/core/`
  is a Laravel application with the standard `assets/` + `core/` structural
  split, but it did not inspect `novavest/core/app/` or
  `novavest/core/database/migrations/` in depth. A follow-up session MUST
  audit both against this SPEC.md's requirements (§9 Functional
  Requirements, §14 Core Data Model) — fund structures, capital calls,
  distributions, NAV calculation, investor portal, RBAC/auth, KYC, and
  wallet/ledger — and produce a concrete gap list (requirement → present/
  absent/partial in novavest) before any new migration or module is built.
  Until that audit runs, treat every "novavest already has X" or "ZodiCapital
  must build Y fresh" statement in this document as unverified against the
  real codebase, not as a completed audit result.
- **Overlap with ZodiYield's own novavest audit.** Because
  [ZodiYield](../ZodiYield/SPEC.md) shares the same novavest/core
  foundation, whichever session performs the novavest audit first SHOULD
  record shared findings (RBAC, KYC, wallet/ledger, or any other
  cross-cutting infrastructure novavest already provides) somewhere both
  specs can reference, so the second product's audit doesn't repeat work
  already done for the first.

## Roadmap (spec depth)

This spec's Architecture and Core Data Model sections were revised to
reflect the corrected standalone, self-hosted, single-tenant deployment
model — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)
and [base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
— and subsequently revised again to reflect the verified ground truth that
ZodiCapital is based on and improved from the existing novavest/core
codebase (see §11) rather than cloned from the sanitized qfsfountains base,
a deliberate exception to the standard pipeline documented alongside
ZodiBank's equivalent "Pay Secure" exception. This spec is Foundation-depth.
Queued for Deep-depth expansion, now gated on the novavest audit in
[Open Questions](#open-questions): a full ER diagram and migration set for
the fund/capital-account/waterfall schema reconciled against whatever
novavest already has (companion `DATA_MODEL.md`), a complete endpoint
catalog including the full investor portal surface (companion
`API_REFERENCE.md`), and a full report catalog covering additional
fund-type-specific reporting (real assets, fund-of-funds). Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).
