# ChavesNaMao City Property Listings Extractor

ChavesNaMão is one of Brazil's busiest property portals, with hundreds of thousands of sale and rental ads in a single city. This actor reads those listings without cookies or login, applies the portal's own filters, walks its result pages, and can open each ad to return the full published details.

## What it collects

- Sale (venda) and rental (aluguel) listings in 17 major Brazilian cities.
- Structured search by property type, price range, area range, minimum bedrooms, bathrooms and parking spaces, advertiser type and sort order.
- Every result page, with duplicate listing IDs removed, up to the requested limit.
- Optional detail enrichment for each listing: full description, publisher name, CRECI and phone, complete photo list, amenities, video and 360° tour links, publication and update dates.

Fields are omitted when ChavesNaMão does not publish them for a particular ad. The actor never writes null, empty-string or placeholder values.

## Input defaults

| Input | Default | Notes |
| --- | --- | --- |
| `collectionMode` | `listings` | `details` for known ads. |
| `transactionType` | `sell` | `rent` for aluguel. |
| `city` | `sao-paulo` | One of 17 built-in city slugs. |
| `propertyType` | `all` | Every type when not restricted. |
| `bedroomsMin` | 0 | No minimum. |
| `bathroomsMin` | 0 | No minimum. |
| `garagesMin` | 0 | No minimum. |
| `priceMin` | – | No minimum. |
| `priceMax` | – | No maximum. |
| `areaMin` | – | No minimum. |
| `areaMax` | – | No maximum. |
| `sortBy` | `relevance` | Site default order. |
| `ownerType` | `all` | All advertisers. |
| `maxItems` | 20 | Daily-test prefill uses 3. |
| `maxPages` | 34 | 30 rows per page. |
| `includeDetails` | false | One extra request per listing. |
| `listingUrls` | – | Any ChavesNaMão search URL. |
| `propertyUrls` | – | Details mode only. |
| `propertyIds` | – | Details mode only. |

## Collection modes

### Search listings

Choose a transaction, city and any combination of filters. The actor translates them into the same filter syntax the ChavesNaMão website uses, then follows the selected sort order across pages.

| Input | Meaning |
| --- | --- |
| `transactionType` | `sell` for venda, `rent` for aluguel. |
| `city` | One of the 17 built-in city slugs. |
| `propertyType` | All types or one of apartments, houses, gated houses, kitnets, penthouses, flats, lofts, commercial rooms/houses/points, buildings, warehouses, land, chácaras, farms or garages. |
| `bedroomsMin`, `bathroomsMin`, `garagesMin` | Minimum room counts. 0 means no minimum. |
| `priceMin`, `priceMax`, `areaMin`, `areaMax` | Numeric bounds in BRL and square metres. |
| `sortBy` | Relevance, newest, price ascending/descending, area ascending/descending. |
| `ownerType` | All advertisers, direct with owner, or agencies only. |
| `maxItems` | Total listings to save (1–1000). |
| `maxPages` | Pages to request per search, 30 listings each. |
| `includeDetails` | Add one detail request per listing for the richer fields. |
| `listingUrls` | Optional ChavesNaMão search URLs, used verbatim. Any city or neighbourhood reachable on the site can be scraped this way. |

### Property details

Set `collectionMode` to `details` and provide `propertyIds`, `propertyUrls`, or both. Each ad is fetched from ChavesNaMão's detail endpoint and returned once. This is the cheapest way to refresh a known shortlist.

## Output per listing

