# Wikipedia Article Lookup by URL or Keyword

Wikipedia exposes far more than an article's opening paragraph: a one-line description, the categories it belongs to, its images, its place on the map, the Wikidata entity behind it, its edit history, and the language editions it exists in. This actor collects that structure for articles you name by URL, exact title or keyword search — or for every page inside a Wikipedia category — and can add the full plain-text body, section outline, links, citation count, and pageview history.

Everything comes from Wikimedia's public APIs. There is no login, token, cookie or proxy setup.

## Ways to feed it

- **Paste URLs** — desktop, mobile and `index.php` links all work, in any language edition. A URL's own domain decides its language.
- **Name exact titles** — one title per list item, in the language edition you choose. Redirects are followed.
- **Search by keyword** — Wikipedia's ranked full-text search, with the result snippet and rank attached to each record.
- **Browse a category** — every page in a category (or several) becomes an article lookup.
- Mix any of the above in one run. An article reached through several URLs, titles or searches is emitted once, with those inputs recorded as provenance.

## What each record contains

| Field | What it is |
|---|---|
| `title` | Canonical article title |
| `url` / `canonicalUrl` | Article URL and canonical URL |
| `displayTitle` | Title as Wikipedia renders it |
| `summary` | Lead-section plain text |
| `fullText` | Complete article body, when enabled |
| `description` | One-line description in the article's language |
| `language` | Language edition code |
| `pageId` / `wikidataId` | Wikipedia page ID and Wikidata entity ID |
| `wordCount` | Words in the best available text |
| `categories` | Category names |
| `thumbnail` / `originalImage` | Lead image objects with `source`, `width`, `height` |
| `images` | Viewable URLs of images used in the article |
| `coordinates` | `latitude` and `longitude` for geographic subjects |
| `languageCount` / `availableLanguages` | How many other editions exist, and their codes |
| `lastModified` / `revisionId` / `lastEditor` / `lastEditComment` / `revisionSize` | Latest revision metadata |
| `isDisambiguation` | Present and true for disambiguation pages |
| `redirectedFrom` | The input that redirected here |
| `sections` | Heading outline (`title`, `level`, `anchor`) |
| `externalLinks` | Outbound URLs cited or listed by the article |
| `links` | Titles of linked Wikipedia articles |
| `referencesCount` | Number of inline citations |
| `pageviews` | `granularity`, `start`, `end`, `total`, and `points` (`date`, `views`) |
| `searchQuery` / `searchRank` / `searchSnippet` | Search provenance when the article came from a keyword |
| `requestedTitle` / `requestedTitles` / `requestedUrls` / `matchedSearchQueries` | All inputs that resolved to this article |
| `sourceCategory` | Category that produced the record, when applicable |
| `scrapedAt` | Capture timestamp |

Fields the source does not have for a particular article are omitted, never filled with `null`, `""`, `[]` or `0`.

## Inputs

| Field | Type | Default | Purpose |
|---|---|---|---|
| `articleUrls` | list | – | Article URLs to look up |
| `articleTitles` | list | – | Exact titles to look up in `language` |
| `searchQueries` | list | – | Keyword searches |
| `maxArticlesPerQuery` | 1–500 | `5` | Results per search query |
| `categoryNames` | list | – | Categories to browse, with or without `Category:` |
| `maxArticlesPerCategory` | 1–500 | `20` | Members per category |
| `language` | 374 codes | `en` | Edition for titles, searches and categories |
| `maxItems` | 1–10000 | `100` | Total record cap |
| `includeFullContent` | boolean | `false` | Add `fullText` |
| `includeSections` | boolean | `false` | Add `sections` |
| `includeExternalLinks` | boolean | `false` | Add `externalLinks` |
| `includeLinks` | boolean | `false` | Add `links` |
| `includeReferences` | boolean | `false` | Add `referencesCount` |
| `includePageviews` | boolean | `false` | Add `pageviews` |
| `pageviewsGranularity` | monthly / daily | `monthly` | Pageview point size |
| `pageviewsPeriods` | 1–365 | `12` | Months (max 12) or days |

