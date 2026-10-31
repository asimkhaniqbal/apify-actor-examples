# Allegro.pl Listings by Search, Category or URL

Pull product listings off **Allegro.pl**, Poland's largest online marketplace, three ways: type a keyword, point at a category page, or paste a list of offer URLs you already have. Each result comes back with price, seller name and rating, condition, shipping, images, and rating details. No API key required.

## What this actor does

- **Three modes:** `search` (keyword search), `byCategory` (browse category URL), `byUrl` (fetch specific offers)
- **Filters:** condition (new/used), free shipping toggle, price range (min/max in PLN)
- **Sort options:** relevance, price low-to-high, price high-to-low, popularity, newest
- **Seller data:** seller login, positive rating percentage
- **Pagination:** automatically pages through results up to your `maxItems` limit
- **Empty fields are omitted** — only populated fields appear in output records

## Output per listing

- `offerId` — Allegro internal offer identifier
- `title` — product title as shown on Allegro
- `url` — full URL to the offer page
- `price` — numeric price in PLN
- `currency` — always `PLN`
- `condition` — `new`, `used`, or `refurbished`
- `seller` — seller login name
- `sellerRating` — seller positive feedback percentage (0–100)
- `freeShipping` — `true` if offer includes free delivery
- `imageUrl` — main product image URL
- `category` — Allegro category name
- `rating` — average product rating (0–5)
- `reviewCount` — number of ratings/reviews
- `location` — seller location (city, country)
- `recordType` — always `listing`
- `scrapedAt` — UTC timestamp of when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byCategory` / `byUrl` |
| `searchQuery` | string | – | Keyword search query (mode=search) |
| `categoryUrl` | string | – | Allegro category URL, e.g. `https://allegro.pl/kategoria/laptopy-491` (mode=byCategory) |
| `startUrls` | array | – | List of offer URLs to fetch (mode=byUrl) |
| `sortBy` | string | `relevance` | Sort order: `relevance`, `price_asc`, `price_desc`, `popularity`, `newest` |
| `conditionFilter` | string | `all` | Product condition: `all`, `new`, `used` |
| `freeShippingOnly` | boolean | `false` | If true, only returns listings with free shipping |
| `minPrice` | integer | – | Minimum price in PLN |
| `maxPrice` | integer | – | Maximum price in PLN |
| `maxItems` | integer | `20` | Maximum listings to return (1–500) |
| `proxyConfiguration` | object | AUTO | Apify proxy config (AUTO datacenter recommended) |

### Example: search for laptops under 3000 PLN

```json
{
  "mode": "search",
  "searchQuery": "laptop",
  "conditionFilter": "new",
  "maxPrice": 3000,
  "sortBy": "price_asc",
  "maxItems": 50
}
```

### Example: browse a category

```json
{
  "mode": "byCategory",
  "categoryUrl": "https://allegro.pl/kategoria/laptopy-491",
  "sortBy": "popularity",
  "freeShippingOnly": true,
  "maxItems": 100
}
```

### Example: fetch specific offers

```json
{
  "mode": "byUrl",
  "startUrls": [
    {"url": "https://allegro.pl/oferta/laptop-dell-inspiron-123456789"},
    {"url": "https://allegro.pl/oferta/macbook-pro-987654321"}
  ]
}
```

### Example: used electronics under 500 PLN

```json
{
  "mode": "search",
  "searchQuery": "telefon komórkowy",
  "conditionFilter": "used",
  "maxPrice": 500,
  "sortBy": "newest",
  "maxItems": 30
}
```

## Use cases

- **Price monitoring** — track laptop, smartphone, and electronics prices across Poland
- **Market research** — analyze seller competition and pricing strategies on Allegro
- **Product catalog building** — collect product data for comparison sites or databases
- **Deal hunting** — find free-shipping offers below a target price point
- **E-commerce intelligence** — monitor category trends and top sellers on Poland's largest marketplace
- **Arbitrage research** — find price discrepancies between Allegro and other marketplaces

## FAQ

**What is Allegro?**
Allegro is Poland's largest online marketplace, operating since 1999. It has over 135 million active offers from both professional sellers and private individuals, covering electronics, fashion, home goods, automotive parts, and more.

**What language is the data in?**
Most titles, categories, and descriptions are in Polish, as Allegro primarily serves the Polish market. Some international sellers list in English.

**Why might some records be missing fields like `rating` or `imageUrl`?**
Not all listings have reviews or photos. The actor only includes fields that have actual data — empty fields are omitted to keep records clean.

**Does this actor work for all Allegro country domains?**
The actor is optimized for `allegro.pl` (Poland). Allegro also operates in other CEE countries (Czech Republic, Slovakia, Hungary, etc.) with similar URL structures — category and offer URLs from those domains are accepted.

**How many results can I get per run?**
Set `maxItems` up to 500. A typical search returns 60 listings per page; the actor paginates automatically.

**Can I filter by price in a specific currency?**
Allegro uses PLN (Polish Złoty) as its primary currency. Set `minPrice` and `maxPrice` in PLN.

**Is this actor affiliated with Allegro?**
No. This is an independent third-party scraper that uses Allegro's public website. It is not affiliated with, endorsed by, or connected to Allegro Group.

**Why does the actor use a proxy?**
Allegro's website employs anti-bot measures that may block repeated automated requests from the same IP. Using Apify's AUTO proxy pool with rotating datacenter IPs helps ensure reliable scraping.
