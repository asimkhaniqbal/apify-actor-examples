# Google Maps Business Finder with Hours, Ratings & Location

Find business records from public Google Maps pages. Run a text search (or a batch of searches across several cities) to discover places, or hand over specific Google Maps place URLs or `ChIJ` Place IDs for direct extraction. Each business is written to the dataset as soon as it has been read.

## What this actor does

- Searches Google Maps for a query, optionally combined with one or more locations, or for a batch of search terms × locations in one run.
- Reads specific public place URLs or canonical Place IDs directly.
- Extracts business details such as name, category, address, phone, website, rating, review count, hours, coordinates, identifiers, photos and, for hotels and fuel stations, prices, when Google shows them.
- Applies optional filters (rating, review count, open now, website, phone, name, category, permanently closed) and skips geographic areas such as a city or state that a search can surface.
- Removes duplicate places across the whole run and omits fields Google doesn't show instead of returning placeholders.

## Output per record

Each business row can include:

| Field | Meaning |
|---|---|
| `recordType` | Always `business` |
| `searchTerm` | The search that found this business (search mode only), so batch runs stay traceable |
| `name` | Business name shown by Google Maps |
| `categoryName` | Category shown on the listing |
| `address` | Public address |
| `phone` | Public phone number, when shown |
| `website` | Public website, when shown |
| `totalScore` | Star rating, when rated |
| `reviewsCount` | Review count |
| `priceLevel` | Price band such as `$$` or `$10–20`, when shown |
| `plusCode` | Plus Code, when shown |
| `location` | `{lat, lng}` read from the place's Maps URL |
| `latitude` / `longitude` | The same coordinates as plain columns |
| `placeId` | Canonical Google Place ID (the `ChIJ...` token), when Google exposes one |
| `featureId` | Google Maps feature ID (the `0x...:0x...` token), when Google exposes one; kept separate from `placeId` |
| `kgmid` | Knowledge Graph identifier (the `/g/...` token in the Maps URL) |
| `locatedIn` | Parent place when Google shows "Located in:" — useful for shops inside malls, stations, and buildings |
| `placeAttributes` | Google's attribute chips exactly as shown, e.g. `LGBTQ+ friendly`, `Check-in time: 3:00 PM` |
| `hasStreetView` / `streetViewCount` | Set when Google shows a Street View control for the place |
| `fuelPrices` | Fuel grades and published prices for a fuel station, e.g. `[{"grade": "Regular", "price": "$4.79"}]`; `stale: true` marks a price Google flags as over 24 hours old |
| `hotelStars` / `hotelStarRating` | Hotel star class as Google labels it, plus the numeric value |
| `checkInTime` / `checkOutTime` | Published check-in and check-out times |
| `priceOfferAmount` / `priceOfferCurrency` | The nightly hotel price Google currently shows, and its currency |
| `priceOfferDateRange` | The dates that price applies to |
| `priceOfferFreeCancellation` | Free-cancellation note on that hotel offer, in Google's wording |
| `hotelAmenities` | Named hotel amenities with availability, e.g. `[{"name": "Pool", "available": true}]` |
| `description` | Editorial description shown on the place page |
| `hoursStatus` / `hours` | Opening-hours status line and weekly hours |
| `isOpenNow` | Whether Google shows the place as open at scrape time |
| `ratingHistogram` | Rating distribution by stars |
| `serviceOptions` | Object of service flags Google shows, e.g. `{"dineIn": true, "takeout": true}` |
| `amenities` | Header amenity labels in Google's own wording |
| `menuLink` | Menu link, when shown |
| `bookingLinks` | Booking, order or reservation links, when shown |
| `photosSample` | Image URLs visible on the place page |
| `photoCount` | Number of photos Google reports for the place |
| `popularTimesSummary` | Live busyness, as `{currentBusyness, currentBusynessText}` or just the text |
| `claimed` | `false` only when Google shows a "Claim this business" link |
| `permanentlyClosed` / `temporarilyClosed` | `true` only when Google marks the place closed |
| `googleMapsUrl` | Maps URL of the place |
| `sourceUrl` | Same as `googleMapsUrl` |
| `scrapedAt` | UTC scrape timestamp |

Fields are conditional. A business without a website, phone, rating, hours, or photos simply omits that field.

