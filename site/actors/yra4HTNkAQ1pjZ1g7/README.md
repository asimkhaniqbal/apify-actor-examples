# Fiverr Gig & Seller Sitemap Directory Scraper

Scrape **Fiverr** gigs, sellers and directories without an account, cookies or API keys. Search live gig results by keyword or category, open full gig pages for packages and reviews, read seller profiles, and list Fiverr's public sitemap catalog of 280,000+ gigs and 280,000+ sellers. Results are real, structured and exported as JSON, CSV or Excel.

## What this actor does

- **Four modes:** `search`, `gigDetails`, `sellerProfiles`, `sitemapDirectory`
- **Live search** with Fiverr's own filters for seller level, country, language, delivery time, budget, Pro, online, agency and hourly gigs, plus five sort options
- **Full gig detail** with every package tier, features, paid extras, FAQ, ratings breakdown, latest reviews, gallery, seller card and orders in queue
- **Seller profiles** with level, response time, rating, skills, certifications, education, languages, portfolio count, notable clients, all gigs and latest reviews
- **Full sitemap directory** for gigs, sellers, categories, search tags and Logo Maker ideas, including exact sitemap counts and category IDs
- **Backward compatible:** the original discovery mode names still work, and every existing output field is kept
- **Empty fields are omitted** — records never contain nulls or placeholder strings

## Output per record type

### Gig search result (`recordType: "gigListing"`)

- `gigId`, `gigUrl`, `gigSlug`, `title`
- `sellerUsername`, `sellerId`, `sellerDisplayName`, `sellerProfileUrl`, `sellerProfilePhotoUrl`
- `sellerLevel`, `sellerCountryCode`, `sellerOnline`, `sellerOverallRating`, `sellerOverallReviewsCount`, `sellerLanguages`
- `priceMin`, `currency`, `minDeliveryDays`, `packagesCount`
- `rating`, `reviewsCount`
- `isPro`, `isFiverrChoice`, `isFeatured`, `hasVideoPreview`, `hasRecurringOption`, `offerConsultation`, `hourlyRateUsd`
- `attributes` — Fiverr's own facet tags, each `{type, values[]}`
- `workSamples` — public delivery previews, each `{type, url}`
- `agency` — when the gig is agency-run (`slug`, `established`, `employeesCount`, `highlights`, `bannerImageUrl`)
- `categoryId`, `subCategoryId`, `nestedSubCategoryId`
- `position`, `page`, `searchQuery`
- `recordType`, `scrapedAt`

### Gig detail (`recordType: "gigDetail"`)

- `sourceUrl`, `sellerUsername`, `sellerProfileUrl`, `gigSlug`, `gigId`, `gigStatus`
- `title`, `description`, `fullDescription`, `imageUrl`
- `rating`, `reviewsCount`, `gigCollectedCount`, `ordersInQueue`
- `priceMin`, `priceMax`, `currency`, `packagesCount`, `availability`, `minDeliveryDays`
- `category`, `categoryUrl`, `subCategory`, `subCategoryUrl`, `nestedSubCategory`, `nestedSubCategoryUrl`, `categoryPath`
- `categoryId`, `subCategoryId`, `nestedSubCategoryId`, `categorySlug`, `subCategorySlug`, `nestedSubCategorySlug`
- `isPro`, `acceptsCustomOrders`
- `sellerLevel`, `sellerIsVerified`, `sellerIsHighlyResponsive`, `sellerCompletedOrders`, `sellerSkills`
- `sellerOverallRating`, `sellerOverallReviewsCount`, `sellerProSubCategories`
- `sellerDisplayName`, `sellerLanguages`, `sellerCountryCode`, `sellerProfilePhotoUrl`
- `sellerOneLiner`, `sellerResponseTimeHours`, `sellerMemberSince`, `sellerRecentDeliveryAt`, `sellerIsOnVacation`, `sellerAllowsContact`, `isTrustedFreelancer`
- `ratingBreakdown` — `{stars, count}` per star value
- `communicationRating`, `recommendRating`, `serviceRating`
- `reviews` — latest reviews, each `{reviewId, rating, comment, reviewerUsername, reviewerCountryCode, reviewerCountry, createdAt, orderDurationDays, orderPriceRangeUsd, repeatBuyer}`
- `packages` — `{tier, title, description, priceUsd, deliveryDays, revisions, extraFastDeliveryPriceUsd, extraFastDeliveryDays, features[]}`
- `gigExtras` — paid add-ons, each `{name, description, priceUsd, extraDeliveryDays}`
- `faq` — `{question, answer}` pairs
- `gigTags`, `gigGallery`, `sellerOtherGigIds`, `sellerPortfolioProjects`, `sellerPortfolioProjectsCount`
- `relatedServiceTypes`, `relatedSubCategoryName`, `notableClients`, `aiSummary`
- `recordType`, `scrapedAt`

### Seller profile (`recordType: "sellerProfile"`)

