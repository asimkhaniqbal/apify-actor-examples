# AutoScout24 Vehicle Listings with Dealer Contact Details

Collect vehicle listings from AutoScout24 with prices, mileage, first registration, power, fuel, transmission, body and colour, equipment, high-resolution photos and the seller's public contact details. Search by filters, paste search URLs, scrape a dealer's inventory, or open known listing IDs.

## What this actor does

- Searches eight AutoScout24 country sites and the English `autoscout24.com` marketplace.
- Supports free-text search plus make, model, vehicle type, price, year, mileage, power, fuel, transmission, seller type, body, colour, drivetrain, postal code/radius, previous owners and listing age.
- Walks the result pages up to the source cap and de-duplicates listings by their AutoScout24 ID.
- Opens each listing's detail page on request for the full specification sheet, equipment list, description, price evaluation and 1280x960 photos.
- Adds public dealer contact details from the dealer's profile: phone, email, website, address, opening hours, rating, review count, contact persons and VAT/register details when the dealer publishes them.
- Scrapes a dealer's live inventory for the selected vehicle type from a dealer profile URL.
- Omits fields the source does not publish; every record contains only real, non-empty values.

## Markets

| Value | Site |
|---|---|
| `de` | autoscout24.de |
| `at` | autoscout24.at |
| `be-nl` / `be-fr` | autoscout24.be (Dutch / French) |
| `es` | autoscout24.es |
| `fr` | autoscout24.fr |
| `it` | autoscout24.it |
| `lu` | autoscout24.lu |
| `nl` | autoscout24.nl |
| `com` | autoscout24.com (English pan-European feed) |
| `all` | Runs the eight local country sites (`de`, `at`, `be-nl`, `es`, `fr`, `it`, `lu`, `nl`) and merges by listing ID; `be-fr` and `com` stay separate |

## Output per listing

| Group | Fields |
|---|---|
| Listing | `platform`, `listingId`, `url`, `title`, `subtitle`, `description`, `descriptionHtml`, `status`, `createdAt`, `lastModifiedAt` |
| Price | `price.amount`, `price.currency`, `price.formatted`, `price.raw`, `price.net`, `price.negotiable`, `price.onRequestOnly`, `price.taxDeductible`, `price.vatRate`, `price.isFinalPrice`, `price.evaluation`, `price.median`, `price.previousAmount` |
| Vehicle | `vehicle.make`, `model`, `modelGroup`, `modelVersion`, `motorType`, `variant`, `condition`, `offerType`, `vehicleType`, `bodyType`, `year`, `month`, `firstRegistrationDate`, `mileage`, `mileageUnit`, `vin`, `hsnTsn`, `previousOwners`, `accidentFree`, `hasFullServiceHistory`, `nonSmoking`, `newInspection`, `warranty`, `doors`, `seats`, `color`, `paintType`, `upholstery`, `upholsteryColor`, `cylinders`, `gears`, `driveTrain`, `displacementCcm`, `weightKg`, `towingCapacity` |
| Specifications | `specifications.powerKw`, `powerHp`, `power`, `fuelType`, `fuelCode`, `transmission`, `transmissionCode`, `driveType`, `consumptionL100km`, `consumption`, `co2Emissions`, `emission`, `electricRangeKm`, `batteryCapacityKwh` |
| Equipment | `features` (flat list), `equipment` (grouped by comfort, entertainment, safety and extras) |
| Seller | `seller.type`, `isDealer`, `name`, `contact`, `address`, `city`, `zip`, `country`, `phone`, `phoneE164`, `phones`, `emails`, `website`, `dealerUrl`, `dealerId`, `rating`, `ratingCount`, `reviewCount`, `recommendPercentage`, `openingHours`, `contactPersons`, `additionalServices`, `about`, `imprint`, `vatId`, `registerNumber`, `logo` |
| Location | `location.country`, `city`, `zip`, `street`, `region`, `latitude`, `longitude`, `coordinates` |
| Media | `images`, `thumbnail`, `imageCount`, `has360`, `videoUrl`, `whatsappNumber` |
| Meta | `meta.scrapedAt`, `meta.country`, `meta.marketplace`, `meta.language`, `meta.source`, `meta.crawlId`, `meta.promoted`, `meta.searchPosition`, `meta.deliveryAvailable` |

