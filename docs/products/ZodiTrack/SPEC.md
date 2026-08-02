# ZodiTrack — Product Specification

> Status: **Live — Extend Only**. See
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md)'s status definition and
> [`BUILD_STATE.md`](../../../BUILD_STATE.md)'s protocol for this status.
> ZodiTrack is not built from this spec — it already exists as a complete,
> working, currently-resold product. Everything in this document is a
> description of the real, live codebase confirmed at
> `/home/script/public_html/zoditrack/` on the build VPS, not a target to
> build toward.

## 0. Audit history

**2026-07-31 pass** (filesystem audit only, one file read in full): found
that this spec's §1–§7 described the wrong domain (an ITAM/enterprise-asset
tool) against a codebase that is actually a freight/shipment-tracking
brokerage site. Flagged the mismatch, did not yet rewrite the domain
sections. See `BUILD_STATE.md`'s history for that entry.

**2026-08-02 pass (this document)**: read every file under `admin/` (all 26
`.php` files, not just filenames), the public-facing `track.php`,
`receipt.php`, `index.php`, and the full `customer/` portal directory, and
queried the live `zoditrack` MySQL database directly (`SHOW CREATE TABLE`
for all 12 tables) to confirm the real data model. §1–§34 below are
rewritten from that direct evidence and replace the old ITAM-framed content
entirely — nothing below should be read as inherited from the prior
version. Where something could not be fully verified in this pass, it is
called out explicitly rather than guessed at (see §33 Known Risks and the
Gap List's "Unverified / ambiguous" subsection).

## 1. Vision

ZodiTrack is a customer-facing shipment-tracking and logistics-brokerage
website: a freight forwarder / courier-style operator uses it to register
shipments, manage them through a status lifecycle from pickup to delivery,
and give senders and receivers a public tracking-number lookup with a
live status timeline, barcode/QR code, and a printable receipt — backed by
an admin back office for shipment entry, branch/staff/customer/vendor-mode
management, invoicing, reporting, and notifications.

## 2. Purpose

A small-to-mid-size freight/courier operator needs a single site that (a)
lets its own staff register and update shipments without touching a
database directly, (b) gives customers a self-service tracking experience
so they stop calling/emailing to ask "where is my package," and (c)
produces the basic paperwork (receipts, invoices) and numbers (reports) the
business needs to run day to day. ZodiTrack is exactly that: a lightweight,
single-operator, single-currency shipment tracking and brokerage front
end + back office, not a multi-tenant SaaS platform and not a carrier
itself — `carrier` and `carrier_ref` on a shipment are free-text fields an
operator fills in to reference a real carrier (DHL, FedEx, etc.), not a
live API integration with one.

## 3. Target Market

Independent freight forwarders, customs brokers, and small courier/logistics
resellers who need a branded, self-hosted tracking site and light back
office without buying into an enterprise TMS (transportation management
system). The demo/default branding text found in the code (`sitename`
fallback "TrustShare Logistics", admin fallback "TrustShare Admin") is
itself evidence this is meant to be white-labeled per deployment — every
operator-facing string (site name, title, logo, hero tagline, about text,
contact info) is a row in the `settings` table, not hardcoded.

## 4. Industries

Freight forwarding, small-parcel/courier delivery, air/ocean freight
brokerage, cargo transportation, and packaging/storage services — confirmed
by the public marketing pages actually present: `air-freight.php`,
`ocean-freight.php`, `cargo-transportation.php`,
`packaging-and-storage.php`, plus `about-us.php`, `contact.php`, `faq.php`,
and the legal pages (`privacy-policy.php`, `terms.php`, `refund-policy.php`,
`shipping-policy.php`, `cookie-policy.php`).

## 5. Competitor Analysis

| Capability | Comparable to | ZodiTrack's actual position |
|---|---|---|
| Public tracking-number lookup + timeline | 17TRACK, AfterShip, DHL/FedEx's own tracking pages | Self-hosted, single-operator equivalent — one operator's own shipments only, no multi-carrier aggregation |
| Branded tracking + receipt/invoice site | Trackship, ShipStation's customer-facing tracking page | Same idea, but the operator's own back office (not a plugin bolted onto Shopify/WooCommerce) |
| Small back-office shipment management | A lightweight TMS module (vs. full TMS like project44, FourKites) | Covers shipment CRUD, branches, staff, customers, invoicing, reporting at a scale appropriate for one operator, not multi-tenant enterprise freight networks |
| Customer self-service portal | Carrier account portals (UPS My Choice, FedEx Delivery Manager) | A much lighter version: register/login, add tracking numbers to a personal dashboard, view notifications, update profile — no delivery preferences, no re-routing |

ZodiTrack does not compete with enterprise TMS/visibility platforms — it
occupies the niche of "a freight/courier reseller's own branded tracking
site," which is exactly what the confirmed feature set supports.

## 6. Personas

- **Operator/Admin** — the business owner or senior staff member who logs
  into `admin/` (against the single `admin` table), manages shipments,
  branches, shipment modes, invoices, and settings, and has no
  role-scoped restrictions (see §18 for the caveat about the `staff` table).
- **Staff record** — a row in the `staff` table with a `role` of
  `super_admin`/`admin`/`staff` and an active/inactive `status`, manageable
  from `admin/staff.php`. **Confirmed**: this table and its admin UI exist.
  **Not confirmed**: any login screen or session path that authenticates
  against `staff` rather than `admin` — see §18 and the Gap List.
- **Sender** — the person shipping a package; their name/contact/email/
  address are captured on the shipment record (`tracking.sender_*`
  columns) and they can be notified by email of a status update.
- **Receiver** — the shipment's destination party; same data shape as
  Sender (`tracking.receiver_*` columns), can also be emailed on updates,
  and is the "customer" a public tracking-page visitor usually is.
- **Registered customer** — an account in the `customers` table (separate
  from `admin`/`staff`) who can register, log in, add tracking numbers they
  care about to a personal dashboard (`customer_shipments` join table),
  view in-app notifications, and update their profile.
- **Anonymous site visitor** — anyone who lands on `track.php` and enters a
  tracking number; no account required for the core public tracking flow.

## 7. User Journeys

1. **Shipment intake (admin)**: Operator opens `add-tracking.php`, fills in
   sender/receiver, package details (weight, quantity, description,
   optional image), ship mode, origin branch, dispatch/destination
   locations, dates, priority, payment mode, and shipment/delivery values.
   A tracking number is auto-generated
   (`{track_prefix}-{MM}-{10-digit random}`) and, on save, a first
   `track_update` row ("Shipment registered") is inserted and — if the
   `mail_track_save` setting is `Yes` — an HTML email with the tracking
   number and a track-page link is sent to the receiver.
2. **Status progression (admin)**: Operator opens `edit-tracking.php` for a
   shipment, either edits the shipment's core fields directly or uses the
   "Add Tracking Update" form to append a new `track_update` row (status,
   current location, date/time, optional note, optional delivery/total
   charge). The previous "current" update is flipped to
   `is_current_location='no'`, the new one is marked `'yes'`, and the
   parent `tracking.status`/`current_location` are updated in lock-step.
   The admin can choose to notify the sender, receiver, both, or neither by
   email of this specific update.
3. **Public tracking lookup (anonymous)**: A sender/receiver visits
   `track.php` (or is linked to it from the confirmation/update email),
   enters the tracking number, and sees: a status badge, a 6-step progress
   stepper (Pending → Picked Up → Processing → In Transit → Out for
   Delivery → Delivered, collapsing the 10-status enum onto that display
   subset), the full `track_update` history as a timeline, shipper/receiver/
   shipment/delivery detail cards, a generated barcode + QR code of the
   tracking number, an optional embedded Google Map of the current
   location (if `show_map = Yes`), and — if `allow_print = Yes` — a link to
   `receipt.php` for a printable receipt.
4. **Customer self-service (registered customer)**: A customer registers
   at `customer/register.php` (bcrypt-hashed password, CSRF-protected,
   8-char minimum), logs in at `customer/login.php` (with a "remember me"
   cookie and `session_regenerate_id()` on success), and from
   `customer/dashboard.php` adds an existing tracking number to their
   personal list (validated against the `tracking` table before insert),
   removes one, views in-app notifications, and updates their profile —
   all without needing to know an admin exists.
5. **Invoicing (admin)**: Operator opens `invoices.php`, enters a tracking
   number and due date; the shipment's `delivery_charge` becomes the
   subtotal, `settings.tax_rate` computes the tax, and an
   `INV-{uniqid}` invoice is created as `unpaid`. The operator later marks
   it `paid` (stamping `paid_at`) or `cancelled`. A print-receipt link is
   available per invoice row.
6. **Reporting (admin)**: Operator opens `reports.php`, picks a date range,
   and sees total/delivered counts, paid revenue in that window, average
   delivery days (`DATEDIFF` between dispatch and delivery date for
   delivered shipments), a status breakdown, and a ship-mode breakdown,
   with a one-click CSV export of the underlying shipment rows for that
   range.

## 8. Business Goals

- Let a small freight/courier operator stop fielding "where is my package"
  calls by giving senders and receivers a self-service, branded tracking
  page with a real status history.
- Give the operator's own staff a shipment-entry and status-update
  workflow that doesn't require touching the database, with email
  notifications handled automatically at each step they choose to notify
  on.
- Produce the basic paperwork (printable receipts, invoices with tax) and
  numbers (CSV reports, revenue-by-period) the business needs without a
  separate invoicing tool.
- Support light multi-branch operation (branches, shipment modes,
  per-shipment origin branch) for an operator running more than one
  physical location.

## 9. Functional Requirements

The functional surface below is entirely **already implemented** —
confirmed by direct code read of every listed screen, not proposed work.

- **Shipment lifecycle**: create (`add-tracking.php`), list/search/filter/
  paginate/bulk-delete (`shipments.php`), view read-only detail
  (`view-details.php`), edit + append tracking updates + delete a tracking
  update (`edit-tracking.php`), delete with cascade to `track_update`
  (`dashboard.php`, `shipments.php`). Canonical status enum (ten values,
  confirmed identical across `add-tracking.php`, `edit-tracking.php`,
  `track.php`, `receipt.php`): `Pending`, `Picked Up`, `Processing`,
  `In Transit`, `Arrived at Facility`, `Out for Delivery`, `Delivered`,
  `Delayed`, `Failed Delivery`, `Returned`.
- **Tracking-number generation**: `{track_prefix}-{MM}-{random 10-digit}`,
  prefix configurable in Settings.
- **Public tracking lookup + timeline**: `track.php`, exact-match then
  case-insensitive fallback lookup, full `track_update` history, barcode
  (JsBarcode CODE128) and QR code (qrcodejs) rendering of the tracking
  number, optional embedded Google Maps view of current location.
- **Receipt generation**: `receipt.php`, a printable receipt/invoice-style
  page keyed by tracking number with a computed (not DB-stored) receipt
  number and the full update timeline.
- **Branches**: CRUD + active/inactive toggle (`branches.php`); a shipment
  optionally references an origin `branch_id`.
- **Shipment modes**: CRUD + active/inactive toggle + sort order
  (`shipment-modes.php`); a shipment's `ship_mode` is a free-text column
  populated from this configurable list, not a foreign key.
- **Staff**: add/edit/toggle active-inactive (`staff.php`); passwords are
  bcrypt-hashed (`password_hash`/`PASSWORD_BCRYPT`) on create and on
  password change.
- **Customers (admin side)**: search/paginate/suspend/activate
  (`customers.php`); per-customer shipment count shown inline.
- **Customer self-service portal**: register, login (with remember-me and
  session regeneration), forgot/reset password, profile update, logout,
  and a dashboard to add/remove tracked shipment numbers and view
  notifications (`customer/*.php`).
- **Invoices**: create from a shipment's `delivery_charge` +
  `settings.tax_rate`, mark paid/cancelled, filter by status
  (`invoices.php`).
- **Notifications**: send an in-app notification (with best-effort email
  via PHP's `mail()`) to a single customer or in bulk to all active
  customers, typed against a fixed enum (`shipment_created`,
  `status_update`, `out_for_delivery`, `delivered`, `delayed`, `failed`)
  (`notifications.php`).
- **Activity log**: append-only log of admin actions (delete shipment, add/
  edit/toggle staff, update customer status, create/pay/cancel invoice,
  send notification) with actor, action, free-text details, IP address,
  and timestamp; paginated and date-filterable (`activity-log.php`,
  `activity_logs` table).
- **Settings**: general (site name/title/URL/logo upload/address/phone/
  meta description/hero tagline/about text/footer-visibility toggles),
  shipping (tracking prefix, invoice terms text, allow-print toggle,
  show-map toggle), email sender identity + notification toggles, SMTP
  credentials, currency + tax rate, and social links — all in one
  `settings` table row, edited section-by-section (`settings.php`).
- **Reports**: date-range totals/delivered/revenue/average-delivery-days,
  status and ship-mode breakdowns, CSV export (`reports.php`).
- **Admin account**: the single `admin` row can change its own username/
  password from `account.php` — **see §27 for a concrete bug in this
  screen**.

## 10. Non-Functional Requirements

No load-testing, uptime, or SLO evidence was found in the codebase (no
monitoring config, no `.github/workflows` performance gates observed under
this product's tree). The general baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) applies to
any new work on this product, but nothing below should be read as
"already met" — it is a target for extension work, not a confirmed
property of the live site:

- Shipment search/filter (`shipments.php`) is already parameterized and
  paginated (20/page) rather than loading the full table client-side —
  keep this pattern for any new list screens.
- Reports/CSV export run ad hoc `SELECT`s per stat with no caching;
  fine at the shipment volumes implied by the live schema's low
  `AUTO_INCREMENT` values (single digits to low tens per table at audit
  time), but would need indexing/caching attention before assuming it
  scales to a busy multi-branch operator.

## 11. Architecture

ZodiTrack is **native procedural PHP**, not a framework: page-routed
`.php` files under the document root, a single global `mysqli` connection
opened in `db.php` (included by nearly every page) and shared application
config/settings loaded from a `settings` DB row in `config.php`. There is
no MVC layer, no router, no ORM, and no dependency-injection container.
Shared chrome is `header.php`/`footer.php` on the public site and a
separate `admin/header.php`/`admin/footer.php` pair for the back office
(custom CSS design system, prefixed `adm-*`, plus Bootstrap 5 loaded from
a CDN and a vendored copy of Chart.js, Select2, jQuery File Upload,
JustGage, jVectorMap, and MDI icons under `admin/vendors/` — **these are
front-end JS/CSS libraries, not a business "vendor/supplier" management
feature**; there is no vendor/supplier table in the database, correcting
an assumption in the prior audit pass). A `dompdf` library directory is
present at the site root but was not confirmed in use by `receipt.php` in
this pass — `receipt.php` renders a printable HTML page styled for
print, not a server-generated PDF, in the code actually read.

Authentication is two separate, non-shared systems:

- **Admin**: a single `admin` table (`id`, `username`, `password`), signed
  in via `admin/signin.php` with `password_verify()` against a bcrypt hash,
  session key `$_SESSION['ADMINID']`. No role/permission distinction exists
  at this layer — whoever holds the one admin credential has full access
  to every `admin/*.php` screen.
- **Customer**: a separate `customers` table with its own bcrypt
  passwords, session keys (`CUSTOMER_ID`/`CUSTOMER_NAME`/`CUSTOMER_EMAIL`),
  registration, login (with remember-me cookie and
  `session_regenerate_id()`), forgot/reset password, and profile update,
  entirely independent of the admin session.

There is no tenancy model and no ZodiCore dependency: this is one
operator's single, standalone deployment against one `zoditrack` MySQL
database — closer in shape to
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)
than to a multi-tenant SaaS product, though this codebase predates that
convention and was not built from it.

## 12. Technology

Native PHP (procedural, `mysqli` extension — a mix of the classic
`mysqli_query`/`mysqli_prepare`/`mysqli_stmt_*` function API and the
object style `$link->prepare(...)`/`$link->query(...)`, both used across
different files in the same codebase); MySQL/MariaDB (schema confirmed
live: 12 tables, InnoDB, `utf8mb4`); front end is Bootstrap (v5 admin,
older Bootstrap + a themed marketing template on the public site),
Chart.js 4 (admin dashboard charts), JsBarcode + qrcodejs (public tracking
page barcode/QR rendering), vanilla JS for all interactivity (no SPA
framework, no build step/bundler evidenced). Outbound email is PHP's
built-in `mail()` function wrapped in a local `sendMail()` helper in
`config.php` — SMTP settings exist in the `settings` table
(`smtp_host`/`smtp_port`/`smtp_user`/`smtp_pass`/`smtp_secure`) but no code
path using them (e.g. PHPMailer/SwiftMailer) was found in the files read;
`mail()` is called directly.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Public Site | Marketing pages (air/ocean freight, cargo transportation, packaging & storage, about, contact, FAQ), legal pages, public tracking lookup, printable receipt |
| Customer Portal | Registration, Login/Remember-me, Forgot/Reset Password, Profile Update, Personal Tracked-Shipment List, In-App Notifications |
| Shipment Management | Shipment CRUD, Tracking-Number Generation, Status/Update Timeline, Image Upload |
| Operations Config | Branches, Shipment Modes |
| People | Staff Records, Customer Records (admin-side suspend/activate) |
| Billing | Invoices (create/paid/cancelled), Tax-Rate Application |
| Comms | Notifications (single/bulk, in-app + best-effort email) |
| Oversight | Activity Log, Reports (date-range stats + CSV export) |
| System | Settings (general/branding, shipping/tracking, email, SMTP, currency/tax, social), Admin Account |

## 14. Core Data Model

Schema below is the **live, confirmed** `zoditrack` database (via
`SHOW CREATE TABLE` against the production DB at audit time), not a
proposed model:

| Table | Key columns |
|---|---|
| `admin` | id, username, password (bcrypt hash) |
| `staff` | id, full_name, email (unique), username (unique), password (bcrypt hash), role (`super_admin`\|`admin`\|`staff`), status (`active`\|`inactive`), last_login |
| `customers` | id, full_name, email (unique), phone, password (bcrypt hash), address, reset_token, reset_expires, email_verified, status (`active`\|`suspended`), last_login |
| `branches` | id, name, city, country, address, phone, email, is_active |
| `shipment_modes` | id, name (unique), description, icon, is_active, sort_order |
| `tracking` | id, tracking_number, sender_*/receiver_* (name/contact/email/address/address2), status, dispatch_location, destination, dispatch_date, delivery_date, delivery_time, pdesc, ship_mode, carrier, carrier_ref, weight, quantity, payment_mode, image, shipment_value, delivery_charge, customer_id, branch_id, priority (`Standard`\|`Express`\|`Overnight`), notes, current_location, date |
| `track_update` | id, track_num, status, date, time, note, current_location, is_current_location, delivery_charge, total_charge, updated_at |
| `customer_shipments` | id, customer_id, tracking_number, added_at, notes — join table for a customer's personal tracked-shipment list, unique on (customer_id, tracking_number) |
| `invoices` | id, invoice_number (unique), tracking_number, customer_id, customer_name, customer_email, subtotal, tax, total, status (`unpaid`\|`paid`\|`cancelled`), due_date, paid_at, notes |
| `notifications` | id, customer_id, tracking_number, type (`shipment_created`\|`status_update`\|`out_for_delivery`\|`delivered`\|`delayed`\|`failed`), message, channel (`email`\|`sms`\|`whatsapp`\|`in_app` — only `in_app` is actually written by the code read), is_read, sent_at |
| `activity_logs` | id, admin_id, action, details, ip_address, created_at |
| `settings` | id (single row, id=1), sitename, site_title, site_url, track_prefix, track_num, invoice_terms, allow_print, show_map, email_name, email_address, phone, address, address2, show_address/show_phone/show_email, site_logo, mail_track_update, mail_track_save, currency, currency_symbol, tax_rate, smtp_host/port/user/pass/secure, social_facebook/twitter/linkedin/instagram, meta_description, about_text, hero_tagline |

Notable real-world shape, not a design choice a rebuild should copy
uncritically: `tracking.ship_mode` and `tracking.carrier` are free-text
columns, not foreign keys to `shipment_modes` or a carriers table; there is
no `vendors` table anywhere in the schema despite a `vendors/` directory
existing under `admin/` — that directory holds front-end JS/CSS libraries,
not a business entity (see §11).

## 15. Key API Endpoints

**There is no JSON/REST API.** Every "endpoint" is a server-rendered HTML
page reached by full-page GET/POST, listed here for completeness rather
than as a `/api/v1/...` catalog (contrast with other Zodize product specs'
§15 convention, which assumes a real API layer):

| Method | Path | Purpose |
|---|---|---|
| GET/POST | `/track.php` | Public tracking-number lookup |
| GET | `/receipt.php?tracking_number=...` | Printable receipt |
| GET/POST | `/customer/login.php`, `/register.php`, `/forgot-password.php`, `/reset-password.php` | Customer auth flows |
| GET/POST | `/customer/dashboard.php?tab=...` | Customer self-service (add/remove tracked shipment, view notifications, edit profile) |
| GET/POST | `/admin/signin.php` | Admin login |
| GET/POST | `/admin/add-tracking.php`, `/admin/edit-tracking.php` | Shipment create/edit + tracking-update append |
| GET | `/admin/shipments.php`, `/view-details.php` | Shipment list (filtered/paginated) and read-only detail |
| GET/POST | `/admin/branches.php`, `/staff.php`, `/customers.php`, `/shipment-modes.php`, `/invoices.php`, `/notifications.php`, `/settings.php`, `/account.php` | Admin CRUD/config screens described in §9 |
| GET | `/admin/reports.php`, `/admin/reports.php?export=csv` | Reports view and CSV export |
| GET | `/admin/activity-log.php` | Activity log (paginated, date-filterable) |

A real API layer (for a future mobile app, a carrier-integration webhook
receiver, or partner access) is a genuine gap — see the Gap List.

## 16. Events

There is no event/dispatcher system. State changes are direct DB writes in
the same request that triggers them (e.g., saving a tracking update writes
`track_update`, updates `tracking.status`/`current_location`, and
optionally calls `sendMail()`, all inline in `edit-tracking.php` — no
`shipment.status_changed` event object exists to hook into). Any future
extension that wants an event/webhook system is new work, not a gap in
something that already exists structurally.

## 17. Notifications, Emails, SMS, Push

| Trigger | In-app | Email | SMS | Push |
|---|---|---|---|---|
| Shipment registered | — | ✔ (to receiver, if `mail_track_save = Yes`) | — | — |
| Tracking status update | — | ✔ (to sender/receiver/both/neither, admin's choice per update, if selected) | — | — |
| Admin-sent notification (single or bulk) | ✔ (`notifications` table, `channel='in_app'`) | ✔ (best-effort, same message body) | — | — |

The `notifications.channel` enum includes `sms` and `whatsapp` values, but
no code path writing or sending through those channels was found — they
exist in the schema, not in the delivery logic. Email is sent synchronously
via PHP's `mail()` inside the same HTTP request (no queue), including for
"bulk send to all active customers," which loops and calls `sendMail()`
once per customer inline. See
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md) for
the baseline any extension work should move this toward.

## 18. Permissions & Roles

**This is the most significant structural gap found in this audit.** The
`staff` table has a `role` enum (`super_admin`/`admin`/`staff`) and an
`active`/`inactive` `status`, and `admin/staff.php` provides full add/edit/
activate/deactivate management of it — but no login screen, session
handling, or authorization check anywhere in the files read authenticates
against the `staff` table. The only admin authentication path found
(`admin/signin.php`) checks the separate, single-row-in-practice `admin`
table with no role column at all. In other words: **the admin back office
today has exactly one privilege level** — whoever has the one `admin`
credential can reach every screen — and the staff-role system that the UI
implies (distinguishing a `staff` role from `admin` from `super_admin`) is
not wired to anything that restricts access. Any extension work that
assumes staff members can log in with scoped permissions must build that
login/authorization path; it does not exist yet. This is called out again
in the Gap List rather than assumed away.

## 19. Workflows & Approval Chains

No multi-step approval workflow exists anywhere in the code read — every
admin action (create/edit/delete a shipment, mark an invoice paid, suspend
a customer, toggle a branch) is a single-actor, single-step write with no
second approver, no draft/pending state, and no maker-checker pattern. The
closest thing to a "workflow" is the tracking-update sequence itself
(§7 journey 2): appending a new status flips the previous update's
`is_current_location` flag and updates the parent shipment row in the same
request — a state transition, not an approval gate.

## 20. Audit Logs

`activity_logs` records `admin_id`, `action`, free-text `details`,
`ip_address`, and `created_at` for: delete shipment, add/edit/toggle staff,
update customer status, create/mark-paid/mark-cancelled invoice (create and
mark-paid confirmed logged; mark-cancelled was not confirmed to write a log
row in the code read), and send notification (single/bulk). **Not logged**,
per the code read: shipment create/edit itself calls
`log_activity($link, $adminid, ...)` conditionally
(`if (function_exists('log_activity'))`) in `edit-tracking.php`, but that
function's definition was not located in any file read in this pass —
whether shipment edits are actually reaching the log table could not be
confirmed and should be verified directly (e.g. by querying
`activity_logs` after making a test edit) before relying on it. Branch and
shipment-mode CRUD, settings changes, and admin account changes were not
observed writing to `activity_logs` at all. See
[audit-logging.md](../../security/audit-logging.md) for the baseline this
falls short of.

## 21. Reports & Analytics & Dashboards

Confirmed in `admin/dashboard.php` and `admin/reports.php`: total/
delivered/in-transit/pending/failed-or-delayed shipment counts, customer
count, paid-invoice revenue, a 6-month shipment-volume bar chart and a
status-distribution doughnut chart (both Chart.js, computed with per-bucket
`SELECT COUNT(*)` queries, not a pre-aggregated table), a recent-shipments
table, and a recent-activity feed on the dashboard; a date-range report
screen with the same total/delivered/revenue/avg-delivery-days stats,
status and ship-mode breakdowns, and CSV export. No dashboard-builder, no
scheduled/emailed reports, and no saved custom report views were found —
these are gaps, see below.

## 22. Integrations

- **Google Maps embed** (iframe, `maps.google.com/maps?q=...&output=embed`)
  for showing a shipment's current location on the public tracking page,
  toggled by the `show_map` setting — a read-only embed, not the Maps API.
- **PHP `mail()`** for all outbound email (see §17).
- No carrier API integrations (DHL/FedEx/UPS/etc.), no payment gateway
  integration (invoices track paid/unpaid status manually; no Stripe/
  PayPal/etc. webhook or checkout flow was found), no SMS/WhatsApp
  provider despite the schema anticipating those channels, and no
  accounting/ERP export. All of these are gaps relative to a competitive
  freight-tracking SaaS, not existing integrations.

## 23. AI Features

None found anywhere in the codebase read. No AI/ML code, no third-party AI
API calls, no "smart suggestion" features. This is a complete gap relative
to the AI Features section other Zodize product specs include, and any
such feature is entirely new work.

## 24. Automation, Scheduled Jobs, CLI Commands

None found. There is no cron/scheduler configuration, no CLI entrypoint
script, and no queue worker anywhere in the tree read. Every action
(including bulk email notification sends) happens synchronously inside an
HTTP request initiated by an admin click. This is a genuine gap for
anything that should run on a schedule (e.g., a delayed-shipment sweep, a
low-activity staff-inactivity sweep, a scheduled report email) — none of
that exists today.

## 25. Seed/Demo Data

The live database has minimal seed data (low `AUTO_INCREMENT` values at
audit time: e.g. `admin` at 2, `customers` at 4, `tracking` at 9,
`track_update` at 15, `invoices` at 2) — this reads as real operational
data from a live, lightly-used deployment, not a generated demo dataset,
and there is no `DemoSeeder`-equivalent script found in the codebase. Any
future work needing a realistic demo/staging dataset (per this repo's Demo
Standard in [README.md](../../../README.md)) would need to build one from
scratch; none exists today.

## 26. Performance Requirements

No performance budget, load test, or profiling artifact was found in the
codebase. See §10 — the general baseline in
[performance-standards.md](../../quality/performance-standards.md) applies
to new work, but nothing here should be read as a confirmed, already-met
property of the live site.

## 27. Security Requirements

The general baseline in
[security-standards.md](../../security/security-standards.md) applies to
all new work. Concrete findings from this pass, all directly observed in
the code — **not inferred**:

- **Confirmed bug — plaintext password write + SQL injection surface in
  `admin/account.php`**: the admin's own username/password change handler
  builds its `UPDATE` with raw string concatenation
  (`"UPDATE admin SET username = '$adminuser', password = '$adminpass' WHERE username = '$username'"`)
  instead of a prepared statement, and — critically — never calls
  `password_hash()` on the new password before storing it, even though
  `admin/signin.php`'s login check uses `password_verify()` against a
  bcrypt hash. In practice this means: (a) the username/password values are
  interpolated directly into SQL with only `htmlspecialchars`-style
  sanitization, not parameterization, and (b) changing the admin password
  through this screen writes it in plaintext, which will make the *next*
  login attempt fail `password_verify()` against a non-bcrypt string. This
  is the single highest-severity, concrete defect found in this audit and
  should be first in line for any security-focused follow-up work on this
  product.
- **Inconsistent CSRF protection.** A CSRF token (`$_SESSION['csrf_token']`
  + `hash_equals()` check against a posted `_token`) is present and
  correctly implemented in `add-tracking.php`, `edit-tracking.php`,
  `shipments.php`, `branches.php`, `shipment-modes.php`, and all of the
  `customer/*.php` auth/portal screens. It is **absent** (no token field,
  no server-side check) on: `dashboard.php`'s shipment-delete POST handler,
  `staff.php` (add/edit/toggle-status), `customers.php`
  (suspend/activate), `invoices.php` (create/mark-paid/mark-cancelled),
  `notifications.php` (send), every save-section in `settings.php`, and
  `account.php`. Any of these unprotected, session-cookie-authenticated
  POST endpoints could be triggered cross-site.
- **No visible rate limiting or lockout** on `admin/signin.php` or
  `customer/login.php` — no failed-attempt counter, no CAPTCHA, no
  progressive delay.
- **`admin/signin.php` does not call `session_regenerate_id()`** on
  successful login, unlike `customer/login.php` which does — a minor
  session-fixation-hardening inconsistency between the two auth systems.
- **`smtp_pass` is stored in plaintext** in the `settings` table with no
  encryption at rest.
- **`customers.email_verified`** exists as a column but no verification-
  email-send or verification-confirm code path was found in
  `customer/register.php` or elsewhere in the files read — the column
  appears unused/unenforced.
- Elsewhere, the code is meaningfully better than the above list suggests
  in isolation: `add-tracking.php`, `edit-tracking.php`, and `shipments.php`
  consistently use parameterized `mysqli` prepared statements, validate
  email addresses with `filter_var(..., FILTER_VALIDATE_EMAIL)`, validate
  uploaded images with both an extension allow-list and `getimagesize()`/
  `exif_imagetype()`, and escape all output with
  `htmlspecialchars(..., ENT_QUOTES, 'UTF-8')`. The account.php bug above
  is an outlier in an otherwise reasonably careful codebase, not
  representative of the whole.

## 28. Testing Requirements

No test suite, test framework config (no `phpunit.xml`, no `/tests`
directory), or CI test workflow was found anywhere in the product's tree.
The full baseline in
[testing-standards.md](../../development/testing-standards.md) applies to
any new work, but there is currently zero automated test coverage to build
on — a fresh gap, not a partially-met one.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md) for
any new work. The live product is deployed directly as PHP files under
`/home/script/public_html/zoditrack/` on a shared-hosting-style VPS with a
local MySQL/MariaDB instance — there is no containerization, no CI/CD
pipeline, and no blue/green or staged-rollout mechanism observed in the
tree read. Changes to the live site are, as far as this audit could
determine, made by editing the files in place.

## 30. Acceptance Criteria

Restated from the confirmed, already-working behavior (useful as a
regression baseline for any future change, not as unmet targets):

- A shipment created in `add-tracking.php` is immediately findable by its
  generated tracking number on the public `track.php` lookup, with its
  first status-history entry ("Shipment registered") already present.
- Appending a tracking update in `edit-tracking.php` updates the parent
  shipment's displayed status/location on `track.php` and `receipt.php`
  and, if an email-notify option was selected, sends the corresponding
  email to the correct recipient(s).
- A customer can register, log in, add a valid tracking number to their
  dashboard, and see it persist across sessions (`customer_shipments`).
- An invoice created against a shipment computes tax correctly from the
  current `settings.tax_rate` and can be transitioned unpaid → paid or
  unpaid → cancelled, each reflected immediately in the invoices list and
  its status filter tabs.

## 31. Production Checklist

Before any further work ships to this **live, currently-resold** product,
per
[production-readiness-checklist.md](../../checklists/production-readiness-checklist.md):
at minimum, the `account.php` plaintext-password/SQL-injection defect in
§27 should be fixed and verified (confirm a password change round-trips
through `password_hash`/`password_verify` correctly) before any other admin
security work, since it is a live, exploitable defect in a product real
customers currently use — not a hypothetical.

## 32. Future Roadmap

Ordered roughly by leverage relative to what's missing (see the Gap List
for the full inventory this is drawn from):

