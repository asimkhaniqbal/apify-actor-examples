# ImmoScout24 Property Detail & Price Breakdown Scraper

ImmoScout24 is Germany's largest property marketplace. This actor reads its public mobile listing feed without cookies, logins or a proxy setup and returns structured property records: the headline price, the rent breakdown where the advert publishes one, living space, rooms, energy data, amenities, address, photos and the advertiser's published contact details.

Give it a search URL, a direct listing URL, or just a city and a property type. It translates the filters in the URL — price, rooms, living space, equipment, energy class, construction year, published date, sort order and the filter slugs ImmoScout24 puts in the path — into the source query, follows the result pages and enriches each listing from its expose.

## Modes

| Mode | What it does |
|---|---|
| `auto` | Processes the URLs you supply, then also runs the location builder when a location is set |
| `urls` | Only pasted ImmoScout24 URLs: search pages, radius or map searches, and direct expose links |
| `search` | Only the location and filter fields below; no URL needed |

## Input

| Field | Default | Description |
|---|---|---|
| `mode` | `auto` | `auto`, `urls` or `search` |
| `startUrls` | `[]` | ImmoScout24 search URLs, radius/shape URLs or expose URLs. Every filter in the URL is carried over |
| `location` | `""` | City (`Berlin`, `München`, `Köln`), state/city path (`bayern/muenchen`) or geocode path (`/de/berlin/berlin`, `/at/wien/wien`) |
| `realEstateType` | `apartmentbuy` | Apartment, house, flat-share room, garage, investment, plot, temporary living, assisted living, compulsory auction, office, retail, gastronomy, trade site or special-purpose property |
| `priceMin` / `priceMax` | unset | Price range in EUR. For rentals it follows `priceType` |
| `priceType` | `calculatedtotalrent` | Warm rent (`calculatedtotalrent`) or cold rent (`rentpermonth`) for rent searches |
| `roomsMin` / `roomsMax` | unset | Room range, including half rooms |
| `livingSpaceMin` / `livingSpaceMax` | unset | Living area range in m² |
| `equipment` | `[]` | Balcony, built-in kitchen, cellar, garden, guest toilet, wheelchair access, lift, parking |
| `energyEfficiencyClasses` | `[]` | A+ through H |
| `newBuildingOnly` | `false` | New-build listings only |
| `keyword` | `""` | Free-text keyword such as `Altbau` or `möbliert` |
| `publishedAfter` | `""` | ISO date; newer listings only |
| `sorting` | `newest` | Newest, oldest, price or living space, ascending or descending |
| `includeDetails` | `true` | Fetch the full expose for each result (default on) |
| `maxItems` | `20` | Maximum listing records (1–10,000) |
| `maxPages` | `20` | Maximum result pages per search (1–200; 100 listings per page) |

Every filter is optional. The smallest useful run is one search URL with `maxItems` 2.

## Output

Each record is a `immoscout24_listing` with the fields the source actually publishes. Empty values are omitted rather than written as nulls.

**Identity and classification:** `id`, `url`, `title`, `realEstateType`, `propertyType`, `contractType`, `operation`, `publicationState`, `isPrivate`, `isProject`, `isNewObject`, `listingType`, `tags`, `publishedText`, `publishedAt`, `scrapedAt`.

**Price breakdown:** `price`, `priceCurrency`, `priceType`, `kaltmiete` (cold rent), `warmmiete` (warm rent), `totalRent`, `nebenkosten` (utilities), `heizkosten` (heating), `heatingIncludedInUtilities`, `kaution` (deposit), `pricePerSqm`, `marketValue` (auction), `originalPrice`, `priceChangePercent`, `priceReduced`.

**Property data:** `livingSpace`, `rooms`, `bedrooms`, `bathrooms`, `plotArea`, `usableArea`, `totalFloorArea` (also commercial floor area), `floor`, `floorCount`, `yearBuilt`, `renovationYear`, `condition`, `heatingType`, `parkingSpaces`, `parkingType`, `availableFrom`, `commission`, `objectNumber`, `listedBuilding`.

**Energy certificate:** `energyCertificate` (class), `energyCertificateType`, `energyCertificateAvailable`, `energyCarriers`, `energyDemand`.

**Amenities and text:** `hasBalcony`, `hasGarden`, `hasCellar`, `hasElevator`, `kitchenIncluded`, `guestToilet`, `barrierFree`, `petsAllowed`, `amenities[]`, `features[]`, `descriptionText`, `furnishingText`, `locationText`, `otherText`.

