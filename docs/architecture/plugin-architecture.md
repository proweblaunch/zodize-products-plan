# Plugin Architecture

> The extension system every Zodize product exposes, powered by ZodiCore's
> plugin runtime. Builds on [`overview.md`](./overview.md) and
> [`multi-tenancy.md`](./multi-tenancy.md); feeds
> [`marketplace-architecture.md`](./marketplace-architecture.md), which
> governs how plugins get distributed.

## Plugin manifest format

Every plugin ships a `zodize-plugin.json` manifest at its package root:

```json
{
  "slug": "acme-shipping-rates",
  "name": "Acme Shipping Rates",
  "version": "1.4.0",
  "vendor": "Acme Logistics Inc.",
  "compatible_products": ["ZodiCommerce", "ZodiBusiness"],
  "min_core_version": "3.2.0",
  "max_core_version": "4.x",
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

- `slug` MUST be globally unique across the marketplace, lowercase,
  hyphenated, and immutable once published — it is the plugin's permanent
  identifier.
- `version` follows semantic versioning; `min_core_version`/
  `max_core_version` declare the ZodiCore compatibility range using the same
  scheme, checked at install time.
- `permissions` is the exhaustive list of `resource.action` permissions
  (per [`../security/rbac-permissions.md`](../security/rbac-permissions.md#permission-naming-convention))
  the plugin requests; this list is what the tenant Admin reviews and
  approves at install time.

## Hook and event points

Plugins subscribe to the same domain events the host product's own modules
emit, per the event-driven pattern in
[`caching-queues-events.md`](./caching-queues-events.md#event-driven-architecture) —
there is no separate "plugin API" event bus, so plugin authors and Zodize's
own module authors work against one consistent contract.

- Every module that wants to be extensible registers its domain events in a
  product-level `docs/products/<product>/plugin-hooks.md` catalog (e.g.
  `order.created`, `invoice.paid`, `patient.admitted`), which is the
  authoritative list a plugin manifest's `hooks` map may reference.
  Un-cataloged events are not a stable extension point and plugins MUST NOT
  rely on them.
- Hook listeners run as queued listeners by default (never synchronously in
  the request that fired the event), so a slow or failing plugin cannot
  degrade core product latency — see the queue standard in
  [`caching-queues-events.md`](./caching-queues-events.md#queue-standard).
  A plugin MAY declare a hook as `synchronous: true` in its manifest only
  for hooks explicitly marked "sync-eligible" in the product's hook catalog
  (e.g. a pricing-calculation hook that must return a value before checkout
  proceeds), and such hooks MUST complete within a 500ms budget, enforced by
  a timeout that disables the plugin on repeated violation.
- Plugins MAY also register UI extension points (a settings panel section,
  a dashboard widget, a menu item) via a `ui_slots` manifest section; slots
  are enumerated per product in the same `plugin-hooks.md` catalog.

## Sandboxing and permission model

- A plugin runs inside the host product's PHP process (not a separate
  container per plugin, for latency and cost reasons at Zodize's scale), so
  sandboxing is enforced at the framework layer, not the OS layer:
  - A plugin's Eloquent access is restricted to its own plugin-owned tables
    (prefixed `plugin_{slug}_...`) plus whatever host-product models its
    declared `permissions` grant read/write to, mediated through the same
    Policy classes every first-party module uses — a plugin cannot query a
    model it has no permission for even if the class is technically
    reachable in PHP.
  - A plugin's routes are automatically prefixed and namespaced
    (`/plugins/{slug}/...`) and run through the full authentication,
    tenant-scoping, and authorization middleware stack; a plugin cannot
    register a route that bypasses tenant isolation.
  - A plugin's outbound HTTP calls (webhooks, third-party API calls) are
    logged and subject to the same SSRF protections described in
    [`../security/security-standards.md`](../security/security-standards.md#owasp-top-10-mapping).
- At install time, the tenant Admin sees a permission-consent screen listing
  every `resource.action` and every hook the plugin requests, in plain
  language (e.g. "This plugin can view and update your orders"). Installing
  without reviewing this screen is not possible — the install button is
  disabled until the list has been rendered.
- A plugin MUST NOT be granted more effective permission than the installing
  Admin's own role holds, following the same no-privilege-escalation rule as
  the custom role builder in
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md#permission-inheritance-and-override-rules).

## Plugin lifecycle

| State | Trigger | Behavior |
|---|---|---|
| Installed | Admin installs from the marketplace or uploads a signed package for a private/dev plugin. | Migrations run in an isolated transaction; plugin is inactive until explicitly enabled. |
| Enabled | Admin toggles the plugin on after reviewing permissions. | Hooks register, routes activate, UI slots render. |
| Disabled | Admin toggles off, or the plugin is auto-disabled after repeated hook failures/timeouts. | Hooks stop firing, routes return `404`, UI slots stop rendering. Data is retained. |
| Uninstalled | Admin explicitly uninstalls. | Plugin's own migrations are rolled back (dropping `plugin_{slug}_...` tables); any host-product data the plugin created via granted permissions is NOT deleted, since it belongs to the tenant, not the plugin. |

Every lifecycle transition is audit-logged per
[`../security/audit-logging.md`](../security/audit-logging.md) (`plugins.install`,
`plugins.enable`, `plugins.disable`, `plugins.uninstall`), including which
Admin performed it.

## Versioning and compatibility

- Plugin updates follow semver: a patch/minor version update is
  auto-installable by the tenant without re-consent; a major version bump
  that changes the `permissions` list MUST re-trigger the permission-consent
  screen before the update applies.
- Every plugin enable/update check validates compatibility against
  the host product's installed `zodize/core` version, per the manifest's
  `min_core_version`/`max_core_version`, and MUST refuse to enable a plugin
  outside that range rather than attempting a best-effort load.
- Deprecating a hook requires a minimum 2 minor-version deprecation window
  during which the hook still fires alongside a deprecation warning
  surfaced to the plugin's vendor (via the marketplace developer console),
  before removal in the next major version.

## Migrations, routes, and views

- A plugin's migrations live under its own `migrations_path` and run inside
  a plugin-specific migration table (`plugin_{slug}_migrations`) so plugin
  and host-product migration history never collide, and so uninstalling one
  plugin cannot affect another's migration state.
- Plugin migrations MUST only create/alter tables prefixed
  `plugin_{slug}_` — a plugin migration that attempts to alter a
  host-product core table is rejected at install time by a manifest-time
  static check.
- Plugin routes are auto-registered from `routes_path` under the
  `/plugins/{slug}/` prefix with the full middleware stack (tenant
  resolution, auth, throttle) applied automatically — a plugin cannot opt
  out of tenant scoping or authentication middleware.
- Plugin views/Vue components are compiled into an isolated JS chunk
  (`assets_entry`) lazy-loaded only when a tenant has the plugin enabled, so
  disabled plugins add zero bytes to the host product's base bundle — see
  the frontend budget in
  [`../quality/performance-standards.md`](../quality/performance-standards.md).

## Related standards

- [`marketplace-architecture.md`](./marketplace-architecture.md)
- [`caching-queues-events.md`](./caching-queues-events.md)
- [`../security/rbac-permissions.md`](../security/rbac-permissions.md)
- [`../security/security-standards.md`](../security/security-standards.md)