- A real JSON/REST API layer, so a future mobile app or carrier-integration
  webhook receiver has something to call instead of scraping server-
  rendered HTML.
- An actual staff-login path with role enforcement, so the existing
  `staff` table's role/status fields do something.
- A queued/async email pipeline (even a simple DB-backed queue + cron
  worker) so bulk notification sends don't block an admin's HTTP request
  and don't risk partial sends on failure.
- Real carrier-API integration (at minimum DHL/FedEx/UPS tracking-number
  lookup) so `carrier`/`carrier_ref` become more than free-text fields.

## 33. Known Risks

- **The account.php defect (§27) is a live, present-tense risk**, not a
  theoretical one — it affects the one credential that guards the entire
  admin back office.
- **Inconsistent CSRF coverage (§27)** means several real admin actions
  (suspending a customer, sending a bulk notification, changing SMTP
  credentials) can currently be triggered by a crafted cross-site request
  against a logged-in admin session.
- **The `staff`/role system's disconnection from actual authentication
  (§18)** is a risk if anyone extending this product assumes role-based
  access already works — it does not, and building on top of that false
  assumption would silently grant a "staff" user the same access as the
  single admin account, or fail to grant any access at all, depending on
  what gets built.
- Synchronous, unqueued email sending (§17, §24) means a large "notify all
  customers" action ties up a single PHP request for as long as `mail()`
  calls take, with no retry if one recipient's send fails partway through
  the loop.

