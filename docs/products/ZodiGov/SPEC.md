# ZodiGov — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth). See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for spec status
> definitions.

## 1. Vision

ZodiGov is a government service delivery and citizen records platform that
gives a municipality, county, or agency one system for citizen service
requests, permit/license workflows, public records management, and
department case routing, with a public-facing transparency portal built in
— so residents get a modern 311-style experience and department staff get
one case queue instead of a different intake tool per department.

## 2. Purpose

Local government software is usually either a decade-old case management
system with no public portal, or a citizen-facing request app with no real
backend workflow engine. ZodiGov exists to close that gap: one platform
that routes a citizen's request or application to the right department,
tracks it to resolution with full public accountability, and does so to
public-sector procurement standards — accessibility, records retention, and
security review — that most government software vendors treat as a special
add-on tier rather than the default.

## 3. Target Market

Municipal and county governments, special districts (utility, transit,
parks), and state-agency service-delivery divisions that need citizen
service requests, permitting/licensing, public records, and department case
management in one platform, procurable under standard public-sector
procurement processes.

## 4. Industries

Municipal government, county government, special-purpose government
districts, and state agency service-delivery operations.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Citizen service requests (311-style) | SeeClickFix, Salesforce Public Sector 311 | Built on the same case-routing engine as permits and records, not a siloed 311-only tool |
| Permit/license workflow | Accela, OpenGov Permitting & Licensing | Configurable workflow builder shared with every Zodize product's approval-chain infrastructure |
| Public records management | NextRequest, GovQA | Records requests share ZodiGov's audit trail and redaction workflow end to end |
| Open data / transparency portal | Socrata, OpenGov | Transparency portal is a first-class module, not a separate CKAN-style deployment |
| Case management across departments | Accela, Tyler Technologies EnerGov | Department routing built on ZodiCore's RBAC/tenancy model, avoiding department-siloed logins |

## 6. Personas

- **Citizen/Resident** — submits service requests and permit/license
  applications, tracks status, accesses public records via the citizen
  portal.
- **Department Staff/Case Worker** — triages and resolves assigned service
  requests and case work within their department queue.
- **Permit/License Reviewer** — reviews and approves/denies permit and
  license applications, including inspections where required.
- **Records Officer** — manages public records requests, redaction, and
  statutory response deadlines.
- **Department Supervisor** — oversees departmental case load, SLA
  compliance, and escalations.
- **Agency Administrator** — configures departments, workflows, and the
  public transparency portal; owns accessibility and security posture.

## 7. User Journeys

1. **311-style service request**: Resident reports an issue (pothole,
   graffiti, streetlight outage) via the citizen portal or mobile-responsive
   web form, optionally with a photo and geolocation → request auto-routes
   to the correct department based on category and location → department
   staff triages, assigns, and works the case → resident receives status
   updates → on resolution, the resident can confirm satisfaction, and the
   closed request appears on the public transparency map (with any PII
   suppressed) per §21.
2. **Permit application through inspection to issuance**: Applicant submits
   a permit application (e.g. building permit) with required documents and
   fees → application routes through the configured review workflow
   (planning review, code review, inspection scheduling as applicable) →
   each reviewer approves, requests changes, or denies with documented
   reasons → required inspections are scheduled and their results recorded
   → on full approval, the permit is issued and available for the applicant
   to download/print from the portal.
3. **Public records request with redaction**: Citizen submits a public
   records request via the portal → Records Officer logs the statutory
   response-deadline clock (jurisdiction-configurable) → responsive
   documents are located and reviewed → any exempt content (e.g. personal
   information, active-investigation material) is redacted with the
   redaction reason recorded → the response package is released to the
   requester before the statutory deadline, with the full request lifecycle
   logged for audit and, where required, published to a request log.
4. **Department case escalation**: A service request or permit review sits
   unresolved past its configured SLA → it escalates automatically to the
   Department Supervisor → supervisor reassigns, requests more resources,
   or documents a justified delay reason visible to the resident (e.g.
   "pending parts delivery") → escalation and resolution are both part of
   the case's public-facing status history.
