# TikTok Playlist & Collection Scraper

Collect every public video in a TikTok playlist or collection, in the creator's order, plus the playlist details that make the data usable: cover image, owner, owner stats, view/like/comment counts, sound metadata, media URLs, hashtags, mentions and slideshow images.

Give it a playlist URL, a collection URL, a numeric ID, a profile URL or a plain `@handle`. It returns one row per playlist or collection, one row per video, and clear status rows when a target cannot be read. No login, cookies or proxy setup are needed.

## What this actor does

- Scrapes **playlists** (`/@user/playlist/...`) and **public collections** (`/@user/collection/...`)
- Accepts full URLs, numeric playlist/collection IDs, profile URLs and handles — mix them in one run
- Returns the **full ordered video list**, not just the first visible page
- Discovers every public playlist on a profile, up to a limit you set
- Adds the playlist cover, creator identity, creator stats, `shareMeta` for collections and a `scrapedAt` timestamp
- Captures slideshow posts as slideshows with their ordered image URLs
- Reports invalid, private, empty, blocked and partial targets as `rowType: "status"` rows
- Deduplicates videos by post ID and omits empty fields at every depth

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `playlistUrls` | string[] | – | Playlist URLs. Collection URLs pasted here are detected automatically. |
| `playlistIds` | string[] | — | Numeric playlist IDs (15–20 digits). A collection ID is retried on the collection route. |
| `collectionUrls` | string[] | — | Public collection URLs. |
| `collectionIds` | string[] | — | Numeric collection IDs. |
| `profileUrls` | string[] | — | Profile URLs; every public playlist on the profile is discovered. |
| `usernames` | string[] | — | Handles with or without `@`. |
| `mode` | select | `both` | `metadata`, `posts` or `both`. |
| `maxPostsPerPlaylist` | integer | 100 | Videos per playlist/collection (1–5000). Raise it for large playlists. |
| `maxPlaylistsPerProfile` | integer | 20 | Playlists processed per profile (1–100). |
| `maxItems` | integer | 0 | Global row cap across all targets and status rows. `0` means no cap. |

### Example: one playlist, metadata only

```json
{
  "playlistUrls": ["https://www.tiktok.com/@natgeo/playlist/Cinematic-7509464647364283178"],
  "mode": "metadata",
  "maxPostsPerPlaylist": 1
}
```

### Example: full playlist in creator order

```json
{
  "playlistUrls": ["https://www.tiktok.com/@natgeo/playlist/Cinematic-7509464647364283178"],
  "mode": "posts",
  "maxPostsPerPlaylist": 500
}
```

### Example: a public collection by URL or ID

```json
{
  "collectionUrls": ["https://www.tiktok.com/@aubreyisamess/collection/places%20to%20go-7154034669288196891"],
  "mode": "both",
  "maxPostsPerPlaylist": 1000
}
```

### Example: discover a profile's playlists

```json
{
  "usernames": ["natgeo"],
  "mode": "metadata",
  "maxPlaylistsPerProfile": 20
}
```

### Example: mixed targets with a global cap

```json
{
  "playlistIds": ["7509464647364283178"],
  "collectionIds": ["7154034669288196891"],
  "usernames": ["natgeo"],
  "mode": "both",
  "maxPostsPerPlaylist": 500,
  "maxPlaylistsPerProfile": 5,
  "maxItems": 250
}
```

## Output

All rows share `rowType`, `scrapedAt` and (when applicable) `playlistId`, `playlistName`, `playlistType` and `playlistUrl`.

### Playlist / collection row (`rowType: "playlist"`)

- `playlistType` — `"playlist"` or `"collection"`
- `playlistId`, `playlistName`, `videoCount`
- `playlistUrl` — canonical TikTok URL
- `playlistCoverUrl` and `coverThumbnailUrl` — cover image (signed, expiring)
- `coverPostId` — only when TikTok names a cover video
- `creatorId`, `creatorUsername`, `creatorDisplayName`, `creatorVerified`, `creatorAvatarUrl`
- `creatorStats` — `followerCount`, `followingCount`, `heartCount`, `videoCount`
- `shareMeta` — `title` and `desc` for public collections
- `scrapedAt` — ISO 8601 UTC timestamp

### Video row (`rowType: "post"`)

- `playlistId`, `playlistName`, `playlistType`, `playlistUrl`, `positionInPlaylist` (zero-based, source order)
- `postId`, `postType` (`video` or `slideshow`), `postUrl`
- `caption`, `textLanguage`, `createTime`, `createdAt`
- `playCount`, `likeCount`, `commentCount`, `shareCount`, `collectCount`, `repostCount`
- `commentStatus`, `shareEnabled`, `duetEnabled`, `stitchEnabled`
- `isAd`, `isECVideo`, `privateItem`, `forFriend`, `originalItem`, `isAigc`, `AIGCDescription`
- `author` — ID, secure ID, username, display name, bio, verified, avatar, follower/following/heart/video counts
- `music` — ID, title, author, duration, original flag, cover URL and play URL when TikTok exposes it
- `video` — dimensions, duration, ratio, definition, codec, bitrate info, play/download/cover URLs
- `images` — ordered slideshow images with width, height and URL list
- `hashtags`, `mentions`
- `mediaUrlExpiresAt` — when signed video/cover URLs expire
- `scrapedAt`

### Status row (`rowType: "status"`)

- `status` — `invalid`, `not_found`, `private`, `empty`, `blocked` or `partial`
- `input` — the URL, ID or handle that produced the row
- `playlistId`, `playlistName`, `playlistType` when known
- `message` — plain-language explanation
- `scrapedAt`

## Ordering and completeness

`positionInPlaylist` follows the order TikTok serves for that playlist or collection. It is not sorted by publish date, because creators deliberately order series, tutorials and trip lists. Every page is read until the source reports no more videos or your `maxPostsPerPlaylist` / `maxItems` limit is reached.

Collections can report more videos than are publicly accessible (deleted, private or region-locked items are still counted). The actor returns every accessible video once and never invents filler rows.

## Use cases

- Archiving a creator's series before videos disappear
- Researching how publishers group content into themed playlists
- Auditing collection-based campaigns and their engagement
- Building ordered video libraries from travel, education or recipe collections
- Tracking creator output, follower growth and playlist size over time
- Feeding ordered short-form video datasets into a CMS, dashboard or model

## FAQ

**Do I need cookies or a login?**
No. The actor reads public pages without credentials.

**Do I need to configure a proxy?**
No. The actor chooses its own connection automatically.

**Why is `positionInPlaylist` not sorted by date?**
Playlists and collections have a creator-defined order. The position preserves that order, which is usually the point of scraping a playlist.

**How many videos will I get?**
All publicly accessible videos, up to `maxPostsPerPlaylist` per target and `maxItems` overall. A playlist reported as 34 videos with 34 public items returns 34 rows.

**Why does a collection say 651 videos but return fewer?**
TikTok counts unavailable entries in the collection total. Only publicly accessible videos can be returned.

**What happens with a bad or private target?**
The run finishes successfully and emits a status row explaining the outcome instead of a fake playlist row.

**Are the video and image URLs permanent?**
No. TikTok's CDN URLs are signed and expire after a few hours. Use `postUrl` as the stable reference and re-run when you need fresh media URLs.

**Can I combine playlists, collections and profiles in one run?**
Yes. Add any mix of the input arrays. Duplicate targets are processed once.

**What does `maxItems` count?**
Every dataset row, including playlist, video and status rows.
