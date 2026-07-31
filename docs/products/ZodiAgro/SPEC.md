# ZodiAgro — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; deep artifacts (full ER diagram, exhaustive endpoint
> catalog, full report catalog) are queued — see
> [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiAgro is the operations system of record for farm and agribusiness
enterprises — mapping every field, planning every crop cycle, logging every
input application, tracking every harvest, and producing the traceability
records regulators and buyers demand — built as a standalone, self-hosted
Laravel application from the shared Zodize base codebase
([base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)),
so a multi-farm operator gets a working admin back office, RBAC, and
multi-entity data scoping without a separate ag-tech vendor relationship.

## 2. Purpose

Mid-size to large farm operations and agribusinesses currently stitch
together a patchwork of spreadsheets, paper spray logs, a separate
livestock app, and a compliance binder for audits. This patchwork breaks
down at scale and creates real regulatory and food-safety exposure.
ZodiAgro exists to unify field operations, input/output tracking, and
compliance record-keeping in one system so an operation can answer "what
was applied to this field, when, by whom, and does it clear the buyer's
residue tolerance" in seconds, not a records search.

## 3. Target Market

Row-crop and specialty-crop farm operations (200–50,000+ acres),
diversified farms combining crop and livestock production, and
agribusiness cooperatives/aggregators managing operations across multiple
grower entities.

## 4. Industries

Agriculture — row crops, specialty/produce crops, mixed crop-livestock
operations, and agricultural cooperatives.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Farm management software | Granular, Climate FieldView | Unified with livestock and compliance modules, not crop-only |
| Precision ag / field mapping | John Deere Operations Center | Vendor-neutral IoT/telematics ingestion, not locked to one equipment brand |
| Livestock management | AgriWebb | Same platform as crop planning — one operation, one system, not two logins |
| Farm-to-table traceability | HarvestMark, Trace Genomics (partial) | Traceability built into the core input/harvest data model, not a bolt-on compliance module |
| Agribusiness ERP | Conservis | Enterprise RBAC, audit trail, and multi-entity data scoping inherited from the base engine from day one |

## 6. Personas

- **Farm Owner/Operator** — oversees the whole operation's profitability,
  compliance posture, and multi-season planning.
- **Farm Manager** — plans crop rotations, assigns field work, approves
  input applications.
- **Field Technician/Crew Lead** — logs planting, input application,
  scouting observations, and harvest data from the field.
- **Livestock Manager** — manages herd/flock records, health treatments,
  and feed operations.
- **Agronomist/Consultant** — reviews field and soil data, recommends
  input programs, monitored via read/advisory access.
- **Equipment Operator** — logs machinery usage and requests maintenance.
- **Compliance/QA Officer** — prepares traceability and audit packages for
  buyers, certifiers (e.g. organic), and regulators.

## 7. User Journeys

1. **Season planning to planting**: Farm Manager maps fields (boundaries,
   soil zones) → creates a crop plan assigning crop/variety and rotation
   history per field for the season → generates a planting schedule →
   Field Technician logs actual planting date, seed lot, and rate against
   the plan, creating the field's traceability record from day one.
2. **Input application logging**: Field Technician applies a fertilizer or
   pesticide product → logs the application (product, rate, field, block,
   applicator, weather conditions at time of application, pre-harvest
   interval) directly from a mobile device → the system checks the
   application against the product's label restrictions and the field's
   next scheduled harvest date, flagging a pre-harvest-interval conflict
   before it becomes a compliance violation.
3. **Harvest to yield tracking**: harvest crew logs harvested quantity per
   field/block, linked to the load ticket and destination (storage bin,
   direct sale, processor) → yield is calculated against planted acreage →
   Farm Manager reviews yield variance against the season's plan and prior
   seasons for the same field to inform next season's rotation.
4. **Livestock health event to withdrawal tracking**: Livestock Manager
   logs a veterinary treatment for an animal or group → system records the
   drug's meat/milk withdrawal period → animal/group is flagged as
   withheld from sale/processing until the withdrawal period clears,
   preventing an accidental compliance violation at sale.
5. **Buyer traceability request**: a buyer or auditor requests the full
   input and handling history for a shipped lot → Compliance Officer runs
   a traceability report from the shipped lot backward through
   harvest → field → every input application and equipment pass on that
   field during the season, exporting a compliance-ready PDF in minutes
   instead of assembling it from paper logs.
6. **Equipment scheduling conflict**: two field operations are scheduled to
   use the same sprayer on overlapping windows → the equipment scheduling
   module flags the conflict when the second job is scheduled, prompting
   the Farm Manager to reschedule or assign a second machine before the
   conflict causes a field delay.

## 8. Business Goals

- Give an operation a single source of truth for field, input, harvest, and
  livestock data, replacing spreadsheets and paper logs.
- Reduce time to produce a buyer/auditor traceability package from days to
  minutes.
- Reduce compliance violations (pre-harvest interval, withdrawal period)
  through proactive, system-enforced flagging at the point of data entry.
- Improve season-over-season decision-making via yield and input-cost
  tracking per field.

## 9. Functional Requirements

- **Field/plot mapping**: field boundary mapping (GeoJSON/polygon storage),
  soil zone/sub-block definition, field grouping by farm/entity.
- **Crop planning & rotation**: season crop plans per field, variety
  selection, rotation history tracking, rotation-conflict warnings (e.g.
  disease-prone repeat planting).
- **Input tracking**: seed lot logging, fertilizer/pesticide application
  logs (product, rate, method, applicator, weather, PHI/REI tracking),
  label-restriction checks.
- **Harvest & yield tracking**: harvest event logging per field/block, load
  ticket linkage, yield-per-acre calculation, destination tracking
  (storage, direct sale, processor).
- **Livestock management**: animal/group records, health/treatment
  history, withdrawal-period tracking, feed logs, breeding/reproduction
  records, movement/location history.
- **Weather/IoT sensor integration**: field-level weather data ingestion,
  soil moisture/temperature sensor feeds, irrigation controller
  integration, alerting on threshold breaches (frost, drought stress).
- **Compliance/traceability**: farm-to-table traceable chain from field
  input through harvest to shipped lot, certifier-ready export (e.g.
  organic certification recordkeeping), regulatory report generation.
- **Equipment/machinery scheduling**: equipment registry, job scheduling
  with conflict detection, usage/hours logging, maintenance work orders.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (input-application approval above a cost/risk threshold),
  automation rules (e.g. auto-flag withdrawal conflicts), saved
  field/harvest filters, custom fields on field and animal records, full
  audit history, soft-delete/restore, mass actions (bulk input logging
  across fields), command palette, report builder, scheduled compliance
  reports.

## 10. Non-Functional Requirements

Baseline from [performance-standards.md](../../quality/performance-standards.md)
and [security-standards.md](../../security/security-standards.md) applies.
ZodiAgro-specific: the mobile field-logging interface must function with
degraded/offline connectivity (common in rural fields) and sync when
connectivity returns, per an offline-first sync design; IoT sensor ingestion
must sustain sustained write throughput from thousands of field sensors
without degrading interactive dashboard query latency (p95 dashboard load
< 2s even during peak sensor ingestion).

## 11. Architecture

ZodiAgro is a standalone, self-hosted Laravel application, sold as source
code to one farm operation or cooperative and deployed entirely within that
buyer's own hosting account — there is no shared platform service and no
other Zodize product it depends on at runtime
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
It is built by cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific tables (`loans`, `dps`, `fdr`, `branches`/
`branch_staff`, `other_banks`, `beneficiaries`, `airtime_operators`) are
stripped, and ZodiAgro's own domain modules — field management, crop
planning, input tracking, harvest, livestock, weather/IoT, compliance &
traceability, equipment — are built on top of the inherited engine's
wallet/ledger, payment gateways, RBAC/auth, KYC, i18n, and admin
configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)).

