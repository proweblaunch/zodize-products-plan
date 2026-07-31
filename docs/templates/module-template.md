# Module Template

Every unit of product functionality at Zodize is built as a **module** —
a self-contained Laravel domain package following this exact structure. This
is the single most-used template in the handbook: dashboard widgets, admin
tools, and every product feature are built as modules. A product customizes
the module's internal business logic; it does not customize the top-level
folder structure, which is fixed so that any engineer or AI agent can
navigate any module in any product identically.

## Directory structure

```
Modules/{ModuleName}/
  Models/
    {Entity}.php
  Policies/
    {Entity}Policy.php
  Http/
    Controllers/
      {Entity}Controller.php
    Requests/
      Store{Entity}Request.php
      Update{Entity}Request.php
  Services/
    {Entity}Service.php
  Repositories/
    {Entity}Repository.php
    Contracts/
      {Entity}RepositoryInterface.php
  Events/
    {Entity}Created.php
    {Entity}Updated.php
    {Entity}Deleted.php
  Listeners/
    Log{Entity}AuditEvent.php
  Notifications/
    {Entity}{Event}Notification.php
  database/
    migrations/
      xxxx_xx_xx_create_{entities}_table.php
    factories/
      {Entity}Factory.php
    seeders/
      {Entity}Seeder.php
  routes/
    api.php
    web.php
  resources/js/
    pages/
      {Entity}/Index.vue
      {Entity}/Show.vue
      {Entity}/Form.vue
    components/
      {Entity}Card.vue
      {Entity}Table.vue
  tests/
    Unit/
      {Entity}ServiceTest.php
    Feature/
      {Entity}ControllerTest.php
    Browser/
      {Entity}FlowTest.php
  routes.permissions.php
  Module.php
```

## Layer responsibilities

Each layer has exactly one responsibility, and business logic MUST live in
exactly one of them — never duplicated across layers:

- **Models** — Eloquent models, relationships, casts, and scopes only. No
  business logic beyond query scopes.
- **Policies** — authorization decisions only, following
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md). A
  controller action MUST NOT contain an inline authorization check that
  bypasses a Policy.
- **Http/Controllers** — thin: validate via a Request class, call a Service
  method, return a response using the standard envelope from
  [api-template.md](./api-template.md). Controllers MUST NOT contain
  business logic or direct Eloquent queries beyond simple `find`/`route
  model binding`.
- **Http/Requests** — validation rules and authorization gate for a single
  action, per [`../development/api-standards.md`](../development/api-standards.md).
- **Services** — business logic and orchestration. This is where workflows
  spanning multiple models/repositories live. Services MUST use constructor
  injection per [`../development/coding-standards-php-laravel.md`](../development/coding-standards-php-laravel.md).
- **Repositories** — data access abstraction implementing a
  `{Entity}RepositoryInterface`. Required whenever a query is non-trivial or
  reused across more than one Service; simple CRUD-only modules MAY skip the
  Repository layer and query the Model directly from the Service, but if a
  Repository exists, all queries for that entity go through it.
- **Events/Listeners** — domain events fired on create/update/delete.
  `Log{Entity}AuditEvent` MUST be registered on every entity's lifecycle
  events per [`../security/audit-logging.md`](../security/audit-logging.md).
- **Notifications** — outbound notifications, dispatched through ZodiCore's
  notification system, never sent directly via a mail/SMS facade from a
  Service.
- **database/migrations, factories, seeders** — schema and test/demo data
  for this module's own tables, layered on top of
  [database-template.md](./database-template.md).
- **routes/api.php, web.php** — this module's routes only, auto-discovered
  and mounted under the product's route prefix; a module MUST NOT register
  routes outside its own `routes/` files.
- **resources/js** — Vue components colocated with the module they belong
  to. Components shared across more than one module MUST be promoted to the
  shared frontend component library, not duplicated.
- **tests** — see [testing-template.md](./testing-template.md) for the
  mandatory coverage this directory must contain.

## Module registration

`Module.php` is the module's manifest: it declares the module's name,
version, dependencies on other modules, and calls out to
`routes.permissions.php` to auto-register the module's permission set with
RBAC on install, per [permission-template.md](./permission-template.md). A
module MUST be independently enableable/disableable per tenant through this
manifest, supporting the plugin architecture in
[`../architecture/`](../architecture).

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the module auto-discovery/loading mechanism, the base
Repository interface and abstract implementation, the audit-log listener
base class, and the notification dispatch pipeline every module's
Notifications layer calls into.

A product customizes: every file under Models, Policies, Http, Services,
Repositories, Events, Listeners, Notifications, database, routes, and
resources/js for each module it builds. A product MUST NOT deviate from this
directory structure — a module missing a required directory (e.g. no
`tests/Feature/`) fails the module-level review gate in
[`../checklists/pr-checklist.md`](../checklists/pr-checklist.md).
