# eBay Multi-Marketplace Listing & Seller Data Scraper

Search eBay listings by keyword or category across 8 international marketplaces. Extract product data including titles, prices, conditions, brands, seller feedback, product ratings, watch counts, sold counts, shipping costs, and more.

## What does eBay Items Scraper do?

This actor lets you search and extract structured data from eBay listings using keyword searches or category pages across different international marketplaces. Specify search terms, apply filters for condition, price range, buying format, and sort order to find exactly the products you need.

Whether you need to monitor competitor pricing, track product availability, conduct market research, or build a product database, this scraper handles the heavy lifting.

## Features

- Search eBay by keywords across 8 global marketplaces (US, UK, Germany, France, Italy, Spain, Australia, Canada)
- Filter by condition (All conditions, New, Open box, Refurbished, Used, For parts / not working)
- Filter by buying format (Buy It Now, Auction)
- Apply price range filters (minimum and maximum price)
- Sort results (Ending Soonest, Newly Listed, Price Low to High, Price High to Low)
- Extract up to 22 data fields per item including seller feedback, product ratings, and sales metrics
- Support for both keyword search and direct URL input modes
- No login or cookies required
- Reliable access to every marketplace via an automatically applied residential proxy

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `startUrls` | Array | – | One or more eBay search result or category URLs (alternative to `keywords`) |
| `keywords` | String | – | Search term(s) to find items on eBay |
| `marketplace` | String | `US` | eBay marketplace to search (US, UK, Germany, France, Italy, Spain, Australia, Canada) |
| `condition` | String | – | Filter by item condition (All conditions, New, Open box, Refurbished, Used, For parts / not working) |
| `buyingFormat` | String | – | Filter by buying format (Buy It Now, Auction) |
| `minPrice` | Number | – | Minimum price filter (in the marketplace's local currency) |
| `maxPrice` | Number | – | Maximum price filter (in the marketplace's local currency) |
| `sortBy` | String | – | Sort order (Best Match - default, Ending Soonest, Newly Listed, Price Low to High, Price High to Low) |
| `maxItems` | Integer | `50` | Maximum number of items to scrape |
| `proxy` | Object | – | Proxy configuration; a residential proxy is used automatically even if left unset, since eBay blocks non-residential traffic |

### Marketplace Options

- **US**: ebay.com
- **UK**: ebay.co.uk
- **Germany**: ebay.de
- **France**: ebay.fr
- **Italy**: ebay.it
- **Spain**: ebay.es
- **Australia**: ebay.com.au
- **Canada**: ebay.ca

### Example Input (Keyword Search)

```json
{
    "keywords": "laptop",
    "marketplace": "US",
    "condition": "new",
    "buyingFormat": "buyItNow",
    "minPrice": 500,
    "maxPrice": 1500,
    "sortBy": "priceLowest",
    "maxItems": 50
}
```

### Example Input (Direct URLs)

```json
{
    "startUrls": [
        { "url": "https://www.ebay.com/sch/i.html?_nkw=laptop&_sop=12" }
    ],
    "maxItems": 50
}
```

## Output

Each scraped item contains the following fields, present when the listing actually has that data (empty fields are omitted rather than returned as null):

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | eBay item ID |
| `url` | String | Direct URL to the item listing |
| `title` | String | Item title |
| `imageUrl` | String | URL of the item thumbnail image |
| `price` | String | Item price with currency symbol (e.g., "$299.99") |
| `wasPrice` | String | Original price if item is discounted |
| `condition` | String | Item condition (New, Used, Refurbished, etc.) |
| `brand` | String | Product brand or manufacturer |
| `seller` | String | Seller name |
| `sellerFeedbackPercent` | String | Seller's positive feedback percentage |
| `sellerFeedbackCount` | String | Total number of seller feedback ratings |
| `productRating` | String | Product rating (e.g., "4.5 out of 5 stars") |
| `reviewCount` | String | Number of product reviews |
| `watchCount` | String | Number of users watching this item |
| `soldCount` | String | Number of units sold |
| `shippingCost` | String | Shipping cost or "Free shipping" |
| `itemLocation` | String | Geographic location of the item |
| `listingType` | String | "Buy It Now" or "Auction" |
| `bids` | String | Number of bids (for auction listings) |
| `isSponsored` | Boolean | Whether this is a sponsored/featured listing |
| `searchUrl` | String | The search URL this item was found on |
| `scrapedAt` | String | ISO timestamp when the item was scraped |

### Example Output

```json
{
    "id": "123456789012",
    "url": "https://www.ebay.com/itm/123456789012",
    "title": "Dell Latitude 15.6\" Laptop Intel Core i7 16GB RAM 512GB SSD",
    "imageUrl": "https://i.ebayimg.com/images/g/abc123/s-l500.jpg",
    "price": "$299.99",
    "wasPrice": "$399.99",
    "condition": "Refurbished",
    "brand": "Dell",
    "seller": "best_deals_store",
    "sellerFeedbackPercent": "99.2%",
    "sellerFeedbackCount": "15432",
    "productRating": "4.5 out of 5 stars",
    "reviewCount": "287",
    "watchCount": "12",
    "soldCount": "5",
    "shippingCost": "Free shipping",
    "itemLocation": "United States",
    "listingType": "Buy It Now",
    "isSponsored": false,
    "searchUrl": "https://www.ebay.com/sch/i.html?_nkw=laptop&_sop=12",
    "scrapedAt": "2026-04-01T12:00:00.000000+00:00"
}
```

## Use Cases

- **Price monitoring**: Track prices for specific products across sellers and marketplaces
- **Market research**: Analyze pricing trends, product availability, and seller competition in a category
- **Competitor analysis**: Monitor competitor product listings and pricing strategies
- **Lead generation**: Find sellers and products in specific niches across regions
- **Product sourcing**: Discover deals and compare prices across listings and countries
- **Inventory tracking**: Monitor stock levels and new listings for specific items
- **International comparison**: Compare product prices and availability across 8 global marketplaces

## Tips for best results

1. **Use specific keywords** — more specific search terms return more relevant results
2. **Apply filters** — use condition, price range, and format filters to narrow results to exactly what you need
3. **Set `maxItems`** — cap the run at the number of listings you actually need
4. **Choose a sort order** — select "Newly Listed" for recent items or "Price Low to High" to find deals
5. **Pick the right marketplace** — search the marketplace matching your target region to get results in the local currency and language

## Frequently Asked Questions

### Does this scraper require an eBay account?

No. The scraper works with publicly available eBay pages and does not require any login or authentication.

### Can I scrape eBay sites outside the US?

Yes. The scraper supports 8 major international eBay marketplaces: US, UK, Germany, France, Italy, Spain, Australia, and Canada. Prices and text will be in the local format.

### How many items can I scrape per run?

You can scrape hundreds of items per run. The `maxItems` parameter controls how many items are collected. Each search results page contains approximately 48-60 items depending on the marketplace.

### Do I need to configure a proxy?

No. A residential proxy is applied automatically, matched to the marketplace's own country, even if you leave the `proxy` input unset — eBay blocks most non-residential traffic. You only need to touch `proxy` if you want to supply your own proxy groups.

### What if some fields are empty?

Some fields may be absent depending on the listing type and available information. For example, `bids` is only populated for auction listings, `wasPrice` only appears if the item is discounted, and `brand` may not be available for all products. Fields with no value are omitted from the record rather than returned as an empty string or null.

### How is this different from searching eBay manually?

This scraper automates the process of finding and extracting data from eBay listings. Instead of manually clicking through pages and collecting data, you specify your search criteria once and the scraper does the work, extracting structured data you can use in spreadsheets, databases, or analysis tools.

### Can I search by category instead of keywords?

Yes. You can provide direct eBay category URLs via `startUrls`, and the scraper will extract items from those category pages.

### Which fields are always populated?

The core fields `id`, `url`, `title`, `price`, and `seller` are typically always populated. Other fields like `wasPrice`, `brand`, `bids`, and `productRating` are present when applicable for that specific listing type.
