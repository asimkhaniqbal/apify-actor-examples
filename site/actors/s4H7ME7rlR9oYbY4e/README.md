# Wikipedia Article Lookup by URL or Keyword

Wikipedia holds a structured summary, category list, and image set behind every one of its articles, in whichever language edition you need. This actor pulls that structured data for a list of articles you name directly, or for articles matching a search term, in the language edition you choose.

## What this actor does

- Fetches structured data for specific articles when you provide their Wikipedia URLs.
- Searches Wikipedia by keyword and fetches structured data for the matching articles, up to a per-query limit.
- Works against any of Wikipedia's language editions by language code.
- Resolves article image filenames to their actual viewable image URLs.
- Returns one record per article; fields a given article doesn't have come through as an empty string, empty list, or `0` rather than being left out.

## Output per article

- `title` — article title
- `url` — full Wikipedia URL for the article
- `summary` — lead-section text extract (the opening paragraphs)
- `description` — short one-line description of the topic
- `categories` — the article's Wikipedia categories
- `thumbnail` — thumbnail image object with `source`, `width`, `height`
- `images` — viewable URLs of images used in the article (icons, logos, and similar boilerplate images are filtered out)
- `lastModified` — timestamp of the article's last edit
- `pageId` — Wikipedia's internal page ID
- `language` — language code used for this article
- `scrapedAt` — ISO timestamp of when the record was captured

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `articleUrls` | array | – | Wikipedia article URLs to fetch directly |
| `searchQueries` | array | – | Search terms to find articles by keyword |
| `maxArticlesPerQuery` | integer | `5` | Maximum articles returned per search query (1–50) |
| `language` | string | `en` | Wikipedia language edition code, e.g. `en`, `fr`, `de`, `ja`. Used for `searchQueries`; for `articleUrls` the language is read from the URL's domain instead. |

### Example: fetch specific articles

```json
{
  "articleUrls": [
    "https://en.wikipedia.org/wiki/Python_(programming_language)",
    "https://en.wikipedia.org/wiki/Artificial_intelligence"
  ]
}
```

### Example: search by keyword

```json
{
  "searchQueries": ["machine learning", "quantum computing"],
  "maxArticlesPerQuery": 3,
  "language": "en"
}
```

### Example: a non-English language edition

```json
{
  "searchQueries": ["intelligence artificielle"],
  "language": "fr",
  "maxArticlesPerQuery": 5
}
```

## Use cases

- **Knowledge base building** — collect structured summaries and categories to seed an internal knowledge graph.
- **Content research** — gather topic descriptions and lead-section text before writing or editing related content.
- **Dataset construction** — assemble clean, structured article text and metadata for NLP or model training.
- **Topic coverage audits** — search a set of keywords to see which related articles Wikipedia already covers.
- **Multilingual comparison** — fetch the same topic across several language codes to compare coverage or description differences.

## FAQ

**Is this affiliated with Wikipedia or the Wikimedia Foundation?**
No. This actor reads Wikipedia's publicly available article data; it is not operated by or affiliated with the Wikimedia Foundation.

**Do I need a login, key, or proxy?**
No. Every article and search lookup works without authentication or a proxy.

**Which languages are supported?**
Any Wikipedia language edition — set `language` to its code, such as `en`, `fr`, `de`, `es`, `ja`, `zh`, `ru`, `pt`, `ar`, or `ko`.

**Does this return the full article text?**
No — `summary` is the lead-section extract (the article's opening paragraphs), not the entire article body. That extract is enough context for most research, enrichment, and content use cases.

**Why might `categories` or `images` be empty for an article?**
Some short or newly created articles have few or no categories assigned, and some articles don't use any non-boilerplate images — in both cases the field is returned as an empty list rather than guessed.

**What happens if I give a URL for an article that doesn't exist?**
That URL is skipped and not included in the output, since there's no article data to return for it.

**Can I mix `articleUrls` and `searchQueries` in the same run?**
Yes — the actor processes any URLs you provide and also runs any search queries you provide, within the same run.

**How current is the data?**
Every run reads Wikipedia live, so summaries, categories, and edit timestamps reflect the article's current state at run time.
