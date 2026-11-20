# Leboncoin France Classified Ads Extractor

Leboncoin is France's largest classifieds site, covering everything from apartment rentals and used cars to job offers and electronics in one marketplace. This actor reads any Leboncoin search — pasted as a URL or built from a category and keyword — and returns every ad as a structured record.

## What this actor does

- Scrapes any Leboncoin search results URL, keeping whatever filters (location, price, category) are already in that URL.
- Builds a search from a category ID and free-text keyword when you don't have a URL handy.
- Walks pagination automatically up to your requested item count.
- Flattens Leboncoin's category-specific attributes (e.g. real estate surface area and room count, vehicle mileage and fuel type) into a single `attributes` object per ad.
- Fields an ad doesn't provide (for example a store ID for a private seller, or category attributes that don't apply) are left out of that record; a couple of numeric fields (`price`, `latitude`/`longitude`) instead come back as `0` when unpublished — see the FAQ.

## Output per ad

| Field | Description |
| --- | --- |
| `id` | Leboncoin's internal ad ID. |
| `url` | Direct link to the ad on leboncoin.fr. |
| `subject` | Ad title. |
| `body` | Ad description text (truncated to 2000 characters). |
| `categoryId` / `categoryName` | Leboncoin category ID and name. |
| `adType` | Ad type (e.g. offer). |
| `status` | Ad status. |
| `publishedDate` / `expirationDate` | When the ad was first published and when it expires. |
| `hasPhone` | Whether the seller listed a phone number. |
| `price` | Asking price in EUR (`0` when no price is published). |
| `city` / `zipcode` / `department` / `region` / `country` | Location of the ad. |
| `latitude` / `longitude` | GPS coordinates of the location (`0` when Leboncoin doesn't publish coordinates for the ad). |
| `imageCount` | Number of photos on the ad. |
| `thumbnail` | Thumbnail image URL. |
| `imageUrls` | Full-size photo URLs. |
| `ownerName` | Seller's display name. |
| `ownerType` | Whether the seller is a private individual or a professional. |
| `ownerId` / `ownerStoreId` | Seller's user ID and store ID (for professional sellers). |
| `isUrgent` / `isHighlighted` | Whether the ad was boosted as urgent or highlighted. |
| `attributes` | Category-specific details (e.g. surface area, rooms, mileage, fuel type), as key/value pairs. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `startUrls` | Array of strings | – | Leboncoin search URLs to scrape, with any filters already applied in the URL. |
| `category` | Integer | – | Leboncoin category ID, used to build a search only when `startUrls` is empty. Common IDs: 9 = real estate sales, 10 = rentals, 2 = cars, 6 = auto equipment, 17 = phones, 33 = job offers. Other IDs can be found in a Leboncoin search URL's `category` parameter. |
| `search` | String | – | Free-text keyword, combined with `category` when no `startUrls` are set. |
| `maxItems` | Integer | 50 | Maximum number of ads to return. |

### Example: rentals in Paris under €2,000

```json
{
  "startUrls": [
    "https://www.leboncoin.fr/recherche?category=10&locations=Paris&price=min-2000"
  ],
  "maxItems": 100
}
```

### Example: build a search from category and keyword

```json
{
  "category": 2,
  "search": "Peugeot 208",
  "maxItems": 50
}
```

### Example: job offers nationwide

```json
{
  "category": 33,
  "maxItems": 200
}
```

## Use cases

- **Real estate agents and renters** tracking new listings in a specific city or price band.
- **Car buyers** comparing used-vehicle listings and mileage across a model.
- **Market researchers** analyzing price trends for a product category over time.
- **Lead-generation teams** segmenting professional vs. private sellers via `ownerType`.
- **Job seekers and recruiters** monitoring job-offer volume and categories.

## FAQ

**Does this need a Leboncoin account?**
No. It reads the same public ad listings shown to any visitor on leboncoin.fr.

**What's the difference between using `startUrls` and `category`/`search`?**
`startUrls` lets you reuse any filters Leboncoin's own search UI already applied to a URL (location, price range, sub-filters). `category`/`search` is a simpler way to start a basic search without first visiting the site.

**What's inside `attributes`?**
It's whatever category-specific fields Leboncoin publishes for that ad type — for example surface area and room count for real estate, or mileage and fuel type for vehicles. The set of keys varies by category.

**Why do some ads show `latitude`/`longitude` as `0`?**
Not every ad includes precise coordinates; when Leboncoin doesn't publish them, these fields come back as `0` rather than being left off the record. The same applies to `price` when no asking price is published.

**How many ads come back per search page?**
Leboncoin serves a fixed number of ads per page; the actor keeps requesting further pages automatically until it reaches `maxItems` or runs out of results.

**Is this affiliated with Leboncoin?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Leboncoin.
