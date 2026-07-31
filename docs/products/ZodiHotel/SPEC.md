# ZodiHotel — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; deep artifacts (full ER diagram, exhaustive endpoint
> catalog, full report catalog) are queued — see
> [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiHotel is the property management system (PMS) that runs a hotel's entire
guest and revenue lifecycle — from the moment a room is priced and listed on
an OTA to the night audit that closes the day's books — on the same
ZodiCore foundation ([SPEC.md](../ZodiCore/SPEC.md)) every Zodize product
shares, so a hotel group running ZodiHotel already has enterprise identity,
multi-property tenancy, and billing solved before their first room type is
configured.

## 2. Purpose

Independent hotels and small groups are stuck choosing between legacy PMS
vendors (Oracle OPERA, Agilysys) priced and architected for large chains, or
lightweight cloud tools (Cloudbeds, Mews) that plateau once a property adds
banquet/event business or needs deep channel-manager control. ZodiHotel
exists to give a 20–500 room independent hotel or boutique group a PMS with
enterprise-grade channel distribution, folio accuracy, and audit rigor
without the OPERA-class implementation cost.

## 3. Target Market

Independent hotels, boutique hotel groups (2–50 properties), resorts, and
extended-stay/aparthotel operators who need a PMS but not the enterprise
overhead of a legacy chain-scale system. Secondary: hostels and small
conference/event hotels needing group booking and banquet functionality.

## 4. Industries

Hospitality — full-service hotels, boutique/independent hotels, resorts,
extended-stay properties, and small conference hotels.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Full-service PMS | Oracle OPERA Cloud | Modern UI, transparent per-room pricing, no multi-year enterprise contract required |
| Cloud-native PMS | Cloudbeds, Mews | Deeper channel-manager control and folio/billing sophistication out of the box |
| Boutique/independent focus | little hotelier, Clock PMS | Enterprise RBAC, audit trail, and multi-property tenancy from day one, not bolted on |
| Channel management | SiteMinder, RateGain | Native two-way channel sync as a first-class ZodiHotel module, not a third-party add-on fee |
| Group/event sales | Tripleseat (event side only) | Unified PMS + group booking + folio in one system instead of a separate event tool |

## 6. Personas

- **Front Desk Agent** — checks guests in/out, manages walk-ins, resolves
  folio disputes at the desk.
- **Reservations Manager** — manages inventory allocation, rates, group
  blocks, and OTA channel mix.
- **Revenue Manager** — sets rate strategy, monitors pickup/pace, adjusts
  rate plans and restrictions across channels.
- **Housekeeping Supervisor** — manages room status board, assigns
  housekeepers, tracks turnover time.
- **Night Auditor** — runs the end-of-day close: reconciles folios, posts
  room/tax charges, rolls the business date.
- **General Manager / Owner** — oversees occupancy, RevPAR, and P&L across
  one or many properties.
- **Group/Events Coordinator** — manages block bookings, banquet event
  orders, and group billing.
- **Guest** (indirect, via booking widget) — books, self-checks-in where
  enabled, receives confirmations and folios.

## 7. User Journeys

1. **Direct booking to check-in**: guest books via the property's booking
   engine widget → reservation created with rate plan and guarantee →
   confirmation email sent (see
   [notification-standards.md](../../standards/notification-standards.md))
   → on arrival day, front desk pulls the reservation, assigns a
   housekeeping-clean room, collects ID/payment, and checks the guest in →
   room status flips to Occupied and the folio opens.
2. **OTA booking to channel sync**: a Booking.com reservation arrives via
   the channel manager integration → ZodiHotel decrements availability
   across all other connected channels in real time → reservation appears
   in the same reservations grid as direct bookings, tagged with its source
   channel and commission rate for accurate net-revenue reporting.
3. **Group block to individual folios**: Events Coordinator creates a group
   block of 40 rooms for a conference with a group rate and a cutoff date →
   attendees book against the block by name or via a group booking link →
   unpicked-up rooms release automatically at the cutoff date per the
   configured release rule → at departure, master billing splits shared
   charges (banquet, AV) to the group folio while incidentals stay on
   individual guest folios.
4. **Split folio at checkout**: guest at checkout wants room charges billed
   to their company and incidentals paid personally → front desk splits the
   folio into two windows, routes charge codes accordingly, processes two
   separate payments, and closes both folios — full split history is
   retained on the reservation's activity timeline.
5. **Night audit close**: Night Auditor runs the night audit routine → system
   verifies all arrivals are checked in and all departures checked out (or
   flagged), posts room and tax charges for the night, reconciles payment
   totals against the day's transactions, generates the manager flash
   report, and rolls the business date forward — no reservation can post to
   a closed business date afterward.
5. **Housekeeping turnover**: a departure is checked out → the room's status
   automatically becomes Dirty on the housekeeping board → a housekeeper is
   assigned (manually or by round-robin rule) → housekeeper marks the room
   Clean via mobile view → status becomes Inspected once a supervisor signs
   off, and only then is the room available for the next check-in.

## 8. Business Goals

- Let a 20–500 room property run 100% of guest lifecycle operations
  (booking through night audit) in one system, eliminating reconciliation
  between a separate PMS, channel manager, and billing tool.
- Maximize channel revenue by keeping OTA and direct inventory/rates in
  sync in near real time, avoiding overbooking and rate parity violations.
- Reduce front-desk transaction time (check-in, folio split, checkout)
  through a single-screen workflow.
- Give ownership groups multi-property visibility (occupancy, RevPAR, ADR)
  from one dashboard without spreadsheet consolidation.

## 9. Functional Requirements

- **Room inventory & rate management**: room types, individual room
  records, rate plans (BAR, corporate, package), seasonal/day-of-week rate
  calendars, availability calendar with stop-sell/min-stay restrictions.
- **Reservations & booking engine**: direct booking widget, phone/walk-in
  reservation entry, hold/guarantee/deposit rules, cancellation policies,
  waitlist for sold-out dates.
- **Channel manager integration**: two-way sync of availability, rates, and
  restrictions with connected OTAs; inbound reservation ingestion with
  automatic mapping to internal rate plans and room types.
- **Front desk operations**: arrivals/departures board, check-in/check-out
  workflow, room assignment/swap, ID capture, walk-in booking, no-show
  processing.
- **Housekeeping status board**: real-time room status (Clean, Dirty,
  Inspected, Out of Order, Out of Service), housekeeper assignment, turnover
  time tracking, maintenance/out-of-order flagging with linked work order.
- **Folio & billing**: per-reservation folio with itemized charges, multiple
  folio windows/splits, routing rules (company/agent/individual), payment
  processing, deposits, city ledger (direct-bill) accounts.
- **Group bookings & events**: group blocks with pickup tracking, cutoff and
  auto-release rules, group/master folios, banquet event order (BEO)
  tracking for linked catering/AV charges.
- **Night audit**: guided end-of-day routine covering unresolved
  arrivals/departures, room/tax posting, payment reconciliation, manager
  flash report generation, and business-date rollover.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (e.g. comp-room/discount approval), rate/availability
  automation rules, saved filters on the reservations grid, custom fields
  on guest profiles, full audit history per folio, soft-delete/restore on
  reservations, mass actions (bulk rate updates), command palette, report
  builder, and white-labeled guest-facing booking widget.

## 10. Non-Functional Requirements

Baseline from [performance-standards.md](../../quality/performance-standards.md)
and [security-standards.md](../../security/security-standards.md) applies.
ZodiHotel-specific: the availability/rate-check API used by the booking
engine and channel manager must respond p95 < 300ms, since OTA channels
penalize slow or stale-availability responses; front-desk check-in/checkout
actions must complete p95 < 1s to keep queue lines moving during peak
arrival windows; the system must support overbooking-safe concurrent
inventory decrements across simultaneous direct and OTA bookings for the
same room type.

## 11. Architecture

ZodiHotel is a tenant application on ZodiCore
([architecture/overview.md](../../architecture/overview.md)), consuming
`zodize/core-identity`, `zodize/core-billing`, `zodize/core-notifications`,
`zodize/core-permissions`, and `zodize/core-plugins` as Composer packages.
Each hotel property is modeled as a ZodiCore `branch` under a tenant
`company` (per
[multi-tenancy.md](../../architecture/multi-tenancy.md)), so a hotel group
manages many properties under one tenant with property-scoped data
isolation and group-level roll-up reporting. Channel manager connections are
implemented as outbound integration adapters behind a
`ChannelConnectorContract`, decoupling ZodiHotel's inventory engine from any
single OTA's API shape; each connector normalizes inbound reservations and
outbound availability/rate pushes to ZodiHotel's canonical rate-plan model.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); Redis-backed
queue for channel manager sync jobs (near-real-time availability push);
scheduled jobs for night audit and group-block auto-release, per
[caching-queues-events.md](../../architecture/caching-queues-events.md).

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Inventory & Rates | Room Types, Individual Rooms, Rate Plans, Rate Calendar, Restrictions (stop-sell/min-stay) |
| Reservations | Booking Engine, Reservation Grid, Waitlist, Cancellation Policies, Deposits/Guarantees |
| Channel Management | OTA Connectors, Rate/Availability Push, Reservation Ingestion, Rate Parity Monitor |
| Front Desk | Arrivals/Departures Board, Check-in/Check-out, Room Assignment, Walk-ins, No-show Processing |
| Housekeeping | Room Status Board, Housekeeper Assignment, Turnover Tracking, Out-of-Order/Maintenance |
| Folio & Billing | Folio Windows, Charge Posting, Split Billing, Payments, City Ledger |
| Groups & Events | Group Blocks, Pickup/Cutoff, Master Folio, Banquet Event Orders |
| Night Audit | Audit Checklist, Charge Posting Run, Reconciliation, Flash Report, Business Date Rollover |
| Reporting | Occupancy/ADR/RevPAR Dashboards, Channel Production Report, Report Builder |

## 14. Core Data Model

| Entity | Key columns | Notes |
|---|---|---|
| `properties` | id, tenant_id, branch_id, name, timezone, currency | Maps to a ZodiCore `branch`; one property per hotel location |
| `room_types` | id, property_id, name, base_occupancy, max_occupancy | e.g. Standard King, Suite |
| `rooms` | id, room_type_id, room_number, floor, status | Physical/bookable room; status FK to housekeeping status |
| `rate_plans` | id, property_id, room_type_id, name, cancellation_policy_id, channel_scope | BAR, corporate, package, OTA-specific |
| `rate_calendar_entries` | id, rate_plan_id, date, rate_amount, min_stay, stop_sell | Daily rate/restriction grid |
| `reservations` | id, property_id, guest_id, room_type_id, rate_plan_id, check_in, check_out, status, source_channel | Core booking record |
| `reservation_rooms` | id, reservation_id, room_id, assigned_at | Supports room moves/multi-room reservations |
| `guests` | id, tenant_id, name, contact_info, id_document_ref, loyalty_id | Guest profile, reused across properties in a group |
| `folios` | id, reservation_id, window_number, billing_type, status | Split-billing windows on a reservation |
| `folio_charges` | id, folio_id, charge_code, amount, tax_amount, posted_at, posted_by | Itemized line items |
| `payments` | id, folio_id, method, amount, gateway_reference | Via ZodiCore payment gateway abstraction (§20 of ZodiCore SPEC) |
| `housekeeping_tasks` | id, room_id, assigned_to, status, started_at, completed_at, inspected_by | Turnover tracking |
| `group_blocks` | id, property_id, name, room_count, cutoff_date, release_rule | Group inventory hold |
| `banquet_event_orders` | id, group_block_id, event_date, room_setup, charges_summary | Catering/AV linkage to group folio |
| `channel_connections` | id, property_id, channel_name, credentials_ref, sync_status | OTA integration state |
| `night_audit_runs` | id, property_id, business_date, status, closed_by, closed_at | One record per night audit close |

## 15. Key API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/v1/properties/{property}/availability` | Room-type availability and rates for a date range |
| POST | `/api/v1/properties/{property}/reservations` | Create a reservation (direct or OTA-sourced) |
| GET | `/api/v1/reservations/{reservation}` | Retrieve reservation detail including folios |
| POST | `/api/v1/reservations/{reservation}/check-in` | Perform check-in, assign room |
| POST | `/api/v1/reservations/{reservation}/check-out` | Perform check-out, close/settle folio |
| PATCH | `/api/v1/reservations/{reservation}/room-assignment` | Assign or swap room |
| POST | `/api/v1/folios/{folio}/split` | Split a folio into an additional window |
| POST | `/api/v1/folios/{folio}/charges` | Post a charge to a folio |
| POST | `/api/v1/folios/{folio}/payments` | Record a payment against a folio |
| GET | `/api/v1/properties/{property}/housekeeping/board` | Current room status board |
| PATCH | `/api/v1/housekeeping/{task}/status` | Update a housekeeping task's status |
| POST | `/api/v1/properties/{property}/rate-plans` | Create a rate plan |
| PUT | `/api/v1/rate-plans/{plan}/calendar` | Bulk-update rate calendar entries |
| POST | `/api/v1/properties/{property}/channel-connections` | Connect a channel manager/OTA |
| POST | `/api/v1/channel-connections/{connection}/sync` | Force an availability/rate re-push |
| POST | `/api/v1/webhooks/channel/{channel}/reservations` | Inbound OTA reservation webhook receiver |
| POST | `/api/v1/properties/{property}/group-blocks` | Create a group block |
| POST | `/api/v1/group-blocks/{block}/pickup` | Book a room against a group block |
| GET | `/api/v1/properties/{property}/night-audit/checklist` | Current night audit checklist status |
| POST | `/api/v1/properties/{property}/night-audit/run` | Execute the night audit close |
| GET | `/api/v1/properties/{property}/reports/occupancy` | Occupancy/ADR/RevPAR report data |

## 16. Events

`reservation.created`, `reservation.modified`, `reservation.canceled`,
`reservation.no_show`, `reservation.checked_in`, `reservation.checked_out`,
`folio.split`, `folio.charge_posted`, `folio.payment_received`,
`housekeeping.status_changed`, `housekeeping.task_completed`,
`group_block.created`, `group_block.released`, `channel.reservation_ingested`,
`channel.sync_failed`, `night_audit.started`, `night_audit.completed`,
`rate_parity.violation_detected`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `reservation.created` (guest-facing) | — | ✔ (confirmation) | ✔ (optional) | — |
| `reservation.checked_in` | ✔ (front desk) | — | — | — |
| `channel.sync_failed` | ✔ (reservations mgr) | ✔ | — | ✔ |
| `rate_parity.violation_detected` | ✔ (revenue mgr) | ✔ | — | — |
| `housekeeping.task_completed` (supervisor) | ✔ | — | — | ✔ |
| `night_audit.completed` (GM) | ✔ | ✔ (flash report) | — | — |
| `group_block.release_pending` (T-3 days) | ✔ (events coordinator) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Inherits ZodiCore default roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles))
scoped per property/branch. ZodiHotel-specific permissions: `reservations.manage`,
`reservations.checkin`, `folios.manage`, `folios.void_charge` (elevated —
requires Manager+), `rates.manage`, `channels.manage`,
`housekeeping.manage`, `groups.manage`, `night_audit.run`,
`night_audit.reopen` (Manager+ only, always audit-logged). Front Desk Agent
role by default cannot void charges or reopen a closed business date.

