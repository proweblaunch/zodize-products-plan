# ZodiFleet — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; deep artifacts (full ER diagram, exhaustive endpoint
> catalog, full report catalog) are queued — see
> [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiFleet is the operating system for a commercial vehicle fleet — vehicle
registry and driver assignment, live GPS/telematics tracking, preventive and
reactive maintenance scheduling, fuel/spend control, route optimization, and
regulatory compliance (inspections, hours-of-service) — built as a
standalone, self-hosted Laravel application from the shared Zodize base
codebase
([base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)),
so a fleet operator gets a working admin back office, RBAC, and multi-depot
data scoping without integrating a separate telematics vendor's back
office.

## 2. Purpose

Fleet operators today typically run three or four disconnected systems: a
telematics hardware vendor's portal, a separate maintenance/CMMS tool, a
fuel card portal, and a spreadsheet for compliance/HOS tracking. None of
them share a driver or vehicle record, so a compliance audit or a cost
analysis requires manual reconciliation across systems. ZodiFleet exists to
be the single system of record for every vehicle, driver, and trip, with
telematics hardware as a data source feeding one unified operational and
compliance picture.

## 3. Target Market

Mid-size to large commercial fleet operators (50–5,000+ vehicles) across
trucking/freight, field service, delivery/last-mile, and passenger transport
operations who need vehicle, driver, maintenance, and compliance management
in one system rather than a telematics-only point solution.

## 4. Industries

Logistics and transportation — freight/trucking carriers, field service
fleets (utilities, telecom, HVAC), last-mile delivery operations, and
passenger transport/paratransit fleets.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Fleet telematics platform | Samsara | Vendor-neutral hardware ingestion, not locked to one device manufacturer |
| Fleet management software | Fleetio, Verizon Connect Fleet | Compliance (HOS/inspections) and maintenance unified with core fleet ops, not a separate module purchase |
| Maintenance/CMMS for fleets | Fleetio Maintenance, Whip Around | Preventive scheduling driven by the same telematics odometer/engine-hour data, no manual sync |
| Route optimization | Route4Me, OptimoRoute | Native to the same platform holding driver HOS and vehicle constraints, producing compliant routes by construction |
| ELD/HOS compliance | KeepTruckin/Motive (compliance side) | Enterprise RBAC, audit trail, and multi-depot data scoping inherited from the base engine from day one |

## 6. Personas

- **Fleet Manager** — oversees vehicle utilization, cost, and compliance
  across the fleet.
- **Dispatcher** — assigns drivers to vehicles/routes, monitors live
  vehicle status.
- **Driver** — operates assigned vehicle, logs hours-of-service, reports
  incidents, uses mobile app for pre/post-trip inspections.
- **Maintenance Manager/Technician** — schedules and performs preventive
  and reactive maintenance work orders.
- **Compliance Officer** — monitors DOT-equivalent inspection records,
  driver HOS compliance, and incident reporting for regulatory readiness.
- **Finance/Cost Analyst** — tracks fuel spend, maintenance cost, and
  total cost of ownership per vehicle.

## 7. User Journeys

1. **Vehicle onboarding to first assignment**: Fleet Manager registers a new
   vehicle (VIN, make/model, telematics device pairing) → vehicle appears in
   the registry with live location once the telematics device reports in →
   Dispatcher assigns a driver to the vehicle for a route → driver receives
   the assignment on the mobile app and completes a pre-trip inspection
   before departure.
2. **Preventive maintenance trigger**: telematics feed reports a vehicle
   crossing a mileage or engine-hour threshold configured on a maintenance
   schedule → system automatically creates a maintenance work order and
   notifies the Maintenance Manager → vehicle is flagged for scheduling
   before the threshold becomes an overdue-service compliance issue.
3. **Reactive maintenance from a driver report**: driver reports a vehicle
   issue (e.g. warning light) via the mobile app during a post-trip
   inspection → a reactive work order is created and prioritized →
   Maintenance Manager assigns a technician → vehicle is flagged
   out-of-service until the work order closes, blocking new dispatch
   assignments to that vehicle.
4. **Route optimization with HOS constraints**: Dispatcher builds a
   multi-stop route for a driver → the route optimizer proposes a stop
   sequence that respects the driver's remaining hours-of-service and the
   vehicle's fuel range, flagging if the proposed route would push the
   driver past their HOS limit before the route is confirmed.
5. **Incident reporting to resolution**: a driver is involved in an
   accident → logs an incident report from the mobile app with photos,
   location, and description → Fleet Manager and Compliance Officer are
   notified immediately → the incident record links to the vehicle's and
   driver's history, and a corrective-action workflow tracks resolution
   (repair, insurance claim reference, retraining) through to close.
6. **DOT-equivalent inspection audit**: Compliance Officer runs a
   compliance readiness report ahead of a regulatory audit → the system
   surfaces every vehicle's current inspection status, every driver's HOS
   violation history, and any overdue maintenance — assembled from data
   already captured operationally rather than reconstructed for the audit.

## 8. Business Goals

- Reduce vehicle downtime by moving maintenance from reactive-only to
  telematics-triggered preventive scheduling.
- Reduce compliance violation exposure (HOS, inspection currency) through
  proactive, system-enforced flagging rather than after-the-fact discovery.
- Lower per-mile operating cost visibility by unifying fuel, maintenance,
  and utilization data per vehicle.
- Reduce dispatcher planning time via constraint-aware route optimization.

## 9. Functional Requirements

- **Vehicle registry**: vehicle records (VIN, make/model/year, ownership/
  lease status, assigned depot), document storage (registration,
  insurance), telematics device pairing.
- **Driver assignment & scheduling**: driver profiles (license class,
  endorsements, certifications), vehicle-driver assignment, shift/dispatch
  scheduling, driver availability tracking.
- **GPS/telematics integration**: live location tracking, trip history,
  odometer/engine-hour ingestion, harsh-event detection (hard braking,
  speeding), idle-time tracking.
- **Maintenance scheduling**: preventive maintenance schedules
  (mileage/engine-hour/time-based triggers), reactive work order creation,
  parts/labor tracking, out-of-service flagging, service history per
  vehicle.
- **Fuel tracking**: fuel purchase logging, fuel card transaction
  integration, fuel efficiency (MPG/L-per-100km) tracking per vehicle,
  anomaly detection (e.g. fuel volume exceeding tank capacity).
- **Route optimization**: multi-stop route planning respecting vehicle
  capacity/range and driver HOS constraints, route assignment to drivers,
  planned-vs-actual route variance tracking.
- **Compliance**: DOT-equivalent inspection record tracking (pre-trip/
  post-trip/annual), driver hours-of-service logging and violation
  detection, certification/license expiry tracking.
- **Incident/accident reporting**: incident logging with photo/document
  attachment, severity classification, corrective-action tracking, linkage
  to vehicle and driver history.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (e.g. out-of-service vehicle return-to-service
  sign-off), automation rules (maintenance-threshold-triggered work
  orders), saved fleet/vehicle filters, custom fields on vehicle/driver
  records, full audit history, soft-delete/restore, mass actions (bulk
  reassignment), command palette, report builder, scheduled compliance
  reports.

## 10. Non-Functional Requirements

Baseline from [performance-standards.md](../../quality/performance-standards.md)
and [security-standards.md](../../security/security-standards.md) applies.
ZodiFleet-specific: telematics ingestion must sustain high-frequency
location/event streams (position updates as frequent as every 10–30
seconds per vehicle) without degrading the live fleet map's render latency
(p95 map refresh < 2s for a 500-vehicle fleet); HOS violation detection
must evaluate in near real time (< 60s from the triggering telematics/log
event) since HOS compliance is a safety-critical, legally enforced
constraint.

## 11. Architecture

ZodiFleet is a standalone, self-hosted Laravel application, sold as source
code to one fleet operator and deployed entirely within that buyer's own
hosting account — there is no shared platform service and no other Zodize
product it depends on at runtime
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
It is built by cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific tables (`loans`, `dps`, `fdr`, `other_banks`,
`beneficiaries`, `airtime_operators`) are stripped, and ZodiFleet's own
domain modules — vehicle registry, driver management, telematics,
maintenance, fuel, routing, compliance, incidents — are built on top of the
inherited engine's wallet/ledger, payment gateways, RBAC/auth, KYC, i18n,
and admin configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)).

