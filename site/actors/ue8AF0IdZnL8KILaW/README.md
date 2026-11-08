# eBay Item, Store & Store Category Detail Extractor

Extract detailed product data from eBay individual item pages, seller store fronts, and store category pages. Get item specifics, all high-resolution images, seller details, shipping terms, return policies, and more.

## What does this scraper do?

This actor extracts **rich, detailed product data** from three types of eBay pages:

- **Individual item pages** (`/itm/` URLs) — full product details with 30 data fields
- **Seller store pages** (`/str/` URLs) — discovers and scrapes all products from a seller's store
- **Store category pages** — scrapes products from specific categories within a seller's store

Unlike search result scrapers that only extract card-level data, this actor visits each product page to collect complete item specifics, all images, seller ratings, shipping/return terms, stock levels, and sales data.

> **Note**: Some fields may be empty for certain listings: `originalPrice` (only when discounted), `soldCount`/`watchCount`/`availableQuantity` (not shown on all listings), `bids` (only for auctions), `sellerFeedbackPercent` (new sellers with zero feedback), `shippingCost` (when eBay doesn't display a computed cost, e.g. calculated/location-dependent shipping), and `returnPolicy` (eBay Motors/vehicle listings are typically sold as-is via dealer arrangement rather than a standard return policy).

## Features

- Extract 30 detailed data fields per product
- Keyword search — no need to construct a URL, just provide search terms
- Multi-marketplace support — search across 8 eBay country sites (US, UK, DE, FR, IT, ES, CA, AU)
- Vehicle-specific fields (VIN, mileage, title status, transmission) automatically promoted for eBay Motors listings
- Full item specifics extraction (Brand, Model, Processor, RAM, etc.)
- All high-resolution product images (up to 1600px)
- Seller details with feedback score and positive rating percentage
- Shipping cost, return policy, and item location
- Stock availability and units sold count
- Works with Buy It Now and Auction listings
- Automatic pagination for store pages
- No login or cookies required

## Input

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `startUrls` | Array | One of `startUrls`/`searchQueries` | `[{"url": "https://www.ebay.com/sch/i.html?_nkw=iphone+15&_sop=12"}]` | eBay item, store, or store category URLs |
| `searchQueries` | Array | One of `startUrls`/`searchQueries` | `[]` | Plain keyword searches — no URL construction needed |
| `marketplace` | String | No | `https://www.ebay.com` | Which eBay country site to search on (used with `searchQueries`; `startUrls` entries use their own domain) |
| `maxItems` | Integer | No | 3 | Maximum number of items to scrape, shared across ALL `startUrls`/`searchQueries` combined (not per-URL or per-query — e.g. with 3 search queries and `maxItems: 3`, only the first query's results count toward the total) |
| `proxy` | Object | No | `{"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"]}` | Proxy configuration. Residential proxy is the default — eBay's bot protection redirects datacenter IPs to a captcha page. |

### Supported URL Formats

- **Single item**: `https://www.ebay.com/itm/267153498498`
- **Single item (with title)**: `https://www.ebay.com/itm/Product-Title-Here/267153498498`
- **Seller store**: `https://www.ebay.com/str/thriftbooks`
- **Store category**: `https://www.ebay.com/str/thriftbooks/Fiction/_i.html?_sacat=261186`

### Example Input

```json
{
    "startUrls": [
        { "url": "https://www.ebay.com/itm/267153498498" }
    ],
    "maxItems": 10
}
```

### Scraping a Seller Store

```json
{
    "startUrls": [
        { "url": "https://www.ebay.com/str/thriftbooks" }
    ],
    "maxItems": 50
}
```

## Output

Each scraped product contains the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `url` | String | Direct URL to the item listing |
| `id` | String | eBay item ID |
| `title` | String | Full product title |
| `price` | String | Current price with currency (e.g., "US $384.92") |
| `originalPrice` | String | Original price if item is discounted |
| `condition` | String | Item condition (New, Used, Refurbished, etc.) |
| `brand` | String | Product brand, empty if not specified |
| `model` | String | Product model, empty if not specified |
| `seller` | String | Seller name |
| `sellerFeedbackScore` | String | Seller total feedback count |
| `sellerFeedbackPercent` | String | Seller positive feedback percentage (empty for new sellers with no feedback) |
| `sellerUrl` | String | URL to seller's eBay store |
| `availableQuantity` | String | Number of items in stock |
| `soldCount` | String | Number of units sold |
| `watchCount` | String | Number of people watching this item |
| `location` | String | Item geographic location |
| `shippingCost` | String | Shipping cost or "Free shipping", empty if eBay doesn't display a computed cost |
| `returnPolicy` | String | Return policy terms, empty for eBay Motors/vehicle listings |
| `paymentMethods` | Array | Accepted payment methods (e.g. PayPal, Klarna Installment) |
| `listingStartDate` | String | ISO 8601 timestamp when the listing went live, empty if unavailable |
| `images` | Array | All high-resolution product image URLs |
| `itemSpecifics` | Object | Key-value pairs of item specifics (Brand, Model, etc.) |
| `categories` | Array | Breadcrumb category path |
| `authenticityGuarantee` | Boolean | Whether this listing is covered by eBay's Authenticity Guarantee |
| `topRatedSeller` | Boolean | Whether the seller has eBay's Top Rated Plus/Seller badge |
| `businessSeller` | Boolean | Whether the seller is registered as a business seller |
| `listingType` | String | "Buy It Now" or "Auction" |
| `bids` | String | Number of bids (for auction listings) |
| `vehicle` | Object | Vehicle-specific fields (vin, mileage, titleStatus, transmission, drivetrain, fuelType, exteriorColor, interiorColor) — present only for eBay Motors listings |
| `scrapedAt` | String | ISO timestamp when the item was scraped |

### Example Output

```json
{
    "url": "https://www.ebay.com/itm/296175164279",
    "id": "296175164279",
    "title": "Apple iPhone 15 Pro 128GB Factory Unlocked AT&T T-Mobile Verizon",
    "price": "US $649.99",
    "originalPrice": "",
    "condition": "Excellent - Refurbished",
    "brand": "Apple",
    "model": "Apple iPhone 15 Pro",
    "seller": "Discount Computer Depot",
    "sellerFeedbackScore": "156714",
    "sellerFeedbackPercent": "99.3%",
    "sellerUrl": "https://www.ebay.com/str/discountcomputerdepot",
    "availableQuantity": "10",
    "soldCount": "5",
    "watchCount": "12",
    "location": "Jacksonville, TX, United States",
    "shippingCost": "Free shipping",
    "returnPolicy": "30 days returns",
    "paymentMethods": ["Klarna Installment", "PayPal", "Venmo", "Visa", "Master Card"],
    "listingStartDate": "2024-01-19T21:58:01Z",
    "images": [
        "https://i.ebayimg.com/images/g/example1/s-l1600.jpg",
        "https://i.ebayimg.com/images/g/example2/s-l1600.jpg"
    ],
    "itemSpecifics": {
        "Brand": "Apple",
        "Model": "iPhone 15 Pro",
        "Storage Capacity": "128 GB",
        "Color": "Black Titanium",
        "Network": "Unlocked"
    },
    "categories": ["eBay", "Electronics", "Cell Phones & Accessories", "Cell Phones & Smartphones"],
    "authenticityGuarantee": false,
    "topRatedSeller": true,
    "businessSeller": false,
    "listingType": "Buy It Now",
    "bids": "",
    "scrapedAt": "2026-08-22T17:26:07.041000+00:00"
}
```

> This example omits `vehicle` because this listing isn't in the eBay Motors category — that field is only present on vehicle listings (see the Output table above).

## Use Cases

- **Market Research**: Analyze product pricing, conditions, and availability across sellers
- **Price Monitoring**: Track price changes on specific eBay listings over time
- **Competitor Analysis**: Scrape competitor store inventories with full product details
- **Product Catalog Building**: Extract complete product specifications for catalog databases
- **Seller Intelligence**: Collect seller ratings, feedback scores, and store inventory data
- **Supply Chain Research**: Monitor product availability and shipping terms from suppliers

## Tips for best results

1. **Start with individual item URLs** when you only need specific listings
2. **Use store URLs** when you need to scrape an entire seller's inventory
3. **Set `maxItems`** to the number of records you actually need, especially on large stores
4. **Leave the default proxy configuration** — a residential proxy is applied automatically because eBay's bot protection blocks datacenter IPs
5. **Raise `maxItems` accordingly with multiple queries** — it's a shared budget across every URL/query, so with several `searchQueries` or `startUrls` entries, size it so later entries aren't starved by earlier ones (e.g. ~10-20 per query)

## FAQ

### Can I scrape eBay search results with this actor?

Yes — provide plain keywords via `searchQueries`, or a full `/sch/` search URL via `startUrls`. Unlike card-level search scrapers, this actor visits each result's item page for full detail (30 fields).

### Does this actor support international eBay sites?

Yes. Set the `marketplace` field to search a specific country site (US, UK, Germany, France, Italy, Spain, Canada, Australia), or pass `startUrls` on any eBay domain directly — the actor always stays on the domain your URLs use.

### How many items can I scrape per run?

There is no hard limit. Use the `maxItems` parameter to control the number of items scraped per run. For large stores with thousands of products, consider splitting into multiple runs.

### Does this actor need cookies or login?

No. The scraper works with publicly available eBay pages and does not require any authentication.

### What happens if an item is no longer available?

The scraper handles ended and unavailable listings gracefully. It will extract whatever data is still visible on the page.

### How is this different from eBay search scrapers?

Search scrapers extract basic card-level data (title, price, thumbnail) from search result pages. This actor visits each individual product page to extract 30 fields including full item specifics, all images, seller details, shipping/return terms, and sales data.