**Search results without a detail page:** in search mode the actor first reads each result card, then opens the place page. If the place page can't be opened in time, the business is still saved with only what the card showed (typically `name`, `totalScore`, `reviewsCount`, `categoryName`, `priceLevel`, `description`, `address` and `googleMapsUrl`). Phone, website, Place ID, coordinates, hours, photos and the other detail fields are missing from those rows.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `placeUrls`, or `placeIds`. The two direct modes read both `placeUrls` and `placeIds`, so you can mix them. |
| `searchQuery` | string | – | Search text such as `coffee shops in Brooklyn`. Used in `search` mode when `searchTerms` is empty. |
| `location` | string | – | Location added to each search as "term in location" (skipped when the term already contains it). Combined with `locations`. |
| `placeUrls` | array | – | Google Maps place URLs for the direct modes. Copied quotes, tracking parameters, country domains and short links are cleaned up; duplicates are removed; non-Maps URLs are rejected. |
| `placeIds` | array | – | Canonical Google Place IDs beginning with `ChIJ` for the direct modes. Feature IDs and CIDs are not accepted here; paste a Maps URL instead. |
| `maxResults` | integer | `20` | In `search` mode: the cap **per search** (each term × location combination gets its own allowance). In `placeUrls`/`placeIds` mode: how many of the supplied URLs/IDs are read in total, so raise it when you pass more than 20. 1–100,000. |
| `language` | string | `en` | Maps interface language: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh`, `ar` or `ru`. |
| `proxyConfiguration` | object | – | Optional proxy override. Leave empty to use the actor's default proxy; set `useApifyProxy` to `false` to connect without one. |
| `searchTerms` | array | `[]` | Several search terms in one run, e.g. `["coffee shops", "dentists"]`. When set, `searchQuery` is not used. Each term is searched in every location. |
| `locations` | array | `[]` | Several locations in one run, e.g. `["New York, NY", "Austin, TX"]`. |
| `minRating` | number | – | Keep businesses rated at or above this value (1–5). Unrated businesses are skipped. |
| `maxRating` | number | – | Keep businesses rated at or below this value. Unrated businesses are skipped. |
| `minReviews` | integer | – | Keep businesses with at least this many reviews. Businesses without a review count are skipped. |
| `maxReviews` | integer | – | Keep businesses with at most this many reviews. Businesses without a review count are skipped. |
| `openNow` | boolean | `false` | Keep only businesses Google shows as open at scrape time. |
| `hasWebsite` | boolean | `false` | Keep only businesses that publish a website. |
| `hasPhone` | boolean | `false` | Keep only businesses that publish a phone number. |
| `nameFilter` | string | – | Keep only businesses whose name contains this text (case-insensitive). |
| `includeClosed` | boolean | `true` | When off, businesses Google marks as permanently closed are skipped. |
| `excludeCategories` | array | `[]` | Skip businesses whose category contains any of these words. |

Filters apply in every mode. A card-only search row (see above) has no phone, website or open-now data, so it is dropped when `hasPhone`, `hasWebsite` or `openNow` is on.

### Example: search for local businesses

```json
{
  "mode": "search",
  "searchQuery": "coffee shops",
  "location": "Brooklyn, NY",
  "maxResults": 20,
  "language": "en"
}
```

### Example: many search terms across several cities, filtered

```json
{
  "mode": "search",
  "searchTerms": ["coffee shops", "bakeries"],
  "locations": ["Austin, TX", "Dallas, TX"],
  "minRating": 4.0,
  "minReviews": 25,
  "hasWebsite": true,
  "maxResults": 50
}
```
This runs all four term-and-city combinations, keeps only well-reviewed businesses that publish a website, and allows up to 50 per combination.

### Example: search in Japanese

```json
{
  "mode": "search",
  "searchQuery": "寿司",
  "location": "東京 日本",
  "maxResults": 10,
  "language": "ja"
}
```

### Example: scrape specific place URLs

```json
{
  "mode": "placeUrls",
  "placeUrls": [
    "https://www.google.com/maps/place/GRIND+THE+NYC+COFFEE+SHOP+%26+BAGEL+HOUSE/@40.7590839,-73.9918101,17z/data=!3m1!4m6!3m5!1s0x89c259125a931087:0x3d471d4575240620!8m2!3d40.7590839!4d-73.9918101!16s%2Fg%2F11kgnrjx7t"
  ],
  "maxResults": 1
}
```

### Example: scrape canonical Place IDs

```json
{
  "mode": "placeIds",
  "placeIds": ["ChIJabtoT7hlwokRLECK0SW3_8I"],
  "maxResults": 1
}
```

## Use cases

- Map every bakery or dentist in a set of cities before opening a new location.
- Build a prospect list of local businesses that have a phone number but no website.
- Refresh ratings, review counts and opening hours for a fixed list of branch URLs.
- Find hotels with their star class, check-in times and the nightly price Google shows.
- Compare fuel prices across the stations in an area.
- Add coordinates, Plus Codes and Place IDs to an existing location database.

## FAQ

**Are all fields always present?** No. Google publishes different fields for different businesses and regions, and card-only search rows carry fewer fields. Unavailable values are omitted.

**Does the actor invent ratings, review counts, or identifiers?** No. Values are only written when they appear on the place page, its result card or its Maps URL.

**Can I get more than about 120 results for one search?** You can request up to 100,000, but Google usually lists around 120 or fewer places for one search. For wider coverage, split a broad search into narrower ones (for example by neighborhood) using `searchTerms` and `locations`.

**Why did my place-URL run return fewer rows than URLs I supplied?** In `placeUrls`/`placeIds` mode `maxResults` caps how many of the supplied URLs/IDs are read (default 20). Duplicate URLs are also merged, filters can skip places, and a place page that won't load is left out rather than saved as an empty row.

**What happens with duplicates?** Places are matched by Place ID, feature ID or Maps URL (or name plus address when none of those are available) across the whole run. A place found by two searches is saved once, under the first search that found it.

**What happens if Google blocks or limits the page?** Already saved records are kept and the run status says whether Google failed to load the results or nothing matched your filters. The dataset is never padded to reach `maxResults`. Very large requests also stop when the run approaches its timeout.

**Do I need a Google account?** No. No login or cookies are needed.

**Is this affiliated with Google?** No. It's an independent tool that reads public Google Maps pages. Use the data responsibly and follow applicable terms and laws.
