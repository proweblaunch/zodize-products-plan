# ZodiReach — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; exhaustive ER diagrams and a full endpoint catalog
> are queued — see [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiReach is the marketing automation and omnichannel outreach system for
organizations that need to segment their audience, run email/SMS/push
campaigns and drip sequences, and prove attribution — without sacrificing
deliverability or violating consent law — using the same customer data
already living in their Zodize products rather than a re-exported CSV.

## 2. Purpose

Marketing teams routinely operate a segmentation tool, an email platform, an
SMS platform, and a separate consent/compliance process that don't agree
with each other, and attribution back to revenue is stitched together by
hand. ZodiReach exists because segmentation, sending, consent, and
attribution are one connected problem: a segment is only as good as the data
behind it, a send is only as good as its deliverability posture, and
attribution is only trustworthy if consent and suppression were honored in
the first place.

## 3. Target Market

Mid-market to enterprise marketing teams running email/SMS/push programs at
volume (10K–10M+ contacts), particularly those already running another
Zodize product (e.g. [ZodiCommerce](../ZodiCommerce/SPEC.md),
[ZodiBusiness](../ZodiBusiness/SPEC.md)) who want outreach built on the same
customer/contact data rather than syncing it to a third-party ESP.

## 4. Industries

Cross-industry — retail/e-commerce (post-purchase and abandoned-cart
flows), SMB services (appointment/renewal reminders), and any product with a
contact/customer base needing lifecycle marketing; deepest native
integration is with [ZodiCommerce](../ZodiCommerce/SPEC.md) customer/order
data and [ZodiBusiness](../ZodiBusiness/SPEC.md) CRM contacts.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Email/SMS marketing automation | Klaviyo, Iterable | Native access to ZodiCommerce order/customer data, no data-sync connector required |
| Enterprise marketing automation | HubSpot Marketing Hub, Marketo | Segmentation and consent share ZodiCore's tenant/RBAC model instead of a separate marketing-cloud login |
| Transactional + marketing send infrastructure | Braze | One notification fabric (ZodiCore's) powers both product notifications and marketing sends, avoiding duplicate infrastructure |
| Deliverability/reputation management | SendGrid/Twilio SendGrid deliverability tooling | Sender reputation monitoring built into the campaign send pipeline, not a bolt-on dashboard |
| A/B testing and attribution | Mailchimp, Braze experimentation | Attribution ties directly to ZodiCommerce order records for real revenue attribution, not just click-through proxies |

## 6. Personas

- **Marketing Manager** — builds segments, campaigns, and automation
  sequences; owns overall program strategy.
- **Campaign Coordinator** — builds and schedules individual email/SMS/push
  sends within campaigns, monitors delivery.
- **Compliance/Privacy Officer** — manages consent policy, suppression
  lists, and unsubscribe/data-subject-request handling.
- **Deliverability Specialist** — monitors sender reputation, bounce/spam
  rates, and domain/IP warm-up.
- **Contact/Recipient** — the end recipient of campaigns; manages their own
  consent preferences via a self-service preference center.
- **Executive/Analyst** — consumes campaign performance and revenue
  attribution dashboards.

## 7. User Journeys

1. **Segment build to campaign send**: Marketing Manager builds a segment
   ("customers who purchased in the last 90 days but not in the last 30")
   using saved-filter-style criteria against contact and order data →
   creates an email campaign in the campaign builder → previews rendering
   across clients → schedules the send → Campaign Coordinator monitors
   delivery/open/click metrics in real time.
2. **Drip/automation sequence**: Marketing Manager configures a
   welcome-series automation triggered by `contact.subscribed` → sequence
   sends email 1 immediately, email 2 after 3 days if no purchase occurred,
   and exits the sequence early if the contact makes a purchase → each step
   is evaluated per-contact against live data, not a static send list.
3. **A/B test to winner rollout**: Campaign Coordinator creates an A/B test
   on subject line for a campaign → system sends variant A and B to a
   sample split of the segment → after the test window, the
   statistically-better-performing variant (by open or click rate,
   configurable) is automatically sent to the remaining segment, with the
   full result recorded for the campaign's analytics.
4. **Consent and suppression enforcement**: a contact clicks unsubscribe in
   an email footer → they're immediately added to the suppression list →
   every future campaign send, including ones already scheduled, excludes
   them before send-time evaluation → Compliance Officer can audit the full
   consent history (opt-in source, opt-in date, opt-out date) for any
   contact on request.
5. **Deliverability incident response**: bounce rate on a sending domain
   crosses a configured threshold mid-campaign → the Deliverability
   Specialist is alerted → the system automatically pauses further sends
   from that domain → Specialist reviews the bounce/complaint detail,
   remediates (e.g. list-hygiene cleanup), and manually resumes sending
   once resolved.

## 8. Business Goals

- Increase attributable revenue per campaign by tying sends directly to
  ZodiCommerce order data instead of proxy click-through metrics.
- Protect sender reputation proactively (automatic pause-on-threshold)
  instead of discovering deliverability damage after inbox placement drops.
- Reduce consent/compliance risk by making suppression enforcement
  structural (checked at send-time, every time) rather than a manual list
  export step.

## 9. Functional Requirements

- Contact segmentation: rule-based and saved segments against contact
  attributes, behavioral events, and (where integrated) order/purchase
  history, with live (dynamic) or snapshot (static) segment modes.
- Campaign builder: email (drag-and-drop + HTML), SMS, and push campaign
  creation with template library, personalization tokens, and
  multi-client/device preview.
- Drip/automation sequences: trigger-based multi-step sequences with
  conditional branching, wait steps, and exit conditions evaluated
  per-contact.
- A/B testing: subject line, content, and send-time variant testing with
  configurable winner criteria and automatic or manual winner rollout.
- Deliverability/sender reputation management: bounce/complaint rate
  monitoring, automatic send-pause on threshold breach, domain/IP
  reputation dashboard, warm-up scheduling for new sending domains.
- Unsubscribe/consent management: one-click unsubscribe, granular
  preference center (topic-level opt-in/out), suppression list enforced at
  send-time across every campaign and automation, full consent audit
  history per contact.
- Campaign analytics/attribution: delivery/open/click/bounce/unsubscribe
  metrics per campaign and per step, revenue attribution linked to
  downstream orders within a configurable attribution window.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  saved segment views, mass actions (bulk-suppress, bulk-tag contacts),
  custom fields on contacts, CSV import/export with consent-source mapping
  in the import wizard, full audit history per contact and campaign,
  version history on campaign templates.

## 10. Non-Functional Requirements

Inherits the baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md).
ZodiReach-specific additions:

- Segment evaluation against a 5-million-contact tenant must complete
  within 30 seconds for a live/dynamic segment used at send-time.
- Suppression-list checks are a hard gate on the send pipeline: no send may
  be dispatched to a contact without a suppression check completing
  successfully immediately beforehand — a suppression-service outage halts
  sends rather than failing open.
- Campaign send throughput must sustain at least 1 million email sends per
  hour per tenant during peak campaign windows without degrading
  concurrent tenants' send queues.

## 11. Architecture

ZodiReach is a tenant application on [ZodiCore](../ZodiCore/SPEC.md),
consuming identity, permissions, and the shared notification fan-out
infrastructure (`zodize/core-notifications`) per
[architecture/overview.md](../../architecture/overview.md) — marketing sends
and transactional product notifications share the same underlying delivery
infrastructure but are logically separated by purpose (marketing sends are
subject to consent/suppression gating; transactional notifications are not).
ZodiReach can read contact and order data from other Zodize tenant products
(e.g. [ZodiCommerce](../ZodiCommerce/SPEC.md) customers/orders,
[ZodiBusiness](../ZodiBusiness/SPEC.md) CRM contacts) within the same tenant
boundary via a read-scoped internal data contract, rather than requiring a
data export/import connector — this is the product's core differentiation
versus a bolt-on third-party ESP. The send pipeline is a queue-driven worker
architecture separated from the segmentation/campaign-authoring API so a
large send doesn't degrade campaign-builder responsiveness.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); a
dedicated high-throughput queue worker pool for campaign sends, decoupled
from the general application queue so a large marketing send cannot starve
transactional notification delivery; email/SMS/push delivery via the
provider abstraction in §22.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Contacts | Contact Records, Custom Fields, Behavioral Event Tracking |
| Segmentation | Rule Builder, Saved/Dynamic Segments, Segment Preview |
| Campaigns | Email Builder, SMS/Push Composer, Template Library, Scheduling |
| Automation | Sequence Builder, Trigger Engine, Branching/Wait Steps |
| Testing | A/B Test Configuration, Winner Selection, Result Recording |
| Deliverability | Bounce/Complaint Monitoring, Domain Warm-Up, Reputation Dashboard |
| Consent | Preference Center, Suppression List, Consent Audit History |
| Analytics | Campaign Metrics, Attribution, Cohort/Funnel Reporting |

## 14. Core Data Model

| Entity | Key columns |
|---|---|
| `contacts` | id, tenant_id, email, phone, source, consent_status, created_at |
| `contact_events` | id, contact_id, event_type, occurred_at, metadata (jsonb) |
| `segments` | id, tenant_id, name, definition (jsonb), mode (dynamic/static) |
| `campaigns` | id, tenant_id, name, channel, status, scheduled_at, sent_at |
| `campaign_variants` | id, campaign_id, variant_label, subject, content_ref, is_winner |
| `campaign_sends` | id, campaign_id, contact_id, variant_id, status, delivered_at |
| `automations` | id, tenant_id, name, trigger_event, status |
| `automation_steps` | id, automation_id, step_order, type (send/wait/branch), config |
| `automation_enrollments` | id, automation_id, contact_id, current_step_id, enrolled_at |
| `consent_records` | id, contact_id, topic, opted_in_at, opted_out_at, source |
| `suppressions` | id, tenant_id, contact_id, reason, suppressed_at |
| `sending_domains` | id, tenant_id, domain, reputation_score, warmup_status |
| `attribution_events` | id, campaign_send_id, order_id, attributed_revenue, attributed_at |

## 15. Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/contacts` | List/search contacts with segment/consent filters |
| POST | `/api/v1/segments` | Create a segment definition |
| POST | `/api/v1/segments/{id}/preview` | Preview segment membership count/sample |
| POST | `/api/v1/campaigns` | Create a campaign |
| POST | `/api/v1/campaigns/{id}/schedule` | Schedule a campaign send |
| POST | `/api/v1/campaigns/{id}/send-now` | Trigger an immediate send |
| GET | `/api/v1/campaigns/{id}/metrics` | Delivery/open/click/attribution metrics |
| POST | `/api/v1/campaigns/{id}/ab-test` | Configure an A/B test on a campaign |
| POST | `/api/v1/automations` | Create an automation sequence |
| POST | `/api/v1/automations/{id}/activate` | Activate/pause an automation |
| POST | `/api/v1/contacts/{id}/unsubscribe` | Process an unsubscribe/preference update |
| GET | `/api/v1/contacts/{id}/consent-history` | Full consent audit trail for a contact |
| GET | `/api/v1/suppressions` | List suppressed contacts with reason |
| GET | `/api/v1/deliverability/domains` | Sending domain reputation status |
| POST | `/api/v1/deliverability/domains/{id}/pause` | Manually pause sending from a domain |
| GET | `/api/v1/reports/attribution` | Revenue attribution report by campaign/date range |
| POST | `/api/v1/webhooks/inbound-events` | Provider webhook for bounces/complaints/opens |

## 16. Events

`contact.subscribed`, `contact.unsubscribed`, `segment.recalculated`,
`campaign.scheduled`, `campaign.sent`, `campaign.bounce_threshold_breached`,
`campaign.ab_test_winner_selected`, `automation.enrolled`,
`automation.step_completed`, `automation.exited`, `consent.updated`,
`sending_domain.paused`, `attribution.order_matched`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `campaign.bounce_threshold_breached` | ✔ (Deliverability Specialist) | ✔ | ✔ | — |
| `campaign.sent` (send complete summary) | ✔ (Campaign Coordinator) | ✔ | — | — |
| `campaign.ab_test_winner_selected` | ✔ (Marketing Manager) | — | — | — |
| `sending_domain.paused` | ✔ (Deliverability Specialist, Marketing Manager) | ✔ | ✔ | — |
| `consent.updated` (data subject request fulfilled) | ✔ (Compliance Officer) | ✔ | — | — |
| `automation.exited` (error state) | ✔ (Marketing Manager) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md). Note
that ZodiReach's own operational notifications above are transactional and
distinct from the marketing sends it dispatches on the tenant's behalf.

