# Alibaba.com Wholesale Sourcing & Supplier Vetting Finder

Sourcing from Alibaba.com means comparing manufacturers on more than price. Trade Assurance coverage, third-party assessment, tenure, response rate, and minimum order quantities decide whether a supplier is worth contacting. This actor reads Alibaba.com's public wholesale and country-search pages, then returns either product rows or supplier shortlists — no Alibaba login, cookie, API key, or proxy setup required.

## What this actor does

- **Search products** by keyword across Alibaba.com's wholesale catalogue and paginate through the source's own result pages.
- **Search suppliers** by aggregating the unique suppliers behind those product results, one row per supplier.
- **Fetch product details** from product URLs or numeric product IDs: price band, MOQ, attributes, images, supplier performance, Trade Assurance, and sold count.
- **Fetch supplier profiles** from minisite URLs or supplier IDs: tenure, location, business type, employees, factory size, main markets, certifications, payment terms, and recent transaction volume.
- Filter by Trade Assurance, third-party assessed suppliers, supplier country, price band, supplier tenure, and supplier rating.
- Sort by price, rating, tenure, or transaction level before the `maxItems` cut-off.
- Omit any field Alibaba did not publish for a listing instead of returning blank or fabricated values.

## Modes

| Mode | What it returns | Input it uses |
|---|---|---|
| `searchProducts` | One row per product listing | `keyword` plus optional filters |
| `searchSuppliers` | One row per unique supplier behind the matching products | `keyword` plus optional filters |
| `productDetails` | Full detail for each product URL/ID | `startUrls` |
| `supplierDetails` | Full profile for each supplier URL/ID | `startUrls` |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | Select | `searchProducts` | One of the four modes above. |
| `keyword` | String | `LED lights` | Product or supplier keyword for the two search modes. |
| `tradeAssuranceOnly` | Boolean | `false` | Keep only listings with Alibaba Trade Assurance. |
| `verifiedSupplierOnly` | Boolean | `false` | Keep only Gold Suppliers or third-party assessed suppliers. |
| `supplierCountry` | String | – | Two-letter ISO code, e.g. `CN`, `IN`, `PK`, `US`. |
| `minPrice` / `maxPrice` | Number | – | Filter on the lowest listed unit price. |
| `minSupplierYears` | Integer | – | Minimum years the supplier has been on Alibaba.com. |
| `minSupplierRating` | Number | – | Minimum supplier rating out of 5. |
| `sortBy` | Select | `relevance` | `relevance`, `priceLowToHigh`, `priceHighToLow`, `supplierRating`, `yearsOnPlatform`, or `transactionLevel`. |
| `startUrls` | Array | – | Product URLs/IDs for `productDetails`; supplier minisite URLs or subdomains for `supplierDetails`. |
| `maxItems` | Integer | `20` | Maximum records to save (1–200). |
| `maxPages` | Integer | `5` | Safety limit on result pages (about 40 offers each). |

### Example: compare verified Indian LED suppliers

```json
{
  "mode": "searchProducts",
  "keyword": "LED lights",
  "supplierCountry": "IN",
  "tradeAssuranceOnly": true,
  "verifiedSupplierOnly": true,
  "minSupplierYears": 3,
  "sortBy": "priceLowToHigh",
  "maxItems": 30
}
```

### Example: build a supplier shortlist

```json
{
  "mode": "searchSuppliers",
  "keyword": "solar panel",
  "tradeAssuranceOnly": true,
  "minSupplierRating": 4.5,
  "sortBy": "transactionLevel",
  "maxItems": 25
}
```

### Example: enrich known products

```json
{
  "mode": "productDetails",
  "startUrls": [
    "https://www.alibaba.com/product-detail/4-Sides-Solar-Garden-Light-Motion_1600132781729.html",
    "1600132781729"
  ]
}
```

### Example: inspect a supplier minisite

```json
{
  "mode": "supplierDetails",
  "startUrls": ["https://shinedo.en.alibaba.com/"]
}
```

## Output per product

