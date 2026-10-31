# YouTube Playlist Contents & Channel Playlist Finder

Pull every video inside a YouTube playlist, or flip it around and list every public playlist a channel has published — podcasts, courses, releases, radio mixes, and ordinary playlists alike. Combine both in one run to go straight from a channel to the full contents of each of its playlists.

## What this actor does

- Lists every video in one or more playlists, with position, duration, and view/publish signals
- Discovers every public playlist on a channel, classified by type (playlist, course, podcast, release, radio mix, store)
- Accepts full URLs, watch URLs with a `list=` parameter, or bare playlist IDs
- Marks deleted or private playlist entries as unavailable instead of silently dropping them
- Empty fields are omitted from every record

## Output: per video (`playlistUrls` mode)

- `position` — position in the playlist (1-based)
- `videoId`, `title`, `url`
- `thumbnailUrl`
- `duration`, `durationSeconds`
- `isAvailable` — whether the video is still playable
- `viewCountText`, `publishedTimeText`
- `videoChannelName`, `videoChannelId`, `videoChannelUrl` — the video's uploader
- `playlistId`, `playlistTitle`, `playlistUrl`, `playlistVideoCount`, `playlistViewCount`
- `playlistOwner`, `playlistOwnerChannelId`, `playlistOwnerHandle`, `playlistDescription`, `lastUpdated`
- `scrapedAt`

## Output: per playlist (`channelUrls` mode)

- `playlistId`, `playlistUrl`, `title`
- `playlistType` — `playlist`, `radio_mix`, `course`, `podcast`, `release`, or `store`
- `videoCount`, `videoCountText`
- `thumbnailUrl`
- `firstVideoId`, `firstVideoUrl`
- `channelName`, `channelId`, `channelHandle`
- `scrapedAt`

A channel with no public playlists produces a single `recordType: "status"` row (`status: "no_public_playlists"`, with `channelUrl`, `channelHandle`, `channelId`, `scrapedAt`) instead of an empty result; a channel page that fails to load after retries produces the same `recordType: "status"` shape with `status: "failed"`, `channelUrl`, and an `error` message.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `playlistUrls` | array | – | Playlist URLs or bare playlist IDs to list videos from |
| `channelUrls` | array | – | Channel handles/URLs/IDs to discover every public playlist from |
| `maxPlaylists` | integer | `100` | Max playlists to list per channel (channel mode) |
| `maxVideos` | integer | `0` | Max videos per playlist; `0` returns all |
| `includeUnavailableVideos` | boolean | `false` | Emit placeholder rows (`isAvailable=false`) for hidden/private videos |
| `market` | string | `US` | Region code controlling the language of returned metadata |

Provide `playlistUrls`, `channelUrls`, or both in the same run.

Supported playlist formats: `https://www.youtube.com/playlist?list=...`, `https://www.youtube.com/watch?v=...&list=...`, or a bare ID (`PL`, `RD`, `UU`, `OL`, `LL`, `FL`, `WL` prefixes).

### Example: full contents of a specific playlist

```json
{
    "playlistUrls": ["https://www.youtube.com/playlist?list=PL-osiE80TeTt2d9bfVyTiXJA-UTHn6WwU"],
    "maxVideos": 50
}
```

### Example: discover every playlist on a channel

```json
{
    "channelUrls": ["@Apify"],
    "maxPlaylists": 10
}
```

### Example: channel discovery plus video listings in one run

```json
{
    "channelUrls": ["@Apify"],
    "playlistUrls": ["PLrAXtmErZgOeiKm4sgNOknGvNjby9efdf"],
    "maxPlaylists": 20,
    "maxVideos": 100,
    "market": "GB"
}
```

## Use cases

- **Course cataloging** — list every lesson video in an educational playlist with position and duration
- **Playlist backups** — snapshot your saved playlists' contents before videos get removed
- **Podcast indexing** — pull episode titles and durations from a channel's podcast-type playlists
- **Channel content audits** — discover every playlist a competitor or partner channel has published
- **Series tracking** — find every episode of a multi-part video series in order

## FAQ

**How many videos can one playlist return?**
YouTube allows up to 5,000 videos in a public playlist, and the actor paginates through all of them automatically (subject to `maxVideos`).

**What's the `UU` playlist prefix I see mentioned?**
Every channel has a hidden "uploads" playlist whose ID starts with `UU` instead of `PL` — pass it as a `playlistUrls` entry to get every public upload from that channel without using channel-discovery mode.

**Why do some videos show `isAvailable: false`?**
They were deleted or made private after being added to the playlist. These only appear when `includeUnavailableVideos` is enabled.

**Does this need a YouTube API key or login?**
No. It reads YouTube's public playlist and channel pages directly.

**What's the difference between the two modes?**
`playlistUrls` returns the videos inside specific playlists. `channelUrls` returns the list of playlists a channel owns, not their video contents — combine both inputs to get a channel's playlists and the videos inside a particular one in a single run.

**Why is `playlistViewCount` missing on some rows?**
YouTube doesn't expose an aggregate view count for every playlist type (for example, auto-generated release or radio-mix playlists) — it's omitted rather than estimated.

**Is this affiliated with YouTube?**
No. This is an independent actor that reads YouTube's public pages and is not operated or endorsed by YouTube or Google.
