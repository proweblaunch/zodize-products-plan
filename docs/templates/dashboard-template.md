# Dashboard Template

This document specifies the default authenticated home for every Zodize
product. The interaction and layout rules it implements are defined in
[`../standards/dashboard-standards.md`](../standards/dashboard-standards.md);
this document is the concrete scaffold. A product customizes its default
widget set beyond the mandatory minimum and its personalization options; it
does not customize the layout regions or remove the getting-started widget
for a new deployment.

## Directory structure

```
resources/js/
  pages/
    Dashboard.vue
  components/Dashboard/
    Layout.vue
    WidgetGrid.vue
    Widgets/
      GettingStartedChecklist.vue
      RecentActivity.vue
      QuickActions.vue
      <ProductSpecificWidget>.vue
    WidgetSettingsPanel.vue
app/
  Services/Dashboard/
    WidgetRegistry.php
    WidgetLayoutService.php
  Models/
    DashboardLayout.php
database/migrations/
  xxxx_create_dashboard_layouts_table.php
```

## Layout regions

Every dashboard MUST have four fixed regions, in this arrangement:

1. **Header region** — page title ("Welcome back, {name}"), date range or
   context selector if applicable, primary quick action button.
2. **Summary region** — a row of at-a-glance metric tiles (0 to 6 tiles);
   MUST collapse gracefully to a single column on mobile per
   [`../design-system/`](../design-system) responsive breakpoints.
3. **Widget grid region** — the reorderable, resizable widget area described
   below; this is the primary customizable surface.
4. **Footer/utility region** — links to documentation, support, and system
   status; consistent across every product.

A product MUST NOT introduce a fifth top-level region or remove one of the
four; a region MAY render empty (e.g. no summary tiles configured) but MUST
still reserve its layout slot so that adding content later is not a layout
change.

## Default widget set for a new deployment

A brand-new deployment's dashboard MUST NOT render empty. As part of the
buyer's initial setup (first admin login after install), `WidgetLayoutService`
MUST seed the widget grid with:

1. **Getting Started checklist widget** (`GettingStartedChecklist.vue`) —
   MANDATORY, always the first widget for any deployment that has not
   completed onboarding. It MUST show a product-specific checklist (minimum 4
   items, e.g. "Configure your first payment gateway", "Add your team",
   "Complete your business profile", "Explore the {core feature}") with a
   visible progress indicator. It MUST auto-dismiss (moving to a collapsed
   "onboarding complete" state, not deletion) once every item is checked, and
   MUST remain manually re-openable from the footer/utility region afterward.
2. **Recent Activity widget** (`RecentActivity.vue`) — a feed of the
   business's own recent actions, sourced from the audit log per
   [`../security/audit-logging.md`](../security/audit-logging.md), filtered
   to activity-relevant (not security-relevant) entries.
3. **Quick Actions widget** (`QuickActions.vue`) — up to 6 shortcuts into
   the product's core creation flows (e.g. "New Invoice", "Add Contact").

A product MAY add product-specific widgets to this default set but MUST NOT
launch with a dashboard whose default state, before user customization, is
visually empty below the header region.

## Personalization requirements

- Widgets MUST be reorderable via drag-and-drop and resizable in at least
  two size classes (compact, expanded), persisted per-user in
  `dashboard_layouts`.
- A user MUST be able to hide any non-mandatory widget and restore it later
  via `WidgetSettingsPanel.vue`'s "Add widget" gallery — hiding MUST NOT
  delete the widget's underlying data.
- The Getting Started checklist widget MUST NOT be permanently hideable
  until its completion condition is met; a user MAY collapse it but not
  remove it while onboarding is incomplete.
- Layout state MUST degrade gracefully: a widget referencing since-removed
  data (e.g. a deleted integration) MUST render its own empty/error state,
  never break the surrounding grid.

## Reusable scaffold vs. what a product customizes

This dashboard scaffold ships inside every product's own codebase — it is
not fetched from another Zodize product at runtime, per
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#no-shared-platform-service).
It provides: `Layout.vue`, `WidgetGrid.vue`, `WidgetRegistry.php` (the
extension point new widgets register against), `WidgetLayoutService.php`,
the `dashboard_layouts` table, and the `GettingStartedChecklist.vue` and
`RecentActivity.vue` widgets.

A product customizes: the getting-started checklist item list (via
`WidgetRegistry` configuration, not by forking the component), its
product-specific widgets, and default summary-tile metrics. A product MUST
NOT build a bespoke dashboard layout system — extend `WidgetRegistry`.
