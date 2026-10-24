# Google Shopping Price Comparison & Deal Finder

Search Google Shopping by keyword and get back structured product listings: current and original price, merchant name, rating, review count, and delivery info — localized to any of 48 countries.

## What this actor does

- Searches Google Shopping for one or more keyword queries and extracts the listed products
- Reports both current and original price so discounted items are easy to spot
- Captures merchant name, rating, review count, and delivery information per listing
- Localizes results and pricing to any of 48 supported countries and their currency
- Passes product condition, price range and sort order (relevance, review score, price) to Google Shopping
- Drops blank fields from each record; `query`, `position`, `title`, `price`, `searchUrl` and `scrapedAt` are always present

## Output per product

- `query` — search query that produced this result
- `position` — 1-based position on the results page the product was read from
- `title` — product title
- `price` — current price with currency symbol (e.g. `"$999.00"`)
- `originalPrice` — original price when the item is discounted
- `currency` — currency code (`USD`, `EUR`, `GBP`, etc.)
- `merchant` — store or merchant name
- `rating`, `reviewCount` — average rating and number of reviews, as text (e.g. `"4.7"`, `"1203"`)
- `imageUrl` — product thumbnail image URL
- `delivery` — delivery info (e.g. `"Free delivery"`), when shown on the result card
- `isSponsored` — currently always `false`; ads are not told apart from organic listings
- `productId` — Google Shopping product identifier, when available
- `productUrl` — link Google gives for the product; when there is none, a Google Shopping search for the product's title
- `merchantUrl` — the merchant link from the result when one is found; otherwise a homepage guessed from the merchant name for well-known stores (e.g. Best Buy, Walmart) or names that look like a domain
- `searchUrl` — the Google Shopping search URL that produced this result
- `scrapedAt` — ISO 8601 collection timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `queries` | array of strings | – | **Required.** Product search queries |
| `maxResultsPerQuery` | integer | `20` | Maximum products to extract per query (1–100) |
| `countryCode` | string | `us` | Country for localized results and pricing (48 supported) |
| `languageCode` | string | `en` | Language for results |
| `sortBy` | string | – | `review_score` / `price_low` / `price_high`; leave empty for relevance |
| `condition` | string | – | `new` / `used` / `refurbished`; leave empty for any |
| `minPrice` | integer | – | Minimum price filter, in local currency units |
| `maxPrice` | integer | – | Maximum price filter, in local currency units |
| `proxyConfiguration` | object | `GOOGLE_SERP` proxy | The actor picks its own proxies for Google; this setting is only used for the browser-based page loads, and only when set to an Apify proxy group other than `GOOGLE_SERP` |

### Example: single query, default country

```json
{
  "queries": ["iPhone 15 Pro Max"],
  "maxResultsPerQuery": 10,
  "countryCode": "us",
  "languageCode": "en"
}
```

### Example: multiple queries

```json
{
  "queries": ["laptop", "wireless headphones", "running shoes"],
  "maxResultsPerQuery": 20
}
```

### Example: price-filtered, sorted by review score

```json
{
  "queries": ["mechanical keyboard"],
  "sortBy": "review_score",
  "condition": "new",
  "minPrice": 50,
  "maxPrice": 200,
  "countryCode": "de",
  "languageCode": "de"
}
```

## Use cases

- **Price monitoring** — track a product's price across merchants over time
- **Competitive analysis** — compare how different sellers price the same product
- **Market research** — survey pricing patterns and the merchant landscape for a category
- **Deal detection** — find discounted products by comparing `originalPrice` to `price`
- **Product catalog building** — assemble a dataset of products with pricing and reviews

## FAQ

**Does this actor need cookies or login?**
No. It reads publicly available Google Shopping search results and requires no authentication.

**Do I need to change the proxy setting?**
No. Google blocks automated requests from data-center IPs, so the actor routes its requests through Apify Proxy on its own, using IPs in the selected country where it can. The default `GOOGLE_SERP` setting is fine for almost every run.

**How many results can I get per query?**
Up to 100 per query via `maxResultsPerQuery`. The actor reads at most three result pages per query and stops early when a page returns fewer than 10 products, so narrow searches can return fewer. Products whose title repeats an earlier one in the same query are skipped.

**Can I reach individual product detail pages?**
No. This actor returns search-result data (title, price, merchant, rating, reviews). The `productUrl` field gives you a link to follow up on the product yourself.

**Does it support international pricing?**
Yes. Set `countryCode` (48 options) to search that country's Google domain and see local prices; the `currency` field reports the currency detected from the price, falling back to the country's usual currency.

**Why are some fields like `productId`, `delivery` or `originalPrice` missing?**
Google doesn't show them on every result card, and blank fields are left out of the record. `merchantUrl` is also missing when the result has no merchant link and the store name isn't one the actor recognizes.

**Can I tell sponsored products from organic ones?**
Not at the moment. `isSponsored` is returned as `false` for every product.

**What's the difference between `price` and `originalPrice`?**
`price` is the current listed price. `originalPrice` only appears when the item is discounted, showing the pre-discount price.

**Is this affiliated with Google?**
No. This is an independent, third-party actor that reads Google's public Shopping results; it isn't affiliated with or endorsed by Google.
