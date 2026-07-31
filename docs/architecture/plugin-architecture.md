# Plugin Architecture

> The extension system a Zodize product MAY expose to let third-party
> developers extend that one product's installed instances. This is a
> **per-product** capability — there is no shared `zodize/core` package and
> no cross-product plugin runtime; see
> [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md).
> Builds on [`overview.md`](./overview.md); feeds
> [`marketplace-architecture.md`](./marketplace-architecture.md), which
> governs how a product's plugins get distributed to buyers of that product.

Not every product needs this. A product's own
[`SPEC.md`](../products/) states explicitly whether it ships a plugin
system; treat this document as the standard to follow **if** it does, not a
mandatory requirement for every product.

## Plugin manifest format

Every plugin ships a `zodize-plugin.json` manifest at its package root:

```json
{
  "slug": "acme-shipping-rates",
  "name": "Acme Shipping Rates",
  "version": "1.4.0",
  "vendor": "Acme Logistics Inc.",
  "min_product_version": "3.2.0",
  "max_product_version": "4.x",
  "permissions": [
    "orders.view",
    "orders.update",
    "webhooks.register"
  ],
  "hooks": {
    "order.created": "App\\Plugins\\AcmeShippingRates\\Listeners\\OnOrderCreated",
    "order.shipped": "App\\Plugins\\AcmeShippingRates\\Listeners\\OnOrderShipped"
  },
  "settings_schema": "resources/settings-schema.json",
  "migrations_path": "database/migrations",
  "routes_path": "routes/plugin.php",
  "assets_entry": "resources/js/entry.js"
}
```

- `slug` MUST be globally unique within the one product's marketplace,
  lowercase, hyphenated, and immutable once published — it is the plugin's
  permanent identifier.
- `version` follows semantic versioning; `min_product_version`/
  `max_product_version` declare the compatibility range against **that one
  product's own version number** (per
  [`../development/versioning-releases.md`](../development/versioning-releases.md)),
  checked at install time — a plugin is written for one product (e.g.
  ZodiCommerce) and is not portable across products, since each product is
  an independent codebase with its own domain modules.
