# Snapchat Explore Search Across Spotlight, Lenses & Places

Search public Snapchat Explore results by keyword and export the whole result page as structured rows: Spotlight videos with engagement counts, AR lenses, user cards, public profiles, show episodes, publishers, places, and related topics. No Snapchat account, cookies, or browser install is needed.

Snapchat's Explore page is server-rendered. The actor reads that public page directly, decodes the embedded result payload, and emits one record per card with a canonical Snapchat link.

## What You Get

- **Spotlight videos** with creator, title, AI summary, keywords, views, likes, comments, shares, preview media, and thumbnail
- **AR lenses** with name, creator, official-creator flag, icon, preview image, unlock link, and canonical lens URL
- **Users** from the "Add Friends" card with username, display name, verification, story flag, avatar, and profile link
- **Public profiles** with handle, title, subtitle, description, logo, hero image, verification, story flag, and follower count
- **Show episodes** with title, publish date, thumbnail, publisher name/logo, and episode link
- **Publishers and Shows** with description, logo, follower count, and page link
- **Places** with address, category image, and place page link
- **Related topics** from the page's topic pills, with links to their topic hubs
- Every row carries `query`, `resultType`, `rank`, `scrapedAt`, and `href`

## Input

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `queries` | string[] | Yes | — | Search keywords (e.g. `food`, `new york`, `selenagomez`). Legacy `keywords`, `query`, `searchQueries` and `searchTerms` inputs are still read in code. |
| `mode` | string | No | `all` | Result family: `all`, `spotlight`, `lenses`, `users`, `profiles`, `episodes`, `publishers`, `topics`, or `places`. `users` returns both user cards and public profiles. Legacy `resultTypes` is still read in code. |
| `maxResultsPerQuery` | integer | No | `100` | Records returned per query in the selected page window (1–500). Legacy `maxResults` and `maxItems` are still read in code. |
| `page` | integer | No | `1` | Page of the query's fixed result set to return (1–100). Legacy `startPage` is still read in code. |
| `proxyConfiguration` | object | No | — | Optional proxy override. Direct requests work by default. |

### Example Input

```json
{
  "queries": ["tbilisi", "food"],
  "mode": "spotlight",
  "maxResultsPerQuery": 20,
  "page": 1
}
```

### Legacy Input

Saved tasks that use the old fields keep working. The actor maps them in code, not in the visible schema:

```json
{
  "keywords": ["food", "travel"],
  "resultTypes": ["spotlight", "lens", "topic", "user"],
  "maxResultsPerQuery": 50,
  "includeSpotlightDetails": true
}
```

`includeSpotlightDetails` is no longer needed because the current page already carries preview media and engagement counts; when it is supplied the actor logs that it was ignored.

## Output

Every record starts with the common fields below. Type-specific fields are added beside them and empty values are omitted.

| Field | Type | Description |
|-------|------|-------------|
| `query` | string | Search keyword that produced this row |
| `resultType` | string | `spotlight`, `lens`, `topic`, `user`, `snapProEntity`, `publisherEdition`, `publisher`, or `place` |
| `rank` | integer | Position in the query's sorted result set (1-based) |
| `scrapedAt` | string | ISO 8601 UTC scrape timestamp |
| `href` | string | Canonical Snapchat URL for the card |

### Spotlight fields

`storyId`, `rawSnapId`, `thumbnailUrl`, `mediaUrl`, `videoPreviewUrl`, `creatorUsername`, `creatorDisplayName`, `teaserText`, `rawCaption`, `title`, `description`, `keywords`, `viewCount`, `likeCount`, `commentCount`, `shareCount`, `gradientFrom`, `gradientTo`, plus the legacy `snapTitle`, `snapDescription`, `caption`, `postedAt`, `durationSeconds`, `width`, `height`, `boltVideoUrl`, `flatVideoUrl` when an older page supplies them.

### Lens fields

`lensId`, `lensName`, `creatorName`, `creatorIsOfficial`, `iconUrl`, `thumbnailUrl`, `deeplinkUrl`, `unlockUrl`, and the legacy `thumbnailSequence` when present.

### User fields

`userId`, `username`, `displayName`, `contextHint`, `profileLogoUrl`, `isOfficial`, `hasStory`, and the legacy `snapProId`, `isPopular`, `emoji` when present.

### Public profile fields (`snapProEntity`)

