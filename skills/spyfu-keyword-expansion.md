---
name: spyfu-keyword-expansion
description: Expand a seed keyword into a researched keyword set with the SpyFu Keyword Research API — related, question, transactional and co-targeted terms, then bulk metrics for the shortlist.
api: openapi/spyfu-keyword-research-api-api-openapi.yml, openapi/spyfu-seo-research-api-api-openapi.yml
operations:
  - RelatedKeywordsV2Api_GetKeywordExpansions_GET
  - RelatedKeywordsV2Api_GetRelatedKeywords_GET
  - RelatedKeywordsV2Api_GetQuestionKeywords_GET
  - RelatedKeywordsV2Api_GetTransactionKeywords_GET
  - RelatedKeywordsV2Api_GetAlsoBuysAdsForKeywords_GET
  - RelatedKeywordsV2Api_GetAlsoRanksForKeywords_GET
  - RelatedKeywordsV2Api_GetKeywordsByBulkSearch_GET
  - RelatedKeywordsV2Api_GetKeywordsByBulkSearchPost_POST
  - OrganicSerpApi_GetSerpAnalysisKeywords_GET
---

# Expand a seed keyword into a researched set

Base: `https://api.spyfu.com/apis/keyword_api` (and `serp_api` for the last
step). Authenticate with HTTP Basic and set `countryCode` on every call.

## Steps

1. **Expand once, not five times.**
   `RelatedKeywordsV2Api_GetKeywordExpansions_GET` on
   `/v2/related/getKeywordExpansions` performs all five research types through
   the `keywordSearchType` parameter — `PhraseMatch`, `Questions`,
   `AlsoBuysAdsFor`, `AlsoRanksFor`, `Transactions`. $1.00 CPM at 100 r/s,
   which makes it both the cheapest and the fastest way in.
2. **Or call the dedicated endpoints** when you want one behaviour explicitly:
   - `RelatedKeywordsV2Api_GetRelatedKeywords_GET` —
     `/v2/related/getRelatedKeywords`, thematic siblings. **$2.50 CPM and only
     5 r/s — the tightest rate limit and highest CPM in the keyword family.**
   - `RelatedKeywordsV2Api_GetQuestionKeywords_GET` —
     `/v2/related/getQuestionKeywords`, interrogative queries for FAQ and
     content work. $1.00 CPM, 10 r/s.
   - `RelatedKeywordsV2Api_GetTransactionKeywords_GET` —
     `/v2/related/getTransactionKeywords`, high commercial intent. $1.00 CPM.
   - `RelatedKeywordsV2Api_GetAlsoBuysAdsForKeywords_GET` —
     `/v2/related/getAlsoBuysAdsForKeywords`, what top advertisers co-target.
   - `RelatedKeywordsV2Api_GetAlsoRanksForKeywords_GET` —
     `/v2/related/getAlsoRanksForKeywords`, what top-ranking domains also rank
     for.
3. **Score the shortlist in bulk.**
   `RelatedKeywordsV2Api_GetKeywordsByBulkSearch_GET` on
   `/v2/related/getKeywordInformation` returns full metrics for an exact
   keyword list at $0.20 CPM — the cheapest endpoint in the API — and allows
   100 r/s. For lists too long for a query string, use
   `RelatedKeywordsV2Api_GetKeywordsByBulkSearchPost_POST` on the same path;
   it is the only non-GET operation in the whole SpyFu API, and it is still a
   read. Note the POST variant drops to 10 r/s.
4. **Check the battlefield for the winners.**
   `OrganicSerpApi_GetSerpAnalysisKeywords_GET` on
   `/v2/seo/getSerpAnalysisKeywords` returns every domain ranking 1-100 for a
   keyword, with `rank`, `rankChange`, `rankingHomepages` and
   `keywordDifficulty`. $0.50 CPM.

## Rules

- **Expand cheap, then measure cheap.** `getKeywordExpansions` ($1.00) followed
  by `getKeywordInformation` ($0.20) costs a fraction of looping
  `getRelatedKeywords` ($2.50 at 5 r/s) and returns the same metrics.
- **Filter at the source.** `searchVolume.min/.max`,
  `keywordDifficulty.min/.max`, `costPerClick.min/.max`,
  `monthlyCost.min/.max`, `wordCount`, `includeTerms`, `includeAnyTerm`,
  `excludeTerms` and the adult filters (`adultFilter`, `onlyAdultKeywords`) all
  cut the row count before billing.
- **Paging**: `startingRow` + `pageSize`; `totalMatchingResults` in the wrapper
  tells you the full size of the match.
- **Keyword metrics are country-specific.** Search volume, difficulty and CPC
  are measured against that country's Google instance, so a `US` volume and a
  `DE` volume are not comparable numbers.
- **Errors**: `400` on an empty or malformed keyword, an oversized `pageSize`,
  or an unsupported `countryCode`; `401` on bad credentials or a plan without
  API access; `429` with `Retry-After` on rate-limit exhaustion.