| Field | Description |
|---|---|
| `productId` | Alibaba product identifier. |
| `title` | Listing title. |
| `price` | Original price text, e.g. `US $1.30-$7.36`. |
| `priceMin` / `priceMax` | Numeric price band. |
| `priceCurrency` | Currency parsed from the price text. |
| `priceUnit` / `unit` | Unit the price and MOQ refer to. |
| `minOrder` / `minOrderQuantity` | MOQ text and numeric quantity. |
| `supplier` | Supplier company name. |
| `supplierId` | Alibaba company ID on search rows; minisite slug on detail rows. |
| `supplierSlug` | Minisite subdomain when available. |
| `supplierUrl` | Supplier profile or minisite link. |
| `supplierLocation` / `supplierCountryCode` | Supplier country. |
| `supplierRating` / `supplierRatingValue` | Store rating out of 5. |
| `supplierReviewCount` | Number of store reviews. |
| `supplierProductRating` | Product review score when Alibaba exposes it. |
| `yearsOnPlatform` / `supplierYears` | Supplier tenure on Alibaba.com. |
| `tradeAssurance` | Present when the supplier offers Trade Assurance. |
| `supplierVerified` / `supplierGold` | Third-party assessment and Gold Supplier flags. |
| `supplierResponseRate` | Supplier response rate. |
| `supplierTransactionLevel` | Alibaba transaction level. |
| `supplierHistoryGmv` / `supplierOrderCount` | Recent trade history. |
| `mainProducts` | Supplier's main product lines. |
| `soldCount` | Units sold, when the result card publishes it. |
| `responseTime` | Typical response time on detail rows. |
| `onTimeDeliveryRate` / `reorderRate` | Supplier performance on detail rows. |
| `productAttributes` | Full attribute map on detail rows (certifications, IP rating, material, etc.). |
| `category` | Breadcrumb category path on detail rows. |
| `images` / `imageUrl` | Product gallery and primary image. |
| `productUrl` | Canonical product page. |
| `searchKeyword` / `position` | Search context on search rows. |
| `recordType` / `scrapedAt` | `product` and collection timestamp. |

## Output per supplier

| Field | Description |
|---|---|
| `supplierId` / `companyId` | Minisite slug when available (Alibaba company ID otherwise) plus the company ID. |
| `supplier` | Company name. |
| `supplierUrl` / `supplierLogo` | Minisite link and logo. |
| `supplierLocation` / `supplierCountryCode` | Province/country and ISO code. |
| `supplierRating` / `supplierRatingValue` / `supplierReviewCount` | Store rating and review count. |
| `yearsOnPlatform` / `supplierYears` | Tenure on Alibaba.com. |
| `tradeAssurance` | Present when Trade Assurance is offered. |
| `supplierVerified` / `supplierGold` | Assessment and Gold Supplier flags. |
| `supplierResponseRate` / `supplierTransactionLevel` | Response and transaction performance. |
| `supplierHistoryGmv` / `supplierOrderCount` | Recent trade history. |
| `mainProducts` | Main product lines. |
| `matchingProductCount` | Products from this supplier in the current result set. |
| `priceMin` / `priceMax` / `priceCurrency` | Lowest and highest listed prices among those products. |
| `yearEstablished`, `employees`, `factorySize`, `businessType` | Firmographics on supplier-profile rows. |
| `supplierMainMarkets`, `annualRevenue`, `certifications` | Company overview fields on supplier-profile rows. |
| `acceptedPaymentMethods`, `acceptedDeliveryTerms`, `acceptedPaymentCurrency` | Trade terms on supplier-profile rows. |
| `recentTransactionAmount` / `recentTransactionCount` | Alibaba's six-month transaction summary. |
| `assessmentCompany` | Third-party assessment provider. |
| `supplierDescription` / `productGroups` | Company description and product groups. |
| `recordType` / `scrapedAt` | `supplier` and collection timestamp. |

## Data source and reliability

Alibaba.com's interactive `/trade/search` page is captcha-protected. This actor instead uses the public SEO wholesale pages (`/wholesale/<keyword>.html`) and country-search pages (`/countrysearch/<CC>/<keyword>.html`), which embed the full offer payload in `window._PAGE_DATA_`. Pagination follows the source's own `_<page>.html` URL rule, and duplicate listings across pages are removed by product ID.

Product and supplier detail pages are fetched only when that mode is selected. They occasionally show a captcha; the actor retries with backoff and, on Apify, can fall back to the included datacenter proxy. No user proxy or credential is needed.

## Limits

- Search pages expose around 40 offers per page; `maxPages` caps how deep a run goes. Alibaba caps pagination at 100 pages for these SEO routes.
- United States and some other country catalogues are smaller than the China catalogue; use `supplierCountry` to target a market.
- Trade Assurance and third-party assessment flags are shown only when Alibaba publishes them for that supplier.
- Supplier contact emails and phone numbers are not public. Use the supplier's Alibaba profile to send an inquiry.
- Prices are the seller's listed unit prices. Shipping, duties, and taxes are not included.

## FAQ

**Is this Alibaba.com or AliExpress?**
Alibaba.com's B2B wholesale marketplace, not AliExpress's retail storefront.

**Do I need an Alibaba account or cookies?**
No. Every mode works from public pages with no login.

**Why is the price a range?**
Wholesale sellers publish tiered or range pricing because the unit price depends on order volume. `priceMin` and `priceMax` give the numeric band; `price` preserves the seller's original text.

**What counts as a verified supplier?**
A Gold Supplier or a supplier assessed by a third-party inspection company. The two signals are also reported separately as `supplierGold` and `supplierVerified`.

**Why do some rows lack MOQ, sold count, or response time?**
Alibaba does not publish those fields on every page type. They are omitted, never filled with placeholders.

**Can I get more than 40 results?**
Yes. Raise `maxItems` and `maxPages`; the actor paginates and de-duplicates by product ID or supplier ID.

**Is this affiliated with Alibaba.com?**
No. This actor independently reads public Alibaba.com pages and is not affiliated with or endorsed by Alibaba Group.
