# UsedVending.com Equipment & Food Truck Listings Finder

UsedVending.com is a marketplace for used vending machines, food trucks, concession trailers, and other mobile-business equipment, organized into dozens of narrow sub-categories. This actor browses any of those categories or runs a keyword search across the whole marketplace, then applies state, price, and sold-status filters on top.

## What this actor does

- Browses a single equipment category page by page, or runs a keyword search across the entire marketplace, optionally narrowed to one category.
- Covers 66 categories the site itself uses in navigation — food trucks, food trailers, vending machines by type, mobile business vehicles, tiny homes, horse trailers, and step vans.
- Filters results by US state, a price range, sold status, and whether the listing confirms shipping or transport is available.
- Sorts output by price, date posted, biggest price reduction, or state.
- Fetches each listing's own page for description, full photo gallery, specifications, equipment list, and condition; a field is left off the record whenever that listing never published it.

## Output per listing

| Field | Description |
|---|---|
| `itemId` | The site's own listing code |
| `title`, `description` | Listing title and description |
| `category`, `categorySlug` | Category derived from the listing's own breadcrumb trail |
| `price` | Asking price in USD |
| `priceCurrency` | Currency code for the price |
| `wasPrice` | Original price, when the listing shows a markdown from a higher price |
| `condition` | `Used`, `New`, `Refurbished`, or `Damaged`, as stated on the listing |
| `manufacturer`, `model`, `year` | Parsed from the listing's own specification table, when present |
| `state`, `stateAbbr` | Listing location |
| `sold` | True if the listing is marked SOLD |
| `images`, `thumbnailUrl` | Full-resolution photo URLs |
| `videoUrl` | YouTube walkaround-video link, when the listing has one |
| `shippingAvailable` | True or false when the listing states whether it can be shipped or transported; left off when the listing gives no shipping signal at all |
| `specifications`, `equipmentList` | `{label, value}` pairs exactly as published on the listing |
| `listingUrl` | Link to the listing on UsedVending.com |
| `datePosted`, `dateUpdated` | Listing timestamps |
| `scrapedAt` | UTC timestamp of the scrape |

Seller or dealer identity isn't published on listing pages — inquiries route through UsedVending itself — so there's no seller field to report.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byCategory` | `byCategory` to walk one category's pages, or `search` for a free-text keyword search across the marketplace. |
| `category` | string | – | Equipment category to browse (`byCategory`; left blank, it browses food concession trailers), or an optional narrowing filter (`search`; left blank, searches every category). |
| `keyword` | string | – | Free-text search term, used in `search` mode. |
| `state` | string | – (any state) | Restrict results to one US state. |
| `priceMin` | integer | – | Drop listings priced below this amount, in USD. |
| `priceMax` | integer | – | Drop listings priced above this amount, in USD. |
| `sortBy` | string | – (site order) | `priceLowHigh`, `priceHighLow`, `dateNewest`, `dateOldest`, `priceReduction`, or `state`. |
| `includeSold` | boolean | `false` | Include listings already marked SOLD. |
| `shippingOnly` | boolean | `false` | Keep only listings that confirm shipping or transport is available. |
| `maxItems` | integer | `20` | Maximum number of listings to return. |

### Example: browse food concession trailers

```json
{
  "mode": "byCategory",
  "category": "used-food-concession-trailers",
  "maxItems": 20
}
```

### Example: keyword search narrowed by state and price

```json
{
  "mode": "search",
  "keyword": "ice cream",
  "state": "TX",
  "priceMax": 30000,
  "maxItems": 25
}
```

### Example: cheapest vending machines first

```json
{
  "mode": "byCategory",
  "category": "used-vending-machines",
  "sortBy": "priceLowHigh",
  "maxItems": 50
}
```

## Use cases

- **Vending route operators** sourcing machines by type, brand, and budget.
- **Food-truck entrepreneurs** comparing trucks, trailers, and new-build pricing side by side.
- **Equipment resellers** benchmarking asking prices across a category or state.
- **Regional market research** — compare how much inventory and at what price points different states carry.
- **Finance and insurance outreach** — find recently posted equipment listings worth following up on.

## FAQ

**Do I need a login or proxy?**
No. UsedVending.com's listings are fully public, and no account or credentials are needed.

**Why might a listing's `category` differ from the category I searched?**
`category`/`categorySlug` reflect the listing's own breadcrumb trail on its detail page, which is the authoritative source — occasionally a listing shows up cross-referenced under a category page that isn't its own primary category.

**Why do some fields not show up on every listing?**
Only values the listing actually publishes are included. `wasPrice` only appears when a listing shows a markdown from a higher original price, and `manufacturer`/`model`/`year` only appear when the spec table states them.

**Does `sortBy` change which listings come back?**
No — it only reorders the listings that already matched your filters and `maxItems`. `priceReduction` sorts by the largest `wasPrice` minus `price` gap first; `state` sorts alphabetically.

**What exactly does `shippingOnly` keep?**
Only listings whose page explicitly says shipping or transport is available. Listings marked pickup-only are excluded, as are listings where the page gives no shipping signal at all — an unstated status is never treated as shippable.

**What does `condition` actually mean here?**
It's the condition value UsedVending.com itself publishes per listing. Many listings labeled "NEW BUILD" in their title are still tagged `Used` by the site's own data — this actor reports exactly what the listing states rather than inferring a condition from the title.

**How does keyword search work if the site's own search doesn't filter by keyword?**
UsedVending.com's search parameter doesn't actually narrow results server-side. This actor instead checks each listing's title, description, category, manufacturer, and model, and keeps only listings where every word of your keyword appears somewhere in those fields.

**How fresh is the data?**
Each listing carries its own posted/updated timestamps straight from the site, and every run reads the live pages at request time.
