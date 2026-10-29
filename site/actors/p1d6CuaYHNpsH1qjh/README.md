# Trendyol Products, Reviews & Q&A Scraper

Collect Trendyol product catalogues, prices, discounts, variants, seller details, reviews and answered customer questions from four storefronts: Turkey, Germany, the UAE and Saudi Arabia. Run it by keyword, category, brand, campaign, seller store, product URL or numeric product ID. No account, login or proxy setup is required.

## What this actor does

- **Eight modes:** `search`, `category`, `store`, `product`, `reviews`, `qna`, `bestSellers`, `newArrivals`
- **Four storefronts:** Turkey (TRY), Germany (EUR), UAE (AED) and Saudi Arabia (SAR), with local titles, prices and seller details
- **Seven sort orders:** relevance, best sellers, most rated, most favourited, price ascending, price descending and newest
- **Filters:** gender, minimum/maximum price, minimum rating, free shipping, fast delivery, discounted, in stock, coupons, special offers, new releases and official sellers
- **Optional enrichment:** open each product page for variants, attributes, barcode, category path, seller identity, delivery estimate and social proof
- **Reviews and Q&A:** collect review rows and answered-question rows per product, with pagination up to your limits
- **Clean records:** empty fields are omitted rather than returned as null or placeholders

## Services

| Service | What you get |
|---|---|
| Search | Products for one or more keywords, with all filters and sort orders |
| Category, brand or campaign | Products from any Trendyol category, brand or campaign URL |
| Seller store | Products from a seller store URL or seller ID |
| Product details | Full detail for product URLs, paths or numeric IDs |
| Reviews | Reviewer name, rating, comment, date, size, variant, photos, helpful votes and seller |
| Answered questions | Question, asker, date, seller answer and response time |
| Best sellers | Products ranked by Trendyol's best-seller sort |
| New arrivals | Products ranked by Trendyol's newest sort |

## How it works

Choose a mode, storefront and optional filters, then run the actor. It reads the public Trendyol catalogue for the selected storefront, follows the same sort and filter parameters as the website, and removes duplicate products across queries and pages. When enrichment is enabled it also reads each product's public detail, delivery, social-proof, review and question data. Records are streamed to the dataset as they are collected.

Trendyol exposes at most about 4,968 products per query or filter axis. For larger collections, split the request by category, brand, gender, price range or sort order; the actor reports when the ceiling is reached instead of silently returning fewer rows.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `category`, `store`, `product`, `reviews`, `qna`, `bestSellers`, `newArrivals` |
| `market` | string | `TR` | `TR`, `DE`, `AE` or `SA` |
| `searchQueries` | array | – | Keywords for `search`, `bestSellers` and `newArrivals` |
| `searchUrls` | array | – | Trendyol search URLs; their filters are preserved |
| `categoryUrls` | array | – | Category, brand or campaign URLs, or category IDs |
| `storeUrls` | array | – | Seller store URLs or seller IDs |
| `productUrls` | array | – | Product URLs, paths or numeric IDs for `product`, `reviews` and `qna` |
| `sortBy` | string | `SCORE` | `SCORE`, `BEST_SELLER`, `MOST_RATED`, `MOST_FAVOURITE`, `PRICE_BY_ASC`, `PRICE_BY_DESC`, `MOST_RECENT` |
| `gender` | string | any | `kadin`, `erkek` or `cocuk` |
| `minPrice` / `maxPrice` | integer | – | Displayed price range in the selected market's currency |
| `minRating` | string | any | `3`, `4` or `4.5` |
| `freeShippingOnly` | boolean | false | Keep free-shipping products |
| `fastDeliveryOnly` | boolean | false | Keep same-day, next-day or express products |
| `discountedOnly` | boolean | false | Keep products with a higher reference price |
| `inStockOnly` | boolean | false | Keep available products |
| `couponsOnly` | boolean | false | Keep products showing a coupon or discount code |
| `specialOffersOnly` | boolean | false | Keep products with a promotion, flash sale or bundle |
| `newlyReleasedOnly` | boolean | false | Keep products tagged as new |
| `officialSellerOnly` | boolean | false | Keep products from official sellers |
| `includeDetails` | boolean | false | Enrich listing records with detail-page fields |
| `includeReviews` | boolean | false | Add review rows |
| `maxReviews` | integer | 20 | Reviews per product (1–100) |
| `includeQna` | boolean | false | Add answered-question rows |
| `maxQna` | integer | 20 | Answered questions per product (1–100) |
| `maxItems` | integer | 20 | Products to return, or product sources to process in reviews/Q&A modes (1–10,000) |

## Output

Each record has `recordType` and `scrapedAt`.

### Product records

