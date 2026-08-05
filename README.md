# OpenGov

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

OpenGov builds cloud ERP and public service software for state and local government, serving more than
2,000 communities across budgeting and performance, financial management, procurement and contract
management, vendor management, permitting and licensing, enterprise asset management, tax and revenue,
utility billing, grants management, 311 request management and open data.

## API surface

OpenGov runs a public developer portal at [developer.opengov.com](https://developer.opengov.com) with an
API catalog of **ten OpenAPI 3.x definitions covering roughly 358 operations**:

| API | Operations | Base URL |
|---|---|---|
| Permitting & Licensing v2 | 113 | `https://api.plce.opengov.com/plce` |
| Permitting & Licensing v1 | 10 | `https://api.plce.opengov.com/plce` |
| Purchase Order | 82 | `https://api-purchase-order.procurement.opengov.com` |
| Vendor Management | 44 | `https://api.vendor.opengov.com` |
| Receipt | 26 | `https://api-receipts.procurement.opengov.com` |
| Procurement & Contract Management v2 | 22 | `https://api.procurement.opengov.com` |
| Procurement & Contract Management v1 | 22 | `https://api.procurement.opengov.com` |
| Open Data (CKAN Action API 2.9) | 16 | per-customer Open Data portal host |
| Enterprise Asset Management | 13 | per-tenant `{serverURL}` |
| Budgeting & Performance | 10 | `https://api.bnp.opengov.com` |

Also published: 31 HMAC-signed Permitting & Licensing webhook events, SCIM 2.0 identity provisioning,
an in-browser API Test Console on every operation, a per-integration permission model, and an `llms.txt`
at [developer.opengov.com/llms.txt](https://developer.opengov.com/llms.txt).

## Links

- Website: https://opengov.com
- Developer portal: https://developer.opengov.com
- API catalog: https://developer.opengov.com/catalog
- Status: https://status.opengov.com/
- Trust center: https://trust.opengov.com/
- Security / responsible disclosure: https://opengov.com/security/
