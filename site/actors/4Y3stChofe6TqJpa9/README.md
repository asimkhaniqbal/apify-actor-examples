# YouTube Shorts Metadata Extractor for Channels & Hashtags

Collect YouTube Shorts from a channel's Shorts tab, a hashtag feed, keyword search results, a list of Shorts links, or a sound — and turn each one into a row with exact view and like counts, comment count, publish date, duration, category, sound title, caption languages and the uploader's channel profile.

## What this actor does

- **Five ways in:** a channel's Shorts tab (`channel`), a hashtag's Shorts feed (`hashtag`), YouTube search filtered to Shorts (`search`), specific Shorts URLs/IDs (`direct`), or other Shorts that use the same sound as seed links (`sound`)
- **Channel sorting:** newest, most popular or oldest first, matching the buttons YouTube shows on the Shorts tab
- **Exact per-Short metrics:** exact views, likes, comment count, duration, publish date, upload date, category and sound — pulled from each Short's own page, not the rounded listing values
- **Captions:** whether captions exist and which language tracks are available
- **Channel profile on every channel-mode row:** subscribers, description, join date, location, links, avatar, banner, total views, total videos and verification
- **Deep back-catalogs:** page through a channel's entire Shorts list (up to 10,000 per channel per run)
- **Fast listing-only mode:** turn `includeDetails` off to skip per-Short pages and return titles, links, thumbnails and displayed view counts quickly
- **Members-only Shorts (optional):** `includeMembersOnly` also collects a channel's members-only Shorts when YouTube offers that filter on its Shorts tab; when the filter is not rolled out for the channel the run notes it and returns the public listing
- **Exact numbers only or nothing:** a field YouTube does not expose for a given Short is omitted instead of emitted as `null` or a guess; long-form videos are never mixed in

## Output per Short

One row per Short. Optional fields are omitted when YouTube does not expose a value.

| Field | Description |
|-------|-------------|
| `videoId` | The Short's YouTube video ID |
| `title` | Short's title |
| `url` | `https://www.youtube.com/shorts/<videoId>` |
| `thumbnailUrl` | Vertical thumbnail (9:16) |
| `viewCount` | Exact view count (or the displayed count when `includeDetails` is off) |
| `viewCountText` | View count as displayed, e.g. `"30M views"` |
| `likes` | Exact like count |
| `commentsCount` | Exact comment count |
| `commentsTurnedOff` | `true` when the uploader disabled comments |
| `duration` | Duration as `MM:SS` / `HH:MM:SS` |
| `durationSeconds` | Duration in seconds |
| `publishDate` | Publish date with timezone, e.g. `"2026-10-16T09:00:02-07:00"` |
| `uploadDate` | Same instant in UTC, e.g. `"2026-10-16T16:00:02.000Z"` |
| `category` | YouTube category, e.g. `"Entertainment"` |
| `soundTitle` | `"Original Sound"` or the track name when a licensed sound is used |
| `captionLanguages` | Caption language codes, e.g. `["en"]` |
| `subtitles` | `"available"` when caption tracks exist |
| `description` | Short's description text |
| `hashtags` | Hashtags found in the description (hashtag mode returns the searched tag) |
| `postedTimeText` | Relative publish label, e.g. `"2 days ago"` |
| `formats` | Available video stream qualities |
| `isAgeRestricted` / `isMembersOnly` / `isUnlisted` | Present and `true` only when that applies |
| `channelName`, `channelUrl`, `channelId`, `channelUsername` | Uploader identity (channel mode) |
| `numberOfSubscribers`, `channelTotalVideos`, `channelTotalViews`, `channelJoinedDate`, `channelLocation`, `channelDescription`, `channelDescriptionLinks`, `channelAvatarUrl`, `channelBannerUrl`, `isChannelVerified` | Channel profile fields (channel mode) |
| `aboutChannelInfo` | Nested object repeating the channel profile fields |
| `searchQuery` | The keyword (search mode) |
| `matchedSoundTitle` | The seed sound a `sound`-mode row was found by |
| `sourceType` / `source` | Which input produced the row |
| `order` | Position in the result set, starting at 0 |
| `scrapedAt` | ISO timestamp of when the row was scraped |

### Error and status rows

- A channel, hashtag or search query that cannot be read produces an error row in the same dataset: `recordType: "error"` with `sourceType`, `source`, `errorCode` (`CHANNEL_NOT_FOUND`, `NO_SHORTS`, `HASHTAG_EMPTY`, `SEARCH_EMPTY`, `INVALID_SHORT_URL`) and an `error` message. The run itself still succeeds.
- When a run finds no Shorts at all, the dataset gets one status row instead: `recordType: "status"`, `status: "no_results"`, plus the inputs you supplied.

