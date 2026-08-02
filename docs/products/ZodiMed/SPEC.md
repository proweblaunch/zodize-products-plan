# ZodiMed — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth). See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for spec status
> definitions.

## 1. Vision

ZodiMed is a clinic and small-to-mid-size hospital management platform that
unifies scheduling, an EHR-lite clinical record, e-prescribing, and
insurance billing into one system built to HIPAA-equivalent standards from
its first commit, so that an independent practice or regional clinic group
can run patient care and revenue cycle operations without stitching
together a legacy PM system, a bolted-on EHR, and a separate clearinghouse
portal.

## 2. Purpose

Clinics below enterprise scale (Epic/Cerner-class deployments) are
underserved: they either overpay for enterprise EHR implementations sized
for hospital systems, or underinvest in compliance-grade access control
using lightweight tools not built for PHI. ZodiMed exists to give a clinic
or small hospital group a system where minimum-necessary access, audit
trails, and claims-ready coding are the default behavior of the product, not
a configuration project.

## 3. Target Market

Independent and small-group outpatient clinics (primary care, specialty,
urgent care), multi-location clinic groups, and small community
hospitals/ambulatory surgical centers that need patient records, scheduling,
billing, and a patient portal in one system without hospital-enterprise
implementation cost or timeline.

## 4. Industries

Ambulatory healthcare, urgent care, specialty practice groups (dermatology,
behavioral health, physical therapy, dental-adjacent), and small community
hospital/ASC operations.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| EHR + clinical documentation | Epic, athenahealth, Cerner (Oracle Health) | Right-sized for clinics that don't need Epic-scale implementation, sold as source code the practice owns and self-hosts, built on the same audited base engine every Zodize product is cloned from |
| Practice management/scheduling | athenahealth, DrChrono, Kareo | Scheduling and billing share one data model — no separate PM/EHR sync problem |
| Patient portal | MyChart, athenahealth Patient Portal | Same design system and portal shell used across every Zodize product, lowering per-product build cost |
| Billing/claims (CPT/ICD) | Kareo, AdvancedMD | Claims workflow built on ZodiMed's own immutable audit trail for payer dispute defense |
| E-prescribing | DrFirst, Surescripts-integrated EHRs | Standards-based Surescripts integration rather than a proprietary network |

## 6. Personas

- **Physician/Provider** — documents encounters, writes prescriptions,
  orders labs, reviews results.
- **Nurse/Clinical Staff** — rooms patients, records vitals, administers
  care under provider orders.
- **Front Desk/Scheduling Staff** — manages appointments, check-in,
  insurance verification, copay collection.
- **Billing/Coding Staff** — assigns CPT/ICD codes, submits claims, works
  denials.
- **Practice Administrator** — manages providers, locations, payer
  contracts, and compliance posture.
- **Patient** — books appointments, messages providers, views results and
  statements via the patient portal.

## 7. User Journeys

1. **New patient visit, start to claim**: Patient self-schedules via the
   portal or front desk books the appointment → intake forms and insurance
   captured pre-visit → check-in verifies eligibility → nurse records vitals
   → provider documents the encounter (chief complaint, assessment, plan)
   → provider orders labs and/or e-prescribes → visit is coded (CPT/ICD-10)
   by billing staff or provider-assisted coding → claim submitted to the
   clearinghouse → remittance posts and patient balance (if any) is billed
   to the portal.
2. **Lab order to result review**: Provider orders a lab from the encounter
   → order transmitted to the integrated lab (§22) → result returns and is
   matched to the order → provider reviews and acknowledges the result
   (required before it is released to the patient portal) → abnormal
   results trigger a provider notification distinct from routine results →
   patient is notified a result is available once provider-released.
3. **E-prescribing with an interaction check**: Provider selects a
   medication in the encounter → system checks against the patient's active
   medication list and documented allergies → interaction/allergy warning
   surfaces if applicable, requiring provider override with a reason if they
   proceed → prescription transmitted electronically to the patient's
   pharmacy of record via the e-prescribing network.
4. **Insurance claim denial and resubmission**: Claim returns denied from
   the clearinghouse → Billing Staff reviews the denial reason code →
   corrects the coding or documentation gap → resubmits within the payer's
   timely-filing window, with the original and corrected claim both retained
   in the audit trail → if unresolved, the claim routes to an appeals
   workflow.
