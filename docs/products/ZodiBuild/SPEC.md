# ZodiBuild — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; deep artifacts (full ER diagram, exhaustive endpoint
> catalog, full report catalog) are queued — see
> [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiBuild is the system of record for a construction general contractor or
owner's-rep firm running projects from preconstruction budget through
closeout — phases and budget-vs-actuals, subcontractor management, the
RFI/submittal exchange, daily site logs, change orders, punch lists, safety
incidents, and drawing/spec version control — built as a standalone,
self-hosted Laravel application from the shared Zodize base codebase
([base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)),
so a construction firm gets a working admin back office, RBAC, and
multi-project data scoping without a separate construction-tech vendor
stack.

## 2. Purpose

Mid-size general contractors are often priced out of Procore's enterprise
contract tier or outgrow lightweight tools like Buildertrend once they run
multiple concurrent commercial projects with real subcontractor volume and
document-control requirements. ZodiBuild exists to give a GC running
5–50 concurrent projects a single platform for project financials,
field documentation, and document control with the audit rigor an
enterprise owner or bonding company expects, without Procore-scale
implementation cost.

## 3. Target Market

Mid-size general contractors, construction management firms, and
owner's-representative firms running multiple concurrent commercial,
industrial, or multi-family residential projects.

## 4. Industries

Construction — commercial general contracting, construction management,
industrial/heavy civil, and multi-family/residential development.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Construction project management | Procore | Modern UI, transparent pricing, no enterprise-only feature gating |
| Field-first project management | Buildertrend | Scales to multi-project commercial GC operations, not residential-remodel-first |
| RFI/submittal workflow | PlanGrid (Autodesk Build) | Unified with budget/change-order tracking, not a documents-only tool |
| Document control | Newforma | Enterprise RBAC, audit trail, and multi-project data scoping inherited from the base engine from day one |
| Safety/incident tracking | SafetyCulture (iAuditor) | Native to the same project record as RFIs and daily logs, not a bolt-on inspection app |

## 6. Personas

- **Project Executive/Owner** — oversees portfolio-level project health,
  budget variance, and risk across all active projects.
- **Project Manager** — manages a project's schedule, budget, RFIs,
  submittals, and change orders day to day.
- **Superintendent** — runs the jobsite, logs daily reports, manages punch
  lists and safety on site.
- **Subcontractor** — receives RFIs/submittals, logs their own daily
  work, and is tracked for compliance (insurance, licensing).
- **Architect/Engineer of Record** — responds to RFIs, reviews and
  approves/rejects submittals.
- **Owner's Representative/Client** — reviews budget-vs-actuals, approves
  change orders, monitors project progress via read-focused access.
- **Safety Officer** — logs and tracks safety incidents and site
  inspections across projects.

## 7. User Journeys

1. **Project setup to phase kickoff**: Project Manager creates a project
   with phases (e.g. sitework, foundation, structure, finishes) and an
   original budget broken out by cost code → subcontractors are added to
   the project with their contract value and insurance/licensing
   compliance documents → project moves to active status and appears on
   the Superintendent's site dashboard.
2. **RFI to resolution**: Superintendent identifies a design ambiguity in
   the field → submits an RFI referencing the relevant drawing → RFI
   routes to the Architect of Record with a due date → Architect responds
   → response is logged against the RFI and linked drawing, and if the
   response implies a cost or schedule impact, the Project Manager
   converts it into a change order draft.
3. **Submittal review cycle**: Subcontractor submits a product submittal
   (e.g. structural steel shop drawings) → routes to the Architect for
   review → Architect marks it "Revise and Resubmit" with markup comments
   → Subcontractor resubmits a revised version → Architect approves →
   the approved version becomes the current record version, with the full
   revision history retained per
   [migration-seeder-standards.md](../../development/migration-seeder-standards.md).
4. **Change order approval chain**: a change order is drafted from an
   approved RFI cost impact → Project Manager prices it and submits it for
   internal approval → once internally approved, it routes to the Owner's
   Representative for client approval → upon client approval, the change
   order updates the project's approved budget and the daily budget-vs-
   actuals report reflects the revised contract value.
5. **Daily site log with photos**: Superintendent logs a daily report at
   end of shift — weather, crew counts by subcontractor, work performed,
   equipment on site, and photos of progress and any issues → the log
   becomes part of the project's permanent record, referenceable in any
   future delay-claim or dispute documentation.
6. **Punch list to closeout**: near substantial completion, the
   Superintendent and Owner's Rep walk the project and create punch list
   items, each assigned to the responsible subcontractor with a photo and
   location → subcontractors mark items complete → Superintendent verifies
   and closes each item → project cannot be marked substantially complete
   while open punch list items with a "must-fix-before-closeout" flag
   remain.
7. **Safety incident reporting**: a jobsite incident occurs → Safety
   Officer or Superintendent logs it immediately from the mobile app with
   photos, witnesses, and severity classification → Project Executive and
   Safety Officer are notified → a corrective-action workflow tracks the
   response through to close, feeding the firm's safety/EMR reporting.

## 8. Business Goals

- Give a GC real-time budget-vs-actuals visibility per project and
  portfolio-wide, reducing month-end financial surprises.
- Reduce RFI/submittal cycle time through a structured, auditable
  workflow instead of email threads.
- Reduce closeout delay by enforcing punch-list-complete gating before
  substantial completion.
- Reduce safety incident recurrence through structured incident tracking
  and corrective-action follow-through.

## 9. Functional Requirements

- **Project/job management**: project records with phases/milestones,
  schedule tracking, project team assignment (internal and subcontractor).
- **Budget vs. actuals**: cost-code-level original budget, committed cost
  (subcontracts + POs), actual cost, change-order impact, real-time
  variance reporting.
- **Subcontractor management**: subcontractor directory, contract value
  tracking, insurance/licensing compliance document tracking with expiry
  alerts, performance history across projects.
- **RFI/submittal workflow**: RFI creation/routing/response with drawing
  reference linkage and due-date tracking; submittal packages with review
  cycles (approved / approved as noted / revise and resubmit / rejected)
  and full revision history.
- **Daily site logs**: weather, crew counts, work performed, equipment,
  delays, and photo attachment, one log per project per day.
- **Change order management**: draft from RFI or independent scope change,
  internal pricing/approval, client approval routing, automatic budget
  update on approval.
- **Punch lists**: item creation with photo/location/assignee, status
  tracking, closeout-gating on unresolved must-fix items.
- **Safety incident reporting**: incident logging with severity
  classification, witness/photo capture, corrective-action tracking,
  linkage to project and involved parties.
- **Document control**: drawing and specification version control with
  supersede tracking, distribution log (who has the current set),
  markup/redline support on submittals and RFIs.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (change order internal + client approval), automation
  rules (e.g. insurance-expiry alerts, RFI overdue escalation), saved
  project/RFI filters, custom fields on project and subcontractor records,
  full audit history, soft-delete/restore, mass actions (bulk punch list
  assignment), command palette, report builder, scheduled budget reports.

## 10. Non-Functional Requirements

Baseline from [performance-standards.md](../../quality/performance-standards.md)
and [security-standards.md](../../security/security-standards.md) applies.
ZodiBuild-specific: the mobile field app (daily logs, punch lists, safety
incidents, RFI photo capture) must function with degraded/offline jobsite
connectivity and sync when connectivity returns, per an offline-first sync
design; document control must support large drawing-set file sizes
(multi-hundred-MB PDF sets) with responsive viewing via progressive/tiled
rendering rather than full-file download before view.

## 11. Architecture

ZodiBuild is a standalone, self-hosted Laravel application, sold as source
code to one general contractor or owner's-rep firm and deployed entirely
within that buyer's own hosting account — there is no shared platform
service and no other Zodize product it depends on at runtime
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
It is built by cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific tables (`loans`, `dps`, `fdr`, `branches`/
`branch_staff`, `other_banks`, `beneficiaries`, `airtime_operators`) are
stripped, and ZodiBuild's own domain modules — project management, budget &
cost, subcontractor management, RFIs & submittals, daily site logs, change
orders, punch lists, safety, document control — are built on top of the
inherited engine's wallet/ledger, payment gateways, RBAC/auth, KYC, i18n,
and admin configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)).

