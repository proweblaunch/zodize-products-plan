# Production Readiness Checklist

This is the go/no-go checklist a Zodize product MUST pass before General
Availability. It is organized into a **Spec Complete** gate, which MUST pass
before implementation begins per [`../../ROADMAP.md`](../../ROADMAP.md)
Phase 4, and the full **GA** gate, which MUST pass before the product is
sold to non-beta buyers. Every item MUST be independently verifiable —
"looks fine" is not a passing state. An unchecked item blocks release; there
is no partial credit.

## Spec Complete gate (required before implementation begins)

Per [`../../ROADMAP.md`](../../ROADMAP.md) Phase 4, no product's
implementation begins until its `docs/products/<product>/SPEC.md` passes
every item below. This gate is satisfiable at **Foundation** depth per
[`../../PRODUCT_CATALOG.md`](../../PRODUCT_CATALOG.md)'s spec status
definitions — it does **not** require ZodiCore-level Reference/Deep depth.
Full ER diagrams and an exhaustive endpoint catalog are explicitly **not**
required here; see the GA gate's "Deep artifacts" item below for when those
are required, and note they are written just-in-time per module as that
module is actually implemented, not produced as a blanket prerequisite
before any code is written.

- [ ] Vision and target market documented.
- [ ] Primary personas documented, each with goals and pain points.
- [ ] Competitive positioning documented.
- [ ] Core data model documented (the product's key entities and their
      relationships, at the depth every current product spec's "Core Data
      Model" section already provides), extending
      [`../templates/database-template.md`](../templates/database-template.md) —
      a full ER diagram is not required at this gate.
- [ ] Every module the product ships is enumerated, each mapped to
      [`../templates/module-template.md`](../templates/module-template.md).
- [ ] Core user workflows documented end-to-end (not just feature lists).
- [ ] Every third-party integration named, with what data crosses the
      boundary in each direction.
- [ ] Acceptance criteria defined for every core workflow.
- [ ] Which templates in [`../templates/`](../templates) the product inherits
      as-is, and which require a documented delta, is explicit.
- [ ] Open questions, if any, are captured under an explicit
      `## Open Questions` section, not left implicit.

## Security

- [ ] Threat model reviewed per [`../security/`](../security).
- [ ] Authentication flows implemented exactly per
      [`../templates/authentication-template.md`](../templates/authentication-template.md).
- [ ] RBAC/permission set fully registered per
      [`../templates/permission-template.md`](../templates/permission-template.md).
- [ ] Full [`../checklists/security-checklist.md`](./security-checklist.md) passed.
- [ ] Penetration test or equivalent security review completed for the
      product's public-facing surface.
- [ ] No secrets present in source control, CI logs, or client-side bundles.

## Performance

- [ ] Performance budget from [`../quality/performance-standards.md`](../quality/performance-standards.md)
      met on the marketing site, dashboard, and top three highest-traffic
      product pages.
- [ ] Database queries on high-traffic endpoints reviewed for N+1 queries
      and missing indexes.
- [ ] Load testing completed at the user/record volume defined in the
      product's own SPEC.md, against the shared/VPS hosting reference target
      in [`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer).
- [ ] Background job queues sized and monitored per
      [`../templates/deployment-template.md`](../templates/deployment-template.md).

## Data

- [ ] Every table follows [`../templates/database-template.md`](../templates/database-template.md)
      base schema plus product-specific extensions.
- [ ] Backup and restore procedure tested end-to-end, not just configured.
- [ ] Data retention policy documented and enforced for every table
      containing personal data.
- [ ] The schema contains no `tenant_id` column, tenant global query scope,
      or other multi-tenant construct, per
      [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md).
- [ ] If the product supports multi-company/multi-branch scoping per
      [`../standards/localization-i18n.md`](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping),
      cross-branch access without the requisite permission is impossible and
      covered by an automated test.
- [ ] Migration rollback strategy verified per
      [`../templates/deployment-template.md`](../templates/deployment-template.md).

## Accessibility

- [ ] Full [`../checklists/accessibility-checklist.md`](./accessibility-checklist.md) passed.
- [ ] Screen reader walkthrough completed for the top three critical flows.

## Documentation

- [ ] Documentation site live per
      [`../templates/documentation-template.md`](../templates/documentation-template.md),
      covering Getting Started, Guides, API Reference, Changelog, and FAQ.
- [ ] Marketing site live per
      [`../templates/marketing-website-template.md`](../templates/marketing-website-template.md).
- [ ] `docs/products/<product>/SPEC.md` is current with the shipped product
      (no drift between spec and implementation).
- [ ] **Deep artifacts complete (GA gate only, not required to start
      implementation)**: a full ER diagram covering every implemented table
      and relationship, an exhaustive endpoint catalog matching the shipped
      OpenAPI spec, and a full report/chart catalog for every implemented
      dashboard/report. These are written module-by-module as implementation
      proceeds — per [`../../ROADMAP.md`](../../ROADMAP.md) Phase 4 — and
      this item verifies they are complete and drift-free by GA, not that
      they existed before the first line of code was written.

## Support

- [ ] Admin/back-office tooling live per
      [`../templates/admin-template.md`](../templates/admin-template.md),
      including the inherited admin sections, feature flags, and system health.
- [ ] Support team trained on the product with access to admin support tools.
- [ ] Status page live and linked from the marketing site footer.
- [ ] Escalation path documented for production incidents.

## Compliance

- [ ] Privacy policy and terms of service published and legally reviewed.
- [ ] Data processing agreement available for enterprise customers, if
      applicable to the product's market.
- [ ] Any regulatory requirement specific to the product's vertical (e.g.
      HIPAA-equivalent for ZodiMed, banking regulation for ZodiBank) is
      enumerated in the product's own SPEC.md and independently verified,
      not assumed covered by the general handbook.
- [ ] Audit logging coverage verified against
      [`../security/audit-logging.md`](../security/audit-logging.md) for
      every sensitive action in the product.

## Rollout

- [ ] Release process followed per
      [`../templates/release-template.md`](../templates/release-template.md),
      including a documented rollback plan.
- [ ] Feature flag rollout stages (internal, beta buyers, GA) completed in
      order for any non-trivial-risk functionality.
- [ ] Zero-downtime deploy verified in a staging rehearsal per
      [`../templates/deployment-template.md`](../templates/deployment-template.md).
