# Spotify Podcast Show & Episode Metadata Extractor

Collect podcast show and episode metadata from Spotify without a Spotify account, client ID or API key. Search the podcast catalog, export a show together with its complete episode archive, resolve a show or episode link, and keep only the release window you need.

The actor reads the same public web-player data that powers Spotify's podcast pages for logged-out visitors. It returns structured records, never audio downloads, and omits empty fields instead of writing `null` placeholders.

## Modes

| Mode | What it returns | Inputs it uses |
|---|---|---|
| `searchShows` | One record per matching podcast show | `searchQuery` |
| `searchEpisodes` | One record per matching episode | `searchQuery` |
| `getShow` | A show record followed by its episode archive | `showIds`, `startUrls` |
| `getEpisode` | One record per episode ID or link | `episodeIds`, `startUrls` |

Search modes page through Spotify's result set until `maxItems` is reached or the source runs out. `getShow` pages through the complete episode archive, not just the first screen.

## Input

| Field | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `searchShows` | One of the four modes above |
| `searchQuery` | text | `true crime` | Keyword for `searchShows` and `searchEpisodes` |
| `showIds` | string list | empty | Show IDs, `spotify:show:` URIs or show links |
| `episodeIds` | string list | empty | Episode IDs, `spotify:episode:` URIs or episode links |
| `startUrls` | string list | empty | Mixed show/episode links; regional and embed links work |
| `includeDetails` | boolean | `true` | Enriches every search hit with the full show/episode object |
| `includeEpisodes` | boolean | `true` | In `getShow`, export the show's episodes as well |
| `releasedAfter` | text | empty | `YYYY-MM-DD`; older episodes are skipped |
| `releasedBefore` | text | empty | `YYYY-MM-DD`; newer episodes are skipped |
| `maxItems` | integer | `50` | Maximum records to save (1-5000) |

Examples:

```json
{
  "mode": "searchShows",
  "searchQuery": "daily news",
  "maxItems": 25
}
```

```json
{
  "mode": "getShow",
  "startUrls": ["https://open.spotify.com/show/3IM0lmZxpFAY7CwMuv9H4g"],
  "releasedAfter": "2026-01-01",
  "maxItems": 120
}
```

```json
{
  "mode": "getEpisode",
  "episodeIds": ["6cCnimrvXTnb5SIl1jXVtQ"],
  "maxItems": 10
}
```

## Show fields

- `showId` - Spotify show ID
- `name` - show title
- `description`, `descriptionHtml` - plain-text and HTML synopsis
- `publisher` - publisher or creator name
- `mediaType` - `audio` or `video`
- `contentType` - Spotify content type, for example `CONTENT_TYPE_PODCAST`
- `totalEpisodes` - episode count reported by Spotify
- `explicit` - whether Spotify marks the show explicit
- `ratingAverage`, `ratingCount` - show rating and number of ratings
- `topics` - array of `{name, uri}` topic objects
- `images`, `imageUrl` - cover art URLs, largest first
- `externalUrl`, `shareUrl` - canonical Spotify and share links
- `playable`, `playabilityReason` - public playback status
- `consumptionOrder` - Spotify episode ordering, for example `RECENT`
- `showTypes` - Spotify show classifications when present
- `trailer` - trailer episode summary when Spotify exposes one
- `recordType` - always `show`
- `scrapedAt` - ISO 8601 collection timestamp

## Episode fields

- `episodeId` - Spotify episode ID
- `showId`, `showName`, `showImageUrl` - parent show identity
- `name` - episode title
- `description`, `descriptionHtml` - plain-text and HTML episode notes
- `releaseDate` - `YYYY-MM-DD`
- `releaseDateTime` - full ISO 8601 release timestamp
- `releaseDatePrecision` - Spotify precision, for example `MINUTE`
- `durationMs` - episode length in milliseconds
- `explicit`, `contentRating` - content rating information
- `mediaTypes`, `hasVideo` - audio/video classification
- `audioPreviewUrl` - public 30-second preview clip
- `videoPreviewUrl` - video preview when Spotify provides one
- `images`, `imageUrl` - episode artwork URLs, largest first
- `externalUrl`, `shareUrl` - canonical Spotify and share links
- `playable`, `playabilityReason` - public playback status
- `isPaywalled` - whether the episode is paywalled
- `accessBadge` - Spotify access badge when present
- `creator` - creator name when Spotify exposes one
- `isTrailer`, `isNineteenPlus` - episode flags when present
- `transcriptUrl` - transcript link only when the public episode object exposes one
- `recordType` - always `episode`
- `scrapedAt` - ISO 8601 collection timestamp

## Pagination, dates and limits

- Search uses Spotify's offset pagination and stops when the result set is exhausted or `maxItems` is reached.
- `getShow` requests 50 episodes per page and follows Spotify's next offset until the archive ends, `maxItems` records have been saved, or `releasedAfter` passes.
- Episodes are returned newest first by Spotify. `releasedAfter` therefore also stops `getShow` paging early, which keeps large archives inexpensive.
- Search has a practical source ceiling of roughly 1,000 matching items; `getShow` can page through archives several thousand episodes long.
- The public episode object does not expose full transcript text, private subscriber content, or downloadable audio.

## Legacy inputs

The previous version required `clientId` and `clientSecret`. Both are no longer needed and are ignored when an old task still sends them. The old `market` input is also ignored: Spotify decides which catalog is visible from the network location where the actor runs, and Apify cloud runs normally see the US catalog. The former `listCategories` mode has no cookieless equivalent; a saved task that still sends it runs `searchShows` instead and reports that in the status message.

## Data source and scope

- Public Spotify web-player data, read without cookies or login.
- No audio files are downloaded. Preview URLs are metadata links only.
- Podcast catalogs differ by region. Shows that Spotify does not offer in the actor's region can be absent or restricted.
- This is an independent tool and is not affiliated with or endorsed by Spotify.

## FAQ

**Do I need a Spotify developer app?**
No. There is no client ID, client secret, cookie or login step.

**Why are some fields missing from a record?**
Empty values are omitted. Not every publisher supplies a description, rating, topic list, trailer, preview clip or video flag.

**Can I fetch a show and its episodes in one run?**
Yes. Use `mode=getShow`; the run emits one show record and then episodes until `maxItems` records have been saved in total.

**Can I fetch only recent episodes?**
Yes. Set `releasedAfter` and, optionally, `releasedBefore`. In `getShow` the date window also stops paging when the archive is older than the window.

**Can I pass links instead of IDs?**
Yes. `showIds`, `episodeIds` and `startUrls` accept bare IDs, `spotify:` URIs, standard `open.spotify.com` links, regional `/intl-xx/` links and embed links.

**Why does a search return fewer records than `maxItems`?**
Spotify exhausted its public result set for that query, or the date window filtered the remaining episodes. The run status reports what was emitted.

**Are transcripts included?**
Only a transcript link is included when Spotify's public episode object exposes one. Full transcript text is not part of the public payload.