## 34. Future Improvements

- Move `smtp_pass` and other credential-shaped settings values to an
  encrypted-at-rest column or a secrets manager rather than a plaintext
  `settings` column.
- Add a verification-email flow that actually uses the existing
  `customers.email_verified` column.
- Add automated tests (there are currently none) starting with the
  highest-risk surfaces identified in §27 (the account.php password-change
  path, and CSRF coverage on the currently-unprotected admin screens).

## Gap list

A genuine feature-by-feature comparison, per the correction notice's
requirement — grounded entirely in the code and schema read in this pass.

### Confirmed-present capabilities (do not propose as new work)

- Tracking-number generation (`{prefix}-{MM}-{10-digit random}`) and public
  lookup with exact-then-case-insensitive fallback matching.
- 10-state shipment status enum with a public 6-step progress stepper and
  a full status-history timeline (`track_update` table).
- Shipment CRUD with sender/receiver detail, package description/weight/
  quantity/image, ship mode, priority (Standard/Express/Overnight), payment
  mode, shipment/delivery values, carrier + carrier reference (free-text),
  and internal notes.
- Barcode (CODE128) and QR-code rendering of the tracking number on the
  public tracking page.
- Optional embedded Google Maps view of a shipment's current location.
- Printable receipt generation (`receipt.php`) with a computed receipt
  number and the full timeline.
