# Fotocasa Spain Sale & Rental Listings Scraper

Scrape [Fotocasa](https://www.fotocasa.es), one of Spain's largest real-estate portals, without cookies, login, API keys or a proxy. Search any Spanish city, district, neighborhood, street or ZIP code, paste Fotocasa search URLs, or fetch individual property and new-construction project pages.

## What this actor does

- **Search by location** — resolve city, province, autonomous community, district, neighborhood, locality, street or ZIP code through Fotocasa's own suggestion service.
- **Paste search URLs** — every filter already in the URL (operation, property type, price, rooms, surface, features, sort) is preserved.
- **Paste property URLs or IDs** — fetch individual listings and new-construction project pages directly.
- **Five operations** — sale, rent, shared rent, rent-to-buy and business transfer.
- **Fifteen property categories** — homes, flats, penthouses, studios, duplex, lofts, chalets, garages, land, commercial premises, offices, storage rooms, buildings and new construction. Subtypes are preserved, not collapsed into "homes".
- **Server-side filters** — price, bedrooms, bathrooms, surface, features and verified sort orders.
- **Special filters** — published in the last 24/48 hours, last 7/30 days, rental duration, private-owner-only, bank-owned, subsidised housing and price drops.
- **New-construction projects** — project name, URL, delivery date, promoter, price/room/bath/surface ranges, images and every unit.
- **Optional detail enrichment** — energy certificate, exact street, ground surface, agency phone and official registrations, extra features and reference.
- **Optional monitor mode** — emit only new listings and price changes using persisted Actor state.
- **Up to 10,000 rows** — searches above Fotocasa's ~9,900-row API window are automatically split into price buckets and deduplicated.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `startUrls` or `propertyUrls` |
| `location` | string | – | City, province, district, neighborhood, street or ZIP code |
| `locationType` | select | `auto` | Country, autonomous community, province, municipality, district, neighborhood, locality, street or ZIP code |
| `operation` | select | `sale` | `sale`, `rent`, `share`, `rentToBuy`, `transfer` |
| `propertyType` | select | `homes` | All / homes / flats / penthouses / studios / duplex / lofts / chalets / garages / land / premises / offices / storageRooms / buildings / newConstruction |
| `minPrice`, `maxPrice` | integer | – | EUR (monthly rent for rentals) |
| `minRooms`, `maxRooms` | integer | – | Bedrooms |
| `minBathrooms`, `maxBathrooms` | integer | – | Bathrooms |
| `minSurface`, `maxSurface` | integer | – | Built surface in m² |
| `features` | multi-select | – | elevator, terrace, balcony, pool, garden, parking, storageRoom, airConditioning, heating, furnished, notFurnished, internet, petsAllowed, bankOwned, subsidised |
| `publishedSince` | select | `any` | `last24h`, `last48h`, `last7d`, `last30d` |
| `rentalType` | select | `any` | `longTerm` or `temporary` (rent only) |
| `ownerType` | select | `any` | `private` or `agency` |
| `onlyPriceDrops` | boolean | `false` | Only listings with a price reduction |
| `sortBy` | select | `relevance` | relevance, newest, oldest, priceAsc, priceDesc, surfaceAsc, surfaceDesc |
| `maxItems` | integer | `100` | 1–10,000 |
| `startUrls` | array of `{ "url": ... }` | – | Fotocasa search URLs (or paste URLs in the UI) |
| `propertyUrls` | array | – | Detail URLs, project URLs or numeric IDs |
| `fetchDetails` | boolean | `false` | Enrich with detail-page fields |
| `maxDetails` | integer | `100` | 0 = all emitted listings |
| `monitorMode` | boolean | `false` | Emit new/price-changed listings only |
| `monitorKey` | string | – | Optional state namespace |

## Examples

### Sale listings in Madrid

```json
{
  "mode": "search",
  "location": "madrid",
  "operation": "sale",
  "propertyType": "flats",
  "minRooms": 2,
  "maxPrice": 450000,
  "sortBy": "priceAsc",
  "maxItems": 200
}
```

### Rentals under €1,500 with detail enrichment

```json
{
  "mode": "search",
  "location": "barcelona",
  "operation": "rent",
  "maxPrice": 1500,
  "rentalType": "longTerm",
  "fetchDetails": true,
  "maxDetails": 50,
  "maxItems": 50
}
```

### Paste a Fotocasa search URL

```json
{
  "mode": "startUrls",
  "startUrls": [
    {"url": "https://www.fotocasa.es/es/comprar/viviendas/madrid/todas-las-zonas/l"}
  ],
  "maxItems": 100
}
```

### Individual property or project URLs

```json
{
  "mode": "propertyUrls",
  "propertyUrls": [
    "https://www.fotocasa.es/es/comprar/vivienda/madrid/x/190909371/d",
    "https://www.fotocasa.es/es/comprar/obra-nueva/barcelona-capital/20517030"
  ]
}
```

### Monitor new listings

```json
{
  "mode": "search",
  "location": "valencia",
  "operation": "rent",
  "monitorMode": true,
  "monitorKey": "valencia-rentals",
  "maxItems": 100
}
```

## Output

Keep all legacy fields (`recordType`, `listingId`, `title`, `price`, `currency`, `area`, `bedrooms`, `bathrooms`, `propertyType`, `city`, `neighborhood`, `province`, `postalCode`, `address`, `imageUrl`, `propertyUrl`, `description`, `isNewDevelopment`, `mode`, `scrapedAt`) and add:

- **Identity** — `recordType` (`property` or `newConstructionProject`), `propertyId`, `realEstateAdId`, `detailUrl`, `operation`, `transactionTypeId`, `propertySubtype`, `propertyTypeId`, `subtypeId`.
- **Price** — `priceOnRequest`, `priceDropAmount`, `previousPrice`, `priceDropPercent`, `pricePerSquareMeter`, `pricePeriod`.
- **Property** — `surface`, `plotArea`, `floor`, `floorType`, `orientation`, `orientationName`, `antiquity`, `antiquityName`, `condition`, `hotWater`, `heating`, `features`, `featureIds`, `amenities`, `hasLift`, `hasTerrace`, `hasBalcony`, `hasPool`, `hasGarden`, `hasParking`, `hasStorageRoom`, `hasAirConditioning`, `hasHeating`, `isFurnished`, `hasInternet`, `hasPetsAllowed`.
- **Media** — `images`, `photoCount`, `videos`, `hasVideo`, `hasVirtualTour`, `hasFloorPlans`, `hasOnlineGuidedTour`, `media`.
- **Location** — `street`, `streetNumber`, `municipality`, `district`, `region`, `country`, `locationText`, `latitude`, `longitude`, `coordinatesAccuracy`, `locationIds`, `addressVisibilityMode`.
- **Agency** — `agencyName`, `agencyAlias`, `agencyId`, `agencyType`, `agencyUrl`, `agencyLogo`, `agencyPhone`, `agencyReference`, `agencyRegistrations`, `publisherId`, `isPrivateOwner`, `isBank`, `phone`, `hasPhoneVisibility`.
- **Dates and flags** — `publishedAt`, `updatedAt`, `bumpedAt`, `isNewConstruction`, `isPromotion`, `isOpportunity`, `isAuctioned`, `isOccupied`, `isRentedWithTenants`, `isBareOwnership`, `isTemporaryRental`, `isTop`, `isPremium`, `hasSubsidies`, `isPhotoReport`.
- **Detail enrichment** — `energyRating`, `energyConsumption`, `energyEmissions`, `energyCertificateStatus`, `groundSurface`, `extraFeatures`, `reference`, `keyDeliveryDate`, `constructionType`, `qualityRate`, `rentReferenceIndex`.
- **Projects** — `projectId`, `projectName`, `projectUrl`, `projectKeyDeliveryDate`, `projectPromoterName`, `projectPromoterPhone`, `projectUnitCount`, `projectPriceMin`, `projectPriceMax`, `projectRoomsMin`, `projectRoomsMax`, `projectBathroomsMin`, `projectBathroomsMax`, `projectSurfaceMin`, `projectSurfaceMax`, `projectImages`, `projectUnits`, `promotionTypologies`.
- **Monitor** — `changeType`, `previousPrice`, `firstSeenAt`, `lastSeenAt`.

Empty fields are omitted at every level. Price `999999999` is treated as "price on request", never as a real price.

## Use cases

- **Buyer research** — compare asking prices and surfaces across neighborhoods.
- **Rental tracking** — watch long-term or temporary rental supply and rents.
- **Investor screening** — filter by price, surface, rooms and features, then enrich energy and agency data.
- **Lead generation** — agency name, phone, reference and registrations, private-vs-agency flags.
- **New-build monitoring** — project names, promoters, delivery dates and unit inventories.
- **Market change alerts** — schedule monitor mode to get only new and re-priced listings.

## FAQ

**Do I need a Fotocasa account, cookie, API key or proxy?** No. The actor uses Fotocasa's public search and SSR JSON endpoints directly, with no login, cookie, key or proxy input.

**Which locations can I use?** Any place Fotocasa recognizes: Spain, autonomous communities, provinces, municipalities, districts, neighborhoods, localities, streets and ZIP codes. Try `Madrid`, `Barcelona`, `Sitges`, `Eixample`, `28001` or a street name.

**Can I paste a normal Fotocasa search URL?** Yes. Use `startUrls` mode and paste the URL; the operation, property type, location and filters are read from the path and query string.

**Why did I get fewer than `maxItems`?** Three reasons: the search genuinely has fewer matches, Fotocasa's public API window is about 9,900 rows per query (the actor partitions above that only when `maxItems` is above 9,900), or a client-side filter such as price drops or private owners scanned its safety limit. The run status message always reports delivered vs reported counts.

**How does detail enrichment work?** With `fetchDetails: true` the actor fetches the listing detail page to add fields the search API does not carry: energy certificate, exact street, ground surface, agency phone/registrations and extra features. It is off by default because it adds one request per listing. `maxDetails: 0` enriches every emitted listing.

**What happens with more than 10,000 results?** `maxItems` is a hard cap at 10,000. For large searches the actor splits the query into the price buckets shown in the status message and deduplicates by property ID. To go deeper than the API window on a smaller request, add price or location filters yourself.

**Does monitor mode need extra setup?** No. It stores a compact map of property IDs and prices in Actor state, keyed by `monitorKey` (or a hash of your filters). The first run emits everything as `new`; later runs emit only `new` and `priceChanged` records.

**Are rental duration, published date and private-owner filters server-side?** Fotocasa's JSON search body ignores them. The actor applies them through Fotocasa's own path filters or client-side cutoffs, and says so in the run status when a scan limit is reached.

**Why is `price` missing on some records?** Fotocasa sometimes publishes "price on request" as `999999999` or a null. The actor omits `price` and sets `priceOnRequest: true` instead of inventing a value.

**Why do private-owner searches return fewer listings?** Fotocasa's `/particulares` page shows 15 private-owner listings per page and ignores extra query filters. The actor paginates that page, applies price/room/surface and price-drop filters client-side, and reports when the private page is exhausted; use a narrower location or price range to see different private listings. Rental-duration filtering is unavailable on the private page.

**Is this affiliated with Fotocasa?** No. It is an independent actor that reads publicly available Fotocasa data and is not affiliated with or endorsed by Fotocasa.
