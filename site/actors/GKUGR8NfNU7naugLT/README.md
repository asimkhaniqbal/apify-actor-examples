# LinkedIn Trending Posts & Top Voices Feed Scraper

Find out what's actually trending on LinkedIn right now. This actor pulls posts from LinkedIn's own trending/top-content surfaces — optionally narrowed to a topic keyword such as "AI" or "leadership" — and returns each post with its author, full text, media type, and engagement numbers so you can see what's winning attention in a niche without scrolling the feed yourself.

## What this actor does

- Fetches currently trending LinkedIn posts, optionally filtered to a single topic keyword
- Narrows results by publish-date window, media type, and minimum reaction/comment thresholds
- Returns reaction, comment, and repost totals plus a breakdown by reaction type (like, celebrate, love, insightful, support, funny, curious)
- Includes the author's name, profile URL, and headline for every post
- Omits any field it could not find data for — you will not see empty strings or nulls in a record

## Output per post

- `postId` — LinkedIn activity ID of the post
- `postUrl` — direct link to the post
- `authorName` — full name of the author
- `authorProfileUrl` — author's LinkedIn profile URL
- `authorHeadline` — author's headline or job title
- `postedAt` — ISO 8601 UTC timestamp the post was published
- `content` — full text of the post
- `mediaType` — `text`, `image`, `video`, `article`, `document`, `carousel`, `repost`, `poll`, or `event`
- `mediaUrls` — CDN URLs of attached images (when the post has image media)
- `articleTitle` / `articleUrl` — title and link of a shared article (when `mediaType` is `article`)
- `reactionsCount` — total reactions on the post
- `commentsCount` — total comments on the post
- `repostsCount` — total reposts/shares
- `reactionBreakdown` — per-type counts (like, celebrate, love, insightful, support, funny, curious)
- `scrapedAt` — ISO 8601 UTC timestamp when the record was scraped

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `maxPosts` | integer | `50` | Maximum number of posts to return (1–500) |
| `category` | string | – | Topic keyword to narrow results (e.g. `AI`, `leadership`, `marketing`); leave empty for general trending posts |
| `dateRange` | string | `""` (any time) | Publish window: any time, past 24 hours, past week, or past month |
| `language` | string | `""` (auto/English) | Preferred response language: English, Spanish, French, German, or Portuguese |
| `mediaTypeFilter` | string | `""` (all types) | Return only one media type: text, image, video, article, document, repost, or poll |
| `minReactions` | integer | – | Only return posts with at least this many reactions |
| `minComments` | integer | – | Only return posts with at least this many comments |
| `cookie` | string | – | LinkedIn session cookie (`li_at` value or exported cookies JSON array) for access to more posts |
| `proxyConfiguration` | object | – | Apify proxy configuration |

### Example: general trending feed

```json
{
    "maxPosts": 50
}
```

### Example: AI posts from the last week with a reaction floor

```json
{
    "category": "AI",
    "dateRange": "7d",
    "minReactions": 500,
    "maxPosts": 100
}
```

### Example: video posts only, in Spanish

```json
{
    "mediaTypeFilter": "video",
    "language": "es",
    "maxPosts": 30
}
```

## Use cases

- **Content strategists** spotting which post formats and topics are winning engagement before planning next week's calendar
- **Social listening** teams tracking how a topic (e.g. "layoffs", "AI agents") is trending on LinkedIn over the past day or week
- **Influencer research** — finding Top Voices and high-engagement authors inside a given niche
- **Agencies** building swipe files of real, currently-trending posts for client pitches
- **Analysts** comparing reaction-type mixes (celebratory vs. insightful) across topics or time windows

## FAQ

**Do I need a LinkedIn account?** No. The actor can run with no cookie and still returns trending posts, though supplying a session cookie typically surfaces more results.

**What does `category` actually filter?** It's a topic keyword — the same kind of term you'd type into LinkedIn's own trending-topics browser, like `AI` or `marketing`. Leave it empty for the general, unfiltered trending feed.

**Why is `reactionBreakdown` sometimes missing a reaction type?** LinkedIn only reports counts for reaction types that were actually used on that post; a type with zero reactions is left out rather than shown as `0`.

**Can I combine `minReactions` and `minComments`?** Yes — both thresholds apply together, so a post must clear both minimums to be returned.

**Why don't all posts have `articleTitle` or `mediaUrls`?** Those fields only apply to posts whose `mediaType` is `article` or `image`; a text-only or video post simply won't have them.

**How fresh is the data?** Every run queries LinkedIn's trending surfaces live — there's no cached or delayed dataset behind this actor.

**Is this affiliated with LinkedIn?** No. This is an independent actor that reads LinkedIn's public and session-authenticated trending-content surfaces; it is not endorsed by or affiliated with LinkedIn Corporation.
