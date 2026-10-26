# YouTube Video Metadata & Chapter Extractor

Give this actor a YouTube video or Short link (or just the 11-character ID) and get back the watch page as structured data: core metadata, channel info, chapters, endscreen elements, the list of caption tracks, a timed transcript when YouTube serves one, related videos, and comments.

## What this actor does

- **Video metadata:** title, description, channel, views, likes, comment count, duration, dates, tags, hashtags, category, thumbnails and playability flags
- **Optional sections:** chapters, endscreen, info cards, caption-track list, related videos and comments, each switched on or off in the input
- **Engagement rate:** (likes + comments) ÷ views, calculated per video
- **Bulk videos:** one row per input link, processed one after another
- **Empty fields are omitted** — records only contain values YouTube actually returned (failure rows are the exception, see below)

## Output

Each row in the dataset represents one input link.

### Video fields

| Field | Type | Example |
|-------|------|---------|
| `type` | string | `"video"` or `"short"` |
| `inputUrl` | string | Original URL provided in input |
| `videoId` | string | `"dQw4w9WgXcQ"` |
| `title` | string | `"Rick Astley - Never Gonna Give You Up"` |
| `description` | string | Full video description |
| `channelId` | string | `"UCuAXFkgsw1L7xaCfnd5JJOw"` |
| `channelName` | string | `"Rick Astley"` |
| `publishedDate` | string | `"2009-10-25"` |
| `uploadDate` | string | Upload date as YouTube reports it |
| `durationSeconds` | integer | `212` |
| `viewCount` | integer | `1234567890` |
| `likeCount` | integer | `15000000` |
| `commentCount` | integer | `500000` |
| `commentsDisabled` | boolean | `false` |
| `tags` | array | `["rick astley", "never gonna give you up"]` |
| `hashtags` | array | `["#rickastley"]` |
| `thumbnails` | object | Keys among `default`, `medium`, `high`, `standard`, `maxres` |
| `category` | string | `"Music"` |
| `defaultAudioLanguage` | string | `"en"` |
| `liveStatus` | string | `"live_now"`, `"was_live"` or `"upcoming"`; omitted for regular uploads |
| `liveBroadcastDetails` | object | Start/end timestamps for live streams |
| `isUpcoming` | boolean | `false` |
| `isUnlisted` | boolean | `false` |
| `uploadType` | string | `"normal"`, `"live"`, `"was_live"`, `"premiere"` or `"short"` |
| `engagementRate` | float | `0.0162` |
| `resolution` | string | Highest available quality, e.g. `"1080p"` or `"4K"` |
| `isFamilySafe` | boolean | `true` |
| `isShortsEligible` | boolean | `true` |
| `isPlayable` | boolean | `true` |
| `isAgeGated` | boolean | `false` |
| `requiresLogin` | boolean | `false` |
| `playabilityStatus` | string | `"OK"` |
| `availableCountries` | array | `["US", "GB", "CA", ...]` |
| `clipId`, `clipStartMs`, `clipEndMs` | string / integer | Only when YouTube's player data carries clip settings |
| `transcript` | array | Timed segments `{start, dur, text}` (seconds as strings), when YouTube serves a transcript for the video |
| `success` | boolean | `true` |
| `scrapedAt` | string | `"2026-02-10T12:00:00.000000+00:00"` |

### Failure rows

When a link can't be parsed, the video is private, removed or region-blocked, or every fetch attempt is blocked, the actor still writes a row for that link with `success: false` and an `error` message (for example `"Invalid YouTube video URL"` or `"Video not available: ..."`). Those rows carry `type`, `inputUrl`, `scrapedAt`, `playabilityStatus` (often `"ERROR"`), and the flags `isPlayable`, `isAgeGated`, `requiresLogin`, `isUpcoming`, `isUnlisted` and `commentsDisabled` set to `false` unless YouTube said otherwise. When the video ID is known they also include `videoId` and `url`, and any title, channel, view, duration or like data YouTube still returned.

### Channel info (embedded in `channel` object)

| Field | Type | Example |
|-------|------|---------|
| `id` | string | `"UCuAXFkgsw1L7xaCfnd5JJOw"` |
| `name` | string | `"Rick Astley"` |
| `handle` | string | `"@rickastley"` |
| `url` | string | `"https://www.youtube.com/@rickastley"` |
| `subscriberCount` | integer | `2500000` |
| `subscriberCountText` | string | `"2.5M subscribers"` |
| `logo` | string | Channel avatar URL |
| `badges` | array | `["Verified"]` |

### Chapters (`chapters` array, when `includeChapters` is on)

| Field | Type | Example |
|-------|------|---------|
| `title` | string | `"Intro"` |
| `startSeconds` | integer | `0` |
| `startTimeText` | string | `"0:00"` |
| `thumbnailUrl` | string | Chapter thumbnail URL |

### Endscreen (`endscreen` array, when `includeEndscreen` is on)

Each item has `style` (video, channel, playlist or link type), `title`, `url`, and when available `videoId`, `channelId`, `thumbnailUrl`, `startMs` and `endMs`.

