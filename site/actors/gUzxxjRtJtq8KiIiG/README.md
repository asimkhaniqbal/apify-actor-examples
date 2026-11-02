# Ahrefs Domain Rating, Backlinks & Traffic Lookup

Scrape **AhrefsTop** website profiles — the public pages Ahrefs publishes at `ahrefs.com/websites/<domain>` — for Domain Rating, linking websites, search traffic, traffic value, keyword counts, top keywords, countries, competitors, similarly ranked sites and monthly history. Six modes cover full profiles, lean authority/backlink/traffic subsets, nine competitor cards per domain, and the AhrefsTop global, country and category rankings. No Ahrefs account, cookie or API key is needed.

## What this actor does

- **Six modes:** `domain`, `authority`, `backlinks`, `traffic`, `competitors`, `topWebsites`
- **Full public profile:** Domain Rating, linking websites and their monthly change, site title and description, global/country/category ranks and changes, organic traffic, traffic value, keyword count and the data month
- **Deeper sections:** top 5 keywords, top 5 traffic countries, top 5 competing websites, similarly ranked sites, and monthly traffic/value plus linking-website history (typically 10+ years)
- **Competitor rows:** one flat row per competitor card with description, ranks, traffic, traffic value, pages and Domain Rating
- **Rankings:** the global top 1000 (100 rows per page), country and category top 100 lists, combined country + category lists, and monthly trending up/down
- **Safe inputs and output:** `www` and non-`www` hosts are both tried, duplicate domains are removed, every request is capped by `maxItems`, and empty fields are omitted instead of written as `null`

## Output per website record

Used by `domain`, `authority`, `backlinks` and `traffic`. Each mode returns its own subset.

- `type` — always `ahrefs_website_stats`
- `domain` — the normalised domain or host you requested
- `resolvedDomain` — the exact host AhrefsTop served
- `sourceUrl` — the public Ahrefs page the record came from
- `scrapedAt` — ISO 8601 collection timestamp
- `notFound` — `true` when AhrefsTop has no page for either host form
- `fetchError` — a short failure reason (`HTTP 403`, `HTTP 429`, `timeout`, `network`) when no page could be read; no metrics are invented
- `domainRating` — Domain Rating (0–100)
- `linkingWebsites` / `linkingWebsitesText` — distinct linking websites, numeric and as displayed
- `referringDomainsDelta` / `referringDomainsDeltaDirection` — month-over-month change and `increase`/`decrease`
- `siteTitle`, `siteDescription` — the site name and description shown on the profile
- `descriptionMeta` — the description from the page's structured metadata
- `globalRank` / `globalRankChange` — worldwide traffic rank and its monthly move
- `countryRank`, `countryRankCountry` / `countryRankChange` — country rank, country name and move
- `category`, `categoryRank` / `categoryRankChange` — category name, rank and move
- `organicTraffic`, `organicTrafficText` / `organicTrafficChange` — estimated monthly organic visits, display text and change
- `trafficValueUsd`, `trafficValueText` / `trafficValueChangeUsd` — estimated monthly traffic value in USD and its change
- `organicKeywords` / `organicKeywordsText` — estimated organic keyword count
- `asOf`, `trafficMonth`, `dataMonth` — snapshot month (display), traffic month and machine-readable `YYYY-MM` data month
- `topKeywordCountry`, `topKeywords[]` — country used for the keyword table; each keyword has `keyword`, `volume`, `volumeChange`, `traffic`, `trafficChange`, `position` and `positionChange`
- `topCountries[]` — `country`, `visits`, `visitsChange`, `share` (percent)
- `topCompetitors[]` — `domain`, `traffic`, `trafficChange`, `pages`
- `similarlyRanked[]` — `domain`, `traffic`, `trafficChange`, `rank`, `rankChange`; only when the requested site is actually in the worldwide top 1000
- `trafficHistoryMonthly[]` — `month`, `organicTraffic`, `trafficValueUsd`
- `linkingWebsitesHistory[]` — `month`, `linkingWebsites`

Mode subsets: `authority` returns identity, Domain Rating/linking websites and their history; `backlinks` returns the same link-focused subset; `traffic` returns ranks, traffic, keywords, countries and traffic history; `domain` returns everything.

## Output per competitor record

Used by `competitors`. One row per competitor card, up to 10 per requested domain.

- `type` — always `ahrefs_competitor`
- `sourceDomain` — the domain whose competitor page was read
- `domain` — the competitor domain
- `sourceUrl`, `scrapedAt`
- `notFound` / `fetchError` — only when the source page could not be read
- `siteDescription` — the competitor's description
- `globalRank`, `globalRankChange`
- `countryRank`, `countryRankCountry`, `countryRankChange`
- `category`, `categoryRank`, `categoryRankChange`
- `organicTraffic`, `organicTrafficChange`
- `trafficValueUsd`, `trafficValueChangeUsd`
- `pages`
- `domainRating`

## Output per top-website record

Used by `topWebsites`. One row per ranked or trending website.

