---
name: Create, submit and issue a purchase order
description: Drive an OpenGov purchase order through its full state machine — draft, submit, approve, issue — using the only OpenGov API that publishes a real idempotency contract.
api: openapi/opengov-purchase-order-openapi.yml
generated: '2026-08-04'
method: generated
source: openapi/opengov-purchase-order-openapi.yml
operations:
  - purchaseOrder.create
  - purchaseOrder.search
  - purchaseOrder.findById
  - purchaseOrder.update
  - purchaseOrder.submit
  - purchaseOrder.approve
  - purchaseOrder.reject
  - purchaseOrder.issue
  - purchaseOrder.cancel
  - purchaseOrder.close
  - purchaseOrder.delete
---

# Create, submit and issue a purchase order

The OpenGov Purchase Order API is a state machine, not CRUD. A PO moves
draft → submitted → approved → issued → closed, and each transition is its own POST.

## Before you start

- Base URL `https://api-purchase-order.procurement.opengov.com`.
- Every path is entity-scoped: `/api/v1/po/entities/{entityId}/purchase-orders/...`.
- Auth: `Authorization` header carrying the Procurement API key generated in the **product Control
  Panel** — not the Developer Portal integration key used by Permitting & Licensing. Bearer tokens
  and an `x-api-key` header are also declared. See `authentication/opengov-authentication.yml`.
- Media type `application/json` (this API is not JSON:API).

## Steps

1. **Create the draft.** `purchaseOrder.create` — `POST /api/v1/po/entities/{entityId}/purchase-orders`.
   Add line items with the `lineItem` operations before submitting.
2. **Check it before you commit.** Every state-transition operation accepts a `dryRun: boolean` in
   the request body. Send `dryRun: true` first — it validates the transition without performing it.
   This is the closest thing OpenGov offers to a sandbox; there is no test mode and no test tenant.
3. **Submit for approval.** `purchaseOrder.submit` —
   `POST /api/v1/po/entities/{entityId}/purchase-orders/{id}/submit`.
4. **Approve or reject.** `purchaseOrder.approve` / `purchaseOrder.reject`.
5. **Issue.** `purchaseOrder.issue` puts the PO into effect.
6. **Close or cancel.** `purchaseOrder.close` at the end of life; `purchaseOrder.cancel` to stop one
   in flight. `purchaseOrder.delete` only works on a draft.
7. **Find POs.** `purchaseOrder.search` (POST, cursor paged with `first`/`after`) or
   `purchaseOrder.findById`.

## Idempotency — use it

This is the **only** OpenGov API with an idempotency contract, and it is not the conventional
header. Where an operation declares it, `idempotencyKey` is a **required UUID in the JSON request
body**:

- Replaying a request with the same `idempotencyKey` returns **200** with the original outcome
  instead of **202**.
- Replaying while the first request is still in flight returns **409** with
  `code: IDEMPOTENCY_CONFLICT` and the detail *"A request with this idempotency key is currently
  being processed. Retry after the in-flight request completes."*

Thirty-nine operations in this spec declare that 409. Generate one UUID per logical intent, reuse
it across retries, and treat 409 as *wait and retry*, not *fail*. The `invoiceSync` and
`receiptSync` operations require it outright.

## Errors

Flat coded JSON, not JSON:API and not RFC 9457:

```json
{"status":409,"code":"IDEMPOTENCY_CONFLICT","detail":"..."}
```

Named types you will see: `ValidationError` (400), `AuthError` (401), `UnauthorizedError` (403),
`EntityNotFoundError` (404), `IdempotencyConflictError` (409), `PayloadTooLargeHttpError` (413),
`InfrastructureError` (500). `BUDGET_INSUFFICIENT` appears as a machine-readable code on budget
checks. See `errors/opengov-problem-types.yml`.

## Safety

Issuing a purchase order commits public funds. There is no sandbox and no test mode — run
`dryRun: true` first, every time, and never issue or approve without explicit authorization.
