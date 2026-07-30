---
name: Source UK development sites with the LandTech API
description: >-
  Find candidate land parcels in a UK local authority, then hydrate each one with its HM Land Registry
  titles, associated properties, planning history and development constraints so it can be triaged as
  a development opportunity.
api: openapi/land-insight-api-openapi.yml
operations:
  - getAuthStatus
  - getRegions
  - searchParcels
  - searchParcelsAdvanced
  - getParcelsBulk
  - getParcel
---

# Source UK development sites

Base URL: `https://app.land.tech/api`
Auth: send your LandTech key in the `X-API-Key` header on every request.

## 1. Confirm the key is active

Call `getAuthStatus` (`GET /status/auth`) before anything else. A `200` returns
`{"user":{"state":"active"}}`. A `403` returns the `Forbidden` schema — branch on which member of the
`oneOf` you got: `{"user":{"state":"expired"|"blocked"}}` means the account needs attention, while
`{"message":"..."}` (for example `"User not found"`) means the key itself is wrong. Do not retry
either case; both are terminal until a human fixes the account.

## 2. Resolve the search region

Call `getRegions` (`GET /regions`) to list local authorities as `{gss_code, name}` pairs. GSS codes
match `^[EWS]\d{8}$` — `E09000033` is Westminster. Parcel search is region-scoped, so pick the code
before searching. Never guess a GSS code; always resolve it from this endpoint.

## 3. Search for parcels

For a simple search call `searchParcels` (`POST /parcels/search`) with the region GSS code and
optional parcel size filters. For richer filtering — tenure, ownership type, amenity distance, use
class, planning-application history, power distance, grey belt favourability — call
`searchParcelsAdvanced` (`POST /parcels/advanced-search`), passing a `SearchLocation` that is either a
`Polygon` or a `PointWithRadius`.

Both return only `{"parcel_ids": [...]}` — UUIDs, no attributes. The result set is capped at
**100,000 IDs** and there is no pagination, cursor or offset. If you hit the cap, narrow the filter
(smaller polygon, tighter size band) rather than trying to page; there is no way to reach beyond it.
An empty result is returned as `{"parcel_ids": []}` with a `200`, not a `404`.

## 4. Hydrate the parcels

Call `getParcelsBulk` (`POST /parcels`) with **at most 50 parcel IDs per request** — exceeding the
batch limit returns `400`. Chunk the ID list and iterate. For a single parcel use `getParcel`
(`GET /parcels/{parcel_id}`).

The response is wrapped: `{"data": {"parcel": {...}}}`. The inner `parcel` object is **nullable** — a
parcel that cannot be resolved comes back as `200` with `data.parcel: null`. Null-check the inner
object; do not rely on the status code to detect a miss.

## 5. Read the parcel graph

Each parcel carries `parcel_size` (m²) and `developed_area_ratio` (0 = undeveloped, 1.0 = fully
developed), plus link arrays into the rest of the model:

- `titles[]` — HM Land Registry title numbers → follow with `getTitleDetails` / `getTitlesBulk`
- `properties[]` and `properties_intersects[]` — UPRNs. `properties` are formally associated with the
  title; `properties_intersects` merely fall inside the geometry. Do not treat them as equivalent
  when reasoning about ownership.
- `planning_apps[]` — planning application IDs in `<gss_code>+<ref>` form
- `local_plan_policy`, `development_constraints`, `development_opportunities`, `shlaa` — maps of layer
  name to `[{id, coverage}]`, where `coverage` is the 0–1 fraction of the parcel that the layer
  covers. A green belt entry with `coverage: 0.1` is a very different site from one with
  `coverage: 1`; always read the ratio, never just the presence of the key.

## 6. Triage

A low `developed_area_ratio`, no blocking constraint at high coverage (green belt, flood zone,
Article 4, conservation area), and a positive SHLAA or brownfield-register entry is the shape of a
promising site. Resolve constraint and opportunity IDs through
`getDevelopmentConstraints` / `getDevelopmentOpportunities` / `getShlaas`, and use the `/lite`
variants of those bulk endpoints when you do not need boundary geometry — the payloads are far
smaller.

## Rules

- The API is read-only. There are no writes, so nothing here needs an idempotency key, and every call
  is safe to retry on a transport error.
- There is no rate-limit header and no published quota. Throttle conservatively and batch to the
  documented limits (50 parcels, 200 titles, 200 properties).
- Errors are plain `application/json`, not RFC 9457 `application/problem+json`. Do not parse for
  `type`/`title`/`detail`.
- `searchParcelsAdvanced` is marked **beta** in LandTech's published docs; prefer `searchParcels`
  where its filters suffice.
