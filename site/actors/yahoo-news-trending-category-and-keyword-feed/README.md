# Yahoo News Trending, Category & Keyword Feed by Region

Yahoo News publishes thousands of headlines every day from wire services, newspapers, broadcasters, and digital publishers. This actor collects them from six Yahoo News editions — US, UK, Canada, Australia, Singapore, and Japan — through three modes: trending top stories, browsable category feeds, or a keyword search. Each article comes with its exact publication time, publisher, author, summary, section, and image.

## What this actor does

- Reads the live Yahoo News section pages as they exist today (the old `/rss/<section>` feeds mostly return 404 or frozen data)
- Follows pagination (`/2/`, `/3/` … for sections; `b=11`, `b=21` … for search) until `maxItems` or the source is exhausted
- Opens each article's detail page to fill exact dates, author, publisher URL, keywords, and image dimensions
- Deduplicates across categories and across multiple search queries
- Falls back to Yahoo's news search or a live RSS feed when a section page is temporarily blocked
- Omits every field an article genuinely lacks — no nulls, empty strings, or placeholders

## Modes

| Mode | What it returns | Pagination |
|---|---|---|
| `getTrending` | The selected edition's front-page top stories | Section pages, up to 25 pages |
| `browseCategory` | One or more category feeds, e.g. Politics + World | Section pages, per-region depth |
| `search` | Yahoo News keyword search results across all Yahoo properties | 10 results per page, up to 50 pages |

