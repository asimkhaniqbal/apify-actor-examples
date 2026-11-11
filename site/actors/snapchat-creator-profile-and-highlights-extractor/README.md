# Snapchat Creator Profile & Highlights Extractor

Paste a list of Snapchat usernames or profile links and receive one structured record per account: identity, bio and links, subscriber and verification signals, curated highlights, Spotlight videos with captions and engagement, active story snaps, lenses, contacts and the accounts Snapchat lists as related. No Snapchat login, cookies or API key is needed.

## What this actor does

- Normalises bare usernames, `@username`, `/add/` links and `https://www.snapchat.com/@username` links, then reads the public profile page once per account.
- Builds a flat, joinable record instead of dumping raw page data: identity, coverage, flags, counts, highlights, Spotlight, story, lenses, comments, contacts and related accounts.
- Merges each Spotlight card with its metadata, so every entry carries the real `storyId`, `spotlightUrl`, view/share/comment/boost/recommend counts, caption, hashtags, AI title/description/keywords and a transcript URL when one exists.
- Drops empty Spotlight slots, placeholder rows, `1970` timestamps, negative sentinel counts and empty strings at every depth. A field the source genuinely lacks is omitted, never zero-filled.
- Derives `hasStory` from the actual active story snaps instead of the stale page flag.
- Can expand one hop into related accounts, with `isSeed`, `sourceUsername` and `discoveryDepth` recorded so seed rows and discovered rows stay distinguishable.
- Supports optional full highlight snap lists with a per-highlight snap cap, and caps the total number of profile records.

## Output per profile

### Core fields

| Field | Description |
|---|---|
| `username` | Snapchat username. |
| `displayName` | Public display name. |
| `accountType` | `"public"` or `"private"` (personal account). |
| `profileStatus` | `"public"`, `"personal"`, `"not_found"` or `"invalid"`. |
| `recordType` | `"profile"` for profile rows, `"error"` for failed entries. |
| `isVerified` | Whether the account carries Snapchat's verification badge. |
| `subscriberCount` | Subscriber count when Snapchat publishes it. Hidden counts are omitted. |
| `bio` / `websiteUrl` | Profile bio and linked website, when present. |
| `profileUrl` | Canonical profile URL (`/@username`). |
| `profilePictureUrl` / `snapcodeImageUrl` / `squareHeroImageUrl` | Avatar, Snapcode and hero image URLs. |
| `category` / `subcategory` / `publisherType` | Content category and publisher identifiers. |
| `businessProfileId` / `hostUserId` | Business profile and host identifiers, when exposed. |
| `address` / `mutableName` / `primaryColor` | Optional profile presentation fields. |
| `shouldHideUsername` | Whether Snapchat hides the username in its own UI. |
| `hasStory` / `hasCuratedHighlights` / `hasSpotlightHighlights` | Whether each content type is present now. |
| `storySnapCount` / `curatedHighlightCount` / `spotlightCount` / `lensCount` / `relatedAccountCount` | Flat counts. `storySnapCount` and `lensCount` describe the profile even when those sections are disabled; `curatedHighlightCount`, `spotlightCount` and `relatedAccountCount` reflect the entries returned after caps. |
| `createdAt` / `lastUpdatedAt` | Account creation and last-update timestamps in ISO 8601 UTC. |
| `sameAsLinks` | Cross-platform links listed for the profile. |
| `contacts` | Locally parsed emails, phones, social handles, Linktree and website URLs. |
| `engagementSummary` | Spotlight totals for the returned entries (after `maxHighlightsPerUser`): views, shares, comments, boosts, recommends and average views. |
| `isSeed` / `sourceUsername` / `discoveryDepth` | `isSeed` is set on every profile row; `sourceUsername` and `discoveryDepth` appear on discovered rows. |
| `scrapedAt` | Timestamp this actor fetched the record (ISO 8601 UTC). |

### `curatedHighlights[]`

Each entry keeps `highlightId`, `storyTitle`, `thumbnailUrl`, `snapCount`, `firstSnapUrl` and `firstSnapType`. With `includeHighlightSnaps` enabled, each entry also contains `snaps[]` with `snapIndex`, `snapId` (only when non-empty), `mediaUrl`, `mediaPreviewUrl`, `mediaType`, `postedAt`, `snapTitle`, `lat`, `lng`, `isSponsored`, `hasAttachment` and `transcriptUrl`.

