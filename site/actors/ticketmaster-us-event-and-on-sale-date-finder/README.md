# Ticketmaster US Event & On-Sale Date Finder

Collect upcoming Ticketmaster events in the United States and turn them into clean, structured records: event and on-sale dates, presale windows, venue address and coordinates, performers, genre, images, sale status, and — when Ticketmaster publishes them — primary price levels.

Run it without an API key, login, cookie, or proxy setup. It reads Ticketmaster's own public event endpoints and returns one row per event (or one row per name suggestion in Suggestions mode).

## What you can collect

| Mode | What it does | What to provide |
|---|---|---|
| **Search by keyword** | Artist, team, show, genre or a combined phrase such as `taylor swift`, `lakers`, `comedy austin` | `searchQuery` |
| **Browse by category** | Music, Sports, Arts & Theatre, Family, Film or Miscellaneous | one or more `categories` |
| **Artist calendar** | Every upcoming event for an artist or attraction | `artistIds` (IDs or Ticketmaster artist URLs) |
| **Venue calendar** | Every upcoming event at a venue | `venueIds` (IDs or Ticketmaster venue URLs) |
| **Event details** | Full record for exact events | `eventIds` (IDs, discovery IDs or Ticketmaster event URLs) |
| **Name suggestions** | Matching artists, venues and events with counts | `searchQuery` |

You can also paste any Ticketmaster search, artist, venue, event or category URL into `startUrls`; the actor routes each URL automatically and can mix several URL types in one run.

## Filters

- **Date window** — `dateFrom` / `dateTo` (`YYYY-MM-DD`). Applied by Ticketmaster for search, artist and venue modes; applied to the returned rows for category and event modes.
- **Categories** — filter keyword, artist and venue results to one or more top-level categories.
- **Sort** — relevance or soonest first.
- **United States only** — on by default; turn it off to include the international listings Ticketmaster mixes into US results.
- **Status cleanup** — optionally drop sold-out, cancelled, postponed or rescheduled events.
- **Maximum records** — 1 to 1,000 per run.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `category`, `artist`, `venue`, `event` or `suggestions`. |
| `searchQuery` | string | – | Artist, team, show or phrase. Used by Search and Suggestions. Legacy tasks that send `keyword` keep working. |
| `categories` | multi-select | – | `concerts`, `sports`, `arts-theater`, `family`, `film`, `miscellaneous`. Targets in Category mode, filters elsewhere. |
| `artistIds` | string list | – | Artist IDs or Ticketmaster artist URLs (mode=artist). |
| `venueIds` | string list | – | Venue IDs or Ticketmaster venue URLs (mode=venue). |
| `eventIds` | string list | – | Event IDs, discovery IDs or Ticketmaster event URLs (mode=event). |
| `startUrls` | string list | – | Ticketmaster search/artist/venue/event/category URLs. Overrides `mode` targets. |
| `dateFrom` / `dateTo` | string | – | Inclusive event-date window (`YYYY-MM-DD`). |
| `sort` | select | `relevance` | `relevance` or `date` (soonest first). Legacy values such as `date,asc` are still accepted and mapped. |
| `maxResults` | integer | `20` | 1–1,000 records across all targets. |
| `skipSoldOut` | boolean | `false` | Drop sold-out events. |
| `skipCancelled` | boolean | `true` | Drop cancelled, postponed and rescheduled events. |
| `usOnly` | boolean | `true` | Keep only US-venue events. |
| `includeDetails` | boolean | `true` | Add description, genre, doors time, images, seat map, venue address and price levels (up to two extra requests per event). |

### Examples

Search with a date window and full details:

```json
{
  "mode": "search",
  "searchQuery": "taylor swift",
  "dateFrom": "2026-12-01",
  "dateTo": "2027-12-31",
  "sort": "date",
  "maxResults": 60,
  "includeDetails": true
}
```

Browse sports events in a category:

```json
{
  "mode": "category",
  "categories": ["sports"],
  "maxResults": 40
}
```

Artist calendar by URL:

```json
{
  "mode": "artist",
  "artistIds": ["https://www.ticketmaster.com/barry-manilow-tickets/artist/735585"],
  "maxResults": 30
}
```

Exact events:

```json
{
  "mode": "event",
  "eventIds": [
    "0D00632F271D385A",
    "https://www.ticketmaster.com/event/1E00648A835556FD"
  ]
}
```

Mixed start URLs:

```json
{
  "startUrls": [
    "https://www.ticketmaster.com/search?q=comedy&startDate=2026-12-01&endDate=2026-12-31",
    "https://www.ticketmaster.com/hertz-arena-tickets-estero/venue/107143",
    "https://www.ticketmaster.com/event/0D00632F271D385A"
  ],
  "maxResults": 50
}
```

## Output

One record per event. Empty fields are omitted — the actor never emits `null`, empty strings or sentinel values.

| Field | Description |
|---|---|
| `recordType` | `event` or `suggestion`. |
| `id` / `discoveryId` | Ticketmaster event identifiers. |
| `title` | Event name. |
| `url` / `sourceUrl` | Event page URL. |
| `startDate` / `endDate` | Event start and end timestamps from Ticketmaster. |
| `onsaleDate` / `offsaleDate` | Public on-sale and off-sale timestamps. |
| `doorsDate` | Doors-open timestamp when published. |
| `timezone` | Venue IANA time zone. |
| `dateDisplay` / `spanMultipleDays` | How the date is displayed and whether the event spans days. |
| `presaleDates` | `[{name, startDateTime, endDateTime}]` presale windows. |
| `isCancelled` / `isPostponed` / `isRescheduled` / `isTBA` / `isVirtual` / `isSoldOut` / `limitedAvailability` | Sale and schedule flags. |
| `ticketingStatus` / `eventChangeStatus` | Raw Ticketmaster status tokens. |
| `eventType` / `productType` / `isGA` | Event type, product type and general-admission flag. |
| `isResaleEnabled` / `faceValueResaleEnabled` / `transferDisabled` | Ticket transfer and resale flags. |
| `venue` | Nested venue object: name, city, state, country, address, postal code, URL, image, coordinates, ID. |
| `venueName` / `venueCity` / `venueState` / `venueCountry` / `venueCountryName` / `venueAddress` / `venuePostalCode` / `venueUrl` / `venueImageUrl` / `latitude` / `longitude` | Flattened venue fields for tables, maps and filters. |
| `artists` | `[{name, url, imageUrl, id, discoveryId, rank}]`. |
| `artistCount` / `performerNames` | Derived lineup counts and a comma-separated lineup. |
| `classifications` | `[{segment, genre, subGenre, type, subType}]` from the event detail. |
| `primaryCategory` / `subCategory` / `majorCategory` / `majorCategoryId` | Human-readable category and genre labels plus the ticketmaster category ID. |
| `lowestPrice` / `highestPrice` / `priceCurrency` / `priceOfferCount` / `priceOffers` / `lowestResalePrice` | Primary price levels (base ticket price before fees) when Ticketmaster publishes offers. Protected access-code offers are ignored. |
| `imageUrl` / `seatMapUrl` / `staticMapUrl` | Event image and seat-map URLs. |
| `description` | Event notes when the organizer publishes them. |
| `currency` / `marketCode` / `sourceName` | Currency and marketplace metadata. |
| `local` / `sameRegion` / `partnerEvent` / `isPartner` | Region and partner-listing flags. |
| `daysUntilEvent` | Whole days from now to the event start. Negative for past events. |
| `scrapedAt` | ISO-8601 timestamp of the run. |

### Sample record

