# Facebook Event Details & New-Event Monitor Scraper

Pull full details from public Facebook events — by direct URL, from a page's event listing, or via keyword/city search — without logging in. Get venue coordinates, hosts, ticket links, and recurrence info whenever Facebook shows them, plus an optional monitor mode that only reports events you haven't seen in a previous run.

## What this actor does

- Fetches full event details from a direct event URL, a page's `/events` listing, or a bare event ID
- Searches Facebook's public events index by keyword; a city in `searchLocation` is simply added to the search words
- Automatically fetches pages beyond Facebook's own ~8-result batch when a listing or search needs more (best-effort)
- Tracks previously-seen events per page/query across scheduled runs so Monitor mode can report only new ones
- Empty fields are omitted from each record

## Output per event

- `event_id`, `name`, `description`, `url`
- `start_timestamp`, `end_timestamp` — Unix timestamps
- `start_date`, `end_date` — ISO 8601
- `formatted_date` — Facebook's own human-readable date/time string
- `timezone`
- `is_online`, `is_canceled`, `is_past`, `is_sale_event`
- `location_name`, `location_address`, `location_city`, `one_line_address`
- `latitude`, `longitude`
- `photo_url`, `photo_width`, `photo_height`, `photo_caption`, `video_url`
- `hosts[]` — `{name, url, type, is_verified, profile_picture_url}`
- `creator` — `{id, name, type}`, the event's original creator
- `categories[]`
- `ticket_url`, `description_links[]` — external ticket link and every URL Facebook recognized in the description
- `online_type` — e.g. `MESSENGER_ROOM`, `THIRD_PARTY`
- `users_responded` — combined going + interested count
- `event_kind`, `rsvp_style`
- `event_frequency`, `has_child_events`, `child_events_count` — recurrence signals for series events
- `input_url` — the event or page URL you supplied, or for search results the Facebook events-search URL the actor built from your query
- `scraped_at` — ISO 8601 timestamp of collection

When an event found via a page listing or search can't be fetched in full (its detail page fails to load, shows a login wall, or can't be parsed), the actor pushes a minimal stub for it instead of skipping it: `event_id`, `name`, `url`, `formatted_date`, plus `is_canceled` (page listings) or `is_online` (searches), `input_url` and `scraped_at` — none of the venue, host, or media fields above. A direct event URL that fails produces no record.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `eventUrls` | array | – | Facebook event URLs, page event-listing URLs, or bare event IDs |
| `searchQuery` | string | – | Keyword search of Facebook's public events index (e.g. "music festival") |
| `searchLocation` | string | – | City or place (e.g. "Chicago") appended to the search text; usable alone or with `searchQuery`. It is a keyword, not a strict geographic filter |
| `monitorMode` | boolean | `false` | Only output events not seen in a previous run for the same page/query; direct event URLs are always fetched |
| `maxItems` | integer | `50` | Max events per page listing and for the search (1–500); doesn't apply to a specific event URL |
| `proxyConfiguration` | object | – | Optional proxy; most event pages work without one |

At least one of `eventUrls`, `searchQuery`, or `searchLocation` is required. They can be combined in one run.

### Example: specific events by URL

```json
{
    "eventUrls": [
        "https://www.facebook.com/events/1855253805343101",
        "https://www.facebook.com/events/1084489187170885"
    ]
}
```

### Example: city-scoped keyword search

```json
{
    "searchQuery": "food festival",
    "searchLocation": "Chicago",
    "maxItems": 100
}
```

### Example: scheduled monitoring of a page's events

```json
{
    "eventUrls": ["https://www.facebook.com/TED/events"],
    "monitorMode": true
}
```

## Use cases

- **Event calendar sites** — pull venue, date, and ticket details to populate a listings page
- **Scheduled monitoring** — run with `monitorMode` on a daily schedule to get alerted only about newly-posted events from a venue or organizer
- **Location intelligence** — map events by `latitude`/`longitude` across a city or region
- **Festival and conference research** — collect attendance signals and host info across similar events
- **Competitive tracking** — watch a competitor's page events to see what they're promoting and when

## FAQ

**Is this affiliated with Facebook or Meta?**
No. This is an independent actor that reads publicly available Facebook event pages and is not affiliated with Meta Platforms, Inc.

**Do I need a Facebook login or cookies?**
No. The actor only reads public event data and needs no authentication.

**Why was an event not returned?**
It's likely private, deleted, or restricted to logged-in viewers — only public events are accessible.

**Does this work for Facebook Group events?**
No. Group events require login and aren't supported — only Pages and public event/search URLs are.

**Why did a page or search return fewer than `maxItems`?**
Facebook's own page serves roughly 8 results per request; fetching additional pages beyond that is best-effort and occasionally returns just the first batch. The run still completes normally either way — re-running usually gets the rest.

**Can I get every date of a recurring event series?**
Not from one event URL. Facebook returns `event_frequency`, `has_child_events`, and `child_events_count` as summary signals but doesn't expose sibling occurrence URLs to logged-out visitors — each occurrence must surface independently via search or a page listing.

**Why is `users_responded` a single number instead of separate going/interested counts?**
Facebook itself stopped exposing the split figures publicly; only the combined count remains available.

**How does Monitor mode decide what's "new"?**
It remembers which events it already reported for a given page or search text across runs, and only outputs events it hasn't reported before for that same target. Changing the query or location wording starts a fresh history. A specific event URL is always fetched since there's no "new" concept for one fixed event.

**Does `searchLocation` guarantee events in that city?**
No. Facebook's public event search has no location filter, so the city name is added to your keywords (e.g. "food festival Chicago"). That usually surfaces local events, but check `location_city` if you need a strict match.
