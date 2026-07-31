# Engineering Principles

These principles govern every technical decision made across every Zodize
product. When a standard elsewhere in this handbook seems to conflict with
another, resolve the conflict by returning to these principles.

## 1. One platform, many products

Every Zodize product is a tenant application on top of [ZodiCore](../products/ZodiCore/SPEC.md).
Identity, billing, tenancy, notifications, permissions, and the plugin runtime
are built once, correctly, and consumed everywhere. A product team does not
get to reimplement authentication because "this product is different." If a
product's requirements genuinely cannot be met by the platform, that is a
platform gap to fix, not a license to fork.

## 2. Boring technology, exceptional execution

Zodize does not chase frameworks. The stack is Laravel (PHP) on the backend
and Vue on the frontend, described fully in
[coding-standards-php-laravel.md](./coding-standards-php-laravel.md) and
[coding-standards-vue.md](./coding-standards-vue.md). Differentiation comes
from the quality of implementation, the design system, and the depth of
domain modeling — not from tooling novelty. New infrastructure requires an
ADR (see [`docs/decisions/`](../decisions/adr-template.md)) justifying why the
boring option was insufficient.

## 3. Modular monolith, not microservices-by-default

Each product is built as a modular monolith: strong internal module
boundaries (see [module-template.md](../templates/module-template.md)),
single deployable unit, single database with tenant scoping. Services are
only extracted when a concrete scaling or isolation requirement demands it,
documented via ADR. This keeps operational complexity proportional to actual
need. See [architecture/overview.md](../architecture/overview.md).

## 4. Every standard is testable and enforced in CI

A standard that isn't checked by a machine will erode. Wherever a standard in
this handbook can be expressed as a lint rule, a static analysis rule, or an
automated test, it must be. See [ci-cd-standards.md](../quality/ci-cd-standards.md).

## 5. Security and audit are not features, they are properties

Authentication, authorization, and audit logging are not modules a product
can opt out of — they are properties of the platform every product inherits.
See [docs/security/](../security/security-standards.md).

## 6. Design once, theme many

The visual language is defined once in [docs/design-system/](../design-system/brand-standards.md)
and every product expresses its own identity through a bounded set of
per-product tokens (accent color, icon mark, product name) layered on top of
the shared system — never through a parallel design language.

## 7. Specs before code

No product's implementation begins until its specification passes the
"Spec Complete" gate in
[production-readiness-checklist.md](../checklists/production-readiness-checklist.md).
A specification is a contract with future engineers (and with AI coding
agents) that implementation will not need to guess.

## 8. Reversibility over speed when the two conflict

Prefer soft delete over hard delete, feature flags over big-bang releases,
additive migrations over destructive ones, and staged rollouts over global
ones. Speed is a feature (see [performance-standards.md](../quality/performance-standards.md))
but it never trumps the ability to undo a mistake.

## 9. Documentation is a deliverable, not an afterthought

A feature is not done when the code merges; it is done when the code merges,
the tests pass, and the relevant documentation (handbook standard, API
reference, or user-facing docs) reflects reality. See
[documentation-standards.md](./documentation-standards.md) and
[definition-of-done.md](../quality/definition-of-done.md).

## 10. Build for the enterprise buyer's second question

The first question an enterprise buyer asks is "does it do X?" The second is
"can I trust it, audit it, extend it, and integrate it?" Every product must
answer the second question by default: audit trails, RBAC, API access,
webhooks, SSO, data export. These are not premium add-ons bolted on later —
they are part of the base architecture from module one.