## 19. Workflows & Approval Chains

- **Comp/discount approval**: any folio charge adjustment beyond a
  configurable threshold requires Manager approval before posting, per
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **Rate override approval**: a front-desk rate override below the rate
  plan's floor requires Reservations Manager or Revenue Manager approval.
- **Group block release**: unpicked rooms auto-release at the cutoff date
  unless the Events Coordinator extends it, which itself requires GM
  sign-off if the extension exceeds 7 days.
- **Night audit reopen**: reopening a closed business date to correct a
  posting error requires Manager-level approval and generates a mandatory
  audit-log entry with a reason code.

## 20. Audit Logs

Every folio charge, payment, void, rate override, room assignment change,
and business-date rollover is recorded to the ZodiCore audit log with actor,
timestamp, before/after values, and property scope — per
[audit-logging.md](../../security/audit-logging.md). Folio and reservation
records additionally carry a per-record activity timeline visible to Manager+
roles.

## 21. Reports & Analytics & Dashboards

Standard dashboards (per
[dashboard-standards.md](../../standards/dashboard-standards.md)): daily
occupancy/ADR/RevPAR, pickup pace vs. same-day-last-year, channel production
mix (direct vs. OTA vs. group), housekeeping turnaround time, folio
aging/city-ledger balances. Report builder allows custom report definitions
saved and scheduled per
[product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).
Multi-property groups get a group roll-up dashboard.