Projects are modeled as first-class entities within the one buyer's
deployment. Subcontractor firms are granted scoped external-collaborator
`web`-guard accounts through the inherited RBAC model (per
[rbac-permissions.md](../../security/rbac-permissions.md)) rather than any
form of separate tenancy — since every subcontractor account still belongs
to the same one deployment the GC owns, a subcontractor's restricted
visibility (their own project's RFIs/submittals/punch items only, never the
GC's full portfolio or another subcontractor's records) is enforced by
ordinary row-level authorization (policy checks scoped to
`project_subcontractors` membership), not tenant isolation — there is no
`tenant_id` anywhere in ZodiBuild's schema. Document control (drawings/
specs) uses project-scoped document storage with a version-chain data
model (each version supersedes the prior, never overwritten), and
large-file rendering is offloaded to an asynchronous tiling/preview
generation job per
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); object
storage for drawing/spec files with an async tiled-preview generation
pipeline; offline-capable mobile field app built on the shared Vue
component library with a local-first sync layer for daily logs, punch
lists, and safety incidents.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Project Management | Projects, Phases/Milestones, Team Assignment, Schedule Tracking |
| Budget & Cost | Cost Codes, Original Budget, Committed Cost, Actuals, Variance Reporting |
| Subcontractor Management | Directory, Contracts, Insurance/Licensing Compliance, Performance History |
| RFIs & Submittals | RFI Routing, Submittal Review Cycles, Revision History, Drawing Linkage |
| Daily Site Logs | Weather/Crew/Equipment Logging, Photo Attachment, Delay Tracking |
| Change Orders | Draft/Pricing, Internal Approval, Client Approval, Budget Update |
| Punch Lists | Item Creation, Assignment, Status Tracking, Closeout Gating |
| Safety | Incident Reporting, Severity Classification, Corrective-action Tracking |
| Document Control | Drawing/Spec Versioning, Supersede Tracking, Distribution Log, Markup |
| Reporting | Budget Dashboards, RFI/Submittal Cycle-time, Safety Trend, Report Builder |

