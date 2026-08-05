---
name: Look up a permit record by its public record number
description: Translate a citizen-facing OpenGov Permitting & Licensing record number (e.g. PL-23-00042) into a record ID, then read the record, its workflow steps and its attachments.
api: openapi/opengov-permitting-licensing-v2-openapi.yml
generated: '2026-08-04'
method: generated
source: openapi/opengov-permitting-licensing-v2-openapi.yml + https://developer.opengov.com/docs/plc/use-cases
operations:
  - listRecords
  - getRecord
  - listRecordSteps
  - listRecordAttachments
  - getRecordAttachment
  - listRecordStepComments
permissions:
  - Record Read
  - Workflow Read
  - Comment Read
---

# Look up a permit record by its public record number

Every OpenGov Permitting & Licensing operation that touches a record needs a **record ID**, but
the number a resident or a staff member quotes is the **record number** shown in the UI
(`PL-23-00042`). Translating one to the other is the first step of almost every real flow.

## Before you start

- Base URL: `https://api.plce.opengov.com/plce`
- Every path is community-scoped: `/v2/{community}/...` where `{community}` is the community
  subdomain (e.g. `sampletown`).
- Auth header: `Authorization: Token <integration API key>` (see
  `authentication/opengov-authentication.yml`).
- Your integration must hold **Record Read**, and it must be enabled on the Record Type you are
  querying — Record Type access is granted separately in Permitting & Licensing System Settings >
  Record Types > Access. Without it every call returns `403 Forbidden` with
  `"No access to requested record type"`.
- Media type is JSON:API — send and expect `application/vnd.api+json`.

## Steps

1. **Resolve the number to an ID.** Call `listRecords` with the JSON:API filter:
   `GET /v2/{community}/records?filter[number]=PL-23-00042`.
   Read `data[0].id`. An empty `data` array means the number does not exist *or* your integration
   is not enabled on that Record Type — those two cases look identical, so check Record Type access
   before concluding the record is missing.
2. **Read the record.** `getRecord` on `/v2/{community}/records/{recordID}`. Use `include` to pull
   related resources in one round trip rather than N+1 calls.
3. **Read the workflow.** `listRecordSteps` returns the workflow steps on the record. Step detail
   lives on the typed step collections (`listApprovalTasks`, `listInspectionTasks`,
   `listDocumentGenerationTasks`, `listPaymentTasks`) — the workflow-step list tells you which type
   each step is.
4. **Read comments and attachments.** `listRecordStepComments` for step commentary,
   `listRecordAttachments` then `getRecordAttachment` for documents.

## Filtering and paging rules

- Filters are JSON:API bracketed: `filter[number]`, `filter[applicantUserID]`, `filter[locationID]`,
  `filter[archived]`.
- Date comparisons nest an operator: `filter[createdAt][gt]=2025-03-01`, `filter[createdAt][lt]=...`.
  Only `gt` and `lt` are documented.
- Page with `page[size]` and `page[number]`. Sort with `sort`.

## Errors

Errors come back as a JSON:API `errors[]` array, not RFC 9457 problem details:

```json
{"errors":[{"status":"403","title":"Forbidden","detail":"No access to requested record type"}]}
```

`401` means the key is wrong or missing. `403` means one of: missing permission, wrong entity, or
Record Type access not granted. `404` is `EntityNotFoundError`. See
`errors/opengov-problem-types.yml`.

## Rate limits

Responses carry `X-RateLimit-Limit` and `X-RateLimit-Remaining`. OpenGov publishes **no numeric
quota**, declares no `429` response and sends no `Retry-After`, so read the remaining counter and
back off on your own schedule rather than assuming a limit.

## Safety

There is **no sandbox**. The API Test Console and these calls run against live data for a real
community. Keep this skill to read operations unless you have explicit authorization to write.