## 22. Integrations

- **OTA channel managers**: Booking.com, Expedia, Airbnb, and aggregator
  channel-manager platforms (SiteMinder, RateGain) via the
  `ChannelConnectorContract`.
- **Payment gateways**: via ZodiCore's payment gateway abstraction (§20 of
  [ZodiCore SPEC.md](../ZodiCore/SPEC.md#20-payment-gateways-wallet-accounting-taxes-invoices)),
  including terminal/card-present integration for front desk.
- **Booking engine widget**: embeddable direct-booking widget for property
  websites.
- **Point of sale**: optional integration with [ZodiPOS](../ZodiPOS/SPEC.md)
  for restaurant/bar charges posting to guest folios.
- **Key card systems**: door-lock encoder integration triggered on check-in.
- **Revenue management/rate shopping**: optional third-party rate
  intelligence feed for competitive rate benchmarking.

## 23. AI Features

- **Rate recommendation**: AI-assisted rate suggestions based on pickup
  pace, comp-set signals (where a rate-shopping integration is connected),
  and historical demand patterns, surfaced to the Revenue Manager as a
  recommendation, never auto-applied without confirmation.
- **No-show risk scoring**: flags reservations with elevated no-show
  likelihood (e.g. no deposit, same-day OTA booking with poor guarantee) for
  proactive front-desk follow-up.
- **AI-assisted guest messaging**: drafts responses to common pre-arrival
  guest questions grounded in the property's policies, sent only after
  staff review.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: nightly channel rate/availability re-sync, group block
  auto-release at cutoff, no-show auto-processing after a configurable grace
  window, night-audit reminder if not run by a configured cutoff time.
- CLI commands (Artisan): `hotel:channel-sync`, `hotel:night-audit:run`,
  `hotel:group-block:release`, `hotel:folio:reconcile` — each requires the
  same authorization context as its API equivalent, no CLI bypass of RBAC.

## 25. Seed Data, Demo Data

`DemoSeeder` provisions 2 demo properties (a 60-room boutique hotel and a
180-room full-service hotel) with a full room/rate-plan matrix, 90 days of
past and future reservations across direct and simulated OTA channels, an
active group block with partial pickup, populated housekeeping history, and
at least 30 completed night audit runs — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders).

