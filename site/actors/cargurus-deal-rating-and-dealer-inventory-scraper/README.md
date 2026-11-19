# CarGurus Deal Rating & Dealer Inventory Scraper

CarGurus rates every listing against its Instant Market Value (IMV) and labels it from Great Deal down to Overpriced. This actor collects that rating, the IMV, the numeric deal score, the asking price, mileage, specs, dealer contact details and photos — by make/model/ZIP search, by a single listing ID, or from a CarGurus search URL you paste in.

## Modes

- **Search** — build a query from make, model, ZIP, radius, condition, deal rating, body style, fuel, transmission, drivetrain, price, mileage, photo and price-drop filters, and walk every result page.
- **Single listing** — fetch one listing by its CarGurus ID (for example `464410035` from `/details/464410035`).
- **Paste a URL** — paste a CarGurus search-results URL (old `searchResults.action` links are accepted) or a listing URL.

## What you get per listing

| Field | Description |
| --- | --- |
| `listingId` | CarGurus listing ID. |
| `url` | Direct `cargurus.com/details/<id>` link. |
| `title` | Year, make, model and trim as shown by CarGurus. |
| `vin`, `stockNumber` | VIN and dealer stock number when published. |
| `year`, `make`, `model`, `trim`, `bodyType` | Vehicle identity. |
| `price`, `pricePlusFees` | Asking price and the advertised price including disclosed fees. |
| `expectedPrice` | CarGurus Instant Market Value (IMV) for this car. |
| `priceDifference` | Price minus IMV (positive means above market). |
| `dealRating`, `dealRatingCode`, `dealScore` | Human label, raw CarGurus code and numeric deal score. |
| `mileage`, `condition` | Odometer reading and New / Used / Certified Pre-Owned. |
| `exteriorColor`, `interiorColor` | Published colors. |
| `transmission`, `driveType`, `engine`, `fuelType` | Powertrain and engine details. |
| `mpgCity`, `mpgHighway`, `mpgCombined` | EPA-style fuel economy when published. |
| `doors`, `options`, `tags`, `safetyRating` | Body details, installed options, badges and NHTSA rating. |
| `dealerName`, `dealerId`, `dealerType` | Selling dealer identity. |
| `dealerCity`, `dealerState`, `dealerPostalCode`, `dealerPhone` | Dealer location and phone. |
| `dealerWebsite`, `dealerRating`, `dealerReviewCount` | Dealer site and review summary (single-listing mode). |
| `dealerStreet`, `dealerLatitude`, `dealerLongitude`, `dealerLogoUrl` | More dealer detail (single-listing mode). |
| `distanceMiles`, `daysOnMarket` | Distance from the search ZIP and days listed. |
| `photoCount`, `primaryPhotoUrl`, `images` | Photo count, main photo and every photo (single-listing mode). |
| `accidentCount`, `ownerCount`, `vehicleHistoryReport` | Vehicle-history summary (single-listing mode). |
| `siteName`, `scrapedAt`, `recordType` | Source and capture metadata. |

Fields the source does not publish for a listing are omitted from that record — never filled with `null` or a placeholder.

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | Select | `search` | `search`, `byListingId` or `byUrl`. |
| `make` | Select | – | Manufacturer (all makes CarGurus lists). |
| `model` | Text | – | Model name, e.g. `Camry`. Empty means every model. |
| `condition` | Select | Any | New, Used or Certified Pre-Owned. Certified searches need a make. |
| `sortBy` | Select | `best_deal` | Best deal/match, price, mileage, year or distance. |
| `zipCode` | Text | – | 5-digit US ZIP code. |
| `distanceMiles` | Select | 50 | Search radius: 10–500 miles. |
| `minPrice`, `maxPrice` | Integer | – | Price range in USD. |
| `maxMileage` | Integer | – | Odometer ceiling. |
| `dealRating` | Select | Any | Great Deal, Good Deal, Fair Deal, High Price, Overpriced, No Price Analysis or Uncertain. |
| `bodyType` | Select | Any | Coupe, Convertible, Hatchback, Minivan, Pickup, Sedan, SUV/Crossover, Van, Wagon. |
| `fuelType` | Select | Any | Gasoline, Hybrid, Electric, Diesel, Biodiesel or Flex Fuel. |
| `transmission` | Select | Any | Automatic or Manual. |
| `drivetrain` | Select | Any | AWD, FWD, RWD, 4WD or 4X2. |
| `onlyWithPhotos` | Boolean | false | Skip listings with no photos. |
| `onlyPriceDrops` | Boolean | false | Only recent price drops. |
| `listingId` | Text | – | Listing ID for `byListingId`. |
| `url` | Text | – | Search or listing URL for `byUrl`. |
| `maxItems` | Integer | 5 | Maximum records to emit (1–1000). |

