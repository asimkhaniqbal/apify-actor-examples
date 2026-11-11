# Nike Product Catalog, Colorway & Price Scraper

Pull Nike.com product data straight from Nike's own public catalog feeds. Search by keyword or style code, browse 40 curated categories in 18 marketplaces, paste any Nike search/category/product URL, or ask for full product pages with size stock, all colorways, images and review summaries. No account, cookies, API key or proxy setup is required.

## Modes

| Mode | What it does |
|---|---|
| `search` | Run one or more keyword searches (`air max`, `dunk low`, `CW2288-111`) against a chosen marketplace. |
| `browseCategory` | Walk one of 40 curated catalog categories: men's, women's and kids' shoes, clothing, accessories, Jordan, running, basketball, soccer, sale and more. |
| `productDetails` | Fetch full detail for product URLs or bare style/style-color codes. Both `/t/` product pages and `/u/` Nike By You pages are accepted. |
| `startUrls` | Paste Nike search (`/w?q=...`), category (`/w/...`) or product (`/t/...`, `/u/...`) links and the actor routes each one automatically. Locale prefixes such as `/gb/` set the marketplace. |

`startUrls` takes precedence over `mode` when non-empty, so a mixed list of search, category and product links works in one run.

## Inputs

| Field | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `search` | `search` / `browseCategory` / `productDetails` / `startUrls`. |
| `searchQueries` | string list | `["air max"]` | Keywords, product names or style codes; one search task per entry. |
| `country` | select | `US` | US, GB, DE, FR, JP, MX, ES, IT, NL, SE, KR, SG, TH, TW, ID, PH, MY, ZA. Applies to search, category and non-prefixed URLs. |
| `category` | select | – | Curated category for `browseCategory`. Works in every supported country. |
| `startUrls` | URL list | `[]` | Nike search, category or product links; overrides `mode`. |
| `productUrls` | string list | `[]` | Full product-page URLs for `productDetails`. |
| `styleCodes` | string list | `[]` | Style or style-color codes such as `CW2288-111` or `CW2288`. For Nike By You products use the 10-digit style-color code (e.g. `6484638358`). |
| `minPrice` / `maxPrice` | integer | – | Keep products inside a price range in the marketplace's currency. |
| `minDiscount` | integer | `0` | Keep products discounted by at least this percentage. |
| `onSaleOnly` | boolean | `false` | Only emit products with an active discount. |
| `includeAllColorways` | boolean | `false` | Emit every colorway as its own record instead of one representative card per style. |
| `includeProductDetails` | boolean | `false` | Enrich listing records with detail-page data (description, sizes, stock, images, all colorways). |
| `includeReviews` | boolean | `false` | Add rating, review count, review summary and the three most recent reviews. Requires product-detail data. |
| `gender` | select | `ANY` | Men / Women facet for search and pasted search URLs. Nike rejects this facet for TH and TW, so it is skipped there. |
| `productType` | select | `ANY` | Footwear / Apparel / Accessories / Equipment. |
| `attributeFilter` | select | `ANY` | Just In, Coming Soon, Best Seller or Sustainable Materials. |
| `filters` | string list | `[]` | Nike's own facet labels, e.g. `Sports: Running`, `Color: Black`, `Brand: Nike`, `Width: Wide`, `Size: 10`. Resolved against the live facet panel for each search/category. |
| `sortBy` | select | `featured` | Featured, Newest, Price low→high, Price high→low, Discount high→low. |
| `maxItems` | integer | `24` | Hard cap across all queries, categories and URLs (1–5000). |

Older task inputs are still accepted: `searchQuery`/`query`/`queries`, `categories`, `startUrl`, `productUrl`/`urls`, `styleColors`/`styleCode`/`sku`/`skus`, `maxProducts`/`maxResults`/`maxResultsPerScrape`/`results_wanted`/`max_items`/`limit`, `marketplace`/`market`, `sort`, `onSale`, `allColorways`, `audience`, `product_type`, `attribute`, `facets`/`filter`/`facet`, `includeDetails`/`fetchProductDetails`/`fetchDetails`/`scrapeProductDetails`, `reviews` and `scrapeMode` are mapped in code. They are intentionally not shown in the input form.

## Output fields

Listing records carry `recordType`, `sourceUrl`, `scrapedAt`, `marketplace`, `language` and the query/category/URL that produced them (`sourceQuery`). Product-detail records carry `recordType`, `sourceUrl`, `scrapedAt` and `styleColor`.

### Listing records (`search`, `browseCategory`, `startUrls`)

