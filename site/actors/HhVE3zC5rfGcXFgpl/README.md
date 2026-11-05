# Pinterest Pins, Profiles and Boards Collector with Filters

Pinterest's public pages carry far more structured data than what's visible on screen — save counts, creator stats, board metadata, shopping flags and more. This actor collects that data for Pins, profiles, boards, search results and public comments in one dataset, working from either a focused mode or a mixed list of public URLs, without requiring a Pinterest login.

## What this actor does

- **Multiple modes:** searches Pins/videos, resolves Pins, profiles, boards, board Pins, creator Pins, creator boards, comments, and mixed URLs.
- **Convenient inputs:** accepts keywords, Pin IDs, Pin URLs, usernames, profile URLs, board URLs, and aliases.
- **Source-backed filtering:** filters by saves, creator followers, verification, links, media, ads, and shopping data.
- **Large exports:** follows Pinterest cursors, removes duplicate Pins, and pushes rows progressively.
- **Truthful output:** preserves published zeroes and `false` values while omitting unavailable data.

## Output per record

Every row includes `record_type`, `source_url`, and `scraped_at`. `record_type` is `pin`, `profile`, `board`, or `comment`.

### Pin fields

Pin rows can contain `id`, `url`, `title`, `description`, `created_at`, `images`, `dominant_color`, `link`, `pinner_id`, `pinner_username`, `pinner_name`, `pinner_followers`, `board_id`, `board_name`, `board_url`, `save_count`, `comment_count`, `like_count`, `reaction_count`, `reaction_counts`, `is_video`, `video_url`, `pinId`, `pinUrl`, `imageVariants`, `imageUrl`, `videoVariants`, `mediaType`, `pinFormat`, `storyPinId`, `storyPages`, `altText`, `seoTitle`, `seoDescription`, `seoUrl`, `mobileLinkUrl`, `trackedLinkUrl`, `utmLinkUrl`, `domain`, `contentCategory`, `mediaMethod`, `creator`, `creatorId`, `creatorUsername`, `creatorName`, `creatorFollowerCount`, `originalCreator`, `board`, `boardId`, `boardName`, `boardUrl`, `boardCoverImageUrl`, `boardPinCount`, `boardFollowerCount`, `boardSectionCount`, `boardCollaboratorCount`, `boardPrivacy`, `boardCreatedAt`, `boardIsCollaborative`, `boardFollowedByMe`, `saves`, `commentCount`, `repinCount`, `shareCount`, `reactionCounts`, `reactionCount`, `likeCount`, `linkUrl`, `linkDomain`, `isPromoted`, `commentsDisabled`, `commentsEligible`, `isEligibleForImageDownload`, `isEligibleForProduct`, `isOutOfStock`, `isStaleProduct`, `isUnsafeForComments`, `isDownstreamPromotion`, `shouldMute`, `richMetadata`, `product`, `isShoppable`, `shoppingFlags`, `canonicalUrl`, `canonicalPinId`, and `canonicalPinUrl`.

`imageVariants` and `videoVariants` preserve Pinterest's available resolution names and may include `url`, `width`, `height`, `duration`, and `thumbnailUrl`. Story Pin pages may contain `pageId`, `layout`, and blocks with `type`, `text`, `images`, and `videos`.

### Profile fields

Profile rows can contain `type`, `id`, `username`, `full_name`, `bio`, `follower_count`, `following_count`, `pin_count`, `board_count`, `image_url`, `website_url`, `is_verified`, `profileId`, `profileUrl`, `fullName`, `location`, `createdAt`, `followerCount`, `followingCount`, `pinCount`, `boardCount`, `monthlyViews`, `avatarUrls`, `avatarUrl`, `websiteUrl`, `domainUrl`, `isPrivate`, `isPartner`, `isVerifiedMerchant`, `isDomainVerified`, `isTastemaker`, and `isVerified`.

### Board fields

Board rows can contain `type`, `id`, `name`, `description`, `url`, `pin_count`, `follower_count`, `privacy`, `owner_username`, `owner_name`, `boardId`, `recordType`, `createdAt`, `pinCount`, `followerCount`, `sectionCount`, `collaboratorCount`, `coverImageUrl`, `isCollaborative`, `followedByMe`, and `ownerUsername`.

### Comment fields

Comment rows can contain `id`, `text`, `created_at`, `type`, `node_id`, `like_count`, `reaction_counts`, `reaction_by_me`, `helpful_count`, `marked_helpful_by_me`, `reply_count`, `is_edited`, `is_highlighted_by_pin_owner`, `is_translatable`, `tags`, `tagged_users`, `media`, `author_name`, `author_username`, `author_id`, `author_first_name`, `author_image`, `author_image_medium`, `author_node_id`, `author_is_private_profile`, plus `pin_id`, `pin_url`, `record_type`, `source_url`, and `scraped_at`.

