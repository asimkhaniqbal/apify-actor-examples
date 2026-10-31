# TikTok Sound Lookup: Metadata & Videos Using It

Give this actor a TikTok sound — by page URL or bare numeric ID — and it returns two things: a metadata row describing the sound itself (title, creator, duration, cover art, streaming links), and a row for every video currently using that sound, complete with caption, hashtags, and engagement counts. No TikTok login or cookies needed.

## What this actor does

- Resolves TikTok music/sound page URLs or raw numeric sound IDs
- Emits one metadata row per sound with title, creator, duration, cover images, video count, and (for licensed tracks) streaming-service links
- Paginates the sound's "videos using this sound" list and emits one row per video, up to your limit
- Lets you fetch metadata only, posts only, or both independently
- Omits any field TikTok did not return for that record — no nulls or empty placeholders

## Output per sound metadata row (`rowType: "music"`)

- `musicId` — unique TikTok sound ID
- `musicTitle` — sound/track title
- `authorName` — credited creator name
- `duration` / `shootDuration` — track length and the clip length TikTok allows creators to use
- `isCopyrighted` / `isCommerceMusic` / `isUnlimitedMusic` / `isOriginal` / `isPrivate` — flags describing the sound's licensing and visibility
- `album` — album name, for commercial tracks that have one
- `playUrl` — streamable (time-limited) audio URL
- `coverLarge` / `coverMedium` / `coverThumb` — cover art at three sizes
- `videoCount` — total videos on TikTok currently using this sound
- `musicUrl` — canonical TikTok page URL for the sound
- `dspLinks` — streaming-platform links (Spotify, Apple Music, etc.), present only for licensed commercial tracks
- `shareMeta` — object with `title` and `desc`, the text TikTok uses when the sound page is shared
- `author` — object with `id`, `username`, `displayName`, `verified`, `avatarUrl`, `followerCount`, `heartCount` for the sound's credited creator

## Output per video row (`rowType: "post"`)

- `postId`, `postType`, `postUrl` — video identifier, type (`video` or `slideshow`), and link
- `matchedMusicId` — the sound ID this video was found under
- `caption`, `textLanguage` — video caption text and its detected language
- `locationCreated`, `createTime`, `createdAt` — where and when the video was posted
- `stitchEnabled`, `shareEnabled`, `duetEnabled`, `isAd` — platform interaction flags
- `playCount`, `likeCount`, `commentCount`, `shareCount`, `collectCount`, `repostCount` — engagement counters
- `hashtags` — array of `{name, id, isCommerce}` entries parsed from the caption
- `mentions` — array of `{username, start, end, awemeId}` entries for @-mentions in the caption
- `author` — object with `id`, `secUid`, `username`, `displayName`, `bio`, `verified`, `privateAccount`, `roomId`, `avatarUrl`, `followerCount`, `followingCount`, `heartCount`, `videoCount`
- `music` — object with `id`, `title`, `authorName`, `original`, `duration`, `album`, `playUrl`, `coverUrl` for the sound used in this video
- `video` — object with `width`, `height`, `duration`, `ratio`, `playUrl`, `cover` describing the video file itself

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `musicUrls` | array | – | TikTok music page URLs, e.g. `https://www.tiktok.com/music/original-sound-7595604258835401494` |
| `musicIds` | array | – | Bare numeric sound IDs (15–20 digits), as an alternative to full URLs |
| `maxPostsPerSound` | integer | `30` | Maximum videos to collect per sound (0–500); `0` skips video collection entirely |
| `includeMusicMetadata` | boolean | `true` | Emit the metadata row for each sound |
| `includePosts` | boolean | `true` | Emit video rows for each sound |

### Example: look up one sound by URL

```json
{
    "musicUrls": ["https://www.tiktok.com/music/original-sound-7595604258835401494"],
    "maxPostsPerSound": 30
}
```

### Example: multiple sounds by ID

```json
{
    "musicIds": ["7595604258835401494", "7234567890123456789"],
    "maxPostsPerSound": 50
}
```

### Example: metadata only, skip videos

```json
{
    "musicIds": ["7595604258835401494"],
    "includeMusicMetadata": true,
    "includePosts": false
}
```

### Example: large video pull for a viral sound

```json
{
    "musicUrls": ["https://www.tiktok.com/music/original-sound-7595604258835401494"],
    "includeMusicMetadata": false,
    "includePosts": true,
    "maxPostsPerSound": 500
}
```

## Use cases

- **A&R and label teams** tracking how a track is spreading on TikTok and which creators are driving it
- **Brand and marketing teams** scouting trending sounds before planning a campaign video
- **Trend researchers** pulling every video using a sound to study format and content patterns
- **Rights-management teams** auditing how widely a licensed or copyrighted track is being reused
- **Creator-discovery platforms** finding accounts that build an audience around a specific sound or niche

## FAQ

**Do I need a TikTok account or cookies?** No — sound metadata and the videos-using-this-sound list are both public data, so no login is required.

**Where do I find a sound's numeric ID?** Open the sound's page on TikTok; the URL ends in `.../music/<name>-<digits>`. The trailing digits are the ID — you can pass the full URL or just that number.

**Why is `dspLinks` missing for some sounds?** Streaming-platform links only exist for licensed commercial tracks; sounds a user recorded themselves (`isOriginal: true`) never have them.

**Can I get just the sound's stats without pulling any videos?** Yes — set `includePosts` to `false` and leave `includeMusicMetadata` as `true`.

**Why don't post rows carry a `scrapedAt` timestamp?** This actor keeps the record shape close to what TikTok returns and doesn't add that bookkeeping field; use the dataset's own run metadata if you need a collection timestamp.

**Are `playUrl` links permanent?** No — TikTok signs its audio and video URLs with a short-lived token, so treat them as temporary and download promptly if you need the actual media file.

**What does `matchedMusicId` tell me?** It ties every video row back to the sound it was collected under in that run, which matters when you request multiple sounds at once.

**Is this affiliated with TikTok?** No. This is an independent actor that reads TikTok's public sound and video data; it is not endorsed by or affiliated with TikTok or ByteDance.
