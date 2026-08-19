---
name: spyfu-api-usage-and-cost-control
description: Track and control SpyFu API spend — read metered consumption from the Account API and apply the row-billing rules before making an expensive call.
api: openapi/spyfu-account-api-api-openapi.yml
operations:
  - AccountApi_GetApiUsageForMonth_GET
  - AccountApi_GetApiUsageForMonthByDay_GET
  - AccountApi_GetApiUsageForMonthByMethod_GET
---

# Track and control SpyFu API spend

Base: `https://api.spyfu.com/apis/accounts_api`. Authenticate with HTTP Basic
(`SPYFU_API_ID:SECRET_KEY`).

SpyFu bills per **row returned**, not per request, so an agent that pages
without checking can spend real money on a single loop. Read this before running
any of the other SpyFu skills unattended.

## The billing rule

```
cost = (successful_rows_returned / 1000) * endpoint_CPM
```

- Billing runs the 1st to the last day of each calendar month, UTC.
- **Pro + AI** includes a **$40** monthly API credit; **Team / Agency** includes
  **$100**. Anything beyond the credit is charged to the card on file.
- Basic plans have no API access at all — those credentials return `401`.
- CPMs are per endpoint and range from **$0.20** (`getKeywordInformation`,
  competitors) to **$5.00** (all Top Pages endpoints). Full table:
  https://developer.spyfu.com/docs/api-pricing and `finops/spyfu-finops.yml`.

## Steps

1. **Read the month to date.** `AccountApi_GetApiUsageForMonth_GET` on
   `/v2/usage/month/{usageDate}` returns `requestCount`, `rowsReturned`,
   `baseUnits`, `unitsUsed`, `finalCost`, `serviceLevelName` and `isPaid`.
   `finalCost` against the plan credit is the number that matters.
2. **Find the day it went wrong.**
   `AccountApi_GetApiUsageForMonthByDay_GET` on
   `/v2/usage/month/{usageDate}/daily` breaks the month into days.
3. **Find the endpoint responsible.**
   `AccountApi_GetApiUsageForMonthByMethod_GET` on
   `/v2/usage/month/{usageDate}/method` breaks it down by `apiMethod`, with
   `rowsReturned`, `unitsPerRow` and `unitsUsed` per method. `unitsPerRow` is
   where an expensive endpoint gives itself away.
4. **Set a hard stop.** SpyFu added an account-level maximum API spending cap
   with alerts on 2026-05-13. It is configured in the account UI at
   https://www.spyfu.com/account/api, not through the API — set it before
   giving an agent credentials.

## Rules for spending less

- **`pageSize` is a cost lever.** Halving it halves the bill for that call.
- **Check coverage before pulling history.**
  `DomainStatsApi_GetActiveDatesForDomain_GET` is cheap; a full
  `getAllDomainStats` pull on an 18-year domain is not.
- **Prefer the aggregating endpoints.** `getKeywordExpansions` ($1.00) over
  looping `getRelatedKeywords` ($2.50); `getKeywordInformation` ($0.20) for
  bulk metrics; `getCombinedTopCompetitors` ($0.20) over separate PPC and SEO
  competitor calls.
- **Treat Top Pages and Ad History as budget items.** $5.00 and $3.00 CPM
  respectively, and `getTopPages` allows only 2 requests/second.
- **A `429` costs nothing but a `200` always does.** Rate limiting protects
  SpyFu, not your bill. Only row-returning success is billed — which is exactly
  why an unbounded retry loop over a successful expensive endpoint is the
  dangerous pattern, not a throttled one.
- **Poll usage, do not infer it.** There are no billing or quota headers on API
  responses; the Account API is the only runtime view of spend.
