---
name: Schedule and record an inspection result
description: Walk an OpenGov Permitting & Licensing inspection from the inspection step on a record through creating an inspection event to posting the inspection result and its checklist results.
api: openapi/opengov-permitting-licensing-v2-openapi.yml
generated: '2026-08-04'
method: generated
source: openapi/opengov-permitting-licensing-v2-openapi.yml
operations:
  - listInspectionTasks
  - getInspectionTask
  - getInspectionTypes
  - postInspectionEvent
  - getInspectionEvents
  - getInspectionResults
  - updateInspectionResult
  - getChecklistResults
  - postChecklistResults
  - patchChecklistResults
permissions:
  - Record Read
  - Workflow Read
  - Workflow Write
---

# Schedule and record an inspection result

Inspections in OpenGov Permitting & Licensing are a chain of four objects: an **inspection step**
on a record's workflow, an **inspection type** on that step, an **inspection event** (the scheduled
visit), and an **inspection result** with **checklist results** underneath it. Getting the order
wrong is the usual failure.

## Before you start

- Base URL `https://api.plce.opengov.com/plce`, community-scoped `/v2/{community}/...`.
- `Authorization: Token <integration API key>`; media type `application/vnd.api+json`.
- Requires **Workflow Write** on the integration, and Record Type access for the record's type.
  Remember the delegation rule: an admin can only grant an integration a permission the admin
  personally holds, so granting Workflow Write requires an Employee role in Permitting & Licensing.

## Steps

1. **Find the inspection step.** `listInspectionTasks` on `/v2/{community}/inspection-steps`, or
   `getInspectionTask` if you already have the `inspectionStepID`. If you started from a record
   number, resolve it first with the *Look up a permit record* skill.
2. **Check what can be inspected.** `getInspectionTypes` on
   `/v2/{community}/inspection-steps/{inspectionStepID}/inspection-types` lists the inspection types
   configured for that step. `postInspectionType` adds one if the type you need is missing.
   The reusable definitions behind these live on `getInspectionTypeTemplates` and
   `getChecklistTemplates` — read the templates to know which checklist items an inspector will face.
3. **Create the inspection event.** `postInspectionEvent` on `/v2/{community}/inspection-events`.
   Confirm with `getInspectionEvent`, or list with `getInspectionEvents`.
4. **Record the result.** `getInspectionResults` to locate the result row created for the event, then
   `updateInspectionResult` (PATCH) on
   `/v2/{community}/inspection-results/{inspectionResultID}` to set the outcome.
5. **Fill the checklist.** `postChecklistResults` to add checklist results under the inspection
   result, `patchChecklistResults` to correct an individual one, `getChecklistResults` to read them
   back.

## Events you will trigger

Writing here produces webhook events on any subscribed integration —
`opengov.plc.inspection.created.v2` and `opengov.plc.inspection.updated.v2`, plus
`opengov.plc.workflowStep.status.changed.v2` when the step advances. Payload schemas are in
`asyncapi/opengov-permitting-licensing-webhooks.yml`. Webhooks are HMAC-SHA256 signed with a
per-subscription secret over `"<X-Webhook-Timestamp>.<raw body>"`; reject anything outside the
5-minute tolerance window.

## No idempotency here

The Permitting & Licensing API exposes **no** idempotency contract — there is no `Idempotency-Key`
header and no `idempotencyKey` body field on these operations (that mechanism exists only in the
Purchase Order API). A retried `postInspectionEvent` or `postChecklistResults` will create a
duplicate. Track your own client-side dedupe key and read back with `getInspectionEvents` before
retrying a POST that timed out.

## Errors

JSON:API `errors[]` envelope. `403` is the common one and means missing permission or missing
Record Type access, not a bad payload. `400` is `"The request did not match the expected schema"`.
See `errors/opengov-problem-types.yml`.

## Safety

No sandbox exists. These are live inspection records for a real municipality. Do not run step 3
onward without explicit authorization.
