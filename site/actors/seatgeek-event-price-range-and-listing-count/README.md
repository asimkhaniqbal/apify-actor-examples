# SeatGeek Event Price Range & Listing Count Finder

SeatGeek is a US ticket marketplace for sports, concerts, theater and comedy. This actor finds upcoming events by keyword, performer, team, venue or category, then returns each event's date, venue, performers, price range and live listing/ticket counts. It reads SeatGeek's public event catalog directly — no account, cookies, API key or proxy required.

## What this actor does

- **Search by intent:** `search` accepts an artist, team, venue or keyword and resolves it to SeatGeek's best-matching performer, venue or category before paging through that entity's events.
- **Browse all 156 categories:** every sports league, concert genre, Broadway/theater type and entertainment category SeatGeek organizes events into.
- **Fetch pasted URLs:** category, performer, venue, event and search URLs all work.
- **Real pagination:** pages through SeatGeek's event catalog up to 10,000 events per query instead of stopping at the first page.
- **Server-side filters and sorting:** date range, lowest-price range, city, state and sort order are pushed to SeatGeek, so `maxItems` keeps the cheapest/soonest/most popular events rather than an arbitrary slice.
- **Listing-count filter:** keep only events with at least N live listings.
- **Clean records:** empty fields are omitted, never emitted as `null` or placeholder text.

## Output per event

