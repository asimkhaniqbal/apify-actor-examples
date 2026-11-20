# Leboncoin France Classified Ads Extractor

Leboncoin is France's largest classifieds marketplace: rentals and property sales, cars and motorbikes, jobs, phones, furniture, and almost everything else. This actor turns any public Leboncoin search into structured records you can filter, join, and chart. It needs no Leboncoin account, no cookies, and no proxy setup.

## What it does

- **Search URLs** — paste one or more `leboncoin.fr/recherche` links; filters already in the URL (location, price, category, sort, property or vehicle facets) are kept.
- **Search builder** — choose a category, keywords, locations, price range, seller type, ad type, and the property, vehicle, goods, or job filters you need.
- **Pagination** — reads Leboncoin's own 35-ad result pages until your `maxItems` is reached or the source runs out, and deduplicates across pages.
- **Rich records** — price, location and coordinates, seller details, photo gallery, publication dates, and every category-specific attribute (surface, rooms, DPE, mileage, fuel, gearbox, contract type, and more).
- **Legacy inputs** — saved tasks that still send `category`, `search`, `startUrls`, or `maxItems` keep working; old keys are mapped in code.

## Inputs

| Field | Purpose |
| --- | --- |
| `startUrls` | Public Leboncoin search URLs. Any filters in the URL are preserved. |
| `category` | Category ID when you are not pasting a URL (9 property sales, 10 rentals, 2 cars, 33 jobs, and so on). |
| `search` | Keywords, for example `peugeot 208` or `studio paris`. |
| `locations` | Places in Leboncoin's own syntax: `Paris`, `75015`, `d_75`, `r_12`, or a full city code. |
| `priceMin` / `priceMax` | Price band in euros. |
| `sort` | Relevance, newest, oldest, price low-to-high, or price high-to-low. |
| `ownerType` | All sellers, private individuals only, or professionals only. |
| `adType` | Offers, wanted ads, or both. |
| `shippable` | Only ads where the seller offers shipping. |
| `searchInTitle` | Match keywords in titles only. |
| `roomsMin` / `roomsMax` | Real-estate room count. |
| `surfaceMin` / `surfaceMax` | Real-estate living area in m². |
| `realEstateType` | House, apartment, land, parking, or other. |
| `mileageMin` / `mileageMax` | Vehicle odometer range in km. |
| `yearMin` / `yearMax` | Vehicle model-year range. |
| `condition` | New, refurbished, very good, good, fair, or for parts. |
| `contractType` | Fixed-term, permanent, temporary, freelance, apprenticeship, internship, or volunteering. |
| `maxItems` | Stop after this many ads (maximum 1,000). |

### Examples

Rentals in Paris under €2,000, newest first:

```json
{
  "startUrls": ["https://www.leboncoin.fr/recherche?category=10&locations=Paris&price=min-2000&sort=time&order=desc"],
  "maxItems": 100
}
```

Used Peugeot 208s with a mileage cap:

```json
{
  "category": 2,
  "search": "Peugeot 208",
  "priceMax": 12000,
  "mileageMax": 120000,
  "sort": "price_low",
  "maxItems": 50
}
```

Three-room apartments for sale in Lyon:

```json
{
  "category": 9,
  "locations": ["Lyon"],
  "roomsMin": 3,
  "roomsMax": 3,
  "realEstateType": ["2"],
  "maxItems": 100
}
```

## Output

Each row is one ad. Empty values are omitted, never sent as `null` or a sentinel.

