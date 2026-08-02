# ZodiTrack — Product Specification

> Status: **Live — Extend Only**. See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md)'s status definition and
> [`BUILD_STATE.md`](../../../BUILD_STATE.md)'s protocol for this status.
> ZodiTrack is not built from this spec — it already exists as a complete,
> working, currently-resold product. This spec is being reconciled against
> the real live codebase; see the correction notice immediately below before
> reading anything further in this document.

## 0. Verified correction — this spec's domain does not match the live product

A direct filesystem audit of the actual live codebase (confirmed present at
`/home/script/public_html/zoditrack/` on the build VPS) found that ZodiTrack
is **not** an internal enterprise asset/inventory-tracking (ITAM) tool as
the rest of this document (written before that audit) describes below.
**The real, live, currently-resold product is a customer-facing freight/
shipment tracking and logistics brokerage website**, confirmed via direct
inspection of its file structure and code:

- Public marketing/service pages: `air-freight.php`, `ocean-freight.php`,
  `cargo-transportation.php`, `packaging-and-storage.php`, plus standard
  `about-us.php`, `contact.php`, `faq.php`, and legal pages
  (`privacy-policy.php`, `terms.php`, `refund-policy.php`,
  `shipping-policy.php`, `cookie-policy.php`).
- A public **tracking number lookup** (`track.php`): a customer enters a
  tracking number and sees shipment status (`Pending`, `Picked Up`,
  `Processing`, `In Transit`, `Arrived at Facility`, `Out for Delivery`,
  `Delivered`, `Delayed`, `Failed Delivery`, `Returned`) with a
  status-history timeline, confirmed via direct code read of `track.php`'s
  status-color/icon mapping and its query against a `tracking` table.
- A `customer/` portal directory and a `receipt.php` (37KB, substantial
  logic) generating shipment receipts.
- A full `admin/` back office (confirmed via direct directory listing, 33
  files) covering: `dashboard.php`, `shipments.php`, `add-tracking.php` /
  `edit-tracking.php` (both 27–31KB — substantial shipment-entry logic),
  `shipment-modes.php`, `branches.php`, `staff.php`, `customers.php`,
  `vendors/`, `invoices.php`, `reports.php`, `notifications.php`,
  `activity-log.php`, and `settings.php`.
- The stack is confirmed **native procedural PHP** — direct database calls
  via `mysqli` (see `db.php`, `track.php`'s `mysqli_prepare`/
  `mysqli_stmt_bind_param` calls), page-based routing (`.php` files, not a
  framework router), and shared `header.php`/`footer.php` includes — not
  Laravel, not any framework. This confirms the "native procedural PHP
  stack" characterization already given to it elsewhere in this handbook.

**This means ZodiTrack is a freight-forwarding/logistics-brokerage
shipment-tracking business (closer to a courier/freight-forwarder's
customer-facing tracking site plus back-office shipment management), not an
ITAM/enterprise-asset-tracking tool.** The Vision, Purpose, Target Market,
Competitor Analysis, Personas, and User Journeys sections below (§1–§7)
describe the WRONG domain and MUST NOT be used to guide any extension work
until they are rewritten to match the real product. Per
[`BUILD_STATE.md`](../../../BUILD_STATE.md)'s protocol — trust the
filesystem over the ledger/spec when they disagree — this mismatch is
recorded here rather than silently guessed past, and rather than this
session fabricating a full rewrite of personas/journeys for the real
freight-tracking business without a deeper audit of every admin screen's
actual behavior (a partial rewrite risks being just as wrong as the
original). A follow-up session MUST:

1. Read every file under `admin/` and the customer-facing pages in full
   (this audit confirmed their existence and rough purpose from filenames,
   sizes, and one file's content — `track.php` — not their complete
   behavior).
2. Rewrite §1–§7 (Vision through Business Goals) to describe the real
   freight/shipment-tracking business.
3. Populate the "Gap list" section below with a genuine
   feature-by-feature comparison once the rest of this document is
   rewritten to match — a gap list against the WRONG domain description is
   not meaningful.

