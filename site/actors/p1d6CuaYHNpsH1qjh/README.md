# Trendyol Product Finder with Price & Discount Tracking

Search Trendyol, Turkey's largest online marketplace, by keyword, browse a category, pull trending or new-arrival items, or fetch full detail for specific product URLs — with price, discount percentage, rating and seller on every product.

## What this actor does

- Searches Trendyol by keyword, with sort order, gender and price-range filters
- Browses a specific category page and returns its product listing
- Fetches trending/best-selling or new-arrival items
- Fetches full detail — including description — for a list of specific product URLs
- Omits any field Trendyol doesn't show for a given product rather than returning a null or placeholder value

## Output per product

| Field | Description |
|---|---|
| `productId` | Trendyol's product ID |
| `title` | Product title |
| `brand` | Brand name |
| `category` | Product category |
| `description` | Full product description — only present on `productDetails` records |
| `price` | Current price, in TRY |
| `originalPrice` | Original price before discount, when a discount applies |
| `discountPercent` | Discount percentage (0-100) |
| `rating` | Average rating (0-5) |
| `reviewCount` | Number of reviews |
| `seller` | Seller/merchant name |
| `freeShipping` | `true` when free shipping is available |
| `imageUrl` | Product image URL |
| `productUrl` | Full product page URL |
| `color` | Product color |
| `gender` | Target gender, as Trendyol categorizes it |
| `recordType` | Always `"product"` |
| `scrapedAt` | UTC timestamp when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | search | search, productDetails (by URL), categoryBrowse, trendingItems, or newArrivals |
| `keyword` | string | sneakers | Search term — used in `search` mode, and optionally to narrow `trendingItems`/`newArrivals` |
| `sortBy` | string | BEST_SELLER | Sort order for search and category browsing: BEST_SELLER, MOST_RATED, PRICE_BY_ASC, PRICE_BY_DESC, or MOST_FAVOURITE |
| `productUrls` | array of strings | `[]` | Trendyol product URLs to fetch details for — used in `productDetails` mode |
| `categoryUrl` | string | – | Trendyol category path or full URL, e.g. "erkek-spor-ayakkabi-x-c87" — used in `categoryBrowse` mode |
| `gender` | string | (any) | Filter by gender: kadin (women), erkek (men), unisex, or cocuk (kids) |
| `minPrice` / `maxPrice` | integer | – | Price range filter, in Turkish Lira |
| `maxItems` | integer | 20 | Maximum number of products to return (1-500) |
| `proxyConfiguration` | object | Apify Proxy enabled | Proxy settings; Trendyol is only reachable from Turkish IP addresses, so a proxy is enabled by default |

### Example: keyword search with filters

```json
{
  "mode": "search",
  "keyword": "sneakers",
  "sortBy": "BEST_SELLER",
  "gender": "erkek",
  "maxItems": 20
}
```

### Example: category browse sorted by price

```json
{
  "mode": "categoryBrowse",
  "categoryUrl": "https://www.trendyol.com/erkek-ayakkabi",
  "sortBy": "PRICE_BY_ASC",
  "maxItems": 50
}
```

### Example: full detail for specific products

```json
{
  "mode": "productDetails",
  "productUrls": [
    "https://www.trendyol.com/nike/air-max-270-p-123456789"
  ]
}
```

## Use cases

- **Price monitoring** — track a product's price and discount percentage over time
- **Market research** — compare ratings and review counts across brands in a category
- **Catalog enrichment** — pull titles, images and descriptions to enrich a product database
- **Trend spotting** — use `trendingItems` and `newArrivals` to see what's currently popular
- **Assortment analysis** — compare sellers and pricing within a category

## FAQ

**Do I need to configure a proxy myself?**
No — Trendyol is only reachable from Turkish IP addresses, so a proxy is enabled by default in `proxyConfiguration`. You generally don't need to change it.

**What do the sort options mean?**
BEST_SELLER ranks by sales volume, MOST_RATED by rating, PRICE_BY_ASC/PRICE_BY_DESC by price, and MOST_FAVOURITE by how often shoppers favorited the item.

**Can I filter by gender and price together?**
Yes — `gender`, `minPrice` and `maxPrice` can all be combined with `search` or `categoryBrowse`.

**Why does `description` only appear on some records?**
It comes from a product's own detail page and is only populated in `productDetails` mode; search, category-browse, trending and new-arrival results don't include it.

**What is `trendingItems` mode?**
It runs the same product search sorted by best-sellers. Without a `keyword`, it searches the Turkish term "trend" so you still get a broad page of currently popular products rather than a specific category.

**What is `newArrivals` mode?**
It runs the same product search sorted by most-favourited. Without a `keyword`, it searches the Turkish term "yeni" ("new") as a general stand-in; add your own keyword to narrow it to a specific category.

**Is this affiliated with Trendyol?**
No. This actor reads Trendyol's public pages and is not affiliated with, endorsed by, or sponsored by Trendyol.
