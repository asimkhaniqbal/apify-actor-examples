# Immowelt Germany Rent & Buy Listings Scraper

Scrape property listings from [Immowelt.de](https://www.immowelt.de), one of Germany's largest real-estate portals. Search rentals, for-sale homes, plots, commercial space and parking by city, paste a prepared Immowelt search link, or look up specific listings by ID. The actor pages through the full result set, applies Immowelt's own filters and sort orders, and can add a detailed exposé record with construction year, heating, energy values, coordinates, amenities, price breakdown and documents.

## What this actor does

- **Three modes:** `search` by German city and filters, `byUrl` for a pasted Immowelt search link, and `byIds` for classified IDs or exposé URLs.
- **Full result-set pagination:** the actor walks every results page instead of stopping at the first 30-40 cards, with a hard `maxItems` ceiling of 10,000.
- **14 property categories:** apartment, house, plot, temporary/furnished living, office, retail, storage/production, gastronomy/hotel, agricultural/forestry land, parking, whole building, senior living, new-build projects and miscellaneous.
- **Deep server-side filters:** price, rooms, living space, plot size, construction year, bedrooms, features, energy class, building condition, project type, seller type, WBS requirement and floor location, plus all nine Immowelt sort orders.
- **Optional exposé enrichment:** construction year, condition, heating type, energy source/consumption/certificate, coordinates, amenities by category, deposit and running costs, videos, virtual tours, floor plans, PDF documents and the full multi-section description.
- **Keywords and clean output:** include/exclude keyword filters run on the title and description; empty fields are omitted at every depth, never returned as `null`, `""` or `N/A`.

## Output per listing

- `classifiedId`, `legacyId` - Immowelt listing identifiers
- `publishedAt`, `updatedAt`, `listingStatus`, `isNewBuildProject`
- `title`, `headline`, `description` (full text when detailed, otherwise the search-card summary)
- `distributionType` (`mieten`/`kaufen`), `distributionSubType`, `estateType`, `estateSubType` (human-readable label), `isTemporaryLiving`
- `priceCold` (net/base rent), `priceWarm` (rent including utilities), `purchasePrice`, `currency`
- `pricePerSqm` (computed from the listing's own price and area), `quotedPricePerSqm` (Immowelt's quoted rate, when published)
- `priceAdditionalInformation`, `priceNote`, `deposit`, `utilitiesCost`, `heatingCost`, `financingUrl`
- `marketAvgPricePerSqm`, `marketPriceRangeLowPerSqm`, `marketPriceRangeHighPerSqm`, `marketPriceTrend1YearPct`, `marketPriceDate`
- `rooms`, `bedrooms`, `livingSpaceSqm`, `plotSqm`, `floor`, `availableFrom`, `yearBuilt`, `condition`
- `energyClass`, `energyCertificateClass`, `energyCertificateType`, `energyCertificateValidity`, `energyCertificateUrl`, `energyConsumption`, `heatingType`, `energySource`
- `buildingState`, `isNewListing`, `has3DVisit`, `hasBrokerageFee`, `isForeclosureAuction`
- `hasFloorPlan`, `hasVirtualTour`, `hasVideo`
- `street`, `district`, `borough`, `city`, `zipCode`, `state`, `country`, `latitude`, `longitude`, `isAddressPublished`
- `providerName`, `providerType`, `providerContactPerson`, `providerWebsite`, `providerPhone`, `providerAddress`, `providerProfileUrl`, `providerImprintUrl`, `providerLogoUrl`
- `providerBadge`, `providerBadgeImageUrl`, `providerBadgeDescription`, `providerIsPrivateOwner`, `providerBadges[]`
- `providerRating`, `providerReviewCount`, `providerRatingUrl`, `providerFeeScheduleUrl`, `projectUrl`
- `images[]` (`url`, `caption`, `type` scene label), `primaryImageUrl`, `imageCount`
- `videos[]`, `virtualTours[]`, `floorPlans[]`, `documents[]`
- `amenities[]`, `amenitiesByCategory`, `aiFeatures`
- `locationDescription`, `additionalDescription`, `extendedInfoDescription`
- `detailsEnriched` (present only when an exposé page was merged), `sourceUrl`
- `recordType: "listing"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (city and filters), `byUrl` (pasted search link) or `byIds` (listing IDs / exposé URLs) |
| `city` | string | `Berlin` | German city or town for `search`, e.g. `München`, `Hamburg`, `Köln`, `Frankfurt am Main` |
| `distributionType` | string | `mieten` | `mieten` (rent), `kaufen` (buy) or `zwangsversteigerung` (compulsory auction) |
| `estateType` | string | `wohnung` | One of the 14 Immowelt property categories |
| `estateSubType` | string | - | Optional narrower sub-type; leave empty for any. Must belong to the selected property type |
| `minPrice` / `maxPrice` | integer | - | Price range in EUR (cold rent or purchase price) |
| `minRooms` / `maxRooms` | number | - | Room-count range |
| `minArea` / `maxArea` | integer | - | Living-space range in m² |
| `minPlotArea` / `maxPlotArea` | integer | - | Plot/land-size range in m² |
| `minYearBuilt` / `maxYearBuilt` | integer | - | Construction-year range |
| `minBedrooms` / `maxBedrooms` | integer | - | Bedroom-count range |
| `features` | array | `[]` | Required amenities; all selected values must be present |
| `energyClass` | array | `[]` | One or more energy classes from `A+` (best) to `H` |
| `buildState` | array | `[]` | One or more building conditions, e.g. first occupancy, mint, well kept, needs renovation |
| `projectTypes` | array | `[]` | One or more project/tenure types, e.g. new build, resale, investment, short-term rental |
| `sellerType` | string | `any` | `any`, `private` or `professional` |
| `wbsRequired` | boolean | `false` | Only listings that require a certificate of eligibility (WBS) |
| `floorLocation` | array | `[]` | Ground floor, half basement or roof storey |
| `excludeGroundFloor` | boolean | `false` | Skip ground-floor listings |
| `includeKeywords` | array | `[]` | Keep only listings whose title/description contains every keyword |
| `excludeKeywords` | array | `[]` | Drop listings whose title/description contains any keyword |
| `sortBy` | string | – | All nine Immowelt orders: newest, price, area, rooms and price per m², ascending or descending |
| `maxItems` | integer | `100` | Hard cap on emitted listings (1-10,000) |
| `searchUrl` | string | - | Full Immowelt search URL for `byUrl` (modern `/classified-search` or `/suche/...`) |
| `propertyIds` | array | `[]` | Classified IDs or exposé URLs for `byIds` |
| `includeDetails` | boolean | `false` | Fetch each listing's exposé page for the extra detail fields |
| `maxDetails` | integer | `0` | Number of listings to enrich; `0` means all emitted listings |

### Example: apartments for rent in Berlin, 500-2,000 EUR

```json
{
  "mode": "search",
  "city": "Berlin",
  "distributionType": "mieten",
  "estateType": "wohnung",
  "minPrice": 500,
  "maxPrice": 2000,
  "minRooms": 2,
  "maxItems": 500,
  "sortBy": "priceAsc"
}
```

### Example: houses for sale in München with a garden

```json
{
  "mode": "search",
  "city": "München",
  "distributionType": "kaufen",
  "estateType": "haus",
  "features": ["garden", "parking"],
  "maxItems": 300
}
```

### Example: paste an Immowelt search link

```json
{
  "mode": "byUrl",
  "searchUrl": "https://www.immowelt.de/suche/mieten/wohnung/berlin/berlin-10115/ad08de8634",
  "maxItems": 1000
}
```

### Example: enrich specific listings by ID or exposé URL

```json
{
  "mode": "byIds",
  "propertyIds": [
    "26XYAMX8494K",
    "https://www.immowelt.de/expose/81052d86-9572-4815-a7e8-bea49be808b3"
  ],
  "includeDetails": true,
  "maxItems": 10
}
```

## Use cases

- **Rental-market research** - pull complete, filtered rent inventories for one city or many scheduled runs.
- **Investment screening** - compare purchase prices, price per m² and local market benchmarks across German cities.
- **Property alerts and lead generation** - monitor new listings that match a saved filter set.
- **Agency and portfolio analysis** - analyse provider names, badges, ratings and listing volumes.
- **Real-estate data platforms** - feed structured German property records into a search, valuation or CRM product.
- **Commercial and land research** - collect office, retail, storage, gastronomy, parking and agricultural/forestry listings that consumer portals often omit.

## FAQ

### Which German cities can I search?

Any city or town Immowelt indexes, including large cities and smaller municipalities. Use the German spelling without postcode; the actor resolves it on Immowelt. A handful of small towns that share a name with a district may not have their own working search page on the site, so a nearby larger town is the practical fallback.

### How many listings can one run return?

Up to 10,000 when you set `maxItems` that high and the search has enough matches. The actor pages through the entire result set rather than taking only the first page. If a search has more matches than `maxItems`, the run reports the site's total count in its status message.

### Why are year built, heating, coordinates or documents missing on some records?

Those values come from the listing's exposé page and are only fetched when `includeDetails` is enabled. Even then, Immowelt does not publish every detail for every property - for example, energy data or construction year may be absent on older or private listings. Missing values are omitted instead of being guessed.

### What is the difference between priceCold, priceWarm and purchasePrice?

`priceCold` is the net/base rent (Kaltmiete or Nettomiete), `priceWarm` is the rent including utilities (Warmmiete), and `purchasePrice` is the asking sale price. Only the values Immowelt publishes for that listing are included.

### Are furnished, short-term, commercial and parking listings supported?

Yes. Temporary/furnished living, office, retail, storage/production, gastronomy/hotel, agricultural/forestry land, parking, whole buildings, senior living, new-build projects and miscellaneous are all selectable property categories. The `features` filter adds individual amenities such as balcony, garden, lift, parking, cellar or a swimming pool.

### Do I need a login, cookies or an API key?

No. The actor works without an account, cookies or credentials. Everything it returns is publicly visible on Immowelt.de.

### Do I need to configure a proxy?

No. The actor chooses a working connection automatically and handles the site's bot protection itself. Just fill in the input and run it.

### Why did a run stop before reaching my maxItems?

Immowelt can temporarily limit a connection when it receives too many requests. The actor paces its requests, retries, and switches to its automatic fallback paths. If the site still blocks the run, already-collected rows are kept and the status message says that the first results page was used or that the run was interrupted.

### How fresh is the data?

Every record is read live from Immowelt during the run. `publishedAt`, `updatedAt` and `scrapedAt` show the listing's publication/update times and the time of the scrape.

### Is this actor affiliated with Immowelt?

No. This is an independent third-party tool that reads publicly available listing pages.
