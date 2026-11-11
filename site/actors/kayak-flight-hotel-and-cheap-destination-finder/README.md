# Kayak Flight, Hotel & Cheap Destination Finder

Kayak compares fares and rates from airlines, hotels and hundreds of travel sellers. This actor turns Kayak search pages into clean JSON: live flight itineraries, live hotel rates, cheap destinations from an origin airport, or any Kayak search URL you paste in.

## What this actor does

- **Search flights** between two airports or cities, on chosen dates and cabin, with nonstop/airline/price/duration/alliance filters, sorting and pagination.
- **Search hotels** for a city and stay dates, with star, guest-rating, price, free-cancellation, breakfast, internet, parking, amenity and property-type filters.
- **Explore cheap destinations** from an origin airport, optionally capped by budget, stops and trip length.
- **Scrape specific URLs**: paste Kayak flight, hotel, explore or flight-route URLs and the actor detects what each one is.
- **Rich records**: every itinerary includes all segments, airlines, flight numbers, aircraft, layovers, booking providers, baggage and a booking link; every hotel includes stars, guest rating, review count, coordinates, images, amenities, description and the cheapest provider.
- **Honest source labels**: each record has `accessMethod` (`live`, `flight-routes` or `wayback`) and, for archives, `archivedAt`. Route-page and archived prices are snapshots and are marked with `priceNote`.
- Omits fields that a particular result does not have, so records never contain blank placeholders.

## Modes

| Mode | What it returns |
|------|-----------------|
| `searchFlights` | Live itineraries for one route and date (or date range for a round trip). |
| `searchHotels` | Live hotel rates for a city and check-in/check-out dates. |
| `exploreDeals` | Cheap round-trip destinations from one origin airport. |
| `byUrls` | Records scraped from the Kayak URLs you provide. |

## Output per record

Every record starts with `recordType`, one of `kayakFlight`, `kayakHotel` or `kayakExplore`.

### Flight records (`searchFlights`, `byUrls` flight URLs, `flight-routes` fallback)

| Field | Description |
|-------|-------------|
| `flightId` | Kayak result identifier (or a stable route/time key for route-page records). |
| `origin` / `destination` | Actual airport codes of the itinerary. |
| `originCity` / `destinationCity` | City names for the airports. |
| `originAirportName` / `destinationAirportName` | Full airport names. |
| `departDate` / `returnDate` | Departure and return dates. |
| `price` | Cheapest fare Kayak displays for the itinerary, in `currency`. |
| `currency` | Price currency for the selected Kayak site. |
| `airline` / `airlines` | First operating airline and all airlines in the itinerary. |
| `stops` / `returnStops` | Connections on the outbound and return legs (0 = nonstop). |
| `duration` / `durationMinutes` | Outbound duration as text and minutes. |
| `totalDurationMinutes` | Outbound + return duration. |
| `departTime` / `arrivalTime` | Outbound departure and arrival times. |
| `returnDepartTime` / `returnArrivalTime` | Return-leg times when present. |
| `outbound` / `return` | Full leg objects with every segment, layover, flight number and aircraft. |
| `flightNumbers`, `aircraft` | Convenience lists across the itinerary. |
| `cabinClass`, `tripType` | Search cabin and one-way/round-trip. |
| `providerName` / `providerCode` | Cheapest booking provider. |
| `providers` | Up to 20 booking options with price, total price, provider and booking link. |
| `baggage` | Carry-on/checked-bag restrictions and fees where Kayak reports them. |
| `deepLink` / `bookingUrl` | Best booking link. `shareableUrl` links back to the itinerary on Kayak. |
| `accessMethod` | `live`, `flight-routes` or `wayback`. |
| `sourceUrl` | URL the record was read from. |
| `priceNote` | Present on fallback records; explains that the price is a route-level snapshot. |
| `archivedAt` | Archive timestamp for `wayback` records. |
| `scrapedAt` | UTC timestamp when the record was produced. |

### Hotel records (`searchHotels`, `byUrls` hotel URLs)

