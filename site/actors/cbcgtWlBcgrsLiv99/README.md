# Google Maps Business Finder with Hours, Ratings & Location

Find businesses on **Google Maps** from a search phrase, a Maps place URL, or a canonical Place ID. The actor returns one clean row per business with its category, address, phone, website, rating, review count, weekly hours, price information where Google publishes it, attributes, photos and coordinates. It runs without a login, an API key, or proxy setup.

## What this actor does

- **Search, URL and Place ID modes:** search by keyword, paste a list of Maps place URLs, or look up `ChIJ` Place IDs. The two direct modes accept a mixed batch.
- **Batch coverage:** run several search terms across several locations in one run, with a separate result allowance for each term × location pair.
- **Complete business rows:** name, category, full address, phone, website, rating, review count, weekly hours, open/closed status, coordinates, Place ID and Maps URL.
- **Optional place details:** add Plus Code, price range, hotel stars and nightly price, fuel prices, accessibility and amenity attributes, photo count, photos and the rating histogram.
- **Useful filters:** rating range, review-count range, open now, must have a website or phone, name contains, exclude categories, and optionally skip permanently closed places.
- **74 languages:** names, categories, addresses and hours follow the language you select.
- **No browser, no login, no API key.** Empty fields are omitted instead of returned as placeholders.

## Output per business

Each row can include:

| Field | Meaning |
|---|---|
| `recordType` | Always `business` |
| `searchTerm` | The search that found the business (search mode) |
| `rankInSearch` | Position in that search's ranked results (search mode) |
| `name` | Business name |
| `categoryName` | Primary category |
| `categories` | All categories Google lists |
| `address` | Public address |
| `phone` | Public phone number |
| `website` | Public website |
| `totalScore` | Star rating |
| `reviewsCount` | Number of reviews |
| `hours` | Weekly schedule with stable English weekday keys, e.g. `{"monday": "9 AM to 5 PM"}` |
| `hoursStatus` | Google's current status line, e.g. `Closed · Opens 9 AM` |
| `isOpenNow` | `true` or `false` when Google states the current state |
| `priceLevel` | Displayed price range for restaurants, e.g. `$20–30` |
| `priceOfferAmount` / `priceOfferCurrency` / `priceOfferDateRange` | Nightly hotel price Google shows, when available |
| `hotelStars` / `hotelStarRating` | Hotel star class as Google labels it, plus the numeric value |
| `hotelAmenities` | Named hotel amenities with availability |
| `checkInTime` / `checkOutTime` | Check-in and check-out times, when shown |
| `fuelPrices` | Fuel grade and price, when Google publishes it |
| `plusCode` | Plus Code |
| `description` | Editorial description, when Google provides one |
| `ratingHistogram` | Review counts by star |
| `photosSample` / `photoCount` | Visible photo URLs and the total photo count |
| `placeAttributes` | Attribute chips such as `LGBTQ+ friendly` or accessibility labels |
| `amenities` | Header amenity labels |
| `attributes` | Structured attribute groups with label and value |
| `serviceOptions` | Known service/accessibility flags as booleans |
| `bookingLinks` | Reservation or booking links Google shows |
| `hasStreetView` / `streetViewCount` | Street View availability |
| `permanentlyClosed` / `temporarilyClosed` | Set only when Google marks the place closed |
| `claimed` | `false` only when Google shows an "Own this business?" link |
| `location` | `{lat, lng}` object |
| `latitude` / `longitude` | The same coordinates as plain columns |
| `placeId` | Canonical Google Place ID (`ChIJ…`) |
| `featureId` | Legacy Maps feature ID (`0x…:0x…`) |
| `cid` | Decimal Maps customer ID derived from the feature ID |
| `kgmid` | Knowledge Graph identifier (`/g/…`) |
| `timezone` | IANA time zone |
| `countryCode` | Two-letter country code |
| `googleMapsUrl` / `sourceUrl` | Maps URL for the place |
| `scrapedAt` | UTC scrape timestamp |

Fields are conditional. A business without a website, phone, rating, hours, price or photos simply omits that field.