Fields are nested for readability. Empty values are omitted rather than emitted as `null`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search`, `details`, `dealer` or `byUrl` |
| `searchQuery` | text | – | Free-text search |
| `make` / `model` | text | – | AutoScout24 make and model slugs, e.g. `bmw`, `3er` |
| `vehicleType` | select | `car` | Cars, motorcycles, caravans or transporters |
| `country` | select | `de` | One market, `com`, or `all` |
| `priceMin` / `priceMax` | number | – | Price range in EUR |
| `yearMin` / `yearMax` | number | – | First-registration year range |
| `mileageMin` / `mileageMax` | number | – | Mileage range in km |
| `powerMin` / `powerMax` | number | – | Power range in kW |
| `fuelType` | select | – | Petrol, diesel, electric, plug-in hybrids, CNG, LPG, ethanol, hydrogen, other |
| `transmission` | select | – | Manual, automatic or semi-automatic |
| `sellerType` | select | – | Dealer or private seller |
| `bodyType` | select | – | Small car, convertible, coupe, SUV, estate, sedan, van, transporter, other |
| `color` | select | – | Exterior colour |
| `driveTrain` | select | – | All-wheel, front-wheel or rear-wheel drive |
| `zipCode` / `radius` | text / select | – | Search around a postal code |
| `sortBy` | select | `standard` | Price, mileage, registration year, power, newest or distance |
| `onlineSinceDays` | number | `0` | Only listings published within N days; 0 = any |
| `previousOwnersMax` | select | – | 1, 2, 3 or 4+ previous owners |
| `damagedListings` | select | `exclude` | Exclude, include or only damaged vehicles |
| `includeDetails` | boolean | `true` | Open every listing's detail page |
| `includeDealerContact` | boolean | `true` | Add the dealer's public profile details |
| `maxResults` | number | `100` | Row cap; 0 means the 4,000-row source cap for the run (shared across markets and de-duplicated) |
| `startUrls` | array | – | Search URLs (`byUrl`), listing URLs (`details`) or dealer URLs (`dealer`) |
| `listingIds` | array | – | Listing UUIDs or listing URLs (`details`). UUIDs use the selected `country`; full URLs select their own market. |

## Examples

Search in Germany:

```json
{
  "mode": "search",
  "country": "de",
  "make": "bmw",
  "model": "3er",
  "priceMax": 30000,
  "yearMin": 2019,
  "fuelType": "diesel",
  "transmission": "automatic",
  "maxResults": 50
}
```

Search all eight country sites:

```json
{
  "mode": "search",
  "country": "all",
  "make": "volkswagen",
  "model": "golf",
  "maxResults": 200
}
```

Open listing IDs:

```json
{
  "mode": "details",
  "country": "de",
  "listingIds": ["dc39eaab-7f63-4b3b-b732-ae82f17ef805"],
  "includeDealerContact": true
}
```

Scrape a dealer's inventory:

```json
{
  "mode": "dealer",
  "startUrls": [{"url": "https://www.autoscout24.de/haendler/autohaus-next-car"}],
  "maxResults": 100
}
```

Use a saved search URL:

```json
{
  "mode": "byUrl",
  "startUrls": [{"url": "https://www.autoscout24.it/lst/bmw?atype=C&fuel=D&sort=price&desc=0"}],
  "maxResults": 100
}
```

## Dealer contact coverage

Dealer phone numbers are present on the listing itself and are collected for every dealer listing. Email, website, opening hours, rating and VAT/register details come from the dealer's public profile and are included when the dealer publishes them; they are never guessed. Private sellers usually do not publish a company name or email.

## Limits

- AutoScout24 returns at most 4,000 listings for one search query. The actor de-duplicates repeated listings across pages, so a cap-level query can return slightly fewer than 4,000 unique rows. Narrow the filters to collect a larger slice of the market.
- Switzerland (`autoscout24.ch`) runs on a separate API and is not included; the eight country sites and `autoscout24.com` are covered.
- Some specification fields (consumption, CO2, warranty, net price) are only shown when the seller provides them.
- Buy-online (`smyle`) listings opened directly by URL or ID return the smyle specification sheet; seller, location and the full photo set are merged when the same listing is reached through search results.
- The actor is stateless: there is no only-new mode. Keep the `listingId` values from a previous run and compare them in your own pipeline.
- `autoscout24.com` is an English pan-European feed and overlaps the country sites; use one or the other when de-duplicating yourself.

## FAQ

**Do I need an account, login or API key?**
No. The actor reads public pages and returns data without any credentials.

**Which markets are covered?**
Germany, Austria, Belgium (Dutch and French), Spain, France, Italy, Luxembourg, the Netherlands and the English `autoscout24.com` feed.

**What is the difference between `includeDetails` on and off?**
With it on, every listing's detail page is opened for specifications, equipment, description, price evaluation and high-resolution photos. With it off, the run is faster and returns the fields shown on the search results card.

**How do I scrape a dealer's inventory?**
Use `mode: "dealer"` and paste the dealer profile URL in `startUrls`. The run returns that dealer's stock for the selected vehicle type (cars by default); run again with another vehicle type if the dealer also lists transporters, motorcycles or caravans.

**Can I use listing IDs instead of URLs?**
Yes. Put the AutoScout24 UUIDs in `listingIds` with `mode: "details"`.

**Why are some dealer fields missing?**
Email, website, opening hours and VAT details are optional on AutoScout24. They are included only when the dealer publishes them.

**Why do I get fewer rows than `maxResults`?**
The source caps each search query at 4,000 listings, some combinations of filters have fewer results, and repeated listings are de-duplicated. The run status reports the reason.

**Is this affiliated with AutoScout24?**
No. This actor reads public pages and is not affiliated with, endorsed by or sponsored by AutoScout24.
