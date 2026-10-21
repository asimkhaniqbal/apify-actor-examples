# Facebook Page & Profile Finder by Keyword or URL

Find Facebook Pages and public profiles by keyword, or read a list of Facebook URLs, IDs and handles. Each row can include the verified badge, category, phone, email, address, website, followers, ratings and reviews — plus optional Page Transparency and the latest visible post. No login, cookies or proxy setup.

## What this actor does

- **Two ways in:** search Facebook by keyword, or read Pages/profiles you already have via URLs, `profile.php?id=`, `/people/`, `/p/`, numeric IDs and `@handles`.
- **Pages, profiles or both:** one search can return Facebook Pages, public profiles, or a deduplicated mix of the two.
- **Deep, paginated results:** follow Facebook's own result pages up to your `maxResults` limit, instead of stopping at a handful of search-engine hits.
- **Enrichment on demand:** turn `fetchPageDetails` on for contact details, category, followers and ratings; add reviews, Page Transparency and the latest visible post when you need them.
- **Verified filtering:** keep only Pages/profiles Facebook marks as verified with `verifiedOnly`.
- Empty fields are omitted — the actor never fills a missing value with `null`, `0`, or a placeholder.

## Output per Page or profile

| Field | Description |
|---|---|
| `facebookUrl` / `pageUrl` | Normalized URL of the Page or profile. |
| `pageName` | Display name. |
| `pageId` / `facebookId` | Facebook identifier (numeric or `pfbid…` for some profiles). |
| `title` | Page title. |
| `verified` | `true`/`false` when Facebook exposes the badge. |
| `entityType` | `page` or `profile` when search identifies it. |
| `searchQuery` / `searchRank` | Keyword that found the row and its 1-based position (reset per UID slice). |
| `searchLocationUid` | The location UID slice that produced the row, when `locationUids` is used. |
| `category` / `categories` | Primary category and full list. |
| `intro` / `info` | Page introduction and up to 5 description lines. |
| `likes` | Likes count (exact when Facebook exposes it). |
| `followers` / `followings` | Follower and following counts (exact when `og:description` carries them, otherwise Facebook's rounded number). |
| `talkingAboutCount` / `checkins` | "Talking about this" and "were here" counts. |
| `phone` / `email` / `messenger` | Contact details when published. |
| `address` / `addressUrl` | Address text and a Google Maps link. |
| `website` / `websites` | Linked website(s). |
| `priceRange` | Price marker such as `££` or `$$`. |
| `business_hours` / `openNowStatus` | Published hours and the current open/closed label when shown. |
| `ratingOverall` / `ratingCount` / `ratingText` / `recommendPercentage` | Rating and recommendation data. |
| `ownerOrganization` | Organization Facebook discloses as responsible for the Page. |
| `profilePictureUrl` / `coverPhotoUrl` | Profile and cover image URLs (signed; ~5-day expiry unless `rehostImages` is on). |
| `profilePictureUrlOriginal` / `coverPhotoUrlOriginal` | Original signed URLs retained when rehosting. |
| `pageAdLibrary` | Link to the Page's Ad Library listing. |
| `pageCreatedDate` / `isRunningAds` / `pageTransparencyId` | Added by `addDetail`. |
| `reviews` | Up to 10 `{reviewerName, recommends, reviewText, reviewDate}` objects. |
| `posts` | Up to `maxPosts` recent posts: `post_text`, `post_timestamp`, `post_url`, `media_urls`, `reaction_count`, `comment_count`, `share_count`. |
| `instagram` / `instagramFollowers` / `instagramVerified` | Linked Instagram profile when the Ad Library index knows it. |
| `scrapedAt` | Collection timestamp (ISO 8601 UTC). |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search` (find by keyword) or `urls` (read URLs/IDs/handles). |
| `searchQueries` | string list | – | Keywords for `mode=search`; a Facebook search URL is accepted. |
| `searchType` | select | `pages` | `pages`, `people` or `all`. |
| `location` | string | – | City/country added to each keyword; relevance hint, not a strict filter. |
| `locationUids` | array | – | Numeric Facebook Place IDs. Runs one page search per UID, splits `maxResults` across them, dedupes by page ID and adds `searchLocationUid`; nearby results can appear. |
| `categoryFilter` | string | – | Case-insensitive category text filter (e.g. `dentist`); requires page details. |
| `startUrls` | URL list | – | Facebook Page/profile URLs for `mode=urls`. |
| `startIds` | ID/handle list | – | Numeric IDs or `@handles` for `mode=urls`; combine with `startUrls` when needed. |
| `maxResults` | integer | `50` | 1–5000 rows per query (or in total for `urls`). |
| `fetchPageDetails` | boolean | `true` | Add category, contacts, followers and ratings. Off = fast identity-only list. |
| `includeReviews` | boolean | `true` | Add up to 10 published reviews per Page. |
| `addDetail` | boolean | `false` | Add Page creation date, ad status and Transparency ID. |
| `includePosts` | boolean | `false` | Add up to `maxPosts` recent timeline posts. |
| `maxPosts` | integer | `5` | 1–20; only used with `includePosts`. |
| `verifiedOnly` | boolean | `false` | Keep only verified results. |
| `rehostImages` | boolean | `false` | Copy profile/cover images to Apify storage; signed Facebook CDN links expire in about five days. |

### Example: search Pages and profiles

```json
{
  "mode": "search",
  "searchQueries": ["coffee shop", "dentist Berlin"],
  "searchType": "all",
  "maxResults": 100,
  "fetchPageDetails": true,
  "includeReviews": true
}
```

### Example: read a list of URLs, IDs and handles

```json
{
  "mode": "urls",
  "startUrls": [
    { "url": "https://www.facebook.com/McDonalds" },
    { "url": "https://www.facebook.com/profile.php?id=100064458773195" }
  ],
  "startIds": ["100064458773195", "@Starbucks"],
  "includePosts": true,
  "maxPosts": 1
}
```

### Example: verified-only identity list

```json
{
  "mode": "search",
  "searchQueries": ["nike", "starbucks"],
  "searchType": "pages",
  "maxResults": 50,
  "fetchPageDetails": false,
  "verifiedOnly": true
}
```

## Limits

- `maxResults` is a ceiling. A single keyword query has its own finite index; add keywords, or use `locationUids` to partition a pages search across several Place IDs and merge the slices.
- Facebook does not expose a server-side category filter to logged-out search. `categoryFilter` filters the enriched rows locally and requires `fetchPageDetails`.
- Profile follower counts are returned only when Facebook shows them publicly; hidden/zero counts are omitted rather than guessed.
- Exact likes in URL mode are filled when the page matches Meta's Ad Library typeahead; otherwise Facebook does not expose the count and it is omitted.
- Facebook image URLs are signed and expire after about five days. Turn on `rehostImages` for durable profile/cover images; the original URLs are kept in `profilePictureUrlOriginal` and `coverPhotoUrlOriginal`.
- `locationUids` partitions page searches by Facebook Place IDs; each row records its slice in `searchLocationUid`. Results can include nearby businesses, not only a strict city boundary.

## Use cases

- **Lead generation** — find local businesses by keyword and collect the phone, email, address and website they publish.
- **Brand research** — compare Pages in a category by followers, rating and verified status.
- **Competitor discovery** — find every Page and public profile active around a keyword.
- **Bulk enrichment** — feed a CRM export of URLs or numeric IDs and add contact and engagement data.
- **Reputation monitoring** — pull published reviews and the latest post for a set of Pages.

## FAQ

**Do I need a Facebook account or cookies?** No. The actor reads only what Facebook serves to logged-out visitors. There is no login, cookie or API-key field.


**What is the difference between `pages` and `people`?** `pages` returns business/brand/organization Pages; `people` returns public personal profiles; `all` returns both and removes duplicates.

**Why are some fields missing?** The Page owner did not publish that information, or Facebook does not show it to logged-out visitors. The actor omits missing fields instead of guessing.

**Does the output include verified badges?** Yes. `verified` is filled when Facebook shows the badge. Use `verifiedOnly` to keep only verified results.

**Why do image URLs stop working after a few days?** Facebook serves signed CDN links that expire. Turn on `rehostImages` to copy profile and cover images into Apify storage; the originals remain in `profilePictureUrlOriginal` and `coverPhotoUrlOriginal`.

**Can I filter by category?** Set `categoryFilter` to a word from the category, such as `dentist`, `restaurant` or `clinic`. It filters the enriched rows after page details are read; with `fetchPageDetails` off, rows without a category are dropped.

**Can I filter by city?** Add a city or country to `location` or include it in the keyword (e.g. "dentist Berlin"). Facebook search is relevance-ranked, so this is a hint, not a strict filter. For Facebook’s Place filter on Pages, set `locationUids` to numeric Place IDs from `facebook.com/places/<id>/` URLs. Multiple UIDs partition the search; nearby businesses can still appear.

**How many reviews and posts can I get?** Up to 10 published reviews per Page and up to `maxPosts` recent timeline posts. Facebook does not expose the full review history or an unlimited timeline without a login.

**Can I use `profile.php?id=`, numeric IDs or handles?** Yes. `startUrls` accepts full URLs, `profile.php?id=…`, `/people/…` and `/p/…`; `startIds` accepts numeric IDs and `@handles`.


**Is this affiliated with Facebook or Meta?** No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Meta Platforms, Inc.
