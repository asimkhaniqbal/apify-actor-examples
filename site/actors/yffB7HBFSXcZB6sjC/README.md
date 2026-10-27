# AutoScout24 Vehicle Listings with Dealer Contact Details

Search AutoScout24 across eight European country domains by make, model, price, mileage, fuel type and location radius, and pull back full vehicle specs, equipment lists, dealer contact details and photos for every matching listing.

## What this actor does

- Builds a search from your filters (make, model, price, year, mileage, fuel type, transmission, seller type, ZIP code and radius) or starts directly from search/listing URLs you provide
- Follows each listing to its detail page to collect specifications, equipment features, description and seller contact details
- Can also run against a list of individual listing URLs only, skipping the search step entirely
- Puts price, power, fuel consumption and emissions into the same nested fields on every country domain
- Groups output into clear sections — price, vehicle, specifications, seller, location and images — rather than one flat row; every record has the same structure

## Output per listing

| Field | Description |
|---|---|
| `platform` | Always `"autoscout24"` |
| `listingId` | Listing identifier taken from its URL |
| `url` | Listing page URL |
| `title` | Listing title as shown on AutoScout24 |
| `price.amount` / `price.currency` | Price and currency |
| `price.gross` / `price.net` | Gross and net price, when the dealer publishes both |
| `price.monthly` | Monthly financing rate, when offered |
| `price.negotiable` | Whether the price is marked negotiable |
| `price.raw` | The price text as originally shown |
| `vehicle.make` / `vehicle.model` / `vehicle.modelGroup` / `vehicle.variant` | Make, model, model group and trim/variant |
| `vehicle.condition` | New or used |
| `vehicle.year` / `vehicle.month` | First registration year and month |
| `vehicle.mileage` / `vehicle.mileageUnit` | Mileage and its unit (km) |
| `vehicle.vin` | Vehicle identification number, when shown |
| `vehicle.previousOwners` | Number of previous owners, when shown |
| `vehicle.accidentFree` | Whether the listing states the vehicle is accident-free |
| `specifications.powerKw` / `specifications.powerHp` / `specifications.power` | Engine power in kW and HP |
| `specifications.fuelType` / `specifications.transmission` / `specifications.driveType` | Fuel type, transmission and drive type |
| `specifications.consumptionL100km` / `specifications.consumption` | Published fuel consumption |
| `specifications.co2Emissions` / `specifications.emission` / `specifications.emissionClass` | CO2 emissions and emission class |
| `specifications.doors` / `specifications.seats` | Door and seat counts |
| `specifications.color` / `specifications.interiorColor` / `specifications.interiorMaterial` | Exterior color, interior color and interior material |
| `specifications.weight` / `specifications.towingCapacity` | Weight and towing capacity, when published |
| `features` | List of equipment/feature labels shown on the listing |
| `description` | Dealer's free-text description |
| `subtitle` | Listing subtitle, when present |
| `highlights` | Short highlight phrases AutoScout24 shows on the listing |
| `warranty` | Warranty text, when stated |
| `seller.type` / `seller.name` | Dealer or private seller, and seller/dealer name |
| `seller.contact` / `seller.phone` | Contact name and phone number, when shown |
| `seller.address` / `seller.city` / `seller.zip` | Seller address components |
| `seller.rating` / `seller.reviewCount` | Dealer rating and number of reviews, when AutoScout24 shows them |
| `seller.website` | Dealer's own website, when listed |
| `seller.dealerId` | Dealer's AutoScout24 ID |
| `seller.openingHours` | Published opening hours, when shown |
| `images` / `thumbnail` | Photo URLs, and the first one as a thumbnail |
| `videoUrl` | Listing video URL, when present |
| `location.country` / `location.city` / `location.zip` / `location.street` / `location.region` | Vehicle location components |
| `location.coordinates` | Geographic coordinates, when published |
| `meta.scrapedAt` | UTC timestamp when the record was scraped |
| `meta.country` | Country domain the listing was collected from |
| `meta.source` | Always `"autoscout24"` |
| `meta.crawlId` | Short run identifier, the same on every record from one run |
| `meta.promoted` / `meta.topListing` | Whether AutoScout24 marks the listing as promoted or a top listing |
| `meta.availabilityStatus` | Availability status text, when shown |