- `type` — always `ahrefs_top_website`
- `sourceUrl`, `scrapedAt`
- `rank`, `rankChange`
- `domain`
- `category`
- `organicTraffic`, `organicTrafficText`, `organicTrafficChange`
- `rankingScope` — the requested scope: `global`, the country slug, the category slug, or `country/category` (for example `united-states`, `reference`, `united-states/reference`); on a country page, trending rows are country-level (see Limitations)
- `rankingType` — `ranking`, `trending_up` or `trending_down`
- `rankingPage` — 1–10 for the global ranking, 1 for country/category and trending
- `notFound` / `fetchError` — only when the ranking page could not be read

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `domain` | `domain` (full profile), `authority` (DR + linking sites), `backlinks` (link summary + history), `traffic` (ranks/traffic/keywords/countries), `competitors` (one row per competitor), `topWebsites` (AhrefsTop rankings). Legacy values `full`, `authority_only` and `backlinks_only` are still accepted and mapped |
| `domains` | array | – | Domains or full URLs for every mode except `topWebsites`. `www` and non-`www` are both tried; duplicates are removed |
| `includeHistory` | boolean | `true` | Include monthly traffic/value and linking-website history (mode=domain/authority/backlinks/traffic) |
| `includeKeywords` | boolean | `true` | Include the top 5 keywords (mode=domain/traffic) |
| `includeCountries` | boolean | `true` | Include the top 5 traffic countries (mode=domain/traffic) |
| `includeCompetitors` | boolean | `true` | Include the top 5 competitors on the profile (mode=domain) |
| `maxItems` | integer | `50` | Hard cap on rows across all requested domains or ranking pages (1–1000) |
| `topWebsitesCountry` | string | `worldwide` | Country ranking (mode=topWebsites). `worldwide` uses the global top 1000 |
| `topWebsitesCategory` | string | `all` | Category ranking (mode=topWebsites); combine with a country for that country's category list |
| `topWebsitesRanking` | string | `ranking` | `ranking`, `trending_up` or `trending_down` (mode=topWebsites) |
| `topWebsitesPage` | integer | `1` | Starting global ranking page from 1 to 10 (mode=topWebsites); country/category/trending expose one page |

### Example: full profile with every section

```json
{
  "mode": "domain",
  "domains": ["wikipedia.org", "www.nhs.uk"],
  "maxItems": 2
}
```

### Example: authority and backlink checks only

```json
{
  "mode": "authority",
  "domains": ["stripe.com", "github.com", "openai.com"],
  "includeHistory": true,
  "maxItems": 3
}
```

### Example: nine competitor rows per domain

```json
{
  "mode": "competitors",
  "domains": ["wikipedia.org"],
  "maxItems": 9
}
```

### Example: the US reference ranking, then trending sites

```json
{
  "mode": "topWebsites",
  "topWebsitesCountry": "united-states",
  "topWebsitesCategory": "reference",
  "topWebsitesRanking": "ranking",
  "maxItems": 100
}
```

```json
{
  "mode": "topWebsites",
  "topWebsitesRanking": "trending_up",
  "maxItems": 10
}
```

## Use cases

- **SEO audits** — pull a client's Domain Rating, traffic, traffic value, keyword count and rank movement in one request
- **Link-building prospecting** — compare linking-website counts and trends across a candidate list before outreach
- **Competitor benchmarking** — export nine competitor cards per domain with traffic, traffic value, pages and DR
- **Market research** — download the global top 1000 or a country/category top 100 for lead scoring and segmentation
- **Portfolio monitoring** — schedule repeat runs and track monthly search traffic, traffic value and referring-site history
- **Content and keyword research** — see the exact keywords a site earns the most traffic from, with volume, position and monthly change

## Limitations

- **Ranking caps:** the global ranking exposes ranks 1–1000 (ten pages of 100). Country, category and combined country + category rankings expose up to 100 rows; combined lists can be shorter when the source has fewer matches (for example, United States + Reference has 3). Trending exposes 10 rows per direction.
- **Country trending:** on a country page the trending lists are country-level; the category filter applies only to the ranking table. On the worldwide page, category trending is category-specific.
- **Conditional ranks:** global rank is printed only for sites in the worldwide top 1000 (`>1000` is omitted). Country, category and keyword-count fields appear only when the source publishes them.
- **Stale data on quiet sites:** some low-traffic profiles stop updating. `dataMonth` reports the real latest month rather than implying the data is current.
- **Public profile surface only:** detailed backlink rows, top pages, keyword ideas, keyword difficulty/SERP, broken links, XML sitemaps and AI visibility are not included. Ahrefs' free tools for those require a Cloudflare Turnstile challenge, and the official free API requires an API key.
- **Third-party estimates:** traffic, traffic value and keyword figures are Ahrefs' estimates, not analytics from the site owner.

## FAQ

**Do I need an Ahrefs subscription or API key?** No. The actor reads the same public website profiles and rankings that any visitor can see on AhrefsTop. There is no login, cookie or token.

**What is AhrefsTop?** It is Ahrefs' public ranking site. Opening `ahrefs.com/websites/<domain>` redirects there; the actor reads the destination page directly.

**Why are some fields missing from a record?** Ahrefs only prints some values conditionally. A site outside the global top 1000 has no `globalRank`, a site without a country rank has no `countryRank`, and some profiles have no keyword count. Empty fields are omitted, never filled with `null` or a sentinel.

**Why is `dataMonth` older than this month for some domains?** Ahrefs stops refreshing profiles it considers inactive. The field shows the latest month the source actually publishes, so you can judge freshness instead of assuming the numbers are current.

**How many rows can `topWebsites` return?** Up to 1000 for the global ranking (start at page 1 and use `maxItems` up to 1000), up to 100 for a country, category or combined country + category list (combined lists can be shorter), and up to 10 for trending up or trending down.

**Can I combine a country and a category?** Yes. Set both `topWebsitesCountry` and `topWebsitesCategory`; the actor reads the combined list, which is capped by the source at 100 rows.

**Do `www` and non-`www` domains both work?** Yes. The actor tries the exact host first and falls back to the other form only when the first returns no page. `resolvedDomain` tells you which host was used.

**How fresh is the data?** Ahrefs updates these profiles monthly. `scrapedAt` records when you collected the row, while `asOf`, `trafficMonth` and `dataMonth` describe the source month.

**Is this affiliated with Ahrefs?** No. This is an independent actor that reads publicly available AhrefsTop pages. It is not endorsed by or affiliated with Ahrefs.
