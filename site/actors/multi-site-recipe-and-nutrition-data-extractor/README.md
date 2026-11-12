# Multi-Site Recipe & Nutrition Data Extractor

Recipe sites and food blogs publish a machine-readable `schema.org/Recipe` record on every recipe page — the same record search engines use for star ratings and cooking times. This actor reads that record (and the common WordPress recipe formats when a site has no JSON-LD), so one input works across major recipe sites and thousands of food blogs.

## What it does

- **Search by keyword** across BBC Good Food, Epicurious, Tasty, Food Network, NYT Cooking, Bon Appétit, Delish, Budget Bytes, Pinch of Yum, Minimalist Baker, Smitten Kitchen, Love and Lemons, and Skinnytaste.
- **Scrape direct recipe URLs** from any supported site or food blog.
- **Crawl category, tag, or search-result pages** and follow their pagination.
- **Walk sitemaps**, including one level of nested sitemap indexes.
- **Parse ingredients, instructions, and nutrition** from JSON-LD, WP Recipe Maker, Tasty Recipes, Jetpack Recipe, or schema.org microdata.
- **Filter** by rating, rating count, total time, and a required keyword.

## Output per recipe (`recordType: "recipe"`)

- `url`, `id` — canonical recipe URL (`id` mirrors `url`)
- `platform` — site slug, e.g. `epicurious`, `tasty`, `bbcgoodfood`, `smittenkitchen`
- `name`, `description` — title and short blurb, HTML stripped
- `image` — hero image URL
- `author` — `{name, url}`
- `ratingValue`, `ratingCount`, `reviewCount` — average rating (0–5), number of ratings, number of written reviews
- `prepTimeMinutes`, `cookTimeMinutes`, `totalTimeMinutes` — ISO-8601 durations and human durations such as `1 hr 15 min` are both supported; `totalTimeMinutes` falls back to prep + cook
- `recipeYield` — servings or pieces, e.g. `"8 slices"`
- `recipeCategory`, `recipeCuisine` — arrays
- `keywords` — keyword tags
- `recipeIngredient` — ingredient lines
- `recipeInstructions` — step strings, HTML stripped, including `HowToSection` groups
- `nutrition` — calories, protein, carbs, fat, saturated fat, cholesterol, sodium, fiber, sugar, serving size when published
- `video` — `{url, name, thumbnailUrl}` when the recipe has one
- `datePublished`, `dateModified` — dates as published by the site
- `scrapedAt` — ISO 8601 UTC timestamp

Empty fields are omitted at every depth — never `null`, `""`, `[]`, or `{}`.

## Output per failed direct URL (`recordType: "url_failed"`)

Direct URLs that cannot be read produce a typed row instead of disappearing:

- `reason` — `unsupported_site` (AllRecipes or another explicitly blocked host), `not_found`, `anti_bot_block`, `no_recipe_jsonld`, `filtered_out`, or `fetch_error`
- `url`, `platform`, `scrapedAt`

If every direct URL fails, or a search/listing yields candidates but none are recipes, the dataset also gets one `recordType: "scrape_summary"` row with the dominant reason and a plain-language message. A search that finds no candidate URLs at all reports it in the run status message.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `byUrls`, `byListing`, `byTag` (legacy alias for `byListing`), or `bySitemap` |
| `searchQuery` | string | `banana bread` | Keyword for `mode=search` |
| `site` | select | `all` | One supported site, or `all` |
| `recipeUrls` | array | `["https://www.epicurious.com/recipes/food/views/banana-bread"]` | Direct recipe URLs for `mode=byUrls` |
| `listingUrls` | array | – | Category/tag/search pages for `mode=byListing` |
| `sitemapUrls` | array | – | Sitemap URLs for `mode=bySitemap` |
| `maxItems` | integer | `50` | Hard cap on recipe records (1–1000) |
| `maxPages` | integer | `2` | Pages per site or listing URL (1–10) |
| `minRating` | integer | – | Minimum rating on a 0–500 scale (400 = 4.0/5) |
| `minRatingCount` | integer | – | Minimum number of ratings |
| `maxTotalTimeMinutes` | integer | – | Maximum prep + cook time |
| `keywordIncludes` | string | – | Required keyword in name or description |
| `proxy` | object | `{"useApifyProxy": false}` | Optional Apify Proxy configuration |

