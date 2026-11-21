# Instacart Storefront Price Scraper

Scrape **real-time grocery prices from Instacart storefronts** — Costco, Safeway, Publix, Kroger, Walgreens, ALDI, Wegmans, and 20+ other Instacart-partnered retailers. Search any storefront by keyword, browse a department/aisle, look up specific products by item ID, or pull a retailer's store overview with trending items. No login, no API key, no cookies required.

## What this actor does

- **Five modes:** `search` (keyword → priced results), `browseAisle` (department/category browse), `listAisles` (discover a retailer's department/aisle taxonomy), `productDetail` (lookup by item ID), `storeOverview` (retailer info + trending items)
- **30 retailers** available via dropdown, plus a free-text override for any other Instacart-partnered retailer
- **Real prices** — current price, price-per-unit, pack/unit size, regular (pre-discount) price, discount %, and sort by price (low-to-high / high-to-low)
- **Deals & ratings** — surfaces sale badges/promo labels and public star ratings/review counts wherever Instacart shows them
- **Filters:** min/max price, brand name, in-stock only, on-sale only, min rating
- **Empty fields are omitted** from every record

## Output per product

- `itemId` — full Instacart item ID (e.g. `items_74-17444603`), reusable in `productDetail` mode
- `productId`, `legacyId` — Instacart's internal product identifiers
- `name`, `brandName`, `size`, `category`
- `priceString` (e.g. `$17.40`), `price` (numeric, e.g. `17.4`)
- `pricePerUnitString` (e.g. `$0.09/oz`), `pricingUnitString` (e.g. `3 x 64 oz`)
- `regularPriceString` / `regularPrice` — pre-discount price when the item is on sale (e.g. `$19.67`)
- `discountPercentString` (e.g. `30%`), `promoLabel` (e.g. `$6 off; limit 5`), `onSale` (boolean)
- `rating` (0–5) and `reviewCount` — when Instacart publicly shows a star rating for the item
- `imageUrl` — product photo
- `available`, `stockLevel`
- `dietaryAttributes[]` — e.g. `organic`, `lactose_free`, `gluten_free`
- `tags[]` — e.g. `storeBrand`
- `aisleSlug` — the aisle/collection slug the item was found in (mode=browseAisle only)
- `retailerSlug`, `retailerName`
- `productUrl` / `sourceUrl`
- `recordType: "product"`, `scrapedAt`

## Output per retailer (mode = `storeOverview`)

- `retailerId`, `retailerSlug`, `retailerName`, `retailerType`
- `categories[]` — e.g. `grocery`, `club`, `wholesale`
- `sourceUrl`
- `recordType: "retailer"`, `scrapedAt`

## Output per aisle/department (mode = `listAisles`)

- `aisleId`, `aisleName`, `aisleSlug` — pass `aisleSlug` into `mode=browseAisle` to fetch its products
- `collectionType` — e.g. `department`, `retailer_collection_v2`, `dynamic`
- `parentAisleId` / `parentAisleName` — present for sub-aisles nested under a department
- `childAisleCount` — number of sub-aisles under a top-level department
- `retailerSlug`, `retailerName`
- `sourceUrl`
- `recordType: "aisle"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `browseAisle` / `listAisles` / `productDetail` / `storeOverview` |
| `retailerSlug` | string | `costco` | Retailer to scrape (dropdown of 30 popular retailers) |
| `customRetailerSlug` | string | – | Any Instacart retailer slug not in the dropdown; overrides `retailerSlug` |
| `searchQuery` | string | `milk` | Keyword to search (mode=search) |
| `orderBy` | string | `bestMatch` | `bestMatch` / `priceAsc` / `priceDesc` (mode=search, browseAisle) |
| `aisleSlug` | string | – | Department/aisle (collection) slug to browse (mode=browseAisle); discover slugs via `mode=listAisles` |
| `itemIds` | array | – | Full item IDs to look up (mode=productDetail) |
| `minPrice` | int | – | Drop items priced below this (USD) |
| `maxPrice` | int | – | Drop items priced above this (USD) |
| `brandContains` | string | – | Keep only items whose brand contains this text |
| `inStockOnly` | boolean | `false` | Drop out-of-stock items |
| `onSaleOnly` | boolean | `false` | Keep only items currently discounted from their regular price |
| `minRating` | number | – | Drop items with an average rating below this (0–5); items without a public rating are kept |
| `maxItems` | int | `20` | Hard cap on emitted records (1–200) |
| `proxyConfiguration` | object | Apify proxy (auto) | Optional; enable if you see repeated 429/403s |

### Example: search a retailer by keyword

```json
{
  "mode": "search",
  "retailerSlug": "costco",
  "searchQuery": "organic eggs",
  "orderBy": "priceAsc",
  "maxItems": 30
}
```

### Example: filter by price and brand

```json
{
  "mode": "search",
  "retailerSlug": "safeway",
  "searchQuery": "paper towels",
  "minPrice": 5,
  "maxPrice": 25,
  "brandContains": "bounty",
  "inStockOnly": true
}
```

### Example: look up specific products by ID

```json
{
  "mode": "productDetail",
  "retailerSlug": "costco",
  "itemIds": ["items_74-17444603", "items_74-19871524"]
}
```

### Example: store overview for a retailer not in the dropdown

```json
{
  "mode": "storeOverview",
  "retailerSlug": "costco",
  "customRetailerSlug": "sprouts-farmers-market"
}
```

### Example: discover a retailer's departments/aisles

```json
{
  "mode": "listAisles",
  "retailerSlug": "costco",
  "maxItems": 200
}
```

### Example: browse a department/aisle for on-sale items

```json
{
  "mode": "browseAisle",
  "retailerSlug": "costco",
  "aisleSlug": "dynamic_collection-sales",
  "onSaleOnly": true,
  "orderBy": "priceAsc",
  "maxItems": 50
}
```

### Example: highly-rated products in a specific aisle

```json
{
  "mode": "browseAisle",
  "retailerSlug": "safeway",
  "aisleSlug": "produce",
  "minRating": 4.5,
  "maxItems": 30
}
```

## Use cases

- **Price monitoring** — track grocery price changes across retailers over time
- **Market research** — compare pricing and product assortment between chains
- **Deal-hunting tools** — feed a price-comparison app or browser extension with `onSaleOnly` results
- **Retail analytics** — monitor stock availability and pricing trends by department/aisle
- **Coupon / cashback apps** — identify current prices and discount % to calculate savings
- **Category audits** — use `listAisles` + `browseAisle` to pull a full department's catalog

## FAQ

**Is this affiliated with Instacart?**
No — this is an independent, third-party actor that reads Instacart's public storefront pages. It is not affiliated with, endorsed by, or supported by Instacart.

**Do I need an Instacart account or login?**
No. All data comes from Instacart's public, unauthenticated storefront pages, the same pages any visitor sees when browsing without signing in.

**Why do prices show a specific location's pricing?**
Instacart prices are location-based. Without a saved delivery address, the actor uses Instacart's default guest location (typically the San Francisco Bay Area). Prices may differ from your local store.

**Why doesn't a retailer I searched for show up in results?**
Only retailers that partner with Instacart are available. Use `customRetailerSlug` with the retailer's exact Instacart storefront slug (found in the URL `instacart.com/store/<slug>/storefront`) if it's not in the dropdown.

**How many results does `search` mode return?**
Instacart's search returns a curated set of best-match results (similar to what a shopper sees on the first search screen) rather than a deep paginated list — typically 10–30 relevant items per query, capped by `maxItems`.

**How fresh is the pricing data?**
Prices are fetched live at run time directly from Instacart's storefront.

**What does `productId` vs `legacyId` mean?**
Both are Instacart's own internal identifiers for the same product; some downstream Instacart surfaces reference one or the other, so both are included for convenience.

**Why do some `productDetail` lookups return an item without price fields?**
Instacart resolves each guest session to a default region/store. If an `itemId` was originally captured from a `search`/`storeOverview` run in a different region than the current run resolves to, Instacart may return the item's catalog details without live pricing for that ID. For guaranteed pricing, use `itemIds` captured from a `search` or `storeOverview` result in the same run, or re-run `search`/`storeOverview` first to get current-session item IDs.

**Why does `productUrl` return a 403 with a bare `curl` (no headers)?**
Instacart blocks requests with no User-Agent header at all. Any normal HTTP client (browsers, `requests`, `curl -A "Mozilla/5.0 ..."`) works fine — this is standard bot-mitigation, not a broken link.

**How do I find a valid `aisleSlug` for `browseAisle`?**
Run `mode=listAisles` first for your chosen retailer — it returns every department and sub-aisle slug Instacart exposes for that storefront (they differ per retailer). You can also copy a slug directly from a collection page URL: `instacart.com/store/<retailer>/collections/<slug>`.

**Do `rating` and `reviewCount` appear on every product?**
No — Instacart only shows a public star rating for products that have received enough reviews. When it's not shown on the storefront, the actor omits the field rather than guessing a value.

**What counts as "on sale" (`onSale` / `onSaleOnly`)?**
An item is marked `onSale: true` when Instacart's storefront shows a crossed-out regular price (`regularPriceString`) alongside the current price. Retailer-wide deals can be browsed directly via `mode=browseAisle` with `aisleSlug: "dynamic_collection-sales"`.

**Are there aisles/collections this actor can't browse?**
`browseAisle` covers every department and sub-aisle slug returned by `mode=listAisles` for a given retailer. Deeper, personalized, or login-gated collections (e.g. order history-based recommendations) aren't included since they require an authenticated account.
