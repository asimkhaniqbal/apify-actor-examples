# Twitch Live Stream, Channel & VOD Metadata Extractor

One actor, nine modes: pull Twitch's current top live streams, browse a game category, look up a channel's profile with its recent VODs and clips, fetch a single video or clip by ID, run a free-text search, or resolve a plain Twitch URL into structured data.

## What this actor does

- Fetches the current top live streams across Twitch, or within a single game category, with language and viewer-count filters
- Looks up a channel's profile — followers, partner/affiliate status, current live state, last broadcast — plus its recent VODs and top clips
- Returns the top games (categories) on Twitch right now, or the top clips for one category over a chosen time window
- Looks up a single VOD by its video ID, a single clip by its slug, or resolves a list of plain twitch.tv URLs to the right record type automatically
- Omits any field Twitch doesn't provide for a given record rather than returning null, empty string, or an empty object

## Modes

| Mode | What you get |
|---|---|
| `topStreams` | Current top live streams, with broadcaster, game, viewer count, language and preview thumbnail |
| `topGames` | Current top categories, sorted by viewer count, with box art and follower/broadcaster counts |
| `search` | Free-text search across channels, games and videos, optionally restricted to one result type |
| `byChannel` | A channel's profile, plus its recent VODs and top clips when enabled |
| `byCategory` | Live streams currently broadcasting in a given category |
| `topClips` | Top clips for a category over a chosen time window |
| `byVideo` | Full metadata for a single VOD by video ID |
| `byClip` | Full metadata for a single clip by slug |
| `byUrl` | Resolves a list of twitch.tv URLs (channel, video, clip, or category pages) to the matching record type |

## Output per record

Every record carries a `recordType` of `stream`, `channel`, `video`, `clip`, or `game`, plus a `scrapedAt` timestamp.

### Live stream (`recordType: "stream"`)

| Field | Description |
|---|---|
| `streamId` | Twitch's stream ID |
| `title` | Stream title |
| `viewersCount` | Current concurrent viewers |
| `streamType` | Broadcast type as reported by Twitch |
| `language` | Broadcast language code |
| `startedAt` | When the stream started |
| `previewImageURL` | Live preview thumbnail |
| `channelId` / `channelName` / `displayName` | Broadcaster identifiers |
| `gameId` / `gameName` / `gameSlug` | Category being streamed |
| `tags` | Stream tags, up to 20 |
| `broadcaster` / `game` | Nested detail objects for the broadcaster and category |
| `url` | Channel URL |

### Channel (`recordType: "channel"`)

| Field | Description |
|---|---|
| `channelId` / `channelName` / `displayName` | Channel identifiers |
| `description` | Channel "About" text |
| `createdAt` | Account creation date |
| `profileImageURL` / `bannerImageURL` / `offlineImageURL` | Channel images |
| `primaryColorHex` | Channel's brand color |
| `isPartner` / `isAffiliate` / `isStaff` | Role flags, present only when true |
| `followersCount` | Total followers |
| `streamTitle` / `currentGame` | Configured stream title and category, when set |
| `isLive` | Whether the channel is currently live |
| `liveStream` | Nested live-stream details, present only when currently live |
| `lastBroadcast` | Nested details of the channel's most recent broadcast |
| `url` | Channel URL |

### VOD (`recordType: "video"`)

| Field | Description |
|---|---|
| `videoId` | Video ID |
| `title` / `description` | Video title and description |
| `lengthSeconds` | Duration in seconds |
| `viewCount` | Total views |
| `createdAt` / `publishedAt` | Creation and publish timestamps |
| `previewImageURL` | Thumbnail |
| `broadcastType` | Archive, Highlight, Upload, etc. |
| `language` | Video language code |
| `gameId` / `gameName` / `gameSlug` / `game` | Associated category |
| `channelId` / `channelName` / `displayName` / `owner` | Channel that owns the video |
| `url` | Video URL |

### Clip (`recordType: "clip"`)

| Field | Description |
|---|---|
| `clipId` / `slug` | Clip identifiers |
| `title` | Clip title |
| `viewCount` | Total views |
| `durationSeconds` | Clip length |
| `createdAt` | When the clip was created |
| `language` | Clip language code |
| `thumbnailURL` / `embedURL` | Thumbnail and embeddable player URL |
| `videoOffsetSeconds` | Offset into the source broadcast, when known |
| `gameId` / `gameName` / `gameSlug` / `game` | Associated category |
| `channelId` / `channelName` / `displayName` / `broadcaster` | Channel the clip is from |
| `curator` | Who created the clip, when different from the broadcaster |
| `sourceVideo` | The source VOD's ID and title, when the clip was cut from one |
| `url` | Clip URL |

### Game / category (`recordType: "game"`)

