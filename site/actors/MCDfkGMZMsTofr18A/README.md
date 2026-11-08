# IKEA Multi-Country Product Catalog Extractor

IKEA sells through separate storefronts in more than 50 markets, and each storefront has its own currency, language, catalog, and stock signals. This actor reads those storefronts directly and returns one clean record per article: price history, rating, measurements, category path, images, color variants, and — on demand — designer, materials, care instructions, package dimensions, and live availability.

No API key, no login, and no proxy configuration are needed.

## What it does

- **Search by keyword** on any supported storefront, with real offset paging through every result IKEA reports.
- **Browse a category** — all products, furniture, sofas, storage, lighting, kitchen, outdoor, and 25+ other departments, also fully paged.
- **Look up article numbers** directly, including dotted and spaced article formats.
- **Fetch product URLs** in any locale; the URL's country and language are respected.
- **Enrich from product pages** to add designer, materials, care instructions, package measurements, availability, and a full description.
- **Filter** by price range, minimum rating, in-stock status, sale status, merchandising badges, or a keyword, and **sort** by relevance, price, rating, popularity, name, or dimensions.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `byProductIds`, or `byUrls` |
| `searchQuery` | string | `bookcase` | Keyword for `search` mode, e.g. `sofa`, `BILLY`, `kallax` |
| `productIds` | array | `[]` | IKEA article numbers for `byProductIds`, e.g. `20522046` |
| `productUrls` | array | `[]` | Full product-page URLs for `byUrls`, one per line |
| `category` | string | – | Category key for `byCategory` |
| `country` | string | `us` | Market storefront, ISO 3166-1 alpha-2 (50 markets) |
| `language` | string | `en` | Storefront language; falls back to the market's primary language |
| `sortBy` | string | `RELEVANCE` | `PRICE_LOW_TO_HIGH`, `PRICE_HIGH_TO_LOW`, `NEWEST`, `RATING`, `MOST_POPULAR`, `NAME_ASCENDING`, `WIDTH`, `HEIGHT`, or `DEPTH` |
| `minPrice` | integer | – | Exclude products priced below this value in the storefront currency |
| `maxPrice` | integer | – | Exclude products priced above this value in the storefront currency |
| `minRating` | integer | – | Exclude products rated below 0-5; unrated products are kept |
| `inStockOnly` | boolean | `false` | Exclude products marked out of stock or not sellable online |
| `containsKeyword` | string | – | Keep only records whose text fields contain this substring |
| `onSaleOnly` | boolean | `false` | Keep only products with a previous price or a discount |
| `onlyNew` | boolean | `false` | Keep only products IKEA tags as new |
| `onlyBestsellers` | boolean | `false` | Keep only products IKEA tags as best sellers |
| `onlyLastChance` | boolean | `false` | Keep only products IKEA tags as last chance |
| `enrichFromPDP` | boolean | `false` | Fetch each product page for designer, materials, care, packages, and availability |
| `maxItems` | integer | `50` | Maximum records to emit, 1-10,000 |

## Output per product

- `productId`, `sku`, `mpn` — IKEA article identifiers
- `name`, `typeName`, `itemType`, `filterClass` — product naming and classification
- `description` — full product-page description when the page is fetched
- `brand` — always `IKEA`
- `color`, `colors` — selected finish and every advertised finish
- `designer` — product designer from the product page
- `materials` — list of `{ part, material }` entries
- `careInstructions` — list of care steps
- `goodToKnow` — list of `{ name, text }` assembly and usage notes
- `measurementsText` — compact measurement summary from listings
- `measurements` — structured `height`, `width`, `depth`, and `weight` when published
- `packageMeasurements` — list of package `width`, `length`, `height`, `weight`, and `volume`
- `numberOfPackages`, `minOrderQuantity`
- `price` — `{ amount, currency }` plus `previousAmount`, `discountPercentage`, `discountAmount`, and `previousPriceLabel` when on sale
- `rating`, `ratingCount`
- `availability` — online-sellable flag, delivery stock status, click-and-collect eligibility, discontinued/sold-out flags, and check time
- `stockStatus` — schema.org-style stock label from the product page
- `onlineSellable` — listing-level sellability flag
- `mainImageUrl`, `imageUrls`, `mainImageAlt`, `contextualImageUrl`, `contextualImageAlt`
- `productUrl`
- `categoryPath` — breadcrumb names from the listing or product page
- `badges`, `badgeText`, `lastChanceText`, `isBestseller`, `isNewArrival`, `isLastChance`
- `quickFacts` — type/value facts such as material or firmness
- `variants` — every color/size sibling with its own ID, URL, image, rating, price, and sellability
- `variantCount`
- `country`, `language` — storefront used for this record
- `scrapedAt` — ISO 8601 collection time

