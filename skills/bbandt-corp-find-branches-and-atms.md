---
name: truist-find-branches-and-atms
description: Search Truist branch and ATM locations by geography — the one Truist API that needs no customer consent.
api: Truist Branch/ATM Locator
provider: BB&T Corp (Truist)
providerId: bbandt-corp
generated: '2026-09-04'
method: generated
source: openapi/bbandt-corp-retail-locator-openapi.yml
operations:
  - searchForAll
  - searchForBranches
  - searchForATMs
base_url: https://api-sandbox.truist.com/retail
---

# Find Truist branches and ATMs

The Locator is the only Truist API whose data is not customer-specific. It authenticates with
**HTTP Basic**, not with a consented OAuth token, so it is the natural first integration.

## Steps

1. `searchForAll` (`GET /v1/open-banking/locator`) — branches and ATMs together.
2. `searchForBranches` (`GET /v1/open-banking/locator/branches`).
3. `searchForATMs` (`GET /v1/open-banking/locator/atms`).

## Two things to know

- **Only a sandbox server is published.** `retail-locator.yaml` declares exactly one server,
  `https://api-sandbox.truist.com/retail` — unlike every other Truist spec, it names no certification or
  production host. Do not assume `api-secure.truist.com` works; confirm the production base with Truist.
- **This API returns `x-RqUID`, not `x-fapi-interaction-id`.** It is the odd one out in the estate. Log
  whichever header comes back; it is the correlation id for support.