| Field | Description |
|---|---|
| `gameId` | Category ID |
| `name` / `displayName` | Category name |
| `slug` | URL slug for the category |
| `viewersCount` | Current total viewers across the category |
| `followersCount` | Total followers of the category |
| `broadcastersCount` | Number of channels currently streaming it |
| `boxArtURL` / `avatarURL` | Category artwork |
| `description` | Category description |
| `url` | Category directory URL |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | topStreams | One of the nine modes listed above |
| `searchQuery` | string | lirik | Free-text query for `search` mode |
| `searchType` | string | all | Restrict `search` results to: all, channel, game, or video |
| `channelName` | string | – | Channel login for `byChannel` mode, e.g. "lirik" |
| `categorySlug` | string | – | Category slug for `byCategory`/`topClips`, e.g. "just-chatting" |
| `videoId` | string | – | VOD ID for `byVideo` mode |
| `clipSlug` | string | – | Clip slug for `byClip` mode |
| `twitchUrls` | array of strings | `[]` | Twitch URLs to resolve for `byUrl` mode |
| `language` | string | EN | Broadcast language filter for `topStreams`, `byCategory` and `search`; leave on "(any)" for no filter |
| `streamType` | string | (any) | VOD type filter for `byChannel`: ARCHIVE, HIGHLIGHT, UPLOAD, PAST_PREMIERE, or PREMIERE_UPLOAD |
| `videoSort` | string | TIME | Sort order for a channel's VODs: TIME (newest first) or VIEWS (most-viewed first) |
| `clipPeriod` | string | LAST_WEEK | Time window for clips: LAST_DAY, LAST_WEEK, LAST_MONTH, or ALL_TIME |
| `includeChannelClips` | boolean | true | In `byChannel` mode, also fetch the channel's top clips |
| `includeChannelVideos` | boolean | true | In `byChannel` mode, also fetch the channel's recent VODs |
| `minViewersCount` | integer | – | Drop live streams below this concurrent-viewer count |
| `minViewCount` | integer | – | Drop VODs/clips below this total-view count |
| `containsKeyword` | string | – | Case-insensitive substring filter on stream/video/clip titles |
| `authToken` | string | – | Optional Twitch session token; not required for any mode, but can unlock subscriber-gated fields |
| `useApifyProxy` | boolean | false | Route requests through a proxy from the start instead of only as a fallback |
| `maxItems` | integer | 50 | Hard cap on the number of records returned (1–5000) |

### Example: top live streams in English

```json
{
  "mode": "topStreams",
  "language": "EN",
  "maxItems": 20
}
```

### Example: channel profile with VODs and clips

```json
{
  "mode": "byChannel",
  "channelName": "lirik",
  "includeChannelVideos": true,
  "includeChannelClips": true,
  "maxItems": 30
}
```

### Example: top clips for a category this month

```json
{
  "mode": "topClips",
  "categorySlug": "league-of-legends",
  "clipPeriod": "LAST_MONTH",
  "maxItems": 25
}
```

### Example: resolve a list of Twitch URLs

```json
{
  "mode": "byUrl",
  "twitchUrls": [
    "https://www.twitch.tv/shroud",
    "https://www.twitch.tv/videos/2766365638"
  ]
}
```

## Use cases

- **Esports and influencer discovery** — find who's streaming a given game right now, and how many viewers they have
- **Clip curation** — pull top clips for a category over the last day, week, or month for highlight reels
- **Channel research** — build a profile of a creator's followers, stream cadence, and recent VOD performance
- **Category trend tracking** — monitor viewer and broadcaster counts for specific games over time
- **Link enrichment** — turn a list of plain Twitch links into structured channel, video, clip, or category data

## FAQ

**Do I need a Twitch account or token?**
No. Every mode works without any login. The optional `authToken` input can raise rate limits or unlock a few subscriber-gated fields, but isn't required.

**Why don't I see subscriber counts on a channel?**
Twitch only exposes subscriber counts and a few other partner-only fields to a logged-in session. Supply `authToken` to attempt to unlock them; without it, those fields are simply left out.

**What's the difference between `byCategory` and `topClips`?**
`byCategory` returns streams that are live right now in that category. `topClips` returns the best-performing clips from that category over the time window you choose, regardless of whether anyone is streaming it currently.

**Can this pull chat messages?**
No. This actor covers streams, channels, VODs, clips and categories only; live or replay chat is not in scope.

**How do I find a category slug?**
Open the category's page on twitch.tv — the slug is the last part of the URL, e.g. `just-chatting` or `league-of-legends` — or run `topGames` mode and read the `slug` field from the results.

**Why might a run return fewer records than `maxItems`?**
If a category, channel, or search has fewer matching items than requested, the actor returns everything it found instead of padding the output.

**Is this affiliated with Twitch?**
No. This actor reads Twitch's public data and is not affiliated with, endorsed by, or sponsored by Twitch.
