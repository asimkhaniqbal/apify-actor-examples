# Twitter Keywords Scraper Pro

Collect public X (Twitter) posts without logging in, without cookies and without an API key.
Search by keyword, walk an author's recent timeline, fetch exact post URLs, or open a
conversation thread — then export clean rows with the author, engagement counts, media,
quoted post and reply context already filled in.

> **Coverage note:** X puts its own full search behind a login. This actor uses the public
> Bing News x.com index plus public post pages, so it returns a real subset of public X
> posts, not X's complete search index. That hard limit is explained below.

## What you get

Every dataset row is one post (`type: "post"`) with:

- **Post**: `id`, `url`, `text`, `createdAt`, `language`, `conversationId`
- **Engagement**: `likeCount`, `replyCount`, `retweetCount`, `quoteCount`, `bookmarkCount`,
  `viewCount`, `engagementRate`
- **Flags**: `isReply`, `isRetweet`, `isQuote`, `isNoteTweet`, `possiblySensitive`
- **Reply context**: `inReplyToTweetId`, `inReplyToUserId`, `inReplyToUsername`, `parentTweet`
- **Quote context**: `quotedTweetId`, nested `quotedTweet`
- **Entities**: `hashtags`, `mentions`, `urls`, `cashtags`
- **Media**: photo, video or GIF with `url`, `previewImageUrl`, `width`, `height`,
  `variants`, `durationSeconds`, `altText`
- **Author**: `id`, `username`, `name`, `bio`, `followers`, `following`, `location`,
  `website`, `profileImageUrl`, `bannerUrl`, `createdAt`, `postCount`, `mediaCount`,
  `verified`, `isBlueVerified`
- **Attribution**: `searchTerm` (keyword, handle, URL or conversation ID) and `collectedAt`

Empty values are omitted at every depth. Missing counters are omitted, never invented as
zero. An empty run writes a `RUN_SUMMARY` to the key-value store and leaves the dataset
empty — there are no status or sentinel rows.

## Four modes

| Mode | Input | What it collects |
|---|---|---|
| `search` | `keywords` | Public posts found for each keyword through the Bing News x.com index, optionally supplemented by X guest search when you supply a proxy. |
| `byAuthor` | `authorHandles` | The profile page's metadata and recent posts, plus the public syndication timeline (up to ~21 posts per author). |
| `byUrls` | `postUrls` | Exact posts from URLs or IDs, using the post page, syndication, FxTwitter and oEmbed fallbacks. |
| `conversation` | `conversationId` | The focal post plus the reply previews X renders publicly, with the focal post kept first. |

## Quick start

### Keyword search

```json
{
  "mode": "search",
  "keywords": ["apify", "#AI", "OpenAI"],
  "maxItems": 50,
  "sort": "Latest",
  "language": "en"
}
```

### Posts by author

```json
{
  "mode": "byAuthor",
  "authorHandles": ["@apify", "https://x.com/OpenAI"],
  "maxItems": 20
}
```

### Specific post URLs

```json
{
  "mode": "byUrls",
  "postUrls": [
    "https://x.com/apify/status/2118015298603274372",
    "2117931222320259247"
  ]
}
```

### Conversation thread

```json
{
  "mode": "conversation",
  "conversationId": "https://x.com/apify/status/2118015298603274372",
  "maxItems": 20
}
```

## Inputs

| Input | Type | Default | Notes |
|---|---|---|---|
| `mode` | select | `search` | `search`, `byAuthor`, `byUrls` or `conversation`. Required. |
| `keywords` | string list | – | Used by `search`. A single string is also accepted. |
| `authorHandles` | string list | – | Used by `byAuthor`. Accepts `@handle`, `handle` or a profile URL. |
| `postUrls` | string list | – | Used by `byUrls`. Accepts post URLs and bare IDs. |
| `conversationId` | text | – | Used by `conversation`. Accepts an ID or a post URL. |
| `maxItems` | integer 1–1000 | `20` | Per keyword/author in search and author modes; total cap in URL and conversation modes. |
| `sort` | select | `Top` | `Top` (engagement), `Latest` (newest), `Top + Latest` (top half by engagement, then the newest remaining posts). |
| `language` | select | `Any language` | Any language or an ISO-639-1 code. Applied after collection. |
| `dateFrom`, `dateTo` | date pickers | – | Inclusive UTC date bounds. |
| `excludeRetweets`, `excludeReplies` | boolean | `false` | Drop reposts or replies. |
| `onlyVerifiedUsers` | boolean | `false` | Keep verified or blue-check accounts. |
| `onlyImages`, `onlyVideos`, `onlyQuotes` | boolean | `false` | Keep photo posts, video/GIF posts, or quote posts. |
| `minimumLikes`, `minimumRetweets`, `minimumReplies` | integer | – | Drop rows below the threshold; rows whose count is unavailable are dropped and reported. |
| `mentioning`, `author`, `inReplyTo` | text | – | Client-side operator filters: mention handle, author handle, reply target handle. |
| `proxyConfiguration` | proxy | – | Optional. Nothing needs a proxy; when supplied it is also used for the best-effort guest search supplement. |

