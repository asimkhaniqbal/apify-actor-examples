# Allegro Scraper: Listings, Offers, Reviews & Sellers

Collect public data from **Allegro.pl**, Poland's largest marketplace, in one run. Start from a keyword search, a category page, a seller storefront, or any offer and product URL. The actor returns listing cards with price, delivery, seller and ranking signals, full offer details with stock, GTIN, specifications and company identity, buyer reviews, and seller profiles.

No API key, account, cookie or proxy setup is required.

## What this actor does

- **Seven ways in:** keyword search, category pages, seller storefronts, direct offer/product URLs, every seller for one product, product reviews, and seller profiles.
- **Nine sort orders:** relevance, price, price with delivery, popularity, ending soonest, newest, and product rating.
- **Live filters:** condition, price range, Buy Now, free delivery (Smart!), Super Seller, sponsored in/out.
- **Listing intelligence:** price and price with delivery, original price and discount, stock, seller rating and company flag, product rating, 30-day buyer counts, offer counts, parameters, images, rank, organic rank and sponsored flags.
- **Offer details:** stock quantity and purchase limit, GTIN, brand, full specifications, description, seller company name and tax ids, delivery methods, returns, warranty, location and publication dates.
- **Reviews:** star rating, text, pros and cons, photos, helpful votes, language, variant and reviewer details.
- **Seller profiles:** recommendation percentage, positive and negative counts, total ratings, time on Allegro, storefront offer count, categories and company identity.
- **Clean records:** empty fields are omitted at every level. No nulls, blank strings or empty objects.

## Modes

| Mode | What it reads | What you get |
|---|---|---|
| `search` | `searchQueries` | Paginated keyword results with filters and sort |
| `byCategory` | `categoryUrls` | Category pages with filters and sort |
| `bySeller` | `startUrls` | A seller's storefront offers |
| `byUrl` | `startUrls` | Full detail for offer and product URLs; listing rows for product-offers URLs |
| `byProduct` | `startUrls` | Every seller's offer for one product UUID or product URL |
| `reviews` | `startUrls` | Buyer reviews for product UUIDs or product URLs |
| `sellerProfiles` | `startUrls` | Seller rating summary, company identity and storefront stats |

`byUrl` also accepts offer, product and product-offers URLs. `bySeller` and `sellerProfiles` accept seller URLs. Wrong-mode URLs are skipped with a status message instead of failing the whole run.

## Output

Every record has a `recordType`:

- `listing` for search, category, seller, product-offers and offer/product rows.
- `review` for buyer opinions.
- `seller` for seller profiles.

### Listing and offer fields

| Field | Meaning |
|---|---|
| `offerId` | Allegro offer identifier |
| `title` | Offer title |
| `url` | Canonical offer URL |
| `price`, `currency` | Current price and currency |
| `originalPrice`, `discountPercent` | Pre-discount price and discount when present |
| `priceWithDelivery`, `deliveryCost`, `deliveryText` | Delivery-aware price and cheapest delivery |
| `freeShipping`, `isSmart`, `hasFreeReturn` | Delivery perks |
| `condition` | `new`, `used`, `refurbished` or `damaged` |
| `seller`, `sellerId`, `sellerUrl`, `sellerRating` | Seller identity and positive-feedback percentage |
| `sellerIsCompany`, `sellerSuperSeller`, `sellerPositiveCount`, `sellerNegativeCount` | Seller badges and feedback counts |
| `sellerCompany` | Legal name, tax id, REGON, KRS, address and verification |
| `imageUrl`, `images`, `thumbnail` | Product media |
| `category`, `categoryId`, `categoryPath` | Category identity and breadcrumbs |
| `rating`, `reviewCount` | Product rating and rating count |
| `productId`, `productUrl`, `productOffersCount` | Catalog product and seller count |
| `soldLast30Days` | Buyers in the last 30 days |
| `stockQuantity`, `stockUnit`, `purchaseLimit` | Stock and order limits |
| `brand`, `gtin` | Brand and GTIN/EAN when published |
| `parameters` | Name/value specification pairs |
| `description`, `warranty`, `returnPolicy` | Full text, warranty and returns |
| `publicationStart`, `publicationEnd` | Offer publication window |
| `location` | Seller city and country |
| `deliveryMethods` | Delivery method, group, cost and time |
| `position`, `organicPosition`, `page` | Overall rank, organic rank and source page |
| `isSponsored`, `isPromoted`, `sponsoredAndOrganic` | Placement flags |
| `recordType`, `scrapedAt` | Record kind and UTC scrape time |

Review records add `reviewId`, `productId`, `productUrl`, `author`, `rating`, `ratingPercent`, `text`, `pros`, `cons`, `images`, `createdAt`, `helpfulYes`, `helpfulNo`, `language`, `sellerLogin`, `variant`, `isBest` and `productFeatures`.

