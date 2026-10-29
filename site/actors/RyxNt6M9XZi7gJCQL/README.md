# Naver Shopping Price & Review Scraper

Collect **Naver Shopping** products by keyword — the displayed price and discount, rating and review count, competing seller count, delivery terms, purchase options, category IDs, image and product link. Results come from Naver's public mobile shopping search, with no account, cookie or API key required.

## What this actor does

- **Keyword search at scale**: one search per keyword, in Korean or English, with up to 90 product cards per keyword (ads included; ads-off runs return the organic subset)
- **30+ card-level fields**: price, original price, discount, rating, reviews, sellers, wishlist saves, delivery, variants, category IDs, images and links
- **Ads under your control**: ads take a large share of the up to 90 cards (16–50 in our samples), so ads-off runs return the organic subset; every record exposes `isAd`
- **Local price filter and sort**: keep a KRW price band and order the fetched set by relevance, price or review count
- **Clean records**: HTML highlight tags are stripped, and empty fields are omitted rather than sent as `null`

## Services and data source

The actor reads Naver's anonymous mobile integrated search page (`m.search.naver.com/search.naver?where=m_shop`). It does not use the login-gated shopping portal, so it needs no Naver account and no proxy configuration. Each keyword returns the product cards that Naver renders on that page, including marketplace cards, catalog cards and sponsored cards.

## How it works

For every keyword, the actor fetches the mobile search page once and reads the embedded shopping state. It flattens the product cards, removes duplicates by Naver product ID, drops ads when `includeAds` is false, applies your KRW price band, sorts the remaining products locally, and saves them up to `maxItems`. Multiple keywords are collected in one run, and a product found by more than one keyword is saved once.

The free mobile page has no pagination, server-side sort or server-side price filter. Sorting and filtering therefore apply to the products this run fetched, not to all of Naver's results.

## Output per product

- `productId` — Naver product ID (`nvMid`), the stable identifier
- `channelProductId` — the seller's own product ID, when the card has one
- `productName` — product title with Naver's `<mark>` highlight tags removed
- `productUrl` — direct seller or marketplace URL; falls back to the Naver catalog page for catalog cards
- `imageUrl` — product image from Naver's image CDN
- `mallId`, `mallName` — selling store ID and name; `mallId` is `naver_model` on catalog cards, where `mallName` is omitted
- `mallBadge` — store badge such as official or certified, when shown
- `isAd` — `true` for sponsored cards, `false` for organic cards
- `cardType` — Naver card type (`CATALOG_CARD`, `ORGANIC_CARD`, `AD_CARD`)
- `searchRank` — Naver's rank within the card's own section (ads and organic cards have separate rank sequences)
- `position` — consecutive position after filtering, sorting and de-duplication
- `keyword` — keyword that found the product
- `price` — displayed selling price in KRW
- `lowPrice` — same as `price` on this source
- `originalPrice` — list price before discount, when higher than `price`
- `discountPercent` — discount rate shown on the card, when greater than zero
- `highPrice` — highest price across sellers, only when a source card carries it
- `priceCurrency` — usually `KRW`
- `brand` — brand or maker, only when the source card carries it
- `sellerCount` — number of competing sellers, when greater than zero
- `reviewScore` — average review rating out of 5, when the product has reviews
- `reviewCount` — total review count, when greater than zero
- `purchaseCount` — recorded purchase count, when shown
- `wishlistCount` — number of shoppers who saved the product
- `purchaseOptions` — variants with `name` and `price`, when the card lists options
- `categoryId` — leaf category ID
- `categoryIds` — category IDs grouped as `l`, `m`, `s` and `ss` (large to small)
- `category` — category names, only when a source card carries them
- `deliveryFee` — delivery fee in KRW (`0` means free)
- `freeDelivery` — `true` when delivery is free, `false` when paid
- `fastDeliveryType` — Naver's fast-delivery label (`NONE`, `TODAY`, `ARRIVAL`, …)
- `isBrandStore` — `true` for official brand stores
- `isSmartStore` — `true` for SmartStore products
- `isOverseas` — `true` for overseas products
- `recordType` — `product` for every record
- `scrapedAt` — ISO 8601 timestamp for the run

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `searchQueries` | array | `[]` | One Naver Shopping search per keyword (Korean or English). The actor reads old single-keyword `keyword` tasks too. |
| `includeAds` | boolean | `true` | Keep sponsored cards. Every record is flagged with `isAd`. |
| `minPrice` | integer | – | Drop fetched products below this KRW price. |
| `maxPrice` | integer | – | Drop fetched products above this KRW price. |
| `sort` | string | `relevance` | `relevance` / `priceLow` / `priceHigh` / `reviews` — local sort of the fetched products. |
| `maxItems` | integer | `100` | Total products to save across all keywords (1–1000). |

### Example: three keywords, ads off

```json
{
    "searchQueries": ["스마트폰", "무선이어폰", "노트북"],
    "includeAds": false,
    "maxItems": 150
}
```

### Example: price band with cheapest first

```json
{
    "searchQueries": ["에어프라이어"],
    "minPrice": 50000,
    "maxPrice": 200000,
    "sort": "priceLow",
    "maxItems": 50
}
```

### Example: most-reviewed products for an English query

