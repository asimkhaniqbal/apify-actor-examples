# Google Maps Address & Coordinate Geocoding Lookup

This actor converts street addresses into latitude/longitude coordinates (forward geocoding), or coordinates back into a human-readable address (reverse geocoding), using Google Maps. It supports batch input, multiple languages, and country bias for ambiguous queries, with no login, cookies, or API key required.

## What this actor does

- **Forward geocoding** — given an address, returns the resolved coordinates, place ID, structured address components, plus code, and a stable Google Maps URL.
- **Reverse geocoding** — given a `{lat, lng}` pair, returns the nearest matched place name and address. When Google answers a coordinate with a dropped pin instead of a named place, the resolved record carries the locality Google prints for it (city / region / country) plus the exact coordinate — never an invented street address.

Only resolved Google Maps results are written to the dataset. Inputs that Google cannot resolve are reported in the actor log/status and do not create placeholder or error rows.

## Output per location

Each dataset record contains a subset of these fields (empty fields are omitted):

| Field | Type | Description |
|---|---|---|
| `recordType` | string | Actor row discriminator (`geocode`). |
| `mode` | string | `"forward"` or `"reverse"`. |
| `inputAddress` | string | Original address (forward mode only). |
| `inputLocation` | object | Original coordinates (reverse mode only). |
| `resolved` | boolean | Whether Google resolved the input to a place. |
| `name` | string | Place name (h1 on the place page); often blank for pure street addresses. |
| `formattedAddress` | string | Full address as Google formats it. |
| `location` | object | `{lat, lng}` of the resolved place. |
| `placeId` | string | Canonical Google Place ID beginning with `ChIJ`, when Google exposes one. |
| `featureId` | string | Legacy feature/CID token (`0x…:0x…`), when Google exposes one. |
| `plusCode` | string | Open Location Code if present. |
| `addressComponents` | object | Structured `{streetNumber, street, city, state, postalCode, country, countryCode}`. |
| `categoryName` | string | Google's category label when the resolved place is a business. |
| `googleMapsUrl` | string | Stable `/maps/place/` URL. |
| `sourceUrl` | string | Resolved Maps page URL used as the source context; omitted when unavailable. |
| `scrapedAt` | string | UTC ISO-8601 timestamp. |

### Example output

```json
{
  "recordType": "geocode",
  "mode": "forward",
  "inputAddress": "1600 Amphitheatre Parkway, Mountain View, CA",
  "resolved": true,
  "name": "Google Building 41",
  "formattedAddress": "1600 Amphitheatre Pkwy, Mountain View, CA 94043, USA",
  "location": {"lat": 37.4224864, "lng": -122.0855962},
  "featureId": "0x808fba02f1822489:0x37a9269fbae1c4f7",
  "plusCode": "CWC8+W4 Mountain View, California",
  "addressComponents": {
    "streetNumber": "1600",
    "street": "Amphitheatre Pkwy",
    "city": "Mountain View",
    "state": "CA",
    "postalCode": "94043",
    "country": "USA",
    "countryCode": "US"
  },
  "categoryName": "Corporate office",
  "googleMapsUrl": "https://www.google.com/maps/place/...",
  "sourceUrl": "https://www.google.com/maps/place/...",
  "scrapedAt": "2026-06-16T12:00:00+00:00"
}
```

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | enum (`forward` or `reverse`) | `forward` | Which direction to geocode. |
| `address` | string | – | A single address (forward mode). |
| `addresses` | array of strings | – | Batch of addresses (forward mode). |
| `location` | object `{lat, lng}` | – | A single coordinate (reverse mode). |
| `locations` | array of `{lat, lng}` | – | Batch of coordinates (reverse mode). |
| `language` | enum | `en` | Interface language (`en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar`, `ru`). |
| `country` | string | – | Optional ISO 3166-1 alpha-2 code (e.g. `US`, `GB`) used to bias results when an address is ambiguous. |
| `proxyConfiguration` | proxy | – | Optional. Only used automatically as a fallback for inputs that don't resolve on the first attempt. |

## Examples

### Example: forward geocode one address
```json
{
  "mode": "forward",
  "address": "1600 Amphitheatre Parkway, Mountain View, CA",
  "language": "en"
}
```

### Example: forward geocode a batch
```json
{
  "mode": "forward",
  "addresses": [
    "Eiffel Tower, Paris",
    "Big Ben, London",
    "Tokyo Tower, Tokyo"
  ],
  "language": "en"
}
```

### Example: reverse geocode one coordinate
```json
{
  "mode": "reverse",
  "location": {"lat": 40.7579, "lng": -73.9855}
}
```

### Example: reverse geocode a batch
```json
{
  "mode": "reverse",
  "locations": [
    {"lat": 48.8584, "lng": 2.2945},
    {"lat": 35.6586, "lng": 139.7454},
    {"lat": -33.8568, "lng": 151.2153}
  ]
}
```

## Use cases

- Convert a CSV of customer addresses to coordinates for mapping.
- Enrich event logs with city/state/country from raw GPS pings.
- Validate user-submitted addresses against Google's canonical form.
- Reverse-geocode IoT device locations or photo EXIF coordinates.
- Compare address-component parsing across multiple countries.

## FAQ

**Q: Do I need an API key?**
No. This actor works without an API key, Google Cloud project, or billing account.

**Q: How many results does it return per input?**
One record per input. If an address is ambiguous, Google picks the highest-confidence match (the same one you would see at the top of Google Maps' own search box).

**Q: How accurate are the coordinates?**
Coordinates are returned at full precision, often to 7 decimal places.

**Q: Should I use a proxy?**
A proxy is optional. Direct access is tried first, and a proxy is only used automatically as a fallback for an input that doesn't resolve on the first attempt.

**Q: What languages are supported?**
The `language` input controls the Google Maps interface language and the format of the returned address (e.g. street names, country names in the target language). Currently exposed: English, Spanish, French, German, Italian, Portuguese, Japanese, Korean, Chinese, Arabic, Russian.

**Q: Can I bias results to a specific country?**
Yes. Set `country` to an ISO 3166-1 alpha-2 code (e.g. `US`, `GB`, `FR`). The scraper appends the country name to ambiguous queries (e.g. `Big Ben` → `Big Ben, UK`).

**Q: What happens for unresolvable inputs?**
The actor logs each unresolved input and names it in the run status, and leaves the dataset unchanged for that item rather than writing a placeholder or error row. Google sometimes answers an address it can't place with an unrelated nearby result instead of a "no match" page, so a forward lookup is only marked resolved when the answer actually matches what you asked for; otherwise it's reported as unresolved.

**Q: Is this affiliated with Google?**
No. This is an independent, third-party actor that reads publicly accessible Google Maps results. You are responsible for using these results lawfully and in line with Google's terms.