5. **Minimum-necessary access in a multi-provider practice**: A provider not
   on a patient's care team attempts to open that patient's chart → access
   requires either a documented treatment relationship, a break-glass
   emergency-access justification, or explicit patient consent on file — any
   of the three is logged with the specific reason, per §27.

## 8. Business Goals

- Let independent and small-group practices run compliant, claims-ready
  operations without an enterprise EHR implementation budget.
- Reduce claim denial rate through coding assistance and clearinghouse
  integration built into the encounter workflow, not a separate step.
- Make minimum-necessary access enforcement a built-in default so practices
  pass HIPAA-equivalent audits without custom configuration.
- Increase patient portal adoption (self-scheduling, result access, billing)
  as a differentiator against legacy PM-only tools.

## 9. Functional Requirements

- Patient records (EHR-lite): demographics, problem list, medication list,
  allergy list, immunization history, encounter documentation (SOAP-style
  notes), care team assignment.
- Appointment scheduling: provider/resource calendars, appointment types,
  recall/reminder scheduling, waitlist, multi-location support.
- Clinical notes: structured and free-text encounter documentation,
  templated note types per specialty, addendum/amendment with full version
  history (never silent edits to a signed note).
- E-prescribing: medication search, interaction/allergy checking, electronic
  transmission, controlled substance workflow support (where applicable,
  with elevated authentication).
- Billing/claims: CPT/ICD-10 coding assistance, claim generation,
  clearinghouse submission, remittance posting, patient statement
  generation, denial/appeal workflow.
- Lab order and results integration: order transmission, result ingestion
  and matching, provider acknowledgment gate before patient release.
- Patient portal: self-scheduling, secure messaging, result viewing
  (post-release), bill pay, intake form completion, document access.
- Minimum-necessary access control: care-team-scoped chart access,
  break-glass emergency access with mandatory justification, patient consent
  tracking.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiMed-specific additions:

- Chart open (patient summary view) p95 < 500ms even for patients with long
  encounter histories, given point-of-care usage.
- E-prescribing transmission acknowledgment surfaced to the provider within
  the network's SLA, with a clear pending/failed state — never a silent
  failure.
- 99.9% uptime target for scheduling and charting during the clinic's
  configured operating hours.

## 11. Architecture

ZodiMed is a standalone, self-hosted Laravel application, built by cloning
the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
The clone strips every banking-specific table that doesn't apply to a
clinical/billing product — `loans`/`loan_plans`, `dps`/`dps_plans`,
`fdr`/`fdr_plans`, `branches`/`branch_staff` and its guard, `other_banks`,
`beneficiaries`, `airtime_operators`/`airtime_configs`. ZodiMed does not
re-add a branch-scoped staff guard: its personas (Physician, Nurse, Front
Desk, Billing/Coding Staff, Practice Administrator, Compliance Officer) all
authenticate through the inherited `admin` guard, with per-location access
governed by RBAC/policy scoping (§18) rather than a separate login guard —
multi-location clinic groups model each location as a `locations` record
under the multi-company/multi-branch scoping pattern in
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping),
not as a distinct auth boundary.

ZodiMed inherits the base engine's admin settings/branding, payment
gateways (for patient billing/copay collection), RBAC/auth, KYC (repurposed
for patient identity verification where a clinic requires it), i18n, cron,
extension toggles, and CMS/page builder as-is — see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps).
On top of that inherited engine, ZodiMed builds its own domain modules
(Patient Records, Scheduling, Clinical Documentation, E-Prescribing, Lab
Integration, Billing & Claims, Patient Portal, Access Governance) per
[`module-template.md`](../../templates/module-template.md).

The one addition that goes beyond ordinary domain-module layering is a
`PHIAccessContract` service, built on top of the inherited `Role`/
`Permission` RBAC engine rather than replacing it: every PHI read is
authorized twice — once by the standard RBAC permission check, once by a
care-relationship/consent check the contract enforces at query time
(care-team scoping, break-glass justification capture). This is the
Policy-layer pattern described in
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie):
additional rules layered on the inherited RBAC engine, not a second,
competing permission system.

Each ZodiMed deployment belongs to exactly one clinic or clinic group,
running on their own hosting with their own database, with zero runtime
dependency on any other Zodize product or Zodize-operated service, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).

## 12. Technology

