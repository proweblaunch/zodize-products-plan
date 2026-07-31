# Definition of Production Ready

> The bar an entire PRODUCT must clear before Zodize ships its first
> sellable release and a real buyer deploys it to their own hosting and
> real business data. This is a strictly higher bar than
> [`definition-of-done.md`](./definition-of-done.md), which governs a single
> PR — every module in the product must independently meet DoD, and the
> product as a whole must additionally meet everything below. Because Zodize
> does not operate any product's production infrastructure (the buyer's own
> shared/VPS hosting is the real production environment — see
> [`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer)),
> "production" checks below are validated against Zodize's own internal
> staging/demo environment before release, or delivered as documentation the
> buyer follows on their own hosting, as noted per item. No product (and no
> major new module of an existing product) ships without every item checked,
> signed off by the roles indicated.

## 1. All modules meet Definition of Done

- [ ] Every module in the product has shipped with every
      [`definition-of-done.md`](./definition-of-done.md) item satisfied —
      no module was fast-tracked past DoD "to save time before launch."
- [ ] No open `high` or `critical` severity bug in the issue tracker against
      any in-scope module.
- [ ] No `TODO`/`FIXME` in production code paths without a linked tracked
      issue.

## 2. Security review passed

- [ ] Full OWASP Top 10 mitigation checklist from
      [`../security/security-standards.md`](../security/security-standards.md#owasp-top-10-mapping)
      reviewed and signed off by a security reviewer (not the implementing
      engineer).
- [ ] Dependency audit (`composer audit`, `npm audit`) clean of `high`/
      `critical` advisories at go-live.
- [ ] On a product with multi-company/multi-branch scoping, the cross-branch
      isolation test suite passes for every branch-owned resource per
      [`../development/testing-standards.md`](../development/testing-standards.md#non-negotiable-test-cases).
- [ ] Third-party penetration test completed within the cadence required by
      [`../security/security-standards.md`](../security/security-standards.md#penetration-testing-cadence)
      for the product's tier, with all `critical`/`high` findings
      remediated or covered by a documented, approved compensating control.
- [ ] `security.txt` published and disclosure policy live per
      [`../security/security-standards.md`](../security/security-standards.md#vulnerability-disclosure).
- [ ] Signed off by: Security lead.

## 3. Load testing passed at defined targets

- [ ] The product has been load-tested, on Zodize's internal staging/demo
      environment sized to match the reference deployment target (a single
      shared/VPS host — see
      [`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer)),
      at a minimum of 3x its projected peak concurrent-user load for a
      typical buyer's business without breaching the latency budgets in
      [`performance-standards.md`](./performance-standards.md).
- [ ] PHP-FPM worker counts, database connection pool, and queue worker
      counts appropriate for shared/VPS hosting have been validated under
      that load without connection exhaustion or unbounded queue backlog
      growth, and the validated baseline configuration is documented in the
      product's deployment guide
      ([`../templates/deployment-template.md`](../templates/deployment-template.md))
      for the buyer to apply.
- [ ] A load test result report is attached to the product's readiness
      review, including p50/p95/p99 latency, error rate, and resource
      utilization at target load.
- [ ] Signed off by: Engineering lead for the product.

## 4. Monitoring and alerting configured

- [ ] Every dashboard and alert threshold required by
      [`monitoring-observability.md`](./monitoring-observability.md) is
      live and verified in Zodize's own internal staging/demo environment
      for this product, not just exercised in a local dev build.
