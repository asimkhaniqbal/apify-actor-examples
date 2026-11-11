# Naver SERP Harvester: Web, Blog, Cafe, News, Image & Video

Search **Naver** and collect the exact results a Korean reader sees: web documents, blog posts, cafe posts, news articles, images, videos, integrated result blocks and autocomplete suggestions. Every row carries the query that produced it, its position, the result link, a clean title and a normalized publish date whenever Naver prints one, plus a vertical-specific set of fields. No login, cookies or proxy setup is required.

## What this actor does

- **Eight modes:** `webSearch`, `blogSearch`, `cafeSearch`, `newsSearch`, `imageSearch`, `videoSearch`, `integratedSearch`, `autocomplete`
- **Batch queries:** pass one or many queries; duplicates and blank entries are removed, and the same result found twice is emitted once
- **Rank and provenance:** every row has `query`, `resultType`, `rank`, `page` and the search page it came from
- **Filters:** relevance or newest-first sorting, date windows, paid-result control, and optional full blog/news content
- **Clean output:** exact destination URLs, normalized dates with a precision label, thumbnails, image dimensions, video duration and views, place details, and AI-briefing text and sources
- **Empty fields are omitted** — you will not see nulls, empty strings or empty arrays in the dataset

## Output per result

Every normal row:

- `recordType` — `searchResult`; failed queries produce `error`
- `resultType` — `web`, `blog`, `cafe`, `news`, `image`, `video`, `place`, `aiBriefing`, `ad` or `suggestion`
- `query`, `mode`, `rank`, `page`
- `title`, `url`, `resultUrl` (same value), `domain`, `sourceName`, `snippet`
- `thumbnailUrl`
- `date` — the raw date text Naver prints
- `publishedAt` — ISO 8601 timestamp normalized to Korea time
- `publishedAtPrecision` — `exact`, `date` or `relative`
- `publishedAtRaw` — the date token used for normalization
- `serpUrl`, `sourceUrl` — the search page that produced the row
- `scrapedAt` — ISO 8601 collection timestamp

Mode-specific additions:

- **Web:** `isAiSource` when the result is flagged as an AI source
- **Blog:** `blogId`, `logNo`, `isAd` for paid placements
- **Cafe:** `cafeName`, `cafeUrl`, `articleId`, `tags[]`, `commentCount`
- **News:** `naverNewsUrl`, `pressUrl`, `officeId`, `articleId`, `section`, `sectionId`, `isRelated` for cluster articles
- **Image:** `imageId` (also `imgId`), `imageUrl`, `originalUrl`, `width`, `height`, `section`, `author`
- **Video:** `duration`, `durationSeconds`, `views`, `videoId`, `platform`, `profileUrl`
- **Place:** `placeId`, `category`, `rating`, `reviewCount`
- **AI briefing:** `aiBriefing` with `text`, `generatedAt` and `sources[]` (each with title, URL, platform and publish time)
- **Autocomplete:** `suggestion`, plus `answer` when Naver returns an answer box
- **Full content:** `content`, `contentLength`, `images[]` when `includeContent` is on and the source exposes a body

A failed query adds one error row with `recordType: "error"`, `errorType`, `message`, `sourceUrl`, `scrapedAt` and `query`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `webSearch` | `webSearch` / `blogSearch` / `cafeSearch` / `newsSearch` / `imageSearch` / `videoSearch` / `integratedSearch` / `autocomplete` |
| `searchQueries` | array | `["비비바"]` | One or more Korean or any-language queries (legacy `searchQuery` and `query` keys are still accepted) |
| `maxItems` | integer | `100` | Hard cap on all normal rows across every query (1–10,000) |
| `maxItemsPerQuery` | integer | `100` | Hard cap per query (1–1,000) |
| `maxPagesPerQuery` | integer | `10` | Page ceiling for paginated modes (1–50) |
| `sortBy` | string | `relevance` | `relevance` or `date`; blog, cafe, news, image and video modes |
| `dateRange` | string | `any` | `any` / `1d` / `1w` / `1m` / `6m` / `1y`; blog, cafe, news, image and video modes |
| `includeAds` | boolean | `true` | Keep paid blog placements and integrated-search ads, flagged `isAd` |
| `includeContent` | boolean | `false` | Fetch blog/news bodies, authors and inline images (one extra request per result) |
| `includeSerpFeatures` | boolean | `true` | Include place cards and AI briefings in `integratedSearch` |
| `containsKeyword` | string | – | Keep results whose title, snippet, domain, source name, cafe name or suggestion contains this text |
| `domain` | string | – | Keep results from this host or suffix, e.g. `blog.naver.com` |

