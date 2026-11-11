# SimilarWeb Traffic & AI Chatbot Analytics Scraper

Get a complete traffic picture for any list of websites: estimated visits, global / country / category rank, engagement, three-month history, audience geography and demographics, competitors, technologies, and the share of visits arriving from AI chatbots such as ChatGPT, Gemini, Claude and Perplexity. Add ranked top-website lists, trending movers, and similar-site discovery for market mapping.

## What this actor does

- **Seven ways to look at a market:** a full website + AI report, deep website analytics, a fast traffic snapshot, AI chatbot traffic, similar sites, top-website rankings, and trending websites.
- **Bulk domains:** one clean row per domain for full, deep, snapshot and AI modes; up to 5,000 domains in the cheap modes and 1,000 deep reports per run.
- **AI referral intelligence:** total AI visits, AI share of referral traffic, which chatbots send the visits, a three-month chatbot mix, and SimilarWeb's example prompts.
- **Audience and competition:** top countries, age and gender split, interests, technologies, up to 10 competitors plus 20 similar sites, and the public referral / ad / social entries.
- **Rankings without a domain list:** the top 50 websites worldwide, in a country or in a category — plus 18 global trending movers.
- Empty fields are omitted. A value is never invented when SimilarWeb does not publish it.

## Modes

| Mode | What you get | Good for |
|---|---|---|
| `fullReport` | Deep website analytics + AI chatbot traffic for each domain | One complete profile per competitor |
| `website` | Deep website analytics only | Traffic, ranks, audience and competitors |
| `traffic` | Fast traffic snapshot from SimilarWeb's similar-sites data + 20 similar sites; a deep report fills a domain the fast source misses | Large domain lists on a budget |
| `aiTraffic` | AI chatbot visits, share, chatbot mix, history and prompts | AI-search visibility tracking |
| `similarSites` | One row per similar site for each seed domain | Competitor discovery |
| `topWebsites` | Ranked websites by category and/or country | Market mapping |
| `trending` | 18 global rising and falling websites | Spotting momentum |

## Output per domain row (`recordType: "domain"`)

- `domain`, `title`, `description` — the site and its public description
- `category`, `categoryId`, `categoryName` — SimilarWeb category and subcategory
- `totalVisits`, `totalVisitsSource` — latest monthly estimate and which SimilarWeb product supplied it
- `bounceRate`, `pagesPerVisit`, `timeOnSite` — engagement; time in seconds
- `engagementMonth`, `visitsChangePercent` — month the engagement figures cover and month-over-month change
- `estimatedMonthlyVisits`, `trafficHistory` — three-month visit history as JSON
- `globalRank`, `countryRank`, `countryRankCountry`, `categoryRank` — current ranks
- `globalRankChange`, `countryRankChange`, `categoryRankChange` — movement since the previous month
- `globalRankHistory`, `countryRankHistory`, `categoryRankHistory` — three-month rank history as JSON
- `topTrafficSource`, `topTrafficSourceShare` — the #1 marketing channel and its share
- `trafficSources` — the public channel ranking as JSON
- `trafficSourceDirect`, `trafficSourceSearch`, `trafficSourceSocial`, `trafficSourceReferrals`, `trafficSourceMail`, `trafficSourcePaidReferrals` — legacy share fields; only the channel that is #1 is filled
- `topCountries` — top five countries with share, as JSON
- `topKeywords` — top five organic keywords with name and CPC, as JSON
- `ageDistribution`, `genderDistribution` — audience demographics as JSON
- `interests` — websites, topics and categories the audience also visits
- `technologies` — top technology per category and totals
- `competitors` — up to 10 similar sites with affinity and category rank
- `incomingReferrals`, `outgoingReferrals`, `ads`, `socialNetworks` — public unlocked entries; SimilarWeb hides some referral/ad hostnames behind a login
- `companyName`, `companyYearFounded`, `companyEmployeesMin`, `companyRevenueMin`, `companyHeadquarters`, `companyCategoryId`, `companyParentDomain` — company profile when SimilarWeb has it
- `aiTotalVisits`, `aiTrafficShareOfReferrals`, `aiTrafficAvailable` — AI-referred visits, share and availability
- `aiTrafficSplit`, `aiChatbotDistribution`, `aiTrafficHistory`, `aiBoundary` — chatbot mix and three-month history
- `topPrompts` — SimilarWeb's three example prompts, only when AI visits exist
- `isSmall` — true when SimilarWeb has no traffic estimate for the domain
- `screenshot`, `previewMobileUrl`, `faviconUrl` — public image URLs
- `websiteAnalyticsAvailable` — full reports only: whether the deep profile was available
- `similarSites` — full reports only when requested: 20 similar sites
- `dataSources`, `snapshotDate`, `scrapeTimestamp`, `recordType`

## Output per similar site (`recordType: "similarSite"`)

`seedDomain`, `domain`, `description`, `category`, `categoryId`, `categoryName`, `topCountryRank`, `totalVisits`, `similarityRank`, `similarityGrade`, `thumbnailUrl`, `source`, `scrapeTimestamp`, `recordType`.

## Output per ranked website (`recordType: "topWebsite"`)

`rank`, `domain`, `country`, `category`, `categoryId`, `categoryName`, `bounceRate`, `pagesPerVisit`, `timeOnSite`, `rankChange`, `isNewRank`, `faviconUrl`, `snapshotDate`, `source`, `scrapeTimestamp`, `recordType`.

