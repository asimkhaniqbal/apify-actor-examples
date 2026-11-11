# eBay Multi-Marketplace Listing & Seller Data Scraper

Collect structured eBay data from eight country sites without cookies, logins, API keys or proxy setup. Five modes cover the whole buyer-side surface: keyword search, category browse, item details, seller feedback profiles, and store inventories. Every row is server-rendered data from the live site with prices, conditions, seller ratings, shipping, returns, item specifics and pagination across US, UK, Germany, France, Italy, Spain, Canada and Australia.

## What can you scrape?

| Mode | What it returns | Good for |
|---|---|---|
| **Keyword search** | Listings matching one or more queries, with filters and up to 240 rows per page | Price monitoring, assortment tracking, resale research |
| **Category browse** | Listings inside a category ID or `/b/` URL, optionally narrowed by a keyword | Category-wide price scans and new-arrival feeds |
| **Item details** | Full item page: JSON-LD product data, item specifics, seller, shipping, returns, images | Product enrichment and catalog building |
| **Seller profile** | Feedback score, positive percentage, 1/6/12-month positive-neutral-negative counts, member-since, store badge and recent feedback | Seller vetting, lead scoring, competitor research |
| **Store listings** | Active listings from a store or seller, enriched with store name, tagline, feedback percentage, items sold and followers | Store monitoring and inventory snapshots |

## Data source and access

The actor reads eBay's public server-rendered pages. eBay fronts its sites with Akamai Bot Manager, so a plain request from many IP ranges receives a generic error page. The actor first tries eBay directly, then transparently falls back to the public Google Translate website mirror of the same page. The mirror carries the identical DOM, JSON-LD, prices and currencies, so no data changes. There are **no cookies, no login, no API key and no Apify Proxy group** in the default path; a `proxy` input from older saved tasks is accepted and ignored.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `browse`, `item`, `seller` or `stores` |
| `searchQueries` | string list | – | Keywords for search mode. One search per entry |
| `categories` | string list | – | Category IDs (for example `177`), `/b/` URLs or `_sacat` search URLs for browse mode |
| `browseKeyword` | string | – | Optional keyword to narrow a category |
| `itemIds` | string list | – | Item IDs or `/itm/` URLs for item mode |
| `sellerUsernames` | string list | – | Seller usernames or `/usr/` URLs for seller mode |
| `storeNames` | string list | – | Store names or `/str/` URLs for stores mode |
| `startUrls` | URL list | – | Shortcut: paste any eBay URLs and each is routed by type. Mixed search, category, item, seller and store URLs run together and share `maxItems` |
| `marketplace` | select | `US` | `US`, `GB`, `DE`, `FR`, `IT`, `ES`, `CA`, `AU` |
| `condition` | select | `""` | All, new, open box, manufacturer refurbished, seller refurbished, used, for parts |
| `buyingFormat` | select | `""` | Both, Buy It Now only, auctions only |
| `minPrice` / `maxPrice` | number | – | Price range in the marketplace currency |
| `sortBy` | select | `bestMatch` | Best match, price low/high, ending soonest, newly listed |
| `freeShipping` | boolean | false | Keep free shipping/delivery listings only |
| `returnsAccepted` | boolean | false | Keep listings that accept returns |
| `itemLocation` | select | `""` | Any location, domestic only, worldwide |
| `maxItems` | integer | 50 | Record cap across all sources (1–10,000) |
| `maxPages` | integer | 10 | Page cap per search/category/store (1–200) |
| `includeItemSpecifics` | boolean | true | Add the item-specifics table to item records |
| `includeRecentFeedback` | boolean | true | Add up to 25 recent feedback rows to seller records |

Legacy keys are mapped in code, not shown in the form: `keywords` and `query` become `searchQueries`, `startUrls` are routed by URL type, and `proxy` is ignored.

## Examples

### Keyword search with filters

```json
{
  "mode": "search",
  "searchQueries": ["thinkpad", "macbook pro"],
  "marketplace": "GB",
  "condition": "used",
  "buyingFormat": "buyItNow",
  "minPrice": 200,
  "maxPrice": 900,
  "sortBy": "priceLowest",
  "maxItems": 120,
  "maxPages": 2
}
```

### Browse a category

```json
{
  "mode": "browse",
  "categories": ["177", "https://www.ebay.de/b/PC-Laptops-Netbooks/177/bn_317584"],
  "marketplace": "DE",
  "maxItems": 240
}
```

### Item, seller and store modes

```json
{
  "mode": "item",
  "itemIds": ["257633093777", "https://www.ebay.co.uk/itm/276493839365"],
  "includeItemSpecifics": true
}
```

```json
{
  "mode": "seller",
  "sellerUsernames": ["discountcomputerdepot"],
  "includeRecentFeedback": true
}
```

```json
{
  "mode": "stores",
  "storeNames": ["discountcomputerdepot"],
  "marketplace": "US",
  "maxItems": 60
}
```

## Output

Fields are omitted when eBay genuinely does not show them for a record; the actor never emits `null`, empty strings, empty lists or empty objects.

### Listing and item fields

