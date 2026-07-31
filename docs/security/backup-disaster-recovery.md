# Backup & Disaster Recovery

> Every Zodize product MUST meet these backup and recovery targets. Financial-
> grade products meet the stricter "financial-grade" column; a product spec
> MAY commit to tighter targets but MUST NOT commit to looser ones.

## Backup frequency and retention

| Backup type | Frequency | Retention |
|---|---|---|
| Continuous transaction log / WAL shipping | Continuous (financial-grade products only) | 7 days rolling |
| Snapshot | Hourly | 24 hours |
| Daily | Once per day | 30 days |
| Weekly | Once per week | 1 year |
| Monthly (financial-grade and healthcare-grade products only) | Once per month | 7 years, aligned with the audit log retention in [`audit-logging.md`](./audit-logging.md#retention-periods) |

Backups cover: the primary relational database, object storage
(user-uploaded files, generated documents), and the encryption keys required
to decrypt `restricted`-classified columns (key backups are stored separately
from the data they protect, per the key management standard in
[`data-protection-privacy.md`](./data-protection-privacy.md#encryption-at-rest-and-in-transit)).

## RPO / RTO targets by product tier

| Metric | Standard products | Financial-grade products (ZodiBank, ZodiTrade, ZodiXchange, ZodiCapital, ZodiYield) and healthcare-grade (ZodiMed) |
|---|---|---|
| Recovery Point Objective (RPO) — maximum acceptable data loss | 1 hour (last hourly snapshot) | 5 minutes (continuous WAL shipping to a standby) |
| Recovery Time Objective (RTO) — maximum acceptable downtime to restore service | 4 hours | 1 hour |

These targets apply to a full-region-loss scenario. A single-node or
single-AZ failure MUST be handled by the deployment topology described in
[`../architecture/overview.md`](../architecture/overview.md) (multi-AZ
database replicas, load-balanced application nodes) with effectively zero
customer-visible downtime, not by falling back to the backup-restore RTO/RPO
above.

## Backup encryption and offsite/cross-region storage

- Every backup MUST be encrypted at rest using the same key management
  service as production data (SSE-KMS or equivalent), never stored
  unencrypted "for restore simplicity."
- Backups MUST be stored in a region distinct from the primary production
  region, subject to the same data-residency jurisdiction constraints
  described in
  [`data-protection-privacy.md`](./data-protection-privacy.md#data-residency-and-multi-region) —
  cross-region within the same legal jurisdiction family (e.g. a US-East
  primary backing up to US-West, an EU primary backing up to a second EU
  region), never cross-jurisdiction without explicit tenant consent.
- Backup storage MUST use write-once/immutable retention (object lock or
  equivalent) for at least the minimum retention period of each tier above,
  so that a compromised production credential cannot also delete the
  backups covering that data.
- Access to trigger a restore MUST be limited to on-call infrastructure
  engineers and MUST itself be audit-logged (who initiated the restore, of
  which backup, to which target).

## Disaster recovery runbook requirement

Every product MUST maintain a written DR runbook, versioned alongside the
product's operational documentation, covering at minimum:

1. **Detection**: which monitoring alert(s) indicate a disaster-level event
   (region outage, primary database loss, corrupted deploy) per
   [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md).
2. **Declaration**: who has authority to declare a disaster and invoke the
   runbook (named on-call role, not a named individual, so the runbook
   survives personnel changes).
3. **Failover/restore procedure**: exact, numbered steps to promote a
   standby, or restore from the most recent backup meeting the RPO target,
   including the specific commands/console actions and the queue-draining
   and cache-warming steps needed before traffic is restored.
4. **Communication plan**: who notifies affected tenants, on what channel,
   within what time window (target: initial notice within 1 hour of
   declaration for financial-grade products, 4 hours for standard products),
   referencing the product's status-page process.
5. **Post-incident review**: a mandatory retrospective within 5 business
   days of any DR invocation, documenting root cause, timeline, and any
   runbook corrections, filed as an entry the product's incident log.

The runbook MUST name a specific owner and a specific "last reviewed" date;
a runbook older than 6 months without review is treated as stale and MUST be
re-validated before the product's next production-readiness sign-off (see
[`../quality/definition-of-production-ready.md`](../quality/definition-of-production-ready.md)).

## Restore testing cadence

- Standard products: a full restore-from-backup drill (to an isolated
  environment, verified by an automated data-integrity check) MUST be
  performed at least quarterly.
- Financial-grade and healthcare-grade products: a full restore drill MUST
  be performed at least monthly, plus an unannounced failover drill
  (promoting a standby without prior operational warning to the on-call
  engineer) at least twice a year to validate the runbook under realistic
  conditions.
- Every drill MUST be logged with: date, backup tested, time to complete
  restore, whether the RTO target was met, and any discrepancies found. A
  drill that fails to meet its RTO/RPO target MUST produce a remediation
  action item with an owner and a deadline before the product can claim
  production-ready status.

## Related standards

- [`data-protection-privacy.md`](./data-protection-privacy.md)
- [`audit-logging.md`](./audit-logging.md)
- [`../architecture/overview.md`](../architecture/overview.md)
- [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)
- [`../quality/definition-of-production-ready.md`](../quality/definition-of-production-ready.md)
