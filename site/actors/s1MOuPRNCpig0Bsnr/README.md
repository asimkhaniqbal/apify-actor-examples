# MercadoLibre Marketplace Scraper

Collect real marketplace data from MercadoLibre in Argentina, Brazil, Mexico, Chile, Colombia, Peru, Uruguay, Ecuador, Venezuela, Bolivia, Paraguay, Guatemala, Honduras, Nicaragua, El Salvador, Costa Rica, Panama and the Dominican Republic. Search products, open full product pages, read reviews and buyer questions, walk seller catalogs, browse categories, and track best sellers and deals.

No account, cookies, login or API key is required.

## What it does

- **Search** by keyword, category, seller or a pasted listing URL, with condition, sort, price range, free shipping, MercadoLibre FULL, official-store and discount filters.
- **Product** pages by catalog URL, item URL, user-product URL or bare ID, with gallery, specs, variations, seller, shipping, payment methods, warranty and returns.
- **Reviews** with rating filters and an optional deep sweep across rating buckets.
- **Questions** with buyer questions and seller answers.
- **Seller** catalogs and an optional seller profile row.
- **Categories** as the 24 root departments, a category tree with attributes, or keyword-to-category discovery.
- **Best sellers** ranked per category and **live deals**, both as product rows.

## Services

Every run covers one country site at a time and one mode at a time. The actor returns JSON records in the default dataset, ready for spreadsheets, BI tools, price monitoring, catalog enrichment and competitor research. The output is omit-empty: fields the source does not expose for a record are left out rather than filled with empty values.

## How it works

The actor reads the public mobile web pages that MercadoLibre serves to its own app and extracts the embedded page state. That cookieless route exposes the same products, prices, reviews and questions a shopper sees, without an OAuth application or an API token. Product searches are paginated in pages of 50 up to the source window, and duplicate item IDs are removed before records are pushed.

## Input

| Field | Used by | Default | Notes |
|---|---|---|---|
| `mode` | all | `search` | `search`, `product`, `reviews`, `questions`, `seller`, `categories`, `bestSellers`, `deals` |
| `siteId` | all | `MLA` | 18 country sites |
| `searchQuery` | search | `laptop` | Keyword to search for |
| `searchUrls` | search, seller | – | Pasted search/category/seller listing URLs |
| `categoryId` | search, categories, bestSellers, deals | – | Category ID or URL |
| `categoryQuery` | categories | – | Keyword for category discovery |
| `sellerId` | seller, search | – | Numeric seller ID or seller URL |
| `productUrls` | product, reviews, questions | – | Product/catalog/item URLs or IDs, one per line |
| `sort` | search, seller | `relevance` | Relevance, price ascending or price descending |
| `condition` | search, seller | `any` | Any, new, used, refurbished or open box |
| `priceMin`, `priceMax` | search, seller | `0` | Price range in the country currency; 0 = open |
| `freeShippingOnly` | search, seller | `false` | Keep free-shipping products |
| `fullShippingOnly` | search, seller | `false` | Keep MercadoLibre FULL products |
| `officialStoresOnly` | search, seller | `false` | Keep official-store products |
| `minDiscount` | search, seller, deals | `0` | Minimum discount percentage |
| `includeSponsored` | search, seller | `true` | Include or remove paid placements |
| `includeDetails` | search, seller | `false` | Enrich every result with the full product page |
| `includeReviews`, `includeQuestions` | product | `false` | Attach review or Q&A data to product records |
| `includeSellerProfile` | seller | `false` | Emit a seller profile before the catalog |
| `includeAttributes` | categories | `false` | Attach the category attribute schema |
| `maxItems` | most modes | `50` | Maximum records; product queries stop at 2,000 |
| `maxReviews`, `reviewRating` | reviews | `30`, `all` | Review ceiling and optional star filter |
| `maxQuestions` | questions, product | `20` | Question ceiling |

Example: search used laptops in Mexico with free shipping, sorted by price.

```json
{
  "mode": "search",
  "siteId": "MLM",
  "searchQuery": "laptop",
  "condition": "used",
  "freeShippingOnly": true,
  "sort": "price_asc",
  "maxItems": 50
}
```

Example: fetch one product by URL and include its reviews and questions.

```json
{
  "mode": "product",
  "siteId": "MLM",
  "productUrls": ["https://www.mercadolibre.com.mx/p/MLM1081091201"],
  "includeReviews": true,
  "includeQuestions": true,
  "maxReviews": 30,
  "maxQuestions": 20
}
```

## Output

