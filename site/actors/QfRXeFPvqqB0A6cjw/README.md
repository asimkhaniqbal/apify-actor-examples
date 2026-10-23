# Shopee Shop Profile, Item & Category Scraper

Pull public storefront data from **Shopee** across six Southeast Asian markets — Malaysia, Singapore, Thailand, the Philippines, Vietnam, and Indonesia. Get shop profiles (ratings, followers, response time), a shop's product feed, full item detail pages (prices, discounts, ratings, sold counts, stock, variations, images, category paths), and curated category listings. No login or cookies required.

## What this actor does

- **Four modes:** `shopProducts` (shop profile + product feed), `itemByUrl` (full item detail), `browseByCategory` (curated categories), `shopProfile` (shop profile only)
- **6 markets:** `MY` / `SG` / `TH` / `PH` / `VN` / `ID` domains with correct local currencies (MYR, SGD, THB, PHP, VND, IDR)
- **Shop profiles:** name, rating, followers, item count, response time, official/verified badges, description, account age
- **Item details:** price range, original price, discount %, sold & 30-day sold, stock, ratings, likes, variations, images, category path, listing date
- **Filters:** min/max price, minimum rating, keyword in title — applied to the item lists of `shopProducts` and `browseByCategory`
- **Anti-bot resilience:** when Shopee blocks an item feed, the mode still emits its public record (shop profile or category) instead of failing; if the very first request is blocked and you supplied a proxy, the actor retries once through it
- **Empty fields are omitted**

## Output

### Per shop (`recordType: "shop"`)

