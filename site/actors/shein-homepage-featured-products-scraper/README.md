# Shein Homepage & Home Feed Scraper

Read what SHEIN is promoting in 32 storefronts without a login, cookie or proxy setup. The actor collects two server-rendered surfaces: the featured products on each regional homepage and the richer `/home` feed, which adds SKU, SPU, brand, rating, review count, sold count and a sales label. It does not offer keyword search, category browse or product detail pages; SHEIN blocks those anonymous routes (see Limitations).

## What this actor does

- **Three feed modes:** `featured` (root homepage, 32 markets), `homeFeed` (20 richer products in 11 markets, root fallback elsewhere) and `all` (both, deduplicated per market).
- **Local and USD pricing:** sale price, retail price, discount percentage, plus SHEIN's own USD equivalent for cross-market comparison.
- **Rich home-feed fields:** brand, SKU, SPU, rating, review count, units sold, sales label and on-sale flag when the feed provides them.
- **Product links and imagery:** canonical product URLs when the feed has a slug, main image and gallery images.
- **Empty fields are omitted.** A field SHEIN does not publish for a particular card is left out rather than sent as `null`.

## Services

- **Storefront snapshots:** capture the current featured assortment for one market or all 32 in a single run.
- **Cross-market price checks:** compare local prices and SHEIN's USD equivalent across storefronts.
- **Promotion monitoring:** track discount depth, sales labels and on-sale flags over time.
- **Catalogue seeding:** collect product IDs, titles, links, prices and images for a lightweight starter catalogue.
- **Assortment research:** see which products each regional storefront chooses to promote.

## How it works

The actor requests the public storefront page, reads the product data that SHEIN renders into that page, and returns one record per product. No account is used, no cookies are passed in, and no browser is required. If a page is temporarily challenged, the actor retries and then tries a different Apify connection automatically; it never asks you for proxy settings.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `featured` | `featured` reads the root homepage; `homeFeed` reads the richer `/home` feed; `all` reads both and keeps one record per product. |
| `countries` | array | `["US"]` | Storefronts to read. Choose any of the 32 supported markets; each returns local prices and its own currency. |
| `maxItems` | integer | `100` | Hard cap on records across all selected storefronts (1–500). Each storefront is read once, then the cap trims the result; once reached, remaining storefronts are not requested. |

Every field has a default, so a run can start with no changes. Existing tasks that still send the old `country` value keep working: the actor maps it to `countries` in code.

### Example: US featured products

```json
{
    "mode": "featured",
    "countries": ["US"],
    "maxItems": 10
}
```

### Example: richer home feed for three markets

```json
{
    "mode": "homeFeed",
    "countries": ["US", "UK", "BR"],
    "maxItems": 60
}
```

### Example: compare featured and home-feed products

```json
{
    "mode": "all",
    "countries": ["US", "DE", "JP"],
    "maxItems": 200
}
```

### Example: legacy input from an existing task

```json
{
    "country": "UK",
    "maxItems": 5
}
```

## Output per product

- `productId` — SHEIN goods ID
- `title` — product name from the feed
- `url` — product page URL on the matching storefront
- `countryCode` — storefront code (`US`, `UK`, `DE`, …)
- `currency` — local ISO currency code
- `mainImage` — main product image URL
- `images` — gallery image URLs
- `salePrice` / `salePriceText` — current price as a number and with its currency symbol
- `retailPrice` / `retailPriceText` — list price as a number and with its currency symbol
- `priceUsd` / `retailPriceUsd` — SHEIN's USD equivalent for the sale and retail price
- `discountPercentage` — discount shown by SHEIN
- `catId` — SHEIN category ID
- `brand` — brand name when the card carries one
- `productSku` — SKU code (home feed)
- `spu` — SPU identifier (home feed)
- `rating` — average customer rating when reviews exist
- `reviewCount` — number of customer reviews
- `soldCount` — units sold as shown on the card (home feed); a `1.5k+ sold` label becomes `1500`
- `salesLabel` — sales label such as `800+ sold` (home feed)
- `isOnSale` — on-sale flag (home feed)
- `badges` — promotional labels shown on the card
- `recordType` — `featured` or `homeFeed`
- `position` — 1-based position in that storefront's returned records (home-feed records come first in `all` mode)
- `scrapedAt` — UTC scrape timestamp