## 14. Core Data Model

All tables belong to the one buyer's one deployment — there is no
`tenant_id` column anywhere in this model
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
Subcontractor collaborator access is scoped by the `project_subcontractors`
join table, not by a separate tenant construct.

| Entity | Key columns | Notes |
|---|---|---|
| `projects` | id, name, status, original_budget, start_date, target_completion | Core project record |
| `project_phases` | id, project_id, name, sequence, status, start_date, end_date | Schedule phases/milestones |
| `cost_codes` | id, project_id, code, description, budget_amount | Budget line-item structure |
| `budget_actuals` | id, cost_code_id, committed_amount, actual_amount, recorded_at | Committed vs. actual roll-up |
| `subcontractors` | id, company_name, trade, contact_info | Directory entity, reused across projects |
| `project_subcontractors` | id, project_id, subcontractor_id, contract_value, status | Contract linkage per project |
| `compliance_documents` | id, subcontractor_id, doc_type, expires_at, file_ref | Insurance/licensing tracking |
| `rfis` | id, project_id, number, subject, drawing_ref, status, due_date, submitted_by | Request for information |
| `rfi_responses` | id, rfi_id, responded_by, response_text, responded_at, cost_impact_flag | RFI answer, may trigger a CO draft |
| `submittals` | id, project_id, subcontractor_id, title, current_version_id, status | Submittal package |
| `submittal_versions` | id, submittal_id, version_number, file_ref, reviewed_by, review_status | Full revision history |
| `daily_logs` | id, project_id, log_date, weather, crew_counts, work_performed, photos | One per project per day |
| `change_orders` | id, project_id, rfi_id, description, amount, status, client_approved_at | Budget-impacting approval chain |
| `punch_items` | id, project_id, description, location, assignee_id, must_fix, status | Closeout gating source |
| `safety_incidents` | id, project_id, severity, occurred_at, description, status | Incident with corrective-action linkage |
| `drawings` | id, project_id, sheet_number, current_version_id | Document control root |
| `drawing_versions` | id, drawing_id, version_number, file_ref, superseded_at | Version chain, never overwritten |