No cookie, token, API key or proxy field is required. Legacy saved inputs that use
`maxTweets`, `searchType`, `startDate`, `endDate` or `tweetLanguage` keep working: the actor
maps those keys in code even though they no longer appear in the form.

## How collection works

1. **Discovery (`search`)** — queries Bing News RSS for `site:x.com <keyword>` with plain
   variants (the keyword, quoted phrase, `x.com`, `twitter`, `news`, `today`, `2026`,
   `update`, `announcement`) and collects unique post IDs. A small job stops after one
   request; large jobs page through `first=13,26,39,52` with a roughly 460-ID ceiling.
2. **Hydration** — fetches each ID from the public syndication CDN in parallel. Deleted,
   protected and tombstoned posts are skipped and counted in the summary. Local probes
   hydrated 197 of 202 IDs (97.5%) and 193 of 199 (97.0%).
3. **Profile and post pages** — `byAuthor` reads the logged-out `x.com/<handle>` page (exact
   profile metadata and recent posts) and falls back to the syndication profile timeline.
   `byUrls` and `conversation` read the logged-out post page, which embeds the focal post,
   quoted post and visible reply previews.
4. **Fallback enrichment** — FxTwitter fills missing view/bookmark/retweet/quote counters
   and author profile fields. oEmbed is the last resort for a known post ID. A mirror
   outage never fails a run; missing fields stay omitted.
5. **Client-side filters** — X-style operators (`mentioning`, `author`, `inReplyTo`) and
   every date/language/engagement/media/verification/flag filter are applied locally after
   hydration. The index never receives opaque operator text.
6. **Deterministic output** — dedupe by post ID, sort, cap, recompute `engagementRate` and
   emit omit-empty rows.

## Coverage and honest limits

- **Full X search requires a login.** X's search page redirects logged-out visitors to a
  login wall, and the guest SearchTimeline endpoint answers only a minority of requests
  and virtually never from datacenter IPs. This actor therefore uses the public index as
  its primary discovery path. Building full search coverage would require a logged-in
  account pool or a paid provider, which this actor deliberately does not use.
- **The index is smaller than X search.** A keyword usually yields tens to a few hundred
  public posts, weighted toward newsworthy and recently linked posts. Broad one-word
  keywords can hit the roughly 460-candidate discovery ceiling; narrow keywords may yield only a
  handful.
- **`Top` is an engagement sort.** It orders by likes and reposts; it is not X's private
  ranking model. `Top + Latest` combines an engagement half with the newest remaining posts.
- **Conversation depth is a preview.** Logged-out X renders the focal post plus a few
  visible replies, so a thread returns those posts, not every reply.
- **Not covered:** geo/place filters, People/Lists tabs, follower/following lists,
  sentiment/AI scoring and full-text search beyond the index. These need endpoints that
  are not public without a login.

## FAQ

**Do I need cookies, a login or an API key?**
No. The actor is cookieless and keyless. A proxy is optional; the default run works without
one and never creates a paid proxy group on its own.

**Why do I get fewer posts than the X search box shows?**
X's full search is login-gated. The actor reads the public Bing News x.com index and public
post pages, which cover a real but smaller subset of public posts. That is the documented
hard limit, not a bug.

**How many posts can one keyword return?**
The discovery buffer is capped at roughly 460 unique IDs, but the practical yield is usually tens
to a few hundred per keyword, depending on how many public posts the index knows. `maxItems`
still caps what is emitted.

**How fresh are the results?**
Discovery favours recently indexed and newsworthy posts, and each post's exact timestamp
comes from X's public payload. Use `dateFrom`/`dateTo` to keep only the window you need.

**Does `Top` match X's Top tab?**
No. X does not expose its ranking model publicly, so `Top` sorts the collected posts by
likes and reposts. `Latest` sorts by the real post timestamp.

**Why did a run return nothing?**
The dataset stays empty and the key-value store `RUN_SUMMARY` says why: no indexed posts,
all candidates deleted/tombstoned, every post removed by filters, or invalid input. The
status message shows the same reason. There are no placeholder rows.

**Why are view or bookmark counts missing on some rows?**
Syndication does not expose every counter. The actor asks FxTwitter to fill the gaps, but
when the mirror also lacks a value the field is omitted rather than fabricated.

**Can I get replies and quotes?**
Yes. Reply IDs/handles and the parent post are emitted when X exposes them, and quoted posts
are nested under `quotedTweet` with `quotedTweetId`. `conversation` mode adds the visible
reply previews of a thread.

**Is the actor affiliated with X?**
No. It reads only public, logged-out pages and public mirrors. It does not bypass a login,
sell account access, or require your credentials.

## Output summary

The default dataset view is called **Posts**. For every run, the `RUN_SUMMARY` key in the
default key-value store records `status` (`complete`, `partial` or `empty`), mode, candidate
and hydrated counts, emitted rows, dropped-filter counts, index coverage and elapsed time.
