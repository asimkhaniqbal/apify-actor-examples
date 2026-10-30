# Yandex SERP & Knowledge Graph Scraper

Yandex remains the dominant search engine across Russia and several neighboring countries, and its results page carries its own mix of organic listings, ads, knowledge-graph cards and inline media. This actor runs your queries against any of Yandex's 17 regional domains and returns each result block as structured data, page by page.

## What this actor does

- **Runs any number of queries** against a chosen Yandex regional domain, with language, region (`lr`) and time-recency filters
- **Captures every SERP block** — organic results, ads, a knowledge-graph card, inline images, inline videos, and related searches — per page
- **Paginates automatically** up to the number of pages you request per query
- **Falls back to a captcha-free domain automatically** when a regional domain blocks the request, and flags the switch so you know when it happened
- Fields a page doesn't have (such as a knowledge-graph card or inline images) are left out of that record rather than sent empty

Fallback domains (`yandex.com.tr`, `yandex.uz`) render their own country's UI chrome — Turkish or Russian — even when your organic results stay in the language you requested. Every affected record carries `search_metadata.fallback_domain`, `search_metadata.fallback_ui_language`, `search_metadata.locale_switched` and a plain-English `search_metadata.warning`, so you can filter out or review fallback results with `search_metadata.locale_switched == true`.

## Output per result page

One record is created per SERP page per query.

- `query` — the search query
- `page` — page number (1-indexed)
- `url` — the Yandex search URL used
- `domain` — the actual domain used (may differ from the one requested if a fallback occurred)
- `lang` — language code
- `total_results_text` — Yandex's own total-result-count text
- `organic_results` — array of organic results, each with `position`, `title`, `link`, `displayed_link`, `snippet`, `favicon_url`, `date`, `thumbnail` and `sitelinks` (sub-link `{title, link}` pairs)
- `ads_results` — array of sponsored results, each with `position`, `title`, `link`, `displayed_link` and `snippet`
- `knowledge_graph` — a card with `title`, `description`, `image_url`, `source_url` and an `attributes` object, when Yandex shows one for the query
- `inline_images` — array of inline image thumbnails, when Yandex renders an image block
- `inline_videos` — array of inline video cards, when Yandex renders a video block
- `related_searches` — array of `{query, link}` suggested searches
- `search_metadata` — domain, language and fallback status for this page (see fallback note above)
- `pagination_info` — current page, requested max pages, whether a next page exists, and results per page
- `scraped_at` — ISO 8601 timestamp of the scrape

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `searchQueries` | array | – | One or more search terms to look up on Yandex (required) |
| `yandexDomain` | string | `yandex.com` | Regional domain (e.g. `yandex.ru`, `yandex.com.tr`, `yandex.de`) |
| `lang` | string | `en` | UI language code (e.g. `en`, `ru`, `tr`, `de`) |
| `lr` | string | – | Region filter: named shortcut (`ru`, `moscow`, `us`) or numeric Yandex region ID |
| `maxPages` | integer | `3` | Number of SERP pages to scrape per query (1-10) |
| `resultsPerPage` | integer | `10` | Organic results per page (10, 20, 30, or 50) |
| `timeFilter` | string | `any` | Filter by recency: `any`, `day`, `week`, `month`, or `year` |
| `noReask` | boolean | `false` | Disable Yandex's automatic query correction (noreask=1); use for exact-match searches |
| `proxyConfiguration` | object | – | Recommended. Yandex triggers SmartCaptcha on datacenter IPs — residential proxies are prefilled by default |

### Supported domains

