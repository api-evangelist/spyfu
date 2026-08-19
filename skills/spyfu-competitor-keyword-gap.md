---
name: spyfu-competitor-keyword-gap
description: Find the keywords a competitor ranks or advertises on that you do not, using the SpyFu Kombat and SEO comparison endpoints.
api: openapi/spyfu-kombat-api-api-openapi.yml, openapi/spyfu-seo-research-api-api-openapi.yml, openapi/spyfu-competitors-api-api-openapi.yml
operations:
  - CompetitorsApi_GetTopSeoCompetitors_GET
  - CompetitorsApi_GetTopPpcCompetitors_GET
  - KombatApi_GetCompetingSeoKeywords_GET
  - KombatApi_GetCompetingPpcKeywords_GET
  - OrganicSerpApi_GetOrganicOutrankingKeywords_GET
  - OrganicSerpApi_GetKeywordsWhereTheyOutRankYou_GET
  - OrganicSerpApi_GetKeywordsWhereTheyJustSurpassedYou_GET
---

# Find the keyword gap against a competitor

Base: `https://api.spyfu.com/apis/{service}` — `competitors_api`, `keyword_api`,
`serp_api`. Authenticate with HTTP Basic (`SPYFU_API_ID:SECRET_KEY`) and set
`countryCode` consistently on every call.

## Steps

1. **Pick the rivals.** `CompetitorsApi_GetTopSeoCompetitors_GET` on
   `/v2/seo/getTopCompetitors` (or
   `CompetitorsApi_GetTopPpcCompetitors_GET` on `/v2/ppc/getTopCompetitors`)
   returns domains ranked by shared-keyword overlap with `commonTerms`. $0.20
   CPM, 1000 r/s — the cheapest step, so start here rather than guessing.
2. **Intersect and subtract.** `KombatApi_GetCompetingSeoKeywords_GET` on
   `/v2/kombat/getCompetingSeoKeywords` takes `includedDomains` and
   `excludedDomains`: it selects keywords belonging to the included domains,
   then removes any that belong to an excluded domain. Put the competitors in
   `includedDomains` and your own domain in `excludedDomains` to get the pure
   gap. `KombatApi_GetCompetingPpcKeywords_GET` on
   `/v2/kombat/getCompetingPpcKeywords` does the same for paid keywords.
   Both are $1.00 CPM; the SEO one is limited to 8 r/s and the PPC one to
   10 r/s.
3. **Rank the head-to-head.**
   `OrganicSerpApi_GetOrganicOutrankingKeywords_GET` on
   `/v2/seo/getOrganicOutrankingKeywords` compares two domains in one call —
   set `query` to one domain, `compareDomain` to the other, and select the mode
   with `keywordSearchType`. Both sides' rank, rank change, clicks and URL come
   back on the same row, with the comparison domain's values in the `your*`
   fields.
4. **Or use the explicit variants.**
   `OrganicSerpApi_GetKeywordsWhereTheyOutRankYou_GET` on
   `/v2/seo/getWhereTheyOutRankYou` and
   `OrganicSerpApi_GetKeywordsWhereTheyJustSurpassedYou_GET` on
   `/v2/seo/getWhereTheyJustSurpassedYou` cover the same ground with fixed
   semantics. Put the competitor in `query` and your domain in `compareDomain`
   to see where they beat you; swap them to see where you win. Both are
   $2.00 CPM — four times the cost of an ordinary SEO keyword call.

## Rules

- **`query` vs `compareDomain` decides the direction of the answer, and the
  `your*` fields always belong to `compareDomain`.** Getting this backwards
  returns a valid, plausible, wrong result — there is no error to catch it.
- **Filter before you page.** The dotted range parameters
  (`searchVolume.min`, `keywordDifficulty.max`, `costPerClick.min`,
  `rank.max`, `seoClicks.min`) cut the row count, and rows are what you are
  billed for.
- **Paging**: `startingRow` (1-based) + `pageSize`; read `totalMatchingResults`
  from the wrapper to size the job before you run it.
- **Rate limits**: 8 r/s on `getCompetingSeoKeywords`, 10 r/s on the PPC Kombat
  and SEO comparison endpoints, 1000 r/s on competitors. `429` returns
  `Retry-After: 1` and `{"error":"rate_limited",...}`.
- **`countryCode` is a data instance, not a filter.** Comparing a domain's
  `US` rows against a competitor's `DE` rows compares two different SpyFu data
  sets. See https://developer.spyfu.com/reference/country-code-enum.