```json
{
    "searchQueries": ["iphone case"],
    "includeAds": false,
    "sort": "reviews",
    "maxItems": 30
}
```

### Example: competitive snapshot including ads

```json
{
    "searchQueries": ["강아지사료"],
    "includeAds": true,
    "sort": "relevance",
    "maxItems": 90
}
```

## Use cases

- **Price monitoring** — capture the current displayed price and discount for a list of Korean keywords
- **Assortment research** — see which stores and brands appear for a category search
- **Review intelligence** — sort a fetched set by rating volume to find established products
- **Delivery benchmarking** — compare free delivery, paid delivery and fast-delivery labels across sellers
- **Catalog enrichment** — attach Naver category IDs, images and product links to an existing feed
- **Campaign tracking** — keep ads in the result and separate them later with `isAd`

## Usage cost

You pay for Apify platform usage under your plan. The actor uses only Naver's public mobile pages: no paid data API, no residential proxy and no external credential. A typical keyword costs one page fetch, small inputs finish in seconds, and `maxItems` bounds the number of saved rows.

## Notes

- Each keyword returns at most **90 product cards**, including ads, because that is the complete set embedded in Naver's mobile page. With `includeAds` off you get the organic subset (roughly 40–75 cards, depending on how many ads Naver inserts).
- Ads are mixed into the same page; read `isAd` instead of assuming every row is organic.
- `productName` is Korean or English exactly as Naver shows it; there is no translation step. Naver sometimes includes double spaces, which are kept as shown.
- `lowPrice` mirrors `price`. `highPrice` is usually absent on this source because a single card shows one displayed price, not every seller's offer.
- When a keyword's page has no shopping module, that keyword yields zero rows and the run continues with the next keyword.
- A small number of carrier-subsidy catalog cards have no listed price at all; those records omit `price` rather than inventing one. When their delivery type and fee contradict each other, delivery fields are omitted too.
- Catalog-card `productUrl` links point at Naver's catalog page, which can redirect to a maintenance page outside Korea; the `productId` and `imageUrl` remain usable.
- The dataset is written once per run; `scrapedAt` is identical for all rows from that run.

## Limitations

- **90 cards per keyword, not 90 organic products.** Naver's mobile page embeds up to 10 pages of 9 cards, and ads take 16–50 of those cards. With `includeAds` off the source ceiling is the organic subset (roughly 40–75). The page exposes no next page or cursor, so there is no way to request more from this free surface; split a broad market into narrower keywords to cover more products.
- **No server-side pagination, sort or price filter.** `sort` and `minPrice`/`maxPrice` apply only to the cards fetched in the same run. They do not reorder or paginate Naver's full result set.
- **No product detail, review text, specifications or stock.** Those live on Naver's shopping portal and product pages, which are gated and return blocks outside Korea. This actor does not ship those modes.
- **No brand, maker or category names from search cards.** Brand and category names come from detail pages that are not available here. Category IDs are provided instead.
- **Some keywords return no cards.** Very broad category words can render neither the shopping module nor the explore module, so they produce zero rows.
- **A few catalog cards have no price.** They are carrier or catalog placeholders without a live offer, so the price fields are omitted. Drop them downstream if your workflow needs a price on every row.
- **The page shape can change.** The actor parses Naver's embedded shopping state; if Naver changes that structure, the run reports it instead of inventing records.

## FAQ

**What is Naver Shopping?**
Naver Shopping is South Korea's largest shopping search surface. It groups products from many Korean stores and shows price, reviews, delivery and seller information on one card.

**Why do I get at most 90 cards per keyword?**
Naver's mobile shopping page contains up to 10 embedded pages of 9 cards, so 90 cards is the source ceiling for each keyword. Ads take 16–50 of those cards, so with ads off you get the remaining organic cards (roughly 40–75). The page does not expose a next-page link or cursor. Run more specific keywords to cover a larger market.

**Why is `sort` described as a local sort?**
The free mobile page ignores sort parameters. The actor therefore sorts the products it fetched, which is useful for a single keyword and for a combined multi-keyword result, but it is not a global Naver ranking.

**Why are `brand`, `category` names or `highPrice` sometimes missing?**
Search cards do not carry a brand tag, a category name path or every seller's price. The actor omits those fields instead of guessing. Category IDs, seller count and the displayed price are still provided.

**Why is there no review text, star distribution, stock or specification list?**
Those fields exist only on Naver's product-detail and review pages. Those pages are gated and blocked from outside Korea, so this actor does not offer detail or review modes. The search card still gives the average rating and review count.

**Do I need a Naver account, cookies or a proxy?**
No. The actor uses Naver's anonymous mobile search page. There is nothing to log in to and nothing to configure.

**Can I include or exclude ads?**
Yes. Set `includeAds` to `true` or `false`. Every record also carries `isAd` so you can keep a mixed snapshot and filter it later.

**Is the data live?**
Yes. Every run reads the current mobile search page. `scrapedAt` records when the snapshot was taken. Naver may personalize or rotate cards over time, so counts can differ slightly between runs.

**Is this affiliated with Naver?**
No. This is an independent third-party actor that reads Naver's public search page. It is not affiliated with or endorsed by Naver Corporation.
