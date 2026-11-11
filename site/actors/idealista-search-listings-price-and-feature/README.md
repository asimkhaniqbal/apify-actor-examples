# Idealista Search Listings Price & Feature Scraper

Scrape **Idealista** property listings across Spain, Italy and Portugal. Search a city or area, paste your own Idealista search URLs, or look up individual properties, and get structured records with price, price-drop signals, size, rooms, bathrooms, coordinates, features, energy certificate, images, description and agency contact.

## What this actor does

- **Three modes:** search by location, paste search-result URLs, or fetch individual property URLs/IDs
- **Three markets:** idealista.com (Spain), idealista.pt (Portugal), idealista.it (Italy)
- **Rich listing data:** price and €/m², previous price and drop %, size, rooms, bathrooms, floor, condition, year built, energy rating, coordinates, gallery and full description
- **Real filters:** operation, property type and subtype, price, size, rooms, bathrooms, condition, lift, terrace, pool, garden, garage, storage, air conditioning, balcony, new development, luxury, sea views, accessible, bank-owned, auction, private owners and price drops. Idealista applies filters as URL path segments; for two or more filters, use `startUrls` mode with a filtered Idealista URL that loads in your browser.
- **Large searches:** walks every result page and, for unfiltered searches where you ask for more than Idealista's 1,800-result cap, splits the search by price buckets and removes duplicates
- **No login, no cookies, no proxy setup** — the actor chooses a working connection automatically

## Output per listing

- `propertyCode` — Idealista property ID
- `url`, `sourceUrl` — canonical listing URL and the search URL that produced it
- `recordType` — `listing`
- `price`, `priceByArea`, `currency` — current price, €/m² and currency
- `previousPrice`, `priceDropAmount`, `priceDropPercent`, `hasPriceDrop`, `priceDrop` — price-cut fields when the listing shows a reduction
- `size`, `constructedArea`, `usableArea`, `plotArea` — areas in m²
- `rooms`, `bathrooms`
- `floor`, `exterior`, `condition`, `yearBuilt`
- `energyRating`, `energyConsumption`, `energyEmissions`, `energyCertification`
- `communityFees` — monthly building fees when shown
- `description` — full listing description when details are fetched
- `address`, `street`, `neighborhood`, `district`, `municipality`, `province`, `country`
- `latitude`, `longitude`, `addressVisibility`, `showAddress`
- `images`, `numPhotos`, `multimedia`
- `hasVideo`, `has3DTour`, `has360`, `hasStaging`, `hasPlan`
- `hasLift`, `hasSwimmingPool`, `hasTerrace`, `hasAirConditioning`, `hasBoxRoom`, `hasGarden`, `hasParkingSpace`
- `features` — advertised feature flags from the card or property page; `featureSource` says which
- `contactInfo` — agency/owner name, logo, phone, reference, profile URL, user type and response time
- `agency`, `phone`, `reference`
- `propertyType`, `propertySubtype`, `operation`, `status`
- `newDevelopment`, `externalReference`, `topPlus`, `isPriceDrop`, `isOffmarket`, `isFeatured`, `isExclusive`, `isBankOwned`, `isAuction`, `isTenanted`, `isOccupied`, `isNudaPropiedad`
- `updatedAtText`, `updatedAt`, `scrapedAt`

Empty fields are omitted. A feature flag is present only when the source confirms it; it is never emitted as a guessed `false`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | – | `search` by location, `startUrls` for your own search URLs, `propertyUrls` for property URLs/IDs; inferred when omitted |
| `location` | string | `madrid-madrid` | City/area slug or a full Idealista search URL (mode=search) |
| `operation` | string | `sale` | `sale` or `rent` |
| `propertyType` | string | `homes` | homes, newDevelopments, offices, premises, garages, lands, storageRooms, buildings, bedrooms |
| `country` | string | `es` | `es`, `pt`, `it`; auto-detected from URLs |
| `startUrls` | array | – | Idealista search URLs, filters and map-area URLs kept as-is (mode=startUrls) |
| `propertyUrls` | array | – | Property URLs or numeric IDs (mode=propertyUrls) |
| `propertySubtype` | string | `any` | any, flat, penthouse, duplex, studio, loft, chalet, independentHouse, semiDetached, terraced, villa, countryHouse, office, premises, garage, land, storageRoom, building, room |
| `priceMin` | integer | – | Minimum price in EUR |
| `priceMax` | integer | – | Maximum price in EUR |
| `sizeMin` | integer | – | Minimum size in m² |
| `sizeMax` | integer | – | Maximum size in m² |
| `rooms` | string | `any` | any, 1, 2, 3, 4, 5+ |
| `bathrooms` | string | `any` | any, 1, 2, 3+ |
| `condition` | string | `any` | any, new, good, toRenovate |
| `hasLift` | boolean | `false` | Only listings with a lift |
| `hasTerrace` | boolean | `false` | Only listings with a terrace |
| `hasPool` | boolean | `false` | Only listings with a pool |
| `hasGarden` | boolean | `false` | Only listings with a garden |
| `hasGarage` | boolean | `false` | Only listings with a garage/parking |
| `hasStorageRoom` | boolean | `false` | Only listings with a storage room |
| `hasAirConditioning` | boolean | `false` | Only listings with air conditioning |
| `hasBalcony` | boolean | `false` | Only listings with a balcony |
| `isNewDevelopment` | boolean | `false` | Only new developments |
| `isBankOwned` | boolean | `false` | Only bank-owned listings |
| `isAuction` | boolean | `false` | Only auction listings |
| `isLuxury` | boolean | `false` | Only luxury listings |
| `hasSeaViews` | boolean | `false` | Only listings advertised with sea views |
| `isAccessible` | boolean | `false` | Only accessible homes |
| `sortBy` | string | `relevance` | relevance, priceAsc, priceDesc, areaAsc, areaDesc, pricePerSqmAsc, pricePerSqmDesc |
| `onlyPriceDrops` | boolean | `false` | Keep only reduced listings |
| `minPriceDropPercent` | integer | – | Minimum price-drop percentage |
| `onlyPrivateOwners` | boolean | `false` | Keep only private sellers |
| `fetchDetails` | boolean | `false` | Visit each listing page for bathrooms, energy, year, condition, full description, gallery and phone |
| `maxDetails` | integer | `100` | Maximum number of detail pages to fetch |
| `maxItems` | integer | `100` | Maximum listings to save (1–10,000) |
| `partitionLargeSearches` | boolean | `true` | Split searches above 1,800 results into price buckets |

