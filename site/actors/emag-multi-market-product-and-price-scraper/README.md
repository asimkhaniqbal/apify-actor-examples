# eMAG Multi-Market Product & Price Scraper

Monitor eMAG across Romania, Bulgaria and Hungary without a login, cookie or API key. Search a keyword, walk a category, open one product, or compare every seller on a product page. The output is one clean dataset with local prices in RON, EUR or HUF, the 30-day lowest price, discount depth, rating, review count, stock wording, seller, offer counts, category, image and product URL.

## Modes

| Mode | What it returns | Required field |
|---|---|---|
| `searchProducts` | Paginated keyword results | `query` |
| `browseCategory` | Paginated category or filtered listing | `categoryUrl` |
| `getProductDetails` | One product with description, brand, specs and optional seller offers | `productUrl` |
| `getProductOffers` | One row per seller offer: buybox and every competing marketplace seller | `productUrl` |

`productUrl` accepts a full URL or a path containing `/pd/` for product details. Offers mode also accepts a bare product code such as `DR3HHTYBM`. A full eMAG URL also selects its own market, so a `emag.bg` link is read from Bulgaria even when `market` says `ro`.

## Markets

| Code | Storefront | Currency |
|---|---|---|
| `ro` | emag.ro | RON |
| `bg` | emag.bg | EUR |
| `hu` | emag.hu | HUF |

Prices and price filters always use the storefront's live currency. Bulgaria switched to EUR in 2026, so the actor reads the currency from each response instead of assuming BGN.

## Filters and sorting

- `minPrice` / `maxPrice` in the local currency. An inverted range returns zero records with a clear status message instead of crashing.
- `minRating` keeps products rated at least 1-5 stars; products without a rating are excluded when this is set.
- `inStockOnly` keeps only records that can be ordered now. Filtering is applied on top of the source response, so the actor keeps paginating until it reaches `maxItems` or the source runs out.
- `sortBy` supports relevance, price ascending, price descending, review count and discount percentage. eMAG has no server-side "highest rating" sort; the old `rating` value is accepted for saved tasks and mapped to review count in code.

## Output fields

- `productId`, `partNumberKey`, `mpn` - eMAG numeric ID and product codes
- `name`, `brand`, `description`, `specifications`, `category`, `categoryPath`
- `price`, `originalPrice`, `lowestPrice30Days`, `discountPercent`, `currency` - eMAG's published discount when no original price is visible
- `rating`, `reviewCount`
- `availability`, `availabilityCode`, `inStock`
- `seller`, `sellerId`, `offerId`, `isBuybox`, `returnPeriodDays`
- `offersCount`, `otherOffersCount`, `otherOffersMinPrice`, `offers`
- `campaigns`, `imageUrl`, `url`, `market`, `scrapedAt`

Empty values are omitted. A field the source does not publish for a given product is absent, never `null` or a placeholder.

## Examples

Search laptops and keep only 4-star products under 3,000 RON:

```json
{
  "mode": "searchProducts",
  "query": "laptop",
  "market": "ro",
  "maxPrice": 3000,
  "minRating": 4,
  "sortBy": "price_asc",
  "maxItems": 100
}
```

Browse a Bulgarian category with a price band:

```json
{
  "mode": "browseCategory",
  "categoryUrl": "/laptopi/c",
  "market": "bg",
  "minPrice": 500,
  "maxPrice": 1500,
  "sortBy": "reviews",
  "maxItems": 200
}
```

Compare every seller on one product:

```json
{
  "mode": "getProductOffers",
  "productUrl": "https://www.emag.ro/laptop-lenovo-v15/pd/DR3HHTYBM/",
  "market": "ro"
}
```

## Pagination and limits

eMAG serves 60 products per listing page and caps a listing at 100 pages. The actor follows those pages, deduplicates by product code and stops at `maxItems` (1-1000) or when the source is exhausted. The default prefill is one small search with 20 products.

## Data source

The actor reads eMAG's own public storefront endpoints: the JSON listing endpoint behind the search and category pages, the product page's structured data, and the multi-offer block shown on product pages. No account, cookie, credential or user proxy is required. A translation mirror is used only when a storefront blocks the direct request; the actor never needs a proxy configuration from the user.

## Compatibility with saved tasks

Existing tasks keep working. The old `searchQuery`, `categoryPath`, `productCode`, `maxResults`, `priceMin` / `priceMax`, `ratingMin`, `inStock` and `sort: "rating"` keys are read from the raw input and mapped in code even though they no longer appear in the input form. No output field was removed or renamed.

## FAQ

**Do I need an eMAG account?** No. Only public product, price and seller data is read.

**Which currency will I get?** The currency eMAG shows for that storefront at run time: RON, EUR or HUF.

**Why are some fields missing on a search row?** Listing pages expose card-level data. Brand, description, specifications and the full offer table live on the product page, so use `getProductDetails` or `getProductOffers` for those.

**What does the 30-day low mean?** It is eMAG's own lowest-price-in-the-last-30-days figure, published under EU pricing rules. It is omitted when eMAG does not show it.

**How fresh is the data?** Every run reads the live storefront; prices, stock and seller offers reflect the moment of the run.

**Are reviews included?** The actor returns rating and review count, not review text.

**What happened to eMAG Poland?** eMAG closed its Polish storefront: `emag.pl` now redirects to `emag.ro` and has no product catalogue. A saved task that still sends `market: "pl"` gets a clear status message instead of wrong-currency data; choose `ro`, `bg` or `hu`.
