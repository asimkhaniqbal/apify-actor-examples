# LinkedIn Hashtag Post Monitor Scraper

Track what a professional community is publishing under a LinkedIn hashtag. Give this actor one or more hashtags and it returns the public posts behind them: full text, author identity and followers, media, reactions, comment totals and the first page of comments. No LinkedIn account, session cookie or proxy setup is required.

## What it does

- **Hashtag monitoring, not keyword guessing** — every returned post is checked to carry the requested hashtag in its own public text, so off-topic index results are dropped
- **Batch hashtags** — collect several tags in one run, or return each post once across them with `dedupeAcrossHashtags`
- **Date precision** — combine a quick recency window (`datePosted`) with exact `postedAfter` / `postedBefore` dates
- **Engagement and audience filters** — minimum reactions, minimum comments, person vs company authors, media type, and an optional repost exclusion
- **Post depth** — author followers and avatar, images or video, video transcript on request, repost origin, and up to 10 public comments
- **Local sorting** — order by relevance, newest first, or highest total engagement
- Empty fields are omitted from each record rather than written as `null` or `0`

## Services

- **Hashtag discovery** across the public web index for `linkedin.com/posts`
- **Public post hydration** for text, author, date, media and engagement
- **First-page comment capture** without a login
- **Video transcript and caption extraction** for posts that publish them
- **Repost detection** with the original author and text
- **Hashtag verification** so a post is only attributed to a tag it actually uses

## How it works

LinkedIn does not publish a cookieless hashtag feed, so this actor finds posts through a public search index and then opens each post's public page. During that second step it reads the post's structured public data, re-checks the hashtag in the text, and applies your date, media, author and engagement filters locally. If a post's page is sign-up walled or deleted, that post is skipped; the run never inserts placeholder or status rows.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `hashtags` | array | – | Required. `#AI`, `AI`, `artificial-intelligence`, or a LinkedIn hashtag URL. Up to 20, deduplicated. |
| `maxPostsPerHashtag` | integer | `50` | Posts to keep per hashtag after verification and filters (1–500). |
| `datePosted` | string | any time | `r86400` (24h), `r604800` (week) or `r2592000` (month). |
| `postedAfter` | date | – | Keep posts on or after this UTC day. |
| `postedBefore` | date | – | Keep posts on or before this UTC day. |
| `sortBy` | string | `relevance` | `relevance`, `date` (newest first) or `engagement`. |
| `mediaTypeFilter` | string | all | `text`, `image`, `video`, `article`, `document`, `repost` or `poll`. |
| `authorType` | string | anyone | `PERSON` or `COMPANY`. |
| `minReactions` | integer | `0` | Drop posts below this reaction count. |
| `minComments` | integer | `0` | Drop posts below this comment count. |
| `includeComments` | boolean | `true` | Add the first public comment page (max 10). |
| `includeVideoTranscript` | boolean | `false` | Add transcript and caption URL when published. |
| `excludeReposts` | boolean | `false` | Drop reshared posts. |
| `dedupeAcrossHashtags` | boolean | `false` | One row per post across all requested hashtags. |

### Example: weekly AI pulse

```json
{
  "hashtags": ["AI", "MachineLearning"],
  "datePosted": "r604800",
  "sortBy": "engagement",
  "minReactions": 25,
  "maxPostsPerHashtag": 100
}
```

### Example: company voices in an exact date range

```json
{
  "hashtags": ["ProductManagement"],
  "postedAfter": "2026-10-01",
  "postedBefore": "2026-10-31",
  "authorType": "COMPANY",
  "excludeReposts": true
}
```

### Example: video posts with transcripts

```json
{
  "hashtags": ["https://www.linkedin.com/feed/hashtag/generativeai/"],
  "mediaTypeFilter": "video",
  "includeVideoTranscript": true,
  "maxPostsPerHashtag": 20
}
```

## Output

| Field | Description |
|---|---|
| `postId` | LinkedIn activity ID |
| `postUrl` | Canonical public post URL |
| `hashtag` | The requested hashtag that produced this row (lowercase, no `#`) |
| `hashtags` | Every hashtag found in the post text |
| `matchedHashtags` | Requested hashtags that appear in this post |
| `authorName` | Author display name |
| `authorProfileUrl` | Canonical LinkedIn profile or company URL |
| `authorType` | `person`, `company`, `school` or `unknown` |
| `authorAvatar` | Author image URL when public |
| `authorFollowers` | Author follower count when public |
| `authorHeadline` | Author headline, only when the public post includes one |
| `postedAt` | Publication time, ISO 8601 UTC |
| `content` | Full public post text |
| `headline` | Post headline or title |
| `mediaType` | `text`, `image`, `video`, `article`, `document` or `repost` |
| `imageUrls` | Image URLs on the post |
| `videoUrl` | Public MP4 URL for video posts |
| `videoDuration` | ISO 8601 duration (e.g. `PT3M36S`) |
| `videoTranscript` | Transcript text when requested and published |
| `videoCaptionUrl` | Caption file URL when requested and published |
| `reactionsCount` | Total reactions |
| `commentsCount` | Total comments on the post |
| `comments` | Up to 10 public comments: `authorName`, `text`, `postedAt`, `likes` |
| `isRepost`, `repostOf` | Repost flag and the original `authorName`, `authorProfileUrl`, `content`, `postUrl` |
| `mediaItems`, `mediaUrls` | Structured media list and flat URL list |
| `articleTitle`, `articleUrl` | Article details when the post shares an article |
| `scrapedAt` | When this row was collected |

