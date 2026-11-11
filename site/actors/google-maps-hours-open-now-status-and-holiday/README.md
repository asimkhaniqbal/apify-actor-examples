# Google Maps Hours, Open-Now Status & Holiday Schedule

Read a business's full week of opening hours, its live open/closed state, and the holiday or special hours Google publishes for a date — from a URL, a Place ID, a batch of either, or a keyword search. Every row is structured, timezone-aware, and ready for a database, dashboard, or alerting job.

## Two ways to use it

- **Places mode** — paste Google Maps place URLs, canonical `ChIJ…` Place IDs, or a mixture of both. Short `maps.app.goo.gl` links are resolved automatically.
- **Search mode** — describe what you want ("coffee shops in New York") and the actor ranks the places Google Maps returns, then reads the opening hours for each one.

Both modes are cookieless. No Google account, API key, or proxy setup is needed.

## What a record contains

```json
{
  "recordType": "businessHours",
  "businessName": "Starbucks Reserve Roastery New York",
  "placeId": "ChIJIQDZbb9ZwokRNRez4RviXGQ",
  "featureId": "0x89c259bf6dd90021:0x645ce21be1b31735",
  "cid": "995473537737540317",
  "kgmid": "/g/11j7w8j09h",
  "placeUrl": "https://www.google.com/maps/place/?q=place_id:ChIJIQDZbb9ZwokRNRez4RviXGQ",
  "sourceUrl": "https://www.google.com/maps/place/Starbucks+Reserve+Roastery+New+York/...",
  "address": "61 9th Ave, New York, NY 10011, United States",
  "categories": ["Coffee shop", "Bakery", "Cafe", "Cocktail bar", "Coffee roastery", "Tourist attraction"],
  "categoryName": "Coffee shop",
  "phone": "+1 212-691-0531",
  "website": "https://www.starbucksreserve.com/locations/new-york-roastery",
  "rating": 4.4,
  "reviewCount": 7956,
  "location": { "lat": 40.7416, "lng": -74.0053 },
  "isOpenNow": false,
  "is24Hours": false,
  "openingHoursText": "Closed · Opens 7:30 AM",
  "hoursStatus": "Closed - Opens 7:30 AM",
  "hours": {
    "monday": { "open": "7:30 AM", "close": "10 PM" },
    "tuesday": { "open": "7:30 AM", "close": "10 PM" },
    "wednesday": { "open": "7:30 AM", "close": "10 PM" },
    "thursday": { "open": "7:30 AM", "close": "10 PM" },
    "friday": { "open": "7:30 AM", "close": "11 PM" },
    "saturday": { "open": "7:30 AM", "close": "11 PM" },
    "sunday": { "open": "7:30 AM", "close": "10 PM" }
  },
  "weeklyPattern": "Mon-Thu 7:30 AM - 10 PM, Fri-Sat 7:30 AM - 11 PM, Sun 7:30 AM - 10 PM",
  "totalHoursPerWeek": 103.5,
  "holidayHours": [
    {
      "day": "wednesday",
      "date": "2026-11-11",
      "label": "Veterans Day",
      "note": "Veterans Day might affect these hours",
      "hoursMayDiffer": true
    }
  ],
  "specialHours": [
    {
      "day": "wednesday",
      "date": "2026-11-11",
      "label": "Veterans Day",
      "note": "Veterans Day might affect these hours",
      "hoursMayDiffer": true
    }
  ],
  "timezone": "America/New_York",
  "currentLocalTime": "2026-11-07T03:21:45-05:00",
  "nextOpenTime": "2026-11-07T07:30:00-05:00",
  "timeUntilNextChange": "Opens in 4 hours",
  "hoursSurfaceRendered": true,
  "scrapedAt": "2026-11-07T08:21:45+00:00"
}
```

Fields are omitted when the source does not publish them — there are no `null` placeholders.

## Field reference

- `businessName` — the place name Google Maps displays.
- `placeId` — canonical Google Place ID (`ChIJ…`), when exposed.
- `featureId` — legacy Maps feature token (`0x…:0x…`), kept separate from `placeId`.
- `cid` — Google's decimal CID for the place, when exposed.
- `kgmid` — Google Knowledge Graph MID, when exposed.
- `placeUrl` — canonical Maps URL derived from the place identity.
- `sourceUrl` — the URL or Place ID that produced this row.
- `searchQuery` — the search query, in Search mode only.
- `address` — street address as Google formats it.
- `categories` — all category labels, with `categoryName` as the primary one.
- `phone` — public phone number.
- `website` — public website URL.
- `rating` — average star rating (0–5).
- `reviewCount` — number of Google reviews.
- `location` — place centroid as `lat`/`lng`; never the map viewport.
- `isOpenNow` — Google's live open/closed state at scrape time.
- `temporarilyClosed` — `true` only when Google shows that exact badge.
- `permanentlyClosed` — `true` only when Google shows that exact badge.
- `is24Hours` — whether the complete weekly schedule is open 24 hours.
- `openingHoursText` — the status line as Google renders it.
- `hoursStatus` — the same line cleaned for display.
- `hours` — structured schedule by weekday; formats below.
- `weeklyPattern` — compact summary of the week.
- `totalHoursPerWeek` — scheduled hours for a complete week.
- `holidayHours` — holiday and special-date entries; see below.
- `specialHours` — the same entries under the alternate field name kept for existing integrations.
- `timezone` — IANA timezone reported by Google for the place.
- `currentLocalTime` — observation timestamp in the place's timezone.
- `nextOpenTime` — next opening as a local-time ISO timestamp.
- `nextCloseTime` — next closing as a local-time ISO timestamp.
- `timeUntilNextChange` — readable countdown such as "Closes in 2 hours".
- `hoursSurfaceRendered` — positive evidence that Google returned a schedule for the place.
- `recordType` — constant row label (`businessHours`).
- `scrapedAt` — UTC timestamp when the row was collected.