Older saved tasks keep working: keys such as `urls`, `titles`, `pageTitles`, `query`, `searchQuery`, `maxResultsPerSearch`, `maxSearchResults`, `lang`, `includeFullArticle` and `fetchFullArticle` are read and mapped inside the actor even though they are not shown in the form.

## Examples

Look up two URLs:

```json
{
  "articleUrls": [
    "https://en.wikipedia.org/wiki/Albert_Einstein",
    "https://de.wikipedia.org/wiki/Berlin"
  ]
}
```

Search, with enrichment:

```json
{
  "searchQueries": ["quantum computing"],
  "maxArticlesPerQuery": 10,
  "language": "en",
  "includeFullContent": true,
  "includeSections": true,
  "includePageviews": true,
  "pageviewsGranularity": "monthly",
  "pageviewsPeriods": 6
}
```

Browse a category:

```json
{
  "categoryNames": ["Physics"],
  "maxArticlesPerCategory": 50,
  "language": "en",
  "includeExternalLinks": true
}
```

Mixed run with titles, URLs and a search:

```json
{
  "articleUrls": ["https://ja.wikipedia.org/wiki/東京"],
  "articleTitles": ["Python (programming language)", "Alan Turing"],
  "searchQueries": ["history of computing"],
  "maxItems": 25,
  "includeReferences": true
}
```

## Data source and attribution

The actor reads Wikimedia's MediaWiki Action API, its parse API, and the Wikimedia pageview API. Wikipedia text is available under CC BY-SA; keep the source attribution when you republish it. This actor is independent and is not affiliated with or endorsed by the Wikimedia Foundation.

## FAQ

**Do I need a key, login or proxy?**
No. All four lookup paths work anonymously from an Apify container.

**Which language editions are supported?**
The 374 language codes Wikimedia lists, from `en` and `de` to `zh-min-nan` and `simple`. Canonical and legacy codes map to the same edition. URL lookups take the edition from the URL; the `language` field applies to titles, searches and categories.

**Why is `summary` shorter than `fullText`?**
`summary` is the lead section. `fullText` is the whole article and is only requested when you switch it on.

**Can I get pageviews for any period?**
Yes — up to 12 monthly points or 365 daily points. The current partial month is included; daily data ends at the most recent day Wikimedia has published (usually two days ago).

**What are the hard limits?**
Search returns at most 500 hits per query; categories, external links, internal links and language links are capped at 500 values per article, and images at 50. `referencesCount` counts inline citations but does not return the citation text. Rendered HTML and infobox key/value extraction are intentionally not part of this actor.

**What happens to missing or non-article pages?**
Pages the API reports as missing are skipped and logged. Disambiguation pages are returned with `isDisambiguation: true` so you can filter them downstream.

**Does it work with old tasks created before the revamp?**
Yes. The four original inputs keep their names, and earlier alias keys are mapped in code.

## Use it as an API (server mode)

Need one answer right away instead of a batch job? Send the input as an HTTP request and the results come back in the response. There's no run to start and poll.

```bash
curl -H "Authorization: Bearer $APIFY_TOKEN" \
  "https://vexaloom-labs--wikipedia-article-lookup-by-url-or-keyword.apify.actor/?articleTitles=Ada%20Lovelace"
```

Or POST the same input as JSON:

```bash
curl -X POST -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"articleTitles": ["Ada Lovelace"]}' \
  "https://vexaloom-labs--wikipedia-article-lookup-by-url-or-keyword.apify.actor/"
```

- The fields are the same as in the input above. With GET, repeat a parameter once for each value in a list.
- You get back a JSON array with the same records a normal run saves. They're saved to the run's dataset too.
- Each request is capped at 20 results and 4 minutes. For bigger jobs, start a normal run.
- The request runs in your own Apify account at the Actor's normal price (a start fee plus a price per result). The server shuts down after a minute with no requests.
- The Server tab on this page lists every parameter and lets you try a request from your browser.