Everything from §8 onward in this document (Functional Requirements
through the closing Roadmap) was written against the incorrect ITAM framing
and needs the same reconciliation pass.

## Gap list (populate after §1–§7 are corrected)

**Not yet populated**, per the reasoning above. Confirmed-present
capabilities that any rewritten spec/gap-list MUST account for as already
built (not gaps): tracking-number lookup with status history, shipment
CRUD (`add-tracking.php`/`edit-tracking.php`), shipment modes, branches,
staff, customers, vendors, invoicing, reporting, in-admin notifications,
an activity log, and a settings screen. Nothing in this list should be
proposed as new work without first confirming it isn't already implemented
under a different name.

## 1. Vision

> **The section below (through §7) describes the wrong domain — see the
> correction notice in §0 above. Do not use it to guide extension work.**

ZodiTrack is the asset and inventory tracking system for organizations that
own physical things across many locations — equipment, tools, IT hardware,
fleet-adjacent gear, shared facilities assets — and need to know, at any
moment, where each one is, who has it, what condition it's in, and when it
needs maintenance, without relying on a spreadsheet someone forgot to update.

## 2. Purpose

Organizations with distributed physical assets lose money three ways: assets
that go missing because nobody logged a transfer, assets that fail because
scheduled maintenance was tracked in someone's head, and assets that are
over-depreciated or under-utilized because nobody has a single source of
truth. ZodiTrack exists to make every asset's location, custody, condition,
and lifecycle status a queryable fact, not institutional memory.

## 3. Target Market

Mid-market and enterprise organizations managing 500–500,000 trackable
physical assets across multiple sites — facilities/property teams, IT asset
management (ITAM) teams, field-service organizations, healthcare equipment
teams, and educational institutions managing shared equipment pools.

## 4. Industries

Logistics and warehousing (primary), facilities management, IT asset
management, healthcare equipment/biomed, education (shared lab/AV
equipment), and light manufacturing tooling.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Asset tracking/tagging | Asset Panda, EZOfficeInventory | Self-hosted deployment with the base engine's own RBAC/audit and the shared Zodize design system built in, instead of a standalone SaaS silo |
| IT asset management | Snipe-IT, ServiceNow ITAM | Cross-industry asset model (not IT-only), with the same rigor extended to facilities/field equipment |
| Maintenance scheduling | UpKeep, Fiix (CMMS) | Maintenance scheduling is one module of the same asset record, not a separate CMMS system to reconcile |
| Check-in/check-out for shared equipment | Booqable | Same permission model and audit trail as every other Zodize product, no separate login |
| Barcode/QR asset tagging | Wasp Barcode | First-party mobile scan flow tied directly into the location/transfer ledger |

## 6. Personas

- **Asset Manager** — owns the asset registry, categories, and lifecycle
  policy (depreciation schedules, retirement rules).
- **Facilities/Field Technician** — performs check-in/check-out, condition
  logging, and scans assets during physical audits.
- **Maintenance Coordinator** — schedules and tracks preventive and
  corrective maintenance work orders.
- **Location/Site Manager** — oversees assets assigned to their site,
  approves transfers in/out.
- **Finance/Accounting** — consumes depreciation schedules for financial
  reporting, integrates with [ZodiBusiness](../ZodiBusiness/SPEC.md) or an
  external ERP.
- **Auditor** — runs periodic physical-count reconciliation against the
  system of record.

## 7. User Journeys

1. **Asset onboarding**: Asset Manager registers a new asset (purchase date,
   cost, category, depreciation method) → system generates a QR/barcode tag
   → tag is printed and physically affixed → asset is assigned to an initial
   location and custodian.
2. **Transfer between locations**: Site Manager at the origin location
   initiates a transfer request → destination Site Manager approves receipt
   → Field Technician scans the asset's tag on arrival to confirm physical
   receipt → the location/transfer ledger updates and the prior location's
   open-asset count decrements.
3. **Check-out and check-in of shared equipment**: an employee checks out a
   shared asset (e.g. a projector, a loaner laptop) via a scan → due-back
   date is set → system sends a reminder before the due date → employee
   checks it back in with a condition note → any condition degradation is
   logged against the asset's history.
