# Webhook Standards

## Event model

Webhooks are fired from the same domain events used internally for
notifications and audit logging (see
[caching-queues-events.md](../architecture/caching-queues-events.md)) — there
is exactly one source of truth for "what happened," fanned out to multiple
consumers (in-app notification, email, webhook, audit log).

## Registration

- A tenant registers a webhook endpoint with: target URL (HTTPS only), a
  subset of event types to receive, and receives a generated signing secret
  at creation time.
- Endpoints support multiple subscriptions with different event filters.
- Endpoint health (recent delivery success rate) is visible in-product;
  an endpoint is auto-disabled and the tenant notified after 100 consecutive
  failures.

## Payload format

```json
{
  "id": "evt_01HXYZ...",
  "type": "invoice.paid",
  "created_at": "2026-07-31T12:00:00Z",
  "tenant_id": "ten_...",
  "data": { "...": "the resource, matching the REST API representation" }
}
```

- `type` follows `resource.action` naming, matching the RBAC permission
  naming convention in [rbac-permissions.md](../security/rbac-permissions.md)
  for consistency of mental model.
- `data` uses the exact same resource shape as the corresponding REST API
  response — a webhook payload and a `GET` response for the same resource
  never diverge in shape.

## Signing and verification

Every delivery includes a `Zodize-Signature` header:
`t=<unix_timestamp>,v1=<hex_hmac_sha256>`, computed over
`{timestamp}.{raw_body}` using the endpoint's signing secret. Receivers must
verify the signature and reject requests where the timestamp is more than 5
minutes old (replay protection). Reference implementation and verification
snippets are published per [sdk-standards.md](./sdk-standards.md).

## Delivery guarantees

- At-least-once delivery. Consumers must be idempotent using the event `id`.
- Retry schedule on non-2xx response or timeout (10s): immediate, then
  exponential backoff — 1m, 5m, 30m, 2h, 12h — up to 24 hours, then marked
  failed and surfaced in the endpoint's delivery log.
- Full delivery log (payload, response status, response body, timestamp) is
  retained for 30 days and viewable/replayable by the tenant.

## Ordering

Webhooks are not guaranteed to arrive in the order the underlying events
occurred across different event types. Within a single resource's lifecycle,
delivery is best-effort ordered but consumers must not assume strict
ordering — payloads carry enough state (not deltas) to be applied safely out
of order.

## Versioning

A webhook payload's `data` shape versions together with the REST API version
the tenant's integration was created against; a breaking payload change
requires a new API version per [api-standards.md](./api-standards.md#versioning).
