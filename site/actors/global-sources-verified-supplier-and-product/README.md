# Global Sources Verified Supplier & Product Finder

Global Sources is a B2B trade platform where buyers source manufacturers and products, mostly out of Asia, and where some supplier profiles carry an on-site "audited/verified" badge. This actor runs product keyword searches, supplier keyword searches, or category browsing, and can restrict any of the three to audited/verified suppliers only.

## What this actor does

- **Three modes:** `searchProducts` (product keyword search), `searchSuppliers` (supplier keyword search), `categoryBrowse` (browse a fixed product category instead of a keyword)
- **Verified-only filter** drops any product or supplier that doesn't carry a verified/audited badge
- **Paginates automatically** through result pages until `maxItems` is reached or a page returns fewer than 5 results
- **Covers ten top-level categories** for `categoryBrowse` — Electronics, Fashion & Accessories, Home & Garden, Auto & Transportation, Gifts & Premiums, Health & Beauty, Computer & Networking, Machinery & Industrial, Building & Construction, Telecommunications
- **Empty fields are omitted** — a listing only has the fields Global Sources actually displayed for it

## Output per product (modes `searchProducts`, `categoryBrowse`)

- `productId` — identifier parsed from the product URL
- `title` — product listing title
- `price` — price or price range as shown on the listing
- `minOrder` — minimum order quantity, as published
- `supplier` — manufacturer/supplier name shown on the listing
- `supplierCountry` — supplier's country
- `imageUrl` — product thumbnail image
- `productUrl` — Global Sources product listing URL
- `verifiedStatus` — verification/audit badge text, when shown
- `recordType` — always `product`
- `scrapedAt` — UTC timestamp of when the record was captured

## Output per supplier (mode `searchSuppliers`)

- `supplierId` — identifier parsed from the supplier profile URL
- `supplier` — company name
- `supplierCountry` — company's country
- `products` — main product categories the company lists
- `verifiedStatus` — verification/audit badge text, when shown
- `supplierUrl` — Global Sources supplier profile URL
- `recordType` — always `supplier`
- `scrapedAt` — UTC timestamp of when the record was captured

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchProducts` | `searchProducts` / `searchSuppliers` / `categoryBrowse` |
| `keyword` | string | `solar panel` | Product or supplier keyword (used for `searchProducts`/`searchSuppliers`; falls back to this default if left empty) |
| `category` | string | `Electronics` | Product category to browse (mode=`categoryBrowse`); leaving it blank falls back to a `keyword` product search instead of browsing all categories |
| `verifiedOnly` | boolean | `false` | Keep only listings with a verified/audited supplier badge |
| `maxItems` | integer | `20` | Maximum number of records to return (1–500) |

### Example: product search, verified suppliers only

```json
{
  "mode": "searchProducts",
  "keyword": "LED lights",
  "verifiedOnly": true,
  "maxItems": 40
}
```

### Example: supplier search

```json
{
  "mode": "searchSuppliers",
  "keyword": "solar panel",
  "maxItems": 20
}
```

### Example: browse a category

```json
{
  "mode": "categoryBrowse",
  "category": "Machinery-Industrial",
  "maxItems": 50
}
```

## Use cases

- **Sourcing shortlists** — pull a first-pass list of manufacturers for a product you want to import
- **Verified-supplier vetting** — filter straight to audited suppliers before reaching out
- **Category price scanning** — browse a whole category to see the spread of prices and MOQs
- **Competitive intelligence** — see which suppliers are listing a specific product keyword and at what price
- **Procurement research** — compare minimum order quantities across multiple suppliers before negotiating

## FAQ

**What does a "verified" or "audited" badge mean?** Global Sources runs on-site factory audits for some suppliers and marks their listings accordingly; it's a trust signal the platform itself assigns, not something this actor calculates.

**Does `verifiedOnly` work in every mode?** Yes — it applies to products, suppliers, and category browsing alike, dropping any record without a verification badge.

**Why do some products show a price range instead of one number?** Global Sources often publishes a price range tied to order quantity tiers; the actor returns exactly what's shown on the listing rather than picking one number.

**Why are some supplier records missing `products`?** That field only appears when the supplier's listing shows its product categories; not every profile publishes this.

**Can I browse a category and filter by keyword at the same time?** No — as long as `category` is set, `categoryBrowse` ignores `keyword` and lists everything in that category; use `searchProducts` with a keyword if you need both a category feel and a specific term. If `category` is left blank ("All categories"), the run falls back to a `keyword` product search instead of browsing every category.

**Do I need a Global Sources account to run this?** No — it reads only the public search, category, and listing pages that are visible without logging in.

**Is this affiliated with Global Sources?** No, this is an independent third-party actor that reads Global Sources' public pages. It is not affiliated with, endorsed by, or sponsored by Global Sources Ltd.