4. **Preventive maintenance cycle**: an asset's maintenance schedule
   (time-based or usage-based) triggers a work order → Maintenance
   Coordinator assigns it to a technician → technician completes the work,
   logs parts/labor, and updates the asset's condition and next-due date →
   completion is recorded against the asset's full maintenance history.
5. **Physical audit reconciliation**: Auditor runs a cycle count for a
   location → scans every asset physically present → the system flags
   assets expected at that location but not scanned (missing) and assets
   scanned but not expected (misplaced) → discrepancies route to the Asset
   Manager for resolution and audit-logged adjustment.

## 8. Business Goals

- Eliminate "lost asset" write-offs caused by undocumented transfers.
- Reduce unplanned equipment downtime by converting maintenance from
  reactive to scheduled.
- Give Finance an accurate, audit-ready depreciation schedule without a
  manual fixed-asset spreadsheet.

## 9. Functional Requirements

- Asset registry: category, custom fields per category, purchase/warranty
  data, current location, current custodian, current condition, status
  (active/in-maintenance/retired/lost).
- QR/barcode tagging: tag generation, printable label templates, mobile scan
  capture (camera-based and dedicated scanner hardware) for lookup and
  transactions.
- Location/transfer tracking: hierarchical locations (site → building →
  room), transfer request/approval, transfer history per asset.
- Maintenance scheduling: time-based and usage-based (e.g. runtime hours)
  triggers, work order lifecycle, parts/labor logging, maintenance history.
- Depreciation: straight-line and declining-balance methods, configurable
  useful-life per category, automated schedule generation and monthly
  posting readiness for accounting integration.
- Check-in/check-out: due-back dates, overdue alerts, condition capture at
  each transaction, reservation of shared assets for future dates.
- Condition/audit logging: condition rating scale, photo attachment at
  transactions, full timeline per asset.
- Low-stock/reorder alerts: for consumable-adjacent trackable inventory
  (e.g. spare parts pools) distinct from durable asset tracking.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  saved filters on the asset list (by category/location/status), bulk
  actions (bulk transfer, bulk retire), custom fields per asset category,
  CSV import/export/bulk-tag-generation wizard, full audit and version
  history per asset, soft delete + restore for retired assets.

## 10. Non-Functional Requirements

Inherits the baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md).
ZodiTrack-specific additions:

- Mobile scan-to-lookup response time: p95 < 500ms including barcode decode,
  since technicians scan in the field and expect near-instant feedback.
- Asset registry search/filter must remain performant at 500,000+ assets per
  deployment via indexed category/location/status filters, not client-side
  filtering.
- Offline-tolerant mobile scanning: scans captured with no connectivity
  queue locally and sync once connectivity resumes (see
  [ZodiPOS's offline model](../ZodiPOS/SPEC.md#11-architecture) for the
  shared offline-sync pattern this reuses).

## 11. Architecture

ZodiTrack is built by cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific `loans`/`dps`/`fdr`/`branches`/`branch_staff`/
`other_banks`/`beneficiaries`/`airtime` tables are stripped, since none of
them serve an asset-tracking product, and the `branch_staff` guard is
dropped by default. ZodiTrack inherits the base engine's RBAC/auth, i18n,
and admin configuration surface unmodified — see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
and layers its own Asset Registry, Locations, Check-In/Check-Out,
Maintenance, Financial, Audit, and Consumables modules (§13) on top, per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#layering-a-products-domain-modules-onto-the-sanitized-base).
There is no shared tenant boundary and no ZodiCore platform dependency: each
ZodiTrack deployment is one organization's standalone, self-hosted instance,
per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).