Every record carries all of the keys above. Values AutoScout24 doesn't publish for a listing, or that are only available from the detail page, come back as `null` or as an empty list/object rather than being left out. A few keys always have a value: `price.currency` falls back to `EUR`, `vehicle.mileageUnit` is always `km`, `location.country` falls back to the chosen country code, and `price.negotiable`, `meta.promoted` and `meta.topListing` are `false` unless the listing says otherwise. Running with detail pages enabled (the default) fills in the most fields.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | AutoScout24 URLs to start from. URLs containing `/offers/`, `/angebot/` or `/lst/` are opened as single listing pages; other URLs are read as search-result pages and paged through. When set, the search filters below are ignored. |
| `make` | string | – | Car manufacturer slug, e.g. "bmw", "audi", "mercedes-benz" |
| `model` | string | – | Car model slug, e.g. "3-series", "a4", "c-class" |
| `priceMin` / `priceMax` | integer | – | Price range filter |
| `yearMin` / `yearMax` | integer | – | First registration year range (1900–2030) |
| `mileageMin` / `mileageMax` | integer | – | Mileage range in kilometres |
| `fuelType` | string | – | petrol, diesel, electric, hybrid, hybrid-petrol, hybrid-diesel, lpg, cng, hydrogen, or ethanol |
| `gearType` | string | – | manual, automatic, or semi-automatic |
| `sellerType` | string | – | dealer or private |
| `country` | string | `de` | Country domain to search: de, it, fr, es, nl, be, at, or lu |
| `zipCode` | string | – | ZIP/postal code for a location-based search |
| `radius` | integer | – | Search radius in kilometres around `zipCode` (1–500) |
| `sortBy` | string | `standard` | Result order: standard (relevance), price_asc, price_desc, mileage_asc, mileage_desc, age_asc, age_desc |
| `maxResults` | integer | `100` | Rough cap on the run (0 = no cap). It limits how many pages the run opens in total (search pages plus listing pages) and how many listings are taken from each search page, so with detail pages on you get somewhat fewer records than this number. With `scrapeDetails` off it does not cap the total across pages — use `maxPages` for that. |
| `maxPages` | integer | `10` | Maximum number of search-result pages to read. 0 means no cap. |
| `maxConcurrency` | integer | `5` | How many pages can be processed in parallel (1–20) |
| `scrapeDetails` | boolean | `true` | Visit each listing's detail page for full specs, features and seller contact details |
| `detailPageOnly` | boolean | `false` | When true, only `startUrls` that are opened as single listing pages are scraped, and no search is run |
| `includePhoneNumber` | boolean | `false` | Click the seller's "show phone number" button on detail pages before reading them |
| `downloadImages` | boolean | `false` | Also save up to 20 images per listing in the run's key-value store (detail pages only) |
| `proxyConfiguration` | object | – | Optional Apify Proxy configuration |
| `headless` | boolean | `true` | Run the browser without a visible window |
| `debugMode` | boolean | `false` | Enable verbose logging and save debug screenshots for troubleshooting |
| `maxRetries` | integer | `3` | Maximum number of retries for a page that fails to load (0–10) |
| `requestTimeout` | integer | `60` | Seconds allowed for processing one page before it counts as failed (10–300) |

### Example: basic make/model search

```json
{
  "make": "bmw",
  "model": "3-series",
  "country": "de",
  "priceMax": 30000,
  "yearMin": 2019,
  "maxResults": 50
}
```

### Example: location-radius search with full filters

```json
{
  "make": "audi",
  "model": "a4",
  "country": "de",
  "priceMin": 15000,
  "priceMax": 40000,
  "fuelType": "diesel",
  "gearType": "automatic",
  "sellerType": "dealer",
  "zipCode": "80331",
  "radius": 100,
  "sortBy": "price_asc",
  "maxResults": 200
}
```

### Example: scrape specific listing URLs only

```json
{
  "startUrls": [
    {"url": "https://www.autoscout24.de/angebot/mercedes-benz-c-klasse-uuid-here"}
  ],
  "detailPageOnly": true
}
```

## Use cases

- **Vehicle shopping** — filter by budget, mileage and location radius to shortlist cars worth viewing
- **Dealer inventory tracking** — monitor a dealer's live stock and pricing over time
- **Pricing research** — compare asking prices for the same make/model/year across countries
- **Feature and equipment comparison** — review which trims and feature packages are available at a given price point
- **Lead building** — pull seller contact details for dealers actively listing a given make or model

## FAQ

**Which AutoScout24 markets are covered?**
Germany, Italy, France, Spain, the Netherlands, Belgium, Austria and Luxembourg, each on its own country domain.

**Do I need a proxy to run this?**
It's optional. Without `proxyConfiguration` the run connects directly; if AutoScout24 starts blocking pages, add an Apify Proxy configuration and run again.

**What's the difference between leaving `scrapeDetails` on and off?**
With it on (the default), every listing gets its own detail-page visit, so specifications, features, description and seller contact details are filled in. With it off, you get faster results limited mostly to what's shown on the search results page, such as title, price and basic vehicle info.

**Why are some specification fields empty?**
Not every dealer fills in every field on AutoScout24, and some fields are only available on the listing's detail page. Enable `scrapeDetails` (on by default) to capture as much as is published for each listing.

**Can I scrape a list of listings I already have instead of searching?**
Yes — put the listing URLs (the `/offers/...` or `/angebot/...` form) in `startUrls` and set `detailPageOnly` to true to fetch just those pages.

**Can I paste a search URL from my browser?**
Only with care. AutoScout24 search URLs usually contain `/lst/`, and those are opened as a single listing page rather than paged through as a search, which gives poor results. Use the filter inputs (make, model, price, ZIP code and so on) to run a search instead.

**How do I search near a specific location instead of a whole country?**
Set `zipCode` to a postal code and `radius` to a distance in kilometres; results are limited to listings within that radius.

**Is this affiliated with AutoScout24?**
No. This actor reads AutoScout24's public pages and is not affiliated with, endorsed by, or sponsored by AutoScout24.