## 26. Performance Requirements

See §10; additionally: the housekeeping status board updates within 2
seconds of a status change across all connected front-desk/housekeeping
clients (WebSocket/broadcast), and the night audit routine for a 200-room
property completes in under 5 minutes.

## 27. Security Requirements

Full baseline from [security-standards.md](../../security/security-standards.md)
applies. Guest ID documents and payment data are encrypted at rest per
[data-protection-privacy.md](../../security/data-protection-privacy.md);
PCI-DSS scope is minimized by tokenizing card data at the gateway, never
storing raw PAN in ZodiHotel's database.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated concurrency test suite for simultaneous inventory decrements
(direct booking vs. OTA booking racing for the last room) and a night-audit
correctness suite verifying no reservation can post to a rolled business
date.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). Channel
manager sync workers deploy independently of the web tier so an OTA API
outage or rate limit does not degrade front-desk responsiveness.

## 30. Acceptance Criteria

- A room booked through an OTA correctly decrements availability across all
  other connected channels within the channel sync SLA.
- A guest can be checked in, have folio charges split across two windows,
  and check out with both windows settled, with full audit history retained.
- A group block auto-releases unpicked rooms at its cutoff date without
  manual intervention.
- Night audit cannot be completed while unresolved arrivals/departures
  exist, and no charge can post to a business date already closed by night
  audit without an explicit, audit-logged reopen.