A multi-depot operator re-purposes the base's branch-scoped staff guard
rather than the base's banking `BranchStaff` code: per
[product-genericization-checklist.md §4](../../architecture/product-genericization-checklist.md#step-4--confirm-guard-configuration-matches-the-products-needs),
ZodiFleet re-adds a `branch_staff`-equivalent guard scoped to a depot, so
depot-level dispatchers and maintenance staff can be restricted to their
depot's vehicles and drivers while a Fleet Manager/Compliance Officer role
sees across all depots in the one deployment. Depots/terminals are modeled
as `branches` per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
— this is data scoping within one buyer's one deployment, not
multi-tenancy; there is no `tenant_id` anywhere in ZodiFleet's schema.
Telematics hardware integration is implemented behind a
`TelematicsConnectorContract` so ZodiFleet's core data model (vehicles,
trips, events) is vendor-neutral; each connector normalizes a hardware
vendor's data feed into ZodiFleet's canonical trip/event schema, with
high-frequency location/event ingestion routed through a dedicated
write-optimized pipeline decoupled from the transactional data model, per
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL with PostGIS extension for geospatial route/location data, plus
Redis, per [database-standards.md](../../development/database-standards.md);
a time-series-optimized table strategy for telematics position/event data;
mobile driver app built on the shared Vue component library for
inspections, HOS logging, and incident reporting.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Vehicle Registry | Vehicle Records, Document Storage, Telematics Device Pairing, Ownership/Lease Tracking |
| Driver Management | Driver Profiles, Licenses/Certifications, Assignment, Availability |
| Telematics | Live Location, Trip History, Odometer/Engine-hour Ingestion, Harsh-event Detection, Idle Tracking |
| Maintenance | Preventive Schedules, Work Orders (Reactive), Parts/Labor Tracking, Out-of-Service Flagging |
| Fuel | Fuel Purchase Logging, Fuel Card Integration, Efficiency Tracking, Anomaly Detection |
| Routing | Route Planning, Constraint-aware Optimization, Route Assignment, Planned-vs-Actual Variance |
| Compliance | Inspection Records, Hours-of-Service Logging, Violation Detection, Certification Expiry |
| Incidents | Incident Reporting, Severity Classification, Corrective-action Tracking |
| Reporting | Utilization/Cost Dashboards, Compliance Readiness, Report Builder |

