# YouTube Video Metadata & Chapter Extractor

Give this actor YouTube video or Short links (or bare 11-character IDs) and get one structured row per video: exact metadata, manual and automatic chapters, heatmap segments, caption-track availability, live and age status, and optional comments or transcript text. No YouTube API key, cookies, or proxy setup.

## What this actor does

- **Exact metadata:** title, description, channel, exact views and likes, duration, ISO publish time, category, creator keywords, thumbnails and canonical URL.
- **Chapters:** manual creator chapters and YouTube-generated automatic chapters, each with title, start, end, source and a deep link. Description timestamps are used only when YouTube exposes no chapter panel.
- **Heatmap:** optional Most replayed segments and YouTube's own labels. Heatmap is available when YouTube provides it and is omitted otherwise.
- **Captions:** the full caption-track list with language, manual or auto-generated status, and a transcript when a track can be downloaded.
- **Status:** live now, was live, upcoming, age-restricted, unlisted, family-safe, available countries and playability flags.
- **Shorts and batches:** watch, youtu.be, Shorts, embed, live, `/v/` and mobile links or bare IDs; repeated IDs are processed once.
- **Comments:** up to 1,000 comment texts per video when enabled. The display comment count is captured separately.
- **Omit-empty:** a field is present only when YouTube returned real data. On a gated page some values, including caption summaries, can be unavailable rather than genuinely empty; see Limitations.

## Output

Each dataset row is one unique video.

| Field | Type | Notes |
|---|---|---|
| `videoId`, `url`, `inputUrl` | string | Canonical watch URL and the original input |
| `type`, `uploadType` | string | `video` or `short`; a Short is labelled `short` from a `/shorts/` link or the Shorts player surface |
| `title`, `description` | string | Full values on normal pages; a gated description can be truncated to about 160 characters |
| `channelId`, `channelName` | string | Uploader identity |
| `channel` | object | id, name, handle, url, subscriber count/text, logo, badges |
| `viewCount`, `likeCount` | integer | Exact values when YouTube exposes them |
| `commentCountText` | string | Display count such as `2.4M` |
| `commentCount` | integer | Only when the display value is an exact integer |
| `durationSeconds` | integer | Omitted for live streams or unknown duration |
| `publishedDate` | string | `YYYY-MM-DD` |
| `publishedAt` | string | ISO-8601 timestamp with the original offset; omitted when YouTube hides it on a gated page |
| `uploadDate` | string | Raw YouTube upload value |
| `category`, `tags`, `hashtags` | string / array | Creator settings and description hashtags |
| `thumbnails` | object | default, medium, high, standard, maxres when available |
| `captionTracks` | array | language, name, auto-generated flag, translatable; empty when YouTube exposes none or gates the page |
| `captionTrackCount`, `hasManualCaptions`, `hasAutoCaptions` | integer / boolean | Caption availability summary; `0`/`false` can mean "not observed" on a gated page |
| `chapterSource` | string | `manual`, `auto` or `description` |
| `chapters` | array | title, startSeconds, endSeconds, startTimeText, source, url, thumbnailUrl |
| `heatmap` | array | startSeconds, endSeconds, intensity (when enabled) |
| `mostReplayed` | array | YouTube's Most replayed labels (when enabled) |
| `heatmapSegmentCount` | integer | Present only when a heatmap exists |
| `liveStatus` | string | `live_now`, `was_live` or `upcoming`; omitted for regular uploads |
| `isLiveNow`, `isUpcoming`, `isUnlisted`, `isFamilySafe`, `isShortsEligible` | boolean | Availability flags |
| `isAgeGated`, `requiresLogin`, `isPlayable`, `playabilityStatus` | boolean / string | Access state |
| `availableCountries` | array | Country codes YouTube reports |
| `transcript` | array | Timed `{start, dur, text}` segments when available |
| `comments` | array | Comment texts when `maxComments` > 0 |
| `success`, `error`, `scrapedAt` | boolean / string | Result state; failure rows still carry real partial data |

Failure rows are written for invalid, removed, private or region-blocked links. They keep the input URL, a real video ID when known, `success: false` and an explanatory `error`; they never invent metadata.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `videoUrls` | array | – (required) | Watch/youtu.be/Shorts/embed/live/`/v/` links or bare 11-character IDs. Duplicates are removed. |
| `maxComments` | integer | `0` | Comment texts to collect per video (0–1000). `0` skips comments. |
| `includeChapters` | boolean | `true` | Manual and automatic chapters with source and end times. |
| `includeHeatmap` | boolean | `false` | Most replayed segments and labels when available. |
| `includeEndscreen` | boolean | `true` | Endscreen elements when YouTube exposes them. |
| `includeCards` | boolean | `false` | Info cards embedded in the player. |
| `includeTranscriptMetadata` | boolean | `true` | Caption-track list with language and manual/auto status. |
| `includeTranscript` | boolean | `true` | Download transcript text when a caption track is available. |
| `includeRelatedVideos` | integer | `0` | Related sidebar videos to include (0–50). |

