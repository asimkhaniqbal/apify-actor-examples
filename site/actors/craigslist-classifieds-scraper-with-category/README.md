# Craigslist Classifieds Scraper with Category Attributes

Scrape **Craigslist** across 700+ local sites without a login, cookie, or proxy setup. Search by keyword or browse a category, follow individual post links, and get the full listing: price, description, every photo, map coordinates, posted and repost data, plus the category-specific attributes Craigslist shows for jobs, housing, autos, and for-sale items.

## What this actor does

- **Four modes:** keyword search, category browse, specific URLs, and a region/subarea reference list
- **700+ regions:** every Craigslist hostname, with 62 common regions ready in a dropdown and a custom-region field for the rest
- **Filters that run on Craigslist:** price, posted today, photos, titles-only, owner/dealer, sort, condition, bedrooms/baths/square feet, vehicle make/year/mileage, postal radius, and duplicate grouping
- **Full detail records:** body text, gallery images, attributes, coordinates, display address, repost link, notices, and typed category fields
- **Empty fields are omitted** from every record; no `null` values or placeholder rows

## Output per listing

- `type` — always `craigslist_listing`
- `id`, `postingUuid`, `url`, `title`
- `price`, `priceValue`, `priceCurrency`, `priceQualifier`
- `postedAt`, `updatedAt`, `recordCreatedAt`, `scrapedAt`
- `region`, `location`, `neighborhood`, `subArea`, `area`, `areaAbbr`, `locationType`, `mapAddress`
- `latitude`, `longitude`
- `category`, `subcategory`, `section`, `categoryAbbr`, `subcategoryAbbr`, `postingType`
- `description`, `images[]`, `imageCount`, `attributes`, `notices[]`
- `phoneNumbers[]`, `emails[]`, `vin`, `replyUrl`
- `repostOf`, `hasContactInfo`, `seeMyOther`
- Jobs: `compensation`, `employmentType`, `jobTitle`, `companyName`, `experienceLevel`, `salary`
- Housing: `bedroomCount`, `bathroomCount`, `sqft`, `availableDate`, `housingType`, `laundry`, `parking`, `rentPeriod`, `listedBy`, `applicationFee`, `brokerFee`, `openHouseDates`, `catsOk`, `dogsOk`, `furnished`, `smoking`, `privateRoom`, `privateBath`, `wheelchairAccessible`, `airConditioning`, `evCharging`
- Autos: `yearManufactured`, `makeManufacturer`, `modelName`, `odometer`, `transmission`, `fuelType`, `titleStatus`, `cylinders`, `paintColor`, `drive`, `vehicleType`
- For sale: `condition`, `size`, `make`, `model`

With `includeDetails` off, rows contain only the search-card fields: identity, price, category, posted time, location, coordinates, and image URLs.

## Output per area (mode = `areas`)