The mobile scanning experience is a lightweight PWA client, built as part of
this same product's codebase, that queues scan transactions in local storage
and syncs via an idempotent transaction API (each queued scan carries a
client-generated UUID so a retried sync never double-applies), following the
same offline-first design principle used in [ZodiPOS](../ZodiPOS/SPEC.md)'s
own, independently deployed register client. Location hierarchy and asset
transfer state are modeled as an append-only ledger — mirroring the
inherited wallet engine's own append-only `Transaction` pattern (see
[wallet-system.md](../../standards/wallet-system.md)) — so an asset's full
custody history is always reconstructable within this one deployment's
database. Organizations tracking assets across multiple sites use the
`locations` hierarchy in §14 for that scoping; a multi-legal-entity
organization additionally uses the `company_id`-scoped multi-company model
per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping) —
this is scoping within one deployment, not tenancy.

## 12. Technology

Laravel (PHP) per the base codebase's stack (Laravel 11, PHP ^8.3, Vite 5) —
see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
following
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md);
MySQL/MariaDB + Redis (where the buyer's hosting supports it, with a file/DB
cache fallback) per
[database-standards.md](../../development/database-standards.md); a PWA
mobile client with a local IndexedDB queue for offline scan capture;
QR/Code128 barcode generation via a standard open-source library, decoded
client-side via the device camera or a paired Bluetooth/USB scanner.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Asset Registry | Categories, Custom Fields, Asset Records, Tagging/Labels |
| Locations | Location Hierarchy, Transfers, Transfer Approvals |
| Check-In/Check-Out | Reservations, Due-Back Tracking, Overdue Alerts |
| Maintenance | Schedules, Work Orders, Parts/Labor Logging |
| Financial | Depreciation Schedules, Disposal/Write-Off |
| Audit | Condition Logging, Physical Count/Cycle Audits, Discrepancy Resolution |
| Consumables | Spare Parts Pools, Reorder Alerts |

## 14. Core Data Model

| Entity | Key columns |
|---|---|
| `companies` | id, name, default_currency, is_active |
| `assets` | id, company_id, tag_code, category_id, status, current_location_id, current_custodian_id |
| `asset_categories` | id, company_id, name, depreciation_method, default_useful_life_months |
| `locations` | id, company_id, parent_location_id, name, type (site/building/room) |
| `asset_transfers` | id, asset_id, from_location_id, to_location_id, requested_by, approved_by, status |
| `checkouts` | id, asset_id, checked_out_to, due_back_at, checked_in_at, condition_out, condition_in |
| `maintenance_schedules` | id, asset_id, trigger_type (time/usage), interval_value, next_due_at |
| `work_orders` | id, asset_id, schedule_id, status, assigned_to, completed_at |
| `work_order_parts` | id, work_order_id, part_id, quantity, cost |
| `condition_logs` | id, asset_id, recorded_at, rating, photo_url, note |
| `depreciation_schedules` | id, asset_id, method, monthly_amount, book_value, as_of_date |
| `spare_parts` | id, company_id, sku, name, quantity_on_hand, reorder_point |
| `cycle_counts` | id, company_id, location_id, performed_by, started_at, completed_at |
| `cycle_count_discrepancies` | id, cycle_count_id, asset_id, expected, actual, resolution |

`companies` is optional multi-legal-entity scoping within this one
organization's single deployment (e.g. a holding company tracking assets
across separately-booked subsidiaries), per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping);
an organization with one legal entity has exactly one seeded `companies`
row. Multi-site scoping within a company uses the `locations` hierarchy
(site → building → room) above, not a separate scoping column.

## 15. Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/assets` | List/search assets with category/location/status filters |
| POST | `/api/v1/assets` | Register a new asset and generate its tag |
| GET | `/api/v1/assets/{id}` | Asset detail including full history timeline |
| GET | `/api/v1/assets/lookup/{tag_code}` | Scan-to-lookup by tag code |
| POST | `/api/v1/assets/{id}/transfer` | Initiate a location transfer |
| POST | `/api/v1/transfers/{id}/approve` | Approve/reject a pending transfer |
| POST | `/api/v1/assets/{id}/checkout` | Check out a shared asset |
| POST | `/api/v1/assets/{id}/checkin` | Check in with condition data |
| POST | `/api/v1/scans/batch` | Sync a queued batch of offline scan transactions |
| GET | `/api/v1/maintenance/schedules` | List upcoming maintenance by due date |
| POST | `/api/v1/work-orders` | Create a maintenance work order |
| POST | `/api/v1/work-orders/{id}/complete` | Complete a work order, update asset condition |
| GET | `/api/v1/assets/{id}/depreciation` | Depreciation schedule and current book value |
| POST | `/api/v1/assets/{id}/dispose` | Retire/dispose an asset |
| POST | `/api/v1/cycle-counts` | Start a physical audit for a location |
| POST | `/api/v1/cycle-counts/{id}/scan` | Record a scan during an active cycle count |
| GET | `/api/v1/cycle-counts/{id}/discrepancies` | List unresolved discrepancies |
| GET | `/api/v1/spare-parts/low-stock` | List spare parts below reorder point |

