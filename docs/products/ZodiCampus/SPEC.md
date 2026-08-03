# ZodiCampus — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth). See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for spec status
> definitions.

## 1. Vision

ZodiCampus is a student information system that runs a school or campus'
entire academic lifecycle — admissions through transcript issuance — as one
system of record, so registrars, faculty, financial aid staff, and families
share one accurate view of a student's enrollment, academic progress, and
account balance instead of reconciling separate admissions, gradebook, and
billing tools.

## 2. Purpose

Campuses run on three systems that are supposed to agree and usually don't:
an admissions/CRM tool, a gradebook/LMS-adjacent tool, and a billing/aid
system. Disagreements between them show up as wrong transcripts, financial
holds on students who actually paid, and FERPA exposure when the wrong
person's records leak across systems that were never designed to share a
consistent access model. ZodiCampus exists to make enrollment, academics,
and billing one consistent record with FERPA-equivalent privacy enforced by
the data model, not policy alone.

## 3. Target Market

K-12 private/charter school networks, higher-education institutions
(colleges, universities, community colleges), and vocational/professional
training programs that need enrollment, registration, gradebook,
attendance, billing, and financial aid in one system.

## 4. Industries

Private and charter K-12 education, higher education (2-year and 4-year),
vocational and professional certification programs, and continuing
education divisions.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Student information system core | PowerSchool, Ellucian Banner, Colleague | Modern Laravel/Vue stack instead of legacy SIS platforms with decade-old UX |
| Course registration/scheduling | Ellucian Banner, Workday Student | Registration built on the same rule/workflow engine used across every Zodize product |
| Gradebook/transcript management | PowerSchool, Infinite Campus | Version-controlled transcript history with full audit trail for grade changes |
| Tuition billing/financial aid | Nelnet Campus Commerce, Ellucian | Billing and aid share one ledger with enrollment status, avoiding the "paid but still on hold" failure mode |
| Parent/guardian portal | ParentSquare, PowerSchool Parent Portal | Same portal shell/design system as every other Zodize product, lower support burden for multi-child, multi-school families |

## 6. Personas

- **Registrar/Admissions Officer** — manages applications, admissions
  decisions, enrollment, and official records.
- **Faculty/Instructor** — manages course sections, gradebook, attendance,
  and course materials.
- **Academic Advisor** — reviews student progress, holds, and degree/program
  requirements.
- **Financial Aid/Bursar Staff** — manages tuition billing, aid packaging,
  and payment plans.
- **Student** — registers for courses, views grades/transcripts, manages
  billing via the student portal.
- **Parent/Guardian** — views permitted student information (per consent/age
  rules) and manages payments via the guardian portal.

## 7. User Journeys

1. **Admissions to enrollment**: Prospective student applies via the
   admissions portal → application reviewed with supporting documents
   (transcripts, test scores) → admissions decision recorded → accepted
   applicant completes enrollment (deposit, forms, course interest) →
   student record is provisioned with a student ID and program assignment →
   registration window opens for the student.
2. **Course registration**: Student (or advisor, for advised programs)
   browses the course catalog for the term → checks prerequisite and
   corequisite requirements (system-enforced, with an advisor override path)
   → registers for sections respecting seat capacity and schedule conflicts
   → financial or academic holds (§19) block registration until resolved →
   registration confirmation updates the student's schedule and generates
   the term's tuition charge.