Seller records add `sellerId`, `seller`, `url`, `positivePercent`, `positiveCount`, `negativeCount`, `ratingsTotal`, `onAllegroSince`, `offersCount`, `isCompany`, `superSeller`, `company`, `location` and `categories`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `bySeller`, `byUrl`, `byProduct`, `reviews` or `sellerProfiles` |
| `searchQueries` | array | – | Keywords for `search` |
| `categoryUrls` | array | – | Category URLs for `byCategory` |
| `startUrls` | array | – | URLs or ids for `bySeller`, `byUrl`, `byProduct`, `reviews` and `sellerProfiles` |
| `market` | string | `pl` | Only `pl` (allegro.pl) is supported |
| `sortBy` | string | `relevance` | One of the nine sort orders |
| `condition` | string | `all` | `all`, `new`, `used`, `refurbished` or `damaged` |
| `minPrice` | integer | – | Minimum price in PLN |
| `maxPrice` | integer | – | Maximum price in PLN |
| `freeShippingOnly` | boolean | `false` | Apply Allegro's free-delivery filter |
| `buyNowOnly` | boolean | `false` | Exclude auctions |
| `superSellerOnly` | boolean | `false` | Keep Super Seller offers |
| `includeSponsored` | boolean | `true` | Include sponsored placements |
| `includeDetails` | boolean | `false` | Open each offer in listing modes for stock, GTIN, specs and company data |
| `maxItems` | integer | `100` | Total records; each source is capped at 6,000 offers |
| `maxReviewsPerProduct` | integer | `100` | Review limit per product in `reviews` mode |

Old inputs remain valid without appearing in the form: `searchQuery`, `categoryUrl`, `conditionFilter` and the old `sortBy`/`mode` values are mapped automatically. A saved `proxyConfiguration` is ignored because the actor handles access itself.

## Examples

### Search for new laptops under 3,000 PLN

```json
{
  "mode": "search",
  "searchQueries": ["laptop"],
  "condition": "new",
  "maxPrice": 3000,
  "sortBy": "price_asc",
  "maxItems": 100
}
```

### Browse a category with free delivery

```json
{
  "mode": "byCategory",
  "categoryUrls": [{"url": "https://allegro.pl/kategoria/laptopy-491"}],
  "sortBy": "popularity",
  "freeShippingOnly": true,
  "includeDetails": true,
  "maxItems": 200
}
```

### Fetch offer and product URLs with full detail

```json
{
  "mode": "byUrl",
  "startUrls": [
    {"url": "https://allegro.pl/oferta/10828773185"},
    {"url": "https://allegro.pl/produkt/880fe6be-3992-4532-97f2-8fd88c55889a"}
  ]
}
```

### Reviews and seller profiles

```json
{
  "mode": "reviews",
  "startUrls": [{"url": "https://allegro.pl/produkt/880fe6be-3992-4532-97f2-8fd88c55889a"}],
  "maxReviewsPerProduct": 100
}
```

```json
{
  "mode": "sellerProfiles",
  "startUrls": [{"url": "https://allegro.pl/uzytkownik/ogit_pl"}]
}
```

## Use cases

- Monitor prices, discounts and delivery costs across Allegro categories.
- Compare every seller's price and stock for the same catalog product.
- Build seller intelligence lists with company identity, rating and storefront size.
- Track rank, sponsored placement and 30-day demand signals for a keyword.
- Collect buyer reviews for sentiment, product quality and variant analysis.
- Feed price-comparison, assortment and repricing workflows with clean records.

## Limits

- **Market:** Allegro.pl only. Czech and Slovak marketplaces are not available in this version.
- **Pagination:** each listing source stops at page 100. Page 100 is served; page 101 redirects back to page 1, which the actor treats as the end. That is about 6,000 offers per query; split broad searches by price, category or condition to go deeper.
- **Detail pages:** anti-bot protection can occasionally block an individual detail page. Listing rows are always kept; missing detail fields are omitted and counted in the status message.
- **Seller storefronts:** `bySeller` reads the seller rating summary once per seller so rows carry `seller` and `sellerRating` even though storefront cards omit them. If that summary is blocked, offers are still returned and the rating is omitted.
- **History:** Allegro does not publish historical price or sales charts, so the actor returns current values only.
- **Private contact data:** only company details that sellers publish are returned. No off-platform email or phone lookup is performed.

## FAQ

**Do I need an Allegro account, API key, cookie or proxy?**
No. The actor reads public pages and manages access automatically. Nothing is required from you.

**Which modes can return the most fields?**
Offer and product URLs always return full detail. For search, category, seller and product-offer runs, turn on `includeDetails` to add stock, GTIN, specifications, company identity, warranty and returns to every listing.

**Why are some fields missing from a record?**
Allegro does not publish every field for every offer. Empty values are omitted rather than filled with nulls or placeholders. Condition, GTIN, brand and discount are the fields most often absent on listing cards; `includeDetails` fills most of them.

**How many results can one search return?**
Up to 100 pages, or roughly 6,000 offers, per source. Use narrower keywords, a category, a price range or a condition filter to reach more of a large result set.

**What do `position` and `organicPosition` mean?**
`position` is the card's overall order on the listing, including sponsored placements. `organicPosition` counts only non-sponsored cards. `page` tells you which listing page the card came from.

**How are sponsored offers handled?**
They are included by default and marked with `isSponsored`. Set `includeSponsored` to `false` for organic results only. When the same offer appears as both sponsored and organic, one record is kept and marked `sponsoredAndOrganic`.

**Can I pass bare ids instead of URLs?**
API callers can pass bare offer ids, product UUIDs and seller logins wherever URLs are accepted; category ids and aliases also work for `byCategory`. In the form, use the full Allegro URL.

**Are the results deduplicated?**
Yes. Each source is deduplicated by offer id, so a repeated placement does not create a duplicate row.

**Does the actor work on allegro.cz or allegro.sk?**
Not yet. Only allegro.pl is supported, and CZ/SK URLs are rejected with a clear status message.

**Is this actor affiliated with Allegro?**
No. It is an independent third-party tool that reads publicly available marketplace data and is not endorsed by or connected to Allegro.