## 16. Events

`asset.registered`, `asset.transferred`, `asset.checked_out`,
`asset.checked_in`, `asset.checkout_overdue`, `asset.condition_degraded`,
`work_order.created`, `work_order.completed`, `maintenance.due_soon`,
`asset.disposed`, `cycle_count.discrepancy_found`, `spare_part.low_stock`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `asset.checkout_overdue` | ✔ (custodian, Site Manager) | ✔ | — | ✔ |
| `maintenance.due_soon` | ✔ (Maintenance Coordinator) | ✔ | — | — |
| `asset.transferred` (pending approval) | ✔ (destination Site Manager) | ✔ | — | — |
| `work_order.completed` | ✔ (Asset Manager) | — | — | — |
| `cycle_count.discrepancy_found` | ✔ (Asset Manager) | ✔ | — | — |
| `spare_part.low_stock` | ✔ (Asset Manager) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the base engine's inherited `Role`/`Permission` RBAC (not Spatie),
per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie).
ZodiTrack's `DemoSeeder` ships its own default admin roles — Asset Manager,
Facilities/Field Technician, Maintenance Coordinator, and Location/Site
Manager — each granted a subset of ZodiTrack's product-specific permissions:
`assets.manage`, `assets.transfer`, `transfers.approve`,
`checkouts.manage`, `work_orders.manage`, `assets.dispose`,
`cycle_counts.perform`, `cycle_counts.resolve`. `assets.dispose` (write-off)
is restricted to `Asset Manager` and above so a Field Technician cannot
retire an asset unilaterally. An organization can create additional custom
roles and reassign any permission from the admin panel with no code change,
per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#roles--permissions).

## 19. Workflows & Approval Chains

- **Transfer approval**: a transfer is not final until the destination
  location's Site Manager approves receipt; until then the asset shows as
  `in_transit`, not yet assigned to the destination.
- **Disposal approval**: disposing/writing off an asset above an
  admin-configured value threshold requires a second approval from Asset
  Manager or Finance, since it affects the depreciation schedule and book
  value reporting.
- **Discrepancy resolution**: a cycle-count discrepancy (missing/misplaced
  asset) must be explicitly resolved (found, written off, or corrected
  location) by an Asset Manager before the cycle count can be closed.

## 20. Audit Logs

Every transfer, check-in/check-out, condition log entry, maintenance work
order, disposal, and cycle-count discrepancy resolution is recorded to this
deployment's own audit log with actor, timestamp, and before/after state,
per [audit-logging.md](../../security/audit-logging.md). An asset's detail
page surfaces its full timeline (registration → transfers → checkouts →
maintenance → disposal) as a first-class activity feed, not just a raw log
export.

## 21. Reports & Analytics & Dashboards

Asset utilization by category/location, maintenance cost trend per asset
and per category, depreciation/book-value summary for Finance, overdue
checkout report, cycle-count accuracy trend over time, and a
maintenance-backlog dashboard. Dashboard-builder and scheduled-report
capability per [dashboard-standards.md](../../standards/dashboard-standards.md).

## 22. Integrations

- **Barcode/QR hardware**: Zebra, Honeywell scanner integration alongside
  camera-based mobile scanning.
- **Label printers**: Brother/Dymo/Zebra label printer integration for tag
  generation.
- **Accounting/ERP**: depreciation schedule export/sync to
  [ZodiBusiness](../ZodiBusiness/SPEC.md) or an external fixed-asset ledger.
