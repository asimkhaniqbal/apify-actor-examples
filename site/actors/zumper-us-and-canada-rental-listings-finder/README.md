# Zumper US & Canada Rental Listings Finder

Zumper lists apartments, houses, condos, and rental buildings across the US and Canada, with rent ranges that often span several available units per building. This actor searches Zumper by city and state or province, or fetches specific listing pages directly, returning rent, bed/bath ranges, address, amenities, and contact details.

## What this actor does

- Searches rentals by city and a US state or Canadian province code, or by city name alone when you don't know the exact state.
- Filters search results to a specific bedroom count, from studio up to five bedrooms.
- Fetches full detail — including square footage — for specific Zumper listing URLs.
- Returns geographic coordinates for listings that publish them, so results can be placed directly on a map.
- Falls back to that city's market-stats summary (average rent, rent trend) when the search modes find no individual listings for it.
- Leaves a field out of a record whenever Zumper doesn't publish that value for the listing, rather than sending it empty.

## Output per rental listing

| Field | Description |
|---|---|
| `listingId` | Zumper's listing ID |
| `name` | Building or listing name |
| `address` | Street address |
| `city`, `state`, `zipCode` | Location fields |
| `neighborhood`, `neighborhoodUrl` | Neighborhood name and its Zumper page |
| `minPrice`, `maxPrice` | Monthly rent range; equal values mean one fixed rent |
| `minBedrooms`, `maxBedrooms` | Bedroom range across available units |
| `minBathrooms`, `maxBathrooms` | Bathroom range |
| `squareFeet` | Unit size, from listing-detail lookups |
| `propertyType` | e.g. apartment, condo, house |
| `leaseType` | Long-term, short-term, or similar |
| `petsAllowed` | Pet policy as advertised |
| `amenities` | In-unit amenities |
| `buildingAmenities` | Shared building amenities |
| `rating` | Zumper rating, where one exists |
| `floorplanCount` | Number of available unit floorplans |
| `dateAvailable` | Move-in availability date |
| `description` | Listing description text |
| `phone`, `agentName`, `brokerageName` | Listing contact details, when published |
| `latitude`, `longitude` | Coordinates, when published |
| `imageUrl` | Primary listing photo |
| `listingUrl` | Link to the listing on Zumper |
| `searchCity`, `searchState` | The city and state used for this search (search modes only) |
| `scrapedAt` | UTC timestamp of the scrape |

## Output when no individual listings are found

If `searchRentals` or `searchByCity` can't find any individual listings for a city, the actor instead returns that city's rent-research market summary, when Zumper publishes one:

| Field | Description |
|---|---|
| `recordType` | Always `market_stats` for this fallback record |
| `city`, `state` | The searched city and state |
| `marketUrl` | Zumper's rent-research page for the city |
| `averageRent`, `averageRentText` | Average rent amount and the sentence it was parsed from |
| `rentTrend`, `rentTrendText` | Recent rent trend and the sentence it was parsed from |
| `vsNationalAverageText` | How the city compares to the national average |
| `affordableNeighborhoodsText` | Mention of cheaper neighborhoods in the city |
| `recommendedSalary`, `salaryText` | Suggested income to afford the average rent |
| `scrapedAt` | UTC timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchRentals` | `searchRentals` (city + state), `searchByCity` (city name, state optional), or `listingDetails` (specific URLs). |
| `city` | string | `San Francisco` | City name, used in `searchRentals` and `searchByCity` modes. |
| `state` | string | `CA` | US state or Canadian province code, used in `searchRentals` mode. |
| `bedrooms` | integer | – | Bedroom count filter, `0` for studio, up to `5`. |
| `listingUrls` | array | `[]` | Zumper listing URLs to fetch, used in `listingDetails` mode. |
| `maxItems` | integer | `20` | Maximum number of listings to return. |

### Example: rentals in a specific city

```json
{
  "mode": "searchRentals",
  "city": "San Francisco",
  "state": "CA",
  "maxItems": 200
}
```

### Example: two-bedroom units

```json
{
  "mode": "searchRentals",
  "city": "Austin",
  "state": "TX",
  "bedrooms": 2,
  "maxItems": 300
}
```

### Example: Canadian city, state resolved automatically

```json
{
  "mode": "searchByCity",
  "city": "Toronto",
  "maxItems": 150
}
```

### Example: specific listing details

```json
{
  "mode": "listingDetails",
  "listingUrls": [
    "https://www.zumper.com/apartments-for-rent/listing/12345678"
  ]
}
```

## Use cases

- **Rental market analysis** — track asking rents by city, neighborhood, and bedroom count.
- **Relocation tools** — surface available units within a budget near a given area.
- **Property investment research** — benchmark comparable rents before acquiring a building.
- **Map-based proptech products** — feed listings with coordinates into a map search interface.
- **Brokerage intelligence** — see which agencies and brokerages list most heavily in a market.

## FAQ

**Is this affiliated with Zumper?**
No. This actor reads Zumper's public listing data and is not affiliated with, endorsed by, or connected to Zumper.

**Do I need an account to use this?**
No. Only publicly visible listing data is collected.

**Why are prices and bedroom/bathroom counts given as ranges?**
Many Zumper entries represent an entire building with several available units. `minPrice`/`maxPrice` and the bedroom/bathroom ranges bracket the units currently listed; for a single unit, both ends of the range are the same.

**What's the difference between `searchRentals` and `searchByCity`?**
Both resolve a city to Zumper's internal location, but `searchRentals` expects an exact city plus its state/province code, while `searchByCity` is more forgiving about the city name alone — useful when you're not sure of the exact spelling or state.

**Does this cover Canadian cities?**
Yes — the state input also accepts Canadian province codes like `ON`, `BC`, and `QC`.

**Why is `squareFeet` missing from search results?**
Search results are often building-level summaries covering units of different sizes. Run `listingDetails` on a specific listing URL to get that unit's square footage.

**What's the difference between `amenities` and `buildingAmenities`?**
`amenities` are in-unit features like in-unit laundry or a dishwasher. `buildingAmenities` are shared facilities like a gym, roof deck, or parking garage.

**Why are `phone`, `agentName`, or `brokerageName` sometimes missing?**
Those fields only appear when the advertiser published them — private landlords often route inquiries through Zumper itself instead of listing direct contact details.

**How fresh is the data?**
Each run reads Zumper's live pages at request time, so results reflect current availability and pricing rather than a cached snapshot.

**What happens if there are no active listings in a city?**
The actor returns that city's rent-research market summary instead — average rent, recent trend, and how it compares nationally — identified by `recordType: "market_stats"`.