- Branches (CRUD + active/inactive) and Shipment Modes (CRUD + active/
  inactive + sort order), both referenced by shipments.
- Staff records (CRUD-ish: add/edit/toggle active, with a role field) —
  **but see the login-path caveat in §18/the ambiguous list below**.
- Customer records: admin-side search/paginate/suspend/activate, plus a
  full separate customer self-service portal (register, login with
  remember-me, forgot/reset password, profile update, personal tracked-
  shipment list via `customer_shipments`, in-app notification viewing).
- Invoices: create from a shipment's delivery charge + tax rate, mark
  paid/cancelled, filter by status, print-receipt link per invoice.
- Notifications: single or bulk (all active customers) in-app + best-
  effort email, typed against a fixed enum.
- Activity log: paginated, date-filterable, actor/action/details/IP/
  timestamp for the admin actions listed in §20.
- Settings: general/branding (including logo upload), shipping/tracking
  config, email sender identity + toggles, SMTP credentials, currency +
  tax rate, and social links — six independently-saved sections in one
  screen.
- Reports: date-range totals/delivered/revenue/average-delivery-days,
  status and ship-mode breakdowns, one-click CSV export.
- Bulk shipment delete and single shipment delete (with cascade to
  `track_update`), both from `shipments.php`/`dashboard.php`.

