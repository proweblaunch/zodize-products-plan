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
every item below.

- [ ] Vision and target market documented.
- [ ] Primary personas documented, each with goals and pain points.
- [ ] Competitive positioning documented.
- [ ] Full data model with ER relationships documented, extending
      [`../templates/database-template.md`](../templates/database-template.md).
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

## Support

- [ ] Admin/back-office tooling live per
      [`../templates/admin-template.md`](../templates/admin-template.md),
      including impersonation, feature flags, system health, and support tools.
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
