# Kayak Flight, Hotel & Cheap Destination Finder

Kayak aggregates flight, hotel and destination prices from hundreds of travel sites. This actor searches Kayak directly for flights between two cities, browses cheap destinations from a chosen origin airport, looks up hotel rates for a stay, or re-scrapes specific Kayak search result URLs you already have.

## What this actor does

- Searches for flights between an origin and destination on your chosen dates, with price, airline, stops and a booking link per result.
- Browses Kayak's "explore" view for cheap destinations from a given origin airport, optionally capped to a maximum budget.
- Looks up hotel rates for a destination and date range.
- Re-scrapes a list of specific Kayak search URLs you already have, returning the same flight-style records.
- Omits any field that wasn't available for a given result, so records never contain blank placeholders.

## Output per record

### Flight records (`searchFlights`, `byUrls`)

| Field | Description |
|-------|-------------|
| `flightId` | Kayak's internal identifier for this flight result |
| `origin` / `destination` | Origin/destination airport codes |
| `originCity` / `destinationCity` | Origin/destination city names |
| `departDate` / `returnDate` | Departure and return dates |
| `price` | Best available price per person |
| `currency` | Price currency |
| `airline` | Primary airline operating the outbound leg |
| `airlines` | All airlines involved, for multi-leg itineraries |
| `stops` | Number of stops on the outbound leg (0 = nonstop) |
| `duration` | Outbound flight duration |
| `departTime` / `arrivalTime` | Outbound departure/arrival times |
| `cabinClass` | Cabin class used for the search |
| `tripType` | `one-way` or `round-trip` |
| `deepLink` | Link to book the flight on Kayak |
| `recordType` | `kayakFlight` |
| `scrapedAt` | UTC timestamp when the record was extracted |

### Cheap-destination records (`exploreDeals`)

| Field | Description |
|-------|-------------|
| `flightId` | Identifier combining the origin and destination city |
| `origin` | Origin airport code |
| `destinationCity` | Destination city name |
| `price` | Lowest price shown for that destination |
| `currency` | Price currency |
| `stops` | Number of stops for the cheapest itinerary shown |
| `departDate` / `returnDate` | Suggested travel dates for that price |
| `imageUrl` | Destination image shown on the Kayak explore page |
| `tripType` | `round-trip` |
| `cabinClass` | `economy` |
| `recordType` | `kayakExplore` |
| `scrapedAt` | UTC timestamp when the record was extracted |

### Hotel records (`searchHotels`)

| Field | Description |
|-------|-------------|
| `flightId` | Kayak's internal identifier for this hotel result |
| `origin` / `destination` | Echo of the `origin`/`destination` input values |
| `originCity` / `destinationCity` | Echo of the `origin`/`destination` input values |
| `departDate` / `returnDate` | Check-in/check-out dates |
| `price` | Rate shown for the stay |
| `currency` | Price currency |
| `airline` | Hotel name |
| `stops` | Always `0` for hotel records |
| `cabinClass` | Cabin class input value, unused for hotels |
| `tripType` | `hotel` |
| `recordType` | `kayakFlight` |
| `scrapedAt` | UTC timestamp when the record was extracted |

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mode` | String (select) | `searchFlights` | `searchFlights`, `searchHotels`, `exploreDeals`, or `byUrls`. |
| `origin` | String | – | Origin airport IATA code or city, e.g. `NYC`, `JFK`, `New York`. |
| `destination` | String | – | Destination airport IATA code or city, e.g. `LON`, `LHR`, `London`. Required for `searchFlights` and `searchHotels`; optional for `exploreDeals`. |
| `departDate` | String | – | Departure date in `YYYY-MM-DD` format. Defaults to roughly 30 days out when left blank. |
| `returnDate` | String | – | Return date in `YYYY-MM-DD` format for round-trip searches. Defaults to roughly a week after the departure date when left blank. |
| `tripType` | String (select) | `round-trip` | `one-way` or `round-trip`. |
| `cabinClass` | String (select) | `economy` | `economy`, `premium-economy`, `business`, or `first`. |
| `adults` | Integer | 1 | Number of adult passengers (1-9). |
| `maxBudget` | Integer | – | Maximum price per person in USD, used in `exploreDeals` mode. |
| `sortBy` | String (select) | `price` | How to sort flight results: `price`, `duration`, `departure`, or `arrival`. |
| `startUrls` | Array of strings | – | Kayak search result URLs to scrape, used in `byUrls` mode. |
| `maxItems` | Integer | 50 | Maximum number of results to return (1-500). |

### Example: Round-trip flight search

```json
{
  "mode": "searchFlights",
  "origin": "NYC",
  "destination": "LON",
  "departDate": "2026-08-01",
  "returnDate": "2026-08-15",
  "tripType": "round-trip",
  "cabinClass": "economy",
  "adults": 1,
  "sortBy": "price",
  "maxItems": 50
}
```

### Example: Cheap destinations from an origin, under a budget

```json
{
  "mode": "exploreDeals",
  "origin": "NYC",
  "maxBudget": 500,
  "maxItems": 30
}
```

### Example: Hotel rates for a stay

```json
{
  "mode": "searchHotels",
  "destination": "LON",
  "departDate": "2026-08-01",
  "returnDate": "2026-08-05",
  "maxItems": 20
}
```

### Example: Re-scrape saved search URLs

```json
{
  "mode": "byUrls",
  "startUrls": ["https://www.kayak.com/flights/NYC-LON/2026-08-01/2026-08-15"],
  "maxItems": 30
}
```

## Use cases

- **Fare comparison**: search a specific route and date range to compare prices across airlines before booking.
- **Travel inspiration**: use `exploreDeals` with a budget cap to discover cheap destinations from your home airport.
- **Hotel rate checks**: pull current hotel pricing for a destination and date range alongside a flight search.
- **Price tracking**: re-run the same `searchFlights` or `byUrls` input periodically to monitor how a fare changes.
- **Deal aggregation**: feed Kayak results into a broader travel-deals dataset alongside other sources.

## FAQ

**Do I need a Kayak account to use this?**
No, this actor only reads Kayak's publicly available search results.

**Why does a search occasionally return no results?**
Kayak's results load asynchronously and can take a little longer to appear for less common routes or dates; retrying the run or adjusting the dates usually helps.

**What's the difference between `origin`/`destination` and `originCity`/`destinationCity`?**
The first pair are airport or city IATA-style codes; the second pair are the resolved human-readable city names for the same locations.

**Can I search one-way flights?**
Yes, set `tripType` to `one-way`; `returnDate` isn't needed in that case.

**What does the `stops` field mean?**
`0` means a nonstop/direct itinerary; `1` means one connection, `2` means two connections, and so on.

**Can I search for multiple passengers?**
Yes, set `adults` to the number of travelers (1-9); the returned `price` reflects the per-person fare Kayak displays for that search.

**How current are the prices?**
Every run searches Kayak live, so prices reflect what Kayak is showing at the time of the run and can change shortly after.

**Is this affiliated with Kayak?**
No. This is an independent data extraction tool and is not affiliated with, endorsed by, or sponsored by Kayak Software Corporation.