| Field | Description |
| --- | --- |
| `propertyId` | ChavesNaMão listing ID. |
| `title` | Listing title as published. |
| `transactionType` | `SELL` or `RENT`. |
| `price`, `priceCurrency`, `priceDisplay` | Numeric asking price, currency and the original formatted text. |
| `condominiumFee`, `condominiumFeeValue` | Monthly condominium fee text and numeric value when available. |
| `iptuValue`, `iptuValueNumeric` | Annual IPTU property-tax text and numeric value when available. |
| `propertyType`, `propertyTypeId`, `propertyTypeSlug` | Property category. |
| `bedrooms`, `bedroomsMax` | Bedrooms (minimum and maximum when the ad publishes a range). |
| `bathrooms`, `bathroomsMax` | Bathrooms. |
| `suites`, `suitesMax` | En-suite bedrooms. |
| `garages`, `garagesMax` | Garage or parking spaces. |
| `commercialRooms`, `commercialRoomsMax` | Commercial rooms when applicable. |
| `areaSqM`, `usableAreaSqM`, `totalAreaSqM` | Primary, usable and total area in square metres. |
| `street`, `addressNumber`, `neighborhood`, `neighborhoodId` | Address details. |
| `city`, `cityId`, `stateAcronym`, `state`, `zipCode` | Location details. |
| `latitude`, `longitude` | Coordinates when the ad or neighbourhood supplies them. |
| `publicAddress` | Whether the full street address is public on the ad. |
| `description` | Full description (HTML stripped, up to 5,000 characters). |
| `reference` | Advertiser's internal reference code. |
| `petFriendly`, `acceptTrade`, `active`, `highlighted` | Booleans published by the ad. |
| `publisherName`, `publisherType`, `publisherCreci`, `publisherVerified` | Advertiser identity. |
| `publisherPhone`, `publisherWhatsApp`, `publisherUrl` | Advertiser contact details and profile link. |
| `imageCount`, `featuredImage`, `imageUrls` | Photo count and full-resolution image links. |
| `videoUrl`, `tour360Url` | Video and virtual-tour links. |
| `amenities`, `buildingAmenities` | Private and common-area features from the detail page. |
| `createdAt`, `updatedAt` | Publication and last-update timestamps. |
| `propertyUrl` | Direct link to the ad. |
| `scrapedAt` | ISO timestamp of collection. |

## Examples

Three apartments for sale in São Paulo with the daily-test prefill:

```json
{
  "collectionMode": "listings",
  "transactionType": "sell",
  "city": "sao-paulo",
  "maxItems": 3,
  "maxPages": 1
}
```

Rentals in Rio de Janeiro between R$ 2,000 and R$ 4,000, at least two bedrooms, newest first, with full details:

```json
{
  "collectionMode": "listings",
  "transactionType": "rent",
  "city": "rio-de-janeiro",
  "bedroomsMin": 2,
  "priceMin": 2000,
  "priceMax": 4000,
  "sortBy": "newest",
  "maxItems": 50,
  "includeDetails": true
}
```

Houses for sale in a neighbourhood using the site's own URL:

```json
{
  "collectionMode": "listings",
  "listingUrls": ["https://www.chavesnamao.com.br/casas-a-venda/sp-sao-paulo/pinheiros/"],
  "maxItems": 100
}
```

Full details for known ads:

```json
{
  "collectionMode": "details",
  "propertyUrls": ["https://www.chavesnamao.com.br/imovel/apartamento-a-venda-2-quartos-sp-sao-paulo-pinheiros-90m2-RS750000/id-26583048/"]
}
```

## Compatibility with older tasks

Older tasks that send `mode` as `sell` or `rent`, plus `city` and `maxItems`, keep working: the actor reads those hidden legacy keys and maps them to `transactionType` and listings mode. The schema exposes `collectionMode` for new tasks. Existing output fields are unchanged; the revamp only adds fields beside them.

## FAQ

**Do I need a ChavesNaMão account or cookies?**
No. The actor reads the same public pages and JSON endpoints shown to any visitor, with no login, cookie or proxy input.

**Can I search several cities at once?**
Yes. Pass several `listingUrls` (one per city) in a single run. When no URL is supplied, one city is searched from the `city` control.

**How many listings can one search return?**
ChavesNaMão reports hundreds of thousands of ads per large city and currently exposes up to 333 pages (about 10,000 rows) per search. Set `maxItems` and `maxPages` to the depth you need. Larger total volumes are reached by splitting the query with price, area or neighbourhood filters.

**Why are some fields missing from a row?**
ChavesNaMão does not publish every detail for every ad. Empty values are omitted rather than fabricated. Turn on `includeDetails` to fill the detail-only fields for the listings that have them.

**Are condominium fee and IPTU numbers or text?**
Both are kept as the original display text (`condominiumFee`, `iptuValue`) and, when parseable, also as numbers (`condominiumFeeValue`, `iptuValueNumeric`).

**What does `publicAddress` mean?**
When true, the ad shows the street and number publicly. When false or absent, the portal hides the exact address even though the neighbourhood is known.

**Is this affiliated with ChavesNaMão?**
No. This is an independent data tool and is not affiliated with, endorsed by or sponsored by ChavesNaMão.
