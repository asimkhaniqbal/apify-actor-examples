# Google Search Results Scraper: Organic & SERP Boxes

Run any search term through Google and get back a structured breakdown of the results page — not just the ten blue links, but the AI Overview, knowledge panel, People Also Ask, related searches, local pack, top stories, sports tables, and ads that Google surrounds them with. Point it at a list of queries, pick a country and language, and it returns one structured record per query/page.

## What this actor does

- Searches one or more queries across 48 countries with language targeting, time filters, custom date ranges, and city-level geo-targeting
- Extracts organic results with position, title, displayed URL, snippet, and sitelinks, plus every SERP feature box Google renders for that query
- Applies Google search operators (`site:`, `intitle:`, `intext:`, `inurl:`, `filetype:`, exclusions, exact phrase) directly in the query
- Supports mobile SERP rendering, multi-page pagination per query, and an optional one-row-per-result CSV layout
- Every feature field is always present in a page record; when Google doesn't render that feature for the query, its value is `null` rather than a placeholder

## Output per search result page

Each run produces one record per query (per page, if `maxPagesPerQuery` > 1). Most feature fields are `null` unless Google actually renders that feature for the query — that's normal, not a bug. Paging stops early once `maxResults` organic results are collected or Google shows no next page.

**Always attempted:**
- `searchQuery` — object with `term`, `page`, `countryCode`, `languageCode` and `resultsPerPage` used for this record
- `organicResults` — array of `{position, title, url, displayedUrl, snippet, siteLinks}`, trimmed so the query never exceeds `maxResults` in total. `url` is Google's own redirect link, not the destination; use `displayedUrl` for the real domain
- `aiOverview` — Google's AI-generated summary text and cited sources, when rendered
- `knowledgePanel` — entity info panel for people, places, or organizations
- `peopleAlsoAsk` — array of related questions (question text only; Google loads answers on click, so they aren't in the static page)
- `relatedQueries` — array of `{query, url}` suggested searches
- `localPack` — local business/map results, with `rating`, `reviewCount`, and `address` parsed regardless of UI language
- `topStories` — news carousel results
- `sportsTable` — title, headers, and rows for sports-standings-style queries
- `paidResults` — ad slots, when Google renders any for the query (often `null` even for commercial terms)
- `resultsTotal` — Google's own rough estimate of total matches; expect it to fluctuate between pages of the same query
- `hasNextPage` — whether another results page is available

**Opt-in via `includeExtraBoxes`:**
- `weatherBox` — temperature, condition, precipitation, humidity, wind, plus the box's raw text
- `timeZoneBox` — local time, date, day of week, and timezone abbreviation for the queried place
- `currencyConverterBox` — from/to amount and currency, exchange rate, and as-of timestamp
- `unitConverterBox` — category, from/to amount and unit, and the conversion formula
- `jobsPack` — job listing title, company, location, and source
- `videoPack` — video carousel title, URL, channel, duration, and position
- `shoppingPack` — product title, price, original price, seller, rating, and review count (no product URL — Google's shopping grid has no real link in the static page)
- `dictionaryBox` — headword, phonetic, part of speech, and one entry per sense with definition and example
- `stockBox` — raw text of a stock-price box when Google renders one (no structured price/change fields, unlike the other boxes above)

**Error rows:** a query that fails outright, or returns nothing because Google blocked every attempt, produces `{"type": "error", "query": ..., "reason": ...}` in place of its page records, in both output layouts.

**One-row-per-result CSV mode (`csvFriendlyOutput`):** reshapes output to one organic or paid result per row with `query`, `page`, `type`, `position`, `title`, `url`, `displayedUrl`, `snippet`, `siteLinks` (organic) or `query`, `page`, `type`, `title`, `url` (paid). AI Overview, knowledge panel, and every other box above are not available in this mode — use the default output if you need them.

Two verticals are currently out of scope and never populated: Images (Google serves placeholder image data, not real URLs, in the static page) and Flights.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `queries` | array | – | Search terms to run; each is searched separately. |
| `maxResults` | integer | `100` | Maximum organic results per query (1–200). |
| `countryCode` | string | *(empty = United States)* | Country for the Google domain and searcher-location signal. |
| `restrictResultsCountry` | string | *(empty = any)* | Only return pages targeting this country (Google's `cr` parameter), independent of `countryCode`. |
| `languageCode` | string | *(empty)* | UI language for the Google interface itself (e.g. `en`, `de`). |
| `restrictResultsLanguage` | string | *(empty = any)* | Only return pages written in this language (e.g. `en` or `en,fr` for either). |
| `maxPagesPerQuery` | integer | `1` | Result pages to fetch per query (1–10). |
| `resultsPerPage` | integer | `10` | Requested results per page (10–100, snapped to Google's supported values). Google often still returns ~7–10 organic results per fetch regardless; use `maxPagesPerQuery` for more. |
| `timePeriod` | string | *(empty = any time)* | `hour`, `day`, `week`, `month`, or `year`. |
| `safeSearch` | string | *(empty = account default)* | `active` to filter explicit content, `off` to not filter. |
| `mobileResults` | boolean | `false` | Fetch the mobile SERP instead of desktop. Organic-result extraction on mobile is experimental. |
| `forceAiOverviewCapture` | boolean | `false` | Wait specifically for the AI Overview box to render before reading the page (slower, best-effort). |
| `csvFriendlyOutput` | boolean | `false` | Output one organic/paid result per row instead of one SERP page per row. |
| `includeExtraBoxes` | boolean | `false` | Also extract weather, time zone, currency/unit converter, jobs, videos, shopping, and dictionary boxes. |
| `proxyConfiguration` | object | – | Optional (prefilled with the Google SERP proxy group). Left empty or at the prefilled value, the actor picks suitable proxies itself; any other setting is used as given. |
| `uule` | string | *(empty)* | Google-encoded location string for city-level geo-targeting. Best-effort; country-level targeting is reliable, city-level precision is not guaranteed. |
| `dateFrom` / `dateTo` | string | *(empty)* | Custom date range (`YYYY-MM-DD`), must be set together. Overrides `timePeriod` when both are set. |
| `exactPhrase` | boolean | `false` | Wrap each query in quotes to match it as an exact phrase. |
| `site` | string | *(empty)* | `site:` operator — results are additionally verified against this domain, and non-matching ones are dropped (positions are renumbered). |
| `intitle` / `intext` / `inurl` / `fileType` | string | *(empty)* | Search operators sent to Google as-is. Only when `intitle` and `intext` are both set does the actor drop results that share no words with the query and operators. |
| `excludeTerms` | string | *(empty)* | Words to exclude, comma- or space-separated, applied as `-word`. |

### Example: SEO rank check with a time filter

```json
{
  "queries": ["best running shoes 2026", "best running shoes for flat feet"],
  "countryCode": "us",
  "maxResults": 50,
  "maxPagesPerQuery": 6,
  "timePeriod": "month"
}
```

### Example: domain-restricted research with extra boxes

```json
{
  "queries": ["machine learning tutorial"],
  "site": "github.com",
  "includeExtraBoxes": true,
  "maxPagesPerQuery": 2
}
```

### Example: CSV-friendly export for localized results

```json
{
  "queries": ["Hotels in NYC"],
  "countryCode": "de",
  "languageCode": "de",
  "csvFriendlyOutput": true
}
```

## Use cases

- **SEO monitoring** — track ranking position for target keywords across countries and languages over time
- **Competitor analysis** — see who occupies the organic and local-pack slots for shared keywords
- **Content research** — mine People Also Ask and related queries to plan content around what people actually search
- **Local SEO** — pull local-pack results for location-based service queries
- **SERP feature tracking** — monitor when Google starts or stops showing an AI Overview, knowledge panel, or shopping pack for a query
- **Lead generation** — surface businesses and sites ranking for a target service in a target region

## FAQ

**Is this an official Google product?**
No. This is an independent, third-party actor that reads publicly available Google Search result pages. It isn't affiliated with, endorsed by, or sponsored by Google.

**How fresh is the data?**
Every run fetches live results at the moment it runs; there's no caching.

**How many results can I get per query?**
Up to 200 organic results (`maxResults`) across up to 10 pages (`maxPagesPerQuery`). Since Google usually returns about 7–10 organic results per page, reaching high `maxResults` values also needs a higher `maxPagesPerQuery`; the default of one page gives roughly 10 results.

**Does it work for non-English searches?**
Yes. Set `countryCode` and `languageCode` for localized results. `localPack`'s rating, review count, and address, and `resultsTotal`, are parsed structurally rather than by matching English text, so they populate correctly across many languages and number formats.

**Can I filter by date?**
Yes, two ways: `timePeriod` for a relative range, or `dateFrom`/`dateTo` for an absolute range (set together). When both are set, the custom range takes priority.

**Why is `paidResults` often empty?**
Google renders zero ads for a large share of queries, even commercial ones. Treat it as best-effort; organic, AI Overview, and other fields are unaffected.

**I set `site` — why did I get fewer results than `maxResults`?**
Google only keeps showing matching pages from that domain while it has them; once it runs out it starts slipping in unrelated domains without marking them. This actor checks every result against your domain and drops ones that don't belong, so a smaller but accurate result set is expected once a site's matching pages run out.

**Why is `aiOverview` usually null?**
Google injects its AI Overview via client-side JavaScript, which the actor's default fast fetch doesn't render. Enable `forceAiOverviewCapture` to specifically wait for it — this is slower per query and still best-effort, since Google may genuinely not show an AI Overview for a given query.

**Does the organic `url` field link straight to the destination site?**
No — it's Google's own redirect link, and the token inside it isn't decodable back to a destination. Use `displayedUrl` for the visible domain instead.

**Why do some `intitle`/`intext`/`inurl`/`fileType` results not actually match?**
Only `site` is checked result by result. The other operators are sent to Google as part of the query, and Google doesn't strictly enforce them, so treat them as relevance hints rather than a guarantee. The one extra check: when `intitle` and `intext` are used together, results that share no words at all with your query and operators are dropped.
