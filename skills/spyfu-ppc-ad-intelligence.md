---
name: spyfu-ppc-ad-intelligence
description: Pull a competitor's paid search strategy from the SpyFu API — the keywords they buy, the ad copy they run, and how both changed over time.
api: openapi/spyfu-ad-history-research-api-api-openapi.yml, openapi/spyfu-ppc-research-api-api-openapi.yml
operations:
  - PaidSerpApi_GetPaidSerps_GET
  - PaidSerpApi_GetMostSuccessful_GET
  - PaidSerpApi_GetNewKeywords_GET
  - AdHistoryApi_GetDomainAdHistory_GET
  - AdHistoryApi_GetTermAdHistory_GET
  - AdHistoryApi_GetTermAdHistoryWithStats_GET
---

# Read a competitor's PPC strategy

Base: `https://api.spyfu.com/apis/{service}` — `serp_api`, `keyword_api`,
`cloud_ad_history_api`. Authenticate with HTTP Basic and set `countryCode` on
every call.

## Steps

1. **What are they buying now?** `PaidSerpApi_GetPaidSerps_GET` on
   `/v2/ppc/getPaidSerps` returns the paid results a domain appears in, with ad
   position, ad count, ad copy (`title`, `bodyHtml`), search volume and keyword
   difficulty. $2.00 CPM, 12 r/s.
2. **What works for them?** `PaidSerpApi_GetMostSuccessful_GET` on
   `/v2/ppc/getMostSuccessful` returns the highest-performing paid keywords of
   the past year by volume and competitive strength. $2.00 CPM, 10 r/s.
3. **What are they testing?** `PaidSerpApi_GetNewKeywords_GET` on
   `/v2/ppc/getNewKeywords` returns keywords the domain started advertising on
   for the first time — the clearest signal of a strategy change. $2.00 CPM,
   10 r/s.
4. **Read their ad copy history.** `AdHistoryApi_GetDomainAdHistory_GET` on
   `/v2/domain/getDomainAdHistory` returns historical ad variations for the
   domain across all its keywords: `title`, `body`, `fullUrl`, `position`,
   `adCount`, `isLeaderboardAd`, keyed by `searchDateId`. $3.00 CPM, 10 r/s —
   the second most expensive family in the API.
5. **Flip to the keyword side.** `AdHistoryApi_GetTermAdHistory_GET` on
   `/v2/term/getTermAdHistory` returns every advertiser that ran ads on a
   keyword. `AdHistoryApi_GetTermAdHistoryWithStats_GET` on
   `/v2/term/getTermAdHistoryWithStats` adds per-advertiser rollups —
   `budget`, `coverage`, `totalAdsPurchased`, `percentageLeaderboard` — plus a
   `topAds` block. Both $3.00 CPM.

## Rules

- **Domain-side and term-side endpoints answer different questions.**
  `/v2/domain/*` is "what did this advertiser run"; `/v2/term/*` is "who
  advertised on this keyword". They are not filters of one another.
- **Ad history is billed per ad variation returned, not per domain**, and a
  long-running advertiser has thousands. Constrain with `pageSize` and read
  `totalMatchingResults` before paging. At $3.00 CPM a careless full pull is
  the most expensive mistake available in this API.
- **Time is keyed by `searchDateId`**, an opaque month identifier, not an ISO
  date. Newer date-range filtering exists upstream via `minSearchDateId` /
  `maxSearchDateId` on `getDomainAdHistoryByDate` (shipped 2026-07-15); that
  operation is not in the definition captured here, so confirm against
  https://developer.spyfu.com/reference/adhistoryapi_getdomainadhistorybydate_get
  before relying on it.
- **`adId` and `termId` are returned but accepted by nothing.** No operation
  takes them as input — join on `domain`, `keyword`/`term` and `searchDateId`
  instead.
- **Rate limits**: 10 r/s across all Ad History endpoints, 12 r/s on
  `getPaidSerps`, 10 r/s on the other PPC endpoints. `429` returns
  `Retry-After: 1`.