- `shopId`, `userid` (shop owner's numeric user ID), `shopName`, `username`, `country`
- `ratingStar`, `rating_good`, `rating_bad`, `rating_normal` (rating-tier counts; kept in Shopee's own snake_case)
- `followerCount`, `itemCount`, `responseTimeSeconds`
- `createdTimestamp`, `lastActiveTimestamp`, `cancellationRate`
- `isOfficialShop`, `isShopeeVerified`, `isPreferredPlusSeller`, `showOfficialShopLabel`
- `hasFlashSale`, `hasInShopFlashSale`, `hasBrandSale`, `hasJoinedMembership`
- `showLiveTab`, `isSemiInactive`, `vacation`, `isFollowed`, `status`
- `description`, `sourceUrl`, `scrapedAt`, `recordType`

### Per item (`recordType: "item"`)

Items from a shop feed or a category feed carry:

- `itemId`, `shopId`, `title`
- `shopName`, `shopUsername` — only on items from a shop feed (`shopProducts`)
- `price`, `priceMin`, `priceMax`, `priceOriginal`, `discountPercent`, `currency`
- `sold` (historical sold), `soldLast30Days`, `stock`
- `ratingStar`, `ratingCount`, `likeCount`
- `categoryId`, `category` (name, for the curated top-level categories), `location`, `listedTimestamp`
- `images[]`, `imageCover` (Shopee image URLs)
- `sourceUrl`, `scrapedAt`, `recordType`

An item fetched with `itemByUrl` has the same fields (without `shopName`/`shopUsername`) plus:

- `description`, `categoryPath` (e.g. `Electronics > Audio`), `itemStatus`
- `variations[]` (e.g. `Colour: Red, Blue, Green`; up to 10 option groups)
- `attributeIds[]` — Shopee's numeric attribute IDs for the item (up to 20)

### Per category (`recordType: "category"`)

- `categoryId`, `category`, `sourceUrl`, `scrapedAt`, `recordType`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `shopProducts` | `shopProducts` / `itemByUrl` / `browseByCategory` / `shopProfile` |
| `shopUrl` | string | `appleflagship.store` | Shop username or page URL (mode=shopProducts/shopProfile) |
| `itemUrl` | string | – | Item page URL (mode=itemByUrl) |
| `category` | string | – | Curated Shopee category (26 options, mode=browseByCategory) |
| `domain` | string | `MY` | `MY` / `SG` / `TH` / `PH` / `VN` / `ID` |
| `minPrice` | int | – | Drop items below this (local currency); feed modes only |
| `maxPrice` | int | – | Drop items above this (local currency); feed modes only |
| `minRating` | number | `0` | Keep items rated at least this (0–5); unrated items are kept. Feed modes only |
| `containsKeyword` | string | – | Keep items whose title contains this text (case-insensitive); feed modes only |
| `maxItems` | int | `50` | Hard cap on all records, including the shop or category record (1–500) |
| `proxyConfiguration` | object | – | Optional Apify proxy, used for one retry when the first request of the run is blocked |

With no input changes, the actor runs `shopProducts` for the `appleflagship.store` shop on the Malaysian site, because those are the schema defaults.

### Example: shop products (Malaysia)

```json
{
  "mode": "shopProducts",
  "shopUrl": "appleflagship.store",
  "domain": "MY",
  "maxItems": 30
}
```

### Example: shop profile only

```json
{
  "mode": "shopProfile",
  "shopUrl": "https://shopee.sg/sony.os",
  "domain": "SG"
}
```

### Example: item detail by URL

```json
{
  "mode": "itemByUrl",
  "itemUrl": "https://shopee.com.my/extended-waterproof-gloves--i.169104911.6601357248",
  "domain": "MY"
}
```

### Example: category browse with filters

```json
{
  "mode": "browseByCategory",
  "category": "Mobile & Accessories",
  "domain": "MY",
  "minPrice": 50,
  "maxPrice": 2000,
  "minRating": 4.5,
  "containsKeyword": "samsung",
  "maxItems": 40
}
```

## Use cases

- **Marketplace monitoring** — track shops, products, prices and discounts across SEA markets
- **Brand intelligence** — follower growth, rating trends and item counts for official stores
- **Competitor pricing** — price ranges, original prices and discount patterns per category
- **Product research** — ratings, sold counts and stock for demand estimation
- **Localized data pipelines** — currency-aware (MYR/SGD/THB/PHP/VND/IDR) e-commerce datasets

## FAQ

**What is the data source?** Shopee (shopee.com.my / .sg / .co.th / .ph / .vn / .co.id). This actor is a third-party tool and is not affiliated with or endorsed by Shopee.

**Why can't I scrape keyword search results?** Shopee gates keyword search behind a login wall for automated access. This actor deliberately avoids search and uses the public storefront surfaces instead — shop profiles, shop product feeds, item pages and category browse.

**Why did I get only a shop profile (or category) record and no items?** Shopee intermittently blocks its item feeds for automated visitors. The actor emits the public shop or category record first, then attempts the feed; if the feed is blocked, the run ends with just that record. Running again later, or with a proxy located in the target market, often gets the items. (The proxy input is only tried automatically when the shop profile, category list or item itself is blocked.)

**Do I need an account or cookies?** No. Everything this actor reads is public.

**What currency are prices in?** The market's local currency: MYR (MY), SGD (SG), THB (TH), PHP (PH), VND (VN), IDR (ID).

**What are `priceMin`/`priceMax` vs `price`?** Shopee items can show a price range (e.g. size-based). The range is kept in `priceMin`/`priceMax`; `price` is the displayed card price (range minimum when they differ).

**Why are some fields missing?** Empty fields are omitted by design — e.g. items without a discount have no `priceOriginal`/`discountPercent`, unrated items have no `ratingStar`, and `description`, `variations` and `categoryPath` only come with `itemByUrl`.

**What happens when nothing can be fetched?** The dataset stays empty and the run's status message explains why (shop not found, invalid item URL, or a block).

**How fresh is the data?** Live — shop profiles, items and categories are fetched at run time.

## Limitations

- Shopee occasionally blocks product-feed requests from data-center IPs. When that happens the mode still returns its public record (the shop profile, or the category record) instead of failing.
- Shop and item data can be gated by the requesting IP's region — a non-local IP may only see the `MY` market reliably. Use a proxy located in the target market for the other five domains.
- Prices are in the market's local currency (MYR/SGD/THB/PHP/VND/IDR). Range-priced items keep `priceMin`/`priceMax` separate from the single `price` shown on the card.
- Category IDs are shared across Shopee's six domains; the 26 curated categories are top-level ones.
- Each feed is read in pages of 24 items, for at most 40 pages.