## Usage cost

This actor is billed per delivered result plus normal Apify platform usage (compute and storage). The daily test uses a tiny prefill, and the actor stops as soon as it reaches `maxPostsPerHashtag`. Larger hashtag sets, deep caps and transcript collection take longer and therefore use more compute. See the Store listing for the current per-result price.

## Use cases

- **Community listening** — see what practitioners post under an industry tag this week
- **Creator and expert discovery** — rank authors by engagement and follower count on a topic
- **Campaign tracking** — follow a branded hashtag and compare post volume and reactions over time
- **Content benchmarking** — collect the top posts in a niche before planning your own
- **Competitive voice analysis** — separate company-page posts from individual posts
- **Research datasets** — export hashtag posts with dates and engagement for analysis

## FAQ

**Do I need a LinkedIn account or cookie?**
No. The actor reads public post pages only and never asks for credentials.

**Why does a hashtag return few or zero posts?**
LinkedIn's hashtag feed is login-gated, so discovery relies on a public search index that is not exhaustive and lags behind the site. Popular tags and wider date windows return more. The run status explains whether the index was empty, the post pages were walled, the hashtag check failed, or your filters removed everything.

**Why is the comment list capped at 10?**
Public post pages expose only the first comment page. The total is still in `commentsCount`; older comments require a signed-in session, which this actor does not use.

**Why are `repostsCount`, `viewsCount` or a reaction breakdown missing?**
LinkedIn does not publish share counts, view counts or per-reaction splits on public posts. Those fields are omitted rather than filled with invented zeros.

**Why is `authorHeadline` missing on many rows?**
Headlines live on profile pages, not on public post pages, and profile pages are frequently blocked. The field appears only when the post itself publishes it.

**Can I get posts from several hashtags at once?**
Yes. Add up to 20 tags. By default each post is returned once per matching hashtag; set `dedupeAcrossHashtags` to return it once.

**Is sorting global?**
No. Sorting applies to the posts found in this run. The public index has no sort parameter, so `date` and `engagement` order the collected set, not all of LinkedIn.

**How fresh is the data?**
The index can take days to pick up new posts. Use the past week or month for monitoring, and treat the 24-hour window as best-effort.

**Does a hashtag URL work?**
Yes. Paste `https://www.linkedin.com/feed/hashtag/ai/` or `.../feed/hashtag/?keywords=ai`; it is normalized to `ai`.

**Is this affiliated with LinkedIn?**
No. It is an independent third-party actor that reads public pages. It is not affiliated with or endorsed by LinkedIn Corporation.

## Notes

- Hashtag matching is case-insensitive and ignores hyphens/underscores, so `#MachineLearning` and `#machine-learning` are treated as the same tag.
- A post with no visible reaction or comment count is treated as `0` when a minimum filter is set.
- `postedBefore` includes the whole chosen UTC day; timestamps inside the input are used exactly.
- Reposts are returned by default with `mediaType: repost` and the original post in `repostOf`.
- A zero-result run is a valid result: the dataset stays empty and the status message explains why.

## Limitations

- **No direct hashtag feed.** LinkedIn login-gates every hashtag page, so coverage depends on the public search index and is not guaranteed to be complete.
- **Index lag.** Very recent posts may not be indexed yet; the 24-hour filter can be sparse.
- **First comment page only.** Public pages show up to 10 comments; full threads need a login.
- **No repost, view or reaction-type counts.** These are not published publicly.
- **Author headlines are rare.** Profile enrichment is not performed because profile pages are inconsistently reachable.
- **Document, poll, carousel and event posts** are classified only when the public page exposes a matching shape; document posts are occasionally seen, while poll, carousel and event shapes have not been observed, so the `poll` filter can return no rows. The enum values are kept for forward compatibility.
- **Set-local sorting.** Ordering applies to the posts found in the current run.
- **Practical caps.** Each hashtag is limited to 500 posts, and the search index itself may stop returning new results before that.
