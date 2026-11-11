# Pinterest Search Demand & Keyword Insights

Pinterest publishes a public Trends site with ranked search terms, interest-over-time charts, related queries, audience distributions, seasonal moments, editorial stories and shopping signals. This actor reads those first-party public endpoints and returns the published values without estimating search volume or inventing forecasts.

## What it does

- **Seven modes:** trending searches, keyword research, featured topics, editorial stories, shopping categories, shopping metrics and seasonal moments.
- **26 markets:** the regional codes Pinterest exposes, including combined groups such as `GB+IE`, `DE+AT+CH` and `SE+DK+FI+NO`.
- **Four trend lists:** top monthly, top yearly, growing and seasonal searches.
- **Filters:** interest category, lookback window, historical end date, audience age, gender, keyword include/exclude and seasonal moment.
- **Keyword research:** weekly interest history from 30 to 730 days, US forecasts, related terms, audience demographics, representative images and search suggestions.
- **Partitioned volume:** trending lists are split by interest and trend type, so a run can collect thousands of unique terms per market instead of stopping at Pinterest's 100-term list ceiling.
- **Additive output:** every record carries `recordType`, the market and a source URL. Existing featured, editorial and shopping fields keep their original names.

## Services

| Mode | What you get | Markets |
|---|---|---|
| `trendingSearches` | Ranked search terms with relative search index, normalized index, week/month/year change, seasonality score and breakout flag. | 26 |
| `keywordResearch` | One row per keyword with weekly history, latest/peak/average counts, changes, forecast flag and optional related terms, audience, image and suggestions. | 26 (forecasts US only) |
| `featuredTopics` | Pinterest's featured topic cards with descriptions, related searches and example pins. | US, GB+IE, CA |
| `editorialStories` | Editorial trend stories with body copy, regions, keywords by region and example pins. | US, GB+IE, CA |
| `shoppingCategories` | Pinterest's full shopping category taxonomy; optionally ranked `shoppingTrend` rows with related searches and click/save/engagement summaries. | US, GB+IE, CA |
| `shoppingMetrics` | Time series and growth rates for named shopping categories. | US, GB+IE, CA |
| `moments` | Seasonal moments with peak date, takeoff date and peak length. | Available Pinterest markets |

## How it works

The actor calls Pinterest's public Trends JSON endpoints directly. It resolves the latest available date from Pinterest's own `latest_available_date` endpoint, so historical windows are aligned with the week Pinterest actually publishes instead of an estimated date. Trending queries are partitioned by trend type and interest and deduplicated by term and market. Keyword requests are batched in groups of 100 terms, demographics in groups of 100 and images in groups of 30.

No account, cookies, API key or proxy configuration is required. The actor uses only the anonymous public session that Pinterest issues for image lookup. If Pinterest temporarily rate-limits a request, the actor retries with backoff and can switch to an alternate HTTP route automatically. Individual failed partitions are skipped and reported in the run status instead of failing the whole run.

## Input

| Field | Type | Default | Applies to | Notes |
|---|---|---|---|---|
| `mode` | select | `trendingSearches` | all | Seven modes listed above. |
| `region` | select | `US` | all | 26 Pinterest markets. |
| `trendType` | select | `topMonthly` | trendingSearches | `topMonthly`, `topYearly`, `growing` or `seasonal`. |
| `lookbackWindow` | select | `30D` | trendingSearches | `7D`, `30D`, `90D`, `180D` or `365D`. |
| `endDate` | text | – | trendingSearches, keywordResearch, shoppingCategories, shoppingMetrics | `YYYY-MM-DD`; leave empty for Pinterest's latest week. |
| `interest` | select | `all` | trendingSearches | 24 Pinterest interest categories plus all. |
| `moment` | select | `all` | trendingSearches | 13 seasonal moment slugs plus all. |
| `keywordsToInclude` | list | empty | trendingSearches | Keep terms containing at least one word. |
| `keywordsToExclude` | list | empty | trendingSearches | Drop terms containing any word. |
| `ageGroup` | select | `all` | trendingSearches, keywordResearch | Nine Pinterest age buckets. |
| `gender` | select | `all` | trendingSearches, keywordResearch | Female, male, unspecified or all. |
| `keywords` | list | empty | keywordResearch | Empty uses Pinterest's current top terms. |
| `days` | integer | `90` | keywordResearch, shoppingMetrics | 30–730 days; values are clamped to that range. |
| `predictedDays` | integer | `0` | keywordResearch, shoppingMetrics | 0, 14 or 28 days; other values fall back to 0. US forecasts only. |
| `includeRelated` | boolean | `false` | keywordResearch | One extra request per keyword. |
| `includeAudience` | boolean | `false` | keywordResearch | Age and gender distributions. |
| `includeImages` | boolean | `false` | keywordResearch | One representative image per keyword. |
| `includeSuggestions` | boolean | `false` | keywordResearch | Pinterest search suggestions per keyword. |
| `shoppingEvent` | select | `OUTBOUND_CLICK` | shoppingCategories, shoppingMetrics | Outbound clicks or saves. |
| `includeTopCategories` | boolean | `false` | shoppingCategories | Adds ranked `shoppingTrend` rows. |
| `categoryNames` | list | empty | shoppingMetrics | Exact Pinterest category names; empty uses featured level-2 categories. |
| `maxItems` | integer | `100` | all | 1 to 10,000 records. |

Legacy task inputs are still accepted in code: `mode=featured` maps to `featuredTopics`, `mode=editorial` maps to `editorialStories`, and `event` maps to `shoppingEvent`. Removed keys `cookies`, `cookiesJson` and `proxyConfiguration` are ignored with a status warning.

## Output

Every record includes `recordType` and `sourceUrl`. Missing values are omitted at every depth; published zeroes and `false` values are preserved.