| Field | Description |
| --- | --- |
| `id` | Leboncoin ad ID. |
| `url` | Direct link to the ad. |
| `subject` | Ad title. |
| `body` | Description text, truncated to 2,000 characters. |
| `categoryId` / `categoryName` | Category ID and label. |
| `adType` / `status` | Offer or demand; active, sold, or expired. |
| `price` / `priceCents` / `priceCurrency` | Asking price in euros and cents. |
| `publishedDate` / `expirationDate` / `indexDate` | Publication, expiry, and last-index timestamps. |
| `city` / `zipcode` / `cityLabel` | City, postal code, and full location label. |
| `department` / `departmentId` / `region` / `regionId` / `country` | Administrative location. |
| `latitude` / `longitude` | GPS coordinates. |
| `ownerName` / `ownerType` / `ownerId` / `ownerStoreId` / `ownerSiren` | Seller identity and seller type. |
| `hasPhone` | Whether the seller published a phone number. |
| `imageCount` / `thumbnail` / `imageUrls` | Gallery size, thumbnail, and full-size photos. |
| `attributes` | Generic category attributes as key/value pairs. |
| `attributesDetailed` | Every published attribute with key, label, value, and generic flag. |
| `isUrgent` / `isHighlighted` / `isBoosted` / `isGallery` / `isPhotosup` / `isSubToplist` | Ad promotion flags. |
| `favorites` | Favourite count when Leboncoin publishes it. |
| `position` / `pageNumber` / `searchUrl` | Where the ad appeared in your search. |
| `scrapedAt` | ISO 8601 scrape time. |

### Example record

```json
{
  "id": 3311246700,
  "url": "https://www.leboncoin.fr/ad/locations/3311246700",
  "subject": "Appartement 3 pièces 62 m²",
  "categoryId": "10",
  "categoryName": "Locations",
  "adType": "offer",
  "status": "active",
  "price": 1200,
  "priceCents": 120000,
  "priceCurrency": "EUR",
  "publishedDate": "2026-11-17 09:15:32",
  "city": "Paris",
  "zipcode": "75011",
  "department": "Paris",
  "departmentId": "75",
  "region": "Île-de-France",
  "regionId": "12",
  "country": "FR",
  "latitude": 48.8566,
  "longitude": 2.3522,
  "ownerType": "private",
  "hasPhone": false,
  "imageCount": 18,
  "imageUrls": ["https://img.leboncoin.fr/api/v1/lbcpb1/images/3d/8e/fb/3d8efb8ddf7925e72a532dd4616782b0bda7fd37.jpg?rule=ad-image"],
  "attributes": {"rooms": "3", "square": "62 m²"},
  "position": 1,
  "pageNumber": 1,
  "searchUrl": "https://www.leboncoin.fr/recherche?category=10",
  "scrapedAt": "2026-11-20T10:30:00+00:00"
}
```

## Use cases

- Lettings and estate agents tracking new listings and price cuts in a city.
- Car dealers and buyers comparing mileage, year, and price for one model.
- Job boards monitoring offer volume by contract type and department.
- Market researchers following resale prices for phones, furniture, or tools.
- Lead-generation teams separating professional sellers from private ones.

## FAQ

**Do I need a Leboncoin account, cookies, or a proxy?**
No. The actor reads the same public pages any visitor sees and picks a working network path on its own.

**What is the difference between `startUrls` and the builder fields?**
`startUrls` reproduces a search you built on leboncoin.fr, including every filter in the address bar. The builder fields create a search from scratch. When `startUrls` is set, the builder fields are ignored.

**Why is a field missing from some records?**
Leboncoin only publishes some fields for some ads, categories, or sellers. Missing values are omitted. Legacy numeric fields such as `price`, `latitude`, `longitude`, and `imageCount` keep `0` when the source has no value.

**How many ads are on a page?**
Leboncoin serves 35 ads per result page. The actor follows the site's own pagination until `maxItems`, the source's page limit, or the end of the results.

**Does it handle every category?**
Yes. Category-specific details are returned in `attributes` and `attributesDetailed`, so property, vehicle, job, and goods listings keep their own facets.

**Are images included?**
Yes, as direct `img.leboncoin.fr` URLs. They are public and do not need a login.

## Data source and limitations

Data comes from the public search pages on `leboncoin.fr` and their embedded Next.js payload. The actor does not log in, does not use a private API, and does not bypass a login wall. Leboncoin's bot protection can occasionally challenge a request; the actor retries across its network paths and stops with a clear status message if every path is blocked. Search results are capped by Leboncoin's own page limit (shown as `max_pages` upstream, usually 100 pages); split a very broad search by price, location, or category to go deeper.

This is an independent tool. It is not affiliated with, endorsed by, or sponsored by Leboncoin.
