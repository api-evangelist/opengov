---
name: Onboard and promote a vendor
description: Create a vendor in OpenGov Vendor Management, attach its certificates and documents, submit it into the approval workflow, and promote it once approved — including the bulk CSV import path.
api: openapi/opengov-vendor-management-openapi.yml
generated: '2026-08-04'
method: generated
source: openapi/opengov-vendor-management-openapi.yml
operations:
  - vendor-search.searchVendors
  - vendors.createVendor
  - vendors.getVendorById
  - vendors.updateVendor
  - attachments.createAttachment
  - vendors.submitVendor
  - vendor-taskmaster.getVendorApprovalPending
  - vendor-taskmaster.getVendorTaskMasterActions
  - vendor-taskmaster.submitVendorTaskMasterOutcome
  - vendors.promoteVendor
  - vendor-imports.downloadTemplate
  - vendor-imports.requestPresignedUpload
  - vendor-imports.processImport
  - vendor-imports.validateJob
  - vendor-imports.remapColumns
  - vendor-imports.triggerImport
---

# Onboard and promote a vendor

## Before you start

- Base URL `https://api.vendor.opengov.com`.
- Entity-scoped: `/api/v1/entities/{entityId}/vendors/...`. Vendors are addressed by `vendorKey`,
  not a numeric id.
- Auth: `Authorization: Bearer <JWT>`. This API declares bearer only —
  see `authentication/opengov-authentication.yml`.
- Media type `application/json`. Paging is offset/limit, unlike Permitting & Licensing.

## Single-vendor path

1. **Check for a duplicate first.** `vendor-search.searchVendors` (POST
   `/vendors/search`) or `vendor-search.searchVendorsByVendorKeys`. Vendor Management has **no
   idempotency contract**, so a retried `createVendor` after a timeout creates a second vendor —
   search before you create, and search again before you retry.
2. **Create.** `vendors.createVendor` — `POST /api/v1/entities/{entityId}/vendors`. Read back with
   `vendors.getVendorById`; correct with `vendors.updateVendor` (PATCH).
3. **Attach documents.** `attachments.createAttachment` on
   `/vendors/{vendorKey}/attachments`. Certificates of insurance and W-9s belong here.
   `attachments.getAttachmentContent` streams a file back; `attachments.deleteAttachments` is a
   bulk POST, not a DELETE.
4. **Add context.** `vendor-notes.createVendorNote` for internal commentary.
5. **Submit into the workflow.** `vendors.submitVendor` —
   `POST /vendors/{vendorKey}/submit`.
6. **Work the approval.** `vendor-taskmaster.getVendorApprovalPending` tells you whether it is
   waiting; `vendor-taskmaster.getVendorTaskMasterActions` lists the actions available to you;
   `vendor-taskmaster.submitVendorTaskMasterOutcome` records the decision.
7. **Promote.** `vendors.promoteVendor` — `POST /vendors/{vendorKey}/promote` — moves the approved
   vendor into the active vendor list. `vendors.updateVendorAvailability` toggles availability
   afterwards.

## Bulk import path

1. `vendor-imports.downloadTemplate` — get the CSV template. Do not guess the columns.
2. `vendor-imports.requestPresignedUpload` — get an upload target, then PUT your CSV to it.
3. `vendor-imports.processImport` — register the uploaded file as an import job.
4. `vendor-imports.validateJob` — validate before committing anything.
5. `vendor-imports.remapColumns` — fix column mapping if validation flags it.
6. `vendor-imports.triggerImport` — run the import.
7. `vendor-imports.getImportDetail` / `vendor-imports.listImports` — watch it and read the outcome.

Always run step 4 and read its output before step 6. There is no undo and no sandbox.

## Errors

Flat coded JSON — `{"status":..., "code":..., "detail":...}`. `403` on a missing permission or the
wrong entity, `404` `EntityNotFoundError` on an unknown `vendorKey`, `409` `ConflictError` on a
state-machine violation such as promoting a vendor that was never approved. See
`errors/opengov-problem-types.yml`.

## No events

Vendor Management publishes **no webhook events** — the developer portal states "No events
available yet." If you need to react to a promotion, you must poll
`vendor-search.searchVendors` or `vendor-taskmaster.getVendorApprovalPending`.

## Safety

Live production vendor master data for a government entity. No sandbox exists. Do not run the
bulk import or the promote step without explicit authorization.