5. **Accessible public transparency portal use**: A resident using a screen
   reader browses the open-data/transparency portal to check the status of
   a nearby service request or view permit activity in their area → every
   interactive element, form, and data visualization on the portal meets
   WCAG 2.1 AA as a hard requirement (§27), not a best-effort target, since
   the portal is the public's primary interface to government services.

## 8. Business Goals

- Reduce time-to-resolution for citizen service requests through
  department auto-routing and SLA-driven escalation instead of manual
  triage.
- Make permit/license review cycle time transparent and predictable for
  applicants, reducing call-center load on department staff.
- Meet statutory public-records response deadlines by default through
  built-in deadline tracking, reducing legal exposure for missed responses.
- Give agencies a procurement-ready transparency and accessibility posture
  out of the box, removing the accessibility remediation project that
  typically follows a government software rollout.

## 9. Functional Requirements

- Citizen service requests: category-based intake, geolocation, photo
  attachment, department auto-routing, status tracking, public map view.
- Permit/license applications: configurable application forms per permit
  type, document/fee collection, multi-stage review workflow, inspection
  scheduling and results, issuance and renewal.
- Public records management: request intake, statutory deadline tracking,
  document search/retrieval, redaction workflow with reason capture,
  response packaging and release, request log.
- Department case routing: unified case queue across departments, SLA
  configuration per case type, escalation rules, reassignment.
- Public transparency/open-data portal: published datasets, service-request
  and permit activity maps/dashboards, WCAG 2.1 AA compliant by default.
- Citizen portal: request submission and tracking, permit application and
  status, records request submission, document downloads.
- Department staff tooling: case queue, assignment, internal notes
  (distinguished from public-facing status updates), SLA dashboards.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiGov-specific additions:

- Every citizen-facing page (portal, transparency portal, forms) meets WCAG
  2.1 AA per §27 — accessibility is a release-blocking requirement, not a
  post-launch remediation item, per
  [accessibility-checklist.md](../../checklists/accessibility-checklist.md).
- Citizen portal and transparency portal pages p95 < 500ms, since these are
  the public's direct interface to government services and are held to a
  stricter responsiveness bar than internal staff tooling.
- Public records statutory deadline tracking must never silently miss a
  deadline — the system escalates before, not after, a deadline lapses.

## 11. Architecture

