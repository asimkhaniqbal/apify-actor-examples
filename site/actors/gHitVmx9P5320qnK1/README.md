# Google Maps Hours, Open-Now Status & Holiday Schedule

Pull structured opening hours and live open/closed status straight from any Google Maps place page. Hand it one place URL, a batch of URLs, or a list of Google Place IDs, and get back a parsed weekly schedule, holiday/special hours, and a countdown to the next opening or closing — all in one run.

## What this actor does

- **Structured hours for every day of the week** — open time, close time, or "closed"
- **Live open/closed status** (`isOpenNow`) at the moment of scraping, plus an inline status badge like "Open - Closes 11 PM"
- **Next open/close times and countdown**, computed in the business's own local timezone
- **Weekly summary** — `is24Hours`, `totalHoursPerWeek`, and a readable `weeklyPattern` (e.g. "Mon-Fri 9 AM - 5 PM, Sat-Sun closed")
- **Holiday and special hours**, plus temporary/permanent closure badges, when Google explicitly shows them
- Fields with no data (e.g. `holidayHours` when there are none) are never included in the output

## Output per place

Each place produces one dataset record:

```json
{
  "businessName": "Starbucks",
  "placeId": "ChIJfbedxbZZwokRpbnR5BBdz1c",
  "featureId": "0x89c259b6c59db77d:0x57cf5d10e4d1b9a5",
  "placeUrl": "https://www.google.com/maps/place/Starbucks/@40.7127281,-74.0060152,17z/",
  "location": { "lat": 40.7127281, "lng": -74.0060152 },
  "isOpenNow": true,
  "openingHoursText": "Open · Closes 9 PM",
  "hoursStatus": "Open - Closes 9 PM",
  "hours": {
    "monday":    { "open": "5:30 AM", "close": "9:00 PM" },
    "tuesday":   { "open": "5:30 AM", "close": "9:00 PM" },
    "wednesday": { "open": "5:30 AM", "close": "9:00 PM" },
    "thursday":  { "open": "5:30 AM", "close": "9:00 PM" },
    "friday":    { "open": "5:30 AM", "close": "9:00 PM" },
    "saturday":  { "open": "6:00 AM", "close": "8:00 PM" },
    "sunday":    { "closed": true }
  },
  "weeklyPattern": "Mon-Fri 5:30 AM - 9:00 PM, Sat 6:00 AM - 8:00 PM, Sun closed",
  "totalHoursPerWeek": 91.5,
  "holidayHours": [
    { "day": "friday", "label": "Juneteenth", "note": "Hours might differ", "open": "5:30 AM", "close": "8:00 PM" }
  ],
  "timezone": "America/New_York",
  "currentLocalTime": "2026-06-17T14:23:11-04:00",
  "nextOpenTime": "2026-06-18T05:30:00-04:00",
  "nextCloseTime": "2026-06-17T21:00:00-04:00",
  "timeUntilNextChange": "Closes in 6 hours",
  "scrapedAt": "2026-06-17T18:23:11+00:00"
}
```

Field reference:

- `businessName` — place name shown by Google Maps
- `placeId` — canonical Google Place ID (`ChIJ…`), when exposed
- `featureId` — legacy Maps feature/CID identifier (`0x…:0x…`), when exposed instead of or alongside a `placeId`
- `placeUrl` — the Maps URL supplied for this place
- `sourceUrl` — source place URL used for the record
- `location` — place centroid as latitude/longitude, when available
- `isOpenNow` — observed live open/closed state, when available
- `temporarilyClosed` — `true` when Google explicitly shows this badge
- `permanentlyClosed` — `true` when Google explicitly shows this badge
- `is24Hours` — whether the parsed weekly schedule is open 24 hours
- `openingHoursText` — opening-hours status text shown by Google Maps
- `hoursStatus` — cleaned inline open/closed status text
- `hours` — parsed schedule by weekday; see format below
- `weeklyPattern` — concise summary derived from the parsed weekly schedule
- `totalHoursPerWeek` — total scheduled hours, when the schedule supports the calculation
- `holidayHours` — parsed holiday-specific hours, when shown
- `specialHours` — the same holiday/special-hours entries under an alternate name
- `timezone` — place's IANA time-zone identifier, when available
- `currentLocalTime` — observation time in the place's local time zone
- `nextOpenTime` — next scheduled opening as a local-time ISO timestamp, when calculable
- `nextCloseTime` — next scheduled closing as a local-time ISO timestamp, when calculable
- `timeUntilNextChange` — human-readable time until the next opening-hours change
- `hoursSurfaceRendered` — whether the opening-hours surface was observed on the page
- `recordType` — record label (`businessHours`)
- `scrapedAt` — UTC timestamp when the record was collected

