---
name: Trace UK land ownership with the LandTech API
description: >-
  Resolve who owns a piece of UK land, starting from a parcel, an HM Land Registry title number or an
  Ordnance Survey UPRN, and follow through to the registered proprietor, lease terms and the ultimate
  corporate owner.
api: openapi/land-insight-api-openapi.yml
operations:
  - getAuthStatus
  - getParcel
  - getTitleDetails
  - getTitlesBulk
  - getPropertyDetails
  - getPropertiesBulk
---

# Trace UK land ownership

Base URL: `https://app.land.tech/api`
Auth: `X-API-Key` header on every request. Verify with `getAuthStatus` first.

## Pick your entry point

Ownership can be entered from three identifiers, each issued by a different UK registry:

| You have | Identifier | Operation |
|---|---|---|
| A parcel from site search | `parcel_id` (UUID) | `getParcel` |
| A Land Registry title | `title_number` (e.g. `WT77573`) | `getTitleDetails` |
| An address / property | `uprn` (1–12 digit numeric string) | `getPropertyDetails` |

## 1. From a parcel to its titles

`getParcel` (`GET /parcels/{parcel_id}`) returns `data.parcel.titles[]` — the HM Land Registry title
numbers associated with the parcel. This array is **nullable**, not merely empty: unregistered land
legitimately has no title. Handle `null` and `[]` as the same "no registered title" outcome.

## 2. Resolve the title

`getTitleDetails` (`GET /titles/{title_number}`) returns `{"data": {"title": {...}}}` with:

- `tenure` — `FREEHOLD`, `LEASEHOLD` or `UNKNOWN_OTHER`. This drives everything downstream: a
  leasehold title tells you about the leaseholder, not the freeholder, so a full ownership picture on
  leasehold land usually needs the superior title too.
- `owners[]` — `OwnerDetails`, each with `last_ownership_change` (nullable date-time) and a `company`
  block that also carries the `ultimate_owner`. Report the ultimate owner when the immediate
  proprietor is a subsidiary or SPV, which is common in development land.
- `registered_lease` — lease terms where the title is leasehold.
- `addresses`, `properties[]`, `properties_intersects[]`.

For many titles at once use `getTitlesBulk` (`POST /titles`) — **up to 200 title numbers per
request**. Chunk beyond that.

## 3. Resolve properties

`properties[]` are UPRNs formally associated with the title. `properties_intersects[]` are UPRNs that
sit inside the parcel geometry but are **not** formally associated with it. Never present an
intersecting UPRN as owned by the titleholder — that is the most common way to get an ownership
answer wrong with this data.

`getPropertyDetails` (`GET /properties/{uprn}`) returns `data.propertyInformation`, which is
**nullable** — a `200` with `null` means no property data was found. It carries `address`
(`full_address`, `property_type`, `property_state`, `property_class_code`, `use_class`),
`building_height` (`highest_point`, `roof_height`, `no_floors_estimate`) and `epc` ratings.
`property_state` is an enum: `Demolished`, `In Use`, `Planning Permission Granted`,
`Under Construction`, `Unoccupied/Vacant/Derelict`. Use `getPropertiesBulk` (`POST /properties`) for
batches — **up to 200 UPRNs per request**.

Note `getPropertyDetails` is the one operation that can return `404` for a missing/unsupplied UPRN, in
addition to the `200`-with-`null` case. Handle both.

## Rules

- Ownership data originates from HM Land Registry and Companies House; the API is a republisher.
  Treat `last_ownership_change` as the recency signal and say so when reporting.
- Read-only: no idempotency contract, all calls safe to retry.
- `403` returns the `Forbidden` schema. `{"user":{"state":"expired"|"blocked"}}` is an account
  problem; `{"message":"..."}` is a request/key problem. Neither is retryable.
- Errors are plain `application/json`, not RFC 9457.