A cooperative/aggregator buyer managing several grower entities under one
deployment scopes them via `company_id`, following
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
— this is data scoping within one buyer's one deployment, not multi-tenancy;
there is no `tenant_id` anywhere in ZodiAgro's schema, and a cooperative's
member-grower entities are not isolated from one another the way separate
SaaS customer organizations would be, since they all belong to the one
buyer who purchased and deployed this instance. IoT/sensor and weather
ingestion runs through a dedicated write-optimized time-series pipeline
(queued, batched inserts) decoupled from the primary transactional data
model, per
[caching-queues-events.md](../../architecture/caching-queues-events.md), so
high-frequency sensor writes never contend with interactive field/harvest
transactions.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL with PostGIS extension for field boundary/geospatial data, plus
Redis, per [database-standards.md](../../development/database-standards.md);
a time-series-optimized table strategy for sensor/weather data; offline-
capable mobile field app built on the shared Vue component library with a
local-first sync layer.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Field Management | Field/Plot Mapping, Soil Zones, Field Grouping |
| Crop Planning | Season Plans, Variety Selection, Rotation History, Rotation Conflicts |
| Input Tracking | Seed Lots, Fertilizer/Pesticide Application Logs, Label Restriction Checks, PHI/REI Tracking |
| Harvest | Harvest Events, Load Tickets, Yield Calculation, Destination Tracking |
| Livestock | Animal/Group Records, Health/Treatment History, Withdrawal Tracking, Feed Logs, Breeding Records |
| Weather & IoT | Weather Feed Ingestion, Soil Sensor Feeds, Irrigation Controller Integration, Threshold Alerting |
| Compliance & Traceability | Chain-of-custody Tracking, Certifier Recordkeeping, Regulatory Exports |
| Equipment | Equipment Registry, Job Scheduling, Usage/Hours Logging, Maintenance Work Orders |
| Reporting | Yield Analytics, Input Cost Tracking, Compliance Dashboards, Report Builder |

