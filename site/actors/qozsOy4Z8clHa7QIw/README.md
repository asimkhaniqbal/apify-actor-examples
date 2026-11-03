# TikTok Playlist Discovery & Video Order Scraper

Scrape TikTok playlists (called "mixes" in the API) by providing playlist URLs, playlist IDs, profile URLs, or plain usernames. The actor discovers all playlists for a profile or targets specific playlists directly, then returns playlist metadata rows and video post rows with each video's position in the playlist. No login or cookies required.

## What this actor does

- Accepts playlist URLs, raw playlist IDs, profile URLs, or usernames — any combination of these inputs
- Discovers all playlists for a given profile and scrapes each one up to a configurable limit
- Emits a metadata row per playlist with name, video count, creator info, cover thumbnail, and share metadata
- Emits a post row for every video in each playlist, including the video's position within the playlist
- Supports three modes: `metadata` (playlist info only), `posts` (videos only), and `both`
- Empty fields are omitted

## Output per playlist metadata record

- `rowType` — always `"playlist"`
- `playlistId` — unique playlist/mix ID
- `playlistName` — playlist name as set by the creator
- `videoCount` — total number of videos in the playlist
- `playlistUrl` — full URL to the playlist page
- `creatorId` — creator's TikTok user ID
- `creatorUsername` — creator handle
- `creatorDisplayName` — creator display name
- `creatorVerified` — whether the creator is verified
- `coverPostId` — ID of the video used as the playlist cover
- `coverThumbnailUrl` — thumbnail image URL of the cover video
- `shareMeta.title` — share title text
- `shareMeta.desc` — share description text
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Output per post record

- `rowType` — always `"post"`
- `playlistId` — ID of the playlist this post belongs to
- `playlistName` — name of the playlist
- `positionInPlaylist` — zero-based position of this video within the playlist
- `postId` — unique TikTok video ID
- `postType` — `"video"` or `"slideshow"`
- `postUrl` — direct URL to the video
- `caption` — full caption text
- `createTime` / `createdAt` — publish timestamp (epoch and ISO 8601)
- `playCount`, `likeCount`, `commentCount`, `shareCount`, `collectCount`, `repostCount` — engagement counters (present when TikTok reports them)
- `commentStatus` — comment-permission flag
- `stitchEnabled`, `duetEnabled`, `shareEnabled` — which interactions are allowed on the video
- `isAd` — whether the video is a paid/branded ad
- `author.id` — author's TikTok user ID
- `author.username` — author handle
- `author.displayName` — author display name
- `author.verified` — verification status
- `author.avatarUrl` — author profile image URL
- `author.followerCount`, `author.followingCount`, `author.heartCount`, `author.videoCount` — author stats snapshot at scrape time
- `music.id` — sound ID
- `music.title` — sound title
- `music.authorName` — sound creator name
- `music.duration` — sound length in seconds
- `video.width` — video width in pixels
- `video.height` — video height in pixels
- `video.duration` — video duration in seconds
- `video.ratio` — resolution tier reported by TikTok (e.g. `"720p"`)
- `video.playUrl` — streamable video URL (expires)
- `video.cover` — cover/thumbnail image URL
- `mediaUrlExpiresAt` — when the signed video/cover URLs expire
- `images` — for slideshow posts, the ordered list of image URLs
- `hashtags` — array of hashtag names parsed from the caption
- `mentions` — array of @-mentioned usernames parsed from the caption
- `scrapedAt` — ISO 8601 timestamp of when the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `playlistUrls` | string[] | — | Full TikTok playlist URLs. Example: `https://www.tiktok.com/@natgeo/playlist/Cinematic-7509464647364283178` |
| `playlistIds` | string[] | — | Raw numeric playlist/mix IDs (15–20 digits). |
| `profileUrls` | string[] | — | Profile URLs — all playlists for each user will be discovered and scraped. |
| `usernames` | string[] | — | TikTok usernames (with or without `@`) — all playlists will be discovered and scraped. |
| `mode` | string | `"both"` | `"both"` returns metadata + posts. `"metadata"` returns playlist info only. `"posts"` returns videos only. |
| `maxPostsPerPlaylist` | integer | 100 | Maximum videos to fetch per playlist (1–5000). |
| `maxPlaylistsPerProfile` | integer | 20 | Maximum playlists to scrape per profile when discovering from a username or profile URL (1–100). |

### Example: discover playlists by username (metadata only)

```json
{
  "usernames": ["natgeo"],
  "mode": "metadata",
  "maxPlaylistsPerProfile": 5
}
```

### Example: scrape a specific playlist by URL

```json
{
  "playlistUrls": ["https://www.tiktok.com/@natgeo/playlist/Cinematic-7509464647364283178"],
  "mode": "both",
  "maxPostsPerPlaylist": 50
}
```

### Example: multiple profiles — full video archive

```json
{
  "usernames": ["natgeo", "nasa"],
  "mode": "both",
  "maxPostsPerPlaylist": 500,
  "maxPlaylistsPerProfile": 20
}
```

### Example: bare playlist IDs

```json
{
  "playlistIds": ["7509464647364283178", "7501234567890123456"],
  "mode": "posts",
  "maxPostsPerPlaylist": 100
}
```

## Use cases

- **Content archivists** backing up all videos from a creator's curated playlists before content is deleted
- **Media researchers** studying how creators organize their content into thematic series or collections
- **Brand teams** auditing playlist-organized product review or tutorial content from influencers
- **Talent managers** tracking playlist growth and video count over time to measure creator output
- **Developers** building structured content libraries from TikTok playlist data for client apps
- **E-learning platforms** collecting structured educational video series from knowledge-creator playlists

## FAQ

**Q: Do I need login or cookies?**  
A: No. The actor works without any credentials.

**Q: How many playlists can a creator have?**  
A: TikTok allows creators to have multiple playlists (called "mixes"). The `maxPlaylistsPerProfile` setting controls how many are scraped per profile.

**Q: What is `mode: "metadata"` useful for?**  
A: It fetches only playlist names, video counts, and creator info without loading any videos. This is the fastest way to audit which playlists a creator has and how large they are.

**Q: What does `positionInPlaylist` mean?**  
A: It is the zero-based index of the video within the playlist as returned by TikTok's API. Position 0 is the first video in the playlist.

**Q: What if a playlist has no public videos?**  
A: Playlists with no accessible videos return only a metadata row (when mode includes metadata) with `videoCount: 0` and no post rows.

**Q: Are video URLs permanent?**  
A: No. TikTok's video play and cover image URLs are signed and expire after several hours. Use `postUrl` as the stable reference.

**Q: Can I mix different input types in one run?**  
A: Yes. You can combine `playlistUrls`, `playlistIds`, `profileUrls`, and `usernames` in a single run. All are processed together.

**Q: How many videos can I get per playlist?**  
A: Up to 5,000 per playlist via the `maxPostsPerPlaylist` setting.