### Info cards (`cards` array, when `includeCards` is on)

Each card has `type` and, when available, `videoId` and `title`.

### Caption tracks (`captionTracks` array, when `includeTranscriptMetadata` is on)

| Field | Type | Example |
|-------|------|---------|
| `languageCode` | string | `"en"` |
| `languageName` | string | `"English"` |
| `isAutoGenerated` | boolean | `false` |
| `isTranslatable` | boolean | `true` |

### Related videos (`relatedVideos` array, when `includeRelatedVideos` > 0)

Each entry has `videoId` and `title`, plus `url`, `channelName` and `viewCountText` when the sidebar shows them. Shorts from the sidebar shelf carry `type: "short"` instead.

### Comments (`comments` array, when `maxComments` > 0)

| Field | Type | Example |
|-------|------|---------|
| `commentId` | string | `"abc123"` |
| `text` | string | `"This song is timeless!"` |
| `author` | string | `"MusicFan"` |
| `authorChannelId` | string | `"UC..."` |
| `authorThumbnail` | string | Profile image URL |
| `publishedTime` | string | `"2 years ago"` |
| `likes` | integer | `1200` |
| `isHearted` | boolean | `false` |
| `isPinned` | boolean | `false` |
| `isVerified` | boolean | `false` |
| `isCreator` | boolean | `false` |
| `isArtist` | boolean | `false` |

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `videoUrls` | array | – | Required. YouTube watch, `youtu.be`, Shorts, `/live/`, `/embed/` or `/v/` links, or plain 11-character video IDs |
| `maxComments` | integer | `0` | Comments to collect per video (0–1000). `0` skips comments |
| `includeChapters` | boolean | `true` | Add chapter markers from the chapter panel or description timestamps |
| `includeEndscreen` | boolean | `true` | Add endscreen elements (videos, channels, playlists or links shown at the end) |
| `includeCards` | boolean | `false` | Add the info cards embedded in the video |
| `includeTranscriptMetadata` | boolean | `true` | Add the list of caption tracks. It does not control the `transcript` field, which is added whenever YouTube serves a transcript |
| `includeRelatedVideos` | integer | `0` | How many sidebar related videos to add (0–50). `0` skips them |
| `proxyCountry` | string | `US` | Country to route requests through — one of 31 countries in the list (e.g. `US`, `GB`, `DE`, `JP`, `IN`, `BR`). Useful for region-locked videos |

### Example: core details

```json
{
  "videoUrls": ["https://www.youtube.com/watch?v=dQw4w9WgXcQ"]
}
```

### Example: details with chapters and comments

```json
{
  "videoUrls": ["https://www.youtube.com/watch?v=9bZkp7q19f0"],
  "includeChapters": true,
  "includeEndscreen": true,
  "maxComments": 20
}
```

### Example: Shorts and IDs with related videos

```json
{
  "videoUrls": ["https://www.youtube.com/shorts/dQw4w9WgXcQ", "jNQXAC9IVRw"],
  "includeTranscriptMetadata": true,
  "includeRelatedVideos": 10,
  "proxyCountry": "GB"
}
```

## Use cases

- **Content research** — pull chapters, tags and engagement rates for a hand-picked list of videos
- **SEO analysis** — collect titles, descriptions, hashtags, and related videos for keyword and ranking research
- **Media monitoring** — re-run the same video list to track view counts, like counts, and playability status over time
- **Competitor review** — compare a set of a competitor's videos on categories, endscreen strategy, and caption availability
- **Subtitles and NLP** — see which caption languages exist and read the timed transcript where YouTube serves one
- **Rights and availability checks** — check `availableCountries`, `isAgeGated`, and `playabilityStatus` for content rights workflows

## FAQ

**Does this actor return transcript text?**
Sometimes. Whenever YouTube serves a transcript for the video, it's added as `transcript` (timed segments). Many videos have no transcript available to signed-out visitors, so the field is often missing. `captionTracks` lists the caption languages separately.

**Why is `commentCount` missing or `comments` empty?**
Comments are only collected when `maxComments` is above 0, and only if YouTube serves the comment section; videos with comments turned off return none. `commentCount` comes from the watch page or the first comment page, so it can be missing when YouTube shows neither.

**Can I use clip links?**
No. `/clip/` links aren't recognised and produce a failure row. Use the parent video's link instead.

**What happens for region-blocked videos?**
Set `proxyCountry` to a country where the video is available. The `availableCountries` array on the output shows where YouTube says the video can be watched.

**Are private or members-only videos supported?**
No. Only public and unlisted videos you have a link to can be read. Private, removed or members-only videos produce a failure row with `success: false`, an `error` message and the `playabilityStatus` YouTube returned.

**Does a failed video stop the run?**
No. Every input link gets its own row, so a failure on one video doesn't affect the others. Filter on `success` to separate them.

**Is this affiliated with YouTube?**
No. It's an independent tool that reads publicly available YouTube pages.