## 14. Core Data Model

All tables belong to the one buyer's one deployment — there is no
`tenant_id` column anywhere in this model
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
A cooperative operating several grower entities scopes them via
`company_id`, following
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).

| Entity | Key columns | Notes |
|---|---|---|
| `farms` | id, company_id, name, location | A grower entity, scoped to the owning company |
| `fields` | id, farm_id, name, boundary_geo, acreage, soil_zone | PostGIS polygon boundary |
| `crop_plans` | id, field_id, season, crop, variety, planned_planting_date | One per field per season |
| `planting_events` | id, crop_plan_id, actual_planting_date, seed_lot_id, rate | Actual planting record |
| `input_applications` | id, field_id, product_name, rate, method, applicator_id, applied_at, weather_snapshot, phi_days | Fertilizer/pesticide log, traceability anchor |
| `harvest_events` | id, field_id, crop_plan_id, harvested_at, quantity, unit, load_ticket_ref, destination | Yield source record |
| `animals` | id, farm_id, tag_id, species, group_id, birth_date, status | Individual or group-tracked livestock |
| `treatments` | id, animal_id_or_group_id, drug_name, administered_at, withdrawal_until, administered_by | Health event with withdrawal tracking |
| `feed_logs` | id, group_id, feed_type, quantity, logged_at | Livestock feed tracking |
| `sensors` | id, field_id, sensor_type, external_device_id, status | Soil moisture/weather/IoT device registry |
| `sensor_readings` | id, sensor_id, reading_type, value, recorded_at | Time-series data, high write volume |
| `equipment` | id, farm_id, name, type, status | Machinery registry |
| `equipment_jobs` | id, equipment_id, field_id, scheduled_start, scheduled_end, operator_id, status | Scheduling with conflict detection |
| `traceability_lots` | id, harvest_event_id, lot_code, shipped_to, shipped_at | Terminal node of the traceability chain |
| `compliance_records` | id, related_entity_type, related_entity_id, certifier, record_type, file_ref | Certifier/regulatory recordkeeping |

