# Booking.com Hotel Price & Rating Snapshot Scraper

Point this actor at a destination and a pair of dates and it returns the hotels Booking.com shows on the first page of results — name, price, rating and location — as clean JSON rows. It's built for quick price and rating pulls rather than a full property breakdown, so it returns fewer fields per hotel than a detail-page scrape but runs through a search in a fraction of the time.

## What this actor does

- Searches Booking.com for any city, region or landmark and reads the listings straight off the results page
- Lets you set check-in/check-out dates, adult count, room count and currency to match a real traveler's search
- Captures guest rating, review count, rating label (e.g. "Excellent") and distance-from-center for every hotel returned
- Caps output at a number of hotels you choose, so you can pull a quick top-N snapshot or a larger batch
- Fields Booking.com doesn't display for a given hotel come back as an empty string or `0` rather than being left out of the row

## Output per hotel

- `name` — hotel or property name
- `url` — direct link to the property's Booking.com page
- `price` — nightly price for the searched dates
- `currency` — currency the price is shown in
- `rating` — guest review score (0-10 scale)
- `reviewCount` — number of guest reviews behind that score
- `reviewWord` — Booking.com's rating label (e.g. "Excellent", "Very good")
- `address` — neighborhood or district as shown in the listing
- `city` — the destination you searched for
- `distance` — distance from the city center/downtown, as shown on the listing
- `photoUrl` — main listing photo
- `checkin`, `checkout` — the dates used for this search
- `scrapedAt` — ISO-8601 timestamp of when the row was captured

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `location` | string | – | City, region or destination to search, e.g. `Paris` (required) |
| `checkinDate` | string | – | Check-in date as `YYYY-MM-DD`; defaults to tomorrow if left empty |
| `checkoutDate` | string | – | Check-out date as `YYYY-MM-DD`; defaults to 3 days after check-in if left empty |
| `adults` | integer | `2` | Number of adult guests |
| `rooms` | integer | `1` | Number of rooms |
| `currency` | string | `USD` | Currency code for returned prices, e.g. `EUR`, `GBP` |
| `maxResults` | integer | `50` | Maximum number of hotel rows to return (1-500) |
| `proxyConfiguration` | object | Residential | Proxy group used for the request; Booking.com blocks plain datacenter IPs |

### Example: weekend city break

```json
{
  "location": "Lisbon",
  "checkinDate": "2026-11-20",
  "checkoutDate": "2026-11-23",
  "adults": 2,
  "rooms": 1,
  "currency": "EUR",
  "maxResults": 30
}
```

### Example: family trip, larger snapshot

```json
{
  "location": "Orlando",
  "checkinDate": "2026-12-15",
  "checkoutDate": "2026-12-22",
  "adults": 4,
  "rooms": 2,
  "currency": "USD",
  "maxResults": 100
}
```

### Example: quick 10-hotel check

```json
{
  "location": "Tokyo",
  "maxResults": 10
}
```

## Use cases

- **Rate shopping** — pull current nightly prices for a destination before booking group travel
- **Revenue management** — snapshot competitor pricing and ratings for hotels in the same city
- **Travel content** — pre-fill a city guide or comparison article with live price and rating ranges
- **Lead qualification** — quickly check what's available and well-rated in a destination before deeper research
- **Price trend tracking** — run on a schedule across the same dates to watch how prices move as a trip date approaches

## FAQ

**Do I need a Booking.com account?**
No. All data comes from Booking.com's public search results; no login is involved.

**Why does this return fewer fields than a full hotel-detail scrape?**
This actor is built for speed: it reads only what's visible on the search results page, not each hotel's own detail page. For room types, amenities and policy details, a detail-level scraper is the better fit.

**Why is a residential proxy configured by default?**
Booking.com's search pages reject requests from plain datacenter IPs. A residential proxy group lets the search load reliably.

**What happens if I leave the dates empty?**
The actor defaults check-in to tomorrow and check-out to three days later, so you always get real current pricing even without specifying dates.

**How many hotels can one run return?**
Up to 500 per search via `maxResults`; the actor returns however many Booking.com's search page actually lists, up to that cap.

**Does `rating` use the same 0-10 scale Booking.com shows on the site?**
Yes, `rating` matches the guest score shown on Booking.com itself, and `reviewWord` is the label Booking.com attaches to that score.

**Can I search in a currency other than USD?**
Yes, set `currency` to any currency code Booking.com supports (e.g. `EUR`, `GBP`, `JPY`) and prices are returned in that currency.

**Is this affiliated with Booking.com?**
No. This is an independent, third-party actor that reads Booking.com's public search pages; it has no affiliation with Booking.com or Booking Holdings.
