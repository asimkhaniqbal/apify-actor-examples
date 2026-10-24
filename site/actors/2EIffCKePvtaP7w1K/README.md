# Facebook Page Photo Gallery & Engagement Scraper

Collect public Facebook photos from pages, profiles, albums, tagged collections, and single-photo links. Each row carries the full-size image and thumbnail, caption, upload time, reaction breakdown, comment and share counts, tagged people, owner/page context, album, and post link. No login, cookies, browser, or proxy setup.

## What this actor does

- Walks a page or profile's **Photos** tab or its **Tagged** tab; add both URLs to collect both
- Expands **albums**, a page's **Albums** list, or a direct album URL into its photos
- Accepts **single-photo permalinks** and photo paths
- Follows Facebook's cursor pagination 8 photos at a time, de-duplicates, and keeps going past the first 8-photo batch
- Applies `newerThan` / `olderThan` on Facebook's side for page and tagged tabs (Unix-second query arguments)
- Opens each photo for caption, timestamp, reactions (7 types), comments, shares, tagged people, album, post, and owner
- Copies images to durable storage by default and keeps the signed original plus its expiry

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | Page, profile, `/photos`, `/photos_by`, `/photos_stream`, `/photos_of`, `/photos_albums`, album, photo permalink, or bare handle |
| `resultsLimit` | integer | `100` | Global cap across all Start URLs |
| `newerThan` | string | – | Keep photos posted on or after this `YYYY-MM-DD` |
| `olderThan` | string | – | Keep photos posted on or before this `YYYY-MM-DD` |
| `photoTab` | select | `uploaded` | For page/profile URLs: `uploaded` or `tagged` |
| `includeAlbums` | boolean | `false` | Also walk every album on a page/profile |
| `includeDetails` | boolean | `true` | Open each photo; turn off for a fast grid-only export. Album date bounds still read only the timestamp needed for filtering |
| `storeImages` | boolean | `true` | Store permanent image and thumbnail copies in Apify storage |

`proxyConfiguration` is no longer an input. The actor selects its own connection path; old saved tasks that still send the key are accepted and the key is ignored.

### Examples

A page's recent photos:

```json
{
  "startUrls": [{"url": "https://www.facebook.com/humansofnewyork/photos"}],
  "resultsLimit": 50
}
```

Tagged photos of a page:

```json
{
  "startUrls": [{"url": "https://www.facebook.com/nike"}],
  "photoTab": "tagged",
  "resultsLimit": 100
}
```

Every album on a page:

```json
{
  "startUrls": [{"url": "https://www.facebook.com/nasa/photos_albums"}],
  "resultsLimit": 300
}
```

A date window:

```json
{
  "startUrls": [{"url": "https://www.facebook.com/nationalgeographic/photos"}],
  "newerThan": "2025-01-01",
  "olderThan": "2025-12-31",
  "resultsLimit": 200
}
```

A single photo:

```json
{
  "startUrls": [{"url": "https://www.facebook.com/photo/?fbid=1547667900257527"}]
}
```

## Output per photo

Legacy fields keep their names: `photoId`, `photoUrl`, `imageUrl`, `caption`, `altText`, `timestamp`, `likesCount`, `commentsCount`, `reactionsByType`, `reactionsTotal`, `shareCount`, `taggedPeople`, `authorName`, `authorUrl`, `inputUrl`, `scrapedAt`.

Added fields include `imageUrlOriginal`, `imageExpiresAt`, `imageWidth`, `imageHeight`, `thumbnailUrl`, `thumbnailUrlOriginal`, `ocrText` (Facebook's accessibility caption, not a second OCR pass), `createdTime`, `likeCount`…`angryCount`, `uploaderId/Name/Url`, `pageId/Name/Url/Followers`, `albumId`, `albumName`, `postId`, `postUrl`, `sourceType`, and `detailsStatus`.

`likesCount` keeps its old meaning (total reactions); `likeCount` is the LIKE reaction only. Empty values are omitted. A photo without a caption simply has no `caption` key; a zero count is kept when Facebook reports zero. With `includeDetails` off, album rows have only the grid thumbnail and thumbnail dimensions; `imageUrl` requires details. Album date bounds still work in this mode: the actor reads just the timestamp and keeps the row grid-only.

## Pagination and limits

Facebook returns 8 photos per collection request. The actor follows the cursor until the source ends, `resultsLimit` is reached, three consecutive requests add no new ids, or a duplicate-heavy safety budget is hit. A collection also stops after 2,000 pages (about 16,000 photos) as a safety cap. When a target stops early, the status message notes the shortfall; the log may not name the exact stop cause. No 1,000-photo or 5,000-photo artificial cap is applied.

## Image links and storage

Facebook's CDN links are signed and expire. With `storeImages` on (default), each image and thumbnail is copied to Apify storage; when a copy succeeds, `imageUrl` / `thumbnailUrl` become permanent public links. `imageUrlOriginal` and `imageExpiresAt` preserve the source. If a copy fails, the signed source link is kept and the run continues. With `storeImages` off, the signed links are returned as-is and the expiry is still recorded.

## What this actor cannot collect

- **Photo location** — Facebook does not expose a place/location for photos on these surfaces
- **Multiple image resolutions** — the photo viewer exposes one image URI plus width/height
- **Private profiles, login-only pages, and groups** — only what a logged-out visitor sees
- **Comment bodies or reactor identities** — counts only; use the comments actor for threads
- **Videos and Reels** — the Photos collections contain photos; use the posts/reels actors for video
- **One row per album** — album metadata is attached to its photos; an albums-list URL expands into photos

## FAQ

**Do I need a Facebook account?**
No. The actor reads only public, logged-out content and never asks for cookies or credentials.

**Why did a target return no photos?**
It may not exist, be private, or hide its Photos tab from logged-out visitors. The final status message names failed targets.

**Why are some captions or owner names missing?**
Facebook genuinely has no caption for some photos, and occasionally omits the uploader's display name. Empty fields are omitted rather than filled with guesses.

**What does `detailsStatus` mean?**
`Complete` means the photo page was read; `GridOnly` means details were turned off; `Unavailable` means the photo page could not be read, so grid fields were kept.

**Why do some images need a slower request?**
Gallery pages are free to read. Individual photo pages are often served as a stripped page to datacenter IPs, so the actor automatically retries the detail request through a residential connection. The gallery does not use that path.

**How many photos can I get?**
As many as the page exposes to a logged-out visitor, up to `resultsLimit`. A collection stops after 2,000 pages (about 16,000 photos) even if more remain. Thousands of photos have been walked from one page; duplicate-heavy pages may stop at the safety budget and the status message notes the shortfall.

**Can I filter by date?**
Yes. Use `YYYY-MM-DD`. Page and tagged tabs send the window to Facebook. Albums have no server-side date filter, so each photo has to be opened and checked. This works even with `includeDetails` off: the actor reads only the timestamp needed for the bound, keeps the row grid-only, and stops with a clear status if the window is not reached within the scan budget. An album date run costs one detail request per photo examined.

**Are image links permanent?**
With `storeImages` on, yes, when the copy succeeds; a failed copy leaves the signed source link. With it off, Facebook's signed links expire.

**Is this affiliated with Facebook?**
No. This is an independent third-party tool that reads public pages; it is not affiliated with or endorsed by Meta.

## Data source

Public `facebook.com` pages, profiles, photo albums, and photo permalinks, read without login. The actor does not bypass private content or login walls.