3. **Grading and transcript issuance**: Instructor enters grades for a
   section by the term's grading deadline → grades are submitted and locked
   (a post-lock change requires a documented grade-change request, not a
   silent edit) → term GPA and cumulative GPA recalculate → official
   transcript reflects the posted grades → student or a third party (with
   the student's consent on file) requests an official transcript, issued
   as a verifiable PDF.
4. **Tuition billing and financial aid disbursement**: Term charges post to
   the student account → financial aid award (grant/loan/scholarship)
   applies against the balance → remaining balance is billed to the
   student/guardian, payable via the portal or a payment plan → aid
   disbursement posts on the institution's disbursement schedule →
   unresolved balances past the due date trigger a financial hold that
   blocks registration and transcript release until resolved.
5. **FERPA-scoped guardian access**: A parent/guardian requests portal
   access to their student's record → for a K-12 student, access is granted
   by default per the guardian relationship on file; for a higher-education
   student who has reached the age of majority, access requires the
   student's explicit FERPA consent/release on file before any guardian
   view is permitted — the system enforces this distinction automatically
   based on the student's record type and consent status.

## 8. Business Goals

- Eliminate the enrollment/billing/academic-record reconciliation gap that
  causes wrongful holds and transcript errors.
- Give registrars and advisors real-time visibility into holds,
  prerequisites, and degree/program progress at the point of registration,
  not after the fact.
- Make FERPA-equivalent consent and access-scoping enforcement automatic
  rather than a manual policy registrars must remember to apply per family.
- Increase parent/guardian portal engagement (payments, grade visibility
  where permitted) as a retention and satisfaction driver for K-12 school
  buyers.

## 9. Functional Requirements

- Admissions: application intake, document upload, decision workflow,
  enrollment deposit and confirmation.
- Student records: demographics, program/major assignment, academic
  standing, holds, document vault (transcripts, test scores, immunization
  records where applicable).
- Course catalog and registration: term/section management, prerequisite/
  corequisite enforcement, seat capacity, waitlists, schedule conflict
  detection.
- Gradebook: assignment/category weighting, grade entry, grade submission
  and lock, grade-change request workflow.
- Transcripts: unofficial and official transcript generation, GPA
  calculation (term and cumulative), verifiable PDF issuance.
- Attendance: session/period-level attendance capture, absence
  threshold alerting, attendance-linked reporting for compliance
  (e.g. financial aid satisfactory-attendance rules).
- Tuition billing: charge generation by term, payment plans, financial aid
  award application, refund processing, financial hold management.
- Faculty portal: section roster, gradebook, attendance entry, course
  materials/announcements.
- Parent/guardian portal: FERPA-consent-scoped access to grades, attendance,
  and billing; payment management.
- Student portal: registration, grades/transcript access, billing, document
  requests.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiCampus-specific additions:

- Registration endpoints must remain responsive under registration-window
  concurrency spikes (many students registering for limited seats
  simultaneously) — seat-capacity checks use row-level locking to prevent
  overbooking, p95 < 500ms even under peak registration load.
- Transcript generation (official PDF) completes in under 5 seconds p95.
- Gradebook submission for a section of up to 300 students completes in
  under 3 seconds p95.

## 11. Architecture

ZodiCampus is a standalone, self-hosted Laravel application, built by
cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
The clone strips every banking-specific table that doesn't apply to a
student-information system — `loans`/`loan_plans`, `dps`/`dps_plans`,
`fdr`/`fdr_plans`, `branches`/`branch_staff` and its guard, `other_banks`,
`beneficiaries`, `airtime_operators`/`airtime_configs`. ZodiCampus does not
re-add a branch-scoped staff guard: its personas (Registrar, Faculty,
Advisor, Financial Aid/Bursar Staff) all authenticate through the inherited
`admin` guard, with per-campus access governed by RBAC/policy scoping (§18)
rather than a separate login guard — a multi-campus school network or
higher-ed institution with multiple locations models each campus as a
`campuses` record under the multi-company/multi-branch scoping pattern in
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping),
not as a distinct auth boundary.

ZodiCampus inherits the base engine's admin settings/branding, payment
gateways (for tuition/payment-plan billing), wallet/ledger (for student
account balances), RBAC/auth, i18n, cron, extension toggles, and CMS/page
builder as-is — see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps).
On top of that inherited engine, ZodiCampus builds its own domain modules
(Admissions, Student Records, Registration, Academics, Attendance, Billing &
Aid, Faculty Portal, Family Portal) per
[`module-template.md`](../../templates/module-template.md).

The one addition that goes beyond ordinary domain-module layering is a
`FERPAConsentContract` service, built on top of the inherited `Role`/
`Permission` RBAC engine rather than replacing it: guardian and
third-party access to a student's record is scoped by the student's
age-of-majority status and any consent/release on file, evaluated at query
time in addition to the standard RBAC permission check. This is the
Policy-layer pattern described in
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie):
an additional rule layered on the inherited RBAC engine, not a second,
competing permission system.

Each ZodiCampus deployment belongs to exactly one school, campus, or
school network, running on their own hosting with their own database, with
zero runtime dependency on any other Zodize product or Zodize-operated
service, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).

## 12. Technology