Each `hours` day uses one of these shapes:

| Shape | Meaning |
|---|---|
| `{"open": "9:00 AM", "close": "5:00 PM"}` | Regular opening window |
| `{"open": "...", "close": "...", "intervals": [...]}` | Split shift; every window is listed |
| `{"closed": true}` | Closed that day |
| `{"open": "12:00 AM", "close": "11:59 PM", "is24Hours": true}` | Open 24 hours |
| `{"raw": "..."}` | Source format was not a clock range; the text is preserved |

Holiday entries describe the date Google attaches them to:

- `date` — the calendar date, `YYYY-MM-DD`.
- `day` — the weekday of that date.
- `label` — Google's holiday name ("Veterans Day", "Thanksgiving").
- `note` — Google's own wording.
- `specialHours: true` plus `status`, `open`, and `close` — Google publishes different hours for that date.
- `hoursMayDiffer: true` — Google warns that the regular hours may change but has not published the replacement yet.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `places` | `places` reads URLs/Place IDs; `search` discovers places by keyword |
| `placeUrls` | array | – | Google Maps place URLs (batch supported; the prefill shows one example) |
| `placeIds` | array | – | Canonical Place IDs beginning with `ChIJ` |
| `searchQuery` | string | – | Keyword query for Search mode |
| `location` | string | – | Optional area appended to the search query |
| `maxResults` | integer | `20` | Maximum places to read in Search mode (1–500) |
| `language` | select | `en` | Maps UI language for the status line and labels |
| `openNowOnly` | boolean | `false` | Keep only places that are open at scrape time |
| `maxConcurrency` | integer | `3` | Places requested in parallel in Places mode (1–8) |

### Example: one URL

```json
{
  "mode": "places",
  "placeUrls": ["https://www.google.com/maps/place/Starbucks/@40.7127281,-74.0060152,17z/"]
}
```

### Example: a mixed batch

```json
{
  "mode": "places",
  "placeUrls": [
    "https://www.google.com/maps/place/Starbucks/@40.7127281,-74.0060152,17z/",
    "https://maps.app.goo.gl/exampleShortLink"
  ],
  "placeIds": ["ChIJabtoT7hlwokRLECK0SW3_8I", "ChIJd3dF_VBZwokRAAFm3WMspbk"],
  "openNowOnly": false
}
```

### Example: keyword search

```json
{
  "mode": "search",
  "searchQuery": "coffee shops",
  "location": "New York, NY",
  "maxResults": 25,
  "openNowOnly": true
}
```

### Example: localized labels

```json
{
  "mode": "places",
  "placeUrl": "https://www.google.com/maps/place/McDonald%27s/@40.7580,-73.9855,17z/",
  "language": "es"
}
```

## Legacy input keys

Saved tasks from earlier versions keep working without edits. The actor maps
`placeUrl`, `placeurls`, `placeids`, `query`, `keyword`, `maxItems`, `limit`,
and `openNow` onto the fields above inside the code. `proxyConfiguration` is
accepted but no longer needed: requests start direct and move to datacenter or
residential connections only after a real block.

## How it works

Google Maps serves its place data through the same public XHR endpoints the
website uses. The actor asks those endpoints for the place record, reads the
weekly schedule, holiday rows, live status, and timezone directly from the
structured payload, and normalizes clock values itself. That avoids cookie
walls, browser rendering, and fragile DOM selectors. A browser-based reader is
kept only as a fallback for unusual URLs that carry no resolvable place token.

## FAQ and honest limits

**Do I need an API key or cookies?** No. The endpoints are public and the actor
never signs in.

**Why is `nextCloseTime` missing?** When the place is closed, there is no
current closing to report. When a holiday replaces the weekly schedule, the
countdown fields are omitted rather than computed from the wrong schedule.

**How far ahead does holiday data go?** Google attaches holiday rows to the
seven dates it currently displays (today plus six days). Dates outside that
window are not published by the source and are not invented.

**A place has no hours row. Why?** Some listings genuinely publish no hours,
and some categories (for example a pure service area) have no schedule. Those
places are reported in the run status and skipped instead of emitting an empty
placeholder row.

**Is this affiliated with Google?** No. It is an independent reader of public
Google Maps data.