## 31. Production Checklist

See
[production-readiness-checklist.md](../../checklists/production-readiness-checklist.md).
ZodiHotel additionally requires sign-off that channel manager connections
have been tested against each connected OTA's sandbox/certification
environment before go-live.

## 32. Future Roadmap

- Dynamic/algorithmic rate pricing beyond recommendation (auto-apply within
  guardrails).
- Guest self-service kiosk and mobile check-in/checkout.
- Loyalty program module shared across a hotel group's properties.

## 33. Known Risks

- Channel manager API instability: OTA API changes or outages can desync
  inventory — mitigated by sync-failure alerting and a manual force-sync
  path, but this remains an external dependency risk.
- Overbooking risk during high-concurrency booking windows (e.g. a flash
  sale) — mitigated by row-level locking on inventory decrements, requires
  continued load testing as channel volume grows.

## 34. Future Improvements

- Real-time rate parity monitoring across all connected channels with
  automatic alerting on violations.
- Predictive housekeeping staffing recommendations based on forecasted
  occupancy.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: full ER
diagram and migration set (companion `DATA_MODEL.md`), exhaustive endpoint
catalog (companion `API_REFERENCE.md`) covering rate-plan CRUD detail,
cancellation-policy configuration endpoints, and full report catalog beyond
the dashboards listed in §21. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).
