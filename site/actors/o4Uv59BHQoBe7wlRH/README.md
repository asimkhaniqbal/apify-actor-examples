# Pinterest Media Downloader

Turn public Pinterest Pins into a folder of usable media files. This actor saves the image, animated GIF or MP4 video into the run's Key-Value Store and writes one dataset row per file with the Pin, creator, board, destination and engagement context Pinterest publishes.

It works without a Pinterest login, without cookies and without a user-supplied proxy. Media is fetched directly from Pinterest's public CDN; the actor only escalates to an Apify proxy when a request is blocked or a search response is empty.

## Modes

| Mode | What it downloads |
|---|---|
| `auto` | Pin links from `pinUrls`, then Pinterest results from `searchTerms` |
| `pinDetail` | Only the Pins in `pinUrls` |
| `searchPins` | Media from Pinterest Pin search results |
| `searchVideos` | Videos from Pinterest's video search vertical |

Search Pins include static images, animated GIFs and video Pins. Search video results often expose only an HLS playlist; the actor derives the sibling public MP4 (`/videos/iht/expMp4/..._720w.mp4`) so those results are still downloadable as real files, and reports the MP4's actual pixel size and duration.

## What you get

Each media row contains:

- `record_type`, `pin_id`, `pinterest_url`, `source_url`
- `media_url` (the URL actually downloaded), optional `source_media_url` (the original HLS or preferred URL)
- `media_type` (`image` or `video`) and `media_format` (`image`, `gif` or `video`)
- `saved_as`, `key_value_store_key`, `download_url`, `file_size_bytes`, `file_sha256`
- `content_type`, `media_width`, `media_height`, `duration_ms`, `thumbnail_url`
- `selected_variant`, `downloaded_variant`
- `scraped_at`
- a nested `pin` object with title, description, alt text, dates, destination link, image and video variants, creator, board, saves, reactions, promotion and shopping flags

The nested `imageVariants` and `videoVariants` maps keep Pinterest's own size and quality names (for example `236x`, `736x`, `orig`, `V_720P`). Animated GIFs are always saved from `images.orig`; selecting a thumbnail size would save a static JPEG instead.

When `includeComments` is enabled, the dataset can also contain public `comment` and `pin_activity` rows for numeric Pins.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | choice | `auto` | `auto`, `pinDetail`, `searchPins` or `searchVideos`. |
| `pinUrls` | array | – | Public Pin links or numeric Pin IDs. |
| `searchTerms` | array | – | Pinterest search phrases. |
| `mediaType` | choice | `all` | `all`, `images`, `videos` or `gifs`. GIFs count as images. |
| `imageSize` | choice | `original` | `original`, `736x`, `564x`, `474x` or `236x`. Falls back to the original or the nearest available size. |
| `videoQuality` | choice | `best` | `best`, `720p`, `540p`, `360p` or `240p`. Falls back to the nearest available quality. |
| `maxItems` | integer | `50` | Maximum files per Pin URL or search phrase. `0` follows available results. |
| `maxPages` | integer | `0` | Maximum search pages per phrase. `0` follows Pinterest's pagination. |
| `minSaves` | integer | – | Keep media only when Pinterest reports at least this many saves. |
| `minCreatorFollowers` | integer | – | Keep media only when Pinterest reports at least this many creator followers. |
| `verifiedOnly` | boolean | `false` | Keep only creators Pinterest marks as verified. |
| `requireOutboundLink` | boolean | `false` | Keep only Pins with a destination link. |
| `excludeAds` | boolean | `false` | Exclude Pins marked as promoted. |
| `commerceOnly` | boolean | `false` | Keep only Pins with product or shopping metadata. |
| `includeComments` | boolean | `false` | Also emit public comment and activity rows. |
| `maxComments` | integer | `10` | Maximum public comment/activity rows per Pin. |
| `proxyConfiguration` | object | – | Optional proxy override. The actor works without one. |