ZodiGov is a tenant application on [ZodiCore](../ZodiCore/SPEC.md),
consuming `zodize/core-identity`, `zodize/core-billing`,
`zodize/core-notifications`, `zodize/core-permissions`, and
`zodize/core-plugins` per ZodiCore's
[Architecture](../ZodiCore/SPEC.md#11-architecture) section. ZodiGov adds a
department-scoped case-routing engine (`CaseRoutingContract`) built on
ZodiCore's RBAC and workflow-builder patterns, and a `PublicRecordContract`
that governs what data is safe to surface on the public transparency
portal — every entity exposed there passes through an explicit
public-visibility policy rather than being exposed by default. Departments
map onto ZodiCore's tenant/company/branch hierarchy per
[multi-tenancy.md](../../architecture/multi-tenancy.md), letting a single
municipality tenant model each department (Public Works, Planning, Parks)
as a branch with its own case queue and staff roster.

## 12. Technology

Laravel + Vue per the shared stack
([coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md),
[coding-standards-vue.md](../../development/coding-standards-vue.md));
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); the
citizen and transparency portals are built against
[accessibility.md](../../design-system/accessibility.md) and audited with
automated + manual WCAG 2.1 AA testing as part of CI, not only at launch;
payment processing (permit/license fees) via ZodiCore's payment gateway
abstraction
(§20 of [ZodiCore's SPEC.md](../ZodiCore/SPEC.md#20-payment-gateways-wallet-accounting-taxes-invoices));
geolocation/mapping via a standards-based mapping provider for the service
request map and transparency portal.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Service Requests | Intake, Category Routing, Geolocation/Map, Status Tracking |
| Permits & Licensing | Application Forms, Review Workflow, Fees, Inspections, Issuance/Renewal |
| Public Records | Request Intake, Deadline Tracking, Redaction Workflow, Response Release, Request Log |
| Case Routing | Department Queues, SLA Configuration, Escalation, Reassignment |
| Transparency Portal | Open Data Publishing, Public Maps/Dashboards, Accessibility Compliance |
| Citizen Portal | Request/Application Submission, Status Tracking, Document Access |
| Department Tools | Case Queue, Internal Notes, SLA Dashboards, Inspection Scheduling |

## 14. Core Data Model

Full ER diagram queued (§ Roadmap). Core entities:

| Entity | Key columns |
|---|---|
| `departments` | id, tenant_id (maps to branch), name, sla_default_hours |
| `service_requests` | id, tenant_id, department_id, category, status, location_lat, location_lng, submitted_by |
| `service_request_updates` | id, service_request_id, is_public, note, created_by, created_at |
| `permit_applications` | id, applicant_id, permit_type, status, submitted_at, fee_amount, fee_paid_at |
| `permit_review_stages` | id, permit_application_id, stage_name, reviewer_id, decision, decided_at |
| `inspections` | id, permit_application_id, inspection_type, scheduled_at, result, inspector_id |
| `records_requests` | id, requester_id, description, statutory_deadline, status, released_at |
| `records_request_documents` | id, records_request_id, document_id, redaction_applied, redaction_reason |
| `cases` | id, department_id, case_type (service_request/permit/records), sla_due_at, escalated_at, assigned_to |
| `public_datasets` | id, tenant_id, dataset_name, source_entity, refresh_frequency, visibility_policy_id |
| `visibility_policies` | id, entity_type, field_allowlist, pii_suppression_rules |

## 15. Key API Endpoints

Full endpoint catalog queued (§ Roadmap). Key routes, all conforming to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md):

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/service-requests` | Submit a citizen service request |
| GET | `/api/v1/service-requests/{request}` | View a service request's status/history |
| PATCH | `/api/v1/service-requests/{request}/assign` | Assign to department staff |
| POST | `/api/v1/service-requests/{request}/updates` | Add an internal or public status update |
| POST | `/api/v1/permit-applications` | Submit a permit/license application |
| POST | `/api/v1/permit-applications/{application}/review-stages/{stage}/decide` | Record a review decision |
| POST | `/api/v1/permit-applications/{application}/inspections` | Schedule an inspection |
| POST | `/api/v1/permit-applications/{application}/issue` | Issue the permit/license |
| POST | `/api/v1/records-requests` | Submit a public records request |
| POST | `/api/v1/records-requests/{request}/documents/{document}/redact` | Apply a redaction with reason |
| POST | `/api/v1/records-requests/{request}/release` | Release the response package |
| GET | `/api/v1/cases` | Department case queue (staff view) |
| PATCH | `/api/v1/cases/{case}/escalate` | Manually escalate a case |
| GET | `/api/v1/public/transparency/datasets/{dataset}` | Public open-data dataset access |
| GET | `/api/v1/public/transparency/service-requests-map` | Public service-request map data |
| GET | `/api/v1/portal/citizen/requests` | Citizen portal — own request history |
| GET | `/api/v1/reports/sla-compliance` | Department SLA compliance report |

## 16. Events

`service_request.submitted`, `service_request.assigned`,
`service_request.resolved`, `permit_application.submitted`,
`permit_review.decided`, `permit.issued`, `inspection.scheduled`,
`inspection.completed`, `records_request.submitted`,
`records_request.deadline_approaching`, `records_request.released`,
`case.escalated`, `dataset.published`. See
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `service_request.submitted` (confirmation) | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `service_request.resolved` | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `permit_review.decided` (changes requested/denied) | ✔ | ✔ | — | ✔ |
| `permit.issued` | ✔ | ✔ | — | ✔ |
| `inspection.scheduled` | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `records_request.deadline_approaching` (to Records Officer) | ✔ | ✔ | — | ✔ |
| `records_request.released` | ✔ | ✔ | — | — |
| `case.escalated` (to Department Supervisor) | ✔ | ✔ | — | ✔ |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits ZodiCore's default system roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles))
plus ZodiGov-specific roles: `Department Staff/Case Worker`,
`Permit/License Reviewer`, `Records Officer`, `Department Supervisor`,
`Agency Administrator`, `Citizen` (portal-only, scoped to own requests/
applications). Key permissions: `service_requests.assign`,
`permits.review_decide`, `permits.issue`, `records_requests.redact`
(Records Officer only), `records_requests.release` (requires Records
Officer + Department Supervisor sign-off for exempt-content releases, per
§19), `cases.escalate`, `datasets.publish` (Agency Administrator only,
gated by `PublicRecordContract`).

## 19. Workflows & Approval Chains

- **Permit multi-stage review**: an application advances through
  tenant-configured review stages (e.g. planning → code → fire); each stage
  requires an explicit decision (approve/request changes/deny) before
  advancing, and a denial at any stage halts issuance until resolved.
- **Records request release with exempt content**: if any responsive
  document contains redacted/exempt content, release requires both the
  Records Officer's redaction sign-off and Department Supervisor
  countersignature before the response package is sent — a two-person
  control on what leaves the agency.
- **SLA escalation chain**: a case unresolved past its configured SLA
  escalates automatically to the Department Supervisor, matching the
  pattern in §7 journey 4; the supervisor's resolution (reassign, extend
  with public-facing reason, resolve) is itself logged to the case's public
  status history.
- **Public dataset publication approval**: publishing a new dataset to the
  transparency portal requires Agency Administrator approval of its
  `visibility_policy_id` field allowlist and PII suppression rules before
  it goes live, preventing accidental PII exposure on the public portal.

## 20. Audit Logs

Every case assignment/escalation, permit review decision, records request
redaction and release, and dataset publication is immutably audit-logged
per [audit-logging.md](../../security/audit-logging.md), supporting
public-sector records-retention and open-government accountability
requirements — the audit log itself is treated as a government record
subject to the tenant's retention policy.

## 21. Reports & Analytics & Dashboards

- Service request volume, category breakdown, and resolution-time
  dashboards, both internal (full detail) and public (PII-suppressed).
- Permit/license review cycle-time and throughput reporting by type and
  stage.
- Public records response-time compliance reporting against statutory
  deadlines.
- Department SLA compliance and escalation-rate dashboards.
- Public transparency portal analytics: published dataset usage, map
  interaction volume.
- Report builder and scheduled reports per the
  [Second-Layer Feature Catalog](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **GIS/mapping**: standards-based mapping provider for service-request
  geolocation and the public transparency map.
- **Payment processing**: permit/license fee collection via ZodiCore's
  payment gateway abstraction.
- **Open data publishing**: export-compatible with common open-data
  catalog standards so datasets can be cross-published to state/federal
  open-data portals where required.
- **Notification/alerting**: emergency/public-notice broadcast integration
  for department-wide or jurisdiction-wide alerts distinct from routine
  case notifications.
- **Identity verification**: optional identity-proofing integration for
  permit applications requiring verified applicant identity (e.g.
  licensing).

## 23. AI Features

- AI-assisted service request categorization and department routing from
  free-text/photo submissions, with department staff able to correct
  mis-routes (feeding back into routing accuracy over time).
- AI-assisted redaction suggestion for public records documents (flagging
  likely PII/exempt content spans), always requiring Records Officer
  confirmation before any redaction is applied — the AI never redacts or
  releases autonomously.
- Anomaly detection on case escalation and SLA-miss patterns, extending
  ZodiCore's audit anomaly detection
  (§23 of [ZodiCore's SPEC.md](../ZodiCore/SPEC.md#23-ai-features)).

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: SLA breach detection and escalation, records-request
  statutory deadline countdown alerts, inspection reminder dispatch, permit
  renewal reminder generation, public dataset refresh.
- CLI commands: `gov:refresh-public-datasets`, `gov:sweep-sla-breaches`,
  `gov:sweep-records-deadlines`, `gov:reindex-transparency-portal` — each
  requiring the same authorization context as its API equivalent.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a demo municipality with 4 departments (Public
Works, Planning, Parks, Records), 200 demo service requests across open/
resolved/escalated statuses, a populated permit application history
including one multi-stage review with an inspection, a public records
request history including one redacted-and-released case, and a published
transparency portal dataset with realistic (synthetic) map data — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10. Additionally: the public transparency map and open-data endpoints
must remain responsive under public, unauthenticated traffic spikes (e.g.
local news coverage driving traffic) without degrading authenticated staff
case-queue performance — rate limiting and caching are applied to the
public surface independently of the staff-facing API.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiGov-specific requirements:

- **Accessibility (WCAG 2.1 AA, mandatory)**: every citizen-facing surface
  (citizen portal, transparency portal, public forms) must pass WCAG 2.1 AA
  automated and manual audit before release, per
  [accessibility-checklist.md](../../checklists/accessibility-checklist.md)
  — this is a release gate, not a target, reflecting public-sector legal
  accessibility mandates (e.g. ADA Title II-equivalent obligations).
- **Public records exposure control**: any entity surfaced on the public
  transparency portal passes through an explicit `visibility_policy_id`
  allowlist (§14, §19); there is no default-exposed entity — publication is
  opt-in and approved, preventing accidental PII leakage to the public.
- **Procurement-grade security review**: ZodiGov tenants undergo a security
  review process matching public-sector procurement expectations (e.g.
  StateRAMP/FedRAMP-adjacent control mapping where applicable to the
  tenant's jurisdiction), documented against
  [security-checklist.md](../../checklists/security-checklist.md) as part
  of onboarding, in addition to Zodize's standard annual penetration
  testing baseline.
- **Records retention**: service requests, permit records, and public
  records request logs follow the tenant's configured government
  records-retention schedule per
  [data-protection-privacy.md](../../security/data-protection-privacy.md),
  which for government records is typically longer and more prescriptive
  than Zodize's default commercial retention baseline.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md);
additionally a dedicated accessibility regression suite (automated axe-core
class checks plus a documented manual audit checklist) run as a required CI
gate on every citizen-facing surface, and a public-visibility-policy test
suite verifying no entity is exposed on the transparency portal without an
approved `visibility_policy_id`.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
ZodiGov deployments additionally require the procurement-grade security
review sign-off in §27 before a tenant's public-facing surfaces go live.

## 30. Acceptance Criteria

- A citizen can submit a service request, have it auto-route to the correct
  department, and track it to resolution, with a PII-suppressed version
  visible on the public transparency map.
- A permit application advances through every configured review stage with
  a documented decision at each, and cannot be issued while any stage is
  unresolved or denied.
- A public records request's statutory deadline is tracked from intake to
  release, escalates before the deadline lapses, and any redaction carries
  a documented reason and Department Supervisor countersignature before
  release.
- Every citizen-facing page passes the WCAG 2.1 AA automated and manual
  audit checklist with zero release-blocking violations.
- No entity appears on the public transparency portal without an approved
  visibility policy.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiGov additionally requires the accessibility checklist
([accessibility-checklist.md](../../checklists/accessibility-checklist.md))
and security checklist
([security-checklist.md](../../checklists/security-checklist.md)) sign-off
described in §27/§29 before go-live.

## 32. Future Roadmap

- Multi-jurisdiction shared-services deployment for regional service
  authorities spanning multiple municipalities.
- Constituent case management extension for elected-official casework
  offices.
- Expanded open-data catalog automation with scheduled cross-publication to
  state/federal portals.

## 33. Known Risks

- Accessibility regressions on the public portal carry direct legal
  exposure for government tenants — mitigated by the mandatory CI gate in
  §28, but continued manual audit discipline is required as new components
  ship, since automated tooling alone does not catch every WCAG criterion.
- Public transparency portal PII exposure is a high-severity risk specific
  to this product's public-facing nature — mitigated by the opt-in
  visibility-policy design in §27, but every new public dataset type
  requires deliberate policy review before publication remains possible by
  design, not merely by discipline.

## 34. Future Improvements

- Configurable statutory deadline rule packs per state/jurisdiction beyond
  the initial covered set.
- Constituent communication preference center spanning service requests,
  permits, and records requests in one place.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), full endpoint
catalog (companion `API_REFERENCE.md`), full jurisdiction statutory
deadline rule library, and a complete report catalog beyond the summary
list in §21.