- Identity: `productId`, `productCode`, `productGroupId`, `title`, `brand`, `brandId`, `category`, `categoryId`, `categoryPath`, `color`, `gender`, `genderId`, `barcode`, `itemNumber`, `listingId`, `boutiqueId`, `sellerId`
- Pricing: `price`, `originalPrice`, `discountPercent`, `sellingPrice`, `discountedPrice`, `currency`, `market`
- Ratings: `rating`, `reviewCount`, `ratingCount`, `commentCount`, `favoriteCount`, `reviewSummary`
- Seller: `seller`, `sellerId`, `sellerScore`, `sellerUrl`, `sellerOfficialName`, `sellerCity`, `sellerCountry`
- Fulfilment: `inStock`, `stockStatus`, `freeShipping`, `fastDelivery`, `estimatedDelivery`, `deliveryDateText`, `carriers`
- Catalogue: `description`, `images`, `imageUrl`, `imageCount`, `variants`, `attributes`, `otherMerchants`, `otherSellerCount`, `badges`, `promotions`, `coupon`, `specialOffer`, `officialSeller`, `socialProof`
- Provenance: `sourceContext`, `productUrl`

### Review records

`reviewId`, `productId`, `productUrl`, `productName`, `market`, `rating`, `comment`, `reviewDate`, `reviewerName`, `verifiedPurchase`, `helpfulCount`, `productSize`, `productVariant`, `mediaUrls`, `language`, `sellerId`, `sellerName`, `scrapedAt`

### Answered-question records

`questionId`, `productId`, `productUrl`, `productName`, `market`, `question`, `questionDate`, `askerName`, `answer`, `answerDate`, `answeredIn`, `sellerId`, `sellerName`, `scrapedAt`

### Example: German sneakers, cheapest first

```json
{
  "mode": "search",
  "market": "DE",
  "searchQueries": ["sneakers"],
  "sortBy": "PRICE_BY_ASC",
  "maxItems": 50
}
```

### Example: Turkish category with detail enrichment and reviews

```json
{
  "mode": "category",
  "market": "TR",
  "categoryUrls": ["https://www.trendyol.com/kadin-sneaker-x-g1-c1172"],
  "includeDetails": true,
  "includeReviews": true,
  "maxReviews": 10,
  "maxItems": 20
}
```

### Example: product details and Q&A by ID

```json
{
  "mode": "product",
  "market": "AE",
  "productUrls": ["1076268986"],
  "includeQna": true,
  "maxQna": 10
}
```

### Example: new arrivals in a Saudi category

```json
{
  "mode": "newArrivals",
  "market": "SA",
  "categoryUrls": ["https://www.trendyol.sa/kadin-sneaker-x-g1-c1172"],
  "maxItems": 30
}
```

## Usage cost

You pay only for Apify platform usage, based on the actor's run time, memory and the number of dataset records you store. The smallest daily test uses one keyword and three products. Larger requests cost more because each product page, review page and question page is an additional request.

## FAQ

### Which countries are supported?

Turkey, Germany, the United Arab Emirates and Saudi Arabia. Choose the storefront in the `market` field; prices use that market's currency.

### Do I need a Trendyol account or a proxy?

No. The actor uses the public storefront and does not require a login, account or proxy configuration. It manages the connection automatically, including retries when the storefront rejects a direct request.

### Why did I get fewer products than I requested?

Trendyol caps each query or filter combination at about 4,968 products. Split broad requests by category, brand, gender, price range or sort order. The actor tells you when it reaches this ceiling.

### Why is a description missing?

Trendyol only publishes long descriptions for some products. When there is no description, the field is omitted instead of being filled with marketing text.

### What is the difference between `price`, `originalPrice` and `discountedPrice`?

`price` and `discountedPrice` are the displayed selling price. `originalPrice` is the higher reference price shown before the discount, and `discountPercent` is calculated from those two values. When there is no higher reference price, `originalPrice` and `discountPercent` are omitted.

### Why do some products have no rating or review count?

New or low-volume products may not have ratings yet. Those fields are omitted for such products.

### Are review and question records included in every run?

Only when you enable `includeReviews`, `includeQna`, or run the `reviews` or `qna` mode. Each row is separate from the product record.

### Can I use IDs instead of URLs?

Yes. Product modes accept numeric product IDs, and store mode accepts seller IDs.

### Is this affiliated with Trendyol?

No. This is a third-party actor that reads publicly available Trendyol pages and is not affiliated with, endorsed by or sponsored by Trendyol.

## Notes

- Prices follow the selected storefront and can change between runs.
- Reviews and answers are shown as published by Trendyol, including masked reviewer names.
- Delivery estimates are the values shown at collection time.
- Use `maxItems`, `maxReviews` and `maxQna` to keep a run small.

## Limitations

- Trendyol's public listing depth is about 4,968 products per query or filter axis; there is no deeper public pagination.
- Trendyol can place a promoted campaign product outside the selected sort order; the actor keeps the storefront's order.
- Long descriptions are available for only a subset of products.
- Review sorting is not exposed because the public endpoint does not reliably change order.
- A product with no public reviews or no answered questions produces no review or question rows.
- Answer dates and response times are included only when Trendyol publishes them.