Missing fields are omitted. A published `0` or `false` remains real data; an omitted field means Pinterest did not provide it to the public response.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | choice | `auto` | `auto`, `pinDetail`, `comments`, `searchPins`, `searchVideos`, `userProfile`, `userPins`, `userBoards`, `boardDetail`, `boardPins`, or `search`. |
| `keywords` | array of text | — | Search phrases for Pins or videos. |
| `searchTerms` | array of text | — | Friendly alias for `keywords`. |
| `search` | string | — | One convenient search phrase. |
| `pinUrls` / `pinterestUrls` | array | — | Pin URLs or numeric Pin IDs. |
| `usernames` / `profiles` | array | — | Usernames, `@usernames`, or public profile URLs. |
| `boardUrls` / `boards` | array | — | Public board URLs or `username/board-name` paths. |
| `startUrls` | array | — | Mixed public Pinterest URLs for `auto`. |
| `maxItems` | integer | `50` | Maximum emitted records; `0` means no user cap. |
| `maxPages` | integer | `0` | Maximum feed pages; `0` follows cursors until Pinterest ends the feed. |
| `endPage` | integer | `0` | Alias for `maxPages`. |
| `minSaves` | integer | — | Keep Pins meeting this published save count. |
| `minCreatorFollowers` | integer | — | Keep Pins whose creator meets this published follower count. |
| `verifiedOnly` | boolean | `false` | Keep only explicitly verified profiles or creators. |
| `requireOutboundLink` | boolean | `false` | Keep only Pins with a Pinterest-published destination link. |
| `imageOnly` | boolean | `false` | Exclude Pins with Pinterest video data. |
| `excludeAds` | boolean | `false` | Exclude Pins explicitly marked promoted. |
| `commerceOnly` | boolean | `false` | Keep only Pins with Pinterest shopping/product data. |
| `includeComments` | boolean | `false` | Add public comment/activity records for each collected Pin. |
| `maxComments` | integer | `10` | Maximum public comments/activity rows per Pin. |
| `enrichDetails` | boolean | `false` | Request additional detail data for feed Pins. |
| `cookies` / `cookiesJson` | array/string | — | Optional Cookie-Editor export; not required for public data. |
| `proxy` / `proxyConfiguration` | object | — | Optional Apify proxy settings. |

### Example: search Pins

```json
{
  "mode": "searchPins",
  "keywords": ["small kitchen ideas", "neutral bedroom"],
  "maxItems": 200
}
```

### Example: collect a profile

```json
{
  "mode": "userProfile",
  "usernames": ["nasa", "crazylaurablog"]
}
```

### Example: large board feed

```json
{
  "mode": "boardPins",
  "boardUrls": ["https://www.pinterest.com/nasa/hubble-images/"],
  "maxItems": 10000,
  "maxPages": 0,
  "excludeAds": true
}
```

### Example: mixed URLs with comments

```json
{
  "mode": "auto",
  "startUrls": [
    "https://www.pinterest.com/pin/142567144448888376/",
    "https://www.pinterest.com/nasa/",
    "https://www.pinterest.com/nasa/hubble-images/"
  ],
  "includeComments": true,
  "maxComments": 50
}
```

## Use cases

- Content and competitor research by keyword, creator, board, or Pin.
- Finding image and video references with destination links and media variants.
- Building creator and board directories from public identity and count fields.
- Collecting public comment and engagement activity for moderation or research.
- Exporting large board or profile feeds for spreadsheet or database analysis.

## FAQ

### Do I need Pinterest cookies?

No. Public records are collected without login cookies. Optional cookies are accepted for a permitted Cookie-Editor export.

### Can I request more than 10,000 rows?

Yes. `maxItems` supports large exports, pagination is incremental, and the actor does not hold all rows until the end. The result can still be smaller when Pinterest's public source is exhausted.

### Why is a field missing from one Pin?

Pinterest does not publish every field for every Pin, account, board, or media type. Missing values are omitted rather than inferred.

### Why did a private or deleted page return no row?

The actor only emits identity and values received from Pinterest's public page or resource response. It does not fabricate a row for an inaccessible object.

### Are save and like counts estimates?

No. Counts are copied from Pinterest's published response. A zero count is preserved when Pinterest explicitly returns zero.

### Can I combine Pin, profile, and board links?

Yes. Use `auto` with `startUrls`, or select a specific mode for one kind of output.

### Can I download image or video files here?

This actor returns Pinterest media URLs and metadata. Use Pinterest Media Downloader when you need files in a key-value store.

### Does `commerceOnly` classify text as a product?

No. It keeps records with Pinterest-provided product or shopping data only.

### Is this affiliated with Pinterest?

No. This is an independent tool that reads Pinterest's public pages and is not affiliated with or endorsed by Pinterest.