### `spotlightHighlights[]`

The canonical Spotlight list. Every entry has a `storyId` and `spotlightUrl`, plus `thumbnailUrl`, `mediaUrl`, `mediaPreviewUrl`, `mediaType`, `transcriptUrl`, `uploadedAt`, `durationMs`, `width`, `height`, `caption`, `hashtags`, `viewCount`, `shareCount`, `commentCount`, `boostCount`, `recommendCount`, creator fields, `aiTitle`, `aiDescription`, `aiKeywords`, `contentKeywords`, `contextCards` and `detectedLanguages` when the page provides them.

### `spotlightMetadata[]` (when `includeSpotlightMeta` is true)

A backward-compatible metadata array with the same placeholder and sentinel fixes, plus `storyId` and `spotlightUrl`. It is capped by `maxHighlightsPerUser`. `spotlightHighlights[]` is the recommended, merged view.

### `activeStorySnaps[]` and `activeStoryMeta` (when `includeActiveStory` is true)

Per snap: `snapIndex`, `snapId`, `mediaUrl`, `mediaPreviewUrl`, `mediaType`, `postedAt`, `snapTitle`, `lat`, `lng`, `hasAttachment`, `audioTranscriptionObjectUrl`, `isSponsored` and `storySource`. Story-level metadata appears in `activeStoryMeta` when the page exposes it.

### `lenses[]` (when `includeLenses` is true)

`scannableUuid`, `lensName`, `lensCreatorDisplayName`, `lensCreatorUsername`, `lensId`, `lensPreviewImageUrl`, `lensPreviewVideoUrl`, `iconUrl`, `unlockUrl`, `userProfileUrl`, `lensCreatorSearchTags`, `isOfficialSnapLens`, `lastUpdatedAt` and `lensUrl`. The UUID is recovered from the unlock link when the page omits the dedicated field.

### `spotlightComments[]` (when `includeComments` is true)

`commentId`, `snapId`, `replyText`, `reactionCounts`, `likeCount`, `commenterDisplayName`, `commenterBitmojiAvatarId`, `commenterBitmojiSelfieId`, `commenterProfileLogoUrl`, `commenterId`, `commenterProfileId`, `approvalState`, `postedAt`, `updatedAt`, `rankingScore`, `threadedReplyCount` and `reportCount`. Duplicate comments are removed within a profile.

### `relatedAccounts[]`

`username`, `displayName`, `profileUrl`, `profilePictureUrl`, `isVerified`, `subscriberCount` (only when published), `bio`, `snapcodeImageUrl`, `category`, `websiteUrl`, presence flags and `subscribeLink` with `oneLinkBaseUrl`, `deepLinkUrl` and `iosAppStoreUrl`.

### Personal and error records

Personal accounts return `username`, `displayName`, `profileUrl`, `snapcodeImageUrl`, optional Bitmoji URLs, `accountType: "private"`, `profileStatus: "personal"`, `isVerified: false`, `recordType` and `scrapedAt`.

An unknown or unparseable profile returns `inputUsername`, `profileStatus`, `recordType: "error"`, `error` and `scrapedAt`. Error rows count toward `maxItems`.

### Legacy `cursors`

`cursors` is retained for compatibility: `curatedHighlightsCursor`, `spotlightHighlightsCursor` and `lensCursor`. It is a raw cursor object and is not required to read the records.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `usernames` | Array of strings | – | Required. Snapchat usernames or profile URLs: `brentrivera`, `@brentrivera`, `https://www.snapchat.com/add/brentrivera`, `https://www.snapchat.com/@brentrivera`. |
| `includeLenses` | Boolean | `true` | Include lenses published by the profile. |
| `includeSpotlightMeta` | Boolean | `true` | Merge Spotlight metadata into `spotlightHighlights[]` and keep the legacy metadata array. |
| `includeActiveStory` | Boolean | `true` | Include the current public story snaps. |
| `includeHighlightSnaps` | Boolean | `false` | Add the full snap list to each curated highlight. |
| `includeComments` | Boolean | `false` | Include embedded Spotlight comments. |
| `maxRelatedAccounts` | Integer | – | Cap related accounts per profile. `0` excludes them. |
| `maxHighlightsPerUser` | Integer | – | Cap curated highlights, Spotlight entries and legacy `spotlightMetadata[]` rows per profile. `0` excludes all three; counts and engagement totals follow the capped list. |
| `maxSnapsPerHighlight` | Integer | `0` | With `includeHighlightSnaps`, cap snaps per highlight. `0` means no cap, maximum `500`. |
| `maxItems` | Integer | – | Stop after this many profile rows, including error rows. |
| `relatedProfilesLimit` | Integer | `0` | Scrape up to this many unique related accounts after the supplied profiles. One hop only. |
| `proxyConfiguration` | Object | – | Optional proxy configuration. Public profiles work without one. |