Every row has `recordType`, `siteId` and `scrapedAt`. Product-like rows keep the legacy fields `itemId`, `title`, `price`, `currency`, `condition`, `availableQuantity`, `soldQuantity`, `sellerId`, `sellerName`, `sellerReputation`, `imageUrl`, `categoryId`, `categoryPath`, `freeShipping`, `officialStoreId`, `itemUrl`, `warranty`, `listingTypeId`, plus the new detail fields:

- **Price and stock:** `originalPrice`, `discountPercent`, `installments`, `hasStock`, `availableQuantity`, `soldText`.
- **Seller and shipping:** `sellerPowerStatus`, `sellerStoreType`, `sellerSales`, `sellerFollowers`, `sellerProducts`, `fullShipping`, `logisticType`, `shippingMode`, `shippingText`, `localPickup`, `storePickup`, `isOfficialStore`.
- **Content:** `specs`, `specGroups`, `description`, `images`, `variations`, `warranty`, `returns`, `benefits`, `paymentMethods`, `rating`, `promotions`.
- **Search:** `position`, `searchQuery`, `totalResults`, `sponsored`, `catalogProductId`, `userProductId`, `domainId`.

Review rows carry `reviewId`, `rating`, `text`, `date`, `relativeDate`, `country`, `variant`, `likes` and `media`. Question rows carry `questionId`, `question`, `answer` and `answerDate`. Category rows carry `categoryId`, `categoryName`, `categoryPath`, `totalItems`, `children`, `attributes` and `permalink`. Best-seller and deal rows add `rank` and `dealType`.

## Usage cost

The actor runs in a 1 GB container. Search and product pages are large (roughly 1.5–2.7 MB each), so cost scales with the number of pages opened and details requested. Start with `maxItems` 50 and `includeDetails` off, then raise it only when needed. Deep review sweeps make extra requests; `maxReviews` 330 is the normal ceiling and values above that intentionally sweep rating buckets.

## FAQ

**Do I need a MercadoLibre account or API key?** No. The actor reads public pages without credentials.

**Can I paste a product link instead of an ID?** Yes. Product, catalog and user-product URLs are accepted, as are bare IDs.

**Can I get more than 50 products?** Yes. Results are paginated up to the source's 2,000-row window per query. For more, split the search by category, price range or condition.

**Why do some products have no condition or shipping field?** Search cards omit those labels for some listings. Turn on `includeDetails` to read the product page and fill them whenever the source exposes them.

**How many reviews can I collect?** A single review query returns up to 330 rows. Setting `maxReviews` above 330 sweeps every star rating and can return over 1,000 unique reviews on popular products. Ratings without text are not review rows.

**Does it support official store URLs?** Seller catalogs work with seller IDs. A store-slug landing page does not expose its catalog server-side, so use the seller ID or `_CustId_`/`seller_id` URL.

**Can I use it for cars or real estate?** This actor covers marketplace products. Vehicles and real estate are separate MercadoLibre verticals with different listing pages.

**Which countries are supported?** All 18 current MercadoLibre country sites listed in the input schema.

**Are prices converted?** No. Prices stay in the local currency shown on the site.

## Notes

- Category browse uses the official category metadata service, which is public; product, review and question data comes from the same pages shoppers use.
- Search cards are the fastest source. Detail enrichment fills specs, gallery, seller reputation and stock but costs one page per product.
- Costa Rica uses the `mercadolibre.co.cr` domain. The old `MLO` site code maps to `MRD` for backward compatibility.
- The `mode`, `siteId` and `condition` dropdowns keep the legacy values `searchProducts`, `searchByCategory`, `getProductDetails`, `getCategories`, `MLO` and `all` so existing saved tasks and API payloads stay valid. The actor maps them in code; new runs should use the current values.

## Limitations

- Search and seller catalogs stop at the source's 2,000-row window per query.
- Review queries cap at 330 rows per star bucket; deep sweeps deduplicate across buckets.
- Buyer questions are served 20 per page and some products stop earlier than their displayed count.
- Store-slug catalogs are not exposed; use a seller ID or seller URL instead.
- Seller catalog rows keep the catalog owner in `sellerId`/`sellerName`; an item card can show a brand sub-store label instead.
- Category API totals can differ from the number of active listings, so category totals are shown as metadata.
- Sponsored placements are visible on some surfaces but not all; the `includeSponsored` toggle can only remove placements the source marks.
- The deals page is a cross-category feed. A `categoryId` filter keeps matching deals from that feed and may return no rows when the category has no active deal. The feed follows the source's 20-page window.
- Autosuggest and verified-purchase flags are not exposed on this route.
