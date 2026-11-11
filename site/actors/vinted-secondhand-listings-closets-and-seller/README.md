# Vinted Secondhand Listings, Closets & Seller Feedback

Vinted is a large second-hand fashion marketplace spread across country storefronts. This actor turns its public listings, item pages, seller profiles, closets, feedback history, brands and categories into clean dataset rows across 27 markets. Search by keyword with Vinted's own filters, paste filtered URLs, open full item details, or vet a seller before buying. No account, cookies or proxy setup is needed.

## What this actor does

- **Seven modes:** `search`, `byUrl`, `byUser`, `byBrand`, `byCatalog`, `byBrandSearch`, `byUserFeedbacks`
- **27 markets:** United States, United Kingdom, France, Germany, Italy, Spain, Poland, Czechia, Netherlands, Lithuania, Luxembourg, Austria, Belgium, Portugal, Romania, Slovakia, Finland, Sweden, Greece, Denmark, Estonia, Croatia, Hungary, Ireland, Latvia, Slovenia and Australia
- **Search filters:** keyword, brand, category, size, colour, condition, material, price range and sort order, combinable in one search
- **Deep search:** split a broad search into price bands to collect beyond Vinted's 960-result ceiling per query
- **Optional item details:** description, colour, material, category path, condition ID, upload age, every photo and seller rating
- **Optional seller stats:** rating, feedback counts, live and sold item counts, followers, business/shop identity
- **Seller feedback:** full review history with rating, text, reviewer and optional owner reply
- **Best-effort enrichment:** item details and seller stats are optional; under Vinted rate limits those fields can be omitted or delayed while the search rows still return
- **Empty fields are omitted** — every value in a row is real; fields Vinted does not expose are absent, not filled with placeholders

## Output per item

- `itemId`, `title`, `description`
- `brandTitle`, `brandId`, `brandUrl`
- `sizeTitle`, `sizeId`, `sizeEquivalents`
- `status` (condition as shown on the storefront), `condition` (English), `conditionId`, `statusId`
- `color`, `colors`, `materials`, `otherAttributes`
- `catalogId`, `categoryName`, `categoryPath`
- `priceAmount`, `priceCurrency`, `totalPriceAmount`, `serviceFeeAmount`, `shippingAmount`, `freeShipping`
- `favouriteCount`, `isPromoted`, `isReserved`, `isClosed`, `isHidden`, `isVisible`, `availability`
- `uploadedAgo` — Vinted's relative posting age (e.g. "2 hours ago")
- `itemUrl`, `canonicalUrl`, `imageUrl`, `photos[]`, `photoCount`
- `sellerId`, `sellerLogin`, `sellerCountryCode`, `sellerProfileUrl`, `sellerAvatarUrl`
- `sellerRating`, `sellerFeedbackCount`, `sellerFeedbackReputation`
- `sellerPositiveFeedbackCount`, `sellerNeutralFeedbackCount`, `sellerNegativeFeedbackCount`
- `sellerItemCount`, `sellerSoldItemCount`, `sellerBoughtItemCount`, `sellerFollowersCount`
- `sellerIsBusiness`, `sellerIsOnline`, `sellerBadges`
- `sellerShopName`, `sellerShopLegalName`, `sellerShopVatNumber`, `sellerShopCountry`, `sellerShop`
- `sourceUrl` when an item came from a pasted URL
- `recordType`, `scrapedAt`, `siteName`, `sourceCountry`, `sourceCountryCode`

Fields marked optional above appear only when the selected mode enriches the row or Vinted supplies them.

## Output per seller profile

- `userId`, `login`, `realName`, `about`, `city`, `countryCode`, `countryTitle`, `countryId`
- `avatarUrl`, `avatarThumbnailUrl`, `profileUrl`
- `rating` (0–5), `feedbackCount`, `feedbackReputation`, `feedbackRating`
- `positiveFeedbackCount`, `negativeFeedbackCount`, `neutralFeedbackCount`
- `itemCount`, `totalItemsCount`, `soldItemCount`, `boughtItemCount`
- `followersCount`, `followingCount`
- `isBusiness`, `isOnHoliday`, `isOnline`, `verifications`
- `shop` — name, legal name/code, VAT number, registrar, country, address and verification state for business sellers
- `lastLoggedOn`, `recordType`, `scrapedAt`, `siteName`, `sourceCountry`, `sourceCountryCode`

One profile row is written before that seller's closet items.

## Output per feedback entry

- `feedbackId`, `feedbackText`, `rating`, `createdAt`
- `itemId`, `itemUrl`
- `feedbackUserId`, `reviewerId`, `reviewerLogin`, `reviewerProfileUrl`
- `isSystem`, `isAutomatic`, `feedbackType` (`member` or `automatic`)
- `ownerComment`, `ownerLogin`
- `recordType`, `scrapedAt`, `siteName`, `sourceCountry`, `sourceCountryCode`

## Output per brand

