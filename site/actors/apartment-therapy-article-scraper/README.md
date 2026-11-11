# Apartment Therapy Article Scraper

Pull articles from **Apartment Therapy**, a home decor and interior design publication covering decorating, DIY projects, organizing, shopping guides, house tours, and small-space living. Get the newest articles, search by keyword, or browse a specific topic.

## What this actor does

- Fetches the newest published articles
- Searches articles by keyword across titles, summaries, and tags
- Browses articles within a specific topic — decorating, DIY, organizing, shopping, kitchen, bathroom, bedroom, living room, house tours, before & after, or news
- Filters any of the above further by author name
- Empty fields are omitted from records

## Output per article

- `title` — article headline
- `articleUrl` — canonical URL on apartmenttherapy.com
- `author` — author name
- `publishDate` — publication date, ISO 8601 (UTC)
- `category` — primary content category, e.g. `Decorating`, `DIY`, `Shopping`
- `tags` — array of content tags/topics attached to the article
- `summary` — brief article description
- `imageUrl` — main article image URL
- `recordType` — always `article`
- `scrapedAt` — ISO 8601 timestamp of when the record was captured

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getLatest` | `getLatest` / `searchArticles` / `browseCategory` |
| `query` | string | – | Keyword(s) to match against titles, summaries, and tags (mode=searchArticles) |
| `category` | string | `main` | Topic to browse (mode=browseCategory) |
| `authorFilter` | string | – | Keep only articles by an author whose name contains this text |
| `maxItems` | integer | `50` | Maximum articles to return (1–200) |

Available `category` values: `main` (all), `decorating`, `organizing`, `diy`, `shopping`, `small-spaces`, `kitchen`, `bathroom`, `bedroom`, `living-room`, `house-tours`, `before-and-after`, `news`.

### Example: latest articles

```json
{
  "mode": "getLatest",
  "maxItems": 20
}
```

### Example: keyword search

```json
{
  "mode": "searchArticles",
  "query": "small apartment ideas",
  "maxItems": 20
}
```

### Example: browse a category, filtered by author

```json
{
  "mode": "browseCategory",
  "category": "diy",
  "authorFilter": "smith",
  "maxItems": 30
}
```

## Use cases

- **Content curation** — pull the latest decorating and DIY articles for a newsletter or feed
- **Trend tracking** — monitor which topics (small spaces, organizing, house tours) are being published most
- **Author monitoring** — follow new pieces from a specific writer
- **Research** — collect a set of articles on a topic like "small apartment ideas" for reference
- **Media monitoring** — keep an eye on a competitor publication's home-and-design coverage

## FAQ

**Where does the data come from?**  Apartment Therapy's own published article feed — the same information available to anyone browsing the site's latest content.

**How many articles can I get?**  The feed this actor reads carries the most recently published articles (roughly the newest 20 per category); use `maxItems` to cap how many of those you receive. It does not go further back into the full historical archive.

**Does search cover the entire archive or just recent articles?**  Just the recent articles available in the feed — keyword matching is applied to their titles, summaries, and tags.

**Can I combine a category with a keyword search?**  Not in a single run — `mode` picks one approach: latest, keyword search, or category browse. `authorFilter` can be combined with any of the three.

**Why is `tags` sometimes empty?**  Not every article is tagged with additional topics beyond its primary category; when there are none, the field is omitted.

**Is this affiliated with Apartment Therapy?**  No. This is an independent actor that reads Apartment Therapy's publicly published content and is not affiliated with or endorsed by Apartment Therapy.
