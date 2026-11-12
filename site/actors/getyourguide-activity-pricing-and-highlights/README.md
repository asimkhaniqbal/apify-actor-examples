# GetYourGuide Activity Pricing & Highlights Extractor

Search GetYourGuide for tours and activities in any destination, or point this actor at specific activity/city/search URLs, and get back structured data for each one — pricing, rating, duration, highlights, cancellation policy, and supplier details.

## What this actor does

- **Destination or keyword search** — enter a city, landmark, or activity keyword (e.g. `rome`, `eiffel tower`, `food tour paris`) and it fetches matching activities
- **Direct URL input** — supply activity pages, city pages, or search-result URLs instead of a keyword
- **Pricing and discounts** — starting price, currency, original price, and discount percentage when a deal is running
- **Logistics and booking flags** — duration, meeting point, group size/age limits, free cancellation, skip-the-line, mobile ticket, pickup, and wheelchair-accessibility flags, where GetYourGuide shows them
- **Honest records** — a field GetYourGuide doesn't show for a given activity (for example, no supplier rating) is left out rather than returned empty

## Output per activity

- `type` — record type, always `gyg_activity` for a successfully scraped activity
- `id`, `url`, `title` — activity identity and link
- `shortDescription` — marketing description
- `city`, `country`, `latitude`, `longitude`, `meetingPoint` — where the activity takes place
- `price`, `originalPrice`, `currency`, `discountPercent`, `pricingType` — pricing and any active discount
- `rating`, `reviewCount`
- `duration`, `category`, `activityType`, `language`
- `images` — image URLs
- `highlights`, `whatIncluded`, `whatExcluded`, `bookingOptions` — bullet-point highlights, inclusions/exclusions, and ticket/option titles, when published
- `cancellationPolicy`
- `maxGroupSize`, `minAge` — group-size and age limits, when published
- `supplierName`, `supplierRating`
- `badges` — labels GetYourGuide displays on the card (e.g. "Skip the line", "Free cancellation")
- `availability` — availability message shown on the listing, when present
- `recentBookings` — recent-bookings social-proof text (e.g. "24 booked yesterday"), when the listing shows it
- `freeCancellation`, `skipTheLine`, `mobileTicket`, `instantConfirmation`, `pickupIncluded`, `wheelchairAccessible` — boolean flags for the matching booking feature
- `scrapedAt` — ISO-8601 timestamp the record was collected

If a run can't retrieve any activities, the dataset instead gets a single explanatory record (`type: "gyg_blocked"`) with the URL or query that was attempted and a reason, rather than returning nothing with no explanation.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `query` | string | `rome` | City, destination, or activity keyword to search. Ignored when `startUrls` is set |
| `startUrls` | array | – | Direct GetYourGuide search, activity, or city URLs. Overrides `query` when provided |
| `maxItems` | integer | `3` | Maximum number of activities to return (1–500) |
| `proxyConfiguration` | object | Apify `RESIDENTIAL` proxy, country `US` | Required. GetYourGuide blocks non-residential traffic, so this must stay set to a US residential proxy |

### Example: search a destination

```json
{
    "query": "rome",
    "maxItems": 5
}
```

### Example: scrape a specific activity and a search page together

```json
{
    "startUrls": [
        "https://www.getyourguide.com/rome-l33/rome-colosseum-arena-floor-palatine-forum-guided-tour-t217332/",
        "https://www.getyourguide.com/s/?q=paris"
    ],
    "maxItems": 10
}
```

## Use cases

- **Price monitoring** — track how an activity's price and discount change over time
- **Competitive research** — compare duration, inclusions, and cancellation terms across similar tours in a city
- **Itinerary planning** — pull highlights, meeting points, and booking flags for a shortlist of activities
- **Supplier mapping** — see which operators run the top-rated activities in a destination

## FAQ

**Does this need cookies or a login?**
No — GetYourGuide's search and activity pages are publicly accessible without an account.

**Why is a residential proxy required?**
GetYourGuide blocks traffic from standard datacenter IP ranges, so this actor needs a residential proxy to load pages at all. The `proxyConfiguration` input is pre-filled correctly — leave it as-is.

**What kind of search terms work?**
Any city, landmark, or activity keyword — e.g. `rome`, `paris`, `tokyo`, `eiffel tower`, `colosseum`, `food tour paris`.

**Can I scrape a single activity page directly?**
Yes — put its URL in `startUrls` instead of using `query`.

**Why are fields like `supplierRating` or `discountPercent` missing on some activities?**
Those fields only appear when GetYourGuide publishes that data for the activity — an activity with no active discount, for example, simply won't have a `discountPercent`.

**How fresh is the data?**
Every run fetches the page live at request time; nothing is cached.

**Is this affiliated with GetYourGuide?**
No. This is an independent actor that reads GetYourGuide's publicly visible pages; it is not operated by or affiliated with GetYourGuide.