- [ ] Zodize's on-call rotation is staffed and the escalation policy in
      [`monitoring-observability.md`](./monitoring-observability.md#alerting-thresholds-and-on-call-escalation)
      is configured in the paging system with real, reachable contacts, for
      Zodize's own internal environment.
- [ ] The health check endpoint responds correctly and is wired into uptime
      monitoring per
      [`monitoring-observability.md`](./monitoring-observability.md#health-check-endpoint-standard),
      and the deployment guide documents how a buyer wires it into their own
      uptime monitoring on their hosting.
- [ ] Signed off by: On-call/SRE lead.

## 5. Backup and disaster recovery verified

- [ ] The backup/restore procedure required by
      [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#backup-frequency-and-retention)
      for the product's tier is documented in the product's deployment guide
      and confirmed by at least one successful backup run and restore drill
      against Zodize's own internal staging/demo environment.
- [ ] A full restore drill has been executed against Zodize's internal
      staging/demo environment (sized to match the reference shared/VPS
      deployment target) within the last quarter (or last month, for
      financial-grade/healthcare-grade products), meeting the RPO/RTO
      targets in
      [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#rpo--rto-targets-by-product-tier).
- [ ] The DR runbook exists, is dated within the last 6 months, names a
      current owner, and its buyer-facing steps are reflected in the
      product's deployment guide
      ([`../templates/deployment-template.md`](../templates/deployment-template.md))
      so a buyer can execute a restore on their own hosting without
      developer assistance.
- [ ] Signed off by: Infrastructure/SRE lead.

## 6. Documentation complete

- [ ] User-facing documentation exists for every feature in scope for
      launch (help center articles or in-app guidance), covering at minimum
      onboarding, core workflows, and account/billing management.
- [ ] API documentation is complete and accurate for every public endpoint,
      per the standard in [`../development/`](../development/), and matches
      the deployed API version.
- [ ] The product's `docs/products/<product>/SPEC.md` is at "Foundation"
      status or better per `../../PRODUCT_CATALOG.md`'s spec status
      definitions, with no unresolved `## Open Questions` blocking launch
      scope.
- [ ] Signed off by: Product lead.

## 7. Support runbooks exist

- [ ] A support runbook exists covering the top anticipated support
      scenarios (account recovery, billing disputes, data export requests
      per
      [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#gdprccpa-equivalent-rights),
      and how Zodize support diagnoses a buyer-reported issue against their
      own self-hosted deployment without direct database/server access,
      typically via the buyer sharing logs/exports or a screen-share
      session — Zodize support has no standing access to any buyer's
      deployment).
- [ ] Escalation path from support to engineering is defined and tested
      (a real ticket has been routed through it at least once before
      launch).
- [ ] Signed off by: Support/Customer Success lead.

## 8. Legal and compliance sign-off (regulated products)

Required only for financial-grade products (ZodiBank, ZodiTrade,
ZodiXchange, ZodiCapital, ZodiYield), healthcare-grade products (ZodiMed),
and government products (ZodiGov). Not applicable to other products, which
skip directly to the final checklist item.

- [ ] Applicable regulatory requirements for the product's target market are
      documented in the product SPEC and mapped to the specific controls in
      [`../security/`](../security/) that satisfy them.
- [ ] Data residency guidance per
      [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-residency)
      is documented in the product's deployment guide for the buyer to apply
      when choosing their own hosting region.
- [ ] Legal has reviewed and approved the Terms of Service, Privacy Policy,
      and any regulated-industry disclosures (e.g. banking disclosures,
      HIPAA-equivalent notices) presented to end users.
- [ ] Signed off by: Legal/Compliance lead and Security lead jointly.

## Final gate

- [ ] All applicable sections above are checked and signed off.
- [ ] A go/no-go review is held with every signing role present (or their
      explicit written sign-off recorded), referencing the go/no-go
      checklist in [`../checklists/`](../checklists/).
- [ ] The decision (go or no-go, and if no-go, the specific blocking items)
      is recorded in the product's own changelog/release record.

## Related standards

- [`definition-of-done.md`](./definition-of-done.md)
- [`ci-cd-standards.md`](./ci-cd-standards.md)
- [`performance-standards.md`](./performance-standards.md)
- [`monitoring-observability.md`](./monitoring-observability.md)
- [`../security/security-standards.md`](../security/security-standards.md)
- [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md)