## 14. Core Data Model

All tables belong to the one buyer's one deployment — there is no
`tenant_id` column anywhere in this model
([single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)).
A multi-depot operator scopes vehicles and drivers via `depot_id` (the base
engine's re-purposed `branches` table).

| Entity | Key columns | Notes |
|---|---|---|
| `vehicles` | id, depot_id, vin, make, model, year, status | `depot_id` FK to the inherited `branches` table |
| `telematics_devices` | id, vehicle_id, external_device_id, vendor, status | Pairing record for a `TelematicsConnectorContract` |
| `drivers` | id, user_id, license_number, license_class, status | FK to the inherited `users` table for the driver's login identity |
| `vehicle_assignments` | id, vehicle_id, driver_id, assigned_at, released_at | Current/historical driver-vehicle pairing |
| `trips` | id, vehicle_id, driver_id, started_at, ended_at, start_location, end_location, distance | Aggregated trip derived from telematics events |
| `telematics_events` | id, vehicle_id, event_type, occurred_at, location, value | Harsh braking, speeding, idle, position ping (high volume) |
| `maintenance_schedules` | id, vehicle_id, trigger_type, trigger_value, last_service_at | Mileage/hour/time-based preventive trigger |
| `work_orders` | id, vehicle_id, type (preventive/reactive), status, opened_at, closed_at, technician_id | Maintenance job record |
| `fuel_transactions` | id, vehicle_id, driver_id, gallons_or_liters, cost, odometer_at_fill, source | Fuel card or manual entry |
| `routes` | id, depot_id, driver_id, vehicle_id, planned_stops, status | Optimizer output and assignment |
| `inspections` | id, vehicle_id, driver_id, type (pre-trip/post-trip/annual), passed, performed_at | DOT-equivalent inspection record |
| `hos_logs` | id, driver_id, duty_status, started_at, ended_at | Hours-of-service duty-status log |
| `hos_violations` | id, driver_id, hos_log_id, rule_violated, detected_at | System-detected HOS violation |
| `incidents` | id, vehicle_id, driver_id, severity, occurred_at, description, status | Accident/incident record |

## 15. Key API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/vehicles` | List fleet vehicles with current status |
| POST | `/api/v1/vehicles` | Register a new vehicle |
| POST | `/api/v1/vehicles/{vehicle}/telematics-device` | Pair a telematics device |
| GET | `/api/v1/vehicles/{vehicle}/location` | Current live location |
| POST | `/api/v1/webhooks/telematics/{vendor}/events` | Inbound telematics event receiver |
| POST | `/api/v1/vehicle-assignments` | Assign a driver to a vehicle |
| GET | `/api/v1/drivers/{driver}/trips` | Trip history for a driver |
| POST | `/api/v1/vehicles/{vehicle}/work-orders` | Create a maintenance work order |
| PATCH | `/api/v1/work-orders/{work_order}/status` | Update work order status (incl. return-to-service) |
| POST | `/api/v1/vehicles/{vehicle}/fuel-transactions` | Log a fuel transaction |
| POST | `/api/v1/webhooks/fuel-cards/{provider}/transactions` | Inbound fuel card transaction receiver |
| POST | `/api/v1/depots/{depot}/routes` | Create/optimize a multi-stop route |
| POST | `/api/v1/routes/{route}/assign` | Assign a route to a driver/vehicle |
| POST | `/api/v1/vehicles/{vehicle}/inspections` | Log a pre-trip/post-trip/annual inspection |
| POST | `/api/v1/drivers/{driver}/hos-logs` | Log a duty-status change |
| GET | `/api/v1/drivers/{driver}/hos-status` | Current available hours and violation history |
| POST | `/api/v1/incidents` | Report an incident/accident |
| PATCH | `/api/v1/incidents/{incident}/corrective-action` | Update corrective-action status |
| GET | `/api/v1/fleet/reports/utilization` | Vehicle utilization/cost report data |
| GET | `/api/v1/fleet/reports/compliance-readiness` | Compliance readiness summary for audit prep |

## 16. Events

`vehicle.registered`, `vehicle.status_changed`, `assignment.created`,
`assignment.released`, `telematics.harsh_event_detected`,
`maintenance.threshold_reached`, `work_order.created`,
`work_order.closed`, `vehicle.out_of_service`,
`vehicle.returned_to_service`, `fuel_transaction.anomaly_detected`,
`route.optimized`, `route.hos_conflict_detected`, `inspection.failed`,
`hos.violation_detected`, `incident.reported`,
`incident.corrective_action_completed`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `maintenance.threshold_reached` | ✔ (maintenance mgr) | ✔ | — | ✔ |
| `vehicle.out_of_service` | ✔ (dispatcher, fleet mgr) | ✔ | — | ✔ |
| `hos.violation_detected` | ✔ (driver, compliance officer) | ✔ | ✔ | ✔ |
| `incident.reported` | ✔ (fleet mgr, compliance officer) | ✔ | ✔ | ✔ |
| `inspection.failed` | ✔ (maintenance mgr) | ✔ | — | ✔ |
| `fuel_transaction.anomaly_detected` | ✔ (fleet mgr) | ✔ | — | — |
| `route.hos_conflict_detected` | ✔ (dispatcher) | — | — | ✔ |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits the base codebase's default admin roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles)),
scoped per depot via the `branch_staff`-equivalent guard. ZodiFleet-specific permissions: `vehicles.manage`,
`drivers.manage`, `assignments.manage`, `work_orders.manage`,
`work_orders.return_to_service` (elevated — requires Maintenance Manager+),
`routes.plan`, `hos.manage`, `hos.override_violation` (elevated — requires
Compliance Officer, always audit-logged), `incidents.manage`. Driver role
by default can log HOS/inspections/incidents for themselves only, not view
fleet-wide data.

## 19. Workflows & Approval Chains

- **Return-to-service approval**: a vehicle flagged out-of-service requires
  Maintenance Manager sign-off before it can be returned to the dispatch
  pool, per
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **HOS violation override**: an HOS violation flag can only be overridden
  (e.g. for a documented adverse-condition exception) by a Compliance
  Officer with a mandatory reason code, always audit-logged.
- **Incident corrective-action sign-off**: an incident cannot be closed
  until its corrective-action items are marked complete and the Fleet
  Manager signs off.

## 20. Audit Logs

Every vehicle status change, work order status change, HOS log entry and
violation override, and incident status change is recorded to the
deployment's audit log with actor, timestamp, before/after values, and
depot scope —
per [audit-logging.md](../../security/audit-logging.md). HOS logs and
inspection records additionally carry an immutable, append-only history
independent of general audit history, since they are the artifact produced
for a regulatory audit.

## 21. Reports & Analytics & Dashboards

Standard dashboards (per
[dashboard-standards.md](../../standards/dashboard-standards.md)): fleet
utilization and downtime, cost-per-mile (fuel + maintenance), maintenance
backlog/overdue schedule, compliance readiness (inspection currency, HOS
violation trend), incident trend by severity. Report builder supports
custom cost and compliance reports, saved and scheduled per
[product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Telematics hardware**: GPS/ELD device vendors (e.g. Samsara-class,
  Geotab-class hardware) via the `TelematicsConnectorContract`.
- **Fuel cards**: fleet fuel card providers for automated transaction
  ingestion and reconciliation against odometer data.
- **Mapping/routing engines**: third-party mapping and route-optimization
  engines for multi-stop route generation.
- **ELD/HOS compliance data**: electronic logging device data feeds where
  HOS is captured on dedicated ELD hardware rather than the mobile app.
- **Insurance/claims systems**: optional export of incident records to
  fleet insurance claims workflows.

## 23. AI Features

- **Predictive maintenance**: AI-assisted prediction of likely component
  failure based on telematics engine-health signals and service history,
  surfaced as an early-service recommendation, never auto-scheduling work
  without Maintenance Manager confirmation.
- **Route optimization scoring**: AI-assisted stop sequencing considering
  traffic patterns, driver HOS remaining, and historical delivery-window
  performance.
- **Driver risk scoring**: aggregates harsh-event telematics data and
  incident history into a driver risk score surfaced to Fleet Managers for
  coaching, never used for automated disciplinary action.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: nightly maintenance-threshold evaluation across the
  fleet, HOS violation sweep, fuel transaction anomaly detection, license/
  certification expiry reminders, telematics device health check
  (detect devices that have stopped reporting).
- CLI commands (Artisan): `fleet:maintenance:evaluate-thresholds`,
  `fleet:hos:sweep`, `fleet:telematics:health-check`,
  `fleet:compliance:export` — each requires the same authorization context
  as its API equivalent, no CLI bypass of RBAC.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions 2 demo depots (a 40-vehicle regional delivery fleet
and a 120-vehicle long-haul freight fleet) with paired simulated telematics
devices, 90 days of trip/event history, a mix of open and closed
maintenance work orders, populated fuel transaction history, HOS logs
including at least one detected violation, and at least one closed
incident with corrective-action history — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10; additionally: the live fleet map supports at least 1,000
concurrently tracked vehicles per deployment with sub-2-second position
refresh, and route optimization for a 50-stop route completes in under 10
seconds.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. Driver location and HOS data are treated as sensitive personal
data with access scoped to roles with an operational need, per
[data-protection-privacy.md](../../security/data-protection-privacy.md);
HOS and inspection records are immutable once logged, corrections handled
via an amendment record with reason code, never in-place edit.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated HOS rule-engine test suite covering duty-status transition edge
cases and violation-detection accuracy, since a false negative here is a
regulatory and safety exposure.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Telematics ingestion workers deploy and scale independently of the web
tier so a device data spike or vendor API burst does not degrade
dispatcher-facing responsiveness.

## 30. Acceptance Criteria

- A vehicle crossing a configured maintenance mileage/hour threshold
  automatically generates a work order and notifies the Maintenance
  Manager without manual monitoring.
- A driver duty-status log that would exceed an HOS limit is detected and
  flagged within the required near-real-time SLA.
- A vehicle flagged out-of-service cannot be assigned to a new route until
  its work order closes and a Maintenance Manager returns it to service.
- An incident report is fully traceable from initial submission through
  corrective-action completion, with full audit history retained.

## 31. Production Checklist

See
[production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).
ZodiFleet additionally requires sign-off that telematics device
integrations have been validated against live hardware (not only
simulated feeds) for each connected vendor before go-live.

## 32. Future Roadmap

- Dynamic route re-optimization mid-route in response to live traffic or
  new stop insertions.
- Driver mobile app offline mode for inspections/HOS logging in
  connectivity dead zones.
- Fleet electrification planning module (EV range/charging scheduling).

## 33. Known Risks

- Telematics hardware/vendor dependency: device outages or API changes can
  create data gaps — mitigated by device health-check alerting, but this
  remains an external dependency risk.
- HOS rule complexity varies by jurisdiction: the rule engine must be kept
  current with regulatory changes across operating regions, requiring
  ongoing compliance-rule maintenance, not a one-time build.

## 34. Future Improvements

- Real-time driver coaching prompts based on live harsh-event detection.
- Total cost of ownership forecasting per vehicle incorporating
  depreciation alongside fuel and maintenance cost.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), exhaustive endpoint
catalog (companion `API_REFERENCE.md`) covering full HOS rule-set
configuration endpoints and route-optimizer parameter tuning, and full
report catalog beyond the dashboards listed in §21. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).