### Unverified / ambiguous (confirm before building on these assumptions)

- Whether `staff` table members can actually log in anywhere — no such
  path was found in the files read (§18). Treat "staff can log in with
  scoped permissions" as **false** until proven otherwise.
- Whether `log_activity()` (called conditionally in `edit-tracking.php`)
  is defined and actually logs shipment edits — its definition was not
  located in this pass (§20).
- Whether SMTP settings (`smtp_host`/`port`/`user`/`pass`/`secure`) are
  actually used by any mail-sending code path, versus `mail()` always
  being called directly regardless of those settings (§12, §17) — the
  settings exist and are editable, but no SMTP-library usage was found in
  the files read.
- Whether `dompdf` (present as a library directory at the site root) is
  used anywhere — not confirmed in `receipt.php`, which appears to render
  a print-styled HTML page rather than a server-generated PDF, but not
  every file in the tree was read.

### Genuinely missing, relative to a competitive freight-tracking SaaS and this repo's own standards

- **No JSON/REST API** — everything is server-rendered HTML; no `/api/v1`
  surface exists for a mobile app, carrier webhook receiver, or partner
  integration (§15).
- **No real carrier-API integration** — `carrier`/`carrier_ref` are
  free-text fields, not linked to a live DHL/FedEx/UPS tracking API
  (§22).