- **IoT/telemetry (optional)**: usage-hour meters for usage-based
  maintenance triggers on equipment that supports it.
- **Facilities/CMMS data import**: bulk import from legacy spreadsheet or
  CMMS exports during onboarding.

## 23. AI Features

- Predictive maintenance suggestion: flags assets whose failure/repair
  pattern suggests a maintenance interval adjustment, surfaced to the
  Maintenance Coordinator as a recommendation, not an automatic change.
- Discrepancy triage assistant: during cycle-count resolution, suggests the
  most likely explanation (e.g. a recent unapproved transfer) based on
  recent activity near the expected location.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: overdue-checkout sweep, maintenance-due-soon sweep,
  monthly depreciation posting, low-stock spare-parts sweep.
- CLI commands: `track:post-depreciation {period}`,
  `track:generate-work-orders`, `track:export-asset-register`,
  `track:bulk-tag {category_id}`.

## 25. Seed/Demo Data

`DemoSeeder` provisions a demo deployment with 4 locations in a 2-level
hierarchy, 300 seeded assets across 6 categories with realistic
purchase/depreciation data, a mix of checked-out and available shared
assets, 12 months of maintenance work order history, and one completed
cycle count with at least one resolved discrepancy, per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: a cycle count involving 5,000+ expected assets at a
single location must reconcile scanned-vs-expected within 3 seconds of the
count being marked complete.

## 27. Security Requirements

Full baseline from
[security-standards.md](../../security/security-standards.md) applies.
Asset custody data (who currently holds what) is treated as
access-sensitive within this one deployment — location-scoped roles only see
assets within their assigned locations by default, per the RBAC model in
§18.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
an offline-sync idempotency test suite confirming a replayed or
duplicate-submitted scan batch never double-applies a transfer or
check-in/check-out.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). The
scan-sync API is deployed with backward-compatible versioning so mobile
clients with a stale app version queued offline can still sync successfully
after a backend deploy.

## 30. Acceptance Criteria

- An asset registered and tagged can be looked up by scanning its tag on a
  mobile device within the performance budget in §10.
- A transfer is not reflected as complete until the destination location
  approves it, and the asset's location history shows an accurate,
  gap-free custody trail.
- A maintenance schedule correctly generates a work order at its due date,
  and completing the work order updates the asset's next-due date and
  condition.
- A cycle count correctly identifies every missing and misplaced asset
  relative to the expected register, and the cycle count cannot close with
  unresolved discrepancies.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiTrack additionally requires sign-off that the offline-sync idempotency
suite (§28) passes against a simulated flaky-connectivity scenario before
go-live.

## 32. Future Roadmap

- Bluetooth/RFID-based passive location tracking for high-value assets, as
  an alternative to manual scan-based transfers.
- Vendor warranty-claim workflow tied directly to an asset's maintenance
  history.
- Predictive spare-parts demand forecasting based on maintenance history.

## 33. Known Risks

- Offline-sync correctness is the module's highest-risk surface: a
  double-applied or lost scan transaction directly corrupts the custody
  ledger — mitigated by idempotent client-generated transaction IDs (§11)
  and the dedicated test suite (§28), but this warrants continued scrutiny
  as scan volume grows.
- Depreciation-method correctness affects Finance's books directly; an
  incorrect schedule generation is a compliance-relevant defect, not just a
  UX bug — mitigated by the same review rigor applied to ZodiBusiness's
  ledger engine.

## 34. Future Improvements

- Configurable per-category maintenance checklists (not just a single work
  order type).
- Geofencing-based automatic transfer detection for GPS-tagged mobile
  assets.

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture and Core Data Model sections
were revised to the standalone, self-hosted, single-tenant base-codebase
model described in
[architecture/overview.md](../../architecture/overview.md) and
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Queued for Deep-depth expansion: a full ER diagram covering multi-hierarchy
location trees and lot-tracked spare-parts tables, the complete endpoint
catalog (bulk transfer, label-template management endpoints), and a
dedicated `DATA_MODEL.md`/`API_REFERENCE.md` pair matching
[ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.
