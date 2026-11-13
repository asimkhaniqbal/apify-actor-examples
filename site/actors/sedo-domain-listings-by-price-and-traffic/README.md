# Sedo Domain Listings by Price & Traffic

Sedo.com is one of the world's largest domain name marketplaces. This actor searches it by keyword or browses any of its 14 top-level categories, returning asking price, sale type (Buy Now / Make Offer), bid count, and estimated monthly traffic for every listed domain. No login, cookies, or paid proxy required.

## What this actor does

- **Two modes:** `search` (free-text keyword) and `byCategory` (browse one of Sedo's 14 top-level categories: Arts, Business, Computers, Education, Games, Health, Home, Miscellaneous, Recreation, Science, Shopping, Society, Sports, Adult)
- **Filters:** TLD (extension), price range, minimum estimated monthly traffic, domain name length range, sale/listing type (Buy Now / Make Offer / Marketplace Auction / Expiring Domain Auction)
- **Sort by:** relevance, price, traffic, name, bid count, or newest listings
- **Empty fields are omitted** — every record only contains fields Sedo actually published for that listing

## Output per domain

- `domainName`, `tld`, `domainLength`
- `price`, `priceCurrency`, `priceFormatted` — only present for listings with a visible asking price
- `saleType` — `Buy Now`, `Make Offer`, `Marketplace Auction`, or `Expiring Domain Auction`
- `bidCount` — number of bids placed so far
- `estimatedMonthlyTraffic`
- `listedAt` — ISO 8601 date the listing was created, when published
- `listingId`
- `category` — the category browsed to find this result (mode=byCategory)
- `sourceUrl` — canonical Sedo domain listing page
- `buyNowUrl` — direct purchase link, for Buy Now listings
- `recordType: "domainListing"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byCategory` |
| `searchQuery` | string | `shop` | Free-text keyword (mode=search) |
| `categoryId` | string | – | One of 14 top-level categories, e.g. `1002` (Business) (mode=byCategory) |
| `tlds` | array | – | Only include these extensions, e.g. `com`, `net`, `ai`, `io` |
| `minPrice` / `maxPrice` | int | – | Price range in USD |
| `minTraffic` | int | – | Minimum estimated monthly visits |
| `minLength` / `maxLength` | int | – | Domain name length range (excluding TLD), 1–63 |
| `saleType` | string | `any` | `any` / `buyNow` / `makeOffer` / `auction` / `expiringAuction` |
| `sortBy` | string | `relevance` | `relevance` / `price` / `traffic` / `name` / `bids` / `dateCreated` |
| `sortDescending` | boolean | `true` | Sort highest-to-lowest (or newest-first) |
| `maxItems` | int | `50` | Hard cap on emitted records (1–2000) |

### Example: keyword search filtered to .com domains

```json
{
  "mode": "search",
  "searchQuery": "crypto",
  "tlds": ["com"],
  "maxItems": 50
}
```

### Example: browse a category, Buy Now only, sorted by price

```json
{
  "mode": "byCategory",
  "categoryId": "1002",
  "saleType": "buyNow",
  "sortBy": "price",
  "sortDescending": false,
  "maxItems": 50
}
```

### Example: high-traffic short domains under $10,000

```json
{
  "mode": "search",
  "searchQuery": "app",
  "maxPrice": 10000,
  "minTraffic": 100,
  "maxLength": 8,
  "maxItems": 50
}
```

## Use cases

- **Domain investing** — screen Sedo's marketplace for undervalued or high-traffic domains matching a keyword or TLD
- **Brand research** — check availability and asking price for domain names related to a new brand or product
- **Market monitoring** — track pricing trends and bid activity within a category over time
- **Portfolio benchmarking** — compare your own domain holdings' pricing against comparable live listings
- **Lead generation for domain brokers** — surface newly listed or actively-bid-on domains in a niche

## FAQ

**What's the data source?**
Sedo's own public domain marketplace search and category browse pages at `sedo.com`. This actor is an independent third-party tool and is not affiliated with or endorsed by Sedo.

**Do I need a proxy or login?**
No. This actor works with no configuration beyond the input fields above.

**Why do some listings have no `price` field?**
Not every domain has a public asking price — some are marketplace-negotiable ("Make Offer") without a stated minimum. Only listings with a real, published price include `price`/`priceFormatted`.

**What currency are prices in?**
Whatever currency Sedo itself displays for that listing — check `priceCurrency` on each record; `minPrice`/`maxPrice` filters use Sedo's own approximate USD equivalent.

**How fresh is the data?**
Every run queries Sedo live — there is no caching.

**Can I combine a keyword search with a TLD filter?**
Yes — `tlds`, price/traffic/length filters, and `saleType` all apply on top of `searchQuery` (mode=search) or `categoryId` (mode=byCategory).
