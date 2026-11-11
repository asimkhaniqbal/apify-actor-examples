# Yandex SERP & Knowledge Graph Scraper

Scrape **Yandex** web search across nine regional fronts and get the whole result page, not just ten blue links: organic rankings, sponsored rows, knowledge graph and AI answers, inline images and videos, product tiles, local companies, weather, and related searches. Choose page records for the familiar one-record-per-SERP shape or flat rows for rank tracking, ad intelligence and analytics. No login, API key, or proxy setup.

## What it does

- **Two modes:** search by keyword, or parse Yandex SERP URLs you already have.
- **Nine live fronts:** `yandex.com`, `.by`, `.kz`, `.uz`, `.com.tr`, `.com.ge`, `.az`, `.md`, `.tj`.
- **Rank tracking by region:** 30+ named region presets plus a numeric `regionId` override, 19 interface languages, and correct past-day / two-week / month filters.
- **The full SERP surface:** organic rows with true slot and organic ranks, ads with click URLs and displayed domains, knowledge graph + AI answer text and sources, inline images, inline videos, products, local companies, weather and related searches.
- **Page or row output:** keep today's page records or switch to one flat row per item with `recordType`.
- **Clean data:** empty fields, arrays and objects are omitted at every depth, never sent as `null`.

## Services

| Service | What you get |
|---|---|
| Keyword search | Multi-query Yandex SERP collection with pagination, region, language, domain and recency controls |
| URL parsing | Fetch specific Yandex result pages from a list of URLs, including translated Yandex URLs |
| Rank tracking | Overall slot, organic-only rank, page number, query and region on every result |
| Ad intelligence | Sponsored title, snippet, click URL, displayed domain and sitelinks |
| Entity research | Knowledge graph attributes, AI answer body, cited sources and "see also" entities |
| Commerce research | Product titles, prices, old prices, currencies, shops, delivery options and ratings |
| Local research | Company names, categories, ratings, review counts, addresses, metro and opening hours |
| Media discovery | Inline image thumbnails and originals, video cards with duration, views, publish date and preview |
| SERP context | Weather card, related searches and result-count text |

## How it works

