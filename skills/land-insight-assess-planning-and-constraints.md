---
name: Assess planning history and development constraints with the LandTech API
description: >-
  Pull the planning application history for a site and resolve every policy layer, constraint and
  opportunity designation covering it, so the planning risk and upside of a UK parcel can be judged.
api: openapi/land-insight-api-openapi.yml
operations:
  - getAuthStatus
  - getRegions
  - searchPlanningApplications
  - getPlanningApplication
  - getPlanningApplicationById
  - getPlanningApplicationByIds
  - getDevelopmentConstraints
  - getDevelopmentConstraintsLite
  - getDevelopmentOpportunities
  - getDevelopmentOpportunitiesLite
  - getLocalPlanPolicies
  - getShlaas
  - getPowers
---

# Assess planning history and constraints

Base URL: `https://app.land.tech/api`
Auth: `X-API-Key` header. Verify with `getAuthStatus`.

## 1. Find the planning applications

Geographically, call `searchPlanningApplications` (`POST /planning-applications/search`) with a
geometry. It returns application references **grouped by planning authority GSS code** — the grouping
matters, because a reference is only unique within its authority.

If you already have a parcel, `getParcel` gives you `planning_apps[]` directly, already in the
composite `<gss_code>+<ref>` form.

## 2. Resolve applications

Three ways in, all returning the same `PlanningApplication` shape:

- `getPlanningApplication` (`GET /planning-applications/{gss_code}/{ref}`) — authority code and
  reference as separate path segments.
- `getPlanningApplicationById` (`GET /planning-applications/{app_id}`) — the composite
  `<gss_code>+<ref>` id, e.g. `E07000112+Y18/1381/FH`.
- `getPlanningApplicationByIds` (`POST /planning-applications`) — a batch of composite ids. Marked
  **beta** in LandTech's published docs.

All three can return `404` (`Application not found`), unlike most of the API.

Key fields: `status` and `status_derived`, `decision` and `decision_date`, `date_received`,
`num_dwellings` (with `found_num_dwellings` telling you whether the count was actually extracted),
`classification`, `applicant_name`, `agent_name`, `boundary` geometry, and `related_applications`.
Boolean type flags — `is_full_application`, `is_outline`, `is_reserved_matters`,
`is_discharge_of_conditions`, `is_change_of_use`, `is_listed_building`, `is_tpo`, `is_eia`,
`is_gpdr`, `is_minor`, `is_uncategorised` — let you classify without parsing free text. Prefer them
over string matching on `title`.

Appeal history rides on the same object under the `appeal_*` prefix (`appeal_case_number`,
`appeal_decision`, `appeal_decision_date`, `appeal_inspector_name`, `appeal_type_of_appeal`). A
refused application with a successful appeal is a materially different signal from a plain refusal —
always check `appeal_decision` before concluding a site was blocked.

## 3. Resolve the layers covering the site

A parcel from `getParcel` carries four layer maps: `local_plan_policy`, `development_constraints`,
`development_opportunities` and `shlaa`. Each maps a layer name to `[{id, coverage}]`.

**`coverage` is the point.** It is the 0–1 fraction of the parcel the feature covers. A flood zone at
`coverage: 0.05` clips a corner; at `coverage: 1` it covers the whole site. Never report a constraint
as applying without its coverage ratio.

Hydrate the ids in bulk:

- `getDevelopmentConstraints` (`POST /development_constraints`) — green belt, flood risk zones,
  Article 4 directions, conservation areas, listed buildings, heritage land, protected areas,
  agricultural land classification, national landscape, rights of way, neighbourhood plan areas, and
  airport / HS2 / rail / wharf safeguarding.
- `getDevelopmentOpportunities` (`POST /development_opportunities`) — named regeneration areas, area
  action plans, declassified green belt, brownfield register.
- `getLocalPlanPolicies` (`POST /local_plan_policy`) — local planning authority, settlement
  boundaries, site allocations, industrial locations.
- `getShlaas` (`POST /shlaa`) — Land Availability Assessment sites.
- `getPowers` (`POST /power`) — substations and network infrastructure, for grid-capacity questions.

Use the `Lite` sibling of any of these (`getDevelopmentConstraintsLite`,
`getDevelopmentOpportunitiesLite`, `getLocalPlanPoliciesLite`, `getShlaasLite`, `getPowersLite`) when
you do not need boundary geometry. On multi-layer assessments this is the difference between a
manageable payload and a very large one — default to `Lite` unless you are drawing a map.

## Rules

- Read-only: no idempotency contract, all calls safe to retry.
- No pagination anywhere; bounded result sets only.
- Feature responses are wrapped as `{"data": {"feature": {...}}}`.
- Errors are plain `application/json` with the `Forbidden` schema on `403`; not RFC 9457.
- Planning data is republished from local planning authorities and can lag the authority's own
  register. Report `updated` / `decision_date` when the recency of a determination matters.
