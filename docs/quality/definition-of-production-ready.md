# Definition of Production Ready

> The bar an entire PRODUCT must clear before it accepts real customer
> traffic and real data. This is a strictly higher bar than
> [`definition-of-done.md`](./definition-of-done.md), which governs a single
> PR — every module in the product must independently meet DoD, and the
> product as a whole must additionally meet everything below. No product
> (and no major new module of an existing product) goes live without every
> item checked, signed off by the roles indicated.

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
- [ ] Cross-tenant isolation test suite passes for every tenant-owned
      resource per
      [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md#cross-tenant-data-leakage-prevention).
- [ ] Third-party penetration test completed within the cadence required by
      [`../security/security-standards.md`](../security/security-standards.md#penetration-testing-cadence)
      for the product's tier, with all `critical`/`high` findings
      remediated or covered by a documented, approved compensating control.
- [ ] `security.txt` published and disclosure policy live per
      [`../security/security-standards.md`](../security/security-standards.md#vulnerability-disclosure).
- [ ] Signed off by: Security lead.

## 3. Load testing passed at defined targets

- [ ] The product has been load-tested at a minimum of 3x its projected
      peak concurrent-user load (or the contractually committed load for a
      named enterprise customer, whichever is higher) without breaching the
      latency budgets in
      [`performance-standards.md`](./performance-standards.md).
- [ ] Database connection pool and queue worker counts have been validated
      under that load without connection exhaustion or unbounded queue
      backlog growth.
- [ ] A load test result report is attached to the product's readiness
      review, including p50/p95/p99 latency, error rate, and resource
      utilization at target load.
- [ ] Signed off by: Engineering lead for the product.

## 4. Monitoring and alerting configured

- [ ] Every dashboard and alert threshold required by
      [`monitoring-observability.md`](./monitoring-observability.md) is
      live in the production environment, not just staging.
- [ ] On-call rotation is staffed and the escalation policy in
      [`monitoring-observability.md`](./monitoring-observability.md#alerting-thresholds-and-on-call-escalation)
      is configured in the paging system with real, reachable contacts.
- [ ] Health check endpoint responds correctly and is wired into uptime
      monitoring per
      [`monitoring-observability.md`](./monitoring-observability.md#health-check-endpoint-standard).
- [ ] Signed off by: On-call/SRE lead.

## 5. Backup and disaster recovery verified

- [ ] Backups are running on the schedule required by
      [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#backup-frequency-and-retention)
      for the product's tier, confirmed by at least one successful
      automated backup observed in production infrastructure (not just
      staging).
- [ ] A full restore drill has been executed against a production-equivalent
      environment within the last quarter (or last month, for
      financial-grade/healthcare-grade products), meeting the RPO/RTO
      targets in
      [`../security/backup-disaster-recovery.md`](../security/backup-disaster-recovery.md#rpo--rto-targets-by-product-tier).
- [ ] The DR runbook exists, is dated within the last 6 months, and names a
      current owner.
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
      impersonation procedure per
      [`../security/rbac-permissions.md`](../security/rbac-permissions.md#default-system-roles)).
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
- [ ] Data residency commitments per
      [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-residency-and-multi-region)
      are verified against actual infrastructure region configuration.
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
