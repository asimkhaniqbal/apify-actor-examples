# ChavesNaMao City Property Listings Extractor

ChavesNaMão is a long-running Brazilian classifieds and real-estate portal covering residential and commercial properties for sale and rent. This actor searches its listings for one of 15 major Brazilian cities at a time, returning each property with price, size, rooms, location and taxes as published on the listing page.

## What this actor does

- Searches ChavesNaMão for properties for sale (venda) or for rent (aluguel) in a chosen city.
- Covers 15 major Brazilian cities, from São Paulo and Rio de Janeiro to Florianópolis and Vitória.
- Extracts price, condominium fee and IPTU (property tax) figures as displayed on the listing.
- Captures bedrooms, bathrooms, suites, garages, area, neighborhood, street and GPS coordinates when published.
- Fields that a given listing doesn't include (e.g. no GPS point, no pet-friendly flag) are left out of that record.

## Output per listing

| Field | Description |
| --- | --- |
| `propertyId` | ChavesNaMão's internal listing ID. |
| `title` | Listing title. |
| `transactionType` | Sale or rent transaction type as reported by the listing. |
| `price` | Numeric price in BRL. |
| `priceDisplay` | Original formatted price text. |
| `condominiumFee` | Monthly condominium fee, when published. |
| `iptuValue` | Annual property tax (IPTU) value, when published. |
| `propertyType` | Property category (house, apartment, commercial, etc.). |
| `bedrooms` | Number of bedrooms. |
| `bathrooms` | Number of bathrooms. |
| `suites` | Number of en-suite bedrooms. |
| `garages` | Number of garage/parking spaces. |
| `areaSqM` | Property area in square metres. |
| `neighborhood` | Neighborhood name. |
| `latitude` / `longitude` | GPS coordinates of the neighborhood, when published. |
| `city` | City name. |
| `stateAcronym` | Two-letter Brazilian state code (e.g. SP, RJ). |
| `street` | Street name, when published. |
| `description` | Listing description text (truncated to 2000 characters). |
| `reference` | Advertiser's internal reference code for the listing. |
| `petFriendly` | Whether the listing is marked pet-friendly. |
| `propertyUrl` | Direct link to the listing on chavesnamao.com.br. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | String (select) | `sell` | Transaction type: `sell` (Venda) or `rent` (Aluguel). |
| `city` | String (select) | `sao-paulo` | One of 15 supported Brazilian cities. |
| `maxItems` | Integer | 20 | Maximum number of listings to return (1–500), capped by how many listings ChavesNaMão's first results page for that city and transaction type actually shows. |

### Example: apartments for rent in Belo Horizonte

```json
{
  "mode": "rent",
  "city": "belo-horizonte",
  "maxItems": 50
}
```

### Example: houses for sale in Curitiba

```json
{
  "mode": "sell",
  "city": "curitiba",
  "maxItems": 100
}
```

## Use cases

- **Real estate agents** in a supported city tracking current asking prices and condominium fees.
- **Relocation and expat services** comparing rent levels and neighborhood options across Brazilian cities.
- **Market researchers** building city-level price-per-square-metre comparisons.
- **Property investors** screening listings by room count and area within one city at a time.
- **Tax and cost analysts** reviewing IPTU and condominium fee figures alongside asking price.

## FAQ

**Does this need a ChavesNaMão account or login?**
No. It reads the same listing pages shown to any visitor on chavesnamao.com.br.

**Can I search multiple cities in one run?**
No, each run searches one `city` value from the supported list. Run the actor again with a different `city` to cover another one.

**Why are `condominiumFee` and `iptuValue` text instead of numbers?**
ChavesNaMão publishes these as formatted strings (e.g. "R$ 450"), so they're kept as the original display text rather than converted to a number.

**Why might `latitude`/`longitude` be missing from a listing?**
Coordinates come from the listing's neighborhood data; when ChavesNaMão doesn't publish a geoposition for that neighborhood, the fields are left out.

**What's the difference between `bedrooms` and `suites`?**
`bedrooms` is the total bedroom count; `suites` counts only the bedrooms that have an attached (en-suite) bathroom.

**Can I get more listings by raising `maxItems`?**
Only up to the number of listings shown on that city and transaction type's first results page — the actor doesn't page through further results, so raising `maxItems` beyond that count won't return more.

**Is this affiliated with ChavesNaMão?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by ChavesNaMão.