- `sellerUsername`, `sellerProfileUrl`, `sellerId`, `sellerDisplayName`
- `sellerCountryCode`, `sellerCountryName`, `sellerLanguages`, `sellerLanguageProficiencies`
- `sellerProfilePhotoUrl`, `sellerBannerUrl`, `sellerDescription`, `sellerOneLiner`
- `sellerLevel`, `sellerAchievementLevel`, `sellerIsPro`, `sellerIsVerified`, `sellerIsHighlyResponsive`, `sellerIsActive`, `sellerIsOnVacation`
- `sellerOverallRating`, `sellerOverallReviewsCount`, `sellerResponseTimeHours`, `sellerMemberSince`, `sellerApprovedGigsCount`, `sellerHourlyRateUsd`
- `sellerSkills` — `{name, level, verified}`
- `sellerCertifications` — `{name, issuer, year}`
- `sellerEducations` — `{school, degree, degreeTitle, countryCode, fromYear, toYear}`
- `sellerPortfolioProjectsCount`, `sellerNotableClients`, `sellerAgency`
- `sellerGigs` — `{gigId, gigSlug, gigUrl, title, priceMin, currency, minDeliveryDays, rating, reviewsCount, imageUrl, isPro, isFiverrChoice, categoryId, subCategoryId, nestedSubCategoryId}`
- `sellerReviews` — latest reviews with the same fields as gig reviews
- `sellerRatingBreakdown`, `sellerReviewTotal`, `sellerCommunicationRating`, `sellerRecommendRating`, `sellerServiceRating`
- `sourceUrl`, `recordType`, `scrapedAt`

### Sitemap directory records

`recordType: "gigUrl"` — `gigUrl`, `gigId`, `gigSlug`, `sellerUsername`, `sellerProfileUrl`, `imageUrl`, `imageUrls`, `imageCaption`, `previewVideo`, `hasVideoPreview`, `sourceSitemapShard`, `recordType`, `scrapedAt`.

`recordType: "sellerProfile"` from the seller sitemap — `sellerUsername`, `sellerProfileUrl`, `sellerId`, `sellerProfilePhotoUrl`, `sourceSitemapShard`.

`recordType: "category"` — `categoryUrl`, `categoryLevel`, `name`, `categorySlug`, `subCategorySlug`, `nestedSubCategorySlug`, `categoryId`, `subCategoryId`.

`recordType: "tag"` — `tagSlug`, `tagName`, `tagSearchUrl`, `sourceSitemapShard`.

`recordType: "logoMakerIdea"` — `ideaSlug`, `ideaType`, `ideaName`, `ideaUrl`, `imageUrl`, `imageCaption`.

`recordType: "marketSummary"` (optional) — `searchQuery`, `totalResults`, `sampledCount`, `priceMin`, `priceMedian`, `priceMax`, `currency`, `sellerLevelCounts`, `topSellerCountries`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `sitemapDirectory` | `search`, `gigDetails`, `sellerProfiles` or `sitemapDirectory`. Legacy names (`discoverGigs`, `discoverSellers`, `discoverCategories`, `discoverTags`, `discoverLogoMakerIdeas`, `discoverAndFetchDetails`) are also accepted. |
| `searchQueries` | array | `[]` | Keywords to search on Fiverr (mode=`search`). |
| `searchUrls` | array | `[]` | Fiverr search or category URLs to browse (mode=`search`). Pasted filters are preserved. |
| `gigUrls` | array | `[]` | Gig URLs to fetch (mode=`gigDetails`). |
| `sellerUrls` | array | `[]` | Seller usernames or profile URLs (mode=`sellerProfiles`). |
| `sitemapType` | string | `gigs` | `gigs`, `sellers`, `categories`, `tags` or `logomakerIdeas` (mode=`sitemapDirectory`). |
| `searchKeyword` | string | – | Case-insensitive substring filter for sitemap URLs/slugs; also filters titles in detail/search modes. |
| `maxShards` | integer | `1` | Sitemap shard files to scan: gigs up to 7, sellers up to 6, tags up to 2. |
| `categoryLevel` | string | *(all)* | Category level filter for `sitemapType=categories`. |
| `ideaType` | string | *(all)* | Logo Maker idea type filter: `tag`, `industry`, `monogram`, `brandValue`, `style`, `symbolType`. |
| `category` | string | *(all)* | Top-level category filter for category directory, gig details and enriched search results. |
| `sortBy` | string | `relevance` | `relevance`, `best_selling`, `price_low_to_high`, `price_high_to_low`, `newest` (mode=`search`). |
| `sellerLevels` | array | `[]` | `top_rated_seller`, `level_two_seller`, `level_one_seller`, `na` (new seller). |
| `sellerLanguages` | array | `[]` | ISO language codes, e.g. `en`, `es`, `de`. |
| `sellerCountries` | array | `[]` | ISO-2 country codes, e.g. `US`, `GB`, `DE`. |
| `deliveryTime` | string | `any` | `any`, `1` (24 hours), `3` or `7` days. |
| `proOnly` | boolean | `false` | Only Fiverr Pro gigs. |
| `onlineOnly` | boolean | `false` | Only sellers online now. |
| `agencyOnly` | boolean | `false` | Only agency-run gigs. |
| `hourlyOnly` | boolean | `false` | Only gigs with an hourly rate. |
| `facets` | array | `[]` | Category-specific filters as `key:value`, e.g. `style:flat_minimalist` or `file_format:svg`. |
| `minRating` | number | – | Minimum gig or seller rating (0–5). |
| `minReviews` | integer | – | Minimum review count. |
| `minPriceUsd` / `maxPriceUsd` | number | – | Starting-price range in USD. |
| `includeGigDetails` | boolean | `false` | Fetch each search result's gig page for packages, reviews and full detail. |
| `includePackages` | boolean | `true` | Include package pricing, features and paid extras on gig details. |
| `includeFaq` | boolean | `true` | Include gig FAQ when present. |
| `includeSellerGigs` | boolean | `true` | Include the seller's gig list on seller records. |
| `includeSellerReviews` | boolean | `true` | Include latest seller reviews and rating breakdown. |
| `includeMarketSummary` | boolean | `false` | Emit one market-summary row per search query. |
| `maxPagesPerQuery` | integer | `21` | Upper bound on search pages per query or URL. Fiverr often stops earlier; the actor stops when Fiverr returns no more results. |
| `maxItems` | integer | `20` | Hard cap on emitted records. |

