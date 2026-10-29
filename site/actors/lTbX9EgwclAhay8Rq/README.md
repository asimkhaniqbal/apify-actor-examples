# Ozon.ru Product Pricing, Ratings & Seller Scraper

Scrape [Ozon.ru](https://www.ozon.ru), Russia's largest e-commerce marketplace, in three modes: keyword search, category browsing, and full product-detail lookups by URL or SKU. Get pricing (including Ozon Card / discount pricing), ratings with a full 5-star breakdown, review counts, full seller profiles (name, storefront URL, rating, lifetime orders), complete spec/characteristic tables, structured category breadcrumbs, other colors/configurations of the same product, similar/alternative products, buyer Q&A counts, competing-seller offers, image galleries, product videos, promo/marketing labels, delivery estimates — ready for price monitoring, market research, and catalog enrichment.

## What this actor does

- **Three modes:** `search`, `byCategory`, `productDetails`
- **Universal product lookup:** full Ozon product URLs or bare numeric SKU/product IDs
- **Filters:** price range, minimum rating, in-stock only
- **Sort control:** relevance, price (asc/desc), rating, newest (search mode)
- **Empty fields are omitted** — every record only contains data that was actually found

## Output per product

| Field | Description |
|---|---|
| `sku` | Ozon product/SKU ID |
| `name` | Product title |
| `brand` | Brand name (productDetails mode, when Ozon exposes a distinct brand widget) |
| `price` | Current price in RUB |
| `originalPrice` | Pre-discount price in RUB, if the product is on sale |
| `discountPercent` | Computed discount percentage |
| `cardPrice` | Ozon Card member price, when different from `price` |
| `currency` | Always `RUB` when pricing is present |
| `rating` | Average rating (0–5) |
| `reviewCount` | Number of reviews |
| `ratingBreakdown` | Star-rating distribution as `[{stars, count, percentage}, ...]` for 5 down to 1 stars (productDetails mode) |
| `sellerName` | Seller/shop name |
| `sellerUrl` | Seller's Ozon storefront URL (productDetails mode) |
| `sellerRating` | Seller/shop rating, 0–5 (productDetails mode) |
| `sellerOrdersCount` | Seller's lifetime order count, as shown on-site e.g. `"12.3 K"` (productDetails mode) |
| `category` | Category breadcrumb trail as a single ` > `-joined string (productDetails mode) |
| `breadcrumbs` | Same trail as structured `[{text, url}, ...]`, each with its own category-page URL (productDetails mode) |
| `characteristics` | Full spec table as `{characteristic name: value}` (productDetails mode) |
| `variants` | Other colors/configurations of this exact product — each a distinct SKU with its own `sku`, `name`, `price`, `originalPrice`, `imageUrl`, `productUrl`, `availability` (productDetails mode) |
| `questionCount` | Number of buyer Q&A questions posted for the product (productDetails mode) |
| `otherSellersCount` | Number of other sellers listing the same product cheaper/faster, when Ozon shows a comparison banner (productDetails mode) |
| `otherSellersMinPrice` | Lowest price among those other sellers, in RUB (productDetails mode) |
| `imageUrl` | Primary product image |
| `images` | All gallery images (productDetails mode) |
| `videos` | Product video URLs (e.g. unboxing/demo clips), when Ozon's gallery carries one — most products don't have one, so this is often absent (productDetails mode) |
| `marketingLabels` | Promo badge texts shown on the page (e.g. cashback offers, "points for a review"), when present (productDetails mode) |
| `similarProducts` | Ozon's own "similar/alternative products" shelf — each a real, distinct SKU with `sku`, `name`, `price`, `originalPrice`, `discountPercent`, `imageUrl`, `productUrl`, `availability` (productDetails mode) |
| `availability` | Stock status — either a status text (e.g. "70 шт осталось") or the canonical `outOfStock` when the exact SKU has sold out |
| `availabilityText` | Ozon's own human-readable out-of-stock message, e.g. "Этот товар закончился" (productDetails mode, only when out of stock) |
| `deliveryInfo` | Delivery estimate text (productDetails mode) |
| `productUrl` | Canonical Ozon product page URL |
| `recordType` | `product` |
| `scrapedAt` | UTC ISO timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byCategory` / `productDetails` |
| `searchQuery` | string | – | Keyword search (mode=search) |
| `sortBy` | string | – | `price` / `-price` / `rating` / `new` (mode=search). Leave empty (default relevance) for the tightest topical match — see FAQ note below on non-default sorts + deep pagination |
| `categoryUrl` | string | – | Category URL or slug (mode=byCategory) |
| `productUrls` | array | – | Product URLs or bare SKUs (mode=productDetails) |
| `minPrice` / `maxPrice` | integer | – | RUB price range filter |
| `minRating` | number | – | Minimum rating (0–5) |
| `inStockOnly` | boolean | `false` | Drop products marked out of stock |
| `maxItems` | integer | `20` | Hard cap on emitted records (1–500) |
| `proxyConfiguration` | object | RESIDENTIAL / RU | Required — see FAQ for why |

### Example: search with filters

```json
{
  "mode": "search",
  "searchQuery": "телевизор",
  "sortBy": "price",
  "minPrice": 10000,
  "minRating": 4,
  "maxItems": 30
}
```

### Example: browse a category

```json
{
  "mode": "byCategory",
  "categoryUrl": "https://www.ozon.ru/category/smartfony-15502/",
  "maxItems": 50
}
```

### Example: fetch full product details

```json
{
  "mode": "productDetails",
  "productUrls": ["https://www.ozon.ru/product/apple-iphone-15-128gb-blue-123456789/", "987654321"]
}
```

## Use cases

- **Price monitoring** — track price and discount changes on competitor SKUs
- **Market research** — analyze ratings, review volume, and seller mix by category
- **Catalog enrichment** — pull structured product data by SKU for your own listings
- **Assortment analysis** — browse a category to see full product mix and pricing spread
- **Deal discovery** — sort by price/rating to surface the best-value listings

## FAQ

**Why does this actor require a residential proxy?** Ozon blocks non-Russian datacenter traffic at the network edge with a JS/CAPTCHA challenge before the storefront ever loads. A Russian residential IP passes this check transparently, so a residential proxy pinned to Russia is required input for every run — the actor fails fast if none is configured.

**Can I search in English?** Ozon's catalog is primarily Russian-language; search terms in Latin script (e.g. brand names, model numbers like "iphone") work well, but broader English keyword queries may return fewer results than the equivalent Russian term.

**What currency are prices in?** Russian rubles (RUB) — Ozon's only supported currency.

**Why is `cardPrice` sometimes missing?** It only appears when Ozon shows a separate Ozon Card member price distinct from the regular price.

**Can I look up a product by just its numeric ID?** Yes — `productUrls` accepts bare numeric SKU/product IDs as well as full product URLs.

**What are `variants`?** Other colors, storage sizes, or configurations Ozon sells under the same product listing (e.g. the same phone in a different color) — each entry is its own real, purchasable SKU with its own price and URL. This is not a generic "customers also viewed" recommendation list, just the same product's own selectable variants.

**Are `characteristics` the full spec sheet?** They're Ozon's own "About this product" panel — the same characteristic table shown on the product page. Coverage varies by category/product; sparse listings will have fewer entries.

**What is `ratingBreakdown`?** The 5-star-to-1-star distribution of a product's reviews, straight from Ozon's own rating widget — useful for spotting products with a polarized or thin review base that a single average rating hides.

**Why do a few results with `sortBy` set (e.g. `rating`) not match my keyword?** For a narrow query, Ozon's own site may only have a handful of genuine matches under a non-default sort order. Once its infinite-scroll widget runs out of true sorted matches, Ozon itself — not this actor — continues the same feed with its own "recommended for you" filler products to keep the scroll going, and the actor has no reliable way to tell filler from a real match at that point (page-level `mainState`/`state` shape is identical). This only shows up when a run needs several pages under a non-empty `sortBy` (e.g. combined with `minPrice`/`maxPrice`/`minRating`/`inStockOnly` that reject many cards, or a high `maxItems`); default relevance search (`sortBy` left empty) reliably stays on-topic for the whole result set in testing. If strict topical relevance matters more than sort order, leave `sortBy` empty and sort the returned dataset yourself.

**What are `similarProducts`?** Ozon's own "similar products" recommendation shelf — real, independently purchasable SKUs (not our guess), most commonly shown as substitutes when the requested product has sold out, but also present on some regular in-stock pages.

**Why does an out-of-stock product still return `name`/`price`/`sellerName`?** Ozon renders a dedicated "this item sold out" page in place of the normal product layout, but that page still carries the real last-known name, price, seller, and image for the exact SKU — the actor reads that data rather than returning an almost-empty record. `availability` will read `outOfStock` and `availabilityText` carries Ozon's own message in these cases.

**What are `videos` and `marketingLabels`?** `videos` are real product video URLs from Ozon's own gallery (brand-supplied unboxing/demo clips) — only a minority of listings have one. `marketingLabels` are the short promo badges Ozon shows on the product page (e.g. cashback or "get points for a review" offers) — informational only, not a parsed price/discount value.

**What if a category or search term has no matches?** The run completes successfully with 0 records and a status message explaining why — this is expected behavior, not a failure.

**How fresh is the data?** Real-time — every run reads Ozon's live storefront data at request time.

**Is this affiliated with Ozon?** No. This is an independent actor that reads Ozon.ru's public storefront pages and is not operated or endorsed by Ozon.