Empty fields are omitted at every depth; the actor never sends `null`, `""`, `[]`, or `{}`.

## Examples

Search with a price window and rating floor:

```json
{
  "mode": "search",
  "searchQuery": "sofa",
  "country": "us",
  "minPrice": 300,
  "maxPrice": 900,
  "minRating": 4,
  "maxItems": 100
}
```

Browse a category with full detail enrichment:

```json
{
  "mode": "byCategory",
  "category": "st002",
  "country": "de",
  "language": "de",
  "sortBy": "PRICE_LOW_TO_HIGH",
  "enrichFromPDP": true,
  "maxItems": 50
}
```

Fetch exact articles:

```json
{
  "mode": "byProductIds",
  "productIds": ["20522046", "00263850"],
  "country": "gb"
}
```

Fetch product URLs in their own locales:

```json
{
  "mode": "byUrls",
  "productUrls": [
    "https://www.ikea.com/us/en/p/billy-bookcase-white-20522046/",
    "https://www.ikea.com/de/de/p/billy-buecherregal-braun-nussbaumnachbildung-50508652/"
  ]
}
```

## Pagination and limits

Search and category results are requested page by page from IKEA's own listing API, using its `window.offset` cursor and `metadata.max` total. Requests stop when `maxItems` is reached, IKEA's own count is exhausted, or a page comes back empty. The practical ceiling is what the storefront reports for your query; broad terms such as `storage` can exceed 2,000 products and full catalog browse (`category: "products"`) exceeds 9,000 in the US. Results are de-duplicated by article number.

## Legacy inputs

Saved tasks that still send older key names keep working: `query`/`search` → `searchQuery`, `ids`/`productId`/`articleNumbers` → `productIds`, `urls`/`startUrls` → `productUrls`, `categoryId`/`categorySlug` → `category`, `countryCode` → `country`, `languageCode` → `language`, `sort`/`sortOrder` → `sortBy`, `limit`/`maxResults` → `maxItems`, `inStock`/`availableOnly` → `inStockOnly`, `keyword` → `containsKeyword`, and `enrich`/`fetchDetails` → `enrichFromPDP`. These aliases are mapped in code and are not visible schema fields.

## FAQ

**Do I need a proxy or an API key?** No. The actor calls IKEA's public storefront endpoints directly. If IKEA ever rate-limits a run, the actor retries with backoff.

**Why do some records have more fields?** Designer, materials, care instructions, package measurements, and availability come from a product's own page. They are always present for `byProductIds` and `byUrls`, and added to `search`/`byCategory` results only when `enrichFromPDP` is true.

**Why is availability missing on listing records?** Listing pages do not publish it. Enable `enrichFromPDP` to fetch each product page and fill it.

**What happens with an invalid language?** It falls back to the selected market's primary language rather than failing.

**Why did my category key stop working?** IKEA occasionally renames taxonomy keys. The schema lists the current top-level keys; a full `/cat/` URL is also accepted and normalized.

**Why are prices different across countries?** Each record uses its storefront's local price and currency. Run the same products across countries to compare.

**Is this affiliated with IKEA?** No. It reads publicly accessible storefront data and is not affiliated with, endorsed by, or sponsored by Inter IKEA Systems or any IKEA franchisee.

## Limitations

- Search returns only what a storefront reports for that query; there is no cross-market search in one request.
- A few category keys are market-specific; the schema lists keys that are valid in the major markets.
- `enrichFromPDP` adds one request per product, so very large enriched runs take longer.
- Availability is a point-in-time snapshot from the product page; it is not a live inventory reservation.

## Data source

IKEA's public storefront listing API and product pages. The actor is designed for catalog research, price monitoring, and assortment analysis.
