# Shein Homepage Featured Products Scraper

Shein's regional homepages each surface a rotating set of featured fashion products with local pricing. This actor reads one country's homepage and returns those featured products as structured data — no keyword search, no category browsing, just what Shein is currently promoting in that market.

## What this actor does

- **Reads a regional Shein homepage** for the country you choose and extracts its server-rendered featured products
- **Returns local-currency pricing** — sale price and retail price, both as numbers and as pre-formatted text with the local currency symbol
- **Covers 22 country storefronts**, from the US and UK to Brazil, Israel, Singapore and South Africa
- **Captures discount and imagery** when Shein includes them for a given product
- Fields Shein doesn't provide for a product (such as a discount percentage) are left out of that record rather than sent as empty values

## Output per product

- `productId` — Shein's internal goods ID
- `title` — product name
- `url` — link to the product page on the matching country storefront
- `countryCode` — the 2-letter country code requested
- `mainImage` — main product image URL
- `currency` — local currency code (e.g. `USD`, `EUR`)
- `salePrice` / `salePriceText` — current sale price as a number and as formatted text with symbol
- `retailPrice` / `retailPriceText` — original retail price as a number and as formatted text with symbol
- `discountPercentage` — percentage off retail, when Shein shows one
- `catId` — Shein's internal category ID for the product, when available
- `brand` — brand name, when Shein attaches one
- `productSku` — Shein's SKU code for the product, when available
- `rating` — average customer rating, when the product has reviews
- `reviewCount` — number of customer reviews, when the product has reviews
- `videoUrl` — product video URL, when Shein includes one
- `badges` — promotional labels Shein shows on the product, e.g. "NEW" or "FLASH SALE", when present
- `images` — additional product image URLs, when available
- `scrapedAt` — UTC timestamp of the scrape

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `country` | string | `US` | Which Shein regional homepage to read (22 storefronts: US, UK, DE, FR, ES, IT, AU, BR, CA, MX, NL, SE, PL, CH, PT, AT, IL, PH, SG, MY, NZ, ZA). |
| `maxItems` | integer | `30` | Maximum featured products to return (1–200). A single homepage typically has only 6–16 featured products. |

### Example: default US homepage

```json
{
    "country": "US",
    "maxItems": 30
}
```

### Example: German storefront, smaller batch

```json
{
    "country": "DE",
    "maxItems": 10
}
```

### Example: emerging market check

```json
{
    "country": "ZA",
    "maxItems": 20
}
```

## Use cases

- **Daily trend tracking** — see what Shein is promoting on a given country's homepage each day
- **Cross-market pricing comparison** — compare sale prices and discount depth between storefronts, e.g. US vs. Brazil
- **Rolling catalog building** — run repeatedly to accumulate a broader sample, since the featured set changes between runs
- **Promotional monitoring** — watch discount percentages shift across seasons or sales events
- **Regional assortment research** — see how product selection differs between mature and emerging Shein markets

## FAQ

**Can I search Shein by keyword or category with this actor?**
No. This actor only reads the featured products shown on the homepage of the country you select; it does not browse categories or search by keyword.

**Why do I get a different set of products each time I run it?**
Shein rotates its featured homepage selection per browsing session, so consecutive runs for the same country commonly return different products. Running it repeatedly builds up a wider sample over time.

**Why is the product count usually well under my `maxItems`?**
A Shein homepage typically only features 6–16 products in a single load. `maxItems` is a cap, not a guarantee — it won't manufacture extra products that aren't on the page.

**Why are fields like `discountPercentage`, `brand`, `rating` or `images` sometimes missing?**
Those fields only appear when Shein's homepage data includes them for a given product; not every featured item carries a discount, brand name, customer rating, or additional images.

**Does pricing reflect real-time Shein prices?**
Yes — every run reads the live homepage at the time it runs, so prices and the featured set reflect that moment.

**Is this affiliated with Shein?**
No, this is an independent, third-party actor that reads Shein's public homepage. It is not affiliated with or endorsed by Shein.
