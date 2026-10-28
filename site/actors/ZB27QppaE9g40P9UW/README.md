# Yandex News Story Clusters Across Russia & the CIS

Yandex News groups coverage of the same event from many outlets into a story cluster. This actor reads the public Dzen News surface that now serves Yandex News, returns the clusters, and lists every linked article inside each one. Search by keyword, browse a category, read the homepage sections, or paste a story URL. No login, API key, or proxy is needed.

## What this actor does

- **Four ways to collect news:** keyword search, 16 category feeds, the 15 homepage sections, or specific story URLs and cluster IDs.
- **Story clusters, not just headlines:** each story carries its cluster ID, source-reported coverage count, actual article count, unique source list, topics, image, country/region and timestamps.
- **Every linked article:** article records include the publisher, headline, clean article URL, publication date, image and source flags.
- **Filters and sorting** on the returned set: date window, source domain, publisher, keyword, relevance, newest, oldest or most covered.
- **Automatic deduplication** by story ID and article ID, and empty fields are omitted rather than stored as nulls.

## Output per story

| Field | Description |
|---|---|
| `recordType` | `"story"` |
| `storyId` | Story cluster ID (UUID); shared with its article records |
| `persistentId` | Yandex's numeric story ID |
| `storyUrl` | Canonical story page on Dzen News |
| `headline` | Cluster headline |
| `summary` | Short cluster summary, when Yandex provides one |
| `category` | Broad rubric name, when available |
| `topics` / `topicTags` | Rubric or topic labels attached to the cluster |
| `topicIds` | Numeric rubric IDs |
| `articleCount` | Number of publications Yandex reports for the cluster |
| `articlesReturned` | Number of article records actually collected |
| `sourceCount` | Number of distinct outlet names among the collected articles |
| `sources` | Outlet names, when articles were collected |
| `leadSource` | Outlet behind the lead article |
| `leadArticleUrl` | Publisher URL of the lead article |
| `publishedAt` | Cluster timestamp, ISO UTC |
| `publishedAtRaw` | Timestamp label as shown by Yandex |
| `imageUrl` | Cluster image, when present |
| `country` / `region` | Story location, when Yandex reports it |
| `importance` / `relatedStories` | Yandex story metrics |
| `commentsCount` / `likesCount` | Interaction counts, when present |
| `domain` | `ru` |
| `scrapedAt` | UTC timestamp when collected |

`articlesReturned`, `sourceCount` and `sources` are emitted only when `includeArticles` is true. With it off, no article records are fetched, so those article-derived fields are omitted.

## Output per article

| Field | Description |
|---|---|
| `recordType` | `"article"` |
| `storyId` | Parent story cluster ID |
| `docId` | Yandex document ID |
| `headline` | Outlet's headline |
| `summary` | Article snippet, when Yandex provides one |
| `sourceName` | Outlet name |
| `sourceUrl` | Outlet site root |
| `articleUrl` | Clean link to the article |
| `publishedAt` | Publication date (`YYYY-MM-DD`) |
| `publishedAtIso` | Publication timestamp, ISO UTC |
| `publishedAtRaw` | Timestamp label as shown by Yandex |
| `imageUrl` | Outlet logo or article image |
| `isForeignAgent` / `isDzenArticle` | Yandex flags |
| `agencyId` | Yandex outlet ID |
| `domain` | `ru` |
| `scrapedAt` | UTC timestamp when collected |

## Output per homepage record