`id`, `username`, `title`, `displayName`, `subtitle`, `description`, `logoUrl`, `profileLogoUrl`, `heroImageUrl`, `isOfficial`, `isOfficialAccount`, `hasStory`, `hostAccountUsername`, `followerCount`, `subscriberCount`, and the legacy contact fields (`phoneNumber`, `address`, `emailAddress`, `whatsappNumber`, `websiteUrl`, `category`, `tier`, and so on) when an older page supplies them.

### Episode, publisher and place fields

- `publisherEdition`: `editionId`, `title`, `publishedAt`, `thumbnailUrl`, `deeplinkUrl`, `publisherName`, `publisherImageUrl`, `publisherHref`, `publisher`, plus legacy `storyDocId`.
- `publisher`: `publisherId`, `displayName`, `title`, `description`, `iconUrl`, `deeplinkUrl`, `followerCount`, `approxSubscriptionCount`, `subscriberCount`, plus legacy `isShow`, `showId`, `latestEditionId`, `businessProfileId`, `primaryColor`.
- `place`: `placeId`, `displayName`, `title`, `address`, `imageUrl`, `imageIsIcon`, plus legacy `locality`, `centerLat`, `centerLng`, `categoryName`, `categoryIconUrl`, `favoriteCount`, `isFavorited`, `boundingBox`, `storyPreview`.

### Example Output

```json
{
  "query": "food",
  "resultType": "spotlight",
  "rank": 1,
  "scrapedAt": "2026-11-07T06:59:37.000000+00:00",
  "href": "https://www.snapchat.com/@foodasmr3/spotlight/W7_EDlXWTBiXAEEniNoMPwAAYZnV3bm5oeGVlAZ84zAenAZ84y_B7AAAAAQ",
  "storyId": "W7_EDlXWTBiXAEEniNoMPwAAYZnV3bm5oeGVlAZ84zAenAZ84y_B7AAAAAQ",
  "rawSnapId": "W7_EDlXWTBiXAEEniNoMPwAAYZnV3bm5oeGVlAZ84zAenAZ84y_B7AAAAAQ",
  "creatorUsername": "foodasmr3",
  "creatorDisplayName": "FOOD",
  "title": "Mukbang Magic: Spicy Ramen & Dumplings Feast!",
  "description": "Get ready for a mouthwatering mukbang experience...",
  "keywords": ["how to eat ramen", "spicy ramen recipe", "ramen bowl"],
  "viewCount": 68553,
  "likeCount": 5972,
  "commentCount": 83,
  "shareCount": 1530,
  "thumbnailUrl": "https://cf-st.sc-cdn.net/d/...",
  "videoPreviewUrl": "https://cf-st.sc-cdn.net/d/...",
  "mediaUrl": "https://cf-st.sc-cdn.net/d/...",
  "gradientFrom": "#2b2b35",
  "gradientTo": "#16191c"
}
```

## FAQ

**Does this need a Snapchat login or cookies?**
No. Explore search is public and the actor reads the server-rendered page without an account.

**How many rows does one query return?**
Snapchat serves one fixed set per query. In current tests the page carries about 22 Spotlight videos, 24 lenses, 24 profiles, 15 episodes, 14 publishers, 10 places, and one user card. Totals vary by query and locale.

**Can I go past the first page?**
Snapchat does not expose a page or cursor parameter for Explore search on the web, so there is no second upstream page. The `page` input splits the fixed result set into deterministic windows: page 1 is the first `maxResultsPerQuery` rows, page 2 the next batch. For a different result set, change the query.

**What is the difference between `users` and `profiles`?**
`users` combines the "Add Friends" user cards and the public `profiles` section. `profiles` returns only the public profile cards (`resultType: "snapProEntity"`).

**Why are some fields missing on a row?**
Fields that Snapchat does not publish for a card are omitted rather than set to null. Older pages still provide the legacy Spotlight card map and Snap Pro contact fields; those values are kept when present.

**What are the topic rows?**
They are the related topic pills rendered beside the results. Each row links to the topic hub and is tagged `resultType: "topic"`.

**Can I query several keywords in one run?**
Yes. Add one item per keyword to `queries`; rows from each keyword carry that keyword in the `query` field.

**Is a proxy required?**
No. Direct access works. The optional proxy field exists only for users who want to route through their own Apify Proxy settings.

**Is this affiliated with Snapchat?**
No. It is an independent tool that reads public Snapchat Explore pages and is not operated or endorsed by Snap Inc.