### Example: download specific Pins

```json
{
  "mode": "pinDetail",
  "pinUrls": [
    "https://www.pinterest.com/pin/112027111109155913/",
    "https://www.pinterest.com/pin/142567144433577360/"
  ],
  "imageSize": "original"
}
```

### Example: search images at a smaller size

```json
{
  "mode": "searchPins",
  "searchTerms": ["minimalist kitchen", "small balcony garden"],
  "mediaType": "images",
  "imageSize": "736x",
  "maxItems": 500
}
```

### Example: search videos as MP4

```json
{
  "mode": "searchVideos",
  "searchTerms": ["ocean waves"],
  "videoQuality": "720p",
  "maxItems": 100
}
```

### Example: GIFs only

```json
{
  "mode": "pinDetail",
  "pinUrls": ["142567144433577360"],
  "mediaType": "gifs"
}
```

### Example: qualified commercial media

```json
{
  "mode": "searchPins",
  "searchTerms": ["running shoes"],
  "mediaType": "images",
  "commerceOnly": true,
  "requireOutboundLink": true,
  "minSaves": 25
}
```

## Backward compatibility

Saved tasks that still send the old keys keep working. The actor maps `pinterestUrls`, `startUrls`, `urls`, `keywords`, `search`, `queries`, `imageOnly`, `videoOnly` and `proxy` in code. Cookie fields (`cookies`, `cookiesJson`) are accepted but ignored because the shipped actor is cookieless.

## Use cases

- Build a local reference library of public Pinterest images, GIFs and videos.
- Archive a creator's or board's visual style with source links.
- Collect product, recipe or design media for content research.
- Pair downloaded files with creator, board and engagement context.
- Feed image or video datasets into downstream tagging and analysis.

## FAQ

### Where are the files stored?

Each file is saved as a record in the run's Key-Value Store. The `download_url` field is a public link to that record; `media_url` points at Pinterest's source CDN.

### Do I need a proxy?

No. Direct Pinterest requests are tried first. The actor falls back to Apify's free `AUTO` datacenter proxy only when Pinterest blocks a request or returns an empty search envelope.

### Why is my GIF a still image?

That happens only when the Pin is not actually animated. Animated GIF Pins are saved from `images.orig`, never from a JPEG thumbnail.

### Why does a search video become an MP4 when Pinterest shows HLS?

Pinterest publishes the same video under both `/videos/iht/hls/` and `/videos/iht/expMp4/`. The actor derives the public MP4 variant so the file is directly playable. It never saves an `.m3u8` playlist as a video file.

### Can I choose quality?

Yes. `imageSize` controls image dimensions and `videoQuality` controls MP4 quality. If the requested size or quality is unavailable, the actor uses the original/best or the nearest available variant.

### Why did a Pin produce no file?

It may be private, deleted, region-blocked, missing public media, or it may expose only a media form Pinterest does not publish for download. The actor never fabricates a file or a download URL.

### Are the media URLs permanent?

Pinterest CDN URLs can change or expire. The `download_url` points at the copy saved by this run and is the stable link for the exported file.

### Do filters use estimates?

No. Metrics filters use only values Pinterest publishes on the same Pin. If a count is missing, the Pin does not pass a minimum-count filter.

### Can I get metadata without downloading files?

This actor is file-first: every media row corresponds to a saved file. For metadata-only Pin, profile and board collection, use a dedicated Pinterest data actor.

### What is not supported?

Carousel/slide expansion is not performed: the primary Pin media is downloaded. Video fps, codec and audio-track flags are not reported because the lightweight image does not run `ffprobe`. Search result IDs can be opaque, so detail-only fields such as saves or comments are omitted for those rows rather than guessed.

### Is this affiliated with Pinterest?

No. This is an independent tool that reads Pinterest's public pages and API responses. It is not operated, endorsed or sponsored by Pinterest.