## 15. Key API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/farms/{farm}/fields` | List fields with boundaries and current crop |
| POST | `/api/v1/farms/{farm}/fields` | Create a field with boundary geometry |
| POST | `/api/v1/fields/{field}/crop-plans` | Create a season crop plan for a field |
| POST | `/api/v1/crop-plans/{plan}/planting-events` | Log an actual planting event |
| POST | `/api/v1/fields/{field}/input-applications` | Log an input application, runs label/PHI check |
| GET | `/api/v1/fields/{field}/input-applications` | Field's input application history |
| POST | `/api/v1/fields/{field}/harvest-events` | Log a harvest event |
| GET | `/api/v1/farms/{farm}/yield-report` | Yield-per-acre report across fields/seasons |
| POST | `/api/v1/farms/{farm}/animals` | Register an animal or livestock group |
| POST | `/api/v1/animals/{animal}/treatments` | Log a treatment, computes withdrawal window |
| GET | `/api/v1/animals/{animal}/withdrawal-status` | Current sale/processing eligibility |
| POST | `/api/v1/fields/{field}/sensors` | Register a sensor on a field |
| POST | `/api/v1/webhooks/sensors/{sensor}/readings` | Inbound sensor/telemetry data receiver |
| GET | `/api/v1/fields/{field}/weather` | Field-level weather history/forecast |
| POST | `/api/v1/farms/{farm}/equipment/{equipment}/jobs` | Schedule an equipment job, checks conflicts |
| GET | `/api/v1/traceability-lots/{lot}/chain` | Full traceability chain for a shipped lot |
| POST | `/api/v1/traceability-lots/{lot}/export` | Generate a compliance-ready traceability export |
| GET | `/api/v1/farms/{farm}/compliance-records` | List compliance/certifier records |
| POST | `/api/v1/compliance-records` | Attach a new compliance record to an entity |
| GET | `/api/v1/farms/{farm}/reports/input-costs` | Input cost analytics per field/season |

## 16. Events

`field.created`, `crop_plan.created`, `planting_event.logged`,
`input_application.logged`, `input_application.phi_conflict_detected`,
`harvest_event.logged`, `traceability_lot.shipped`, `animal.registered`,
`treatment.logged`, `treatment.withdrawal_conflict_detected`,
`sensor.threshold_breached`, `equipment_job.scheduled`,
`equipment_job.conflict_detected`, `compliance_record.attached`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `input_application.phi_conflict_detected` | ✔ (farm mgr) | ✔ | ✔ | ✔ |
| `treatment.withdrawal_conflict_detected` | ✔ (livestock mgr) | ✔ | ✔ | ✔ |
| `sensor.threshold_breached` (frost/drought) | ✔ | ✔ | ✔ | ✔ |
| `equipment_job.conflict_detected` | ✔ (farm mgr) | — | — | ✔ |
| `harvest_event.logged` (season summary) | ✔ | — | — | — |
| `compliance_record.expiring_soon` | ✔ (compliance officer) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits the base codebase's default admin roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles)),
scoped per farm/entity via `company_id`. ZodiAgro-specific permissions: `fields.manage`,
`crop_plans.manage`, `input_applications.log`,
`input_applications.override_phi_flag` (elevated — requires Farm Manager+),
`harvest.log`, `livestock.manage`, `treatments.log`, `equipment.schedule`,
`compliance.manage`, `traceability.export`. Field Technician role by default
can log data but cannot override a PHI/withdrawal conflict flag.

## 19. Workflows & Approval Chains

- **PHI/withdrawal override approval**: overriding a pre-harvest-interval or
  withdrawal-period conflict flag requires Farm Manager or Livestock
  Manager approval with a documented reason code, per
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **High-cost input approval**: input applications above a configurable
  per-field cost threshold require Farm Manager pre-approval before the
  application is logged as planned.
- **Certifier recordkeeping sign-off**: organic/certifier-scoped compliance
  records require Compliance Officer sign-off before being included in an
  export package.

## 20. Audit Logs

Every input application, treatment, harvest event, equipment job, and
compliance record attachment is recorded to the deployment's audit log with
actor, timestamp, before/after values, and field/farm scope — per
[audit-logging.md](../../security/audit-logging.md). Traceability lots carry
an immutable, append-only chain-of-custody log distinct from general audit
history, since it is the artifact shown to buyers/auditors.

## 21. Reports & Analytics & Dashboards

