# Apartment Therapy Article Scraper

Collect **Apartment Therapy** articles about decorating, DIY, organizing, house tours, kitchen and bathroom makeovers, small-space living, shopping guides and home technology. Pull the newest posts, search by keyword across recent and historical content, or browse a topic. Every record is captured cookielessly — no login, API key or proxy setup.

## Modes

| Mode | What it returns |
|---|---|
| `getLatest` | Newest articles from the live feed, continuing into historical snapshots when `maxItems` is larger than the live feed |
| `searchArticles` | Keyword matches from the live feed, historical snapshots and the public news index |
| `browseCategory` | Articles carrying a selected topic tag, including historical snapshots |

`feed` selects the main home-and-design feed, the tech feed, or both. `authorFilter`, `dateFrom` and `dateTo` narrow any mode. `maxPages` controls how deep the historical scan goes, and `maxItems` caps the result.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `getLatest` | `getLatest`, `searchArticles` or `browseCategory` |
| `query` | string | – | Keywords for `searchArticles`; prefill is `small apartment ideas` |
| `category` | select | `main` | Topic for `browseCategory` (see list below) |
| `authorFilter` | string | – | Partial, case-insensitive author name |
| `feed` | select | `main` | `main`, `tech` or `all` |
| `sortBy` | select | `relevance` | `relevance`, `newest` or `oldest` |
| `dateFrom` | string | – | Earliest publish date, `YYYY-MM-DD` |
| `dateTo` | string | – | Latest publish date, `YYYY-MM-DD` |
| `maxItems` | integer | `50` | Maximum records to return (1–200) |
| `maxPages` | integer | `2` | Historical snapshots to scan (1–20; 1 = live feed only) |

Categories: `main`, `decorating`, `organizing`, `diy`, `shopping`, `small-spaces`, `kitchen`, `bathroom`, `bedroom`, `living-room`, `house-tours`, `before-and-after`, `home-projects`, `cleaning`, `plants`, `real-estate`, `money`, `life`, `tech`, `news`.

### Examples

Newest articles:

```json
{"mode": "getLatest", "maxItems": 25}
```

Keyword search across the archive:

```json
{"mode": "searchArticles", "query": "small kitchen storage", "maxItems": 40, "maxPages": 6}
```

Browse a topic with an author and date filter:

```json
{"mode": "browseCategory", "category": "before-and-after", "authorFilter": "sarah", "dateFrom": "2026-01-01", "maxItems": 30, "maxPages": 8}
```

Deeper history for the latest timeline:

```json
{"mode": "getLatest", "maxItems": 150, "maxPages": 10}
```

## Output

| Field | Type | Description |
|---|---|---|
| `title` | string | Article headline |
| `articleUrl` | string | Canonical article URL on apartmenttherapy.com |
| `sourceUrl` | string | Same canonical URL, provided as the standard source field |
| `articleId` | string | Numeric article ID from the URL |
| `slug` | string | URL slug without the numeric ID |
| `author` | string | Author byline |
| `publishDate` | string | Publication date, ISO 8601 UTC |
| `category` | string | Primary topic as published |
| `categorySlug` | string | Normalised topic slug |
| `tags` | array | Public topic tags |
| `summary` | string | Short article description |
| `imageUrl` | string | Lead image URL |
| `feed` | string | `main`, `tech` or `web` |
| `dataSource` | string | `live`, `archive` or `webSearch` |
| `isSponsored` | boolean | Present when the article is marked sponsored |
| `hasVideo` | boolean | Present when the article carries the video tag |
| `recordType` | string | Always `article` |
| `scrapedAt` | string | Capture time, ISO 8601 UTC |

Empty fields are omitted, never emitted as `null`.

## Data sources and limits

- **Live feed** — Apartment Therapy's public RSS feeds carry the newest 20 main-feed posts and 20 tech posts, including title, author, date, categories, summary and lead image.
- **Historical snapshots** — the public Internet Archive keeps snapshots of the main feed. `maxPages` scans them newest-first for older complete records. Coverage depends on snapshot availability, so very deep history is not guaranteed.
- **News index** — keyword search supplements the feed and snapshots with results from Google News' public Apartment Therapy index. These records carry title, date and canonical URL; author, summary, category and image are omitted when the index does not provide them.
- **WordPress REST API** — `/wp-json/wp/v2/...` returns `403 Access denied` to cookieless server clients (direct, datacenter proxy and residential proxy all tested). The actor uses the public feeds and index instead.
- Category browsing uses the real topic tags carried by feed and snapshot records. A category that has no recent or snapshotted articles returns no records; raise `maxPages` to search deeper.

## FAQ

**How many articles can one run return?** `maxItems` accepts 1–200. The live feed covers the newest 20 per feed; historical snapshots and the news index supply older records.

**Why do some search results lack an author or image?** Search-index records only expose title, publish date and URL. Complete records come from the live feed or historical snapshots.

**Can I combine a keyword with a category?** Yes. `searchArticles` honours `category` and `authorFilter` for feed and snapshot records. When either filter is set, search-index records are skipped because they cannot prove the match.

**How do I get the whole archive?** There is no complete public archive endpoint. Use larger `maxPages` values and partition with `category`, `authorFilter`, `dateFrom` and `dateTo` to reach more records.

**Is this affiliated with Apartment Therapy?** No. This is an independent actor that reads publicly published content and is not affiliated with or endorsed by Apartment Therapy.