### `trend`

`term`, `region`, `trendType`, `rank`, `searchCount`, `normalizedCount`, `weekOverWeekChange`, `monthOverMonthChange`, `yearOverYearChange`, `seasonalityScore`, `isBreakout`, `interestIds`, `endDate`, `trendsUrl`, `pinterestSearchUrl`, `sourceUrl`.

Changes are percentages derived from Pinterest's published ratios. Pinterest caps changes at `100.01`; those rows carry `isBreakout: true`.

### `keyword`

`term`, `region`, `periodStart`, `periodEnd`, `days`, `aggregation`, `timeSeries[]`, `latestCount`, `peakCount`, `peakDate`, `averageCount`, `weekOverWeekChange`, `monthOverMonthChange`, `yearOverYearChange`, `hasPrediction`, optional `relatedTerms[]`, `audience{}`, `imageUrl`, `suggestions[]`, `trendsUrl`, `sourceUrl`.

`latestCount`, `peakCount`, `peakDate`, `averageCount` and `periodStart`/`periodEnd` are computed from the observed history only; Pinterest's forecast points remain in `timeSeries` with their predicted bounds. Each `timeSeries` point can contain `date`, `count`, `normalizedCount`, `predictedLowerBound` and `predictedUpperBound`. Related terms contain `term`, `counts[]` and `hasPrediction`. Audience contains `genderDistribution` and `ageDistribution`. Suggestions contain `query` and `url`.

### `shoppingTrend`

`categoryId`, `name`, `region`, `rank`, `parentCategoryIds[]`, `relatedSearches[]`, `outboundClicks{}`, `saves{}`, `engagement{}`, `endDate`, `sourceUrl`. Each summary contains `total`, `percentGrowth`, `percentRelativeVolume` and `lookback`.

### `moment`

`name`, `region`, `peakDate`, `takeoffDate`, `peakLengthDays`, `sourceUrl`.

### Legacy records

- `trendFeatured`: `trendId`, `name`, `description`, `region`, `isPublished`, `growthPercentMonth`, `interestIds`, `relatedSearches`, `examplePins[]`, `timeSeries[]`, `sourceUrl`.
- `trendEditorial`: `trendId`, `title`, `body`, `trendType`, `region`, `regions`, `interestIds`, `keywordsByRegion`, `startDate`, `endDate`, `isPublished`, `isReadyForTranslation`, `examplePins[]`, `sourceUrl`.
- `shoppingCategory`: `categoryId`, `name`, `level`, `parentCategoryId`, `childCategoryIds[]`, `sourceUrl`.
- `shoppingMetric`: `categoryId`, `name`, `region`, `event`, `dailyValues[]`, `growthRates`, `endDate`, `sourceUrl`.

## Usage cost

Apify usage is charged for compute time, dataset storage and any automatically selected proxy traffic. A typical trending run uses a small number of JSON requests per market; keyword research costs one request per 100 keywords plus one request per enabled enrichment and keyword. Large partitioned sweeps over many markets cost proportionally more. Set `maxItems` and leave optional enrichments off when you only need a quick sample.

## FAQ

### What does the 0–100 index mean?

Pinterest publishes a relative index, not absolute search volume. `searchCount` and `normalizedCount` are Pinterest's own 0–100 scale for the selected market and period. The actor does not convert them into estimated monthly searches.

### Why do forecasts only appear for the US?

Pinterest currently publishes forecast bounds only for the US market. For other markets the actor returns real history and sets or omits `hasPrediction` according to the source; it never invents a forecast.

### Why does one trend list stop at 100 terms?

A single Pinterest list returns at most 100 terms. The actor partitions top monthly, top yearly, growing and seasonal lists by interest category, then removes duplicates, so multiple partitions can return several thousand unique terms for one market.

### Can I request a historical week?

Yes. Set `endDate` to a `YYYY-MM-DD` value. Pinterest returns its nearest available week, and the actor reports the date Pinterest returned in each record. Leave `endDate` empty to use the latest published week.

### Why did my shopping category return no rows?

Category names must match Pinterest's taxonomy exactly, ignoring case. If none match, the actor reports a warning instead of silently substituting unrelated categories. Call `shoppingCategories` first to see valid names.

### Does this actor need a Pinterest account?

No. It reads public Pinterest Trends data without a login, cookies, API key or proxy setup. The anonymous session is used only to request public term images.

## Notes

- Pinterest's combined market codes are preserved exactly as published, for example `GB+IE` and `DE+AT+CH`.
- Editorial stories and shopping taxonomy are published for US, GB+IE and CA only. Other markets return a status warning for those modes.
- Related terms are unavailable for the combined `DE+AT+CH` market; keyword research still returns history and other enrichments there.
- The actor reports partial failures in the run status message and keeps any records that were successfully collected.
- No cookies, login or proxy configuration are required.
- Output is written to a single dataset; use the `recordType` field to separate record families.

## Limitations

- Pinterest does not publish absolute search volume, private account analytics or ad performance data through these public endpoints.
- Forecast coverage is partial even inside the US: some terms have no prediction bounds. The actor reports the source's `hasPrediction` value and omits empty bounds.
- A single Pinterest list is capped at 100 terms. Higher totals depend on partitioning across interests and trend types; highly overlapping partitions can yield fewer unique terms than the sum of their caps.
- Shopping taxonomy and ranked shopping categories are limited to US, GB+IE and CA.
- Related terms are omitted for keywords without Pinterest trend history, because the source returns unrelated popular terms for unknown seeds.
- Search suggestions are not market-scoped by Pinterest; they reflect Pinterest's global search autocomplete.
- Undocumented Pinterest endpoints can change without notice. The actor treats unexpected shapes as empty results and reports them rather than fabricating values.