```json
{
  "recordType": "event",
  "id": "0D00632F271D385A",
  "discoveryId": "vvG1VZbwHE6K-Q",
  "title": "Manilow: The Last Estero Concerts",
  "url": "https://www.ticketmaster.com/manilow-the-last-estero-concerts-estero-florida-01-16-2027/event/0D00632F271D385A",
  "startDate": "2027-01-17T00:00:00Z",
  "onsaleDate": "2025-09-26T14:00:00Z",
  "timezone": "America/New_York",
  "venueName": "Hertz Arena",
  "venueCity": "Estero",
  "venueState": "FL",
  "venueCountry": "US",
  "majorCategory": "concerts",
  "classifications": [
    {
      "segment": "Music",
      "genre": "Pop",
      "subGenre": "Adult Contemporary"
    }
  ],
  "artists": [
    {
      "name": "Barry Manilow",
      "url": "https://www.ticketmaster.com/barry-manilow-tickets/artist/735585",
      "id": "735585"
    }
  ],
  "artistCount": 1,
  "lowestPrice": 110.0,
  "highestPrice": 354.0,
  "priceCurrency": "USD",
  "priceOfferCount": 14,
  "description": "The Event Organizer has had to postpone your event...",
  "scrapedAt": "2026-11-07T16:23:11+00:00"
}
```

Suggestion records use `recordType: "suggestion"` with `id`, `title`, `url`, `suggestionType`, `count`, `imageUrl`, and location fields when Ticketmaster provides them.

## Use cases

- Watch on-sale and presale windows for artists on a tour.
- Build city or venue event calendars without visiting Ticketmaster.
- Track sold-out, postponed and rescheduled events for alerts.
- Enrich event datasets with venue coordinates, categories and price levels.
- Research touring patterns and pricing across a date range.
- Power event-discovery apps, newsletters and research notebooks.

## FAQ

**Do I need an API key or a Ticketmaster account?**
No. The actor reads Ticketmaster's publicly served event data. There is no login, cookie, token or API-key input.

**Do I need to configure a proxy?**
No. The actor chooses its network path automatically; there is nothing to set up.

**Why is a price missing for some events?**
Ticketmaster only publishes primary price levels for events with open inventory. Partner listings (TicketWeb, Universe) and some add-on products have no ticket offers, so price fields are omitted rather than estimated. The actor never invents a price.

**Are prices the final checkout price?**
No. Price fields are Ticketmaster's published base ticket prices before service and processing fees. The event page adds those fees at checkout, so use the values to compare price levels rather than as a final quote.

**Why did the run stop before `maxResults`?**
Ticketmaster serves 20 events per page and stops at page 48, so one query tops out at 980 rows. Narrow the date window, use more categories, or run separate queries to collect more. A page that Ticketmaster keeps rejecting is retried automatically; if it still fails, the run's status message says the query stopped early so you can rerun it.

**Why are details missing for a specific event?**
Some events are hosted by partner ticketing sites. Ticketmaster's public event detail endpoint does not return those records, so the actor keeps the search-level fields it can verify and omits the rest.

**Does it return international events?**
The actor is positioned for the US market and keeps US venues by default. Turn off `usOnly` to include the international listings Ticketmaster returns on the US site.

**What happens with duplicate IDs or mixed URLs?**
Duplicates are removed by event ID across all targets, and mixed `startUrls` are merged into one dataset. With several targets, `maxResults` is split evenly and any unused share rolls over to the next target.

**Are old tasks still compatible?**
Yes. Legacy inputs using `keyword`, `dateFrom`, `dateTo`, `categories`, `sort` values such as `date,asc`, `skipSoldOut`, `skipCancelled` and `maxResults` keep running; renamed keys are mapped inside the actor.

## Limitations

- 980 rows per query (Ticketmaster's page ceiling). Split by date range or category to go wider.
- Price levels are base prices before fees, available only when Ticketmaster publishes ticket offers; partner events do not expose detail.
- A page that keeps failing after automatic retries can end a query early; the run says so and a rerun resumes with fresh proxy sessions.
- Seat-level and resale listings, and artist reviews, are not part of this actor.
- US-market focus; other Ticketmaster country sites are out of scope.
- Ticketmaster can change its public endpoints or markup at any time. The actor fails cleanly with a status message instead of emitting placeholder data.

## Notes

This is an independent data tool. It is not affiliated with, endorsed by, or operated by Ticketmaster or Live Nation. Ticketmaster and Live Nation are trademarks of their respective owners.
