# Yahoo Auctions Japan Live & Sold Auction Scraper

Yahoo Auctions Japan (ヤフオク!) is the country's biggest consumer auction site. This actor turns its public listings into clean JSON: live auctions with current bids and buy-it-now prices, closed auctions with the price they actually sold for, seller storefronts, and individual item pages with descriptions and full photo sets.

No Yahoo account, cookie, API key or proxy setup is needed.

## What you can scrape

| Mode | What it returns |
|---|---|
| `keywordSearch` | Live listings for one or more keywords |
| `browseCategory` | Live listings from a category ID, optionally narrowed by keyword |
| `soldSearch` | Closed auctions with final sold prices — useful for sold-price comparisons |
| `sellerListings` | Live listings from one or more seller IDs |
| `itemDetails` | Full detail pages for a list of item IDs or item URLs |

Every mode supports price bounds, condition grade, listing type, prefecture, free-shipping and buy-it-now filters where Yahoo exposes them. Keyword modes also accept excluded words.

## Record fields

Prices are whole yen (`JPY`). Empty fields are omitted rather than written as `null`.

| Field | Meaning |
|---|---|
| `itemId` | Yahoo auction ID |
| `itemUrl` | Direct link to the listing |
| `title` | Listing title |
| `currentPrice` | Current bid, or the final price on sold rows |
| `soldPrice` | Final winning price on closed auctions |
| `priceType` | `current` or `final` |
| `buyItNowPrice` | Buy-it-now price, when offered |
| `startPrice` | Starting price |
| `currency` | Always `JPY` |
| `bids` | Bid count (`0` is kept) |
| `watchCount` | Number of watchers |
| `biddersCount` | Distinct bidders (detail pages) |
| `condition`, `conditionCode` | English condition and Yahoo's grade code |
| `conditionNameJa` | Yahoo's Japanese condition label (detail pages) |
| `startDate`, `endDate` | ISO 8601 timestamps |
| `timeRemainingSeconds` | Seconds left on live listings |
| `seller`, `sellerId`, `sellerUrl` | Seller identity and storefront link |
| `sellerDisplayName`, `sellerRating`, `sellerRatingCount` | Seller reputation |
| `sellerLocation`, `prefecture`, `prefectureCode` | Item location |
| `category`, `categoryId`, `categoryPath` | Category name, ID and hierarchy |
| `brand` | Brand path, when Yahoo lists one |
| `imageUrl`, `imageUrls` | Main image and full photo set |
| `shippingFee`, `shippingFeeType`, `freeShipping` | Shipping cost and who pays |
| `shippingMethods`, `shipSchedule`, `shippingPaidBy` | Delivery options |
| `listingType` | `auction` or `fixedPrice` |
| `marketplace` | `yahooAuctions` or `paypayFleaMarket` |
| `status` | `open` or `closed` |
| `quantity`, `returnable`, `isWorldwideDelivery` | Item terms |
| `description`, `descriptionHtml` | Full item description (detail pages) |
| `scrapedAt` | ISO 8601 scrape timestamp |

## Input reference

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `keywordSearch` | `keywordSearch`, `browseCategory`, `soldSearch`, `sellerListings` or `itemDetails` |
| `searchQueries` | array | – | Keywords, Japanese or English; each runs until `maxItems` |
| `excludeKeywords` | array | – | Remove listings containing any of these words |
| `categoryIds` | array | – | Numeric category IDs, e.g. `23336` for Computers |
| `sellerIds` | array | – | Seller IDs or seller page URLs (`sellerListings`) |
| `itemIds` | array | – | Auction IDs or item URLs (`itemDetails`) |
| `condition` | string | `any` | `any`, `new`, `used`, `likeNew`, `excellent`, `good`, `acceptable` or `poor` |
| `listingType` | string | `all` | `all`, `auction` or `fixedPrice` |
| `sortBy` | string | `relevance` | 14 orders: ending soonest/last, newest, price, bids, buy-it-now price, total price incl. shipping, watchers, relevance, featured |
| `minPrice` | integer | – | Minimum price in yen |
| `maxPrice` | integer | – | Maximum price in yen |
| `priceType` | string | `currentPrice` | Apply `minPrice`/`maxPrice` to `currentPrice` or `buyNowPrice` |
| `prefecture` | string | Any prefecture | Two-digit Japanese prefecture code; use the dropdown for names |
| `freeShippingOnly` | boolean | `false` | Keep only free-shipping listings |
| `buyItNowOnly` | boolean | `false` | Keep only listings with a buy-it-now price |
| `maxItems` | integer | `20` | 1 to 5,000 records per run |
| `includeDetails` | boolean | `false` | Open each item page for description, photos, bidder count, seller rating/location and shipping methods |

### Example: live keyword search

```json
{
  "mode": "keywordSearch",
  "searchQueries": ["PlayStation 5", "Nintendo Switch"],
  "condition": "used",
  "minPrice": 5000,
  "maxPrice": 40000,
  "sortBy": "endingSoon",
  "maxItems": 100
}
```

### Example: sold-price comparison

```json
{
  "mode": "soldSearch",
  "searchQueries": ["Leica M6"],
  "sortBy": "endingLast",
  "maxItems": 50,
  "includeDetails": true
}
```

### Example: browse a category

```json
{
  "mode": "browseCategory",
  "categoryIds": ["23336"],
  "listingType": "fixedPrice",
  "sortBy": "priceAsc",
  "maxItems": 200
}
```

### Example: item pages from a list of URLs

```json
{
  "mode": "itemDetails",
  "itemIds": [
    "w1250838886",
    "https://page.auctions.yahoo.co.jp/jp/auction/e1251357399"
  ]
}
```

## Pagination and limits

The actor follows result pages until `maxItems` is reached or Yahoo stops returning new records. Yahoo serves roughly 100 records per page and makes about 15,000 results browsable per search; very broad queries can hit that window. Split a broad search by category, price band or condition to reach deeper stock. Closed-auction history covers roughly the last 180 days.

## FAQ

**Do I need a Yahoo Japan login or API key?**
No. The actor reads publicly visible Yahoo Auctions Japan pages and the site's own public JSON endpoints.

**Are sold prices real winning prices?**
Yes — `soldSearch` reads closed auctions and reports the final price Yahoo records, not an estimate.

**Why do some rows have no buy-it-now price?**
Only fixed-price listings and auctions that offer instant purchase have one; the field is omitted otherwise.

**Can I search in Japanese?**
Yes. Pass Japanese keywords directly, exactly as you would type them into Yahoo's search box.

**How do I find a category or seller ID?**
Open the category or seller page on auctions.yahoo.co.jp and copy the numeric category ID or the seller ID from the URL. Seller and item URLs are accepted too.

**Why are some prices 0 or 1 yen?**
Those are live auctions with a low starting bid and no bids yet. Check `bids` and `startPrice` to tell them apart.

**Does this actor cover PayPay Flea Market?**
Closed searches can include flea-market rows from Yahoo's network; they are marked with `marketplace: "paypayFleaMarket"` and link to the flea-market item page. `itemDetails` covers Yahoo auction pages only.

**Is this affiliated with Yahoo Japan?**
No. This is an independent tool that reads publicly available pages and is not endorsed by LY Corporation.

## Data source

Public listing, category, seller, closed-auction and item pages on `auctions.yahoo.co.jp` (plus the site's own JSON endpoints). No login, cookies, credentials or user proxy configuration are required.
