# Land Insight

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