Standard dashboards (per
[dashboard-standards.md](../../standards/dashboard-standards.md)):
season yield-per-acre by field/crop, input cost per acre, PHI/withdrawal
compliance status board, equipment utilization, sensor/weather trend
charts. Report builder supports custom compliance and yield reports, saved
and scheduled per
[product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Weather APIs**: third-party weather data providers for field-level
  forecasts and historical weather snapshots attached to input applications.
- **IoT/sensor hardware**: soil moisture, temperature, and irrigation
  controller vendors via a normalized `SensorConnectorContract`.
- **Precision ag equipment telemetry**: optional ingestion from farm
  equipment telematics (planting/harvest machine data) where the operator's
  equipment vendor exposes an API.
- **Commodity/grain marketing systems**: optional export of harvest/yield
  data to grain marketing or ERP systems.
- **Certifier/regulatory portals**: export formats compatible with common
  organic certifier and regulatory reporting requirements.

## 23. AI Features

- **Rotation and input recommendations**: AI-assisted crop rotation and
  input-program suggestions based on field history, soil data, and yield
  outcomes, surfaced as recommendations to the Farm Manager/Agronomist,
  never auto-applied.
- **Compliance risk scoring**: flags fields or animals with an elevated
  compliance risk pattern (e.g. repeated near-miss PHI conflicts) for
  proactive review.
- **Yield forecasting**: mid-season yield forecast based on planting data,
  weather trends, and historical field performance.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: nightly weather data refresh, sensor threshold evaluation,
  withdrawal-period expiry checks (auto-clear animals once cleared), harvest
  season yield rollup computation.
- CLI commands (Artisan): `agro:weather:sync`, `agro:sensors:evaluate-thresholds`,
  `agro:withdrawal:sweep`, `agro:traceability:export` — each requires the
  same authorization context as its API equivalent, no CLI bypass of RBAC.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions 2 demo farms (a 3,000-acre row-crop operation and a
mixed crop-livestock operation) with mapped field boundaries, 2 seasons of
crop plans and input application history, sensor readings for 30 days, a
livestock herd with treatment/withdrawal history, and at least one fully
traceable harvest-to-shipment lot chain — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10; additionally: sensor ingestion pipeline sustains at least 500
readings/second per deployment without degrading interactive query latency, and
a traceability chain export for a single shipped lot completes in under 5
seconds regardless of how many input applications are in the chain.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. Compliance/certifier records and traceability exports are treated
as regulatory evidence — immutable once finalized, with corrections handled
via an amendment record rather than in-place edit, per
[data-protection-privacy.md](../../security/data-protection-privacy.md).

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated compliance-logic test suite covering PHI and withdrawal-period
conflict detection edge cases (boundary dates, overlapping applications),
since a false negative here is a regulatory exposure.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). Sensor
ingestion workers deploy and scale independently of the web tier so a
sensor-data spike (e.g. after a connectivity outage catches up) does not
degrade interactive dashboard performance.

## 30. Acceptance Criteria

- Logging an input application within a field's pre-harvest interval of a
  scheduled harvest date triggers a visible conflict flag before the entry
  saves.
- A treatment logged against livestock correctly computes and enforces a
  withdrawal period, blocking sale/processing eligibility until cleared.
- A traceability export for a shipped lot reconstructs the complete chain
  from field input applications through harvest to shipment with no gaps.
- Equipment job scheduling detects and surfaces overlapping-use conflicts
  before a job is confirmed.

## 31. Production Checklist

See
[production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).
ZodiAgro additionally requires sign-off that PHI/withdrawal compliance
logic has been validated against real product label data for the
operation's actual input catalog before go-live.

## 32. Future Roadmap

- Satellite/drone imagery integration for NDVI-based crop health mapping.
- Automated variable-rate application prescription generation.
- Carbon/sustainability program reporting module.

## 33. Known Risks

- Rural connectivity dependence: offline-first mobile logging mitigates
  most impact, but extended offline periods risk delayed compliance
  visibility until sync occurs.
- Regulatory data accuracy dependence: PHI/withdrawal logic quality depends
  on the accuracy of the underlying product label database, which must be
  kept current — a stale label dataset is a compliance risk outside the
  system's direct control.

## 34. Future Improvements

- Deeper equipment telematics integration for automatic input-application
  logging from machine data (reducing manual entry).
- Multi-year rotation optimization recommendations informed by soil health
  trend data.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), exhaustive endpoint
catalog (companion `API_REFERENCE.md`) covering livestock breeding/
reproduction endpoints and full sensor/IoT device management, and full
report catalog beyond the dashboards listed in §21. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).
