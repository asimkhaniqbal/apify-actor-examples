# Trulia Listings & Price History Scraper

Turn any Trulia search results page or individual property page into structured data — for-sale, for-rent, and sold listings all work. Paste a Trulia URL with whatever filters you've already applied on the site (price, beds, neighborhood, property type) and this actor collects every matching listing, optionally following through to each one's own page for price history, tax history, and open-house details.

## What this actor does

- **Search or direct mode in one input:** mix Trulia search-result URLs (paginated automatically) with individual property URLs in the same run
- **Covers for-sale, for-rent, and sold listings** — just change the URL path (`/for_sale/`, `/for_rent/`, `/sold/`)
- **Optional deep fetch:** enable `fetchDetails` to add price history, tax history, full feature lists, open-house schedules, and the agent's phone number
- **Filters inherited from the URL:** any filter you set on Trulia's own site (price range, bedrooms, property type, etc.) carries straight through, since the actor just follows the URL you give it
- **Empty fields are omitted** — a field only appears on a record when Trulia actually published that value for the listing

## Output per property

- `url` — canonical Trulia property URL
- `listingType` — `for_sale`, `for_rent`, or `sold`
- `propertyType` — e.g. Single Family Residential, Condo, Townhouse
- `price` — numeric price in USD
- `priceLabel` — formatted price as shown on the page (e.g. `$485,000`)
- `address`, `city`, `state`, `zipCode`, `neighborhood`
- `latitude`, `longitude`
- `bedrooms`, `bathrooms`, `sqft`
- `photos[]` — listing photo URLs
- `listingDate`, `daysOnMarket`
- `agentName`, `brokerName`, `mlsId`
- `priceHistory[]` — past price changes, each with date/price/event/source (requires `fetchDetails`)
- `taxHistory[]` — past tax-year assessments and tax amounts (requires `fetchDetails`)
- `features` — feature categories grouped by section, including highlights (requires `fetchDetails`)
- `openHouses[]` — scheduled open-house start/end times (requires `fetchDetails`)
- `agentPhone` — listing agent's phone number (requires `fetchDetails`)
- `scrapedAt` — ISO 8601 UTC timestamp

If a specific URL can't be fetched or parsed, that record instead contains `inputUrl`, `error`, and `scrapedAt`, so you can see exactly which input failed without losing the rest of the run.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | `["https://www.trulia.com/for_sale/Austin,TX/"]` | Trulia search-result pages or individual property pages |
| `maxItems` | integer | `3` | Max listings to collect from search pages (1–10,000); ignored for direct property URLs |
| `fetchDetails` | boolean | `false` | Fetch each listing's own detail page for price/tax history, features, and open houses |
| `proxyConfiguration` | object | Apify residential proxy | Proxy configuration; residential is required because Trulia's PerimeterX protection blocks datacenter IPs |

### Example: for-sale homes in Austin under $600K

```json
{
  "startUrls": ["https://www.trulia.com/for_sale/Austin,TX/0-600000_price/"],
  "maxItems": 100
}
```

### Example: 3+ bedroom rentals in New York with full detail

```json
{
  "startUrls": ["https://www.trulia.com/for_rent/New_York,NY/3p_beds/"],
  "maxItems": 50,
  "fetchDetails": true
}
```

### Example: specific property pages

```json
{
  "startUrls": [
    "https://www.trulia.com/p/tx/austin/123-main-st-austin-tx-78701--1234567890",
    "https://www.trulia.com/p/ma/boston/456-oak-ave-boston-ma-02108--9876543210"
  ],
  "fetchDetails": true
}
```

## Use cases

- **Market research** — pull every active listing in a metro area to analyze pricing by neighborhood and property type
- **Investor screening** — collect sold-listing price and tax history to estimate comps for a target address
- **Rental analysis** — gather for-rent listings filtered by bedroom count to benchmark rents in a market
- **Agent/brokerage monitoring** — track which agents and brokerages are listing in a given area
- **Price-drop tracking** — re-run on the same search URL periodically and diff `price`/`priceLabel` over time

## FAQ

**Do I need a Trulia or Zillow account?**
No. This actor only reads data visible on Trulia's public pages without logging in.

**Is a proxy required?**
Yes. Trulia's PerimeterX bot protection blocks datacenter IPs, so residential proxy is required for reliable results on Apify's cloud; the default `proxyConfiguration` is already set to a residential group.

**How many listings can I get from one search URL?**
That depends on the market — a typical city search can return anywhere from a few dozen to several thousand matching listings. The actor paginates through the search results automatically up to your `maxItems` cap.

**What's missing compared to a logged-in Trulia/Zillow account?**
Full school ratings and reports, detailed neighborhood demographics, crime scores, mortgage pre-qualification data, and private listing notes are all gated behind an account and aren't available to this actor.

**Can I scrape sold listings?**
Yes — use a URL under `https://www.trulia.com/sold/`, e.g. `https://www.trulia.com/sold/Austin,TX/`.

**What's the difference between a search URL and a direct property URL in `startUrls`?**
A search URL gets paginated and yields many listing records up to `maxItems`. A direct property URL returns exactly one record for that property. You can mix both types in the same `startUrls` list.

**Why do some fields only show up when `fetchDetails` is enabled?**
Price history, tax history, the full feature list, open houses, and the agent's phone number only live on each property's own detail page, not on the search-results card — so the actor only fetches them when you opt in, since it roughly doubles the number of page requests per run.