## Use cases

- **Merchandising teams** — snapshot what each storefront is promoting today.
- **Pricing analysts** — compare local prices against SHEIN's USD equivalent across markets.
- **Promotion trackers** — monitor discount depth and sales labels around seasonal events.
- **Market researchers** — study assortment differences between mature and emerging storefronts.
- **Data teams** — seed a product catalogue with stable IDs, titles, links and images.

## Usage cost

You pay for Apify platform usage (run time and memory). There is no per-record charge from this actor and no paid proxy is required. A typical run reads one page per selected storefront, so keeping `countries` focused keeps both runtime and cost down.

## FAQ

**Can I search SHEIN by keyword or browse a category with this actor?**
No. Keyword search, category pages and product detail pages currently answer anonymous requests with a SHEIN risk challenge. This actor deliberately stays on the two storefront feeds that work without a login and documents that boundary instead of returning partial or fabricated detail data.

**Why does a market return fewer products than `maxItems`?**
Each root homepage carries 6–14 products and each home feed carries 20. `maxItems` is a cap, not a quota; the actor never invents products that are not on the page.

**Why are brand, rating, sold count or SKU missing on some records?**
Those fields live on the richer home feed. Root-homepage cards do not carry them, and even on the home feed a card may omit a brand, rating or sales label. Missing values are omitted.

**Why does `homeFeed` return featured records for some countries?**
The `/home` feed is available for US, UK, AU, BR, CA, MX, PH, SG, MY, NZ and ZA. For other markets, and when a feed is temporarily empty, the actor reads the root homepage and labels those records `featured`.

**Are prices live?**
Yes. Every run reads the storefront at run time. Local prices, discounts and USD equivalents reflect that moment.

**Does the featured set change between runs?**
SHEIN can change its storefront merchandising at any time, but within a short test window the feed is usually stable. Do not assume a new set on every run; schedule the actor at the cadence your use case needs.

**Do I need a proxy or a SHEIN account?**
No. The actor works without user credentials and chooses a working connection automatically.

**Is this affiliated with SHEIN?**
No. This is an independent third-party actor that reads publicly available storefront pages. It is not affiliated with or endorsed by SHEIN.

## Notes

- 32 storefronts are supported; each record carries the correct local currency and SHEIN's USD equivalent.
- Home-feed records are richer because the `/home` payload includes SKU, SPU, brand, rating, review count, sold count and a sales label.
- When both feeds contain the same product in `all` mode, the home-feed record is kept and the root duplicate is dropped.
- `soldCount` follows the card's sold label, so `800+ sold` becomes `800`; SHEIN's separate internal counter is used only when no label is shown.
- Records are deduplicated per product and storefront; the same product can appear once for each market.
- Fields are omitted when empty, so downstream code should treat every optional field as optional.

## Limitations

- **No keyword search, category browse or product detail.** Those routes return a SHEIN risk challenge to anonymous requests from every tested connection, so the actor does not offer them. It is a storefront-feed collector, not a full catalogue scraper.
- **No variants, sizes, stock or review text.** Those data points exist only on product detail pages, which are part of the same blocked surface.
- **Fixed row ceiling.** A storefront provides one feed screen: 6–14 root products and 20 home-feed products. There is no reliable anonymous pagination, so a single market tops out at about 34 products; in testing the 32 root pages plus 11 home feeds held about 580 product-and-storefront records (499 unique product IDs).
- **Home feed is not universal.** The 11 listed markets have the rich feed; other markets return root-homepage records even in `homeFeed` mode.
- **Conditional fields.** Rating, brand, sales label and gallery images appear only when SHEIN includes them on a card.
