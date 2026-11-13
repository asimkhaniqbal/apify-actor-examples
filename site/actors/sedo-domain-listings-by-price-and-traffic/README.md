# Sedo Domain Listings by Price & Traffic

Sedo.com is one of the world's largest domain-name marketplaces. This actor searches it by keyword, browses any of its 14 top-level categories, monitors expiring domain auctions, or looks up specific domains. Every record carries the asking price, sale type, bids, estimated monthly traffic, TLD, name length, and — when you enable details — the seller's country, member-since date, portfolio URL, description, offer views, watchlist count, and parking traffic. No login, cookies, or paid proxy required.

## Modes

| Mode | What it does |
|---|---|
| `search` | Search the marketplace for one or more keywords. |
| `byCategory` | Browse one of Sedo's 14 top-level categories: Arts, Business, Computers, Education, Games, Health, Home, Miscellaneous, Recreation, Science, Shopping, Society, Sports, Adult. |
| `auctions` | Monitor Sedo's expiring domain auction roster with reserve, deadline, bid, and traffic filters. |
| `byDomain` | Look up specific domain names or Sedo URLs and return their full detail record. |

## Output per listing

| Field | Meaning |
|---|---|
| `domainName` | Domain as listed (Unicode when Sedo returns it). |
| `domainNameAce` | Punycode/ACE form, when it differs from `domainName`. |
| `tld` | Top-level extension, without the leading dot. |
| `domainLength` | Name length excluding the TLD. |
| `price` | Asking/current price in `priceCurrency`; omitted when Sedo publishes no price. |
| `priceCurrency` | ISO currency code for `price`. |
| `priceFormatted` | Price exactly as Sedo displays it, e.g. `15,000,000 USD`. |
| `priceType` | `buynow`, `bidrange`, or `auction` (detail records). |
| `minimumOfferPrice` | Minimum offer when Sedo publishes one (detail records). |
| `saleType` | Buy Now, Make Offer, Marketplace Auction, or Expiring Domain Auction. |
| `bidCount` | Bids placed so far. |
| `estimatedMonthlyTraffic` | Sedo's estimated monthly visits. |
| `listedAt` | ISO 8601 listing-created timestamp. |
| `listingId` | Sedo domain/listing id. |
| `domainType` | Sedo's domain type (detail records). |
| `searchKeyword` | The keyword that produced this record (search mode). |
| `category` | Category browsed (byCategory mode). |
| `domainLanguageId` | Sedo's numeric domain-language id. |
| `watchlistCount` | Number of Sedo users watching the listing. |
| `isIdn`, `isTopDomain`, `isTldSpecialAgreement` | Sedo flags, present only when true. |
| `auctionId` | Sedo auction id. |
| `auctionStartAt`, `auctionEndAt` | ISO 8601 auction window. |
| `auctionTimeLeft` | Human-readable time left at scrape time. |
| `auctionReserveMin`, `auctionReserveMax`, `auctionReserveRange` | Published reserve range. |
| `auctionReserveMet`, `auctionHasReserve`, `auctionScheduled` | Auction state flags. |
| `auctionCurrency` | Currency of the reserve range. |
| `currentBid`, `nextBid` | Current and next bid (detail records). |
| `auctionBidCount`, `lastBidAt` | Bid history size and latest bid time (detail records). |
| `sellerCountry` | Seller's stated country (detail records). |
| `sellerMemberSince` | Seller's Sedo member-since date (detail records). |
| `sellerPortfolioUrl` | Seller's public Sedo portfolio URL, when published. |
| `sellerVatApplicable` | Whether the seller must charge VAT. |
| `description` | Seller's public listing description, when present. |
| `offerViews` | Sedo's offer-view count. |
| `parkingTrafficLast31Days` | Sedo's parking-traffic count for the last 31 days. |
| `sourceUrl` | Canonical Sedo listing page. |
| `buyNowUrl` | Direct purchase link for Buy Now listings. |
| `recordType` | Always `domainListing`. |
| `scrapedAt` | ISO 8601 scrape timestamp. |