## 15. Key API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/projects` | List projects with status and budget summary |
| POST | `/api/v1/projects` | Create a project |
| GET | `/api/v1/projects/{project}/budget-summary` | Real-time budget-vs-actuals for a project |
| POST | `/api/v1/projects/{project}/subcontractors` | Add a subcontractor to a project |
| GET | `/api/v1/subcontractors/{subcontractor}/compliance` | Insurance/licensing compliance status |
| POST | `/api/v1/projects/{project}/rfis` | Create an RFI |
| POST | `/api/v1/rfis/{rfi}/responses` | Respond to an RFI |
| POST | `/api/v1/projects/{project}/submittals` | Create a submittal package |
| POST | `/api/v1/submittals/{submittal}/versions` | Upload a new submittal revision |
| PATCH | `/api/v1/submittal-versions/{version}/review` | Record a review decision |
| POST | `/api/v1/projects/{project}/daily-logs` | Submit a daily site log |
| POST | `/api/v1/projects/{project}/change-orders` | Draft a change order |
| POST | `/api/v1/change-orders/{co}/approve` | Record an internal or client approval step |
| POST | `/api/v1/projects/{project}/punch-items` | Create a punch list item |
| PATCH | `/api/v1/punch-items/{item}/status` | Update punch item status |
| GET | `/api/v1/projects/{project}/closeout-readiness` | Check open must-fix punch items blocking closeout |
| POST | `/api/v1/projects/{project}/safety-incidents` | Report a safety incident |
| PATCH | `/api/v1/safety-incidents/{incident}/corrective-action` | Update corrective-action status |
| POST | `/api/v1/projects/{project}/drawings/{drawing}/versions` | Upload a new drawing version |
| GET | `/api/v1/portfolio/reports/budget-variance` | Portfolio-wide budget variance report |

## 16. Events

