# eMAG Multi-Market Product & Price Scraper

Scrape **eMAG** — Central and Eastern Europe's largest online marketplace, covering Romania, Bulgaria, Hungary and Poland. Search products by keyword, browse any category page, or fetch full detail for a specific product, and get names, brands, current and original prices in the local currency, discount percentage, ratings and review counts, stock status, seller, images, specifications, and the product URL. Prices, currency and catalog all follow the market you select. HTTP-only against eMAG's storefronts. No auth, no proxy.

## What this actor does

- **Three modes:** `searchProducts`, `browseCategory`, `getProductDetails`
- **Four markets:** Romania (RON), Bulgaria (BGN), Hungary (HUF), Poland (PLN)
- **Filters and sorting:** min / max price in local currency, plus sort by relevance, price ascending or descending, or rating
- **Discount maths done for you:** where an original price is shown, `discountPercent` is computed from it
- **Empty fields are omitted**

## Output per product

- `productId` — eMAG product identifier
- `name` — product title
- `brand` — manufacturer (mode=getProductDetails)
- `description` — product description, trimmed to 3,000 characters (mode=getProductDetails)
- `price` — current price in the market's currency
- `originalPrice` — pre-discount price, when one is shown (mode=`searchProducts`/`browseCategory` only)
- `discountPercent` — computed discount off the original price (mode=`searchProducts`/`browseCategory` only)
- `currency` — `RON`, `BGN`, `HUF`, or `PLN`
- `rating` — average star rating
- `reviewCount` — number of customer reviews
- `inStock` — availability
- `seller` — marketplace seller name (mode=getProductDetails)
- `category` — eMAG category
- `specifications` — technical spec name/value pairs (mode=getProductDetails)
- `imageUrl` — main product image
- `url` — product page on eMAG
- `market` — the storefront the record came from
- `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchProducts` | `searchProducts` / `browseCategory` / `getProductDetails` |
| `query` | string | `laptop` | Keyword to search for (mode=searchProducts) |
| `categoryUrl` | string | `/laptopuri/c` | Category path or full URL (mode=browseCategory) |
| `productUrl` | string | – | Full product URL ending in `/pd/XXXXXX/` (mode=getProductDetails) |
| `market` | string | `ro` | `ro` / `bg` / `hu` / `pl` |
| `sortBy` | string | `relevance` | `relevance` / `price_asc` / `price_desc` / `rating` |
| `minPrice` | number | – | Keep products at or above this price (local currency) |
| `maxPrice` | number | – | Keep products at or below this price (local currency) |
| `maxItems` | int | `30` | Hard cap (1–300) |

### Example: keyword search on the Romanian market

```json
{
  "mode": "searchProducts",
  "query": "laptop",
  "market": "ro",
  "sortBy": "price_asc",
  "maxItems": 100
}
```

### Example: browse a category with a price band

```json
{
  "mode": "browseCategory",
  "categoryUrl": "/laptopuri/c",
  "market": "ro",
  "minPrice": 2000,
  "maxPrice": 5000,
  "maxItems": 200
}
```

### Example: best-rated products on another market

```json
{
  "mode": "searchProducts",
  "query": "telefon",
  "market": "bg",
  "sortBy": "rating",
  "maxItems": 50
}
```

### Example: full detail for one product

```json
{
  "mode": "getProductDetails",
  "productUrl": "https://www.emag.ro/laptop-example/pd/D1ABCDEFG/",
  "market": "ro"
}
```

## Use cases

- **Price monitoring** — track competitor pricing and discount depth across eMAG markets
- **Cross-border pricing** — compare the same product in RON, BGN, HUF and PLN
- **Marketplace sellers** — benchmark your listings against rival offers in your category
- **Assortment analysis** — measure how many SKUs and which brands compete in a category
- **Promotion tracking** — capture original vs current price to quantify real discounts
- **Product data enrichment** — pull specifications, images, and ratings for a catalog

## FAQ

**What is eMAG?**  The largest e-commerce marketplace in Central and Eastern Europe, operating storefronts in Romania, Bulgaria, Hungary and Poland. See [emag.ro](https://www.emag.ro).

**Is this affiliated with eMAG?**  No. This is a third-party actor reading their public storefront pages.

**Do I need an account?**  No. Only publicly visible product data is collected.

**Which currency are prices in?**  The market's own — RON for `ro`, BGN for `bg`, HUF for `hu`, PLN for `pl`. Price filters are applied in that same currency, so a `maxPrice` of 5000 means something very different on the Hungarian market than the Romanian one.

**Where do I get a category URL?**  Copy it from the address bar on eMAG — either the full URL or just the path, e.g. `/laptopuri/c`. Category paths are localised, so use paths from the market you're scraping.

**Why do search results have less data than product details?**  Listing pages carry the card-level fields: name, price, original/crossed-out price and the discount computed from it, rating, stock, and image. Brand, description, seller, and specifications only appear on the product page, which `getProductDetails` fetches — but that page doesn't surface a crossed-out original price, so `originalPrice`/`discountPercent` are listing-only fields.

**How is `discountPercent` calculated?**  From `originalPrice` and `price` on the listing card itself (`searchProducts`/`browseCategory` modes). Products with no crossed-out original price have no discount field rather than a zero, and `getProductDetails` records never include it.

**Does this cover marketplace sellers as well as eMAG itself?**  Yes. Results include both eMAG's own stock and third-party marketplace offers; `seller` on the detail record tells you which.

**Are search keywords language-specific?**  Yes. Each storefront indexes in its own language, so `laptop` works broadly but `telefon` will out-perform `phone` on `ro` and `bg`.

**How fresh is the data?**  Live at run time — prices and stock reflect the moment of the run, which matters during eMAG's frequent campaign periods.