### Example: Madrid homes for sale with a pool

```json
{
  "mode": "search",
  "location": "madrid-madrid",
  "operation": "sale",
  "propertyType": "homes",
  "country": "es",
  "hasPool": true,
  "priceMax": 600000,
  "fetchDetails": true,
  "maxItems": 50
}
```

### Example: Lisbon rentals by search URL

```json
{
  "mode": "startUrls",
  "startUrls": [
    { "url": "https://www.idealista.pt/arrendar-casas/lisboa/" }
  ],
  "maxItems": 100
}
```

### Example: Rome offices with details

```json
{
  "mode": "search",
  "location": "roma-roma",
  "operation": "sale",
  "propertyType": "offices",
  "country": "it",
  "fetchDetails": true,
  "maxItems": 30
}
```

### Example: look up specific properties

```json
{
  "mode": "propertyUrls",
  "propertyUrls": [
    "https://www.idealista.com/inmueble/111379612/",
    "https://www.idealista.it/immobile/37181412/"
  ]
}
```

## Use cases

- **Market research** — compare asking prices and €/m² across cities, districts and property types
- **Investment screening** — find reduced listings and compare them with the local market
- **Lead generation** — collect agency names, phone numbers and references for real-estate outreach
- **Portfolio monitoring** — schedule runs to track new listings, price cuts and removed properties
- **Comparable sales** — pull a neighbourhood's listings with size, condition, year and energy rating
- **Data enrichment** — add coordinates, features and images to an existing property database

## Limitations

- **1,800 results per search.** Idealista serves at most 60 pages of 30 listings. The actor partitions larger searches by price when you ask for more, but a single search URL cannot exceed the cap.
- **Market protection can slow a run.** Italy, Spain and Portugal may challenge some requests; the actor retries and switches to a slower working connection when needed. A run can take longer on those days.
- **Valid location slugs matter.** Portugal uses Portuguese names (`lisboa`, `porto`); Italy uses Italian names (`roma-roma`, `milano-milano`). A wrong slug returns no listings.
- **Private sellers often hide their phone.** The phone field is included only when Idealista publishes it.
- **No publication date.** Idealista does not expose a reliable published timestamp on public pages; the actor includes the “last updated” text/date when shown.
- **Detail fields require `fetchDetails`.** Bathrooms, energy rating, year built, condition, full description, gallery and phone come from the property page.
- **Agency email is not included.** Idealista publishes the agency phone/reference, not its email.

## FAQ

**Do I need an Idealista account or cookies?**
No. The actor reads public listing pages and needs no login.

**Do I need to configure a proxy?**
No. The actor chooses a working connection automatically. Just fill in the input and run.

**Which countries are supported?**
Spain (`idealista.com`), Portugal (`idealista.pt`) and Italy (`idealista.it`). Country is auto-detected when you paste a URL.

**How many listings can I get from one search?**
Idealista serves at most 60 result pages of 30 listings (1,800) per search. For unfiltered searches, when you ask for more, the actor splits the search into price buckets, removes duplicates and keeps going. Filtered searches are capped at 1,800; use `startUrls` mode with a filtered URL that loads in your browser for full control.

**Why are some fields missing?**
Search cards do not show everything. Bathrooms, energy rating, year built, condition, full description and phone come from the property page; turn on `fetchDetails` to collect them. Private sellers often hide their phone, so that field is omitted for those listings.

**Why does the first page sometimes take a minute?**
Idealista can challenge requests in any of the three markets. When the fast path is unavailable, the actor prepares a slower working connection before loading the first page. Later pages reuse that connection and are much faster.

**Are the images hotlink-safe?**
Yes. Image URLs are public CDN links and open without cookies or a referrer.

**Why is there no published date?**
Idealista does not expose a reliable publication timestamp on its public pages. The actor includes the “last updated” text and date when the page shows it, but does not invent a publication date.

**Can I scrape multiple cities in one run?**
Yes. Use `startUrls` with one search URL per city or district; they are processed in order up to `maxItems`.

**Is this affiliated with Idealista?**
No. This is an independent, third-party actor that reads publicly available Idealista pages. It is not affiliated with or endorsed by Idealista.