### Examples

Search Epicurious for banana bread:

```json
{
  "mode": "search",
  "searchQuery": "banana bread",
  "site": "epicurious",
  "maxItems": 20
}
```

Search every supported site:

```json
{
  "mode": "search",
  "searchQuery": "vegan dinner",
  "site": "all",
  "maxItems": 50,
  "maxPages": 2
}
```

Scrape direct URLs:

```json
{
  "mode": "byUrls",
  "recipeUrls": [
    "https://www.epicurious.com/recipes/food/views/banana-bread",
    "https://tasty.co/recipe/banana-bread",
    "https://www.bbcgoodfood.com/recipes/vegan-banana-bread"
  ]
}
```

Crawl a category page and its next pages:

```json
{
  "mode": "byListing",
  "listingUrls": ["https://www.epicurious.com/recipes/category/dessert"],
  "maxPages": 3,
  "maxItems": 40
}
```

Walk a sitemap:

```json
{
  "mode": "bySitemap",
  "sitemapUrls": ["https://www.bbcgoodfood.com/sitemap.xml"],
  "maxItems": 100
}
```

## Supported search sites

BBC Good Food, Epicurious, Tasty, Food Network, NYT Cooking, Bon Appétit, Delish, Budget Bytes, Pinch of Yum, Minimalist Baker, Smitten Kitchen, Love and Lemons, and Skinnytaste. Direct URL and sitemap modes work on any site that publishes recipe JSON-LD or one of the supported WordPress recipe formats.

## Compatibility with older inputs

Old saved tasks keep working because legacy keys are mapped in code:

- `mode=byTag` and `tagUrls` → `byListing` and `listingUrls`
- `query`, `searchTerm`, `keyword`, `q` → `searchQuery`
- `urls`, `recipeUrl`, `url` → `recipeUrls`
- `sitemapUrl`, `sitemaps` → `sitemapUrls`
- `maxResults` → `maxItems`
- `maxPagesPerSite`, `pages` → `maxPages`
- `siteScope`, `searchSites`, `sites` → `site`
- `minRatingScore` (0–5) → `minRating` (0–500)
- `mustContain`, `keywordFilter` → `keywordIncludes`

Only fields the actor actually reads are visible in the input schema. Unknown legacy keys are accepted by Apify and mapped inside the actor.

## Limitations

- **AllRecipes** is protected by Akamai Bot Manager and is rejected as `unsupported_site`.
- **Serious Eats, Simply Recipes, Food & Wine, EatingWell, and Martha Stewart** currently return HTTP 403 to cookieless datacenter requests. They work in `byUrls` only if your proxy configuration can reach them; otherwise they appear as `anti_bot_block`.
- Search pagination depends on the site. Tasty, NYT Cooking and Delish expose the first page only; BBC Good Food, Epicurious, Food Network, Bon Appétit and the WordPress sites support deeper pagination.
- Search results are returned in the site's own relevance order. A result can occasionally be a round-up or article that has no recipe record; those candidates are skipped.
- **Budget Bytes and Smitten Kitchen** return HTTP 403 from Apify cloud IPs for both search and direct recipe pages; both work from a local/residential connection. On cloud they surface as `anti_bot_block`; the other WordPress sites in the search list are unaffected.
- Recipe data is only as complete as the source page. Missing fields are omitted instead of guessed.

## Notes

No cookies, logins, or API keys are required. The actor fetches pages with a Chrome TLS fingerprint. Enable the optional `proxy` field only if you want to supply Apify Proxy groups; no proxy is needed for the supported sites' default paths.