Laravel 11 + PHP ^8.3 per the inherited base codebase
([coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)),
with Blade/Bootstrap/jQuery for new module UI per
[coding-standards-frontend.md](../../development/coding-standards-frontend.md);
MySQL/MariaDB per the base codebase's inherited schema, matching
[database-standards.md](../../development/database-standards.md), with
field-level encryption for PHI columns per
[data-protection-privacy.md](../../security/data-protection-privacy.md);
HL7/FHIR-oriented integration layer for lab and e-prescribing connections
(§22); clearinghouse integration for claims (X12 837/835 transaction
formats).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Patient Records | Demographics, Problem List, Medication/Allergy List, Immunizations, Care Team |
| Scheduling | Appointment Calendar, Recall/Reminders, Waitlist, Multi-Location Scheduling |
| Clinical Documentation | Encounter Notes, Templates, Amendments/Addenda, Signature/Lock |
| E-Prescribing | Medication Search, Interaction Checking, Transmission, Controlled Substance Workflow |
| Lab Integration | Order Transmission, Result Ingestion, Provider Acknowledgment |
| Billing & Claims | CPT/ICD Coding, Claim Submission, Remittance Posting, Denials/Appeals |
| Patient Portal | Self-Scheduling, Secure Messaging, Results, Bill Pay, Intake Forms |
| Access Governance | Minimum-Necessary Enforcement, Break-Glass Access, Consent Tracking |

## 14. Core Data Model

Full ER diagram queued (§ Roadmap). Every entity below belongs to the one
clinic or clinic group that owns this deployment — there is no `tenant_id`
column anywhere in this schema, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`location_id` (and `company_id` for a multi-entity clinic group) provide
multi-location scoping within that one deployment, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).
Core entities:

| Entity | Key columns |
|---|---|
| `patients` | id, mrn, demographics (encrypted), primary_provider_id |
| `care_team_assignments` | id, patient_id, provider_id, role, relationship_start_at, relationship_end_at |
| `appointments` | id, patient_id, provider_id, location_id, scheduled_at, status, appointment_type |
| `encounters` | id, patient_id, provider_id, appointment_id, encounter_date, status (draft/signed/amended) |
| `clinical_notes` | id, encounter_id, note_type, body (encrypted), signed_at, signed_by, amended_from_id |
| `medications` | id, patient_id, drug_name, dose, prescriber_id, status, prescribed_at |
| `allergies` | id, patient_id, allergen, reaction, severity |
| `lab_orders` | id, encounter_id, patient_id, test_code, ordering_provider_id, status |
| `lab_results` | id, lab_order_id, result_value, flag (normal/abnormal/critical), acknowledged_by, acknowledged_at |
| `claims` | id, encounter_id, patient_id, payer_id, cpt_codes, icd_codes, status, submitted_at |
| `remittances` | id, claim_id, payer_id, paid_amount, adjustment_amount, denial_reason_code |
| `phi_access_logs` | id, patient_id, accessed_by, access_reason (care_team/break_glass/consent), justification |
| `consents` | id, patient_id, consent_type, granted_to, granted_at, revoked_at |

## 15. Key API Endpoints

