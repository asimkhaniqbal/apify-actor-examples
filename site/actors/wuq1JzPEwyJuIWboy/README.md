# LinkedIn Keyword & Hashtag Post Discovery Scraper

Find public LinkedIn posts about a topic, a brand, a person, or a hashtag — without a LinkedIn account, cookie, or login. The actor searches the public search-engine index of LinkedIn posts for your keyword, then opens each matching post's public page to return the full text, author, date, reaction and comment counts, top comments, and media. It is built for social listening, lead research, and competitor tracking, and it says plainly what the index can and cannot deliver.

## What this actor does

- **Three modes:** `auto`, `search` (keyword / phrase / hashtag), `postUrls` (enrich specific posts or articles)
- **Batch search:** run up to 20 queries in one run; a post found by two queries is returned once
- **Filters:** date window and exact date range, author type, content type, minimum reactions and comments, exclude reposts
- **Full post records:** text, author name / profile / avatar / followers, ISO timestamp, reactions, comments, top comments with authors and likes, media, hashtags
- **Documents:** PDF/PPTX document-card posts resolve to the original PDF, page count, and (optionally) per-page transcript and page images
- **Engagement:** reaction-type breakdown, repost counts, and up to ~18 comments with profile links when LinkedIn serves its richer public guest layout
- **Monitor mode:** on a schedule, only posts you have not received before
- **Fields a post does not have are omitted** — never `null` and never made up

## Output per post

- `postId`, `postUrl` — canonical post identity and URL
- `sourcePostUrl` — the URL the post was discovered from (provenance)
- `authorName`, `authorProfileUrl`, `authorAvatar`, `authorFollowers`
- `authorType` — `PERSON` or `COMPANY` when the public page says which
- `authorHeadline` — when the public page shows one
- `postedAt` — ISO 8601 timestamp
- `content` — full post text
- `articleTitle`, `articleUrl` — for LinkedIn articles
- `hashtags` — parsed from the post text
- `mediaType` — `text`, `image`, `video`, `article`, `document`, or `repost`
- `mediaItems` — media URLs with thumbnail and duration where available
- `comments[]` — top comments: `{authorName, text, postedAt, reactionsCount, authorProfileUrl?, dateText?}`; up to ~10 on the plain page, up to ~18 when the richer public guest layout is used
- `commentsSource` — `post` or `sdui` (which public layout supplied the comments)
- `documentUrl`, `documentTitle`, `documentPageCount` — for document posts (the original PDF and page count)
- `documentTranscript`, `documentPageImages` — per-page text and page-image URLs when `includeDocumentDetails` is on
- `reactionsCount`, `commentsCount` — as shown on the public post page
- `reactionBreakdown` — per-type counts (`like`, `celebrate`, `love`, `insightful`, `support`, `funny`, `superLike`) when the richer guest layout is used; the sum matches the total
- `repostsCount` — when the richer guest layout exposes it
- `videoDuration`, `videoTranscript` — for video posts (transcript only when the input asks for it)
- `repostOf` — details of the original post when the result repeats someone else's content
- `snippet` — the search engine's excerpt of the post
- `keywordMatch` — whether every word of your query is visible in the post
- `searchQuery`, `searchEngine`, `source`, `contentSource`, `scrapedAt` — run provenance
- Fields a post does not expose are left out of that record.

### Marker rows

