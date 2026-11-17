# Just Eat UK Restaurant Search & Menu Extractor

Just Eat UK returns more than 1,700 delivery restaurants for a busy postcode, each with its own menu, pricing, and delivery terms. This actor pulls the complete restaurant result set for any UK postcode or city, filters and pages through it, and can also fetch full menus — categories, items, sizes, and add-ons — for specific restaurants.

## What this actor does

- Pulls the **whole** restaurant result set for a UK postcode/outcode or a major city name (not just the first page of ten).
- Filters by cuisine, minimum star rating, and open-now-for-delivery, then sorts by relevance, rating, delivery time, or real distance from the searched area.
- Pages through results with `offset` + `maxItems`, so a large area can be collected in chunks across runs.
- Fetches a restaurant's complete menu from its menu URL, slug, or numeric ID: categories, item descriptions, prices, size variations, add-on groups, dietary labels, energy information, and item images.
- Reports delivery ETA ranges, delivery fee, minimum order value, coordinates, distance, and current open status.
- Leaves out any field that Just Eat doesn't publish for a given restaurant or item; no nulls or placeholders.

## Output per restaurant (search mode)

| Field | Description |
|---|---|
| `restaurantId`, `uniqueName` | Identifiers used to build the restaurant's menu URL |
| `name` | Restaurant name |
| `address` | `{firstLine, city, postcode, latitude, longitude}` |
| `cuisines` | List of cuisine names |
| `rating` | `{count, starRating, userRating}` where published |
| `deliveryEtaMinutes` | `{rangeLowerMinutes, rangeUpperMinutes, approximateMinutes}` |
| `deliveryCost`, `minimumOrderValue` | In GBP |
| `distanceMeters` | Straight-line-ish drive distance returned by Just Eat |
| `isOpenNow`, `isOpenNowForDelivery`, `isOpenNowForCollection`, `isOpenNowForPreorder` | Current availability |
| `offersDelivery`, `offersCollection`, `isFreeDelivery`, `isHalal`, `isNew`, `isPremier`, `isTemporarilyOffline` | Service and status flags |
| `logoUrl`, `tags` | Logo image and public listing tags |
| `restaurantUrl` | Canonical Just Eat menu page |
| `recordType` | Always `restaurant` for search-mode records |
| `scrapedAt` | UTC timestamp of the scrape |

## Output per menu (menu mode)

One record per restaurant, with nested categories and items:

| Field | Description |
|---|---|
| `restaurantId`, `name`, `description` | Restaurant identity |
| `currency` | Currency code for all prices (`GBP`) |
| `menuVersion` | Just Eat's menu revision identifier |
| `categories` | List of `{categoryId, name, description, preview, items}`, where each item has: |
| &nbsp;&nbsp;`itemId`, `name`, `description`, `type` | Item identity and description |
| &nbsp;&nbsp;`price` | Normal selling price (deal-only bundle prices excluded) |
| &nbsp;&nbsp;`priceRange` | `{min, max}` when the item has multiple paid sizes |
| &nbsp;&nbsp;`variationNames` | Size/variation names, e.g. `["Regular", "Large"]` |
| &nbsp;&nbsp;`labels` | Dietary labels, e.g. `vegetarian`, `vegan` |
| &nbsp;&nbsp;`energyContent` | Energy label, e.g. `282 kcal` |
| &nbsp;&nbsp;`imageUrl` | Item image URL |
| &nbsp;&nbsp;`modifierGroups` | Add-on groups with each option's extra cost |
| `itemCount` | Number of resolved items across categories |
| `sourceUrl` | Canonical menu page URL |
| `recordType` | Always `menu` for menu-mode records |
| `scrapedAt` | UTC timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (restaurants near a postcode/city) or `menu` (full menus). |
| `location` | string | – | UK postcode/outcode or major UK city name. In `menu` mode, optional but needed to resolve numeric restaurant IDs. |
| `cuisine` | string | `Any` | Restrict results to one of the listed cuisine categories; restaurants with no cuisine data always pass through. |
| `sortBy` | string | `relevance` | `relevance`, `rating`, `deliveryTime`, or `distance`. |
| `openNowOnly` | boolean | `false` | Keep only restaurants currently open for delivery. |
| `minRating` | integer | – | Drop restaurants below this star rating (1-5); restaurants with no rating always pass through. |
| `restaurantIds` | array | `[]` | Menu URLs, restaurant slugs, or numeric IDs. Numeric IDs also need `location`. |
| `offset` | integer | `0` | Skip this many matching restaurants before emitting records. |
| `maxItems` | integer | `20` | Maximum number of records to emit (1-500). |
| `proxyConfiguration` | object | automatic | Optional override. The actor automatically uses Apify's included AUTO proxy when a direct request is blocked. |