## Output per trending website (`recordType: "trendingWebsite"`)

`trend` (`increasing`, `decreasing`, `in` or `out`), `rank`, `rankChange`, `isTotallyNew`, `domain`, `faviconUrl`, `snapshotDate`, `source`, `scrapeTimestamp`, `recordType`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `fullReport` | `fullReport` / `website` / `traffic` / `aiTraffic` / `similarSites` / `topWebsites` / `trending` |
| `domains` | array | – | Domains or URLs for the domain modes. Duplicates and invalid entries are skipped. |
| `maxItems` | integer | `100` | Maximum domains or rows to process (1–5,000). Deep reports are capped at 1,000; top lists at 50. |
| `includePrompts` | boolean | `true` | Add SimilarWeb's three example prompts when a domain has AI traffic. |
| `includeSimilarSites` | boolean | `false` | Add 20 similar sites to each full report. |
| `category` | string | `all` | Category or subcategory for `topWebsites`. |
| `country` | string | `worldwide` | Country for `topWebsites`. |
| `includeRankChange` | boolean | `true` | Include rank movement and new-entry fields in ranking modes. |

## Example: compare AI traffic for three competitors

```json
{
  "mode": "aiTraffic",
  "domains": ["openai.com", "anthropic.com", "perplexity.ai"],
  "includePrompts": true
}
```

## Example: full reports for a shortlist

```json
{
  "mode": "fullReport",
  "domains": ["github.com", "gitlab.com", "bitbucket.org"],
  "includeSimilarSites": true
}
```

## Example: a large traffic snapshot

```json
{
  "mode": "traffic",
  "domains": ["shopify.com", "woocommerce.com", "bigcommerce.com", "squarespace.com"],
  "maxItems": 1000
}
```

## Example: top e-commerce websites in the United States

```json
{
  "mode": "topWebsites",
  "category": "e-commerce-and-shopping",
  "country": "united-states",
  "maxItems": 50
}
```

## Use cases

- **AI-search visibility:** track which competitors already receive traffic from ChatGPT, Gemini, Claude or Perplexity, and which prompts send it.
- **Competitive benchmarking:** line up visits, rank, engagement, geography and audience for a shortlist of domains.
- **SEO and content research:** use the public keyword names, CPC and top countries to prioritise markets.
- **Market mapping:** pull the top 50 websites in a category or country and the 18 trending movers.
- **Competitor discovery:** turn one seed domain into 20 similar sites with traffic and category rank.
- **Due diligence:** check a site's three-month trend, audience and referral mix before a partnership.

## Limits worth knowing

- SimilarWeb publishes only the **#1 marketing channel share**. The other legacy `trafficSource*` fields appear only when that channel is #1.
- Keyword **search volume and estimated value are not public**; `topKeywords` contains name and CPC only.
- Visit history is **three months**.
- Deep profiles expose the top **5 countries**, top **10 competitors**, and top **3 chatbots** in the distribution (the full chatbot ranking is in `aiTrafficSplit`).
- AI prompts are **three examples** and are shown only when the domain has AI-referred visits.
- Deep reports are capped at **1,000 domains per run**; use `traffic` for larger lists. `traffic` falls back to one deep report for a domain when the fast snapshots have no data.
- Top-website lists are **50 rows**; a country plus category is capped at **5 rows**. For broader coverage, query the country list and the category list separately, or query subcategories. Trending is global only.
- `traffic` uses SimilarWeb's similar-sites product, so its `totalVisits` can differ from a full report; `totalVisitsSource` tells you which source was used.

## FAQ

### Do I need a SimilarWeb account or API key?

No. The actor reads SimilarWeb's publicly available website data and does not require a login, subscription or API key.

### What exactly is "AI traffic"?

It is traffic that arrives at the website **from** AI chatbots — ChatGPT, Gemini, Claude, Perplexity, Copilot, Grok and DeepSeek — when someone clicks a link or citation in an AI answer. It is not traffic to the chatbots themselves.

### Why is only one traffic-source share filled?

SimilarWeb's public report exposes the leading channel's share and the ranking of the other channels, but not their percentages. The actor fills the leading channel and leaves the others out rather than writing zeros.

### Why is there no keyword search volume?

SimilarWeb no longer publishes keyword volume or estimated value on the public report. The actor returns the keyword names and the public CPC.

### Why do deep reports stop at 1,000 domains, and why is one occasionally missing?

SimilarWeb meters deep website reports per connection. The actor rotates connections automatically; 1,000 is the practical per-run cap, and `traffic` is designed for larger lists. If a deep report is still limited, `fullReport` falls back to AI traffic plus the fast snapshot; `website` mode skips that domain rather than returning a zeroed row.

### Why does `totalVisits` differ between a full report and a traffic snapshot?

They come from two SimilarWeb data products with different snapshots. The `totalVisitsSource` field says whether the number came from the website report or the similar-sites product.

### Why do ranked websites not show total visits?

The public ranking list publishes rank, engagement and category, but not absolute visits. Use a domain mode for a specific site's visits.

### How fresh is the data?

Each row carries `snapshotDate`, usually the first day of the latest published month. `engagementMonth` says which month the engagement figures cover.

### Do I need to set up a proxy?

No. The actor chooses a working connection automatically. Just fill in the input and run it.

### Is this affiliated with SimilarWeb?

No. This is an independent, third-party actor that reads publicly available SimilarWeb data; it is not affiliated with or endorsed by SimilarWeb.