Laravel 11 + PHP ^8.3 per the inherited base codebase
([coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)),
with Blade + Bootstrap 5 + jQuery for new module UI per
[coding-standards-laravel-frontend.md](../../development/coding-standards-laravel-frontend.md);
MySQL/MariaDB per the base codebase's inherited schema, matching
[database-standards.md](../../development/database-standards.md), with
row-level locking for seat-capacity-constrained registration writes; payment
processing via the inherited base codebase's payment gateway abstraction
(see [payment-gateways.md](../../standards/payment-gateways.md)).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Admissions | Applications, Document Intake, Decisions, Enrollment Confirmation |
| Student Records | Demographics, Program/Major, Academic Standing, Holds, Document Vault |
| Registration | Course Catalog, Sections/Terms, Prerequisites, Waitlists, Schedule Builder |
| Academics | Gradebook, Grade Change Requests, Transcripts, GPA Calculation |
| Attendance | Session/Period Attendance, Absence Alerting |
| Billing & Aid | Tuition Charges, Payment Plans, Financial Aid Awards, Disbursements, Holds |
| Faculty Portal | Roster, Gradebook Entry, Attendance Entry, Announcements |
| Family Portal | Guardian Access (FERPA-scoped), Student Self-Service, Payments |

## 14. Core Data Model

Full ER diagram queued (§ Roadmap). Every entity below belongs to the one
school, campus, or school network that owns this deployment — there is no
`tenant_id` column anywhere in this schema, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`campus_id` provides multi-campus scoping within that one deployment, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).
Core entities:

| Entity | Key columns |
|---|---|
| `applicants` | id, campus_id, program_id, application_status, decision, decided_at |
| `students` | id, campus_id, student_number, program_id, enrollment_status, is_minor |
| `guardians` | id, student_id, guardian_user_id, relationship, ferpa_consent_on_file |
| `terms` | id, campus_id, name, start_date, end_date, registration_opens_at |
| `courses` | id, campus_id, code, title, credit_hours, prerequisite_course_ids |
| `sections` | id, course_id, term_id, instructor_id, seat_capacity, seats_filled |
| `registrations` | id, student_id, section_id, status (enrolled/waitlisted/dropped), registered_at |
| `grades` | id, registration_id, grade_value, submitted_by, submitted_at, locked_at |
| `grade_change_requests` | id, grade_id, requested_by, reason, new_value, approved_by |
| `attendance_records` | id, registration_id, session_date, status (present/absent/excused) |
| `holds` | id, student_id, hold_type (financial/academic/document), placed_by, resolved_at |
| `tuition_charges` | id, student_id, term_id, charge_type, amount, due_date |
| `financial_aid_awards` | id, student_id, term_id, award_type, amount, disbursed_at |

## 15. Key API Endpoints

