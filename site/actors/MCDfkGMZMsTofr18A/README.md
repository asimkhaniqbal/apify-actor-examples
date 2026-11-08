# IKEA Multi-Country Product Catalog Extractor

IKEA runs separate storefronts for each of its 50+ country markets, each with its own pricing, language, and stock status. This actor searches or browses any one of those storefronts and returns structured product records — search by keyword, browse a category, or fetch exact products by ID or URL, with an optional deeper pass over each product's own page for measurements and description.

## What this actor does

- Searches a chosen country/language storefront by keyword, or browses one of twenty top-level categories
- Fetches exact products directly by IKEA article number or by full product-page URL
- Optionally visits each product's own page (`enrichFromPDP`) to add description, measurements, material, and full category breadcrumb
- Sorts results by relevance, price, rating, popularity, name, or dimension, and filters by price range, minimum rating, in-stock status, or a keyword
- Omits any field IKEA's page doesn't publish for that product rather than sending it empty

## Output per product

- `productId` — IKEA's article number
- `name`, `typeName` — product name and product type (e.g. `Bookcase`)
- `description` — only present when fetched from the product page (`byUrls`, `byProductIds`, or `enrichFromPDP`)
- `sku`, `mpn` — present on product-page fetches
- `brand` — `IKEA`
- `color`, `material`, `filterClass`
- `measurementsText` — a short measurement summary from search/category listings
- `measurements` — a structured `height`/`width`/`depth`/`weight` object, present on product-page fetches
- `price` — `{ amount, currency }`
- `rating`, `ratingCount`
- `mainImageUrl`, `imageUrls`
- `productUrl`
- `stockStatus` — present on product-page fetches
- `onlineSellable` — `true`/`false`, present on search/category listings
- `badges` — e.g. `TOP_SELLER`, `NEW_PRODUCT`, `LAST_CHANCE`, present on search/category listings
- `isBestseller`, `isNewArrival` — booleans derived from `badges`, present on search/category listings
- `variants` — other colors/sizes of the same product, each with `productId`, `name`, `color`, `imageUrl`, `productUrl`, and price, present on search/category listings when IKEA lists variants
- `variantCount`
- `categoryPath` — breadcrumb list, present on product-page fetches
- `country`, `language` — the storefront used for this record
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byProductIds`, `byUrls`, or `byCategory` |
| `searchQuery` | string | `bookcase` | Free-text search query (mode=search) |
| `productIds` | array | `[]` | IKEA article numbers, e.g. `20522046` (mode=byProductIds) |
| `productUrls` | array | `[]` | Full IKEA product page URLs (mode=byUrls) |
| `category` | string | – | A top-level category slug; required for `byCategory`. Ignored in `search` mode |
| `country` | string | `us` | IKEA market storefront, ISO 3166-1 alpha-2 code (50 supported markets) |
| `language` | string | `en` | Storefront language; falls back to the country's primary language if invalid for that market |
| `sortBy` | string | `RELEVANCE` | `RELEVANCE`, `PRICE_LOW_TO_HIGH`, `PRICE_HIGH_TO_LOW`, `NEWEST`, `RATING`, `MOST_POPULAR`, `NAME_ASCENDING`, `WIDTH`, `HEIGHT`, or `DEPTH` |
| `minPrice` | integer | – | Exclude products priced below this value, in the storefront's currency |
| `maxPrice` | integer | – | Exclude products priced above this value, in the storefront's currency |
| `minRating` | integer | – | Exclude products rated below this value (0-5); unrated products are kept |
| `inStockOnly` | boolean | `false` | Exclude products that are out of stock or unavailable online |
| `containsKeyword` | string | – | Only emit products whose name, description, color, or category contains this substring |
| `enrichFromPDP` | boolean | `false` | Also fetch each product's own page for description, measurements, and category breadcrumb (slower) |
| `maxItems` | integer | `50` | Maximum number of products to return (1-1000) |

### Example: search with price and rating filters

```json
{
    "mode": "search",
    "searchQuery": "bookcase",
    "country": "us",
    "minRating": 4,
    "maxPrice": 300,
    "maxItems": 30
}
```

### Example: browse a category, enriched with full detail

```json
{
    "mode": "byCategory",
    "category": "sofas-armchairs-fu002",
    "country": "de",
    "language": "de",
    "sortBy": "PRICE_LOW_TO_HIGH",
    "enrichFromPDP": true,
    "maxItems": 50
}
```

### Example: fetch exact products by article number

```json
{
    "mode": "byProductIds",
    "productIds": ["20522046", "00263850"],
    "country": "gb"
}
```

## Use cases

- **Price monitoring** — track price and stock status for a known list of article numbers across one or more country markets
- **Catalog syncing** — pull a full category with measurements and images to populate a product database
- **Localization research** — fetch the same product across multiple `country`/`language` combinations to compare pricing and availability
- **Assortment analysis** — browse a category sorted by rating or popularity to see which products perform best
- **Variant discovery** — collect `variants` from search results to map every color/size option of a product line

## FAQ

**Why do some products have `description` and `measurements` while others don't?** Those fields only come from a product's own page. They're always present for `byProductIds` and `byUrls` (which fetch that page directly), and only added to `search`/`byCategory` results when `enrichFromPDP` is set to `true`.

**What happens if I set an invalid `language` for a `country`?** The actor falls back to that country's primary storefront language rather than failing the run.

**Does `category` work the same way in `search` and `byCategory` modes?** No — `category` is only used in `byCategory` mode, where it selects which category page is browsed. In `search` mode it is ignored; narrow results with your `searchQuery` wording instead.

**Why is `onlineSellable` sometimes missing?** It's only included when IKEA's listing explicitly reports the flag; it's omitted rather than guessed when the source doesn't provide it.

**What are `badges`?** They're IKEA's own merchandising labels, such as `TOP_SELLER` or `NEW_PRODUCT`; `isBestseller` and `isNewArrival` are convenience booleans derived from that same list.

**Why are prices in different currencies across runs?** Pricing reflects whatever currency the selected `country` storefront uses; run the same input across different countries if you need a cross-market comparison.

**Is this affiliated with IKEA?** No. This actor reads IKEA's publicly accessible storefront data and is not affiliated with, endorsed by, or sponsored by Inter IKEA Systems or any IKEA franchisee.