Empty fields are omitted at every depth — a record never contains `null`, `""`, or `[]`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byCategory`, `auctions`, or `byDomain`. |
| `searchQuery` | string | `shop` | Keyword for `search` mode. |
| `keywords` | array | `[]` | Optional multiple keyword searches; overrides `searchQuery`. |
| `categoryId` | string | `""` | One of 14 category ids, e.g. `1002` (Business); required for `byCategory`. |
| `domains` | array | `[]` | Domains or Sedo URLs for `byDomain`. |
| `tlds` | array | `[]` | Only these extensions, e.g. `com`, `net`, `ai`; empty means all. |
| `minPrice` | integer | – | Minimum asking price in approximate USD. |
| `maxPrice` | integer | – | Maximum asking price in approximate USD. |
| `minTraffic` | integer | – | Minimum estimated monthly visits. |
| `maxTraffic` | integer | – | Maximum estimated monthly visits. |
| `minLength` | integer | – | Minimum name length (excluding TLD), 1–63. |
| `maxLength` | integer | – | Maximum name length (excluding TLD), 1–63. |
| `minBids` | integer | – | Minimum bid count. |
| `minWatchlist` | integer | – | Minimum watchlist count. |
| `saleType` | string | `any` | `any`, `buyNow`, `makeOffer`, `auction`, `expiringAuction`. |
| `minReservePrice` | integer | – | Minimum auction reserve (auctions mode). |
| `maxReservePrice` | integer | – | Maximum auction reserve (auctions mode). |
| `endingWithinDays` | integer | – | Only auctions ending within this many days. |
| `sortBy` | string | `relevance` | `relevance`, `price`, `traffic`, `name`, `bids`, `dateCreated`, `auctionEndTime`, `auctionReserve`. |
| `sortDescending` | boolean | `true` | Highest-to-lowest / newest-first; auctions default to ending soonest. |
| `includeIdn` | boolean | `true` | Include internationalized domain names. |
| `includeHyphenated` | boolean | `true` | Include names containing a hyphen. |
| `includeNumeric` | boolean | `true` | Include names containing digits. |
| `includeDetails` | boolean | `false` | Fetch seller/description/stats detail fields (one extra request per record). Forced on in `byDomain`. |
| `maxItems` | integer | `50` | Hard cap on emitted records (1–10,000). |

### Example: keyword search filtered to .com domains

```json
{
  "mode": "search",
  "searchQuery": "crypto",
  "tlds": ["com"],
  "maxItems": 50
}
```

### Example: browse Business, Buy Now only, sorted by price

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

### Example: expiring auctions under 500 with traffic, ending this week

```json
{
  "mode": "auctions",
  "maxReservePrice": 500,
  "minTraffic": 10,
  "endingWithinDays": 7,
  "sortBy": "auctionEndTime",
  "maxItems": 100
}
```

### Example: look up specific domains or URLs

```json
{
  "mode": "byDomain",
  "domains": [
    "outlet.ai",
    "https://sedo.com/search/details/?domain=giftshop.com&language=us"
  ],
  "maxItems": 10
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

## Backward compatibility

Saved tasks that use `searchQuery`, `categoryId`, `tlds`, `minPrice`, `maxPrice`, `minTraffic`, `minLength`, `maxLength`, `saleType`, `sortBy`, `sortDescending`, or `maxItems` keep working unchanged. The actor also maps older aliases in code: `query`/`keyword`/`q` → search keyword, `category` → category, `tld` → TLD list, `listingType` → sale type, `maxResults`/`limit` → max items, `sort`/`descending` → sort order, `min_price`/`max_price`/`min_traffic`/`max_traffic`/`min_length`/`max_length`, `includeSeller` → `includeDetails`, and `domain`/`urls`/`url` → `domains`.

## Use cases

- **Domain investing** — screen Sedo's marketplace for undervalued or high-traffic domains matching a keyword, TLD, price bucket, or name shape.
- **Auction monitoring** — watch expiring auctions with reserve limits and deadline windows before they close.
- **Brand research** — check availability, asking price, and seller details for names related to a new brand or product.
- **Market benchmarking** — compare pricing, traffic, and bid activity across categories and extensions.
- **Broker lead generation** — surface newly listed or actively watched domains in a niche.

## FAQ and limitations

**What is the data source?**
Sedo's public marketplace search endpoint, its public expiring-auction search results, and its public domain-details endpoint at `sedo.com`. This actor is an independent third-party tool and is not affiliated with or endorsed by Sedo.

**Do I need a proxy or login?**
No. Everything the actor returns is public and cookieless.

**Why is there no revenue field?**
Sedo does not publish parking revenue for marketplace listings; only the signed-in seller sees it. Public traffic figures (`estimatedMonthlyTraffic`, `parkingTrafficLast31Days`) and offer statistics are included instead.

**Why is the seller's name missing?**
Sedo shows sellers by country, member-since date, and (optionally) a public portfolio URL. Their account name is not public on the listing page.

**Why do some listings have no `price`?**
Not every domain has a public asking price — some are marketplace-negotiable without a stated minimum. Only listings with a real, published price include `price`/`priceFormatted`; bid-range listings expose `minimumOfferPrice` when Sedo publishes one.

**What currency are prices in?**
Whatever currency Sedo displays for that listing — check `priceCurrency`. Price filters use Sedo's approximate USD equivalent.

**How fresh is the data?**
Every run queries Sedo live; nothing is cached.

**How many rows can I get?**
The marketplace has hundreds of thousands of listings and expiring auctions currently number in the tens of thousands. `maxItems` supports up to 10,000 per run; split by keyword, category, TLD, or price bucket for larger exports. The actor stops on `maxItems`, source exhaustion, or repeated empty pages — never on a wall-clock timer.

**Can I combine filters?**
Yes — TLD, price, traffic, length, sale type, bids, watchlist, and name-shape filters all combine with `searchQuery`, `keywords`, or `categoryId`.
