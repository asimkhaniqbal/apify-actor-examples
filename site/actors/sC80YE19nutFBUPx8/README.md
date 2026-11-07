# Google Maps Business Search & Reviews MCP Server

A single Google Maps actor that doubles as an MCP (Model Context Protocol) server, so AI agents and automation tools can search businesses or pull reviews through one consistent interface. It reads Google Maps' public pages directly — no login, API key, or cookies required.

## What this actor does

- **Search mode** — find businesses by keyword and location, with ratings, contact details, hours and more
- **Reviews mode** — pull reviews for one place by URL or canonical Place ID, flattened to one row per review
- **MCP-compatible** — the same `search`/`reviews` modes are exposed as callable tools for MCP clients, alongside normal Apify actor runs
- **Multi-language results** — choose the Google Maps interface language for search mode
- Fields Google doesn't expose for a given record (e.g. `website`, `priceLevel`) are omitted rather than returned empty

## Output per record

### Search mode

Each business uses consistent camelCase field names:

```json
{
  "recordType": "business",
  "index": 1,
  "name": "Joe's Pizza",
  "category": "Pizza restaurant",
  "rating": 4.5,
  "reviewsCount": 1234,
  "priceLevel": "$$",
  "address": "7 Carmine St, New York, NY 10014",
  "phone": "+1 212-366-1182",
  "website": "https://www.joespizzanyc.com",
  "googleMapsUrl": "https://www.google.com/maps/place/Joe's+Pizza/@40.7308314,-73.9973325,17z",
  "placeId": "ChIJxxx...",
  "featureId": "0x...:0x...",
  "location": {"lat": 40.7308314, "lng": -73.9973325},
  "scrapedAt": "2025-11-02T20:30:00+00:00",
  "sourceUrl": "https://www.google.com/maps/place/Joe's+Pizza/@40.7308314,-73.9973325,17z"
}
```

Other fields that appear when Google publishes them: `description`, `categoryName`, `categories`, `totalScore`, `ratingHistogram`, `plusCode`, `phoneUnformatted`, `hours`, `additionalInfo`, `photosSample`, `menuUrl`, `reserveTableUrl`, `reservationsUrl`, `orderOnlineUrl`, `permanentlyClosed`, `temporarilyClosed`.

### Reviews mode

One row per review, with place details repeated on every row (both flattened at the top level and grouped under `placeMetadata`) so no join is needed:

```json
{
  "recordType": "review",
  "order": 1,
  "placeId": "ChIJxxx...",
  "featureId": "0x...:0x...",
  "businessName": "Joe's Pizza",
  "businessRating": 4.5,
  "businessReviewCount": 1234,
  "businessCategory": "Pizza restaurant",
  "businessAddress": "7 Carmine St, New York, NY 10014",
  "scrapedAt": "2025-11-02T20:30:00+00:00",
  "placeMetadata": {
    "placeUrl": "https://www.google.com/maps/place/...",
    "placeId": "ChIJxxx...",
    "featureId": "0x...:0x...",
    "businessName": "Joe's Pizza",
    "businessRating": 4.5,
    "businessReviewCount": 1234,
    "businessCategory": "Pizza restaurant",
    "businessAddress": "7 Carmine St, New York, NY 10014"
  },
  "reviewId": "ChZDSUhNMG9nS0VJQ0FnSUQ...",
  "reviewerName": "John Smith",
  "reviewerAvatarUrl": "https://lh3.googleusercontent.com/...",
  "rating": 5.0,
  "reviewText": "Best pizza in NYC! The crust is perfect and the sauce is amazing...",
  "reviewDateOriginal": "2 months ago",
  "placeUrl": "https://www.google.com/maps/place/...",
  "sourceUrl": "https://www.google.com/maps/place/..."
}
```

`order` is the review's 1-based position in this run's output. `reviewDateOriginal` preserves the date text exactly as Google shows it (often a relative date like "2 months ago"). Fields such as `likesCount`, `isLocalGuide`, `reviewerTotalReviews`, `reviewerProfileUrl`, `photosInReview`, `ownerResponseText` and `ownerResponseDate` appear only when Google publishes them for that review.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` or `reviews` |
| `searchQuery` | string | – | What to search for, e.g. `restaurant` (mode=search) |
| `location` | string | – | Where to search, e.g. `New York, NY` (mode=search) |
| `maxResults` | integer | `20` | Maximum businesses to return, 1–500 (mode=search) |
| `language` | string | `en` | Google Maps result language (`en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, `ru`) |
| `placeUrl` | string | – | Google Maps place URL (mode=reviews; use this or `placeId`) |
| `placeId` | string | – | Canonical Google Place ID beginning with `ChIJ` (mode=reviews; use this or `placeUrl`) |
| `maxReviews` | integer | `50` | Maximum reviews to return, 1–200000 (mode=reviews) |
| `proxyConfiguration` | object | – | Apify proxy settings |

### Example: search for businesses

```json
{
  "mode": "search",
  "searchQuery": "pizza restaurant",
  "location": "New York, NY",
  "maxResults": 20
}
```

### Example: scrape reviews for a known place

```json
{
  "mode": "reviews",
  "placeUrl": "https://www.google.com/maps/place/Joe's+Pizza/@40.7308314,-73.9973325,17z",
  "maxReviews": 100
}
```

### Example: search in another language

```json
{
  "mode": "search",
  "searchQuery": "寿司",
  "location": "東京 日本",
  "maxResults": 20,
  "language": "ja"
}
```

## Use cases

- **Market research** — compare competitor locations, ratings and pricing tier in one dataset
- **Site selection** — scan a target area's existing businesses before choosing a new location
- **Review analysis** — pull every review for a place to study sentiment or recurring complaints
- **AI agent tooling** — expose Google Maps search and reviews as callable tools to an MCP-compatible agent
- **Recommendation and RAG pipelines** — feed structured business and review data into a downstream model

## FAQ

**Does this need Google login or cookies?** No — both modes read Google Maps' public pages without any sign-in.

**Why is a field missing from a result?** Only fields Google actually exposes for that record are included; nothing is fabricated to fill a gap.

**Which mode should I use for one specific place?** Use `reviews` with either `placeUrl` or a canonical `placeId`; use `search` for free-text business discovery.

**Why might a run return fewer records than `maxResults` or `maxReviews` asks for?** Google's own public result set for a query or place sets the ceiling — the actor returns everything it can observe, never a padded count.

**Can this be used as an MCP tool?** Yes — the actor's `search` and `reviews` modes are also exposed as MCP tools for agent frameworks that support the protocol, alongside normal Apify actor runs.

**Is a proxy required?** No login is required to use this actor, and most runs complete without a proxy. If Google briefly blocks the direct request, the actor automatically retries through an Apify proxy before giving up; set `proxyConfiguration` to `{"useApifyProxy": false}` if you want to disable that automatic retry and only ever connect directly.

**Is this affiliated with Google?** No — this is an independent actor reading Google Maps' public pages; it is not operated by or affiliated with Google.
