# Blocket.se Classifieds Search & Price Filter Tool

Blocket.se is Sweden's largest second-hand marketplace: cars, electronics, furniture, tools and almost everything else. This actor searches its public listings by keyword, narrows results with real Blocket filters, follows pagination until your item limit, and can open individual ads for the full description, condition, category and seller details.

Everything runs cookieless. No Blocket account, login, cookie or proxy setup is needed.

## What this actor does

- Searches Blocket by keyword across all categories, or restricts the search to cars, electronics or furniture.
- Applies Blocket's own filters: price range in SEK, condition, private or company seller, ad type, Swedish county, publication date, sort order, and car-specific year, mileage, fuel, gearbox and body type.
- Walks the result pages until `maxItems` is reached, removes duplicate ads, and stops at Blocket's own pagination ceiling.
- Fetches full ad details by URL or ad ID, including description, category path, condition, specifications and public seller contact data.
- Optionally enriches every search result with the full ad page (`includeDetails`).
- Leaves a field out whenever Blocket does not publish it for that ad instead of emitting `null` or a fake value.

## Output per listing

Core fields:

| Field | Description |
|---|---|
| `adId` | Blocket ad ID |
| `title` | Listing headline |
| `price` | Asking price in SEK |
| `currency` | Currency code, normally `SEK` |
| `priceExVat` | Price excluding VAT when the seller publishes it (mostly cars) |
| `location` | City or locality |
| `postalCode` | Postal code when the ad page publishes it |
| `category` | Category path, e.g. `Elektronik och vitvaror > Datorer > Laptops` |
| `condition` | Stated condition, e.g. `Bra skick - varsamt använd` |
| `publishedDate` | Publication time (ISO 8601) |
| `imageUrl` | Main image URL |
| `imageUrls` | All image URLs when more than one is published |
| `adUrl` | Direct link to the ad |
| `seller` | Seller or shop name when Blocket publishes it |
| `sellerType` | `Privat` or `Företag` |
| `sellerCity`, `sellerZip` | Shop address locality and postal code |
| `sellerPhone`, `sellerProfileUrl` | Public shop phone number and profile link |
| `brand` | Brand when the listing states one |
| `tradeType` | `Säljes`, `Bortskänkes` or `Köpes` |
| `latitude`, `longitude` | Ad coordinates when published |
| `shippingAvailable`, `buyNow` | Blocket shipping and Buy Now flags |
| `description` | Full ad text (details mode or `includeDetails`) |
| `specifications` | All ad attributes published on the page, as label/value pairs |
| `mode` | Mode that produced the record |
| `scrapedAt` | UTC timestamp of the scrape |

Car listings add:

| Field | Description |
|---|---|
| `make`, `model`, `modelSpecification`, `series` | Vehicle identity |
| `year` | Model year |
| `mileage`, `mileageUnit`, `mileageKm` | Published mileage, its unit, and the same value in kilometres |
| `fuel`, `transmission` | Powertrain and gearbox |
| `registrationNumber`, `vin` | Registration plate and chassis number |
| `registrationClass`, `salesForm` | Registration class and sales form |
| `bodyType`, `colour`, `power`, `engineVolume`, `co2Emissions` | Specs read from the ad page |
| `drivetrain`, `weight`, `seats`, `doors`, `maxTrailerWeight`, `registrationDate`, `interiorColour` | Further specs read from the ad page |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchAll` | `searchAll`, `searchCars`, `searchElectronics`, `searchFurniture`, or `details`. |
| `query` | string | `laptop` | Keyword to search for. Ignored in details mode. |
| `urls` | array | – | Ad URLs or bare ad IDs for details mode. |
| `priceMin` | integer | – | Minimum price in SEK. |
| `priceMax` | integer | – | Maximum price in SEK. |
| `condition` | string | `any` | `any`, `new`, `likeNew`, `good`, `ok`, `broken` (not for cars). |
| `sellerType` | string | `any` | `any`, `private`, `company`. |
| `tradeType` | string | `any` | `any`, `sell`, `giveAway`, `wanted` (not for cars). |
| `region` | string | `any` | `any` or a Swedish county key such as `stockholm`, `skane`, `vastraGotaland`. |
| `sort` | string | `relevance` | `relevance`, `priceAsc`, `priceDesc`, `newest`, `oldest`. |
| `publishedToday` | boolean | `false` | Only ads published today. |
| `yearMin` | integer | – | Cars: minimum model year. |
| `yearMax` | integer | – | Cars: maximum model year. |
| `mileageMax` | integer | – | Cars: maximum mileage in Swedish mil (1 mil = 10 km). |
| `fuel` | string | `any` | Cars: `petrol`, `diesel`, `electric`, `ethanol`, `gas`, `hybridPetrol`, `hybridDiesel`, `hybridGas`, `plugInPetrol`, `plugInDiesel`. |
| `transmission` | string | `any` | Cars: `automatic`, `manual`, `sequential`. |
| `bodyType` | string | `any` | Cars: `convertible`, `coupe`, `familyVan`, `hatchback3`, `hatchback5`, `estate`, `pickup`, `sedan`, `van`, `suv`, `other`. |
| `includeDetails` | boolean | `false` | Open every search result to add description, condition, category and seller details. Slower. |
| `maxItems` | integer | `20` | Maximum records to return, 1–1000. |

### Example: keyword search across all categories

```json
{
  "mode": "searchAll",
  "query": "iPhone",
  "priceMax": 5000,
  "sort": "newest",
  "maxItems": 50
}
```

### Example: cars within a price and model-year range

```json
{
  "mode": "searchCars",
  "query": "Volvo",
  "priceMin": 50000,
  "priceMax": 150000,
  "yearMin": 2018,
  "yearMax": 2024,
  "transmission": "automatic",
  "maxItems": 100
}
```

### Example: furniture in Stockholm

```json
{
  "mode": "searchFurniture",
  "query": "IKEA soffa",
  "region": "stockholm",
  "condition": "good",
  "maxItems": 40
}
```

### Example: fetch specific ads by URL or ID

```json
{
  "mode": "details",
  "urls": [
    "https://www.blocket.se/recommerce/forsale/item/28292092",
    "https://www.blocket.se/mobility/item/28293706"
  ]
}
```

## Use cases

- **Price monitoring** — track asking prices for a product or car model over time.
- **Resale sourcing** — find underpriced items in one category or county.
- **Market research** — see what a category actually lists for across Sweden.
- **Inventory alerts** — schedule a narrow search and push new ads into your own system.
- **Ad enrichment** — paste known Blocket URLs and collect the full description and specs.

## Data source and reliability

The actor reads Blocket's public search responses and public ad pages directly. Requests are HTTP-only, use a normal browser identity, retry on rate limits and server errors, and never require user credentials. No browser is launched, so runs stay fast and memory-light.

Blocket paginates searches to a maximum number of pages; when that ceiling is reached the actor stops and reports how many records it returned. For very large result sets, split the query by county, price band, or a narrower keyword.

## FAQ

**Do I need a Blocket account or cookies?**
No. The actor uses Blocket's public listing and ad pages, which are readable without an account.

**What is the difference between the search modes?**
`searchAll` covers every category. `searchElectronics` and `searchFurniture` apply Blocket's own category filter, and `searchCars` uses Blocket's car vertical, which exposes car-specific filters and fields.

**How do I get descriptions and specifications for many ads?**
Set `includeDetails` to `true`. Each record then gets one extra ad-page request, so the run takes longer and returns richer records.

**Why are some seller fields missing?**
Blocket publishes shop names, phones and addresses for company sellers. Private sellers are shown only as `Privat`; their names are not public without logging in, so the actor leaves those fields out.

**Why is `condition` missing on some records?**
Only ads that state a condition have one. With `includeDetails` the value is read from the ad page; otherwise Blocket's search response does not contain it.

**Can I search by city?**
Yes. Add the place name to `query` (for example `laptop Stockholm`), or use the `region` filter for a whole county.

**How many results can one run return?**
Up to `maxItems` (maximum 1000), bounded by Blocket's own pagination limit for that search.

**Are the image URLs stable?**
They point at Blocket's public CDN and open without cookies or a referrer. The `imageUrls` field keeps every image when the ad publishes more than one.

**Is this affiliated with Blocket?**
No. This actor reads Blocket's public listings and is not affiliated with, endorsed by, or connected to Blocket or Schibsted.
