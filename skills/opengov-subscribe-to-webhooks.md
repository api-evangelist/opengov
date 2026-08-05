---
name: Subscribe to OpenGov webhook events and verify signatures
description: Stand up a verified webhook endpoint for the 31 OpenGov Permitting & Licensing events, complete the OPTIONS origin handshake, and verify the HMAC-SHA256 signature on every delivery.
api: asyncapi/opengov-permitting-licensing-webhooks.yml
generated: '2026-08-04'
method: generated
source: https://developer.opengov.com/docs/webhooks/managing-subscriptions
operations: []
note: This is a configuration + receiver skill, not an API-call skill — OpenGov has no public API for managing subscriptions, only the Developer Portal UI.
---

# Subscribe to OpenGov webhook events and verify signatures

OpenGov's only real-time surface is Permitting & Licensing webhooks: **31 events**, listed in
`asyncapi/opengov-permitting-licensing-webhooks.yml` with their payload schemas. Enterprise Asset
Management, Procurement and Vendor Management publish no events at all.

## 1. Enable subscriptions

In the integration's settings page in the Developer Portal, open **Event Subscriptions** and flip
the switch in the top right. There is no API for this — it is UI-only.

## 2. Verify your Request URL

Enter your HTTPS endpoint under **Request URL**. OpenGov will not send events to an unverified URL.

**Automatic verification.** OpenGov sends an `OPTIONS` request carrying a `WebHook-Request-Origin`
header. Respond:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Allow: POST
WebHook-Allowed-Origin: https://webhooks.opengov.com
```

Echo the value you received:

```js
res.setHeader("WebHook-Allowed-Origin", req.headers["webhook-request-origin"]);
```

**Manual verification.** The same `OPTIONS` request includes a `WebHook-Request-Callback` header —
visit that URL in a browser or GET it.

## 3. Choose events

Search each event by name in the Event Subscriptions table and add it. The table's **Required
Permission** column tells you which integration permission the event needs; without it the event is
not delivered. Events also respect Record Type access — an integration only receives events for
records it may read.

The event families:

- `opengov.plc.record.*` — submitted, archived, unarchived, applicant added/removed/updated,
  attachment created/deleted/updated, primary and additional locations
- `opengov.plc.workflowStep.*` — created, deleted, restored, assigned, status changed, comment created
- `opengov.plc.inspection.*` — created, updated
- `opengov.plc.payment.*` — made, updated (status: Pending, Failed, Disputed, Lost, Voided)
- `opengov.plc.changeRequest.*` — created, completed, canceled
- `opengov.plc.location.*` — created, updated
- `opengov.plc.comment.created.v2`, `opengov.plc.file.uploaded.v2`

## 4. Verify every delivery

All payloads are signed HMAC-SHA256 with a **per-subscription secret** generated at subscription
creation. Each POST carries:

- `X-Webhook-Timestamp` — Unix epoch seconds when the request was signed
- `X-Webhook-Signature` — `sha256=<hex_digest>`

Recompute over `"<timestamp>.<raw request body>"` — the exact raw bytes, before any JSON parsing:

```js
const signedPayload = `${timestampHeader}.${rawBody}`;
const expected = crypto.createHmac('sha256', secret).update(signedPayload, 'utf8').digest('hex');
```

Reject on mismatch **and** reject when the timestamp is outside the tolerance window (default 300
seconds). Compare digests in constant time.

## 5. Acknowledge

Respond with any `2xx` to confirm receipt. Do your work asynchronously — acknowledge first, process
after.

## Gaps to plan around

- No AsyncAPI document is published, so there is no machine-readable channel/message contract; the
  payload schemas live inside the Permitting & Licensing v2 OpenAPI as `components.schemas`
  (`*WebhookV2`) and are mapped for you in `asyncapi/opengov-permitting-licensing-webhooks.yml`.
- No retry policy, delivery-guarantee statement or dead-letter behaviour is published. Assume
  at-least-once and make your handler idempotent on the event payload.
- No API exists for creating or listing subscriptions — everything is Developer Portal UI.