- `type` — `craigslist_area`
- `areaId`, `abbreviation`, `hostname`, `description`, `shortDescription`
- `region`, `country`, `latitude`, `longitude`, `timezone`
- `subareas[]` — each with `abbreviation`, `description`, and `subAreaId`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `category` / `byUrl` / `areas` |
| `region` | string | `sfbay` | Common Craigslist region (mode=search/category) |
| `customRegion` | string | – | Any hostname or abbreviation; overrides `region` |
| `subarea` | string | – | Optional subarea such as `sfc`, `eby`, `brk` |
| `category` | string | `sss` | Craigslist category; `sss` is all for-sale listings |
| `searchQueries` | array | – | One or more keywords (mode=search); leave empty to browse a category |
| `startUrls` | array | – | Search pages or post URLs (mode=byUrl) |
| `minPrice` | integer | – | Minimum price |
| `maxPrice` | integer | – | Maximum price |
| `postedToday` | boolean | `false` | Only posts from the last 24 hours (Craigslist's posted-today filter) |
| `hasImage` | boolean | `false` | Only posts with a photo |
| `titlesOnly` | boolean | `false` | Match keywords against titles only |
| `sellerType` | string | `all` | `all` / `owner` / `dealer` |
| `sortBy` | string | `newest` | `newest` / `oldest` / `relevance` / `priceAsc` / `priceDesc` / `distance` |
| `condition` | string | `any` | `any` / `new` / `like-new` / `excellent` / `good` / `fair` / `salvage` |
| `minBedrooms` | integer | – | Housing only |
| `maxBedrooms` | integer | – | Housing only |
| `minBathrooms` | integer | – | Housing only |
| `maxBathrooms` | integer | – | Housing only |
| `minSqft` | integer | – | Housing only |
| `maxSqft` | integer | – | Housing only |
| `autoMakeModel` | string | – | Auto categories only, e.g. `honda civic` |
| `minAutoYear` | integer | – | Auto categories only |
| `maxAutoYear` | integer | – | Auto categories only |
| `maxAutoMiles` | integer | – | Auto categories only |
| `postalCode` | string | – | Optional ZIP/postal code anchor |
| `searchRadiusMiles` | integer | – | Radius around `postalCode` |
| `hideDuplicates` | boolean | `false` | Group duplicate/reposted listings |
| `deliveryAvailable` | boolean | `false` | For-sale/auto only |
| `cryptoOk` | boolean | `false` | For-sale/auto only |
| `countryCode` | string | – | Two-letter country filter for `areas` mode |
| `maxItems` | integer | `100` | Hard cap across all queries and URLs (1–10000) |
| `includeDetails` | boolean | `true` | Fetch full descriptions, attributes, images, and coordinates |

## Examples

### Search a city for a keyword

```json
{
  "mode": "search",
  "region": "sfbay",
  "category": "sss",
  "searchQueries": ["bike"],
  "maxItems": 50,
  "includeDetails": true
}
```

### Browse apartments with housing filters

```json
{
  "mode": "category",
  "region": "chicago",
  "category": "apa",
  "minBedrooms": 2,
  "maxBedrooms": 2,
  "maxPrice": 3000,
  "sortBy": "priceAsc",
  "maxItems": 100
}
```

### Scrape specific posts

```json
{
  "mode": "byUrl",
  "startUrls": [
    { "url": "https://www.craigslist.org/view/d/richmond-195-iron-horse-maverick-21/scJ6mscDAryikpFRAS61aq" },
    { "url": "https://sfbay.craigslist.org/bik/7962037752.html" }
  ],
  "maxItems": 10
}
```

### List regions in one country

```json
{
  "mode": "areas",
  "countryCode": "CA",
  "maxItems": 100
}
```

## Use cases

- **Marketplace price tracking** — monitor asking prices for a product across cities and categories
- **Vehicle shopping** — pull year, make, model, mileage, transmission, and title status for matching cars
- **Rental research** — collect rent, bedrooms, bathrooms, square footage, laundry, parking, and pet policy
- **Job aggregation** — gather compensation, employment type, experience level, and company names
- **Lead sourcing** — filter by price, photos, seller type, and posted date to find fresh posts
- **Local market mapping** — join listing coordinates and neighborhoods to city/subarea reference data

## FAQ

**Do I need a Craigslist account, cookie, or proxy?**
No. The actor reads public Craigslist listings and connects automatically. There is no login or proxy field to fill in.

**How many results can one run return?**
Craigslist caps a single search at 10,000 results. To collect more, split by category, subarea, price range, or posted-today and combine the runs. The actor tells you in the run status when the cap is reached.

**Why are some fields missing from a row?**
Craigslist posts differ by category and by what the seller filled in. A job may have no price; a generic for-sale post may have no structured attributes; a seller may not publish a phone number. Empty fields are omitted rather than filled with `null` or an empty value.

**Can I get the poster's email address?**
Only when the seller typed an email into the post body. Craigslist's reply relay hides the real address behind its own form and sometimes a captcha; this actor does not bypass that.

**Does the actor support RSS or JSON feeds?**
No. Craigslist discontinued its RSS output, and its old `output=json` parameter no longer returns JSON. This actor uses Craigslist's current listing data and returns the same fields in its dataset.

**Can I paste an old Craigslist post link?**
Current `/view/d/...` links work directly. Older `region/category/post-id.html` links also work. Very old links that contain a `/d/` slug but a broad category are resolved by searching for the post; if Craigslist has removed it, the URL is skipped with a message.

**Does an old search URL's `s=` offset still work?**
No. Craigslist no longer offers an offset cursor for search results, so the actor ignores `s=` and starts from the first result. Use `maxItems`, a narrower price range, or a subarea to control how deep a run goes.

**Why did a run return no listings?**
The query may have no matches, the category may not exist in that region, the post may have been removed, or a filter may be too narrow. A zero-result run completes with a status message and no placeholder record.

**Is this affiliated with Craigslist?**
No. This is an independent actor that reads publicly available Craigslist pages and is not operated or endorsed by Craigslist.
