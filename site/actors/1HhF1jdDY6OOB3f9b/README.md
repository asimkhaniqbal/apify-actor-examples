# Ecwid App Market Apps, Ratings & Reviews Scraper

Scrape **the Ecwid App Market** — the add-on marketplace for Ecwid (Lightspeed E-Series) stores. Collect every listed app with its developer, categories, pricing, free-trial flag, logo, rating and review count, then optionally open detail pages for feature bullets, screenshots and support contacts, or pull paginated customer reviews. No login, cookies, or proxy setup.

## What this actor does

- **Five modes:** browse the whole market, filter by category or collection, search by keyword, fetch exact apps, or collect reviews.
- **Complete catalog:** reads the live listing in one request — currently 405 apps — and deduplicates by app namespace.
- **Exact categories:** all 14 marketplace categories plus the Featured, New, and Free collections.
- **Decision signals:** rating, review count, real logo URL, pricing type, and a free-trial flag on every app that has them.
- **Optional depth:** developer website, support URL and email, feature bullets, screenshots, attached reviews, and standalone review records.
- **Filters and sorting:** pricing type, free-only, minimum rating, minimum review count, and sorting by name, rating or review count.
- **Empty fields are omitted:** a field is only included when the marketplace actually provides it.

## Output per app

- `appId` — unique app namespace (for example `the-printful`)
- `name` — app title
- `tagline` — short listing description
- `description` — full listing description (HTML preserved)
- `developer` — developer or company name, when published
- `categories[]` — marketplace category names
- `pricingType` — `free`, `freemium`, or `paid`
- `pricing` — the marketplace's own pricing text, when it is more than just “Free”
- `hasFreeTrial` — true when the pricing text advertises a trial
- `rating` — average rating from 0 to 5; omitted when the app has no rating
- `reviewCount` — number of published reviews; `0` is a real value
- `url` — app page on the Ecwid marketplace
- `iconUrl` — the app's real logo URL
- `headerImageUrl` — the app's header/banner image URL
- `developerWebsite` — developer site from the detail page (when `includeDetails` is on)
- `supportUrl` — support link from the detail page (when `includeDetails` is on)
- `supportEmail` — support email from the detail page (when `includeDetails` is on)
- `features[]` — feature bullets from the detail page (when `includeDetails` is on)
- `screenshots[]` — gallery image URLs, excluding the header image (when `includeDetails` is on)
- `reviews[]` — attached reviews (when `includeReviews` is on), each with `reviewId`, `rating`, `text`, `author`, `date`
- `recordType` — always `app`
- `scrapedAt` — ISO 8601 UTC capture timestamp

## Output per review

Returned by `mode: "reviews"`, or as extra records when `emitReviewRecords` is on:

- `reviewId` — marketplace review ID
- `appId`, `appName` — the reviewed app
- `rating` — 1 to 5 stars
- `text` — review body as published
- `author` — display name shown by the marketplace
- `date` — review timestamp from the source
- `url` — the app page
- `recordType` — always `review`
- `scrapedAt` — ISO 8601 UTC capture timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `browse` | `browse` / `byCategory` / `search` / `byApp` / `reviews` |
| `category` | string | `all` | Category or collection (mode=byCategory); legacy names accepted |
| `searchQuery` | string | – | Keyword matched against name, tagline, description, developer and categories (mode=search) |
| `appIds` | array | – | Namespaces or full marketplace URLs (mode=byApp or mode=reviews) |
| `includeFreeOnly` | boolean | `false` | Keep only completely free apps; overrides `pricingType` |
| `pricingType` | string | `all` | `all` / `free` / `freemium` / `paid` |
| `minRating` | number | – | Keep apps rated at or above this value (0–5); unrated apps are excluded when set |
| `minReviewCount` | integer | – | Keep apps with at least this many reviews |
| `includeDetails` | boolean | `false` | Fetch developer website, support contact, feature bullets and screenshots per app |
| `includeReviews` | boolean | `false` | Attach reviews to app records (app modes only) |
| `maxReviewsPerApp` | integer | `10` | Review cap per app (1–100) |
| `emitReviewRecords` | boolean | `false` | Also emit one record per attached review |
| `sortBy` | string | `default` | `default` / `name` / `rating` / `reviewCount` |
| `maxItems` | integer | `100` | App-record cap in app modes; review-record cap in `reviews` mode (1–1000) |