- `permissions` is the exhaustive list of `resource.action` permissions
  (per [`../security/rbac-permissions.md`](../security/rbac-permissions.md#permission-naming-convention))
  the plugin requests; this list is what the buyer's admin reviews and
  approves at install time.

## Hook and event points

Plugins subscribe to the same domain events the product's own modules emit,
per the event-driven pattern in
[`caching-queues-events.md`](./caching-queues-events.md#event-driven-architecture) —
there is no separate "plugin API" event bus, so plugin authors and the
product's own module authors work against one consistent contract.

- Every module that wants to be extensible registers its domain events in a
  product-level `docs/products/<product>/plugin-hooks.md` catalog (e.g.
  `order.created`, `invoice.paid`, `patient.admitted`), which is the
  authoritative list a plugin manifest's `hooks` map may reference.
  Un-cataloged events are not a stable extension point and plugins MUST NOT
  rely on them.
- Hook listeners run as queued listeners by default (never synchronously in
  the request that fired the event), so a slow or failing plugin cannot
  degrade the product's core latency — see the queue standard in
  [`caching-queues-events.md`](./caching-queues-events.md#queue-standard),
  including its documented fallback for buyers without a persistent queue
  worker. A plugin MAY declare a hook as `synchronous: true` in its manifest
  only for hooks explicitly marked "sync-eligible" in the product's hook
  catalog, and such hooks MUST complete within a 500ms budget, enforced by a
  timeout that disables the plugin on repeated violation.
- Plugins MAY also register UI extension points (a settings panel section,
  a dashboard widget, a menu item) via a `ui_slots` manifest section; slots
  are enumerated per product in the same `plugin-hooks.md` catalog.

## Sandboxing and permission model

- A plugin runs inside the product's own PHP process (not a separate
  container per plugin), so sandboxing is enforced at the framework layer,
  not the OS layer:
  - A plugin's Eloquent access is restricted to its own plugin-owned tables
    (prefixed `plugin_{slug}_...`) plus whatever product models its
    declared `permissions` grant read/write to, mediated through the same
    Policy classes every first-party module uses.
  - A plugin's routes are automatically prefixed and namespaced
    (`/plugins/{slug}/...`) and run through the full authentication and
    authorization middleware stack inherited from the base codebase's
    guard/RBAC system (see
    [`../templates/admin-template.md`](../templates/admin-template.md)) —
    a plugin cannot register a route that bypasses that middleware.
  - A plugin's outbound HTTP calls (webhooks, third-party API calls) are
    logged and subject to the same SSRF protections described in
    [`../security/security-standards.md`](../security/security-standards.md#owasp-top-10-mapping).
- At install time, the buyer's admin sees a permission-consent screen
  listing every `resource.action` and every hook the plugin requests, in
  plain language (e.g. "This plugin can view and update your orders").
  Installing without reviewing this screen is not possible — the install
  button is disabled until the list has been rendered.
- A plugin MUST NOT be granted more effective permission than the
  installing admin's own role holds, following the same
  no-privilege-escalation rule as the role system in
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md).

## Plugin lifecycle

| State | Trigger | Behavior |
|---|---|---|
| Installed | Admin installs from the product's marketplace tab or uploads a signed package for a private/dev plugin. | Migrations run in an isolated transaction; plugin is inactive until explicitly enabled. |
| Enabled | Admin toggles the plugin on after reviewing permissions. | Hooks register, routes activate, UI slots render. |
| Disabled | Admin toggles off, or the plugin is auto-disabled after repeated hook failures/timeouts. | Hooks stop firing, routes return `404`, UI slots stop rendering. Data is retained. |
| Uninstalled | Admin explicitly uninstalls. | Plugin's own migrations are rolled back (dropping `plugin_{slug}_...` tables); any product data the plugin created via granted permissions is NOT deleted, since it belongs to the buyer's business, not the plugin. |

Every lifecycle transition is audit-logged per
[`../security/audit-logging.md`](../security/audit-logging.md)
(`plugins.install`, `plugins.enable`, `plugins.disable`,
`plugins.uninstall`), including which admin performed it.

## Versioning and compatibility

- Plugin updates follow semver: a patch/minor version update is
  auto-installable by the buyer's admin without re-consent; a major version
  bump that changes the `permissions` list MUST re-trigger the
  permission-consent screen before the update applies.
- Every plugin enable/update check validates compatibility against the
  installed product's own version, per the manifest's
  `min_product_version`/`max_product_version`, and MUST refuse to enable a
  plugin outside that range rather than attempting a best-effort load.
- Deprecating a hook requires a minimum 2 minor-version deprecation window
  during which the hook still fires alongside a deprecation warning
  surfaced to the plugin's vendor (via the marketplace developer console),
  before removal in the next major version.

## Migrations, routes, and views

- A plugin's migrations live under its own `migrations_path` and run inside
  a plugin-specific migration table (`plugin_{slug}_migrations`) so plugin
  and product migration history never collide, and so uninstalling one
  plugin cannot affect another's migration state.
- Plugin migrations MUST only create/alter tables prefixed
  `plugin_{slug}_` — a plugin migration that attempts to alter one of the
  product's own core tables is rejected at install time by a manifest-time
  static check.
- Plugin routes are auto-registered from `routes_path` under the
  `/plugins/{slug}/` prefix with the full middleware stack (auth, throttle)
  applied automatically — a plugin cannot opt out of authentication
  middleware.
- Plugin views/Vue or Blade components are compiled into an isolated JS
  chunk (`assets_entry`) lazy-loaded only when the deployment has the
  plugin enabled, so disabled plugins add zero bytes to the product's base
  bundle — see the frontend budget in
  [`../quality/performance-standards.md`](../quality/performance-standards.md).

## Related standards

- [`marketplace-architecture.md`](./marketplace-architecture.md)
- [`caching-queues-events.md`](./caching-queues-events.md)
- [`../security/rbac-permissions.md`](../security/rbac-permissions.md)
- [`../security/security-standards.md`](../security/security-standards.md)