| Field | Description |
|---|---|
| `recordType` | `item`, `seller` or `store` |
| `id` / `itemId` | eBay item ID |
| `url` | Canonical item URL |
| `title`, `imageUrl`, `images` | Listing title and image URLs |
| `price`, `priceValue`, `priceCurrency` | Formatted price, numeric value and ISO currency |
| `wasPrice`, `wasPriceValue`, `discountPercent` | Original price and discount when shown |
| `condition`, `conditionDescription` | Item condition (localized card labels are normalized to eBay's English condition names) and the full condition explanation |
| `brand`, `model`, `mpn`, `gtin`, `color` | Product identifiers from JSON-LD |
| `seller`, `sellerFeedbackPercent`, `sellerFeedbackCount`, `sellerFeedbackScore` | Seller identity and reputation |
| `productRating`, `reviewCount` | Product star rating and number of ratings |
| `watchCount`, `soldCount`, `bids`, `timeLeft` | Demand and auction signals |
| `listingType`, `isBestOffer`, `isSponsored`, `programBadge` | Format, offers, sponsored flag and programme badge |
| `shippingCost`, `shippingCostValue`, `shippingCurrency`, `deliveryEstimate`, `shippingDestination` | Shipping data |
| `itemLocation`, `returns`, `returnDays`, `returnFees` | Location and returns policy |
| `quantityAvailable`, `quantitySold` | Stock and units sold |
| `paymentMethods` | Accepted payment methods |
| `itemSpecifics`, `sellerNotes` | Full item-specifics table and seller notes |
| `categoryPath` | Breadcrumb categories |
| `availability`, `startTime` | Stock state and listing start time |
| `marketplace` | Marketplace code |
| `storeName`, `storeUrl`, `storeTagline`, `storeFeedbackPercent`, `storeItemsSold`, `storeFollowers` | Store context on store-mode rows |
| `searchUrl`, `scrapedAt` | Source search URL and ISO scrape time |

### Seller profile fields

| Field | Description |
|---|---|
| `sellerUsername`, `sellerProfileUrl` | Seller identity and feedback URL |
| `sellerFeedbackScore` | Lifetime feedback score |
| `positiveFeedbackPercent` | Positive feedback percentage for the last 12 months |
| `positiveFeedbackCount`, `neutralFeedbackCount`, `negativeFeedbackCount` | 12-month counts |
| `feedbackBreakdown` | Counts for the last 1, 6 and 12 months |
| `totalFeedbackReceived`, `automatedFeedbackCount` | Total and automated feedback |
| `memberSince`, `memberCountry` | Membership date and country |
| `isEbayStore`, `isTopRatedSeller`, `storeUrl` | Store and top-rated flags |
| `recentFeedback` | Up to 25 rows: comment, item title/ID, rating, buyer, price, when |

### Store record fields (when a store has no active listings)

`recordType: "store"`, `storeName`, `storeUrl`, `storeTagline`, `storeFeedbackPercent`, `storeItemsSold`, `storeFollowers`, `storeListingCount`, `storeCategories`, `marketplace`, `scrapedAt`.

## Pages, limits and reliability

- Each result page asks for 60, 120 or 240 rows depending on the remaining record budget, up to eBay's 200-page search cap.
- Pagination stops on `maxItems`, `maxPages`, an empty page, a short page, a repeated page (no new IDs) or an explicit "no exact matches" page.
- Network errors, 403/429/5xx responses and throttle pages are retried with exponential backoff and jitter; the client switches between direct eBay and the public mirror automatically.
- Duplicate item IDs are removed across queries, categories, stores and mixed URL types; a richer item-detail row replaces a duplicate listing row.
- Localized condition, seller-feedback and shipping labels from the German, French, Italian and Spanish sites are normalized to the same English field values as the US, UK, Canadian and Australian sites.
- No wall-clock cut-off is used, so the actor does not silently return fewer rows than requested.

## FAQ

**Do I need cookies or an API key?** No. The actor never asks for credentials and does not use an eBay developer account.

**Do I need to configure a proxy?** No. If an old task still sends a `proxy` object, the actor logs that it is ignored and continues without it.

**Why do some listings lack watch or sold counts?** eBay only renders those counters on some cards and only above its own thresholds. Missing means the source did not show it.

**Can I scrape sold/completed listings?** Not without an eBay login, which this actor deliberately does not require. eBay serves a sign-in wall for `LH_Sold=1`/`LH_Complete=1`.

**How are prices represented?** `price` is the display string (`$339.99`, `£189.99`, `EUR 229,00`, `AU $375.00`); `priceValue` is the numeric value and `priceCurrency` the ISO code.

**How many rows can one run return?** Up to `maxItems` (maximum 10,000), subject to eBay's own 200-page ceiling per source. Split very large jobs by category, query or price range.

**Is this fast?** Listing modes use one HTTP request per page. Item mode uses one request per item. Seller and store modes add one profile/store request per source.

## Limitations

- Sold/completed listings are not available without an eBay account; the actor does not use cookies.
- eBay caps search pagination at 200 pages per query. For a broader sweep, split by category, keyword or price range.
- eBay stores show only a curated storefront plus the seller's active listings; ended listings are not part of a store inventory.
- Item descriptions are not included because eBay serves them from a separate signed frame; the structured specifics table is included instead.
- Some fields are category- or listing-conditional (bids, watchers, sold counts, GTIN, return policy). They are omitted for records where eBay does not display them.
- eBay does not render sponsored labels inside the card HTML on the German, Italian and Spanish sites, so `isSponsored` is reliable on the US, UK, Canadian, Australian and French sites and simply omitted elsewhere.
- Category browse uses the category name (from the ID page title or the `/b/` URL slug) as the search phrase because eBay's keywordless category page is a navigation page, not a listing river. If eBay blocks that category page from a run's IP, the actor falls back to eBay's broad `all` query inside the category; very unusual titles may then be missed, so narrow with `browseKeyword` for a precise sweep.
- Store statistics are shown only when eBay renders them on the storefront; sellers without a storefront return their active listings with store name and URL only.