- **No payment gateway integration** — invoices are marked paid/cancelled
  manually with no Stripe/PayPal/etc. checkout or webhook (§22).
- **No functioning role-based access control for admin users** despite a
  `staff` table with a role enum — see §18, the single largest structural
  gap found.
- **No 2FA/MFA** on admin or customer login.
- **No rate limiting or lockout** on either login form (§27).
- **Inconsistent CSRF protection** across roughly half of the admin
  screens with state-changing POST handlers (§27) — a genuine security
  gap against this repo's own
  [security-standards.md](../../security/security-standards.md), not just
  a competitive-feature gap.
- **No queue/async job system** — bulk email notifications and all
  outbound mail run synchronously inline in the request (§17, §24).
- **No scheduled jobs/cron/CLI** of any kind (§24) — no delayed-shipment
  sweep, no scheduled report email, nothing.
- **No automated tests** anywhere in the product's tree (§28).
- **No i18n/multi-language support** — all UI strings are hardcoded
  English.
- **No soft-delete** — shipment and (implicitly) other deletes are hard
  `DELETE`s with cascading removal of `track_update` rows; nothing is
  recoverable after deletion.
- **No audit trail for customer-side actions** — `activity_logs` only
  records `admin_id`-attributed actions, never a customer's own actions
  (e.g., adding/removing a tracked shipment).
- **No per-shipment currency** — `currency`/`currency_symbol` are single,
  site-wide settings values, not attached to individual shipments or
  invoices.
- **No dashboard-builder or scheduled/emailed reports** — reports are
  view-and-export-on-demand only (§21).
- **No companion `DATA_MODEL.md`/`API_REFERENCE.md` documents** matching
  this repo's own documentation-depth convention for other products.

## Roadmap (spec depth)

This spec is now grounded directly in the live codebase and database
(Foundation-depth, evidence-based). Queued for Deep-depth expansion: a full
ER diagram of the 12 confirmed tables, a screen-by-screen inventory of
`admin/settings.php`'s remaining sections not walked line-by-line in this
pass (SMTP/currency/social save-handlers were read; the full rendered form
markup was not), direct confirmation of the `log_activity()` and SMTP-usage
open questions in the Gap List's "Unverified" subsection, and — if this
product is ever pointed at a real API build-out — a dedicated
`DATA_MODEL.md`/`API_REFERENCE.md` pair matching
[ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.
