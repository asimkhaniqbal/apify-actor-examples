# Yahoo Auctions Japan Keyword & Category Price Scraper

[Yahoo Auctions Japan](https://auctions.yahoo.co.jp) (ヤフオク) is Japan's largest online auction marketplace. This actor searches current listings by keyword or browses a specific category, returning each listing's price, bid count, condition and end time, with no account needed.

## What this actor does

- **Searches by keyword** across current Yahoo Auctions Japan listings
- **Browses a specific category** by its category ID instead of a keyword
- **Filters by current bid price** (minimum and/or maximum, in yen)
- **Sorts results** by ending soonest, price, bid count, newest listed, or most popular
- **Empty fields are omitted** — a listing only includes the fields Yahoo Auctions actually published for it

## Output per listing

| Field | Description |
|---|---|
| `itemId` | Yahoo Auctions item ID |
| `title` | Listing title |
| `currentPrice` | Current bid price in yen |
| `currency` | Always `JPY` |
| `buyItNowPrice` | Buy-it-now price in yen, when the listing offers one |
| `condition` | Item condition, e.g. New, Used, Used – Like New |
| `bids` | Number of bids placed so far |
| `endDate` | Auction end date/time |
| `seller` | Seller name or ID |
| `imageUrl` | Item image URL |
| `category` | Category the listing is filed under |
| `itemUrl` | Direct link to the listing |
| `scrapedAt` | ISO 8601 timestamp of the run |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchItems` | `searchItems` (keyword search) or `browseCategory` (browse by category ID) |
| `query` | string | – | Keyword to search for (mode=searchItems) |
| `category` | string | – | Category ID to browse, e.g. `23336` (mode=browseCategory) |
| `sortBy` | string | `endTime` | `endTime` (ending soon), `price_asc`, `price_desc`, `bids`, `new`, or `popular` |
| `minPrice` | integer | – | Only include listings at or above this current bid price (JPY) |
| `maxPrice` | integer | – | Only include listings at or below this current bid price (JPY) |
| `maxItems` | integer | `20` | Maximum number of listings to return (1–500) |

### Example: search ending soon

```json
{
  "mode": "searchItems",
  "query": "PlayStation 5",
  "sortBy": "endTime",
  "maxItems": 50
}
```

### Example: browse a category sorted by bid count

```json
{
  "mode": "browseCategory",
  "category": "23336",
  "sortBy": "bids",
  "maxItems": 100
}
```

### Example: keyword search within a price band

```json
{
  "mode": "searchItems",
  "query": "Nintendo Switch",
  "minPrice": 5000,
  "maxPrice": 20000,
  "sortBy": "price_asc",
  "maxItems": 40
}
```

## Use cases

- **Resale and arbitrage pricing** — see current bid levels and buy-it-now prices for a product before sourcing it
- **Collector tracking** — monitor listings for a specific item by keyword as they come up for auction
- **Category trend-watching** — browse a category sorted by newest or most popular to spot demand shifts
- **Deadline-driven bidding** — sort by ending soonest to find auctions closing within your window
- **Price-band shopping** — use `minPrice`/`maxPrice` to filter out listings outside your budget

## FAQ

**Does this require a Yahoo account or API key?**
No. The actor reads Yahoo Auctions Japan's publicly accessible listing pages.

**What currency are prices in?**
Always Japanese yen (JPY) — `currentPrice` and `buyItNowPrice` are both yen amounts.

**Can I search using Japanese text?**
Yes — `query` accepts Japanese keywords directly, the same as typing into Yahoo Auctions' own search box.

**Why do some listings have no `buyItNowPrice`?**
Not every auction offers a buy-it-now option — the field is left out rather than set to zero when it isn't offered.

**Does this cover sold/completed auctions?**
No — this actor covers currently active listings only, not historical sold prices.

**How do I find a category ID for `browseCategory`?**
Browse to that category on auctions.yahoo.co.jp and copy the numeric ID from the category's URL.

**Is this affiliated with Yahoo Japan?**
No, this is an independent third-party actor that reads Yahoo Auctions Japan's publicly available pages. It is not affiliated with or endorsed by Yahoo Japan Corporation.