## Examples

### Example: top-rated apps with a meaningful review history

```json
{
  "mode": "browse",
  "minRating": 4.5,
  "minReviewCount": 10,
  "sortBy": "rating",
  "maxItems": 50
}
```

### Example: one category with full detail enrichment

```json
{
  "mode": "byCategory",
  "category": "shipping",
  "includeDetails": true,
  "maxItems": 40
}
```

### Example: keyword search for free inventory tools

```json
{
  "mode": "search",
  "searchQuery": "inventory",
  "includeFreeOnly": true,
  "maxItems": 30
}
```

### Example: exact apps with reviews attached

```json
{
  "mode": "byApp",
  "appIds": [
    "the-printful",
    "https://www.ecwid.com/apps/featured/kliken-stats"
  ],
  "includeDetails": true,
  "includeReviews": true,
  "maxReviewsPerApp": 25
}
```

### Example: review records for one app

```json
{
  "mode": "reviews",
  "appIds": ["kliken-stats"],
  "maxReviewsPerApp": 50,
  "maxItems": 50
}
```

## Use cases

- **Competitive research** — compare the apps available to Ecwid merchants across categories, pricing models and ratings.
- **App directory sites** — mirror marketplace listings with logos, ratings, review counts and screenshots.
- **Pricing analysis** — separate genuinely free apps from freemium and externally billed ones.
- **Partner and developer scouting** — find active developers and the support channels they publish.
- **Voice-of-customer research** — export paginated reviews for sentiment analysis and product-gap studies.
- **Market monitoring** — re-run on a schedule to catch new listings and changing ratings.

## FAQ

**Do I need an Ecwid account?** No. The marketplace pages and review endpoints used here are public.

**How many apps are on the market?** The live total changes as developers add and remove listings. A full `browse` run currently returns about 405 apps; the actor deduplicates by namespace and reports the real total.

**What is the difference between free, freemium and paid?** `free` means the listing is marked completely free. `freemium` covers a free tier, a free trial, free tokens, or a “Free to $X” plan. Everything else is `paid`, including externally billed apps and in-app charges. The raw marketplace pricing text is kept in `pricing` when it adds detail.

**Why does a “Free to $X” app say freemium?** Because it has a free entry point but paid tiers above it. The `hasFreeTrial` flag tells you whether the pricing text also advertises a trial.

**Why do some apps have no rating?** Ratings come from the marketplace's public review widget. Apps with no reviews, or with no review product yet, omit `rating`. A real `reviewCount: 0` is kept; apps with no review product at all omit both fields.

**How many reviews can I collect?** The source returns five reviews per page. `maxReviewsPerApp` accepts 1–100, and the actor stops at the last page or after two empty pages. The largest app currently has 225 reviews, so the 100-review cap is deliberate.

**Which categories are available?** All 14 live marketplace categories plus Featured, New, and Free. A few categories (CMS, Social Media, Shipping carriers) have no public category page, so the actor matches them exactly against each app's category list.

**Does the actor expose install counts, app versions, last-updated dates, languages, badges or structured pricing tiers?** No. The marketplace does not publish those datapoints; the actor ships the raw pricing text and the fields the source actually provides.

**Are review authors anonymized?** Sometimes. The marketplace displays some reviewers as anonymized names; the actor publishes the author name exactly as shown and does not attempt to de-anonymize it.

**Is this affiliated with Ecwid or Lightspeed?** No. This is an independent third-party actor reading publicly available marketplace pages.