- `brandId`, `title`, `slug`
- `favouriteCount`, `itemCount`, `currency`
- `isVisibleInListings`, `requiresAuthenticityCheck`, `isLuxury`, `isHighValue`
- `brandUrl`, `canonicalUrl`
- `recordType`, `scrapedAt`, `siteName`, `sourceCountry`, `sourceCountryCode`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byUrl`, `byUser`, `byBrand`, `byCatalog`, `byBrandSearch` or `byUserFeedbacks` |
| `country` | string | `com` | Market suffix, e.g. `com`, `co.uk`, `fr`, `de`, `pl` |
| `searchText` | string | – | Keyword (mode=search) |
| `order` | string | `relevance` | `relevance`, `newest_first`, `price_low_to_high`, `price_high_to_low` |
| `urls` | array | – | Item, member, brand, catalog or filtered search URLs (mode=byUrl) |
| `userId` | string | – | Numeric user ID, username or member URL (mode=byUser / byUserFeedbacks) |
| `brandId` | string | – | Brand ID or brand URL (mode=byBrand) |
| `catalogId` | string | – | Category ID or catalog URL (mode=byCatalog) |
| `brandKeyword` | string | – | Brand registry keyword (mode=byBrandSearch) |
| `brandIds` | array | – | Brand ID filters |
| `catalogIds` | array | – | Category ID filters |
| `sizeIds` | array | – | Size ID filters |
| `color` | string | `any` | Vinted colour ID; the dropdown shows labels such as Black, Gray, White, Cream, Beige, Red, Blue, Green, Brown, Yellow, Silver, Gold and Multi |
| `materialIds` | array | – | Material ID filters |
| `condition` | string | `any` | `new_with_tags`, `new_without_tags`, `very_good`, `good`, `satisfactory` |
| `priceFrom` / `priceTo` | integer | – | Price range in the market's local currency |
| `deepSearch` | boolean | `false` | Split broad searches into price bands past 960 results; a saturated single-price band is treated as truncated |
| `includeItemDetails` | boolean | `false` | Add description, attributes, all photos and seller rating from item pages |
| `includeSellerStats` | boolean | `false` | Add seller rating, counts, followers and shop identity |
| `minRating` | number | – | Drop items below this seller rating (0–5); items without a rating pass |
| `containsKeyword` | string | – | Keep items whose title or description contains this text |
| `maxItems` | integer | `50` | Hard cap (1–5000) |
| `feedbackType` | string | `all` | `all`, `member` or `automatic` (mode=byUserFeedbacks) |

### Example: search a brand with filters

```json
{
  "mode": "search",
  "country": "fr",
  "searchText": "nike air max",
  "brandIds": ["53"],
  "sizeIds": ["207"],
  "condition": "very_good",
  "priceFrom": 20,
  "priceTo": 80,
  "maxItems": 100
}
```

### Example: item details and seller stats

```json
{
  "mode": "search",
  "country": "de",
  "searchText": "carhartt jacket",
  "includeItemDetails": true,
  "includeSellerStats": true,
  "maxItems": 25
}
```

### Example: a seller's profile, closet and feedback

```json
{
  "mode": "byUrl",
  "country": "co.uk",
  "urls": [
    "https://www.vinted.co.uk/member/123456789-seller-name"
  ],
  "maxItems": 50
}
```

```json
{
  "mode": "byUserFeedbacks",
  "country": "fr",
  "userId": "123456789",
  "feedbackType": "member",
  "maxItems": 100
}
```

### Example: deep search past 960 results

```json
{
  "mode": "search",
  "country": "pl",
  "searchText": "nike",
  "deepSearch": true,
  "maxItems": 3000
}
```

## Use cases

- **Resale sourcing** — scan a brand, size or category for underpriced listings before they sell
- **Price monitoring** — track asking prices and buyer-protection totals across markets over time
- **Seller vetting** — pull a seller's rating, sold count, feedback history, followers and shop identity
- **Market research** — measure listing volume, condition mix and price bands for a brand or category
- **Closet monitoring** — schedule a seller's closet to spot new stock as it appears
- **Brand and category discovery** — resolve Vinted brand IDs and category paths for saved searches

## FAQ

**Do I need a Vinted account or cookies?**
No. Every mode reads publicly available Vinted pages and endpoints. No login, API key or cookie setup is required.

**How many markets are supported?**
27 live Vinted markets, each with its own currency and storefront language. Use the `country` dropdown; pasted URLs use their own domain.

**How many results can I get?**
Vinted returns at most 960 results per query. Turn on `deepSearch` to split a broad query into price bands and collect beyond that ceiling. If a single price point still has more than 960 matches, that band cannot be fully enumerated and is treated as truncated, so deep coverage is broader than 960 but not exhaustive. Closets are capped by Vinted at 10,000 visible items per seller.

**Why is the posting date relative instead of an exact timestamp?**
Vinted stopped exposing an exact listing creation time. The actor returns the relative `uploadedAgo` value the site shows and does not invent a date. View counts are no longer public either, so neither field is fabricated.

**Why do some items lack a brand, material, colour, description or seller rating?**
Sellers can leave optional fields blank, and new sellers have no feedback yet. Item-page and seller-stat enrichment is best-effort: if Vinted rate-limits those routes, the row keeps its search fields and omits the enrichment fields instead of inventing values. Seller-stat enrichment stops after repeated API failures, so a rate-limited run can take several minutes and finish with fewer seller counts. Optional fields are omitted rather than filled with a placeholder.

**Why is `status` in a local language while `condition` is in English?**
Vinted localizes condition names per market. `status` keeps the storefront wording, while `condition` and `conditionId` provide a stable cross-market equivalent. The ID is the reliable key for filtering.

**Can I scrape sold listings or sold prices?**
No. Vinted removes sold listings from public search and exposes no complete sold feed. Seller `soldItemCount` and review rows with an item ID are useful sale signals, but they are not a sold-price history.

**Do the same listings appear in several markets?**
Some markets share one inventory pool and show the same item at converted prices. Match rows by `itemId`; a price difference between markets is often currency conversion, not a separate listing.

**Do images open without authentication?**
Yes. Image URLs are public and load without cookies or a referrer.

**Is this affiliated with Vinted?**
No. This is an independent third-party actor that reads publicly accessible marketplace data. It is not affiliated with or endorsed by Vinted.