- `eventId`, `eventTitle`, `eventType`
- `eventDate` (`YYYY-MM-DD`), `eventDateTimeLocal`, `eventDateTimeUtc`, `eventEndDateTimeUtc`
- `dateTbd`, `timeTbd`, `datetimeTbd`, `salesEndDateTimeUtc`, `seasonStage`
- `venueId`, `venueName`, `venueSlug`, `venueAddress`, `venueExtendedAddress`, `venueDisplayLocation`
- `venueCity`, `venueState`, `venueCountry`, `venuePostalCode`, `venueCapacity`, `venueUpcomingEventCount`
- `venueLatitude`, `venueLongitude`, `venueTimezone`, `venueUrl`
- `performers[]`, `primaryPerformer`, `performerIds[]`, `primaryPerformerId`
- `category`, `categorySlug`, `taxonomyPath[]`, `taxonomyIds[]`
- `priceMin`, `priceMax`, `priceAverage`, `priceMedian`, `currency`
- `listingCount`, `visibleListingCount`, `ticketCount`
- `thumbnailUrl`
- `isOpen`, `isRescheduled`, `eventStatus`, `announceDate`
- `popularityScore` (SeatGeek's own 0–1 event popularity)
- `listingUrl`, `sourceUrl`, `searchQuery`, `searchMatch` (`performer`, `venue`, `category`, `keyword`, `event` or `page`)
- `recordType: "event"`, `scrapedAt`

Price and listing fields are omitted for events that currently have no ticket inventory — that is a real source condition, not a scrape failure.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byCategory` / `byUrl` |
| `searchQuery` | string | `New York Yankees` | Artist, team, venue or keyword (mode=search) |
| `category` | string | `nba` | One of 156 SeatGeek categories (mode=byCategory) |
| `startUrls` | array | – | Category, performer, venue, event or search URLs (mode=byUrl) |
| `dateFrom` | string | – | Only events on/after this date (`YYYY-MM-DD`) |
| `dateTo` | string | – | Only events on/before this date (`YYYY-MM-DD`) |
| `minPrice` | int | – | Lowest listed price ≥ this amount |
| `maxPrice` | int | – | Lowest listed price ≤ this amount |
| `minListingCount` | int | – | Only events with at least N live listings |
| `city` | string | – | Venue city (case-insensitive) |
| `state` | string | – | Venue state/province code, e.g. `NY` |
| `sortBy` | string | `relevance` | `relevance` / `date` / `priceLow` / `priceHigh` / `popularity` |
| `maxItems` | int | `20` | Hard cap on emitted records (1–10,000) |

### Example: search by team name

```json
{
  "mode": "search",
  "searchQuery": "New York Knicks",
  "maxItems": 50
}
```

### Example: cheapest NFL tickets with listings

```json
{
  "mode": "byCategory",
  "category": "nfl",
  "sortBy": "priceLow",
  "minListingCount": 10,
  "maxItems": 100
}
```

### Example: filter concerts by date and price range

```json
{
  "mode": "byCategory",
  "category": "concerts",
  "dateFrom": "2026-12-01",
  "dateTo": "2026-12-31",
  "minPrice": 50,
  "maxPrice": 300,
  "maxItems": 200
}
```

### Example: paste mixed SeatGeek URLs

```json
{
  "mode": "byUrl",
  "startUrls": [
    {"url": "https://seatgeek.com/venues/madison-square-garden/tickets"},
    {"url": "https://seatgeek.com/coldplay-tickets"},
    {"url": "https://seatgeek.com/nba-tickets"}
  ]
}
```

## Use cases

- **Price monitoring:** track the lowest listed price for a team, artist, venue or category over time.
- **Inventory research:** compare listing and ticket counts across an upcoming season or tour.
- **Event discovery:** build a feed of upcoming events filtered by date, city, state and budget.
- **Fan alerts:** power a "price drop" or "new date announced" notification for a favorite performer.
- **Venue analytics:** pull every upcoming event at a specific arena or theater.

## FAQ

**Do I need a SeatGeek account or API key?** No. The actor uses the public client id that SeatGeek's own website ships to every visitor. No login, cookie, API key or proxy is required. If SeatGeek ever blocks a request, the actor automatically retries through Apify's built-in datacenter proxy.

**How does `search` resolve a name?** It asks SeatGeek's autosuggest endpoint for matching performers and venues, prefers an exact name match, and otherwise picks the closest performer/venue match. If the query is a known category (for example `nba`, `comedy` or `concerts`), it browses that category. When the exact performer or venue has no upcoming events, it falls back to SeatGeek's keyword results so the run still returns listings; `searchMatch` on each record says whether it came from an exact `performer`, `venue`, `category`, or a `keyword` fallback.

**Is `dateTo` inclusive?** Yes. A date range includes every event on both the `dateFrom` and `dateTo` days in the venue's local time.

**Why do some events have no price?** SeatGeek returns price and listing statistics only while at least one ticket is listed. For sold-out, postponed or not-yet-on-sale events those fields are omitted.

**How many events can I get?** Up to 10,000 per query — SeatGeek's API returns at most 100 events per page and stops paginating at page 100. Split a larger request by date range or category to cover more.

**Does it include per-listing Deal Score?** No. Deal Score is computed per individual ticket listing, not at the event level. This actor returns event-level price and listing aggregates instead.

**Is this US-only?** Yes. It reads the US `seatgeek.com` marketplace; SeatGeek's separate country sites are not covered.

## Compatibility with saved tasks

Existing tasks keep working. Legacy input keys such as `query`, `searchQueries`, `categoryId`, `startDate`/`endDate`, `priceMin`/`priceMax`, `venueCity`/`venueState`, `maxResults`/`limit` and `sort` are mapped in code. The old `proxyConfiguration` key is accepted and ignored because the public API needs no proxy. Existing output fields were kept and new fields were added beside them.

## Limitations

- **10,000 events per query.** The upstream events API caps pagination at page 100 with 100 events per page.
- **No event-level guarantee of inventory.** Price and listing fields appear only when SeatGeek has live listings for that event.
- **Category snapshots.** Categories and their ids follow SeatGeek's public taxonomy; an entirely new league may need an actor update. Like SeatGeek's own category pages, parent leagues hide add-on taxonomies (Stadium Tours under NFL, Arizona Fall League under MLB); both remain browsable as their own categories.
- **No per-listing seat/section/row data.** This actor returns event-level pricing and inventory, not individual ticket rows.
