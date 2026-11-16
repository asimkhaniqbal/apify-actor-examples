# StreetEasy NYC Listing & Agent Detail Scraper

StreetEasy is the primary listing site for apartments and homes across New York City. This actor takes any StreetEasy search URL — sales or rentals, with whatever filters you've already applied on the site — and turns the matching listings into structured rows, with an optional detail pass that adds amenities, agent contact info, transit access, and building facts.

## What this actor does

- Scrapes sale and rental listings from any StreetEasy search URL across all five boroughs
- Respects whatever filters are already baked into the URL (price range, bedrooms, neighborhood, no-fee, property type, etc.)
- Paginates automatically to collect all matching results up to your limit
- In detailed mode, visits each listing page to add description, amenities, agent info, coordinates, transit access, and more
- No field is ever `null` — a value StreetEasy doesn't publish for a listing comes back as an empty string, `0`, or an empty array instead

## Output per listing

Always present:

- `url` — direct link to the listing page
- `price` — listed sale price or monthly rent
- `address` — full street address including unit
- `neighborhood` — StreetEasy's neighborhood label
- `bedrooms`, `bathrooms` — bed and bath counts
- `squareFeet` — interior square footage
- `propertyType` — e.g. Co-op, Condo, Townhouse
- `photoUrl` — main listing photo
- `listingType` — `sale` or `rental`
- `searchUrl` — the search URL that produced this result
- `scrapedAt` — ISO 8601 extraction timestamp

Added only when `detailedMode` is on:

- `description` — full listing description text
- `amenities[]` — building/unit amenities (doorman, gym, laundry, etc.)
- `buildingName` — the building's name, when it has one
- `agentName`, `agentBrokerage` — listing agent and their brokerage
- `latitude`, `longitude` — GPS coordinates
- `daysOnStreetEasy` — days the listing has been active
- `photos[]` — every listing photo
- `nearbyTransit[]` — nearby subway lines and stops
- `maintenanceFee` — monthly maintenance/HOA fee
- `taxes` — annual property taxes
- `pricePerSqFt` — computed price per square foot
- `status` — e.g. Active, In Contract, Sold, Rented, Delisted
- `yearBuilt` — year the building was constructed

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | StreetEasy search URLs, built with any filters on the site |
| `maxResults` | integer | `50` | Maximum listings to scrape per search URL (1–500) |
| `detailedMode` | boolean | `false` | Visit each listing page for amenities, agent info, coordinates, etc. |
| `proxyConfiguration` | object | – | Not used — a residential proxy is applied automatically because StreetEasy blocks non-residential traffic; this field exists only for schema compatibility |

### Example: Manhattan sales in a price/bedroom band

```json
{
  "startUrls": [
    { "url": "https://streeteasy.com/for-sale/manhattan/price:500000-1500000|beds>=2" }
  ],
  "maxResults": 50,
  "detailedMode": true
}
```

### Example: Brooklyn rentals under $3,000/mo

```json
{
  "startUrls": [
    { "url": "https://streeteasy.com/for-rent/brooklyn/price:-3000" }
  ],
  "maxResults": 100
}
```

### Example: no-fee one-bedroom rentals in a neighborhood

```json
{
  "startUrls": [
    { "url": "https://streeteasy.com/for-rent/east-village|beds>=1|no_fee:1" }
  ],
  "maxResults": 50,
  "detailedMode": true
}
```

## Use cases

- **Market analysis** — track price trends and inventory by neighborhood over time
- **Investment research** — compare price-per-square-foot across listings to spot undervalued properties
- **Lead generation** — build a list of listings and agent contacts matching specific criteria
- **Competitive monitoring** — watch how long comparable listings sit on the market via `daysOnStreetEasy`
- **Academic and policy research** — study NYC housing affordability and inventory patterns

## FAQ

**Do I need a StreetEasy account to use this?**
No. Everything comes from publicly visible listing pages; no login is required.

**How do I build a search URL?**
Go to streeteasy.com, apply whatever filters you want (borough, price, beds, no-fee, etc.), then copy the resulting URL into `startUrls`. Any filter combination the site supports will carry through.

**What's the difference between normal and detailed mode?**
Normal mode reads what's shown on the search-results cards: price, address, beds/baths, and a photo. Detailed mode additionally visits each listing's own page to extract the full description, amenities, agent contact, coordinates, transit access, and building facts — at the cost of one extra page load per listing.

**Why is a proxy needed, and why can't I configure it?**
StreetEasy blocks traffic that doesn't look like it's coming from a normal residential connection, so a residential proxy is applied automatically on every run. The `proxyConfiguration` input field is kept only for schema compatibility and has no effect.

**Can I cover all five boroughs?**
Yes — Manhattan, Brooklyn, Queens, The Bronx, and Staten Island are all covered; just point `startUrls` at the appropriate borough or neighborhood search.

**How many listings can one run return?**
Up to 500 per search URL via `maxResults`. For larger datasets, add several narrower search URLs (by neighborhood or price band) to `startUrls` in the same run.

**How fresh is the data?**
Every run reads StreetEasy live, so results reflect what's currently posted at run time.

**Is this affiliated with StreetEasy?**
No. This is an independent, third-party actor that reads StreetEasy's publicly visible listing pages; it is not affiliated with or endorsed by StreetEasy or Zillow Group.
