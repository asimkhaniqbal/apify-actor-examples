# Wholesale Marine Parts & Pricing Scraper

Pull structured product data from [Wholesale Marine](https://www.wholesalemarine.com), a US retailer of boat engine parts, propellers, electrical components, trailer parts, watersports gear and outdoor equipment. The actor talks directly to the store's public product API, so it returns full product records rather than just listing cards.

No account, cookies, API key or proxy setup is required.

## Ways to use it

- **Keyword search** — search the catalog by product name, part number, SKU or UPC. Add several queries in one run.
- **Category browse** — walk any category, including nested paths such as `boat-anchors/claw-anchors`, or paste a category URL.
- **Brand browse** — page through all products for a brand. The dropdown covers all 322 brands on the store; extra brand slugs are accepted.
- **Product URLs** — paste product links (or slugs) and get the full record for each one.
- **Full catalog** — stream the complete 190,000+ product catalog in newest, best-selling, featured or default order.

Every enriched result can include price, list price, inventory count, SKU/MPN/UPC/GTIN, brand, variants, specifications, fitment notes, gallery images, category breadcrumbs, weight and dimensions. Ratings and review counts are optional.

## Output fields

| Field | Type | Description |
|---|---|---|
| `productId` | string | Store product ID |
| `title` | string | Product name |
| `productUrl` | string | Canonical product page |
| `brand`, `brandUrl` | string | Brand name and brand page |
| `sku`, `mpn`, `upc`, `gtin` | string | Identifiers where the store publishes a valid value |
| `price`, `priceFormatted` | number/string | Current selling price (USD) |
| `listPrice`, `listPriceFormatted` | number/string | Original price when the item is discounted |
| `discountAmount`, `discountPercent` | number | Discount versus list price |
| `priceMin`, `priceMax` | number | Variant price range |
| `currency` | string | `USD` |
| `inStock` | boolean | Inventory availability |
| `stockQuantity` | integer | Units available to sell |
| `stockTracked` | boolean | Whether the store tracks inventory for the item |
| `availabilityText` | string | The store's own shipping/lead-time text (for example "Special Order") |
| `description` | string | Clean plain-text description |
| `descriptionHtml` | string | Cleaned description HTML, including spec/fitment tables (optional) |
| `images`, `primaryImage`, `imageUrl` | array/string | Gallery, main image and listing thumbnail |
| `categoryPath`, `categories` | array | Primary breadcrumb and all category assignments |
| `categorySlug` | string | Category used for a category run |
| `variantOptions` | array | Purchasable option names and values |
| `variantCount` | integer | Total number of variants (1 for simple products) |
| `variants` | array | Per-variant SKU, identifiers, price, list price, stock and selected options |
| `specifications` | object | Product attributes such as Material, Pitch, Voltage or Style |
| `fitmentNotes` | array | Free-text application/fitment notes published by the store |
| `replacesPartNumbers` | array | Interchangeable/replacement part numbers |
| `weight`, `weightUnit`, `dimensions` | number/string/object | Shipping weight and L×W×H dimensions |
| `condition` | string | Product condition when the store publishes one (for example `NEW`) |
| `warranty` | string | Warranty text when published in a dedicated field |
| `videos` | array | Product video title/URL pairs |
| `specificationTables` | array | Structured headers/rows from specification and application charts in the description |
| `ratingValue`, `reviewCount`, `ratingDistribution` | number/object | Customer rating summary from the store's review widget |
| `source`, `sourceValue` | string | Which query, category, brand or URL produced the row |
| `recordType`, `scrapedAt` | string | Always `product`; UTC scrape timestamp |

Empty values are omitted, never emitted as `null` or placeholders.

## Inputs

| Field | Type | Default | What it does |
|---|---|---|---|
| `mode` | select | `keyword_search` | `keyword_search`, `category_browse`, `brand_browse`, `direct_product_url` or `full_catalog` |
| `searchQuery` | string | `propeller` | Main keyword, part number or UPC |
| `extraSearchQueries` | array | `[]` | Additional keyword queries in the same run |
| `categoryPath` | select | — | 250 popular categories, including nested paths |
| `extraCategoryPaths` | array | `[]` | Any additional category slug, path or URL |
| `brandSlug` | select | — | All 322 store brands |
| `extraBrandSlugs` | array | `[]` | Any additional brand slug, name or URL |
| `productUrls` | array | `[]` | Product URLs or slugs for URL mode |
| `sortOrder` | select | `bestselling` | Best selling, featured, newest, A–Z, Z–A, price low/high, or best match |
| `fullCatalogSort` | select | `newest` | Order for full-catalog runs |
| `minPrice`, `maxPrice` | integer | — | USD price bounds |
| `inStockOnly` | boolean | `false` | Drop records whose inventory is out of stock |
| `includeDetails` | boolean | `true` | Enrich listing rows with API-only fields |
| `includeRatings` | boolean | `true` | Fetch rating/review summaries |
| `includeDescriptionHtml` | boolean | `false` | Include the cleaned description HTML |
| `maxItems` | integer | `20` | Total record cap (1–100,000) |
| `maxItemsPerQuery` | integer | `1000` | Cap per keyword query or category batch (1–10,000); brand/full runs use `maxItems` |

## Examples

Search by part number:
```json
{"mode":"keyword_search","searchQuery":"8M0097854","sortOrder":"relevance","maxItems":20}
```

Browse a subcategory sorted by price:
```json
{"mode":"category_browse","categoryPath":"boat-anchors/claw-anchors","sortOrder":"priceasc","inStockOnly":true,"maxItems":50}
```

Browse a brand, with a second brand and ratings:
```json
{"mode":"brand_browse","brandSlug":"Yamaha","extraBrandSlugs":["Mercury-Mercruiser"],"sortOrder":"bestselling","maxItems":100}
```

Fetch exact products:
```json
{"mode":"direct_product_url","productUrls":["https://www.wholesalemarine.com/johnson-pump-cartridge-bilge-pump/"]}
```

Pull the newest catalog slice:
```json
{"mode":"full_catalog","fullCatalogSort":"newest","maxItems":500,"includeRatings":false}
```

## Use cases

- **Price monitoring** — schedule category or brand runs and compare price, list price and discount over time.
- **MAP compliance** — compare a brand's live listings and identifiers against your own catalog.
- **Inventory and stock-out alerts** — watch `stockQuantity` and `inStock` for key SKUs.
- **Parts catalog enrichment** — fill SKU, MPN, UPC, specs, images and fitment notes in a PIM or storefront.
- **Dropshipping research** — export sellable products with live cost, stock and images.
- **Fitment-aware sourcing** — search engine parts by part number and read the store's application notes before ordering.

## FAQ

**Do I need a Wholesale Marine account or cookies?** No. The actor uses the public storefront and its public product API.

**Do I need to configure a proxy?** No. The actor handles its own connection fallbacks.

**How large is the catalog?** About 190,000 products across 1,162 categories and 322 brands.

**Why does a keyword search stop at 10,000 rows?** The store's search index has a 10,000-result ceiling. The actor partitions large requests by price to get past that ceiling when you ask for more.

**Does it return out-of-stock items?** Category and keyword pages follow the store's in-stock search index. Brand, full-catalog and URL runs can include out-of-stock items; set `inStockOnly` to drop them.

**Where do ratings come from?** From the same customer-review widget shown on product pages. Products without reviews omit the rating fields.

**How long does a full-catalog run take?** Roughly 25–30 minutes for all 190,000 products. Use `maxItems` for a smaller snapshot.

**Can I use a category or brand that is not in the dropdown?** Yes — use `extraCategoryPaths` or `extraBrandSlugs` with a slug, path, name or full URL.

**Is this affiliated with Wholesale Marine?** No. It is an independent data tool that reads publicly available storefront pages and APIs.

## Limitations

- The store's keyword and category pages expose at most 10,000 results per query; larger runs are split by price or child category.
- Full-catalog price/name sorting is not supported by the store API; use keyword, category or brand mode when you need those orders.
- `fitmentNotes` are free-text application notes. Full fitment charts are preserved in `descriptionHtml` rather than converted to a normalized table.
- Wholesale Marine is an online-only retailer, so there is no per-store inventory data.
- `availabilityText` is the store's shipping/lead-time message, not a stock flag; use `inStock` and `stockQuantity` for stock decisions.
- Price-filtered brand and full-catalog runs use the store's search index, which lists in-stock products; unfiltered brand/full runs include out-of-stock products unless `inStockOnly` is set.
- With a price range set, brand and full-catalog results follow the search index order rather than `sortOrder`/`fullCatalogSort`.
- The full catalog cursor reaches all 190,565 products, but one run is capped at 100,000 rows and the 190k pass takes longer than the default one-hour timeout. Split by category/brand for a complete snapshot, or raise the run timeout.
- `warranty` is only available when the store publishes a dedicated field; warranty prose inside a description is not converted. `condition` and `videos` are conditional (most listings are new and have no video).
- `specificationTables` and `fitmentNotes` expose the store's published charts/notes; there is no normalized fitment API.
- The category dropdown is curated to 250 paths; every one of the 1,162 storefront categories is accepted through `extraCategoryPaths`.
