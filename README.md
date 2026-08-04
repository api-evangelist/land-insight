# Land Insight

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

LandTech ([land.tech](https://land.tech)) is a UK property technology company whose LandInsight platform is used by developers, land agents, planners and investors to source and assess development sites. It combines HM Land Registry ownership and title data, planning applications, local plan policy, development constraints (Green Belt, flood zones, Article 4 directions, heritage designations), Land Availability Assessments, brownfield and regeneration opportunities, EPC and property attributes, and electricity network / renewable energy infrastructure into a single map-based product. Alongside LandInsight the company ships LandFund for development finance appraisal and Give My View for community engagement.

Backed by: seedcamp

## API

The **LandTech API** ([land.tech/api](https://land.tech/api)) exposes the same UK land and planning datasets as a paid HTTP product, so customers can feed parcel search, ownership, planning application and constraint data into their own systems.

- Base URL: `https://app.land.tech/api`
- Reference: https://developers.land.tech/openapi
- Spec: OpenAPI 3.0.3, version 0.3.0 — 38 operations, 140 schemas, 12 tags
- Auth: a single API key in the `X-API-Key` header
- Access: commercial, arranged via [demo request](https://land.tech/request-ecosystem-demo)

The data model is parcel-centric: a parcel links out to HM Land Registry titles, Ordnance Survey UPRNs, planning applications, and four families of overlay layers, each carrying a `coverage` ratio for how much of the parcel it covers.

## Artifacts

| Directory | Artifact |
|---|---|
| `openapi/` | LandTech API OpenAPI 3.0.3, harvested verbatim |
| `authentication/` | API-key auth profile |
| `conventions/` | Cross-cutting semantics — batching, bounded result sets, response envelope, identifiers |
| `errors/` | Error catalog derived from 4xx responses |
| `data-model/` | Entity-relationship graph |
| `lifecycle/` | Versioning, beta operations, access model |
| `conformance/` | Standards posture and upstream data registries |
| `security/` | Domain security probe, trust center (ISO 27001, GDPR) |
| `well-known/` | `/.well-known/` probe results |
| `llms/` | `llms.txt`, harvested verbatim from the developer portal |
| `mcp/` | Candidate MCP tool surface derived from the OpenAPI |
| `skills/` | Three Agent Skills grounded in real operationIds |
| `overlays/` | OpenAPI Overlay 1.0.0 of API Evangelist enhancements |
| `packages/` | Registry survey — no first-party API client SDK published |