Full endpoint catalog queued (§ Roadmap). Key routes, all conforming to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md):

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/patients/{patient}` | Retrieve patient chart summary (PHI-access-gated) |
| POST | `/api/v1/patients/{patient}/break-glass` | Request break-glass access with justification |
| GET | `/api/v1/appointments` | List/search appointments |
| POST | `/api/v1/appointments` | Book an appointment |
| POST | `/api/v1/encounters` | Create an encounter from a checked-in appointment |
| POST | `/api/v1/encounters/{encounter}/notes` | Add/sign a clinical note |
| POST | `/api/v1/encounters/{encounter}/notes/{note}/amend` | Amend a signed note (creates new version) |
| POST | `/api/v1/patients/{patient}/medications` | Add medication / e-prescribe |
| POST | `/api/v1/prescriptions/{prescription}/interaction-check` | Run interaction/allergy check |
| POST | `/api/v1/patients/{patient}/lab-orders` | Create a lab order |
| POST | `/api/v1/lab-results/{result}/acknowledge` | Provider acknowledges a result |
| POST | `/api/v1/claims` | Generate a claim from an encounter |
| POST | `/api/v1/claims/{claim}/submit` | Submit claim to clearinghouse |
| POST | `/api/v1/claims/{claim}/resubmit` | Resubmit a corrected claim |
| GET | `/api/v1/claims/{claim}/remittance` | View remittance/denial detail |
| GET | `/api/v1/portal/patient/appointments` | Patient portal appointment view |
| POST | `/api/v1/portal/patient/messages` | Patient secure message to provider |
| GET | `/api/v1/portal/patient/results` | Patient-released lab results |
| POST | `/api/v1/patients/{patient}/consents` | Record a patient consent |
| GET | `/api/v1/reports/denials` | Claim denial analytics |

## 16. Events

`appointment.booked`, `appointment.checked_in`, `encounter.signed`,
`encounter.amended`, `prescription.transmitted`,
`prescription.interaction_flagged`, `lab_order.transmitted`,
`lab_result.received`, `lab_result.acknowledged`, `lab_result.critical`,
`claim.submitted`, `claim.denied`, `claim.paid`, `phi.accessed`,
`phi.break_glass_used`, `consent.recorded`. See
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| Appointment reminder (T-24h) | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `lab_result.critical` (to ordering provider) | ✔ | ✔ | ✔ | ✔ |
| `lab_result.acknowledged` (result released to patient) | ✔ (portal) | ✔ | — | ✔ |
| `prescription.interaction_flagged` | ✔ (to prescriber) | — | — | — |
| `claim.denied` | ✔ (to billing staff) | ✔ | — | — |
| `phi.break_glass_used` (to compliance officer) | ✔ | ✔ | — | — |
| Patient portal secure message received | ✔ | ✔ (notification only, no PHI in body) | — | ✔ |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md);
PHI content is never included in email/SMS/push bodies, only a
notification-to-portal prompt, per §27.

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine and `admin` guard per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie).
ZodiMed's own roles: `Physician/Provider`, `Nurse/Clinical Staff`,
`Front Desk`, `Billing/Coding Staff`, `Practice Administrator`,
`Compliance Officer`, `Patient` (portal-only, scoped to own record). Key
permissions, registered into the inherited permission system per
[permission-template.md](../../templates/permission-template.md):
`charts.view` (always additionally gated by `PHIAccessContract` care-team
check — the HIPAA-equivalent minimum-necessary-access rule from §27,
implemented as an additional Policy-layer check on top of the inherited
RBAC, not a separate permission system), `charts.break_glass`,
`notes.sign`, `notes.amend`, `prescriptions.write`,
`prescriptions.write_controlled` (requires elevated/step-up
authentication), `claims.submit`, `claims.appeal`, `consents.manage`.

## 19. Workflows & Approval Chains

- **Break-glass access approval**: emergency access outside the care team
  requires a justification at request time and triggers a mandatory
  post-hoc review by the Compliance Officer within a configured SLA;
  repeated break-glass use by the same user without adequate justification
  escalates automatically.
- **Note amendment**: a signed clinical note cannot be edited in place — an
  amendment creates a new linked version, and the original remains visible
  with a clear "amended" marker, preserving the legal record.
- **Controlled substance prescribing**: requires step-up authentication
  (matching [authentication-authorization.md](../../security/authentication-authorization.md))
  at the moment of signing, distinct from the session's standing login.
- **Claim denial and appeal**: denials route to Billing/Coding Staff for
  correction and resubmission within the payer's timely-filing window;
  unresolved denials escalate to a formal appeal workflow with document
  attachment.

## 20. Audit Logs

Every PHI read, write, break-glass access, prescription, and claim action is
immutably audit-logged per [audit-logging.md](../../security/audit-logging.md),
recording actor, patient, timestamp, and — for PHI reads outside the care
team — the specific access justification, satisfying HIPAA-equivalent
accounting-of-disclosures requirements.

## 21. Reports & Analytics & Dashboards

- Provider schedule utilization and no-show rate dashboards.
- Claim denial rate, days-in-A/R, and payer performance analytics.
- Quality/outcome measure tracking dashboards (e.g. chronic disease
  management panels) where applicable to the practice's specialty.
- PHI access audit report, filterable by patient, user, and access type
  (routine/break-glass/consent), for compliance review.
- Report builder and scheduled reports per the
  [Second-Layer Feature Catalog](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **E-prescribing network**: Surescripts-class network for electronic
  prescription transmission and formulary/interaction data.
- **Clinical lab integration**: HL7/FHIR-based order and result exchange
  with reference labs (Quest/LabCorp-class) and in-house lab equipment
  interfaces.
- **Insurance clearinghouse**: X12 837 claim submission and 835 remittance
  ingestion via a clearinghouse (Availity/Change Healthcare-class).
- **Eligibility verification**: real-time payer eligibility checks at
  scheduling/check-in.
- **Immunization registries**: state immunization information system
  reporting where required.

## 23. AI Features

- AI-assisted clinical documentation: ambient/structured note drafting from
  provider dictation or structured intake, with the provider required to
  review and sign before the note is final — AI never signs on a provider's
  behalf.
- AI-assisted coding suggestions (CPT/ICD) from encounter documentation,
  surfaced to Billing/Coding Staff as a suggestion requiring human
  confirmation, never auto-submitted.
- Denial pattern analysis flagging claims likely to be denied before
  submission, built on ZodiMed's own claims and remittance history.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: appointment reminder dispatch, recall list generation,
  eligibility re-verification ahead of upcoming visits, claim timely-filing
  deadline alerts, break-glass access review SLA alerts.
- CLI commands: `med:reindex-patient-search`, `med:run-eligibility-batch`,
  `med:reconcile-remittances`, `med:audit-break-glass-review` — each
  requiring the same authorization context as its API equivalent.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a demo clinic with 3 providers across 2 locations,
150 demo patients with realistic encounter histories, a populated
scheduling calendar (past and upcoming), sample e-prescriptions and lab
orders/results (including one flagged interaction and one critical result),
and a populated claims history including at least one denial-and-resubmit
cycle — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).
All demo PHI is synthetic, never real patient data.

## 26. Performance Requirements

See §10. Additionally: interaction/allergy checks return within 1 second
p95 so they do not disrupt the provider's prescribing workflow.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiMed-specific requirements (HIPAA-equivalent):

- **Minimum-necessary access**: chart access is scoped to the patient's
  documented care team by default; access outside that scope requires
  break-glass justification or a recorded patient consent, both logged per
  §20 — matching the HIPAA minimum-necessary standard.
- **PHI encryption**: PHI fields (clinical notes, diagnoses, demographics,
  medications) are field-level encrypted at rest per
  [data-protection-privacy.md](../../security/data-protection-privacy.md);
  PHI is never transmitted in notification bodies (§17).
- **Business Associate Agreement posture**: ZodiMed's architecture and audit
  trail are built to support a BAA-equivalent trust posture for the clinic
  or clinic group operating under HIPAA or an equivalent regional
  health-privacy framework; any third-party integration (§22) handling PHI
  must itself meet this standard before activation.
- **Accounting of disclosures**: the PHI access log (§14, §20) provides a
  complete, exportable accounting of who accessed a given patient's record
  and why, on request.
- **Controlled substance prescribing**: requires step-up authentication per
  §19, distinct from standard session login.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md);
additionally a dedicated PHI access-control test suite verifying that no
code path can return chart data to a user outside the care team without a
break-glass or consent record, run as a required CI gate.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).

## 30. Acceptance Criteria

- A patient can be scheduled, checked in, seen, documented, coded, and
  billed end to end with every step's actor and PHI access logged.
- A provider outside a patient's care team cannot open that patient's chart
  without triggering and logging a break-glass justification or a recorded
  consent.
- An e-prescription cannot be transmitted past an unacknowledged
  interaction/allergy warning without an explicit provider override reason
  captured.
- A signed clinical note cannot be silently edited — only amended with full
  version history preserved.
- A denied claim can be corrected and resubmitted within the payer's
  timely-filing window with both versions retained in the audit trail.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).

## 32. Future Roadmap

- Telehealth visit module (video encounter integrated with scheduling and
  documentation).
- Inpatient/bed-management module for the small-hospital segment of the
  target market.
- Value-based care quality measure reporting expansion.

## 33. Known Risks

- PHI access-control defects carry the highest compliance and legal
  exposure in the product — mitigated by the dedicated access-control test
  suite in §28, but this remains the module requiring the most conservative
  change-review process.
- E-prescribing and lab integrations depend on external network
  availability outside Zodize's control — mitigated by clear pending/failed
  state surfacing (§10) rather than silent failure, but an extended network
  outage still degrades clinical workflow.

## 34. Future Improvements

- Structured clinical decision support (guideline-based alerts) beyond
  interaction/allergy checking.
- Direct patient-to-provider video visit billing integration.

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture (§11), Core Data Model
(§14), and Permissions & Roles (§18) sections were revised to the
standalone, self-hosted, single-tenant model in
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md);
no product-domain content (vision, personas, journeys, clinical/billing
workflows) changed. Queued for Deep-depth expansion: full ER diagram and
migration set (companion `DATA_MODEL.md`), full endpoint catalog (companion
`API_REFERENCE.md`), full HL7/FHIR message mapping detail, and a complete
quality/outcome measure report catalog beyond the summary list in §21.