Each day in `hours` is one of:

| Format | Meaning |
|---|---|
| `{"open": "9:00 AM", "close": "5:00 PM"}` | Regular open hours |
| `{"closed": true}` | The place is closed that day |
| `{"open": "12:00 AM", "close": "11:59 PM", "is24Hours": true}` | Open 24 hours |
| `{"raw": "..."}` | Unusual format — raw source text preserved |

Closure flags are only set when Google positively displays the matching badge; an omitted flag does not assert that the place is open or operating.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `placeUrl` | string | – | Google Maps URL of a single place |
| `placeUrls` | array | – | Scrape multiple places in one run |
| `placeIds` | array | – | Canonical Google Place IDs beginning with `ChIJ`, resolved through Google's public Maps URL lookup |
| `language` | string | `en` | Interface language for labels and localized hours (`en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, `ru`) |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify proxy settings |

### Example: single place

```json
{
  "placeUrl": "https://www.google.com/maps/place/Starbucks/@40.7127281,-74.0060152,17z/"
}
```

### Example: batch of places

```json
{
  "placeUrls": [
    "https://www.google.com/maps/place/Starbucks/@40.7127281,-74.0060152,17z/",
    "https://www.google.com/maps/place/McDonald%27s/@40.7580,-73.9855,17z/"
  ],
  "proxyConfiguration": { "useApifyProxy": true }
}
```

### Example: canonical Place IDs

```json
{
  "placeIds": ["ChIJabtoT7hlwokRLECK0SW3_8I", "ChIJd3dF_VBZwokRAAFm3WMspbk"]
}
```

### Example: localized hours

```json
{
  "placeUrl": "https://www.google.com/maps/place/McDonald%27s/@40.7580,-73.9855,17z/",
  "language": "es"
}
```

## Use cases

- **Business-hours databases** — build and maintain a dataset of hours across many locations
- **Franchise monitoring** — track schedule changes across chain locations over time
- **Local-SEO dashboards** — keep displayed hours accurate and up to date
- **Booking and notification tools** — feed live hours into CRMs or reminder systems
- **"Open now" widgets** — power a site or app feature that needs the current open/closed state

## FAQ

**Which Google Maps URLs are supported?** Place URLs from desktop or mobile, country-specific Google domains, and supported short links. Copied wrapping quotes, trailing commas, fragments, and tracking parameters are cleaned up automatically.

**Can I scrape hundreds of places?** Yes — use the `placeUrls` array. For very large batches, consider splitting the list across separate runs.

**Why might `hours` be missing for a place?** Google Maps occasionally renders hours behind an extra interaction or varies its page structure by location. When hours can't be extracted, `hours` is omitted from the record, though `openingHoursText` may still be present.

**How are localized hours handled?** The selected `language` is sent through to the Maps page. Day names, closed/24-hour labels, and time formats are normalized when the source exposes them; unrecognized text is kept under `raw`.

**Is a proxy required?** No login or cookies are required to run the actor. The `proxyConfiguration` input is on by default; set `{"useApifyProxy": false}` only if you want a direct-only run for testing.

**Does this use the Google Maps API?** No — it reads the same public Google Maps page you'd see in your browser. No API key or login is needed.

**What is `placeId` versus `featureId`?** `placeId` is the canonical Google Place ID (`ChIJ…`), matching the `place_id` the Google Maps Places API returns. Some places only expose a legacy `0x…:0x…` token instead; that's recorded separately as `featureId` and is never relabelled as a Place ID.

**Is this affiliated with Google?** No — this is an independent actor reading Google Maps' public pages; it is not operated by or affiliated with Google.