| Field | Description |
|-------|-------------|
| `hotelId` / `flightId` | Kayak hotel identifier. |
| `hotelName` / `airline` | Hotel name (`airline` is kept for backward compatibility). |
| `stars`, `rating`, `reviewCount`, `ratingCategory` | Star rating and guest reviews. |
| `price` / `totalPrice` | Nightly rate and total stay price. |
| `currency` | Rate currency. |
| `departDate` / `returnDate` | Check-in and check-out. |
| `checkIn` / `checkOut` / `nights` | Stay dates and number of nights. |
| `adults` / `rooms` | Occupancy used for the search. |
| `latitude` / `longitude`, `distanceFromCenter` | Location and distance from the city centre. |
| `city`, `propertyType`, `brandName` | Location and property classification. |
| `imageUrl` / `images` | Primary image and up to 10 image URLs. |
| `description` / `descriptionAttribution` | Hotel description and its source. |
| `amenities` / `amenityCodes` | Amenity names and Kayak codes. |
| `freeCancellation` / `freebies` | Cancellation and freebie information for the cheapest rate. |
| `providerName` / `providerCode`, `providers` | Cheapest provider plus up to 10 provider offers. |
| `bookingUrl` / `deepLink` | Booking link. `detailsUrl` opens the hotel on Kayak. |
| `accessMethod`, `sourceUrl`, `scrapedAt` | Source and timing metadata. |

### Cheap-destination records (`exploreDeals`)