### Examples

Used Toyota Camry near New York:

```json
{
  "mode": "search",
  "make": "Toyota",
  "model": "Camry",
  "condition": "Used",
  "zipCode": "10001",
  "distanceMiles": 50,
  "maxPrice": 30000,
  "maxItems": 50
}
```

Great-deal hybrid SUVs with all-wheel drive and photos:

```json
{
  "mode": "search",
  "make": "Toyota",
  "zipCode": "10001",
  "dealRating": "GREAT_PRICE",
  "bodyType": "SUV / Crossover",
  "fuelType": "Hybrid",
  "drivetrain": "All-Wheel Drive",
  "onlyWithPhotos": true,
  "maxItems": 25
}
```

One listing by ID:

```json
{"mode": "byListingId", "listingId": "464410035", "maxItems": 1}
```

A pasted search URL:

```json
{"mode": "byUrl", "url": "https://www.cargurus.com/Cars/l-Used-Toyota-Camry-Raleigh-d292_L25152", "maxItems": 100}
```

## Data source and reliability

CarGurus renders its search results from a public Remix loader payload. The actor reads that payload through Google's live translate mirror (no account or cookies) and rotates exits/languages, with an Apify unblocker fallback for DataDome-protected detail pages. You do not need to supply cookies, API keys or proxy settings; the actor selects its route automatically.

## Use cases

- **Car shoppers** compare deal ratings and IMV on a shortlist near a ZIP code.
- **Dealers** monitor competitors' inventory, pricing and days on market.
- **Pricing analysts** track how deal scores and price differences move by model and region.
- **Lead generation** collects dealer name, phone and location alongside active listings.
- **Fleet buyers** screen by mileage, body style, fuel and drivetrain.

## FAQ

**Does this need a CarGurus login?**
No. It reads publicly visible listing data and never asks for an account.

**What does `dealRating` mean?**
CarGurus compares a car's price with its Instant Market Value. Great Deal and Good Deal are below IMV, Fair Deal is close, High Price and Overpriced are above it. `dealRatingCode` keeps CarGurus's raw value.

**Why is a field missing on some records?**
Not every listing publishes a VIN, MPG, dealer rating or full photo set. Missing fields are omitted, not filled with a placeholder.

**Can I filter by model year?**
Not yet: CarGurus's current search loader ignores year-range parameters. Use `minPrice`/`maxPrice` and sort by model year, or paste a URL with the site's own year filter.

**Can I scrape certified pre-owned cars?**
Yes, with a make selected. CarGurus exposes certified inventory through its make pages; a make-less certified search is not available.

**How many results can I get?**
Up to 1,000 per run via `maxItems`. Large runs walk result pages until `maxItems` or the source ends. CarGurus shows an approximate national total that is not filter-specific, so result counts in the dataset are authoritative.

**Is this affiliated with CarGurus?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by CarGurus.

## Legacy inputs

Saved tasks that still send `zip`, `searchUrl`, `startUrl`, `startUrls`, `maxResults`, `carCondition`, `makeName`, `modelName`, `listingID` or `sort` keep working: the actor maps them in code to the current fields.