### Example: search logo gigs with filters

```json
{
  "mode": "search",
  "searchQueries": ["logo design"],
  "sellerLevels": ["top_rated_seller", "level_two_seller"],
  "sellerCountries": ["US", "GB"],
  "deliveryTime": "3",
  "minPriceUsd": 10,
  "maxPriceUsd": 100,
  "sortBy": "price_low_to_high",
  "maxItems": 50
}
```

### Example: fetch full detail for known gigs

```json
{
  "mode": "gigDetails",
  "gigUrls": [
    "https://www.fiverr.com/michael7878/master-your-song-in-1-day",
    "https://www.fiverr.com/aurexhq/create-modern-minimalist-logo-design-for-your-business"
  ],
  "includePackages": true,
  "includeFaq": true
}
```

### Example: seller profiles

```json
{
  "mode": "sellerProfiles",
  "sellerUrls": ["michael7878", "jidesigns"],
  "includeSellerGigs": true,
  "includeSellerReviews": true
}
```

### Example: list a sitemap directory

```json
{
  "mode": "sitemapDirectory",
  "sitemapType": "categories",
  "categoryLevel": "subCategory",
  "maxItems": 100
}
```

## Use cases

- **Market research** — size a Fiverr niche by gig and seller counts, price bands and seller-level mix
- **Pricing intelligence** — compare package tiers, delivery times, revisions and paid extras
- **Seller lead lists** — build shortlists by level, country, language, response time and rating
- **Competitor tracking** — schedule the same search or seller list and compare prices and reviews over time
- **SEO and keyword research** — mine Fiverr's own search-tag vocabulary and category tree
- **Catalog integrations** — sync the full public gig and seller directory into a database or CRM

## FAQ

**Is this affiliated with Fiverr?**
No. This is an independent third-party actor that reads Fiverr's public pages and sitemaps. It is not affiliated with or endorsed by Fiverr.

**Do I need a Fiverr account, cookies or an API key?**
No. There are no login, cookie or API-key inputs. Fill in a mode and run it.

**Do I need to configure a proxy?**
No. The actor chooses a working connection to Fiverr automatically.

**Why does a search return fewer gigs than Fiverr says exist?**
Fiverr's search window is shallower than the headline total and varies by query. On 2026-10-26 a `logo design` search returned 11 pages (516 cards) and about 328 unique gigs before Fiverr stopped; promoted gigs repeat across pages. The actor reads until Fiverr reports no further results, and `maxPagesPerQuery` is only an upper bound. Fiverr also echoes some requested sort or filter values without applying them. The actor validates the response and applies price sorts locally; unsupported filter values are named in the run log. Use narrower filters or the `sitemapDirectory` mode for the full public catalog.

**What is the difference between search and sitemapDirectory?**
`search` reads live Fiverr search/category results with filters and sorting. `sitemapDirectory` reads Fiverr's public sitemaps, which cover 280,000+ gigs and 280,000+ sellers but only carry URL-level metadata.

**Can I get a seller's email or phone number?**
No. Fiverr does not publish seller emails or phone numbers on public profiles.

**How many reviews can I get?**
Gig and seller pages expose the latest reviews on the page, so the actor returns those reviews plus the full star breakdown and total count. It does not return every historical review.

**Why are some fields missing on a record?**
Fields are omitted when Fiverr does not expose them for that gig or seller. The actor never fills missing values with guesses, nulls or placeholders.

**How fresh is the data?**
Search, gig and seller records are read live at run time. Directory records follow Fiverr's latest sitemap refresh.

**Can I still use the old mode names?**
Yes. `discoverGigs`, `discoverSellers`, `discoverCategories`, `discoverTags`, `discoverLogoMakerIdeas` and `discoverAndFetchDetails` map to the new directory and detail paths, and existing output fields are unchanged.