## 18. Permissions & Roles

Extends ZodiCore's default roles
([rbac-permissions.md](../../security/rbac-permissions.md#default-system-roles))
with: `segments.manage`, `campaigns.create`, `campaigns.send`,
`automations.manage`, `consent.manage`, `suppressions.manage`,
`deliverability.manage`. `campaigns.send` is separated from
`campaigns.create` so a Campaign Coordinator can build a send but require a
Marketing Manager to actually dispatch it, per the approval chain in §19.
`consent.manage` and `suppressions.manage` are restricted to the Compliance
Officer role by default.

## 19. Workflows & Approval Chains

- **Campaign send approval**: campaigns targeting a segment above a
  configurable contact-count threshold require Marketing Manager approval
  before `campaigns.send` executes, to prevent an accidental full-list
  blast.
- **Automation activation approval**: a new automation touching contact
  consent-sensitive topics (e.g. anything outside pure transactional
  triggers) requires Compliance Officer review before activation.
- **Deliverability pause/resume**: an automatic pause on bounce-threshold
  breach can only be resumed by a Deliverability Specialist after
  explicitly acknowledging the root cause, never auto-resumed on a timer.
- **Data subject request (DSAR) fulfillment**: consent/suppression changes
  triggered by a formal data-subject request are handled through a
  dedicated Compliance Officer workflow distinct from a routine
  self-service unsubscribe, per
  [data-protection-privacy.md](../../security/data-protection-privacy.md).

## 20. Audit Logs

Every segment definition change, campaign send, consent status change,
suppression addition/removal, and deliverability pause/resume is recorded to
ZodiCore's shared audit log with actor, timestamp, and before/after state,
per [audit-logging.md](../../security/audit-logging.md). Consent history is
additionally retained as its own permanent, non-purgeable audit trail per
contact (opt-in source and timestamp, every opt-out event) to support
regulatory proof-of-consent requirements even after a contact is deleted
from the active contact list.

## 21. Reports & Analytics & Dashboards

Campaign performance (delivery/open/click/bounce/unsubscribe rate),
automation funnel/drop-off analysis per step, A/B test result history,
revenue attribution by campaign and by segment, sender reputation trend per
domain, and a compliance dashboard summarizing suppression list growth and
consent-source breakdown. Dashboard-builder and scheduled-report capability
per [dashboard-standards.md](../../standards/dashboard-standards.md).

## 22. Integrations

- **Email delivery providers**: SendGrid, Postmark, Amazon SES via a
  provider-abstraction layer supporting multi-provider failover.
- **SMS delivery providers**: Twilio, MessageBird.
- **Push notification delivery**: Firebase Cloud Messaging, Apple Push
  Notification service, via the same fan-out infrastructure ZodiCore uses
  for product push notifications.
- **Deliverability/reputation tooling**: integration with mailbox-provider
  feedback loops (Gmail Postmaster Tools-class signals, ISP complaint feeds).
- **Commerce/CRM data sources**: native read access to
  [ZodiCommerce](../ZodiCommerce/SPEC.md) order/customer data and
  [ZodiBusiness](../ZodiBusiness/SPEC.md) CRM contacts within the same
  tenant.

## 23. AI Features

- AI-assisted subject-line and copy suggestions within the campaign builder,
  always presented as an editable draft.
- Send-time optimization: predicts the best send time per contact based on
  historical engagement patterns, applied only when the tenant opts into
  send-time-optimization mode for a campaign.
- Churn-risk segment suggestion: surfaces a candidate "at-risk" segment
  based on declining engagement/purchase recency, offered as a
  one-click-create segment rather than an automatic action.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: dynamic segment recalculation sweep, automation
  trigger-evaluation tick, scheduled-campaign dispatch, bounce/complaint
  rate rollup for deliverability monitoring, sending-domain warm-up
  schedule advancement.
- CLI commands: `reach:recalculate-segment {id}`, `reach:dispatch-campaign {id}`,
  `reach:pause-domain {domain}`, `reach:export-suppression-list`.

## 25. Seed/Demo Data

`DemoSeeder` provisions a demo tenant with 10,000 seeded contacts with
varied consent states and behavioral event history, 3 saved segments (one
dynamic, one static), 2 completed campaigns with realistic
delivery/open/click metrics and one recorded A/B test result, one active
3-step automation sequence with enrolled contacts at various steps, and a
populated consent/suppression history including at least one processed
unsubscribe, per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the campaign analytics dashboard must render metrics
for a campaign sent to 1 million+ contacts within 5 seconds via
pre-aggregated rollup tables rather than querying raw send records live.

## 27. Security Requirements

Full baseline from
[security-standards.md](../../security/security-standards.md) applies.
Contact PII and consent records are tenant-isolated per
[multi-tenancy.md](../../architecture/multi-tenancy.md); suppression-list
enforcement is implemented as a non-bypassable gate in the send pipeline
(§10, §11) — no API path, including CLI-triggered sends, may dispatch to a
suppressed contact, per
[data-protection-privacy.md](../../security/data-protection-privacy.md).

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated suppression-enforcement test suite asserting no send path (API,
scheduled job, or CLI) can dispatch to a suppressed or non-consenting
contact, and an automation-branching test suite covering every
wait/branch/exit condition combination.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). The
send-worker pool can scale and deploy independently from the
campaign-authoring API so a large in-flight send is never interrupted by an
unrelated application deploy.

## 30. Acceptance Criteria

- A contact who unsubscribes is excluded from every subsequent send,
  including sends already scheduled at the time of unsubscribe, with zero
  exceptions.
- A dynamic segment correctly reflects live contact/order data at
  send-time, not a stale snapshot from segment creation.
- An A/B test correctly determines and rolls out the winning variant per
  the configured criteria, with the full result retained on the campaign
  record.
- Revenue attribution correctly links a campaign send to a downstream order
  within the configured attribution window, and does not attribute orders
  outside that window.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiReach additionally requires sign-off that the suppression-enforcement
test suite (§28) passes with 100% coverage of send-initiating code paths
before go-live, given the regulatory exposure of a suppression-bypass
defect.

## 32. Future Roadmap

- In-app and web push message orchestration alongside email/SMS/push as a
  unified "message" abstraction.
- Predictive lifetime-value scoring feeding directly into segment
  eligibility.
- Native SMS two-way conversation threading for customer replies.

## 33. Known Risks

- Consent/suppression enforcement is the module's highest-risk surface from
  a regulatory standpoint (CAN-SPAM, GDPR-equivalent exposure) — mitigated
  by the non-bypassable send-pipeline gate (§11, §27) and dedicated test
  suite (§28), but this remains the area warranting the strictest change
  review of any ZodiReach code path.
- Deliverability is partly outside Zodize's control (mailbox-provider
  reputation algorithms change without notice) — mitigated by the
  automatic pause-on-threshold behavior (§7, §19), but sustained
  deliverability requires ongoing sender-reputation operational discipline
  beyond what software alone guarantees.

## 34. Future Improvements

- Multi-touch attribution modeling beyond last-touch within the attribution
  window.
- Self-service sending-domain DKIM/SPF/DMARC setup wizard with automated
  validation.

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram covering multi-step automation branching state and attribution
windowing tables, the complete endpoint catalog (template management,
inbound webhook event types), and a dedicated
`DATA_MODEL.md`/`API_REFERENCE.md` pair matching
[ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.