- Identifiers: `productCode`, `styleCode`, `colorCode`, `styleGroupId`, `globalProductId`, `internalPid`, `merchProductId`
- Copy: `title`, `subtitle`, `fullTitle`
- Product type: `productType`, `productSubType`, `isCustomizable`
- Color: `colorLabel`, `colorHex`, `colorDescription`, `colorwayCount` (with `includeProductDetails`, this matches the length of the detail `colorways[]`; otherwise it is the number of buyable colorways in Nike's grid grouping)
- Price: `currentPrice`, `initialPrice`, `currency`, `discountPercentage`, `onSale`, `employeePrice`, `employeeDiscountPercentage`
- Media: `imageUrl`, `portraitImageUrl`
- Merchandising: `badgeLabel`, `featuredAttributes`, `isNew`, `isComingSoon`, `isBestSeller`, `isSustainableMaterials`, `newUntilDate`, `promotionTitle`
- Links: `productUrl`, `sourceUrl`

### Product-detail records

All listing fields that Nike publishes on the product page, plus:

- `styleColor`, `pdpUrl`, `canonicalUrl`
- `description`, `featuresAndBenefits`, `productDetails`
- `brands`, `genders`, `sportTags`, `taxonomyProductTypes`, `taxonomy`
- `sizes` with `label`, `localizedLabel`, `status`, `merchSkuId`, `gtin`
- `inStock`, `availableSizeCount`
- `images` (up to 12), `imageUrl`
- `colorways` with each color's `styleColor`, `colorCode`, `colorDescription`, `imageUrl`, `portraitImageUrl`, `productUrl`, `statusModifier`
- `colorwayCount`
- `availabilityDate`, `isLaunchProduct`, `consumerReleaseType`, `statusModifier`
- `badge`, `featuredAttributes`, `isNBY`, `isMemberProduct`, `netQuantity`, `manufacturingCountriesOfOrigin`
- `fitRecommendation`, `sizeChartUrl`, `isSustainableMaterials`

### Review fields (when `includeReviews=true`)

`averageRating`, `reviewCount`, `averageSizeRating`, `averageComfortRating`, `reviewSummary`, and `reviews[]` with `rating`, `title`, `text`, `author`, `createdAt`, `verifiedPurchase`, `purchasedFrom`.

Empty values are omitted at every depth. A field is present only when Nike actually publishes it for that product.

## Examples

Search two keywords, sort by price and keep the 60 cheapest matches:

```json
{
  "mode": "search",
  "searchQueries": ["air max", "pegasus"],
  "country": "US",
  "sortBy": "priceLowToHigh",
  "maxItems": 60
}
```

Browse the UK men's running category and enrich each listing with sizes and stock:

```json
{
  "mode": "browseCategory",
  "category": "mensRunningShoes",
  "country": "GB",
  "includeProductDetails": true,
  "maxItems": 40
}
```

Fetch two products by URL and one by style code, with colorways and reviews:

```json
{
  "mode": "productDetails",
  "productUrls": [
    "https://www.nike.com/t/air-force-1-07-mens-shoes-jBrhbr/CW2288-111",
    "https://www.nike.com/u/custom-air-max-95-big-bubble-by-you-10002422/6484638358"
  ],
  "styleCodes": ["CT8529-108"],
  "includeReviews": true
}
```

Process a mixed list of saved Nike links:

```json
{
  "mode": "startUrls",
  "startUrls": [
    {"url": "https://www.nike.com/w?q=jordan"},
    {"url": "https://www.nike.com/gb/w/mens-sale-shoes-3yaepznik1zy7ok"},
    {"url": "https://www.nike.com/t/air-force-1-07-mens-shoes-jBrhbr/CW2288-111"}
  ],
  "maxItems": 100
}
```

Use Nike's own facets to narrow a category to one sport, colour and size:

```json
{
  "mode": "browseCategory",
  "category": "mensShoes",
  "filters": ["Sports: Running", "Color: Black", "Size: 10"],
  "maxItems": 40
}
```

Track discounted footwear:

```json
{
  "mode": "search",
  "searchQueries": ["running shoes"],
  "onSaleOnly": true,
  "minDiscount": 20,
  "productType": "FOOTWEAR",
  "sortBy": "discountHighToLow",
  "maxItems": 50
}
```

## What people use it for

- Price and discount monitoring across Nike's catalog and 18 storefronts
- Colorway and assortment tracking for sneaker releases
- Size and stock checks before buying or restocking
- Building product feeds for comparison pages and affiliate sites
- Review and rating snapshots for product research

## Data source and reliability

The actor reads the same public APIs that power Nike's own search, category and product pages. It does not log in, store cookies, solve challenges or depend on a proxy. Requests are retried on 403/429/5xx with backoff, and the run reports a clean empty result or a clear status message instead of inventing rows.

Nike's wall API has no working public sort parameter, so `sortBy` is applied to the fetched records locally. For very large categories, raise `maxItems` and let the actor page through the source until it runs out or hits the cap.

## FAQ

**Do I need cookies, an account or a proxy?** No. Every supported field is public and cookieless. The actor never asks for credentials.

**Why does `browseCategory` now work outside the US?** The curated category paths are stable US catalog paths, and Nike's public wall API accepts them with any supported marketplace/language pair. Prices and URLs come back localized.

**What is the difference between `productCode` and `styleCode`?** `productCode` identifies a single style-color (for example `CW2288-111`); `styleCode` is the style family (`CW2288`) shared by its colorways.

**How do I get every colorway?** Set `includeAllColorways=true` in listing modes, or use `productDetails` where `colorways` is always included.

**How fresh is stock data?** `productDetails` reflects the product page at scrape time, including per-size `ACTIVE`/`OUT_OF_STOCK` status and the available-size count. Nike can change stock between requests.

**Can I search by a style code or style-color code?** Yes. Put it in `searchQueries` or `styleCodes`; both resolve through Nike's public search and return the matching product. Nike By You pages are matched by their 10-digit style-color code (for example `6484638358`), not by the shorter style code shown on the page.

**Why are only three reviews returned?** Nike's public review endpoint exposes a rating summary, an AI summary and the three most recent reviews per style. The separate review actor in the Store covers deep review pagination.

**Why is a field missing from one record?** Nike does not publish every attribute for every product (for example, a size chart on an accessory). Empty values are omitted rather than pushed as null or sentinel text.

**What happens when a filter leaves nothing?** The run succeeds with an empty dataset and a status message explaining that no records matched; it never fabricates a placeholder row.

**Does the daily test hit the cheapest path?** Yes. The saved prefill is a two-item `air max` search with detail and review enrichment off.
