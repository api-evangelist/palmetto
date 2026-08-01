---
name: Model a home's energy with Palmetto Energy Intelligence
description: >-
  Estimate a US home's energy consumption, production, cost and emissions -
  including baseline vs hypothetical upgrade scenarios - using the Palmetto
  Energy Intelligence building-energy-modeling endpoint.
api: openapi/palmetto-energy-intelligence-openapi.json
operations:
  - health_api_v0_health_get
  - calculate_api_v0_bem_calculate_post
---

# Model a home's energy with Palmetto Energy Intelligence

Palmetto Energy Intelligence provides physics-based building energy modeling and
solar simulation for any US home, down to hourly granularity and disaggregated
to end uses.

## Auth
- Base URL: `https://ei.palmetto.com`
- Send your API key in the `X-API-Key` header on every request.
- Sign up for a free sandbox key (up to 500 calls/month) at
  https://ei.docs.palmetto.com/docs/getting-started

## Steps

1. **Check service health** (optional) with `health_api_v0_health_get`:
   `GET /api/v0/health` — expect HTTP 200 before running calculations.

2. **Run a calculation** with `calculate_api_v0_bem_calculate_post`:
   `POST /api/v0/bem/calculate`. Provide a JSON body containing:
   - `location` — `{ latitude, longitude }` of the home (required).
   - `parameters` — the window and shape of results: `from_datetime`,
     `to_datetime`, `group_by` (e.g. `month`), `interval_format`,
     `clip_by`, and the `variables` you want (e.g. `consumption.electricity`).
   - `consumption` / `production` — optional actuals plus baseline vs
     `hypothetical` building attributes to model upgrade scenarios.
   - `storage` / `costs` — optional battery and rate/cost inputs.

3. **Read the response**: `CalculateResponse` returns `version`, `meta` and a
   `data` payload with the requested variables at the requested granularity.

## Rules
- Errors return HTTP 422 with a FastAPI validation envelope
  `{ "detail": [ { "loc": [...], "msg": "...", "type": "..." } ] }` — read
  `detail` to find the offending field. See `errors/palmetto-problem-types.yml`.
- There is no idempotency-key contract; calculations are pure and safe to retry.
- To model an upgrade (solar, HVAC, electrification), keep the `baseline`
  building attributes and vary the `hypothetical` attributes in the same call.
