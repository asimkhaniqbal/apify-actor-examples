# Craigslist Classifieds Scraper with Category Attributes

Scrape [Craigslist](https://craigslist.org) search results and individual posts across any city subdomain — `sfbay`, `newyork`, `chicago`, `losangeles`, `london`, and hundreds more. Get titles, prices, descriptions, coordinates, images, and the category-specific attribute set for jobs, housing, autos, and for-sale posts. No login, no cookies, and no proxy needed for normal volume.

## What this actor does

- Scrapes any Craigslist search page or direct post URL, auto-detecting the city/region from the subdomain
- Pages through search results (up to 20 result pages per search URL) and opens each post's detail page for full description, attributes, and images
- Extracts category-specific fields — vehicle specs for autos, bedroom/bathroom counts for housing, compensation for jobs
- Filters the collected posts by minimum/maximum price or by whether a post has at least one image
- Empty fields are omitted from each record

## Output per listing

- `type` — always `craigslist_listing`
- `url`, `id` — post URL and numeric post ID
- `title`, `category`, `subcategory`, `location`, `region`
- `postingType` — category slug from the URL path (`cto`, `apa`, `lab`, `fud`, ...)
- `price`, `priceQualifier` — e.g. `/ 2br - 1083ft²`, `OBO`, `/ month`
- `postedAt`, `updatedAt`
- `description` — plain-text post body
- `images[]` — large gallery image URLs
- `latitude`, `longitude`, `mapAddress` — from the post's embedded map, when shown
- `attributes` — the raw label-to-value set from the post's attribute group
- `phoneNumbers[]`, `notices[]`, `replyUrl`
- Jobs: `compensation`, `employmentType`, `jobTitle`
- For-sale (generic): `condition`, `size`
- Autos: `yearManufactured`, `makeManufacturer`, `modelName`, `odometer`, `transmission`, `fuelType`, `titleStatus`, `cylinders`, `paintColor`, `drive`, `vehicleType`, `vin`
- Housing: `bedroomCount`, `bathroomCount`, `sqft`, `availableDate`, `housingType`, `laundry`, `parking`, `rentPeriod`, `listedBy`, `applicationFee`, `brokerFee`, `openHouseDates`, `catsOk`, `dogsOk`, `furnished`, `smoking`
- `scrapedAt`

With `scrapeDetails` off, records only carry the search-card fields: `type`, `url`, `id`, `region`, `title`, `price`, `location` and `scrapedAt`. A search result whose detail page fails to load also falls back to these fields. Direct post URLs need `scrapeDetails` on; without it (or if the post page can't be loaded) they produce no listing.

If the run ends without a single listing (bad URL, removed post, no search results, or filters that removed everything), one informational record is returned instead: `type: "craigslist_blocked"`, `requestedUrls`, `searchTerm` (empty string when none was given), `message`, `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | `[{"url": "https://sfbay.craigslist.org/search/jjj"}]` | Craigslist search URLs (`/search/...`) or direct post URLs (ending in `<id>.html`); other URLs are skipped. Filters already in a search URL (e.g. `?min_price=`) are kept |
| `searchTerm` | string | – | Keyword appended to each search URL as `?query=<term>`; ignored for direct post URLs |
| `maxItems` | integer | `3` | Maximum posts to return across all start URLs (1–1000) |
| `scrapeDetails` | boolean | `true` | Fetch each post's detail page for full description, attributes, coordinates, and images; off returns listing-card data only |
| `minPrice` | integer | – | Minimum price, compared with the number in the post's price (in the listing's own currency). Posts without a price are dropped when either price filter is set |
| `maxPrice` | integer | – | Maximum price, same rules as `minPrice` |
| `hasImage` | boolean | `false` | Only include posts with at least one image (needs `scrapeDetails` on, since images come from the detail page) |
| `proxyConfiguration` | object | `{"useApifyProxy": false}` | Optional — Craigslist works from datacenter IPs for normal volume |

### Example: search a category in one city

```json
{
    "startUrls": [{ "url": "https://sfbay.craigslist.org/search/jjj" }],
    "maxItems": 50
}
```

### Example: keyword search with a price range

```json
{
    "startUrls": [{ "url": "https://newyork.craigslist.org/search/sss" }],
    "searchTerm": "bicycle",
    "minPrice": 50,
    "maxPrice": 300,
    "hasImage": true,
    "maxItems": 100
}
```

### Example: scrape one specific post

```json
{
    "startUrls": [{ "url": "https://sfbay.craigslist.org/eby/grd/d/hayward-item/7918207144.html" }],
    "maxItems": 1
}
```

## Use cases

- **Marketplace price tracking** — monitor asking prices for a product category across one or more cities
- **Vehicle shopping** — pull year, mileage, and condition for every car listing matching a search
- **Rental research** — collect bedroom count, rent, and pet policy across a city's housing posts
- **Job board aggregation** — gather compensation and employment-type data from a labor category
- **Lead sourcing** — filter by price and image presence to find postings worth following up on

## FAQ

**Do I need a proxy?**
No. Craigslist usually responds normally to data-center IPs at typical run volumes; enable a proxy only if requests start getting refused.

**Does this work on a login wall?**
Craigslist's public listings have no login wall, so this actor works without any authentication.

**Can I scrape just one specific post?**
Yes. Put the direct post URL (ending in `<id>.html`) into `startUrls` and set `maxItems: 1`.

**What cities are supported?**
Every Craigslist subdomain — the region is detected automatically from the URL, so `sfbay`, `newyork`, `london`, or any other subdomain works the same way.

**Why do some listings lack `attributes` or `images`?**
Not every post fills in Craigslist's optional attribute fields or uploads photos — those keys are only included when the source post actually has them.

**What happens if a search or post URL returns nothing?**
If the whole run produces no listing, it still completes with one `craigslist_blocked` record listing the requested URLs and the possible reasons (invalid URL, removed post, or no matching results) instead of failing silently. A single empty URL in a run that finds listings elsewhere is just skipped.

**Why did I get fewer posts than `maxItems` when using filters?**
The actor collects up to `maxItems` posts from the search first and then applies `minPrice`, `maxPrice` and `hasImage`, so posts removed by the filters aren't replaced. For a fuller result, put the price filter in the search URL itself (for example `https://newyork.craigslist.org/search/sss?min_price=50&max_price=300`) or raise `maxItems`.

**Is this affiliated with Craigslist?**
No. This is an independent actor that reads Craigslist's public pages and is not operated or endorsed by Craigslist.