1. The actor builds a Yandex search URL from your query, domain, language, region and time filter.
2. It reads the server-rendered result page, using Yandex's own translated result view when a regional front challenges the request.
3. It parses every result block in page order and assigns ranks across organic and sponsored rows.
4. It removes duplicate URLs and duplicate block items, then stops at `maxItems`, an empty page, or the end of Yandex's 25-page window.
5. It pushes either one page record or one flat row per item. Page records keep the original dataset fields and add the new blocks beside them.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` by keyword or `byUrls` for existing SERP URLs |
| `queries` | array | `["python programming"]` | One or more search queries (mode=`search`) |
| `serpUrls` | array | – | Yandex SERP URLs, one per page (mode=`byUrls`) |
| `domain` | string | `yandex.com` | Regional front: `.com`, `.by`, `.kz`, `.uz`, `.com.tr`, `.com.ge`, `.az`, `.md`, `.tj` |
| `language` | string | `en` | Yandex interface/search language (19 options) |
| `region` | string | – | Named region preset such as `moscow`, `almaty`, `istanbul` |
| `regionId` | integer | – | Numeric Yandex region ID; overrides `region` |
| `maxPages` | integer | `3` | Pages per query, 1–25 |
| `startPage` | integer | `0` | Zero-based first page, 0–24 |
| `maxItems` | integer | `100` | Global unique-item cap, 1–10,000 |
| `timeFilter` | string | `any` | `any`, `day`, `twoWeeks`, `month` |
| `fixTypos` | boolean | `true` | Let Yandex correct misspelled queries |
| `includeAds` | boolean | `true` | Include sponsored rows |
| `includeBlocks` | boolean | `true` | Include knowledge graph, AI answer, images, videos, products, companies, weather |
| `includeRelatedSearches` | boolean | `true` | Include related-search suggestions |
| `outputFormat` | string | `pages` | `pages` for page records, `rows` for flat items |
| `failIfEmpty` | boolean | `false` | Fail the run when nothing was collected |

### Example: rank tracking in Moscow

```json
{
  "mode": "search",
  "queries": ["купить пластиковые окна", "пластиковые окна цена"],
  "domain": "yandex.com",
  "language": "ru",
  "regionId": 213,
  "maxPages": 5,
  "maxItems": 500,
  "timeFilter": "month",
  "outputFormat": "rows"
}
```

### Example: audit a saved SERP URL

```json
{
  "mode": "byUrls",
  "serpUrls": ["https://yandex.com.tr/search/?text=ikinci+el+araba&lr=87&p=0"],
  "includeBlocks": true,
  "includeAds": true,
  "outputFormat": "pages"
}
```

## Output

### Pages mode (`outputFormat: "pages"`)

One record per SERP page. Existing fields are preserved; new fields appear only when the block exists.

- `query`, `page`, `url`, `domain`, `lang`, `scraped_at`
- `total_results_text` — the result-count sentence when Yandex publishes one
- `organic_results[]` — `position` (all slots), `organicPosition`, `resultType`, `title`, `link`, `displayed_link`, `displayedDomain`, `host`, `snippet`, `favicon`, `thumbnail`, `date`, `sitelinks[]`, `rating`, `reviewCount`, `price`, `priceCurrency`, `shopName`, `delivery[]`
- `ads_results[]` — `position`, `title`, `link`, `clickUrl`, `displayed_link`, `displayedDomain`, `url` (only when Yandex exposes the destination), `snippet`, `sitelinks[]`
- `knowledge_graph` — `title`, `description`, `image_url`, `source_url`, `attributes{}`, `label`, `answerText`, `sources[]`, `seeAlso[]`
- `aiAnswer` — `label`, `text`, `sources[]`, `imageUrl` for standalone AI answer blocks
- `inline_images[]` — `title`, `thumbnail_url`, `source_url`, `originalImageUrl`, `sourceHost`, `alt`
- `inline_videos[]` — `title`, `link`, `thumbnail_url`, `duration`, `durationSeconds`, `views`, `publishedAt`, `previewVideoUrl`, `sourceHost`
- `products[]` — `title`, `url`, `price`, `priceCurrency`, `oldPrice`, `rating`, `reviewCount`, `shopName`, `delivery[]`, `imageUrl`
- `companies[]` — `title`, `url`, `rating`, `reviewCount`, `category`, `hours`, `priceLevel`, `address`, `metro`, `distance`, `features[]`, `imageUrl`
- `weather` — `location`, `dateText`, `temperature`, `feelsLike`, `condition`, `wind`, `forecast[]`, `hourly[]`, `url`
- `related_searches[]` — `query`, `link`
- `search_metadata` — `requested_domain`, `served_domain`, `locale_switched`, `source_path`, `captcha_encountered`, `pages_fetched`, `results_found`, `region_id`, `region_name`
- `pagination_info` — `current_page`, `max_pages_set`, `has_next_page`, `results_per_page`, `start_page`, `source_exhausted`

### Rows mode (`outputFormat: "rows"`)

One record per item with `recordType` set to `organic`, `ad`, `knowledgeGraph`, `aiAnswer`, `inlineImage`, `inlineVideo`, `product`, `company`, `weather`, or `relatedSearch`.

Every row carries `query`, `page`, `domain`, `language`, `regionId`, `regionName`, `sourcePath`, `scrapedAt`, and the fields that belong to its type: `position`, `organicPosition`, `resultType`, `blockType`, `title`, `url`, `clickUrl`, `displayedUrl`, `displayedDomain`, `host`, `snippet`, `thumbnail`, `favicon`, `sitelinks[]`, `rating`, `reviewCount`, `price`, `priceCurrency`, `oldPrice`, `shopName`, `delivery[]`, `duration`, `durationSeconds`, `views`, `publishedAt`, `sourceHost`, `imageUrl`, `originalImageUrl`, `sources[]`, `attributes{}`, plus weather and company details when relevant.

## Usage cost

You pay Apify's standard actor start and dataset-item pricing for this run. There is no proxy, API key, or external data subscription to configure, and the actor never asks you to supply credentials.

## FAQ

### Do I need a proxy or a Yandex account?

No. The actor uses a working connection path automatically and needs no login, cookie, or API key.

### Why do some fields disappear from a record?

Empty values are omitted on purpose. If a page has no knowledge graph, no ad, no price, or no video, that field is absent rather than `null` or an empty array.

### Why does `served_domain` differ from the domain I selected?

Yandex sometimes serves an international or regional entry point from another front, and `.com` is commonly served from `.ru`. The requested and actual fronts are both reported in `search_metadata`.

### Why do I get fewer results than I asked for?

Yandex serves at most 10 results per page and 25 pages per query, so one query tops out around 250 results. Split a large job by region, time window, or query variant to collect more.

### Are sponsored results included?

Yes, when `includeAds` is true. Each ad has a click URL and a displayed domain. A final destination URL is included only when Yandex exposes it; the actor never invents one.

### What is the difference between `position` and `organicPosition`?

`position` counts every result slot in page order, including ads. `organicPosition` counts only organic rows. Use the latter for classic rank tracking.

### Can I scrape a URL I already have?

Yes. Use `mode: "byUrls"` and paste one or more Yandex result URLs. The actor reads the query, region, page and time filter from each URL.

### Are old saved tasks still supported?

Yes. Legacy keys such as `searchQueries`, `yandexDomain`, `lang`, `lr`, `noReask`, old time-filter names and the old proxy field are accepted and mapped automatically. They are not shown in the input form anymore.

### Is this affiliated with Yandex?

No. This is an independent third-party actor that reads publicly available Yandex search pages.

## Notes

- Yandex serves 10 organic results per page; page numbers are zero-based in the input and one-based in the output.
- `maxItems` counts unique items across all queries. In pages mode it limits the items inside page records; in rows mode it limits rows.
- `regionId` wins over `region`. Use it when you need an exact Yandex region rather than a named preset.
- `language` should match your query language to keep result text in the original language.
- Empty values are omitted at every depth. Arrays and nested objects are never sent as `null`, `""`, `[]` or `{}`.
- The actor stops on `maxItems`, an empty page, or the end of Yandex's page window. It never stops on a wall-clock timer.

## Limitations

- **Vertical search is separate.** This actor returns the inline blocks that appear inside the web SERP. Dedicated image, video, news, and maps search belong to the separate Yandex vertical actors.
- **Inline blocks are conditional.** Yandex decides per query, front and layout whether to render image, video, product, company or weather cards. The actor emits a block only when the served page contains it; an absent block is not an error and is never synthesized.
- **Ad destinations are sometimes hidden.** Yandex often keeps only a click-tracker URL. `clickUrl` and `displayedDomain` are always honest; a direct destination is added only when Yandex reveals it.
- **Dates are conditional.** Web results rarely expose a publication date, so `date` appears only when the page shows one. Video publish dates are normalized to ISO when they can be parsed.
- **Default region is context-sensitive.** When `region` and `regionId` are empty, Yandex chooses a region from the query and request context, so two runs can rank differently. Set `regionId` (for example `213` for Moscow) when you need reproducible rankings.
- **Deep pages lose relevance.** Pages 11–25 are available, but Yandex's ranking gets less precise beyond the first ten. Use smaller page windows when relevance matters more than volume.
- **Translated result text.** When Yandex serves a regional front through its translated view, the original language is preserved when `language` matches the query language. A language mismatch can produce translated snippets.
- **Localized labels.** Some SERP labels (for example weather conditions and company categories) stay in the language Yandex renders for that front; the actor does not machine-translate them.
