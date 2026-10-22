# AliExpress Multi-Region Search, Store & Review Scraper

Pull products, store profiles, and customer reviews from AliExpress across 18 regional storefronts and 42 currencies, by keyword search, numeric ID, or a pasted AliExpress URL. No AliExpress login or cookies are needed.

## What this actor does

- Searches AliExpress by keyword and returns up to 60 product cards per page, with price, discount, rating, and order-count data
- Fetches product detail, store profile, or customer review records by numeric ID, or auto-detects the right mode from a pasted AliExpress URL
- Supports 18 regional storefronts, 42 currency codes, and 18 UI languages, plus sort order, price range, rating, and ship-from/ship-to filters for keyword searches
- With `autoEscalateOnBlock` on (the default), switches to Apify's residential proxy mid-run if AliExpress blocks or rate-limits a request
- Empty fields are omitted from every record

## Output: per product (`mode=search`)

- `productId`, `title`, `productUrl`, `imageUrl`
- `salePrice` — `{amount, currencyCode, formatted, discountPercent}`
- `originalPrice` — `{amount, currencyCode, formatted}`, present when the item is discounted
- `discountPercent`, `currencyCode`
- `rating` — star rating out of 5
- `ordersText`, `ordersCount` — e.g. `"5,000+ sold"` and its parsed number
- `sellingPoints[]` — promo lines shown on the card (e.g. free shipping threshold)
- `listedAt`, `productType`, `itemType`, `shipFromCountry`, `region`
- `recordType` (`product`), `siteName`, `scrapedAt`
- `reviews[]` — attached review objects (same fields as the review record below), only when `enrichReviews` is on and `maxReviewsPerProduct` is above 0

## Output: per product (`mode=byProduct` or `byUrl` on an item link)

- `productId`, `productUrl`, `canonicalUrl`, `title`, `imageUrl`, `description`
- `salePrice` — `{amount, currencyCode}`, `currencyCode`
- `rating`, `ratingCount` — from the page's structured product data, when present
- `storeId`, `storeUrl`, `sellerAdminSeq`
- `region`, `recordType` (`product`), `siteName`, `scrapedAt`
- `reviews[]` — attached review objects (same fields as the review record below), only when `enrichReviews` is on and `maxReviewsPerProduct` is above 0

If a product page can't be loaded, the record still appears but carries only `productId`, `productUrl`, `region`, `recordType`, `siteName` and `scrapedAt` (plus any reviews).
## Output: per store (`mode=byStore` or `byUrl` on a store link)

- `storeId`, `storeUrl`, `storeName`, `storeLogoUrl`
- `sellerId`, `storeNumber`, `openedAt`, `positiveFeedbackRate`
- `region`, `recordType` (`store`), `siteName`, `scrapedAt`

## Output: per review (`mode=byReviews` or `byUrl` on a feedback link)