### Example: quick profile lookup

```json
{
  "usernames": ["brentrivera", "https://www.snapchat.com/@khaby.lame"]
}
```

### Example: lightweight run

```json
{
  "usernames": ["teamsnapchat"],
  "includeLenses": false,
  "includeSpotlightMeta": false,
  "includeActiveStory": false,
  "maxRelatedAccounts": 0
}
```

### Example: full enrichment with comments and highlight snaps

```json
{
  "usernames": ["kyliejenner"],
  "includeLenses": true,
  "includeSpotlightMeta": true,
  "includeActiveStory": true,
  "includeHighlightSnaps": true,
  "maxSnapsPerHighlight": 10,
  "includeComments": true,
  "maxHighlightsPerUser": 5
}
```

### Example: related-account discovery

```json
{
  "usernames": ["nba"],
  "relatedProfilesLimit": 3,
  "maxItems": 4
}
```

## Use cases

- **Influencer vetting** — check verification, published subscriber counts, recent Spotlight performance and the links a creator lists.
- **Campaign benchmarking** — compare total Spotlight views, shares and comments across a roster of accounts.
- **Contact discovery** — collect public emails, phone numbers and social handles from bios and linked pages without extra requests.
- **Content research** — export captions, hashtags, AI titles and transcript URLs for a creator's current Spotlight catalog.
- **Network mapping** — expand one hop into the accounts Snapchat associates with a brand, with provenance on every discovered row.
- **Media archiving** — capture highlight, story and Spotlight media URLs for a point-in-time snapshot.

## FAQ

**Do I need a Snapchat account or cookies?**
No. The actor reads the same public profile page that opens in a browser without signing in.

**Why is `subscriberCount` missing on some records?**
Snapchat hides the count on some accounts. A hidden count is omitted rather than written as `0`. When Snapchat publishes a count it is rounded by Snapchat, commonly to the nearest 100.

**Why does a Spotlight entry have no view count?**
Some entries have no public engagement data. A count is included only when Snapchat supplies a non-negative value; negative sentinels are dropped.

**What is the difference between `curatedHighlights` and `spotlightHighlights`?**
Curated highlights are albums saved from the creator's story. Spotlight highlights are Spotlight videos pinned to the profile. Both are limited to what the public profile page exposes.

**How many highlights, Spotlights, lenses or related accounts can a profile have?**
The public page currently exposes about 30 Spotlight slots, about 18 curated highlights, up to 16 lenses and up to 3 related accounts per profile. The actor reports exactly what the page returns; it does not promise full history.

**Why are some media URLs no longer playable later?**
Snapchat's CDN links can carry expiring tokens. Download or cache media promptly after a run.

**Does a personal account return highlights or stories?**
Personal accounts expose very little publicly. They return identity, Snapcode and Bitmoji assets when available, and `profileStatus: "personal"` explains the reduced record.

**How does related-account expansion work?**
Set `relatedProfilesLimit` above `0`. The actor reads the related list from the supplied profiles, de-duplicates usernames, and scrapes one hop only. Discovered profiles keep `isSeed: false`, `sourceUsername` and `discoveryDepth: 1`.

**Are empty fields written as null?**
No. Empty strings, nulls, empty arrays and empty objects are removed at every depth. Legitimate `false` values and numeric `0` from the source are preserved.

**Is this affiliated with Snapchat?**
No. This actor reads public profile pages and is not affiliated with, endorsed by, or supported by Snap Inc.

## Limitations

- Subscriber counts are rounded by Snapchat and hidden on some accounts.
- The profile page exposes a fixed batch of highlights and Spotlight entries, not the account's full history.
- Full comment-thread pagination and transcript text are owned by separate actors; this actor returns the embedded comments and transcript URLs only.
- Personal accounts expose minimal public data.
- Snapchat can change its page format at any time. When the page cannot be parsed, the actor returns an explicit error row instead of invented values.