| Field | Description |
|-------|-------------|
| `flightId` | Origin-destination key. |
| `origin` / `destination` | Origin city/airport code and destination airport code. |
| `originCity` / `destinationCity` / `destinationCountry` | City and country names. |
| `destinationCountryCode`, `departureAirportCode`, `airportName` | Extra location detail. |
| `price` / `currency` | Lowest round-trip price shown. |
| `airline` / `airlineCode` | Airline on the cheapest itinerary. |
| `stops` / `duration` / `durationMinutes` | Connections and maximum itinerary duration. |
| `departDate` / `returnDate` / `tripDays` | Suggested travel dates and length of stay. |
| `imageUrl` / `deepLink` | Destination image and Kayak flight link. |
| `latitude` / `longitude` | Destination airport coordinates. |
| `accessMethod`, `sourceUrl`, `scrapedAt` | Source and timing metadata. |

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mode` | String (select) | `searchFlights` | `searchFlights`, `searchHotels`, `exploreDeals` or `byUrls`. |
| `site` | String (select) | `kayak.com` | Country site, which also sets the currency. |
| `origin` | String | `NYC` | Origin airport or city for flights and explore. |
| `destination` | String | `LON` | Destination airport or city; in hotels mode, the city to stay in. |
| `departDate` | String | — | Departure date or hotel check-in (`YYYY-MM-DD`). Defaults to about 30 days from today. |
| `returnDate` | String | — | Return date or hotel check-out (`YYYY-MM-DD`). Defaults to seven days after departure. |
| `tripType` | String (select) | `round-trip` | `one-way` or `round-trip`. |
| `cabinClass` | String (select) | `economy` | `economy`, `premium-economy`, `business` or `first`. |
| `adults` | Integer | 1 | Adult travellers (1-9). |
| `children` | Integer | 0 | Children aged 2-17 (0-4). |
| `maxStops` | String (select) | `any` | `any`, `0`, `1`, `2` or `3`. |
| `airlines` | Array | – | Two-letter IATA airline codes. |
| `minPrice` / `maxPrice` | Integer | – | Price range in the site currency. |
| `maxDurationMinutes` | Integer | – | Maximum outbound flight duration. |
| `redeyeOnly` | Boolean | false | Overnight flights only. |
| `alliance` | String (select) | `any` | `any`, `STAR_ALLIANCE`, `ONEWORLD` or `SKYTEAM`. |
| `wifiOnly` | Boolean | false | Flights advertising Wi-Fi only. |
| `sortBy` | String (select) | `price` | `price`, `duration`, `departure`, `arrival` or `best`. |
| `hotelStars` | Array | – | Star ratings to include (`1`-`5`). |
| `rooms` | Integer | 1 | Hotel rooms (1-8). |
| `minRating` | String (select) | `any` | `any`, `6`, `7`, `8` or `9`. |
| `freeCancellation`, `freeBreakfast`, `freeInternet`, `freeParking`, `payLater` | Boolean | false | Rate/property filters. |
| `amenities` | Array | – | Kayak amenity IDs (for example `wifi`, `pool`). |
| `propertyType` | String (select) | `any` | `any`, `hotel` or `rental:apartment`. |
| `dealsOnly` | Boolean | false | Member/private deals only. |
| `hotelSortBy` | String (select) | `recommended` | `recommended`, `price_low`, `rating`, `distance` or `stars`. |
| `maxBudget` | Integer | – | Maximum explore-deal price. |
| `minTripDays` / `maxTripDays` | Integer | 1 / 30 | Explore trip-length range. |
| `destinationCountry` | String | – | Keep explore deals in one country (name or ISO code). |
| `urls` | Array | – | Kayak URLs for `byUrls` mode. |
| `maxItems` | Integer | 50 | Maximum records for the whole run (1-1000). |

### Example: round-trip flight search

```json
{
  "mode": "searchFlights",
  "origin": "NYC",
  "destination": "LON",
  "departDate": "2026-12-10",
  "returnDate": "2026-12-17",
  "tripType": "round-trip",
  "cabinClass": "economy",
  "adults": 1,
  "maxStops": "0",
  "airlines": ["BA", "VS"],
  "sortBy": "price",
  "maxItems": 50
}
```

### Example: hotels with filters

```json
{
  "mode": "searchHotels",
  "destination": "London",
  "departDate": "2026-12-10",
  "returnDate": "2026-12-17",
  "adults": 2,
  "rooms": 1,
  "hotelStars": ["4", "5"],
  "minRating": "8",
  "freeCancellation": true,
  "hotelSortBy": "price_low",
  "maxItems": 40
}
```

### Example: cheap destinations under a budget

```json
{
  "mode": "exploreDeals",
  "origin": "NYC",
  "maxBudget": 500,
  "maxStops": "0",
  "minTripDays": 4,
  "maxTripDays": 10,
  "destinationCountry": "Spain",
  "maxItems": 30
}
```

### Example: saved URLs

```json
{
  "mode": "byUrls",
  "urls": [
    "https://www.kayak.com/flights/NYC-LON/2026-12-10/2026-12-17",
    "https://www.kayak.com/hotels/London/2026-12-10/2026-12-17/2adults"
  ],
  "maxItems": 60
}
```

## How live access works

Kayak renders search results with JavaScript and streams them from its own internal search API. The actor opens the search in a real browser, captures the page's own session, and then reads the result pages directly. No account, cookie or proxy setup is required. If the direct search is blocked, the actor automatically retries through the platform's datacenter proxy.

When `origin` or `destination` is a city name such as `New York`, the actor resolves it to Kayak's metro or airport code (`NYC`) before searching. Three-letter IATA codes are used as-is.

If a **flight** search cannot be read live, the actor falls back to Kayak's public flight-route page for the same route. If that page is unavailable, it tries the Wayback Machine copy of the route page. Fallback records are real Kayak data but are **route-level snapshots, not date-specific live fares**; they carry `accessMethod` = `flight-routes` or `wayback` and a `priceNote` saying so. Hotel and explore searches have no such fallback and report the block in the run status message.

## Limitations

- Hotel and explore searches have no fallback when Kayak blocks live access; the run status message says when that happens.
- Route-page and Wayback fallback records are route-level snapshots. Price and airline filters apply to them; stop, duration and cabin filters and sorting do not.
- Kayak's hotel URL accepts adults and rooms, not child ages, so hotel searches use adults and rooms only.
- Time-of-day flight filters are not exposed because Kayak rejects them in the filter syntax; use `sortBy` `departure` or `arrival` instead.

## Use cases

- **Fare comparison**: compare airlines and providers for a route and dates.
- **Travel inspiration**: find cheap destinations from your home airport under a budget.
- **Hotel rate checks**: pull nightly and total rates with ratings and amenities.
- **Price tracking**: schedule the same search and store each run's dataset.
- **URL re-scraping**: turn saved Kayak search links into structured data.

## FAQ

**Do I need a Kayak account or API key?**
No. The actor reads publicly available search results and needs no credentials.

**Which currency will prices be in?**
The currency of the selected Kayak country site. Set `site` to `kayak.co.uk` for GBP, `kayak.de` for EUR, and so on.

**Why do some records say `accessMethod: flight-routes`?**
Kayak blocked the live search for that route. The record comes from Kayak's flight-route page and is a snapshot, so treat its price as indicative and re-run later for a live fare.

**What does `price` mean for hotels?**
`price` is the nightly rate from the cheapest provider; `totalPrice` is the total for the stay.

**Does `stops: 0` mean nonstop?**
Yes. Flights also have `returnStops` for the return leg.

**Can I search one-way flights?**
Yes. Set `tripType` to `one-way`; `returnDate` is ignored.

**How current are prices?**
`live` records are read from Kayak during the run. Route-page and `wayback` records are snapshots and can be stale; see `priceNote` and `archivedAt`.

**Is this affiliated with Kayak?**
No. This is an independent data extraction tool and is not affiliated with, endorsed by, or sponsored by Kayak Software Corporation.