### Example: newest blog posts with full bodies

```json
{
    "mode": "blogSearch",
    "searchQueries": ["비비바", "서울 맛집"],
    "maxItems": 200,
    "maxItemsPerQuery": 100,
    "maxPagesPerQuery": 5,
    "sortBy": "date",
    "dateRange": "1m",
    "includeAds": false,
    "includeContent": true
}
```

### Example: cafe discussion from the past week

```json
{
    "mode": "cafeSearch",
    "searchQueries": ["제주도 여행"],
    "maxItems": 100,
    "sortBy": "date",
    "dateRange": "1w"
}
```

### Example: news and image monitoring for a brand

```json
{
    "mode": "newsSearch",
    "searchQueries": ["BTS", "삼성전자"],
    "maxItems": 300,
    "maxPagesPerQuery": 10,
    "sortBy": "date"
}
```

### Example: integrated page with places and AI briefing

```json
{
    "mode": "integratedSearch",
    "searchQueries": ["비트코인 전망"],
    "maxItems": 50,
    "includeAds": false,
    "includeSerpFeatures": true
}
```

### Example: autocomplete ideas

```json
{
    "mode": "autocomplete",
    "searchQueries": ["비비", "서울 맛집"],
    "maxItems": 20
}
```

## Use cases

- **Korean SEO and rank tracking** — record which pages and blogs rank for a query, in relevance or newest-first order, with stable ranks and exact URLs
- **Brand and news monitoring** — batch a company or product name, restrict to a date window, and keep publisher, section, headline, snippet and publish time
- **Content and RAG datasets** — collect full blog and news bodies with authors, images and normalized timestamps
- **Local and place research** — pull the place pack from integrated results with ratings, review counts, categories and map links
- **Visual and video research** — collect images with original dimensions and videos with duration, views, channel and platform
- **Keyword discovery** — expand seed terms with Naver autocomplete suggestions

## FAQ

**Do I need a Naver account, cookies or a proxy?** No. The actor collects public Naver result pages without login, cookies or any proxy configuration.

**Why does `webSearch` return only a handful of rows?** Naver's web-document section on the integrated page currently shows roughly 2–17 documents per query and does not paginate. Use `integratedSearch` to also capture places, ads and AI briefings, or the dedicated blog, cafe, news, image and video modes for deeper verticals.

**How deep can the verticals go?** It depends on the query. Broad Korean queries can return hundreds of blog, cafe, news, image or video rows; narrow or very recent queries return fewer. The actor stops when Naver runs out of results, when a page repeats, or when your `maxPagesPerQuery` / `maxItems` limits are reached. No rows are invented to fill a quota.

**Are paid results included?** Only when `includeAds` is on. Paid blog placements and integrated-search ads are marked `isAd`, and you can switch them off.

**What does `includeContent` add?** For blog and news rows it fetches the post or article body, inline image URLs, the author where available, and an exact publish time. Some posts are media-only or paywalled; those rows keep their card data and simply omit the missing content fields.

**Why are some publish dates relative?** Naver often prints labels such as `3주 전` or `11시간 전`. Those rows keep the raw label in `date` / `publishedAtRaw`, get a computed `publishedAt`, and are marked `publishedAtPrecision: "relative"` so you can treat them differently from exact timestamps.

**Why is there no cafe body or comment text?** Naver does not expose cafe bodies or comment threads on public search pages. Cafe rows include the card fields Naver prints, including the comment count.

**Why do some image links look like redirects?** Naver's image tab returns signed bridge links for shopping and clip results. They are the official destination Naver serves at collection time; some merchant or short-link tokens expire later, so keep `imageUrl` / `originalUrl` when you need the media itself.

**What is `integratedSearch`?** It collects the blocks on Naver's main result page: web documents, paid ads, place cards and the AI briefing when Naver generates one. Place and AI rows can be turned off with `includeSerpFeatures`.

**Can I search several queries at once?** Yes. Each row records the query that found it, and a result found by two queries is emitted only once.

**Is this affiliated with Naver?** No. This is an independent actor that reads Naver's public search results; it is not endorsed by or affiliated with NAVER Corporation.