### Example: core metadata and chapters

```json
{
  "videoUrls": ["https://www.youtube.com/watch?v=dQw4w9WgXcQ"],
  "includeChapters": true,
  "includeHeatmap": false
}
```

### Example: batch IDs with heatmap and comments

```json
{
  "videoUrls": ["jNQXAC9IVRw", "https://youtu.be/dQw4w9WgXcQ", "https://www.youtube.com/shorts/IaSm0LeErrc"],
  "includeHeatmap": true,
  "maxComments": 20,
  "includeTranscript": false
}
```

## Use cases

- **Content research:** compare exact views, likes, dates and chapter structure across a list of videos.
- **SEO and publishing:** inspect titles, descriptions, tags, categories and hashtags from competitors.
- **Retention analysis:** see which parts YouTube marks as Most replayed and how they line up with chapters.
- **Caption and accessibility audits:** find which languages have manual captions versus auto-generated tracks.
- **Monitoring:** re-run the same list to track view growth, live status and availability changes.
- **AI and analytics pipelines:** feed clean, stable JSON into summaries, dashboards or datasets.

## FAQ

**Does it need cookies or a proxy?**
No. It reads public YouTube pages and public InnerTube endpoints without cookies, logins or API keys. No proxy setup is required. After a block it can retry through Apify's built-in datacenter proxy automatically; there is no proxy field and nothing to configure.

**Why is `commentCount` missing when I can see a count on YouTube?**
YouTube shows abbreviated counts such as `2.4M`. That value is returned in `commentCountText`. `commentCount` is populated only when the display value is an exact integer, so a rounded number is never presented as exact.

**Why is a description short or a field missing?**
On Apify cloud/datacenter IPs YouTube sometimes shows a bot-check page. The actor still returns the chapters and heatmap visible on that page and recovers what it can, but some fields (publish time, duration, tags, caption tracks, transcript) may be missing and a gated description can be cut to about 160 characters. It never invents values. Re-running can recover more because the gate is per request.

**Why is a heatmap missing?**
YouTube does not publish Most replayed data for every video. It is commonly absent on live streams, very new or low-view videos, some music videos and age-restricted videos. The field is omitted rather than fabricated.

**Why do some videos have no chapters?**
Only creators who add chapters, or videos where YouTube generates automatic chapters, have them. `chapterSource` tells you which kind was found. Videos with neither have no `chapters` field.

**Are auto-generated chapters supported?**
Yes. They use YouTube's `AUTO_CHAPTERS` surface and are labelled `auto`; creator chapters are labelled `manual`.

**Why are captions missing for some videos?**
YouTube only exposes caption tracks when the uploader provided them or the platform generated them. Live streams, some age-restricted videos and bot-gated pages often have none. `captionTrackCount`, `hasManualCaptions` and `hasAutoCaptions` show what was observed; a zero on a gated page can mean "not observed" rather than "no captions".

**Is the transcript always available?**
No. Transcript text is returned only when a caption track can be downloaded. Set `includeTranscript` to `false` to skip the extra request.

**What happens for age-restricted or private videos?**
Age-restricted videos can return real metadata with `isAgeGated: true`. Private, removed or members-only videos produce a failure row with `success: false` and the reason YouTube gave. The actor does not bypass access controls.

**Does a failed link stop the run?**
No. Every unique input gets its own row; failures do not stop the remaining videos.

**Is this affiliated with YouTube?**
No. It is an independent tool that reads publicly available YouTube data.

## Limitations

- Heatmap, automatic chapters, captions and transcripts are conditional on what YouTube publishes for each video.
- Cloud runs can be partially gated by YouTube. Missing fields (publish time, duration, tags, caption tracks, transcript) are omitted rather than guessed; a gated description can be truncated to about 160 characters, and summary values such as `captionTrackCount: 0` can mean "not observed" rather than "none". Re-running can recover more because the gate is per request.
- A Short is labelled `short` when the input uses a `/shorts/` link or YouTube exposes the Shorts player; a bare ID without those signals is labelled `video` (`isShortsEligible` and duration are still returned).
- Exact comment counts and exact subscriber counts are not available from the public surfaces used here; display text is returned instead.
- Private, members-only and login-gated content is not supported.
- YouTube's public endpoints can intermittently ask for a bot check; the actor retries with a fresh connection before reporting a real failure.