`topStories` (and the legacy `getTrending` mode) emits one `recordType: "trending"` record with `topics` (homepage section names), `sectionCounts`, `domain` and `scrapedAt`, followed by the story records for those sections.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchNews` | `searchNews`, `browseCategory`, `topStories` or `byStoryUrls` |
| `searchQueries` | array | – | Keywords or phrases (mode=searchNews); the daily-test prefill uses `["Яндекс"]` |
| `category` | string | `politics` | Rubric slug or `all` (mode=browseCategory) |
| `storyUrls` | array | – | Story/instory URLs or cluster UUIDs (mode=byStoryUrls) |
| `maxStories` | integer | `20` | Maximum story clusters across the run (1–500) |
| `sortOrder` | string | `newest` | `relevance`, `newest`, `oldest` or `mostCovered` (mode=searchNews) |
| `timeRange` | string | `any` | `any`, `day`, `week` or `month` (mode=searchNews) |
| `publishedAfter` | string | – | Inclusive `YYYY-MM-DD` start date (mode=searchNews) |
| `publishedBefore` | string | – | Inclusive `YYYY-MM-DD` end date (mode=searchNews) |
| `includeDomains` | array | `[]` | Keep clusters with an article from any listed domain |
| `excludeDomains` | array | `[]` | Drop clusters with an article from any listed domain |
| `includePublishers` | array | `[]` | Keep clusters matching any outlet name substring |
| `excludePublishers` | array | `[]` | Drop clusters matching any outlet name substring |
| `includeKeywords` | array | `[]` | Keep clusters matching any keyword in the title, summary or articles |
| `excludeKeywords` | array | `[]` | Drop clusters matching any keyword |
| `includeDetails` | boolean | `true` | Fetch cluster details such as topics, reported count, image and metrics |
| `includeArticles` | boolean | `true` | Emit one record per linked article; when off, article-derived story fields are omitted |
| `maxArticlesPerStory` | integer | `100` | Maximum articles collected per cluster (1–1000) |

### Example: brand monitoring by keyword

```json
{
  "mode": "searchNews",
  "searchQueries": ["Яндекс", "Яндекс Такси"],
  "maxStories": 50,
  "timeRange": "week",
  "includeArticles": true
}
```

### Example: all categories, story records only

```json
{
  "mode": "browseCategory",
  "category": "all",
  "maxStories": 200,
  "includeArticles": false
}
```

### Example: homepage top stories

```json
{
  "mode": "topStories",
  "maxStories": 40
}
```

### Example: resolve specific story clusters

```json
{
  "mode": "byStoryUrls",
  "storyUrls": [
    "https://dzen.ru/news/story/85c864f8-36d7-5b96-ab8d-937bea5d9c9b",
    "c29b2d3c-c183-546b-9a23-b9aba12e981e"
  ],
  "maxArticlesPerStory": 300
}
```

## Use cases

- **Media monitoring:** follow a company, person or event across many Russian outlets.
- **Share-of-voice measurement:** use the reported and returned article counts to size coverage.
- **PR and communications:** see which outlets picked up a story and how the headline evolved.
- **Newsroom research:** export a cluster's full article list with dates and publisher links.
- **Risk and compliance:** monitor keywords, source domains and publication windows.

## FAQ

**What happened to Yandex News?**
The public Yandex News pages now redirect to Dzen News, which provides the same story-cluster model. This actor reads that public surface.

**How many stories can one search return?**
Yandex search returns 20 clusters per page and its depth varies by query; observed runs reached 9–22 pages (roughly 175–400 unique clusters) before the result set ended. `maxStories` is a hard run-wide cap.

**Why is `articleCount` different from `articlesReturned`?**
`articleCount` is the number Yandex reports for the cluster, while `articlesReturned` is the number of article records actually collected. The two can differ in either direction: Yandex's live count changes as outlets publish or remove articles (observed 96 reported vs 122 returned, and 246 vs 362), and very large clusters may be capped by `maxArticlesPerStory`.

**Are the date and sort filters applied by Yandex?**
Yandex's public search page applies them in the browser rather than in the server response. This actor applies the same filters and sorting to the set it returns, so a very old custom date range may return fewer results than a dedicated archive search.

**Which categories are available?**
All, politics, society, SVO, business, world, sport, incidents, culture, technology, science, cars, army, show business, health, Moscow and interesting. `all` reads every feed.

**Do I need a proxy or login?**
No. The actor connects directly and handles the site's anonymous session automatically. No cookies or credentials are required.

**Can I search other regional editions or languages?**
The current public surface serves one Russian-language feed; the old `ru/kz/by/uz/com` edition selector no longer changes the content. Search queries can still be written in Russian, English or any language Yandex indexes.

**Why is there no trending keyword list?**
The old “what people discuss” panel is no longer present on the homepage. `topStories` returns the homepage sections and their top clusters instead.

**Does this actor download full article text?**
No. It returns article metadata and the publisher's URL. Full text stays on the publisher's site and may be paywalled or geo-restricted.

**Is this affiliated with Yandex or Dzen?**
No. This is a third-party actor reading public pages; it is not affiliated with, endorsed by, or sponsored by Yandex or Dzen.
