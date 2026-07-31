# Security Checklist

This is the pre-release security checklist every Zodize product MUST pass.
It is a companion to [`../security/`](../security), which defines the
underlying standards; every item below MUST be verified against the actual
running system, not assumed from design intent. This checklist is a
mandatory sub-gate of
[`../checklists/production-readiness-checklist.md`](./production-readiness-checklist.md).

## Authentication and session management

- [ ] Authentication flows match
      [`../templates/authentication-template.md`](../templates/authentication-template.md)
      exactly, including all required states (loading/error/success).
- [ ] Password policy enforced both client- and server-side per
      [`../security/authentication-authorization.md`](../security/authentication-authorization.md).
- [ ] MFA enrollment available and, where required by the product's tier or
      role, mandatory before core functionality is accessible.
- [ ] Session tokens expire per policy and are invalidated on logout,
      password change, and MFA re-enrollment.
- [ ] Account enumeration is not possible via login, registration, or
      password-reset error messages.
- [ ] Rate limiting is active on login, MFA challenge, and password-reset
      endpoints.

## Authorization

- [ ] Every permission in use is registered per
      [`../templates/permission-template.md`](../templates/permission-template.md);
      no ad hoc authorization checks exist outside the Policy layer.
- [ ] Every API endpoint has an automated test asserting a `403` for a
      request lacking the required permission, per
      [`../templates/testing-template.md`](../templates/testing-template.md).
- [ ] Cross-tenant access is impossible and covered by an automated test
      per [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).
- [ ] User impersonation, if used, writes a mandatory audit log entry on
      start and end per [`../templates/admin-template.md`](../templates/admin-template.md).

## Data protection

- [ ] All data in transit is encrypted (TLS) with no mixed-content or
      HTTP-accessible endpoints.
- [ ] Sensitive data at rest (MFA secrets, webhook secrets, API tokens) is
      encrypted per [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md).
- [ ] No raw API tokens, passwords, or secrets are logged, ever, at any log
      level.
- [ ] Personally identifiable information is excluded from analytics event
      payloads per [`../templates/marketing-website-template.md`](../templates/marketing-website-template.md).

## API and input handling

- [ ] Every endpoint follows the standard error format from
      [`../templates/api-template.md`](../templates/api-template.md), with
      no internal error detail (stack traces, SQL, file paths) exposed to
      clients.
- [ ] All user input is validated server-side via Request classes, never
      trusted from client-side validation alone.
- [ ] File uploads (if any) are validated by type and size, scanned per
      [`../security/`](../security) policy, and stored outside the
      web-accessible document root.
- [ ] Webhook payloads are signed (HMAC) per
      [`../templates/database-template.md`](../templates/database-template.md)
      `webhooks.secret` and verified on receipt where the product consumes
      inbound webhooks.

## Audit and monitoring

- [ ] Every sensitive action (auth events, permission changes, data export,
      impersonation, billing changes) writes an audit log entry per
      [`../security/audit-logging.md`](../security/audit-logging.md).
- [ ] Audit log entries are immutable and cannot be deleted or edited
      through any application code path.
- [ ] Error tracking (per [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md))
      is live in production and alerts a real on-call channel.

## Infrastructure

- [ ] Secrets are sourced from the secrets manager, never committed to
      source control, per [`../templates/deployment-template.md`](../templates/deployment-template.md).
- [ ] Dependencies are scanned for known vulnerabilities as part of CI.
- [ ] Production access is restricted to the minimum necessary set of
      engineers, per [`../security/`](../security) access control policy.
- [ ] Disaster recovery procedure tested per
      [`../security/`](../security) DR standard.

## Sign-off

- [ ] Security review completed by someone other than the implementing
      engineer.
- [ ] All findings from the security review are resolved or explicitly
      accepted as a documented risk, not silently deferred.