- With `emitNoResultsMarker` on (default), a search that finishes cleanly with zero matches pushes one row: `{searchQuery, noResults: true, scrapedAt}`.
- If a post page cannot be opened, the post is still returned as a partial row with the link, snippet, date derived from its ID, and `contentSource: "serp"` instead of being invented or silently dropped.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `auto` | `auto` / `search` / `postUrls` |
| `searchQuery` | string | – | Keyword, phrase, or `#hashtag` (mode=search/auto; prefill: `artificial intelligence`) |
| `searchQueries` | array | `[]` | Additional queries to run in the same run |
| `postUrls` | array | `[]` | Post or article URLs to enrich (mode=postUrls/auto) |
| `sortBy` | string | `relevance` | `relevance` / `recency` / `date_posted` (last two are aliases) |
| `datePosted` | string | `""` | `""` / `r86400` / `r604800` / `r2592000` |
| `postedAfter` | date | `""` | Keep posts on or after this date |
| `postedBefore` | date | `""` | Keep posts on or before this date |
| `authorType` | string | `""` | `""` / `PERSON` / `COMPANY` |
| `contentType` | array | `["articles","posts","documents","videos","images"]` | `articles`, `posts`, `documents`, `videos`, `images` |
| `minReactions` | integer | `0` | Minimum reactions (posts with unknown counts are skipped when > 0) |
| `minComments` | integer | `0` | Minimum comments (same rule) |
| `excludeReposts` | boolean | `false` | Skip reposts of other people's content |
| `includeUnmatched` | boolean | `false` | Also return posts that miss a query word (`keywordMatch: false`) |
| `language` | string | `en` | Interface language used when opening post pages |
| `countryCode` | string | `auto` | Prefer a country's LinkedIn domain (`site:xx.linkedin.com`) |
| `maxResults` | integer | `25` | Posts to return (1–200, after filters) |
| `hydratePosts` | boolean | `true` | Open each post page for the full record; off = link + snippet only |
| `includeVideoTranscript` | boolean | `false` | Add video transcripts when the page exposes one |
| `includeDocumentDetails` | boolean | `false` | For document posts, also return the per-page transcript and page-image URLs |
| `enrichSdui` | boolean | `true` | Use LinkedIn's anonymous public guest session (an ephemeral site cookie, like a normal browser) to read the richer post layout. No account or credentials involved. |
| `includeRepostCounts` | boolean | `false` | Last resort: if the richer layout was not obtained, try up to five extra requests to catch it for repost counts |
| `monitorMode` | boolean | `false` | On schedules: only posts not delivered by an earlier run with the same input |
| `emitNoResultsMarker` | boolean | `true` | Push a zero-match marker instead of an empty dataset |
| `mentions` | array | `[]` | Extra words added to every query |
| `mentionsCompany` / `mentionsPerson` | string | `""` | Names added to the query (numeric IDs are ignored and reported) |
| `talksAbout` | array | `[]` | Topics/hashtags; each becomes `#topic` |

### Example: keyword search with engagement filter

```json
{
  "mode": "search",
  "searchQuery": "b2b sales automation",
  "datePosted": "r604800",
  "minReactions": 10,
  "maxResults": 50
}
```

### Example: hashtag monitoring

```json
{
  "mode": "search",
  "searchQuery": "#MachineLearning",
  "sortBy": "recency",
  "includeVideoTranscript": false,
  "monitorMode": true,
  "maxResults": 100
}
```

### Example: enrich exact posts

```json
{
  "mode": "postUrls",
  "postUrls": [
    "https://www.linkedin.com/posts/example-company_product-launch-activity-7123456789012345678-abCD",
    "https://www.linkedin.com/pulse/example-article-slug"
  ]
}
```

### Example: several queries in one run

```json
{
  "mode": "search",
  "searchQuery": "AI agents",
  "searchQueries": ["LLM evaluation", "\"first sales hire\""],
  "authorType": "PERSON",
  "maxResults": 120
}
```

## Use cases

- **Social listening** — track what is being posted about your brand, product, or an event
- **Lead research** — find people talking about a problem your product solves, with their recent posts
- **Competitor monitoring** — surface company posts and announcements around a topic or brand name
- **Content and SEO research** — see which posts and angles are indexed for your keywords
- **Event and PR tracking** — follow a hashtag or event name during and after a campaign
- **Research datasets** — export clean post text with dates and engagement for analysis

## FAQ

### Do I need a LinkedIn account or cookie?

