# Steam Game Pricing & Metacritic Scraper

Steam's storefront is a catalog, a chart board, and a price list at the same time. This actor reads those public surfaces without a login, API key, cookie, or proxy setup, and returns one clean record per app: localized price and discount, Metacritic score, review totals, community tags, media, DLC, platforms, and release information.

## Five ways to build a dataset

| Mode | What it does |
|---|---|
| `search` | Matches one or more keywords against Steam's own store search and pages through the results. |
| `browse` | Walks a chart: top sellers, new releases, current specials, coming soon, weeklong deals, popular new releases, or the legacy featured mix. |
| `appDetail` | Looks up numeric App IDs, or store URLs pasted into the App ID field. |
| `urls` | Accepts full `store.steampowered.com/app/<id>/` URLs and resolves each one. |
| `featured` | Keeps the old behavior: top sellers, new releases, specials, and coming soon, in that order. |

Search and chart modes paginate through Steam's 50-result pages until `maxItems` is reached or the source runs out. Every record is deduplicated by App ID, so overlapping queries and chart pages do not create duplicates.

## What lands in each record

- **Identity** — `appId`, `appName`, `appType`, `appUrl`, `recordType`, `scrapedAt`
- **People** — `developer`, `publisher`, plus `developers[]` and `publishers[]`
- **Pricing** — `currentPrice`, `originalPrice`, `priceCurrency`, `priceInitialCents`, `priceFinalCents`, `discountPercent`, `isDiscounted`, `isFree`, `packages[]`, `packageOptions[]`
- **Critical reception** — `metacriticScore`, `metacriticUrl`, `recommendationsTotal`, `pressQuotes[]`
- **User reviews** — `reviewScore`, `reviewScoreDesc`, `totalReviews`, `totalPositive`, `totalNegative`, `positiveReviewPercent`
- **Classification** — `genres[]`, `categories[]`, `tags[]`, `controllerSupport`, `requiredAge`, `contentDescriptors[]`, `platforms`
- **Release** — `releaseDate`, `releaseDateIso`, `comingSoon`
- **Content** — `shortDescription`, `detailedDescription`, `website`, `supportedLanguages`
- **Media** — `headerImage`, `capsuleImage`, `capsuleImageV5`, `backgroundImage`, `backgroundRawImage`, `screenshots[]`, `screenshotThumbnails[]`, `movies[]`
- **Extras** — `achievementsTotal`, `dlcAppIds[]`, `dlc[]`, `supportUrl`, `supportEmail`

Steam does not publish every field for every app. Missing values are omitted from the record rather than written as `null`, `""`, or a made-up placeholder.

## Input

| Field | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `search` | `search`, `browse`, `appDetail`, `urls`, or `featured`. |
| `searchQueries` | string list | `["adventure"]` | One or more keywords for `search`. |
| `browseCategory` | select | `topSellers` | Chart used by `browse`; also `newReleases`, `specials`, `comingSoon`, `weeklongDeals`, `popularNew`, `featuredMix`. |
| `appIds` | string list | – | Numeric IDs or store URLs for `appDetail`. |
| `storeUrls` | string list | – | Full store URLs for `urls`. |
| `appTypeFilter` | select | `all` | Keep only `game`, `dlc`, `demo`, or `software`. |
| `sortBy` | select | `relevance` | `released`, `priceAsc`, `priceDesc`, `reviews`, `metacritic`, or `nameAsc`. |
| `onlyDiscounted` | boolean | `false` | Drop records that are not on sale. |
| `minDiscount` | integer | `0` | Minimum discount percentage (0–100). |
| `maxPrice` | number | – | Maximum final price in the storefront currency. |
| `minMetacritic` | integer | `0` | Minimum Metacritic score (0–100). |
| `country` | text | `us` | Two-letter Steam store country, e.g. `us`, `gb`, `de`, `ca`. Controls currency. |
| `language` | select | `english` | Steam store language for names, descriptions, and store-page text. |
| `includeTags` | boolean | `true` | Read the public store page for community tags and DLC names. |
| `includeReviewSummary` | boolean | `true` | Add Steam's positive/negative review totals and label. |
| `maxItems` | integer | `20` | Stop after this many saved records (1–1000). |

### Daily test / smallest useful run

```json
{
  "mode": "search",
  "searchQueries": ["Hades"],
  "maxItems": 1,
  "includeTags": false,
  "includeReviewSummary": false
}
```

### Browse current specials, deep discounts only

```json
{
  "mode": "browse",
  "browseCategory": "specials",
  "onlyDiscounted": true,
  "minDiscount": 50,
  "maxPrice": 20,
  "maxItems": 25
}
```

### Look up specific games

```json
{
  "mode": "appDetail",
  "appIds": ["1245620", "570", "730"],
  "maxItems": 3
}
```

### Localized prices from pasted URLs

```json
{
  "mode": "urls",
  "storeUrls": [
    "https://store.steampowered.com/app/1245620/",
    "https://store.steampowered.com/app/1091500/"
  ],
  "country": "de",
  "language": "german",
  "maxItems": 2
}
```

## Compatibility with saved tasks

Older tasks keep working. The actor still reads the former input keys and maps them in code:

- `searchQuery` → `searchQueries`
- `appId` → `appIds`
- `urls` / `startUrls` → `storeUrls`
- `appType` → `appTypeFilter` (including the old empty-string value, which means `all`)
- `mode: "featured"` → the featured top sellers / new releases / specials / coming soon mix

The current production prefill (`mode=featured`, `searchQuery=adventure`, `maxItems=20`) still runs and returns records.

## FAQ

**Do I need a Steam account, API key, cookie, or proxy?**
No. The actor reads Steam's public store endpoints and product pages. No credentials are required and no proxy configuration is needed.

**Why is `metacriticScore` missing for some games?**
Steam only publishes a Metacritic score for a subset of titles. When the store has no score, the field is omitted.

**Why is `discountPercent` missing on a game that is on sale?**
The field is written only when Steam reports a non-zero discount at the moment of the run. A zero discount is not treated as a sale.

**Why do prices differ from my region?**
Steam localizes prices by store country. Set `country` to the storefront you want; `priceCurrency` and the formatted price follow that storefront.

**What is the difference between `tags` and `genres`?**
`genres` and `categories` come from Steam's app metadata. `tags` are the community tags shown on the public store page, capped at the 20 Steam displays.

**How fresh is the data?**
Every run reads Steam live. Prices, discounts, chart positions, review totals, and tags reflect the moment the actor runs.

**Are review numbers the same as Metacritic?**
No. `reviewScore` and the review totals are Steam user reviews. `metacriticScore` is the critic score Steam displays from Metacritic.

**Can I get more than 50 results from a chart?**
Yes. Search and chart modes page through 50-result pages up to `maxItems` (maximum 1000) or until the chart/search is exhausted. Steam's featured new-releases block is a fixed 30-item list, and the featured mix is a fixed set of featured blocks; those are source limits, not pagination bugs.

## Data source

Public Steam Store pages: `store.steampowered.com/api/appdetails`, `featuredcategories`, `search/results`, the product page, and `ajaxappreviews`. The actor is independent and not affiliated with Valve or Steam.