## Input

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `inputMode` | string | No | `"channel"` | `channel`, `hashtag`, `search`, `direct` or `sound` |
| `channelUrls` | string[] | Channel mode | `["https://www.youtube.com/@MrBeast"]` | Channel URLs, `@handles` or channel IDs |
| `hashtags` | string[] | Hashtag mode | – | Hashtags like `#cooking` (with or without `#`) |
| `searchQueries` | string[] | Search mode | – | Keywords; results are filtered to Shorts |
| `shortUrls` | string[] | Direct / sound mode | `["https://www.youtube.com/shorts/pfUG56htjBc"]` | Shorts URLs, youtu.be/watch links, or 11-character IDs; seed links in sound mode |
| `sortShortsBy` | string | No | `"newest"` | Channel Shorts order: `"newest"`, `"popular"` or `"oldest"` |
| `maxShorts` | integer | No | `3` | Max Shorts per channel, hashtag or query (per run in direct mode), 1–10000 |
| `includeDetails` | boolean | No | true | Visit each Short's page for exact metrics, dates, category, sound and captions. `false` = fast listing-only rows |
| `includeMembersOnly` | boolean | No | false | Channel mode: also collect members-only Shorts when YouTube offers the `Members only` filter (rollout-dependent; reads each member Short's own page) |
| `oldestPostDate` | string | No | – | Only Shorts published on/after this date (`YYYY-MM-DD`) |
| `market` | string | No | `"US"` | Region whose YouTube settings are used (31 countries), which sets the language of returned metadata |

### Supported inputs per mode

- `channel` — `@MrBeast`, `https://www.youtube.com/@MrBeast`, `https://youtube.com/channel/UCX6OQ3DkcsbYNE6H8uQQuVA`, a bare `UC…` ID, or a name
- `hashtag` — `#cooking`, `cooking`
- `search` — any keyword phrase, e.g. `"cooking shorts"`
- `direct` — `https://www.youtube.com/shorts/<id>`, `https://youtu.be/<id>`, `watch?v=<id>`, or the bare 11-character ID
- `sound` — the same forms; each link's sound is looked up and its other Shorts are returned

### Example: a channel's most popular Shorts

```json
{
  "inputMode": "channel",
  "channelUrls": ["https://www.youtube.com/@MrBeast"],
  "sortShortsBy": "popular",
  "maxShorts": 50
}
```

### Example: keyword search

```json
{
  "inputMode": "search",
  "searchQueries": ["cooking shorts", "home workout"],
  "maxShorts": 100
}
```

### Example: hashtag feed

```json
{
  "inputMode": "hashtag",
  "hashtags": ["#cooking"],
  "maxShorts": 100
}
```

### Example: specific Shorts, fast listing only

```json
{
  "inputMode": "direct",
  "shortUrls": ["https://www.youtube.com/shorts/pfUG56htjBc"],
  "includeDetails": false
}
```

## Use cases

- **Benchmark a creator's Shorts** — exact views, likes and comments for their entire Shorts back-catalog, sorted by newest or most popular
- **Trend and hashtag research** — see what creators and videos are active under a hashtag right now, then drill into exact engagement
- **Keyword discovery** — find Shorts around a topic with YouTube search and rank them by exact performance
- **Sound tracking** — group Shorts by the sound they use (`Original Sound` vs licensed tracks)
- **Content cadence monitoring** — schedule a run with `oldestPostDate` to collect only recent uploads
- **Data enrichment** — paste a list of Shorts links and receive clean rows with full metadata for each

## FAQ

**Do I need a YouTube account, API key or proxies?** No. The actor reads YouTube's public pages and picks its network route automatically; there is nothing to configure.

**Why do some rows miss `likes` or `commentsCount`?** YouTube hides those counters for some Shorts, or the page was briefly gated; the actor retries and falls back to its secondary source, and simply omits what is genuinely unavailable rather than guessing.

**What is the difference between `viewCount` and `viewCountText`?** With `includeDetails` on, `viewCount` is the exact number from the Short's page and `viewCountText` is YouTube's display string (e.g. `"30M views"`). With `includeDetails` off, only the displayed value is available.

**How deep can a channel's Shorts history go?** To the end of the channel's public Shorts list, up to 10,000 rows per channel per run. `maxShorts` caps each source.

**How deep can a hashtag go?** YouTube's hashtag Shorts feed exposes one page (roughly 36–37 Shorts) and rejects further pages. The actor fills the rest of `maxShorts` from YouTube search results for `"#tag shorts"`, which keeps returning Shorts that carry the tag.

**What does sound mode return?** For each seed link you provide, the actor lists other Shorts that use the same sound (8–15 per seed) and tags them with `matchedSoundTitle`. The seed itself is not included; a seed whose sound has no feed yields a `SOUND_UNAVAILABLE` error row.

**Can I get members-only Shorts?** Yes, when YouTube offers the `Members only` filter on that channel's Shorts tab — the filter is a gradual rollout, so the actor retries the tab a few times and, when it is not offered, says so in the status message and returns the public listing. Members-only Shorts count toward `maxShorts` and cost up to two player requests each; they carry `isMembersOnly: true`. YouTube sometimes serves the filter with an empty listing on datacenter networks — the actor then says so and returns the public listing.

**Which sort orders are supported?** The three YouTube offers on the Shorts tab: newest, most popular and oldest. If a channel does not show those buttons, the run falls back to newest order and says so in the status message.

**Are view counts live?** They are exact at the moment of scraping. YouTube updates counters continuously, so repeated runs can show slightly different numbers.

**Is this affiliated with YouTube?** No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by YouTube or Google.
