# Yandex News Story Clusters Across Russia & the CIS

Yandex News clusters coverage of the same event from many outlets into a single story, across Russian, Kazakh, Belarusian, Uzbek and international editions. This actor searches, browses by category, or reads the trending homepage, and returns both the story clusters and the individual articles inside each one.

## What this actor does

- Searches Yandex News by keyword or phrase, in Russian or English
- Browses a specific news category page, such as politics, sport, or technology
- Reads the trending homepage for its real-time trending topic list and top story clusters
- Links every article record back to its parent story via a shared `storyId`
- Omits any field that has no value for a given record rather than returning an empty placeholder

## Output per record

Records carry a `recordType` of `story`, `article`, or `trending`.

### Story record

| Field | Description |
|---|---|
| `recordType` | `"story"` |
| `storyId` | Identifier shared with that story's article records |
| `storyUrl` | Link to the story cluster on Yandex News |
| `headline` | Story headline |
| `summary` | Short summary text, when shown |
| `category` | News category, when shown |
| `topicTags` | Topic keyword tags attached to the story |
| `articleCount` | Total number of publications Yandex reports for the story — not limited by `maxStories` |
| `domain` | The regional edition this record came from |
| `scrapedAt` | UTC timestamp when the record was scraped |

### Article record

| Field | Description |
|---|---|
| `recordType` | `"article"` |
| `storyId` | Identifier of the parent story record |
| `headline` | This outlet's version of the headline |
| `summary` | Short summary text, when shown |
| `sourceName` | Publishing outlet's name |
| `sourceUrl` | Publishing outlet's own site, when resolvable |
| `articleUrl` | Direct link to the article on the outlet's site |
| `publishedAt` | Publication date |
| `domain` | The regional edition this record came from |
| `scrapedAt` | UTC timestamp when the record was scraped |

### Trending record

| Field | Description |
|---|---|
| `recordType` | `"trending"` |
| `topics` | List of trending topic keywords from the homepage panel |
| `domain` | The regional edition this record came from |
| `scrapedAt` | UTC timestamp when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | searchNews | searchNews (keyword search), browseCategory (browse a category page), or getTrending (homepage trending + top stories) |
| `searchQueries` | array of strings | – | One or more search queries — used in `searchNews` mode |
| `categoryUrls` | array of strings | – | Category page URLs to browse — used in `browseCategory` mode. Confirmed working slugs include computers, politics, world, sport, business, science, society, culture, auto, and health. |
| `domain` | string | ru | Regional edition: ru (Russia), kz (Kazakhstan), by (Belarus), uz (Uzbekistan), or com (international) |
| `maxStories` | integer | 20 | Maximum number of story clusters to scrape per query or category (1-200) |
| `sortOrder` | string | newest | Result order: newest or popular |
| `language` | string | ru | Content language: ru, kk, be, uz, or en |
| `proxyConfiguration` | object | Apify Residential, country RU | Yandex News only serves Russian IP ranges reliably, so a Russia-located proxy is set by default |

### Example: search by keyword

```json
{
  "mode": "searchNews",
  "searchQueries": ["Яндекс", "Apple"],
  "maxStories": 20
}
```

### Example: browse a category in Kazakhstan's edition

```json
{
  "mode": "browseCategory",
  "categoryUrls": ["https://dzen.ru/news/rubric/business"],
  "domain": "kz",
  "maxStories": 10
}
```

### Example: trending homepage

```json
{
  "mode": "getTrending",
  "domain": "ru",
  "maxStories": 15
}
```

## Use cases

- **Brand and reputation monitoring** — track Russian and CIS press coverage of a company, product, or public figure
- **Share-of-voice measurement** — use `articleCount` to see how widely a topic was picked up across outlets
- **Cross-market media comparison** — run the same search across the ru, kz, by, and uz editions
- **News-cycle tracking** — use `getTrending` to see what's rising on the homepage right now
- **Academic and journalism research** — build a dataset of Russian-language coverage for systematic analysis

## FAQ

**Which Yandex News editions are supported?**
Five: ru (Russia), kz (Kazakhstan), by (Belarus), uz (Uzbekistan), and com (international).

**How do story and article records connect?**
Every `article` record carries a `storyId` that matches its parent `story` record's `storyId`; filter by `recordType` to separate story clusters from individual articles.

**What does `articleCount` actually measure?**
It's the total number of publications Yandex reports for that story cluster, independent of `maxStories` — a useful signal of how widely a story was covered even if you only collected a handful of articles from it.

**Can I search in English?**
Yes — set `language` to `en` and use English search queries; results include English-language sources Yandex News indexes.

**Why do some stories list fewer articles than `articleCount` suggests?**
Yandex's reported total can include articles that are paywalled, geo-restricted, or no longer listed; the actor returns only what's actually rendered on the page.

**Do I need to configure a proxy myself?**
No — a Russia-located residential proxy is set by default since Yandex News is not reliably reachable from most other regions. You can override it in `proxyConfiguration` if needed.

**Is this affiliated with Yandex?**
No. This actor reads Yandex News's public pages and is not affiliated with, endorsed by, or sponsored by Yandex.