Full endpoint catalog queued (§ Roadmap). Key routes, all conforming to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md):

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/v1/applications` | Submit an admissions application |
| POST | `/api/v1/applications/{application}/decision` | Record admissions decision |
| POST | `/api/v1/students/{student}/enroll` | Confirm enrollment from an accepted application |
| GET | `/api/v1/terms/{term}/course-catalog` | Browse course catalog for a term |
| POST | `/api/v1/registrations` | Register a student for a section |
| DELETE | `/api/v1/registrations/{registration}` | Drop a registration |
| GET | `/api/v1/students/{student}/holds` | List active holds |
| POST | `/api/v1/sections/{section}/grades` | Submit gradebook entries |
| POST | `/api/v1/grades/{grade}/change-request` | Request a post-lock grade change |
| GET | `/api/v1/students/{student}/transcript` | Generate unofficial/official transcript |
| POST | `/api/v1/sections/{section}/attendance` | Record attendance for a session |
| GET | `/api/v1/students/{student}/account` | Student billing account summary |
| POST | `/api/v1/students/{student}/financial-aid-awards` | Record a financial aid award |
| POST | `/api/v1/students/{student}/payments` | Process a tuition payment |
| POST | `/api/v1/students/{student}/guardians/{guardian}/consent` | Record/update FERPA consent |
| GET | `/api/v1/portal/student/schedule` | Student portal schedule view |
| GET | `/api/v1/portal/guardian/{student}/summary` | Guardian portal view (consent-gated) |
| GET | `/api/v1/faculty/sections/{section}/roster` | Faculty portal section roster |
| GET | `/api/v1/reports/registration-demand` | Section demand/capacity report |

## 16. Events

`application.submitted`, `application.decided`, `student.enrolled`,
`registration.created`, `registration.dropped`, `waitlist.promoted`,
`grade.submitted`, `grade.locked`, `grade_change.approved`,
`hold.placed`, `hold.resolved`, `charge.posted`, `aid.disbursed`,
`payment.received`, `consent.recorded`. See
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `application.decided` | — | ✔ | — | — |
| Registration window opening (T-3 days) | ✔ | ✔ | — | ✔ |
| `waitlist.promoted` | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `hold.placed` | ✔ | ✔ | — | — |
| `grade.submitted` (term grades posted) | ✔ | ✔ | — | ✔ |
| Absence threshold reached | ✔ (to guardian, if consented) | ✔ | — | — |
| Tuition payment due (T-7 days) | ✔ | ✔ | ✔ (opt-in) | ✔ |
| `aid.disbursed` | ✔ | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md);
guardian-channel notifications are suppressed entirely absent a valid
consent record for students past the age of majority, per §27.

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine and `admin` guard per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie).
ZodiCampus's own roles: `Registrar`, `Admissions Officer`,
`Faculty/Instructor`, `Academic Advisor`, `Financial Aid/Bursar Staff`,
`Student` (portal-only, scoped to own record), `Guardian` (portal-only,
FERPA-consent-scoped). Key permissions, registered into the inherited
permission system per
[permission-template.md](../../templates/permission-template.md):
`applications.decide`, `registrations.manage` (Registrar/Advisor),
`grades.submit` (section instructor of record only, gated additionally by
the `FERPAConsentContract` — the FERPA-equivalent access-control need is an
additional Policy-layer rule on top of the inherited RBAC, not a separate
system), `grades.change_post_lock` (requires approval, §19), `holds.manage`,
`financial_aid.award`, `transcripts.issue_official`.

## 19. Workflows & Approval Chains

- **Prerequisite override**: registration blocked by an unmet prerequisite
  can be overridden only by an Academic Advisor or Registrar, with the
  override reason logged against the registration record.
- **Grade change request**: a post-lock grade change requires the
  instructor's documented reason and Registrar approval before the
  transcript reflects the new grade; the original submitted grade remains
  visible in history.
- **Financial hold resolution**: a financial hold blocking registration or
  transcript release is auto-lifted only when the triggering balance
  condition clears (payment posts or a payment plan is approved) — never
  manually dismissed without a linked resolving transaction, to prevent
  bypass.
- **FERPA consent/release approval**: a student's release of records to a
  guardian or third party is a first-class, revocable consent record,
  reviewable in the student portal at any time.

## 20. Audit Logs

Every grade entry, grade change, hold placement/resolution, consent change,
and guardian record access is immutably audit-logged per
[audit-logging.md](../../security/audit-logging.md), including the actor,
the student affected, and — for guardian/third-party access — the specific
consent record relied upon, supporting FERPA-equivalent recordkeeping.

## 21. Reports & Analytics & Dashboards

- Enrollment funnel (applications → decisions → confirmed enrollment)
  dashboards.
- Section demand/capacity and waitlist analytics for registration planning.
- Academic standing, GPA distribution, and at-risk (attendance/grade
  threshold) dashboards for advisors.
- Tuition receivables aging and financial aid disbursement reporting.
- FERPA consent/access audit report for compliance review.
- Report builder and scheduled reports per the
  [Second-Layer Feature Catalog](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Standardized testing/transcript exchange**: electronic transcript
  exchange networks (Parchment/National Student Clearinghouse-class) for
  inbound prior-institution transcripts and outbound official transcript
  delivery.
- **Payment processing**: tuition and payment-plan processing via the
  inherited base codebase's payment gateway abstraction.
- **Financial aid systems**: federal/state aid system integration for
  higher-education institutions (e.g. FAFSA-adjacent data exchange where
  applicable to the institution's jurisdiction).
- **LMS integration**: grade passback and roster sync with common learning
  management systems (Canvas/Moodle-class) for institutions running a
  separate LMS alongside ZodiCampus.
- **SIS data exchange**: standard education data interchange formats for
  district/network-level reporting (K-12 school networks).

## 23. AI Features

- AI-assisted advising: flags at-risk students (attendance/grade trend) to
  advisors before a hold would otherwise be triggered, with the advisor
  confirming any outreach action.
- AI-assisted transcript/document review during admissions, extracting
  structured data (courses, grades) from uploaded transcripts for
  Admissions Officer review — never an auto-decision.
- Anomaly detection on grade-change and access-record patterns, built on
  ZodiCampus's own audit log data per
  [audit-logging.md](../../security/audit-logging.md).

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: registration window open/close processing, waitlist
  promotion, term charge generation, financial aid disbursement batch
  processing, absence threshold evaluation, grade submission deadline
  reminders.
- CLI commands: `campus:open-registration`, `campus:process-waitlists`,
  `campus:generate-term-charges`, `campus:disburse-aid-batch` — each
  requiring the same authorization context as its API equivalent.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions a demo campus with 2 programs, 6 terms of history,
300 demo students across enrolled/graduated/withdrawn statuses, a populated
course catalog with prerequisite chains, gradebook and transcript history
including one grade-change request, attendance history including a
threshold-triggered alert, and billing/aid history including one financial
hold and its resolution — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10. Additionally: waitlist promotion processing for a section
completes within 60 seconds of a seat opening, so promoted students receive
timely notice within the registration window.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. ZodiCampus-specific requirements (FERPA-equivalent):

- **Consent-scoped record access**: guardian and third-party access to a
  student's education records is scoped by the student's age-of-majority
  status and any consent/release on file, enforced by the
  `FERPAConsentContract` at query time — not merely by portal role
  assignment. Access without a valid basis (guardian relationship for a
  minor, or explicit consent for an adult student) is denied and logged as
  a denied attempt.
- **Directory information handling**: fields designated as directory
  information are configurable per the institution's own policy and
  excludable per student opt-out, distinct from the broader consent-gated
  record set.
- **Data segregation for third-party requesters**: transcript/record
  requests from third parties (e.g. another institution) require the same
  consent basis as guardian access and are logged identically.
- **Retention and disposal**: student records follow the institution's
  configured retention policy per
  [data-protection-privacy.md](../../security/data-protection-privacy.md),
  respecting jurisdiction-specific minimum retention for academic records.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md);
additionally a dedicated FERPA-consent access-control test suite verifying
that no code path returns a student's non-directory record data to a
guardian or third party without a valid consent/relationship record, run as
a required CI gate.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).

## 30. Acceptance Criteria

- A prospective student can apply, be decided on, and enroll, with a
  student record, course access, and billing account provisioned end to
  end.
- Registration correctly enforces prerequisites, seat capacity, and holds,
  with no code path able to overbook a section under concurrent registration
  load.
- A grade cannot be changed after lock without a documented, approved
  grade-change request, and the original value remains in history.
- A guardian of an adult student cannot view that student's grades or
  billing without a valid FERPA consent record on file.
- A financial hold correctly blocks registration and transcript release
  until the triggering balance is resolved, and lifts automatically on
  resolution.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).

## 32. Future Roadmap

- Degree audit / program-requirement completion tracking module.
- Alumni relations and advancement (donor) module extension.
- Expanded standardized-testing intake automation.

## 33. Known Risks

- FERPA consent-scoping defects carry direct legal exposure for the schools
  and institutions running this product — mitigated by the dedicated
  access-control test suite in §28, but this remains the module requiring
  the most conservative change-review process.
- Registration-window concurrency is a recurring real-world failure point
  for SIS platforms (overbooking, race conditions) — mitigated by
  row-level locking per §12, but load testing ahead of each deployment's
  registration windows remains an operational requirement.

## 34. Future Improvements

- Predictive at-risk scoring beyond simple attendance/grade thresholds.
- Configurable degree-audit rule builder for complex multi-major programs.

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture (§11), Core Data Model
(§14), and Permissions & Roles (§18) sections were revised to the
standalone, self-hosted, single-tenant model in
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md);
no product-domain content (vision, personas, journeys, academic/billing
workflows) changed. Queued for Deep-depth expansion: full ER diagram and
migration set (companion `DATA_MODEL.md`), full endpoint catalog (companion
`API_REFERENCE.md`), full degree-audit rule engine design, and a complete
report catalog beyond the summary list in §21.
