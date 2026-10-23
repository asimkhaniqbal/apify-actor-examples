# Facebook Page Photo Gallery & Engagement Scraper

Collect publicly posted photos from Facebook pages, profiles, and albums — full-resolution image URLs, Facebook's own alt-text descriptions, captions, and per-photo engagement including reactions, comments, shares, and tagged people. No login or cookies required.

## What this actor does

- Scrapes photos from a page's photo grid, a specific album, or an individual photo URL
- Returns the photo ID, photo URL, full-resolution image URL, and Facebook's alt-text description whenever Facebook provides them
- Adds the author name/URL when the photo grid or the photo's own page shows them
- Adds caption, timestamp, reactions (by type and total), comment count, share count, and tagged people when Facebook renders that detail for the photo
- Filters results to a date range with `newerThan`/`olderThan`, applied after collection
- Only includes fields that actually have data — a photo without a caption simply has no `caption` key, and zero counts (e.g. no shares) are left out too

## Output per photo

- `photoId` — unique Facebook photo identifier
- `photoUrl` — direct link to the photo on Facebook
- `imageUrl` — full-resolution image file URL from Facebook's CDN
- `altText` — Facebook's auto-generated image description
- `authorName`, `authorUrl` — the page or profile that posted the photo
- `caption` — photo description/caption text
- `timestamp` — when the photo was uploaded (ISO 8601)
- `likesCount`, `commentsCount` — like and comment totals as Facebook reports them
- `reactionsByType` — breakdown of reactions (`LIKE`, `LOVE`, `CARE`, `HAHA`, `WOW`, `SAD`, `ANGRY`) by count
- `reactionsTotal` — total reaction count across all types
- `shareCount` — number of times the photo was shared
- `taggedPeople` — array of people tagged in the photo, each with `name`, `url`, `id`, `type`
- `inputUrl` — the Start URL that discovered this photo
- `scrapedAt` — ISO 8601 timestamp of collection

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | **Required.** Facebook page, profile, album, or individual photo URLs to scrape |
| `resultsLimit` | integer | – | Maximum number of photos to scrape, shared across all Start URLs combined. Leave empty to collect as many as the page loads (at most 1,000 per URL). The date filter runs afterwards, so it can leave fewer |
| `newerThan` | string | – | Only include photos posted on or after this date (`YYYY-MM-DD`) |
| `olderThan` | string | – | Only include photos posted on or before this date (`YYYY-MM-DD`) |
| `proxyConfiguration` | object | – | Proxy configuration. The input is prefilled with the residential group, and when left empty the actor uses a residential proxy anyway, because per-photo detail pages require it |

Photos whose posting date can't be determined are always included regardless of `newerThan`/`olderThan`.

### Example: a page's photo grid

```json
{
  "startUrls": [{ "url": "https://www.facebook.com/humansofnewyork/photos" }],
  "resultsLimit": 10
}
```

### Example: filtered to a date range

```json
{
  "startUrls": [{ "url": "https://www.facebook.com/humansofnewyork/photos" }],
  "resultsLimit": 50,
  "newerThan": "2024-01-01",
  "olderThan": "2024-12-31"
}
```

### Example: a single photo URL

```json
{
  "startUrls": [{ "url": "https://www.facebook.com/photo/?fbid=1547667900257527" }]
}
```

## Use cases

- **Brand asset archiving** — collect a page's photo history with full-resolution URLs
- **Engagement analysis** — compare reaction breakdowns and share counts across a page's photos
- **Content moderation research** — review tagged people and captions across an album
- **Accessibility auditing** — check alt-text coverage across a page's images
- **Competitive monitoring** — track a competitor's photo posting cadence and engagement over time

## FAQ

**Can I scrape photos without a Facebook account?**
Yes. The actor requires no login, cookies, or API keys — only publicly visible content is read.

**Why are caption and engagement fields sometimes missing?**
Those fields are only present when Facebook includes them in the public page data for that specific photo. Some photos genuinely have no caption, comments, reactions, or shares.

**Why don't I see a label for every reaction type?**
The actor maps Facebook's reaction IDs to labels (`LIKE`, `LOVE`, `CARE`, `HAHA`, `WOW`, `SAD`, `ANGRY`) using a known mapping. An unrecognized reaction ID ships as its raw numeric string instead of a guessed label, so `reactionsByType` stays accurate.

**Can I scrape private profiles or albums?**
No. Only publicly accessible Facebook content is supported.

**How many photos will I get from one Start URL?**
A single page, profile, or album URL typically surfaces on the order of dozens to around a hundred photos per run, depending on how much Facebook loads for that view. For larger collections, list multiple album/set URLs in `startUrls` or run the actor multiple times.

**Why does `resultsLimit` apply across all my Start URLs instead of per URL?**
It's a shared cap: if an early URL alone reaches the limit, later URLs in the list won't be processed. Set it generously, or run each source as a separate call, if you need every URL covered.

**Why is a residential proxy used by default?**
Photo grid fields (`photoId`, `imageUrl`, `authorName`, `authorUrl`, `altText`) work without any proxy, but Facebook serves a login wall on individual photo detail pages to non-residential IPs — the fields that come from there (`caption`, `timestamp`, reactions, `shareCount`, `taggedPeople`) need a residential proxy to come through reliably.

**Can I filter photos by date?**
Yes. Use `newerThan`/`olderThan` (format `YYYY-MM-DD`); photos whose date couldn't be determined are always included regardless of the filter. The filter is applied after `resultsLimit` photos have been collected, so a narrow date range can return fewer photos than the limit — raise `resultsLimit` to compensate. An invalid date is ignored.

**Is this affiliated with Facebook?**
No. This is an independent, third-party actor that reads Facebook's public pages; it isn't affiliated with or endorsed by Meta.