No. The actor never logs in and never uses a user session; older tasks that still send a cookie value keep working, and the value is ignored. While reading public post pages it may receive an anonymous, server-issued guest cookie from LinkedIn and reuse it only within that run, exactly as a normal browser would. It is never an input, never stored between runs, and never logged. No `li_at` or signed-in session is involved.

### How many posts can I get per run?

Up to 200 with `maxResults`. The practical number depends on how many public posts the search index holds for your query — a specific phrase usually has far fewer than a broad one. The actor never pads results to hit the number: if the index or your filters run out of matches, the run returns fewer (filters such as `datePosted`, `minReactions`, `contentType` and `authorType` remove posts). Raising `maxResults` does not create posts the index does not have.

### Do document posts work?

Yes. Document-card posts (PDF/PPTX carousels) expose a public manifest; the actor reads it and returns the original PDF (`documentUrl`), title, and page count, with per-page transcript and page images when `includeDocumentDetails` is on.

### What “search” means here

The actor reads a public search-engine index of `linkedin.com/posts` pages; it does not query LinkedIn's own search database, so the result set is smaller and not identical to what a signed-in user sees, and it is not date-sorted unless you choose `recency`. As a result:

- **No result counts.** Neither the index nor the actor can tell you how many posts LinkedIn would return. The status reports how many indexed posts were discovered, not a LinkedIn total.
- **Recency is index-recency, not LinkedIn-recency.** The past-week and past-month filters were fully within their windows in testing, but the past-24-hours window often returned zero or one post because the index lags. For monitoring, run on a schedule with the past week/month window.
- **Connection degree is unavailable.** 1st/2nd/3rd+ is viewer-specific and needs a signed-in session. Older tasks that still send a connection-degree value keep working; it is ignored.
- **Author industry is unavailable.** Public post pages do not say which industry the author's company is in. Older tasks that still send an industry filter keep working; it is ignored. Adding an industry word to the query finds posts about that industry, not necessarily authors who work in it.
- **Mention filters use text, not LinkedIn entity IDs.** Numeric company/person IDs cannot be resolved without a session. Enter a name or slug: the actor searches for that text, which may include third-party mentions and may miss @mentions whose visible text omits the name.

### Company and school post feeds

Company and school post feeds are not supported. LinkedIn requires a sign-in for the “Posts” tab on company and school pages, and the logged-out page only shows the latest handful of posts (~6–14) with no way to load more. This actor searches the public post index instead: it can find public posts that mention a company or school, but it cannot return that entity's complete or paginated feed. If you need an entity's own posts, use the entity name as the query and check each result's author, or use a signed-in LinkedIn tool.

### Why are some engagement fields missing?

LinkedIn's plain public page embeds only the first ~10 comments and the totals; the richer public guest layout exposes up to ~18 comments (including author replies and commenter profile links), reaction-type breakdowns and repost counts, and the actor establishes an anonymous guest session per run so those are usually available. It never claims completeness: reading every comment, or viewer lists (likers) and view counts, requires a signed-in session and is not supported, so those fields are omitted rather than guessed. If the richer layout is not served for a post, the plain page still provides text, author, date, totals and up to ~10 comments. Counts are a snapshot of the page at the moment the post is opened.

### Why are some posts returned without full text?

A small share of indexed post pages show a sign-up wall to logged-out visitors. Those posts are still returned with their link, search snippet, and date, marked `contentSource: "serp"`, instead of being dropped or invented.

### What does `keywordMatch` mean?

It tells you whether every word of your query is visible in the post's text, title, or URL. Posts that do not match are skipped unless `includeUnmatched` is on, so the dataset does not quietly fill with loosely related posts.

### Do I need to set up a proxy?

No. The actor connects to the public index and to public post pages on its own, including a built-in fallback when a connection is refused.

### Is this affiliated with LinkedIn?

No. This is an independent, third-party actor. It reads only publicly visible pages, and it is not operated by or affiliated with LinkedIn.