`popularTimesSummary`, `menuLink` and `locatedIn` are retained as legacy conditional fields for saved integrations; the browserless path does not populate them.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `placeUrls`, or `placeIds` |
| `searchQuery` | string | – | Search phrase such as `coffee shops in Brooklyn` (mode=search) |
| `location` | string | – | Location appended to each search term |
| `placeUrls` | array | – | Google Maps place URLs (direct modes) |
| `placeIds` | array | – | Canonical `ChIJ…` Place IDs (direct modes) |
| `maxResults` | integer | `20` | Places per search, or total in the direct modes (1–100,000) |
| `language` | string | `en` | One of 74 Google Maps interface languages |
| `includeDetails` | boolean | `true` | Read Plus Code, price, hotel/fuel, attributes, photos and histogram |
| `searchTerms` | array | `[]` | Batch search terms |
| `locations` | array | `[]` | Batch locations; each term is searched in each location |
| `minRating` | number | – | Keep ratings at or above this value |
| `maxRating` | number | – | Keep ratings at or below this value |
| `minReviews` | integer | – | Keep review counts at or above this value |
| `maxReviews` | integer | – | Keep review counts at or below this value |
| `openNow` | boolean | `false` | Keep places Google shows as open |
| `hasWebsite` | boolean | `false` | Keep places with a website |
| `hasPhone` | boolean | `false` | Keep places with a phone number |
| `nameFilter` | string | – | Keep names containing this text |
| `includeClosed` | boolean | `true` | When off, skip permanently closed places |
| `excludeCategories` | array | `[]` | Skip categories containing these words |

### Example: search a city

```json
{
  "mode": "search",
  "searchQuery": "coffee shops",
  "location": "Brooklyn, NY",
  "maxResults": 50,
  "includeDetails": true
}
```

### Example: several categories across several cities

```json
{
  "mode": "search",
  "searchTerms": ["coffee shops", "bakeries"],
  "locations": ["Austin, TX", "Dallas, TX"],
  "minRating": 4.0,
  "minReviews": 25,
  "hasWebsite": true,
  "maxResults": 100
}
```

### Example: read specific place URLs

```json
{
  "mode": "placeUrls",
  "placeUrls": [
    "https://www.google.com/maps/place/GRIND+THE+NYC+COFFEE+SHOP+%26+BAGEL+HOUSE/@40.7590839,-73.9918101,17z/data=!3m1!4b1!4m6!3m5!1s0x89c259125a931087:0x3d471d4575240620!8m2!3d40.7590839!4d-73.9918101"
  ],
  "maxResults": 1
}
```

### Example: read Place IDs

```json
{
  "mode": "placeIds",
  "placeIds": ["ChIJhxCTWhJZwokRIAYkdUUdRz0", "ChIJ-Y-kFABZwokRIpR1-ZpuRjQ"],
  "maxResults": 2
}
```

### Example: search in Japanese

```json
{
  "mode": "search",
  "searchQuery": "東京 レストラン",
  "language": "ja",
  "maxResults": 30
}
```

## Use cases

- Build local prospect lists with phone, website, rating and hours for a city or neighborhood.
- Compare ratings, review counts, price ranges and amenities across competitors.
- Refresh opening hours, ratings and contact details for a fixed list of branch URLs or Place IDs.
- Find hotels with star class, nightly price and check-in/check-out times.
- Compare fuel prices at stations in an area.
- Add coordinates, Place IDs and Plus Codes to an existing location database.

## FAQ

**What is the data source?** Public Google Maps pages and their public place data. No Google account, API key or login is used.

**Do I need to set up a proxy?** No. The actor tries a direct connection first and escalates to Apify's proxy pool only if Google repeatedly blocks requests. There is no proxy field to configure; just fill in the input and run it.

**How many results can one search return?** Google returns a ranked window of roughly 160–230 places for a typical city query; broad categories such as hotels can return more. That is the source's window, not an unlimited directory. Split a broad search by neighborhood or category for wider coverage.

**Why did I get fewer rows than `maxResults`?** The source window ended, filters removed rows, duplicate places were merged, or a place page could not be read. The run status says which of these happened.

**Why are some fields missing?** Google publishes different details for different businesses, categories and countries. A place without a published website, phone, price or hours omits that field; the actor never invents a value.

**Are prices always available?** No. Restaurant price ranges, hotel nightly prices and fuel prices appear only when Google publishes them for that place and date.

**Does this actor scrape reviews?** No. It returns the rating and review count, plus the rating histogram. Review text and reviewer profiles are outside this actor's scope.

**Can I search several cities or categories in one run?** Yes. Use `searchTerms` and `locations`; every term is searched in every location with its own `maxResults` allowance.

**What happens to duplicates?** Places are matched by Place ID, feature ID or Maps URL (or name plus address when none is available) across the whole run. A place found twice is saved once.

**Is this affiliated with Google?** No. It is an independent tool that reads public Google Maps pages. Use the data responsibly and follow applicable terms and laws.
