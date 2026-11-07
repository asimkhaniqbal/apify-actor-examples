# Ticketmaster US Event & On-Sale Date Finder

Search Ticketmaster's US events catalog by keyword and get back structured event records — title, dates, venue, artists, sale status, and presale windows — without visiting the site yourself.

## What this actor does

- **Keyword search** — enter an artist name, genre, or a combined query like `concerts new york` the same way you would on Ticketmaster itself
- **Category and date filters** — narrow to one or more top-level categories (concerts, sports, arts & theater, family, film, miscellaneous) and/or a date range
- **Sale-status flags** — every event includes whether it's cancelled, postponed, rescheduled, sold out, or has limited availability
- **Optional cleanup filters** — drop cancelled/postponed/rescheduled or sold-out events automatically
- **Empty fields are omitted** — a field Ticketmaster doesn't populate for an event is left out rather than returned as null

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keyword` | string (required) | – | Free-text search — artist name, genre, city + category. |
| `maxResults` | integer | `20` (1–1000) | Hard cap on events emitted. |
| `skipSoldOut` | boolean | `false` | Drop events flagged sold-out by Ticketmaster. |
| `skipCancelled` | boolean | `true` | Drop events flagged cancelled, postponed, or rescheduled. |
| `dateFrom` | string (ISO date) | – | Earliest event start date (e.g. `2026-05-01`). Events before this are dropped. |
| `dateTo` | string (ISO date) | – | Latest event start date. Events after this are dropped. |
| `categories` | array of enums | – | Filter to: `concerts`, `sports`, `arts-theater`, `family`, `film`, `miscellaneous`. Multiple allowed. |
| `sort` | enum | – | Empty defaults to relevance ordering. One of `date,asc`, `date,desc`, `name,asc`, `name,desc`, `venueName,asc`, `random`. |

### Example input

```json
{
  "keyword": "taylor swift",
  "maxResults": 60,
  "skipSoldOut": true,
  "skipCancelled": true
}
```

## Output

One record per event, fields below. Empty fields are omitted.

```json
{
  "id": "2D006427A74987FB",
  "discoveryId": "G5eVZ_erE3bLD",
  "title": "James Taylor",
  "url": "https://www.ticketmaster.com/james-taylor-columbia-09-11-2026/event/2D006427A74987FB",
  "startDate": "2026-09-11T23:30:00Z",
  "onsaleDate": "2026-01-16T15:00:00Z",
  "timezone": "America/New_York",
  "isCancelled": false,
  "isPostponed": false,
  "isRescheduled": false,
  "isTBA": false,
  "isVirtual": false,
  "isSoldOut": false,
  "limitedAvailability": false,
  "venueName": "Colonial Life Arena",
  "venueCity": "Columbia",
  "venueState": "SC",
  "venue": {
    "name": "Colonial Life Arena",
    "city": "Columbia",
    "state": "SC",
    "country": "US",
    "countryName": "United States",
    "address": "801 Lincoln St.",
    "postalCode": "29208",
    "url": "https://www.ticketmaster.com/colonial-life-arena/venue/369551",
    "imageUrl": "https://s1.ticketm.net/dbimages/23068v.jpg",
    "latitude": 33.9942619,
    "longitude": -81.036104
  },
  "artists": [
    {
      "name": "James Taylor",
      "url": "https://www.ticketmaster.com/james-taylor/artist/736262",
      "imageUrl": "https://s1.ticketm.net/dam/.../ARTIST_PAGE_3_2.jpg"
    }
  ],
  "artistCount": 1,
  "daysUntilEvent": 269,
  "presaleDates": [
    {"name": "Ticketmaster Presale", "startDateTime": "2026-01-15T15:00:00Z", "endDateTime": "2026-01-16T03:00:00Z"}
  ],
  "scrapedAt": "2024-12-16T14:23:11+00:00"
}
```

### Output fields

- **`id`** / **`discoveryId`** — Ticketmaster's stable identifiers.
- **`title`** — event / lead artist name.
- **`url`** — direct link to the event ticket page.
- **`startDate`** / **`onsaleDate`** — ISO-8601 timestamps (UTC).
- **`timezone`** — IANA zone for the venue (e.g. `America/New_York`).
- **`isCancelled`** / **`isPostponed`** / **`isRescheduled`** / **`isTBA`** / **`isVirtual`** / **`isSoldOut`** / **`limitedAvailability`** — booleans straight from Ticketmaster.
- **`venueName`** / **`venueCity`** / **`venueState`** — flattened for quick filtering / table display.
- **`venue`** — nested object with the full venue record (name, address, country, lat/lon, image, URL).
- **`artists`** — array of `{name, url, imageUrl}` (lead + supporting acts).
- **`artistCount`** — derived count of items in `artists`.
- **`daysUntilEvent`** — derived integer: days from now (UTC) to `startDate`. Negative for events that have already happened.
- **`presaleDates`** — array of `{name, startDateTime, endDateTime}` for any presale windows.
- **`scrapedAt`** — ISO-8601 timestamp of the run.

## Use cases

- **Demand monitoring** — track when on-sale dates open for hot artists.
- **Tour analytics** — pull every event for an artist + parse city/state for tour-mapping.
- **Resale arbitrage research** — combine Ticketmaster face-value with secondary-market data.
- **Local-events digests** — query "concerts <city>" weekly to populate a curated list.
- **Sold-out alerting** — filter on `isSoldOut: true` and route to a notification.

## FAQ

**Do I need to configure anything to avoid being blocked?**
No — there's nothing to set up. Just supply a keyword and run it.

**Is this Ticketmaster's official API?**
No — this actor reads Ticketmaster's own public search results page. For high-volume programmatic access, Ticketmaster offers its own Discovery API (free developer tier at developer.ticketmaster.com).

**Why are some fields missing?**
Ticketmaster doesn't always populate every field. The actor follows an omit-empty contract: fields that aren't present on the source event are simply absent from the record (no nulls).

**How many results can it return?**
Up to `maxResults` (max 1,000). Ticketmaster paginates ~20 events per page; the actor stops once it reaches the cap or runs out of matching events. For thousands of records, prefer Ticketmaster's official Discovery API.

**Can I search by city / venue / date range?**
Use the keyword field (required for every run) — Ticketmaster's search understands queries like `concerts new york`, `taylor swift dallas`, or `comedy shows this weekend`. For an explicit date window, set `dateFrom` / `dateTo`. To narrow a keyword search to one or more top-level categories (e.g. only sports events among the results), add the `categories` filter alongside it.

**Why was a specific event skipped?**
- `skipCancelled: true` (default) drops cancelled / postponed / rescheduled events.
- `skipSoldOut: true` drops sold-out events.
- Events without a `title` or `id` are dropped (malformed).

**Is this affiliated with Ticketmaster?**
No. This is an independent actor that reads Ticketmaster's publicly available search results; it is not operated by or affiliated with Ticketmaster or Live Nation.