- `productId`, `reviewId`, `rating` (1–5)
- `reviewText` (AliExpress's translated text when it provides one, otherwise the original), `reviewerName`, `reviewerCountry`, `reviewerHeadUrl`
- `reviewedAt`, `skuInfo`
- `imageUrls[]`, `helpfulCount`
- `recordType` (`review`), `siteName`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byProduct`, `byStore`, `byReviews`, or `byUrl` (auto-detects from the URL) |
| `searchQuery` | string | – | Free-text query (mode=search) |
| `productIds` | array | – | Numeric product IDs (mode=byProduct or byReviews) |
| `storeIds` | array | – | Numeric store IDs (mode=byStore) |
| `urls` | array | – | AliExpress search, item, store, or feedback URLs (mode=byUrl) |
| `region` | string | `com` | Regional storefront, e.g. `com`, `us`, `de`, `ja` (for `byUrl`, the region comes from each URL's host instead) |
| `currency` | string | `USD` | ISO-4217 currency code sent as a locale hint; AliExpress may still answer in another currency |
| `language` | string | `en_US` | UI locale AliExpress should render text in |
| `sortBy` | string | `default` | `default`, `orders_desc`, `newest`, `price_asc`, or `price_desc` (mode=search only) |
| `priceMin` | integer | `0` | Minimum price in the selected currency (mode=search only); `0` disables the lower bound |
| `priceMax` | integer | `0` | Maximum price (mode=search only); `0` disables the upper bound |
| `ratingMin` | number | `0` | Minimum star rating (0–5, decimals allowed; mode=search only). Products with no rating are kept |
| `shipFrom` | string | `""` (any) | ISO 3166-1 alpha-2 warehouse country (mode=search only) |
| `shipTo` | string | `""` (any) | ISO 3166-1 alpha-2 destination country; filters keyword searches and is also sent as the delivery region on product and store pages |
| `maxItems` | integer | `25` | Hard cap on emitted records (1–1000) |
| `maxPages` | integer | `3` | Maximum search result pages to paginate through (1–60), for `search` and search URLs in `byUrl` |
| `enrichReviews` | boolean | `false` | Attach up to `maxReviewsPerProduct` reviews to each product record |
| `maxReviewsPerProduct` | integer | `0` | Review cap per product when `enrichReviews` is on (0 = no reviews attached). In `byReviews` mode, and for feedback URLs in `byUrl`, 0 means 20 reviews per product |
| `useProxy` | boolean | `false` | Route requests through Apify proxy |
| `autoEscalateOnBlock` | boolean | `true` | Switch to Apify residential proxy automatically if a request is blocked or rate-limited |
| `proxyConfiguration` | object | `{"useApifyProxy": true, "apifyProxyGroups": []}` | Proxy settings, used when `useProxy` is enabled |

### Example: keyword search with filters

```json
{
    "mode": "search",
    "searchQuery": "phone case",
    "region": "com",
    "currency": "USD",
    "sortBy": "orders_desc",
    "priceMax": 10,
    "ratingMin": 4,
    "shipFrom": "CN",
    "maxItems": 50,
    "maxPages": 2
}
```

### Example: enrich specific products with reviews

```json
{
    "mode": "byProduct",
    "productIds": ["1005010155028387"],
    "enrichReviews": true,
    "maxReviewsPerProduct": 20
}
```

### Example: resolve a pasted AliExpress URL

```json
{
    "mode": "byUrl",
    "urls": ["https://www.aliexpress.com/store/1101493050"]
}
```

## Use cases

- **Price tracking** — monitor sale price and discount percentage on a set of known products over time
- **Market research** — search a product keyword across regions to compare pricing and order volume
- **Seller due diligence** — pull a store's profile and positive-feedback rate before sourcing from it
- **Voice-of-customer analysis** — collect review text and ratings for a product line
- **Catalog building** — resolve a batch of AliExpress item/store/search URLs in one run via `byUrl`

## FAQ

**Do I need an AliExpress account or cookies?**
No. Every mode reads AliExpress's public pages and reviews without authentication.

**Do I need a proxy?**
Not usually. The actor works from a standard IP for typical volumes. `autoEscalateOnBlock` is on by default and switches to Apify's residential proxy mid-run if AliExpress rate-limits a request; enable `useProxy` up front for sustained, high-volume scraping.

**Why does `byProduct` return fewer fields than `search`?**
An AliExpress product page only carries a short summary when it's first opened — title, main image, and sometimes price, rating and store — and that summary is what the actor records. For the richest per-product fields (price, discount, orders, selling points), use `mode=search`.

**Why might `currencyCode` differ from the `currency` I requested?**
AliExpress determines currency partly from the requesting IP's location. For deterministic currency, pair `currency` with `useProxy` and a residential proxy in a country that uses your target currency.

**Can I scrape a category page?**
AliExpress's category URLs redirect to the matching keyword search — pass the category name as `searchQuery` instead for the same results.

**How many results can one run return?**
At most `maxItems` records in total (up to 1,000). Searches stop after `maxPages` result pages, so raise both for larger pulls. Reviews are collected page by page up to the per-product cap.

**Do my filters apply to pasted search URLs?**
No. In `byUrl` mode a search URL is re-run with its keyword only, in the storefront of its host; `sortBy`, price, rating and ship-from filters are not applied there. Feedback URLs return up to 20 reviews per product unless `maxReviewsPerProduct` is set.

**Why did a run on a less common region return 0 records?**
Some regional storefronts occasionally serve a search page with no product list, or block the request. When that happens the run finishes with 0 records and a status message explaining likely causes instead of failing. Retrying, or switching to the `com` storefront, usually helps.

**Is this affiliated with AliExpress?**
No. This is an independent actor that reads AliExpress's public pages and is not operated or endorsed by AliExpress or Alibaba Group.
