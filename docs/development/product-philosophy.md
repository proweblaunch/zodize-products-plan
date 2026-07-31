# Product Philosophy

This document defines how Zodize decides *what* to build, distinct from
[engineering-principles.md](./engineering-principles.md), which defines *how*.

## We build vertical enterprise software, not horizontal toys

Every Zodize product targets a specific industry (banking, healthcare, real
estate, hospitality, etc.) with deep domain modeling, not a generic
"business management" tool with industry skins. A feature belongs in a
product's core if the target industry's practitioners would consider its
absence a dealbreaker, not merely a convenience.

## Enterprise-grade from the first commit

A product is not allowed to ship a "demo" version that later gets
"hardened" for enterprise customers. RBAC, audit logs, SSO-readiness, API
access, and data export exist from the first internal build. See
[definition-of-production-ready.md](../quality/definition-of-production-ready.md).
This is cheaper than retrofitting and it means every demo is a real product a
prospect could deploy tomorrow — see the [Demo Standard](../../README.md)
principle: every button performs a real action, every chart shows real
(seeded) data, nothing is a static mockup.

## Discover the second layer of requirements

Customers ask for the feature they know to ask for. Enterprise buyers later
discover they also needed approval workflows, a rule/automation engine,
saved filters, global search, custom fields, an import/export wizard, mass
actions, a command palette, API tokens, webhooks, version history, and
undo/soft-delete. Every product specification in `docs/products/` must
account for this "second layer" explicitly — see the Second-Layer Feature
Catalog below — rather than leaving it to be discovered by an unhappy
customer.

### Second-layer feature catalog (baseline for every product)

| Category | Required baseline capability |
|---|---|
| Workflow | Approval chains, configurable workflow builder for key entities |
| Automation | Rule engine ("when X, do Y"), scheduled/triggered automations |
| Data access | Saved filters/views, global search, advanced/faceted filtering |
| Extensibility | Custom fields on core entities, plugin hooks, API tokens, webhooks |
| Trust | Full audit history, activity timeline per record, version history |
| Safety | Soft delete + restore, undo for recent destructive actions |
| Bulk operations | Mass actions on list views, import wizard, export wizard |
| Power users | Command palette, keyboard shortcuts |
| Intelligence | Report builder, dashboard builder, scheduled reports, forecasting where domain-relevant |
| Operations | Health monitoring/system diagnostics for admins, license/seat management |
| Platform | White-labeling, tenant/organization management, session management, rate-limited API, developer portal |

A product spec that omits a row from this table without an explicit,
justified exception is incomplete.

## Competitive posture

Every product spec includes a competitor analysis (see
[docs/products/ZodiCore/SPEC.md](../products/ZodiCore/SPEC.md) for the
canonical structure). Zodize does not aim to be the cheapest option — it
aims to be the option an enterprise buyer trusts to run their operation for a
decade, competing directly with category leaders (e.g. Salesforce, SAP,
Oracle NetSuite, Toast, Epic/Cerner-class systems, depending on vertical).

## Multi-everything by default

Every product must support multi-company, multi-branch, multi-currency, and
multi-language operation from its data model up, per
[localization-i18n.md](../standards/localization-i18n.md) and
[multi-tenancy.md](../architecture/multi-tenancy.md), because the enterprise
customers Zodize targets operate across borders and subsidiaries by default,
not as an edge case.

## Definition of a good Zodize product decision

A feature, workflow, or scope decision is good if it would survive being
explained to (a) a Fortune 500 buyer's procurement team, (b) a security
auditor, and (c) the engineer who has to maintain it in five years. If a
decision only survives explaining to a demo audience, it is not yet a Zodize
decision.
