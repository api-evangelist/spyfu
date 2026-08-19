---
name: spyfu-domain-competitive-audit
description: Audit a domain's search presence with the SpyFu API — historical SEO/PPC performance, current organic visibility, top competitors, and the pages driving traffic.
api: openapi/spyfu-domain-stats-api-api-openapi.yml, openapi/spyfu-seo-research-api-api-openapi.yml, openapi/spyfu-competitors-api-api-openapi.yml
operations:
  - DomainStatsApi_GetLatestDomainStats_GET
  - DomainStatsApi_GetAllDomainStats_GET
  - DomainStatsApi_GetActiveDatesForDomain_GET
  - OrganicSerpApi_GetLiveSeoStats_GET
  - OrganicSerpApi_GetMostValuableKeywords_GET
  - TopPagesApi_GetTopPages_GET
  - CompetitorsApi_GetCombinedTopCompetitors_GET
---

# Audit a domain's search presence

Base: `https://api.spyfu.com/apis/{service}` — `domain_stats_api`, `serp_api`, `competitors_api`.

## Authenticate

Every call needs credentials from https://www.spyfu.com/account/api. Prefer HTTP Basic:

```
Authorization: Basic base64(SPYFU_API_ID:SECRET_KEY)
```

An `api_key` query parameter also works but puts the account Secret Key into
logs and referrers — do not use it in shared or logged environments. A
timestamped HMAC `Authentication` header is the strongest option.

Set `countryCode` on every call. It defaults to `US` and selects a distinct data
instance (`DE` means google.de), so mixing countries across steps produces rows
that cannot be compared.

## Steps

1. **Confirm coverage.** `DomainStatsApi_GetActiveDatesForDomain_GET` on
   `/v2/getActiveDatesForDomain?domain={domain}` returns the months SpyFu has
   data for. Do this first — an empty calendar means every later step returns
   nothing, and you will have paid for the rows anyway.
2. **Get the current position.** `DomainStatsApi_GetLatestDomainStats_GET` on
   `/v2/getLatestDomainStats` for the most recent month: organic rank, monthly
   paid clicks, ad rank, budget, organic value, ads purchased, strength.
3. **Get the trend.** `DomainStatsApi_GetAllDomainStats_GET` on
   `/v2/getAllDomainStats` returns one row per month of history. This is a
   `Multiple` endpoint at $0.50 CPM — a domain with 18 years of history is
   roughly 216 rows, so read `resultCount` and `totalMatchingResults` before
   assuming you got everything.
4. **Read live organic visibility.** `OrganicSerpApi_GetLiveSeoStats_GET` on
   `/v2/seo/getLiveSeoStats` returns continuously-refreshed totals (organic
   results, monthly organic clicks, click value, search volume). Accepts a
   domain, subdomain, path or full URL.
5. **Find the money keywords.** `OrganicSerpApi_GetMostValuableKeywords_GET` on
   `/v2/seo/getMostValuableKeywords`. Page with `startingRow` and `pageSize`,
   sort with `sortBy` / `sortOrder`, and narrow with the dotted range filters
   (`searchVolume.min`, `keywordDifficulty.max`, `seoClicks.min`).
6. **Find the pages behind them.** `TopPagesApi_GetTopPages_GET` on
   `/v2/seo/getTopPages` with `searchType=MostTraffic`. This is the most
   expensive endpoint in the API at $5.00 CPM and the tightest rate limit at
   2 requests/second — request a small `pageSize` and do not loop it.
7. **Name the competitors.** `CompetitorsApi_GetCombinedTopCompetitors_GET` on
   `/v2/combined/getCombinedTopCompetitors` returns PPC and SEO rivals ranked by
   shared-keyword overlap, at $0.20 CPM. Feed the top domains into
   `spyfu-competitor-keyword-gap`.

## Rules

- **Paging is offset-based**: `startingRow` (1-based) and `pageSize`. The
  response wrapper carries `resultCount` (this page) and `totalMatchingResults`
  (everything matching). Plan paging from `totalMatchingResults`.
- **Every row costs money.** Billing is `(successful rows / 1000) x endpoint
  CPM`, so `pageSize` is a cost lever, not just a convenience. CPMs on this flow
  range from $0.20 (competitors) to $5.00 (top pages). See
  `finops/spyfu-finops.yml`.
- **Rate limits are per endpoint on a rolling 1-second window** and vary widely:
  1000 r/s for domain stats and competitors, 10 r/s for most SEO endpoints,
  2 r/s for `getTopPages`. On exhaustion you get `429` with `Retry-After`
  (typically `1`) and a body of `{"error":"rate_limited","message":"..."}`.
  There are no `RateLimit-*` headers, so you cannot see how close you are —
  cap concurrency per endpoint instead of reacting.
- **Errors**: `400` on a malformed domain, an oversized `pageSize` or an
  unsupported `countryCode`; `401` on bad credentials or a plan without API
  access (API access requires Pro + AI or Team/Agency); `500` on server error.
  None of these declare a response schema. All operations are reads, so retry
  with backoff is always safe. See `errors/spyfu-problem-types.yml`.
