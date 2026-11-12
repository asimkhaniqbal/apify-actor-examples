# Odysee Video Search by Keyword, Channel & Tag

Odysee is a video platform built on the LBRY content network, popular with independent creators covering tech, science, and commentary. This actor searches its catalog by keyword, lists videos from a given channel, or browses by content tag — returning structured metadata for each video found.

## What this actor does

- **Three modes** — `search` (full-text keyword search), `byChannel` (videos from one creator), `byTag` (browse a content category)
- **Sort control** — order results by newest, most-supported, or trending
- **Rich per-video metadata** — title, description, duration, tags, languages, and view count
- **Direct playback links** — a ready-to-open `viewUrl` for each video
- **Clean records** — a field Odysee doesn't provide for a given video (e.g. no view count yet) is left out rather than returned as a placeholder

## Output per video

- `claimId` — Odysee's unique content identifier
- `title`, `description` — description is truncated to 2000 characters
- `channelName`, `channelUrl` — publishing channel and a link to it
- `thumbnailUrl`
- `durationSeconds`
- `mediaType` — file MIME type, e.g. `video/mp4`
- `tags` — up to 30 content tags
- `languages` — language codes set on the video
- `publishedAt` — ISO 8601 publish date
- `views` — view count, when Odysee has recorded one
- `viewUrl` — direct link to the video's page on odysee.com
- `canonicalUrl` — fallback canonical content URL, only present on the rare video where `viewUrl` can't be built
- `scrapedAt` — ISO 8601 timestamp the record was collected

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (select) | `search` | `search`, `byChannel`, or `byTag` |
| `keyword` | string | `open source` | Search term (mode=`search`) |
| `channel` | string | `@veritasium` | Channel name, with or without `@` (mode=`byChannel`) |
| `tag` | string | `technology` | Tag to browse, e.g. `science`, `gaming` (mode=`byTag`) |
| `orderBy` | string (select) | `release_time` | `release_time` (newest first), `effective_amount` (most supported), or `trending_group` (trending) |
| `maxItems` | integer | `20` | Maximum number of videos to return (1–500) |

### Example: keyword search

```json
{
  "mode": "search",
  "keyword": "open source",
  "maxItems": 20
}
```

### Example: all videos from a channel, newest first

```json
{
  "mode": "byChannel",
  "channel": "@veritasium",
  "orderBy": "release_time",
  "maxItems": 50
}
```

### Example: trending videos in a tag

```json
{
  "mode": "byTag",
  "tag": "gaming",
  "orderBy": "trending_group",
  "maxItems": 30
}
```

## Use cases

- **Content discovery** — find independent creators publishing on a topic before mainstream platforms pick it up
- **Channel monitoring** — track a specific creator's upload history and view counts over time
- **Trend tracking** — follow what's trending or most-supported in a tag to spot emerging topics
- **Research and archiving** — collect structured metadata for videos in a subject area for cataloging

## FAQ

**Is a login or API key required?**
No. Odysee's video catalog is publicly browsable without an account.

**What kind of content is on Odysee?**
Odysee hosts independent creators across technology, science, education, news commentary, and entertainment.

**How do I find a specific creator's videos?**
Use `mode: "byChannel"` with their channel handle, e.g. `@veritasium` or `@LinusTechLips` — the `@` is optional.

**What's the difference between the three `orderBy` options?**
`release_time` sorts newest first, `effective_amount` sorts by how much a video has been supported/boosted, and `trending_group` reflects Odysee's own trending ranking.

**Why is `views` missing on some videos?**
Odysee doesn't expose a view count for every video; when it's not available, the field is left out of that record.

**Is this affiliated with Odysee or LBRY?**
No. This is an independent actor that reads Odysee's publicly available content index; it is not operated by or affiliated with Odysee or LBRY.
