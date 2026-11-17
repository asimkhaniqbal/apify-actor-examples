# Just Eat UK Restaurant Search & Menu Extractor

Just Eat UK lists thousands of delivery restaurants by postcode, each with its own menu, pricing, and delivery details. This actor searches restaurants near a UK postcode or city, with cuisine, rating, and open-now filters, and can pull a restaurant's full menu — categories, items, variations, and add-ons — by its ID.

## What this actor does

- Searches restaurants near a UK postcode or a major city name, with cuisine, minimum rating, and open-now-for-delivery filters applied.
- Sorts search results by relevance, rating, or delivery time (a `distance` option exists but falls back to relevance order since Just Eat's search response doesn't carry a usable distance value).
- Fetches a restaurant's complete menu by ID: categories, items, size variations, modifier/add-on groups, and prices.
- Reports delivery ETA ranges, delivery cost, minimum order value, and UK food hygiene rating where Just Eat publishes them.
- Leaves a field out of a record whenever Just Eat doesn't have that data for the restaurant or item, rather than sending it empty.

## Output per restaurant (search mode)

| Field | Description |
|---|---|
| `restaurantId`, `uniqueName` | Identifiers used to build the restaurant's URL |
| `name`, `brandName`, `description` | Restaurant name, brand, and description |
| `address` | `{firstLine, city, postcode, latitude, longitude}` |
| `cuisines` | List of cuisine tags |
| `rating` | `{average, count, starRating}` |
| `deliveryEtaMinutes` | `{rangeLowerMinutes, rangeUpperMinutes, approximateMinutes}` |
| `deliveryCost`, `minimumOrderValue` | In GBP |
| `isOpenNow`, `isOpenNowForDelivery`, `isOpenNowForCollection` | Current open status |
| `offersDelivery`, `offersCollection`, `isFreeDelivery`, `isHalal`, `isNew`, `isPremier`, `isTemporarilyOffline` | Service and status flags |
| `hygieneRating` | UK Food Hygiene Rating (1-5), where published |
| `logoUrl`, `tags`, `menuId` | Logo image, listing tags, and the ID used to fetch the menu |
| `restaurantUrl` | Canonical Just Eat menu page |
| `recordType` | Always `restaurant` for search-mode records |
| `scrapedAt` | UTC timestamp of the scrape |

## Output per menu (menu mode)

One record per restaurant, with nested categories and items:

| Field | Description |
|---|---|
| `restaurantId`, `name`, `description` | Restaurant identity |
| `currency` | Currency code for all prices in the menu |
| `categories` | List of `{categoryId, name, description, items}`, where each item has: |
| &nbsp;&nbsp;`itemId`, `name`, `description`, `type` | Item identity and description |
| &nbsp;&nbsp;`price` | Lowest listed price for the item |
| &nbsp;&nbsp;`priceRange` | `{min, max}`, when the item has multiple sizes |
| &nbsp;&nbsp;`variationNames` | Size/variation names, e.g. `["Half", "12 inch", "14 inch"]` |
| &nbsp;&nbsp;`labels` | Dietary labels, e.g. `vegetarian`, `vegan`, `spicy` |
| &nbsp;&nbsp;`modifierGroups` | Add-on groups with each option's extra cost |
| `itemCount` | Total number of items on the menu |
| `sourceUrl` | Canonical menu page URL |
| `recordType` | Always `menu` for menu-mode records |
| `scrapedAt` | UTC timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (restaurants near a postcode/city) or `menu` (full menu by restaurant ID). |
| `location` | string | – | UK postcode/outcode or major UK city name, used in `search` mode. |
| `cuisine` | string | `Any` | Restrict results to one of 39 cuisine categories; restaurants with no cuisine data always pass through. |
| `sortBy` | string | `relevance` | `relevance`, `rating`, or `deliveryTime`. `distance` is also accepted but currently behaves like `relevance`. |
| `openNowOnly` | boolean | `false` | Keep only restaurants currently open for delivery. |
| `minRating` | integer | – | Drop restaurants below this star rating (1-5); restaurants with no rating always pass through. |
| `restaurantIds` | array | `[]` | Restaurant IDs to fetch full menus for, used in `menu` mode. |
| `maxItems` | integer | `20` | Maximum number of records to return. |
| `proxyConfiguration` | object | AUTO proxy enabled | Proxy configuration for the request. |

### Example: pizza near a postcode, highest rated first

```json
{
  "mode": "search",
  "location": "SW1A 1AA",
  "cuisine": "Pizza",
  "sortBy": "rating",
  "maxItems": 30
}
```

### Example: open-now restaurants above a rating threshold

```json
{
  "mode": "search",
  "location": "Manchester",
  "openNowOnly": true,
  "minRating": 4,
  "maxItems": 25
}
```

### Example: full menus for specific restaurants

```json
{
  "mode": "menu",
  "restaurantIds": ["100059", "200060"]
}
```

## Use cases

- **Market research** — compare cuisine mix, ratings, and pricing across different postcodes.
- **Menu intelligence** — track competitor menu items, prices, and portion options over time.
- **Delivery benchmarking** — compare delivery ETAs, fees, and minimum order values across an area.
- **Partnership outreach** — find restaurants in a postcode by cuisine for aggregator or supplier outreach.
- **Local directory enrichment** — add live rating, hygiene, and delivery data to a restaurant listing.

## FAQ

**Is this an official Just Eat tool?**
No. This actor reads Just Eat's own public restaurant and menu data and isn't affiliated with or endorsed by Just Eat.

**Do I need a login or API key?**
No.

**Why does `location` only work for the UK?**
This actor covers Just Eat's UK service only; Just Eat's other country sites (Ireland, Lieferando in Germany, Thuisbezorgd in the Netherlands, and others) run as separate platforms and aren't covered here.

**Can I search by an arbitrary place name?**
`location` accepts a UK postcode/outcode directly, or one of the curated major city names. Place names outside that list won't resolve to an area — use a postcode instead for anywhere not on the list.

**Why did my search return no restaurants?**
Either there's no Just Eat coverage for that postcode or city, or your `cuisine`/`minRating`/`openNowOnly` combination filtered everything out — try widening the filters.

**How fresh is the data?**
Every run reads Just Eat's live data at request time, so ratings, open status, and menu prices reflect what's current when the actor runs.

**Why would a restaurant be missing a hygiene rating or logo?**
Not every restaurant has those published in Just Eat's own data; fields with nothing to report are left off the record instead of showing a placeholder.

**What's the difference between `price` and `priceRange` on a menu item?**
`price` is the item's lowest listed price. `priceRange` only appears when the item has more than one size or variation, giving the lowest and highest price across those options.

**Why would a proxy help?**
Just Eat's restaurant and menu data sits behind bot-detection; routing requests through the proxy configuration improves how reliably each request succeeds.