### Example: pizza near a postcode, nearest first

```json
{
  "mode": "search",
  "location": "SW1A 1AA",
  "cuisine": "Pizza",
  "sortBy": "distance",
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

### Example: page through a large area

```json
{
  "mode": "search",
  "location": "EC4M",
  "sortBy": "rating",
  "offset": 100,
  "maxItems": 100
}
```

### Example: full menus by URL, slug, or ID

```json
{
  "mode": "menu",
  "restaurantIds": [
    "https://www.just-eat.co.uk/restaurants-pretamanger-paternostersquarelondon/menu",
    "grain-mediterranean-london"
  ],
  "maxItems": 5
}
```

Numeric IDs still work when you also pass a `location`:

```json
{
  "mode": "menu",
  "location": "EC4M",
  "restaurantIds": [114934]
}
```

## Use cases

- **Market research** — compare cuisine mix, ratings, fees, and distance across postcodes.
- **Menu intelligence** — track competitor menu items, prices, sizes, and add-ons over time.
- **Delivery benchmarking** — compare ETAs, delivery fees, and minimum order values across an area.
- **Partnership outreach** — find restaurants in a postcode by cuisine for supplier or aggregator outreach.
- **Local directory enrichment** — add live ratings, coordinates, and menu data to a restaurant listing.

## Limitations

- **UK only.** Just Eat's other country sites run on separate stacks and aren't covered.
- **Numeric menu IDs need a location.** The public API has no restaurant-ID lookup; without `location` the actor cannot map an ID to the menu slug. Pass a menu URL or slug instead for exact targeting.
- **Paging is local.** Just Eat's endpoint accepts a result `limit` but ignores `offset`/`page`, so the actor fetches the area result set and slices it with `offset`.
- **Some competitor fields aren't published.** Hygiene ratings, phone numbers, reviews, and opening hours are not in the public consumer payload; they are omitted rather than guessed.
- **City names come from a curated list.** Any UK postcode or outcode works; outside the supported city list, use a postcode.

## FAQ

**Is this an official Just Eat tool?**
No. This actor reads Just Eat's own public restaurant and menu data and isn't affiliated with or endorsed by Just Eat.

**Do I need a login or API key?**
No.

**Do I need to configure a proxy?**
No. The actor tries a direct request first and automatically switches to Apify's included AUTO proxy if Just Eat's bot protection blocks it.

**Why did my search return no restaurants?**
Either there's no Just Eat coverage for that postcode/city, the offset is past the end of the result set, or the cuisine/rating/open-now combination filtered everything out. Try widening the filters or lowering `offset`.

**How many restaurants can one run return?**
A busy London outcode currently has about 1,700 delivery restaurants. Increase `maxItems` (up to 500) and use `offset` to collect the rest in further runs.

**What's the difference between `price` and `priceRange` on a menu item?**
`price` is the item's normal selling price; deal-only bundle prices are excluded. `priceRange` appears when the item has more than one paid size.

**Why would a restaurant be missing a fee, rating, or logo?**
Just Eat doesn't publish every field for every restaurant, and some are collection-only. Fields with nothing to report are left off the record instead of showing a placeholder.

**How fresh is the data?**
Every run reads Just Eat's live data at request time, so ratings, open status, fees, and menu prices reflect what's current when the actor runs.