| Domain | Region |
|--------|--------|
| `yandex.com` | International |
| `yandex.ru` | Russia |
| `yandex.com.tr` | Turkey |
| `yandex.by` | Belarus |
| `yandex.kz` | Kazakhstan |
| `yandex.uz` | Uzbekistan |
| `yandex.ua` | Ukraine |
| `yandex.fr` | France |
| `yandex.de` | Germany |
| `yandex.com.ge` | Georgia |
| `yandex.am` | Armenia |
| `yandex.az` | Azerbaijan |
| `yandex.md` | Moldova |
| `yandex.tj` | Tajikistan |
| `yandex.lt` | Lithuania |
| `yandex.lv` | Latvia |
| `yandex.ee` | Estonia |

### Example: two queries, international domain

```json
{
    "searchQueries": ["machine learning", "python programming"],
    "yandexDomain": "yandex.com",
    "lang": "en",
    "maxPages": 2,
    "resultsPerPage": 10,
    "timeFilter": "any"
}
```

### Example: Russian region, past week only

```json
{
    "searchQueries": ["новости технологий"],
    "yandexDomain": "yandex.ru",
    "lr": "moscow",
    "timeFilter": "week",
    "maxPages": 1
}
```

### Example record returned

```json
{
    "query": "artificial intelligence",
    "page": 1,
    "url": "https://yandex.com.tr/search/?text=artificial+intelligence&lang=en&numdoc=10",
    "domain": "yandex.com.tr",
    "lang": "en",
    "total_results_text": "182 bin sonuç bulundu",
    "organic_results": [
        {
            "position": 1,
            "title": "Artificial Intelligence - Wikipedia",
            "link": "https://en.wikipedia.org/wiki/Artificial_intelligence",
            "displayed_link": "Wikipedia.org",
            "snippet": "Artificial intelligence is used in astronomy to analyze...",
            "favicon_url": "https://favicon.yandex.net/favicon/en.wikipedia.org",
            "date": "",
            "thumbnail": null,
            "sitelinks": []
        }
    ],
    "ads_results": [],
    "knowledge_graph": null,
    "inline_images": [],
    "inline_videos": [],
    "related_searches": [],
    "search_metadata": {
        "yandex_domain": "yandex.com",
        "domain_description": "International / English",
        "language": "en",
        "results_per_page": 10,
        "max_pages": 2,
        "fallback_domain": "yandex.com.tr"
    },
    "pagination_info": {
        "current_page": 1,
        "max_pages_set": 2,
        "has_next_page": true,
        "results_per_page": 10
    },
    "scraped_at": "2026-03-02T08:32:57.904000+00:00"
}
```

## Use cases

- **SEO rank tracking** — monitor where a site lands on Yandex for target keywords across regional domains
- **Localized market research** — compare organic results and ads between Russia, Turkey and other Yandex markets
- **Knowledge-graph research** — pull Yandex's own entity cards for people, places or topics
- **Content gap analysis** — review top-ranking titles and snippets for a keyword to spot what competitors cover
- **Rich-result monitoring** — track when and where inline images, videos or related-search blocks appear for a query

## FAQ

**Do I need a residential proxy?**
It's recommended rather than mandatory. Yandex's SmartCaptcha frequently blocks regional domains on datacenter IPs; a residential proxy avoids that entirely, while the actor's automatic fallback domain keeps runs working even without one.

**Why did my result's `domain` differ from the one I requested?**
That's the fallback behavior: when your chosen domain is blocked, the actor automatically retries on a captcha-free domain and records the switch in `domain` and `search_metadata`.

**Why are `inline_images` or `inline_videos` sometimes empty?**
Yandex only renders those blocks for certain queries and domains; when a block isn't present on the page, there's nothing for the actor to return for it.

**Does `resultsPerPage` always return exactly that many results?**
Not guaranteed above 10 — some Yandex domains don't honor higher values, so you may see fewer organic results per page than requested on those domains.

**How many pages can I scrape per query?**
Up to 10 per query via `maxPages`, run across as many queries as you list in `searchQueries`.

**Is this affiliated with Yandex?**
No, this is an independent, third-party actor that reads Yandex's public search results. It is not affiliated with or endorsed by Yandex LLC.
