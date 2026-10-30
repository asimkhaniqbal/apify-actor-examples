# Pinterest Trending Topics & Shopping Category Scraper

Pinterest publishes its own Trends site with featured topics, editorial trend stories, and a shopping category taxonomy with click-based metrics by region. This actor reads those public pages directly, by region and time window, and returns exactly what Pinterest published — it never invents search volume, forecasts, rankings, or private account analytics.

## What this actor does

- **Four public modes:** featured topics, editorial stories, shopping categories, and shopping metrics.
- **Regional results:** supports the public `US`, `GB+IE`, and `CA` choices.
- **Time windows:** requests published historical and forecast periods for shopping metrics.
- **Rich trend context:** includes public examples, related searches, category hierarchy, and time-series values when supplied.
- **Truthful output:** preserves published zeroes and `false` values and omits unavailable data.

## Output per record

Every row includes `sourceUrl` and `recordType`. Depending on the selected mode, rows are `trendFeatured`, `trendEditorial`, `shoppingCategory`, or `shoppingMetric`.

### Featured topic fields

`trendId`, `name`, `description`, `region`, `isPublished`, `growthPercentMonth`, `interestIds`, `relatedSearches`, `examplePins`, `timeSeries`, and `sourceUrl`.

Each `examplePins` item can contain `pinId`, `pinUrl`, `imageUrl`, `width`, `height`, `color`, and `verticalOffset`. Each `timeSeries` item can contain `date`, `count`, `normalizedCount`, `normalizedPredictedLowerBound`, and `normalizedPredictedUpperBound`.

### Editorial story fields

`trendId`, `title`, `body`, `trendType`, `region`, `regions`, `interestIds`, `keywordsByRegion`, `startDate`, `endDate`, `isPublished`, `isReadyForTranslation`, `examplePins`, and `sourceUrl`.

### Shopping category fields

`categoryId`, `name`, `level`, `parentCategoryId`, `childCategoryIds`, and `sourceUrl`.

### Shopping metric fields

`categoryId`, `name`, `region`, `event`, `dailyValues`, `growthRates`, and `sourceUrl`.

Missing values are omitted. Pinterest-published zero counts, dates, and boolean `false` values are retained. Metric arrays and category names are copied from Pinterest and are not transformed into estimates.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | choice | `featured` | `featured`, `editorial`, `shoppingCategories`, or `shoppingMetrics`. |
| `region` | choice | `US` | Public Trends region: `US`, `GB+IE`, or `CA`. |
| `event` | choice | `OUTBOUND_CLICK` | Shopping event currently exposed by Pinterest. |
| `categoryNames` | array of text | — | Public shopping category names for metrics, such as `Sofas`. |
| `maxItems` | integer | `50` | Maximum records to return. |
| `days` | integer | `60` | Historical metric window, from 7 to 90 days. |
| `predictedDays` | integer | `28` | Optional Pinterest forecast window, from 0 to 28 days. |
| `cookies` / `cookiesJson` | array/string | — | Optional Cookie-Editor export; public Trends does not require it. |
| `proxyConfiguration` | object | — | Optional Apify proxy configuration. |

### Example: featured topics

```json
{
  "mode": "featured",
  "region": "US",
  "maxItems": 25
}
```

### Example: editorial stories

```json
{
  "mode": "editorial",
  "region": "GB+IE",
  "maxItems": 25
}
```

### Example: shopping taxonomy

```json
{
  "mode": "shoppingCategories",
  "maxItems": 1000
}
```

### Example: shopping history and forecast

```json
{
  "mode": "shoppingMetrics",
  "region": "US",
  "categoryNames": ["Sofas", "Wedding dress"],
  "days": 60,
  "predictedDays": 14,
  "maxItems": 50
}
```

## Use cases

- Discover emerging public Pinterest topics by region.
- Build editorial trend calendars from Pinterest story content.
- Map Pinterest's public product-category hierarchy.
- Compare public shopping interest over a selected historical window.
- Feed real Pinterest examples and time series into planning or reporting workflows.

## FAQ

### Is this Pinterest Ads analytics?

No. It collects public Trends responses. Private business analytics, advertiser accounts, and exact private audience volumes are not promised.

### Is `growthPercentMonth` a calculated estimate?

No. It is emitted only when Pinterest supplies the value for a featured topic.

### Why are there fewer rows than `maxItems`?

The public Trends source may publish fewer topics, categories, or metric terms than the requested cap. The actor does not duplicate or invent rows.

### Why did a forecast disappear?

Some region/category combinations publish history but no forecast. The actor returns real historical values and does not substitute a prediction.

### Can I use another region?

Use the regions exposed by the actor input. The currently verified public choices are `US`, `GB+IE`, and `CA`.

### Do category names need an exact match?

Names are matched against Pinterest's public category dictionary. If no requested name matches, the actor uses publicly returned categories when available.

### Do I need cookies?

No. Public Trends data is collected without login cookies. Optional cookies can be supplied for a permitted session.

### Can this actor return 10,000 rows?

The input supports a large cap and emits progressively, but verified public Trends modes currently contain far fewer categories/topics/terms. The actor returns the complete source-bounded result.