`search` queries the whole Yahoo News index rather than a single regional front page, so it can surface articles from any Yahoo edition. The `region` value still labels the run and is written to each record.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getTrending` | `getTrending`, `browseCategory`, or `search` |
| `region` | string | `us` | `us`, `uk`, `ca`, `au`, `sg`, or `jp` |
| `searchQueries` | array | – | Keywords or phrases for `search` mode. Multiple queries are interleaved fairly. |
| `categories` | array | – | Category slugs for `browseCategory`; availability varies by region (below) |
| `maxItems` | integer | `50` | Total articles to return (1–1000) |
| `keyword` | string | – | Keep only articles whose title or summary contains this text |
| `sourceFilter` | string | – | Keep only articles whose publisher name contains this text, e.g. `Reuters` |
| `includeArticleDetails` | boolean | `true` | Open each article page for exact dates/author/publisher URL/keywords. Turn off for faster list-only records. |

### Regions and available categories

| Region | Categories |
|---|---|
| United States (`us`) | Top Stories, US, World, Politics, Science, Weather, Entertainment, Lifestyle, Health, Sports, Finance |
| United Kingdom (`uk`) | Top Stories, UK, World, Politics, Science & Tech, Weather, Finance, Sports |
| Canada (`ca`) | Top Stories, Canada, World, Politics, US, Science & Tech, Weather, Sports, Finance |
| Australia (`au`) | Top Stories, Australia, World, Politics, Environment, Sport, Weather, Finance |
| Singapore (`sg`) | Top Stories, Singapore, World, Politics, Sports, Personal Finance, Weather |
| Japan (`jp`) | Top Picks, Domestic, World, Business, Entertainment, Sports, IT, Science, Local |

A category that does not exist for the chosen region is skipped with a warning; the run continues with the valid ones.

### Examples

Trending stories for the UK:

```json
{ "mode": "getTrending", "region": "uk", "maxItems": 20 }
```

Browse two Canadian categories, paginated to 100 articles:

```json
{
  "mode": "browseCategory",
  "region": "ca",
  "categories": ["canada", "politics"],
  "maxItems": 100
}
```

Search two topics across Yahoo News, only from one publisher:

```json
{
  "mode": "search",
  "region": "us",
  "searchQueries": ["AI regulation", "semiconductor exports"],
  "sourceFilter": "Reuters",
  "maxItems": 30
}
```

## Output per article

| Field | Meaning |
|---|---|
| `title` | Exact headline (from the article page when details are enabled) |
| `description` | Summary/excerpt |
| `link` | Canonical article URL (Yahoo-hosted, or the publisher when Yahoo links out directly) |
| `publishedAt` | Exact publication time in UTC ISO 8601 |
| `updatedAt` | Last update time, when different from publication |
| `author` | Byline, when the article names one |
| `sourceName` | Original publisher, e.g. `Reuters`, `Associated Press`, `BBC` |
| `sourceUrl` | Publisher's homepage |
| `section` | Human-readable section, e.g. `Politics` or `U.S.` |
| `category` | Category slug used by the run; `search` for search-mode records |
| `searchQuery` | The query that produced a search-mode record |
| `yahooProperty` | `News`, `Finance`, or `Sports` when search identifies the Yahoo property |
| `region` | Edition requested (`us`, `uk`, `ca`, `au`, `sg`, `jp`) |
| `imageUrl`, `imageWidth`, `imageHeight` | Lead image and its dimensions |
| `keywords` | Article keywords, when the publisher supplies them |
| `guid` | Stable article identifier |
| `scrapedAt` | When the record was collected |

## Pagination and limits

- Category and trending modes paginate through Yahoo's section pages (about 50 cards per page; page 20 is populated and page 21 is empty, so roughly 1,000 recent articles per large section). `maxItems` stops the run as soon as enough unique articles are collected.
- Search mode follows Yahoo's 10-results-per-page offset (up to 50 pages). Yahoo's search index exposes roughly 500 results per query (offset 501 returns only a couple, offset 1001 returns none), so very broad `maxItems` values can return fewer results than requested; the run reports the actual count.
- Yahoo Japan topic pages paginate with `?page=N` (up to 9 pages per topic).
- Some lanes (Yahoo Sports and Yahoo Finance) expose a single page rather than paginated sections; their natural inventory is smaller.
- `includeArticleDetails=false` keeps the run fast and avoids one request per article, but exact dates, authors, and full summaries require details. List-only search dates come from Yahoo's relative-time labels and are approximate.

## Use cases

- **News monitoring** — track a brand, person, or topic across Yahoo's syndicated network
- **Media research** — see which publishers cover a story and how headlines are framed
- **Regional feeds** — build a US, UK, Canadian, Australian, Singaporean, or Japanese news stream
- **NLP datasets** — collect headlines, summaries, publishers, sections, and timestamps for classification or sentiment work
- **Data journalism** — compare coverage across editions and categories

## Data source

The actor reads Yahoo News's public HTML pages and Yahoo's public news search results. No login, cookie, API key, or proxy configuration is required. It does not bypass paywalls or access controls.

## FAQ

**Is this affiliated with Yahoo?** No. It is an independent third-party actor that reads publicly accessible Yahoo News pages.

**Why did US category results used to be empty?** Yahoo retired the old `news.yahoo.com/rss/<section>` feeds in 2026. The revamp reads the live `www.yahoo.com/news/<section>` pages instead, so categories work again.

**Why does search sometimes return fewer than `maxItems`?** Yahoo's news search index exposes a finite number of results per query. The actor also drops duplicates and non-article cards, so the final count can be below the requested maximum. The status message reports the real count.

**How current is the data?** Section and search pages show whatever Yahoo is serving at run time. With details enabled, `publishedAt` is the exact timestamp from the article page.

**Can I filter by date?** Not as an input, but every record has `publishedAt`, so you can filter downstream.

**Why do a few search results point outside Yahoo?** Yahoo News search occasionally links straight to a publisher instead of a Yahoo-hosted copy. Those records keep the search-card fields; detail enrichment only opens Yahoo-hosted article pages.

**Why are some fields absent on some records?** Yahoo and its publishers do not provide every field for every story (for example, not all articles name an author). Those fields are omitted rather than filled with placeholders.

**Does search respect `region`?** Search queries Yahoo's global news index, which can include any Yahoo edition. The `region` value labels the run; each result keeps its canonical Yahoo URL so you can see which edition hosted it.
