# Data Protection & Privacy

> Defines how every Zodize product classifies, protects, and disposes of
> data. Financial-grade and healthcare-grade products inherit these as a
> floor and MUST layer additional controls documented in their own SPEC.

## Data classification levels

Every database column and file-storage bucket MUST be classified into one of
four levels, documented in the product's data dictionary
(`docs/products/<product>/SPEC.md`):

| Level | Definition | Examples | Baseline control |
|---|---|---|---|
| Public | Safe for unauthenticated disclosure. | Marketing page content, public API documentation. | No special control. |
| Internal | Not for public disclosure but low individual harm if leaked. | Internal feature flags, non-PII configuration. | Authenticated access only. |
| Confidential | Business-sensitive or personal data whose disclosure causes real harm. | Names, email addresses, phone numbers, invoice amounts, addresses. | Authenticated + authorized access only; encrypted in transit; audit-logged export per [`audit-logging.md`](./audit-logging.md). |
| Restricted | Highly sensitive personal, financial, or health data; regulated. | Bank account/routing numbers, card PANs, SSNs/national IDs, patient diagnoses, trading positions, authentication secrets. | Encrypted at rest at the column level, encrypted in transit, audit-logged on read and write, access restricted to the minimum role set required. |

Columns storing `restricted` data MUST use Laravel's encrypted casts
(`encrypted`, `encrypted:array`) or an application-layer field-level
encryption library — never rely on disk/volume encryption alone as the sole
control for `restricted` data.

## Encryption at rest and in transit

- **In transit**: TLS 1.2+ everywhere per
  [`security-standards.md`](./security-standards.md#tls-requirements),
  including internal service-to-service traffic.
- **At rest, database**: the production database volume MUST use
  storage-level encryption (e.g. AES-256 EBS/RDS encryption or equivalent)
  as a baseline for every product. `Confidential` and `restricted` columns
  additionally require application-level column encryption as noted above,
  so that a raw database dump or backup does not expose them in plaintext.
- **At rest, files**: object storage (S3-compatible) buckets MUST have
  server-side encryption enabled by default (SSE-S3 or SSE-KMS); buckets
  holding `restricted` files (identity documents, medical attachments) MUST
  use SSE-KMS with a customer-managed key so access can be revoked
  independently of the storage layer.
- Encryption keys MUST be managed through the platform's key management
  service, never hardcoded or stored alongside the data they protect. Key
  rotation follows the same schedule as the secrets rotation policy in
  [`security-standards.md`](./security-standards.md#secrets-management).

## PII handling

- Personally identifiable information MUST be collected only when a product
  feature requires it, documented in the product's SPEC as a stated purpose
  — no speculative collection "in case it's useful later."
- PII MUST NOT appear in application logs, error-tracking payloads (see
  [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)),
  or URL query strings. Logging middleware MUST redact known PII field names
  (`password`, `ssn`, `card_number`, `dob`, etc.) before a log line is
  written.
- PII in non-production environments (staging, local development, QA) MUST
  be synthetic or masked — production data MUST NOT be copied into a
  lower environment without field-level masking of every `confidential` and
  `restricted` column.

## Data residency and multi-region

- Every tenant MUST have a declared data residency region at provisioning
  time (see
  [`../architecture/multi-tenancy.md#tenant-provisioning-and-deprovisioning`](../architecture/multi-tenancy.md)).
  Primary data storage (database, object storage, backups) for that tenant
  MUST remain within the declared region.
- Products serving EU tenants MUST offer an EU-region deployment; products
  serving tenants in jurisdictions with data-localization law (e.g. certain
  government or banking contracts) MUST document the specific region
  commitment in the tenant's contractual record, not just in infrastructure
  config.
- Cross-region replication for disaster recovery (see
  [`backup-disaster-recovery.md`](./backup-disaster-recovery.md)) MUST stay
  within the same legal jurisdiction family as the primary region unless the
  tenant has explicitly consented to broader replication.

## GDPR/CCPA-equivalent rights

Every product MUST implement the following as self-service features,
available to any authenticated end user for their own data and to a tenant
Admin/Owner for their tenant's data, without requiring a support ticket:

- **Right to access/export**: a "Download my data" action that produces a
  machine-readable (JSON) export of the user's personal data within 30 days
  of request (target: within minutes via an async job for standard
  accounts).
- **Right to deletion**: a "Delete my account" action that triggers the
  deletion workflow described below. Requests MUST be fulfilled within 30
  days, consistent with GDPR Article 12 timelines.
- **Right to rectification**: users can edit their own profile/PII fields
  directly; where a field is not self-editable (e.g. an identity-verified
  name), a correction request workflow routed to an Admin MUST exist.
- **Consent tracking**: every marketing/communication consent and every
  acceptance of Terms of Service or Privacy Policy MUST be recorded with a
  timestamp, the version of the document accepted, and the mechanism
  (checkbox, click-through), stored in a `consents` table — this is a
  distinct, permanent record, not overwritten on the next consent change.

## Data retention and deletion

- Every resource type MUST have a documented retention period in the
  product's SPEC. In the absence of a longer regulatory requirement, the
  default retention for operational business data is the life of the
  tenant's subscription plus 90 days.
- **Soft-delete standard**: user-facing deletions (a user deletes an
  invoice, a record, a document) default to a soft delete (`deleted_at`
  timestamp via Eloquent `SoftDeletes`), recoverable from a "Trash" view for
  30 days, after which a scheduled job MUST hard-delete the record and any
  associated files.
- **Hard-delete standard**: hard deletion (irreversible purge from the
  primary database and from backups on their next rotation) is required for:
  (a) records past their soft-delete recovery window, (b) any record subject
  to a fulfilled right-to-deletion request, (c) `restricted`-classified data
  once its documented retention period expires.
- A right-to-deletion request MUST hard-delete or irreversibly anonymize
  personal data within the fulfillment window, with the sole exception of
  audit log entries, which retain a reference per
  [`audit-logging.md`](./audit-logging.md#retention-periods), and any record
  a product is legally required to retain (e.g. a completed financial
  transaction ledger entry) — the SPEC MUST name that exception explicitly
  and anonymize every non-required field on the retained record.
- Backups containing deleted data are permitted to retain it only until that
  backup's own retention period expires naturally, per
  [`backup-disaster-recovery.md`](./backup-disaster-recovery.md); products
  MUST NOT promise instantaneous backup purging, but MUST disclose the
  effective maximum retention tail in their privacy policy.

## Related standards

- [`audit-logging.md`](./audit-logging.md)
- [`backup-disaster-recovery.md`](./backup-disaster-recovery.md)
- [`security-standards.md`](./security-standards.md)
- [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md)