`project.created`, `phase.status_changed`, `budget.variance_threshold_exceeded`,
`rfi.created`, `rfi.responded`, `rfi.overdue`, `submittal.status_changed`,
`change_order.drafted`, `change_order.internally_approved`,
`change_order.client_approved`, `punch_item.created`,
`punch_item.completed`, `project.closeout_blocked`,
`safety_incident.reported`, `safety_incident.corrective_action_completed`,
`drawing.version_superseded`, `compliance_document.expiring_soon`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `rfi.overdue` | ✔ (architect, PM) | ✔ | — | ✔ |
| `submittal.status_changed` | ✔ (subcontractor) | ✔ | — | — |
| `change_order.client_approved` | ✔ (PM, owner's rep) | ✔ | — | — |
| `budget.variance_threshold_exceeded` | ✔ (PM, project exec) | ✔ | — | ✔ |
| `safety_incident.reported` | ✔ (safety officer, project exec) | ✔ | ✔ | ✔ |
| `compliance_document.expiring_soon` | ✔ (PM) | ✔ | — | — |
| `punch_item.completed` (assignee-facing) | ✔ | — | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits the base codebase's default admin roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles)),
scoped per project. ZodiBuild-specific permissions: `projects.manage`,
`budget.manage`, `rfis.manage`, `submittals.review` (Architect-scoped),
`change_orders.approve_internal`, `change_orders.approve_client` (Owner's
Rep-scoped, external collaborator role), `punch_items.manage`,
`safety_incidents.manage`, `drawings.manage`. Subcontractor collaborator
accounts by default see only their own project's RFIs, submittals, and
punch items assigned to them — never portfolio-wide or another
subcontractor's data.

## 19. Workflows & Approval Chains

- **Change order approval**: internal approval (Project Manager or Project
  Executive per configured threshold) required before routing to the
  Owner's Representative for client approval; only a fully client-approved
  change order updates the project's approved budget, per
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **Submittal review cycle**: each submittal version requires an explicit
  review decision (approved / approved as noted / revise and resubmit /
  rejected) before the project can reference it as current.
- **Closeout gating**: a project cannot transition to "Substantially
  Complete" while open punch items flagged must-fix-before-closeout
  remain, enforced at the workflow-transition level, not just a UI warning.
- **Safety incident sign-off**: an incident cannot close until its
  corrective-action items are complete and the Safety Officer signs off.

## 20. Audit Logs

Every budget change, RFI response, submittal review decision, change order
approval step, punch item status change, and drawing version supersession
is recorded to the deployment's audit log with actor, timestamp,
before/after values, and project scope — per
[audit-logging.md](../../security/audit-logging.md). Submittals and
drawings additionally carry a complete, immutable version chain distinct
from general audit history, since it is the record of what was actually
built against.

## 21. Reports & Analytics & Dashboards

Standard dashboards (per
[dashboard-standards.md](../../standards/dashboard-standards.md)):
portfolio budget variance, RFI/submittal cycle-time by project, open punch
item aging, subcontractor compliance status board, safety incident trend
by severity and project. Report builder supports custom budget and
compliance reports, saved and scheduled per
[product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Accounting/ERP**: export of committed cost and change order data to
  external accounting systems (or [ZodiBusiness](../ZodiBusiness/SPEC.md)
  where used as the firm's back-office system).
- **BIM/CAD viewers**: drawing preview rendering compatible with common CAD
  export formats for in-browser markup without native software.
- **Scheduling tools**: optional two-way sync with external critical-path
  scheduling tools (e.g. Microsoft Project-class CPM schedules) for phase
  and milestone dates.
- **Payment/lien waiver processing**: optional integration with
  construction-specific payment and lien waiver management services.
- **Weather data**: automatic weather capture on daily logs via a weather
  API feed, reducing manual entry.

## 23. AI Features

- **RFI drafting assistance**: AI-assisted drafting of an RFI from a
  photo and short description, grounded in the linked drawing/spec
  context, always reviewed by the Superintendent before submission.
- **Budget risk flagging**: AI-assisted detection of cost codes trending
  toward overrun based on committed-vs-actual velocity, surfaced to the
  Project Manager before it becomes a variance-threshold breach.
- **Daily log summarization**: AI-assisted summary of a week's daily logs
  into a client-facing progress summary, always reviewed before sending.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: nightly compliance-document expiry sweep, RFI
  overdue-escalation check, budget variance-threshold evaluation, drawing
  preview/tiling generation queue processing.
- CLI commands (Artisan): `build:compliance:sweep`, `build:rfi:escalate`,
  `build:budget:evaluate-variance`, `build:drawings:regenerate-previews` —
  each requires the same authorization context as its API equivalent, no
  CLI bypass of RBAC.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions 2 demo projects (a $12M commercial build-out and a
$4M multi-family renovation) with phased schedules, cost-code budgets with
realistic committed/actual variance, a subcontractor roster with
compliance documents (including one intentionally near-expiry), an active
RFI/submittal history with at least one full revise-and-resubmit cycle, 60
days of daily logs with photos, at least one fully approved change order,
an open and a closed punch list, and one closed safety incident with
corrective-action history — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10; additionally: a project's budget-vs-actuals summary computes in
under 1 second even for a project with thousands of cost-code line items,
and drawing preview rendering for a multi-hundred-page sheet set begins
displaying the first sheet in under 3 seconds via progressive load.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. Subcontractor external-collaborator access is strictly scoped to
their own project and their own submitted/assigned records, verified by a
dedicated cross-subcontractor authorization test suite covering every
policy that gates project, RFI, submittal, and punch-item access (this is
row-level authorization within one deployment, not the tenant-isolation
category described in
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model)),
per
[data-protection-privacy.md](../../security/data-protection-privacy.md).
Drawing/submittal version history is immutable — corrections are new
versions, never edits to a prior version.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated closeout-gating test suite verifying a project cannot transition
to substantially complete while blocking punch items remain, and a
subcontractor-isolation suite verifying no subcontractor can see another
subcontractor's or the GC's non-shared project data.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Drawing preview/tiling generation workers deploy independently of the web
tier so large document-set processing does not degrade interactive
RFI/submittal/daily-log responsiveness.

## 30. Acceptance Criteria

- A change order only updates the project's approved budget after both
  internal and client approval steps are recorded.
- A project cannot be marked substantially complete while open
  must-fix-before-closeout punch items remain.
- A submittal's revision history is fully retained and viewable in
  sequence, with the current approved version clearly distinguished.
- A subcontractor's collaborator account cannot access another
  subcontractor's RFIs, submittals, or punch items, or any project they
  are not assigned to.

## 31. Production Checklist

See
[production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).
ZodiBuild additionally requires sign-off that the subcontractor
cross-isolation test suite passes before a buyer's deployment is allowed to
invite external subcontractor collaborators in production.

## 32. Future Roadmap

- Native critical-path schedule (CPM/Gantt) authoring, not just external
  sync.
- Prefabrication/BIM clash-detection integration.
- Bid management module for preconstruction subcontractor bidding.

## 33. Known Risks

- Large-file document control at scale: multi-hundred-MB drawing sets
  stress storage and preview-generation infrastructure — mitigated by
  async tiling, but requires continued capacity planning as project volume
  grows.
- Subcontractor collaborator model complexity: external-party access
  scoping is a persistent, high-consequence attack surface, mitigated by
  the dedicated isolation test suite but requiring ongoing review as
  collaboration features expand.

## 34. Future Improvements

- Real-time collaborative markup on drawings (multi-user simultaneous
  redlining).
- Predictive schedule-slip forecasting based on daily log delay patterns.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), exhaustive endpoint
catalog (companion `API_REFERENCE.md`) covering full document-control
distribution-log endpoints and cost-code-level budget import/export, and
full report catalog beyond the dashboards listed in §21. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).