**Location and media:** `address`, `street`, `postcode`, `city`, `district`, `latitude`, `longitude`, `images[]`, `videos[]`.

**Advertiser:** `realtor`, `realtorName`, `realtorCompany`, `realtorPhone`, `realtorPhones[]`, `realtorUrl`, `realtorRating`, `realtorRatingCount`.

When a run finds nothing, it emits one `immoscout24_blocked` record with a `reason` and a message so the run still finishes successfully.

### Price fields explained

- `kaltmiete` is the cold rent before utilities.
- `nebenkosten` is the monthly service charge; `heizkosten` is separated only when the advert does.
- `warmmiete` is the warm rent when stated. `totalRent` also covers package rent and monthly rent for temporary living or garages.
- `price` is the headline figure: purchase price for sales, warm rent (or the available rent figure) for rentals, minimum bid for compulsory auctions.
- `pricePerSqm` is the published or source-calculated price per square metre.

## Examples

Berlin apartments for sale, first two results:

```json
{
  "mode": "auto",
  "startUrls": ["https://www.immobilienscout24.de/Suche/de/berlin/berlin/wohnung-kaufen"],
  "maxItems": 2
}
```

Munich rental search with filters:

```json
{
  "mode": "search",
  "location": "bayern/muenchen",
  "realEstateType": "apartmentrent",
  "priceMin": 800,
  "priceMax": 1800,
  "roomsMin": 2,
  "livingSpaceMin": 55,
  "equipment": ["balcony"],
  "sorting": "newest",
  "maxItems": 50
}
```

A single expose:

```json
{
  "mode": "urls",
  "startUrls": ["https://www.immobilienscout24.de/expose/172626075"],
  "maxItems": 1
}
```

Austrian rentals:

```json
{
  "mode": "search",
  "location": "/at/wien/wien",
  "realEstateType": "apartmentrent",
  "maxItems": 20
}
```

## Data source and reliability

The actor calls the public JSON endpoints used by ImmoScout24's own mobile apps. The website itself (including its Next.js payload and sitemaps) is protected by an AWS WAF challenge, but the mobile listing feed answers directly. No cookie, API key or account is needed. The actor tries a direct connection first and only falls back to Apify Proxy automatically if ImmoScout24 rejects the request, so you never have to configure a proxy.

Search pages are read in order, duplicates are removed by expose ID, and the run stops at `maxItems`, `maxPages`, the end of the result list, or two pages without new listings.

## FAQ

**Do I need cookies or an API key?**
No. The listing feed is public and the actor never asks for credentials.

**Why does the actor sometimes skip the street?**
ImmoScout24 hides the exact street for many adverts until you contact the advertiser. The record then contains the postcode, city and district, and omits `street` instead of inventing it.

**Why do some records lack `kaltmiete` or `warmmiete`?**
Only the figures the advertiser publishes are returned. Some adverts show a single headline rent, some show cold rent plus utilities, and some show the full warm rent. Sale listings normally have none of the rent fields.

**Why is `realtorPhone` missing on some listings?**
Phone numbers are included only when the expose publishes them. Private advertisers often hide the number behind ImmoScout24's contact form; the actor does not automate that form.

**Can I search Austria?**
Yes. Use an Austrian geocode such as `/at/wien/wien` or paste an immobilienscout24.at regional search URL. Switzerland uses a different platform and is not part of this feed.

**Can I scrape offices, shops and gastronomy?**
Yes. Choose the matching `realEstateType` (`office`, `store`, `gastronomy`, `tradesite`, `specialpurpose`) or paste a commercial search URL such as `/Suche/de/berlin/berlin/buero-mieten`. The commercial area is returned as `totalFloorArea`; the living-space fields stay empty because those listings do not have a Wohnfläche. ImmoScout24's feed does not accept the price range or tradesite energy filter for commercial types, so those two filters apply to residential searches only.

**How many results can I get?**
The feed paginates well beyond the website's visible result window (100 listings per page). For a very large query, raise `maxPages`; if the source still stops, split the search by city or price range.

**Is this affiliated with ImmoScout24?**
No. This is an independent third-party actor that reads publicly accessible listing data. It is not affiliated with or endorsed by Immobilien Scout GmbH.
