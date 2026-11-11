# LinkedIn Public Post Scraper — Profile Feeds, Comments & Video

Collect public LinkedIn posts without logging in. Give the actor one or more post links, or a profile URL, and it returns the post text, the author and their follower count, reaction and comment totals, the comments LinkedIn shows to logged-out visitors, images, videos with duration and (optionally) the full video transcript. No account, no cookie, no browser session.

## Ways to use it

1. **Specific posts** — paste any LinkedIn post link (`linkedin.com/posts/...`), a feed-update link (`/feed/update/urn:li:activity:...`, also `ugcPost`), or just the numeric activity ID. Country subdomains (`uk.linkedin.com/...`) work too.
2. **A profile's recent posts** — provide a profile URL and the actor reads the profile's public activity feed, then opens each post it finds for the full record.

Set `mode` to force one pipeline, or leave it on `Auto` and the actor figures it out from the URLs you supply.

## What each row contains

- `postId`, `postUrl`, `postedAt` (ISO 8601), `inputUrl`, `scrapedAt`
- `authorName`, `authorProfileUrl`, `authorAvatar`, `authorFollowers`
- `content` — the full post text, exactly as published
- `hashtags` — pulled out of the text
- `reactionsCount`, `commentsCount`
- `comments` — the newest comments visible to logged-out visitors: `authorName`, `text`, `postedAt`, and `reactionsCount` when the comment has likes
- `mediaType` — `text`, `image`, `video` or `repost`
- `mediaUrls` / `mediaItems` — image links, or the MP4 link plus its poster frame for videos
- `videoDuration` (e.g. `PT1M14S`), `videoTranscript` when you turn it on
- `isRepost` and `repostOf` (original author and reshared text) for reshares

Optional extras with a session cookie for people who still want them: `likers` (first visible page), `commenters` (with profile URLs), `reactionBreakdown`, `repostsCount` and `viewsCount` — plus profile feeds that paginate past the logged-out ceiling. See the cookie note below.

## Examples

A single post:

```json
{
  "mode": "postUrls",
  "postUrl": "https://www.linkedin.com/posts/nvidia_ai-is-technology-that-is-built-and-improved-activity-7508569677040893954-sffp",
  "maxPosts": 1
}
```

A batch of posts with quality filters:

```json
{
  "mode": "postUrls",
  "postUrls": [
    "https://www.linkedin.com/posts/nvidia_ai-is-technology-that-is-built-and-improved-activity-7508569677040893954-sffp",
    "urn:li:activity:7469496437156925441"
  ],
  "minReactions": 25,
  "includeReposts": false,
  "postedLimit": "month"
}
```

A profile's latest posts with their top comments:

```json
{
  "mode": "profilePosts",
  "profileUrl": "https://www.linkedin.com/in/mohamed-rhioui",
  "maxPosts": 10,
  "maxComments": 10
}
```

## Input reference

| Field | What it does | Default |
|---|---|---|
| `mode` | `auto`, `postUrls`, or `profilePosts` | `auto` |
| `profileUrl` | Profile URL or bare username (profile-feed mode) | – |
| `postUrl` / `postUrls` | One or many post links, feed-update links, URNs, or activity IDs | – |
| `maxPosts` | Upper bound on returned posts | `10` (max 200) |
| `postedLimit` | Any time / past hour / 24h / week / month / 3 / 6 months / year | Any time |
| `postedAfter`, `postedBefore` | Exact date window (`2026-01-31` or ISO 8601) | – |
| `maxComments` | Top comments kept per post (LinkedIn caps this at 10) | `10` |
| `includeVideoTranscript` | Add the spoken transcript of video posts | `false` |
| `includeReposts` | Keep or drop reshares | `true` |
| `minReactions`, `minComments` | Skip low-engagement posts | `0` |
| `mediaTypes` | Restrict to text / image / video / repost (and other legacy categories) | all |
| `language` | Language hint for localized content | `en` |
| `scrapeInteractors` | Optional cookie-only extra: add `commenters` (with profile URLs/headlines) | `false` |
| `scrapeReactions` | With `scrapeInteractors`, also add the first page of `likers` | `false` |
| `maxInteractors` | Cap per likers list and per commenters list | `50` |
| `cookie` | Optional. Unlocks feed pagination + reaction breakdowns/likers/commenters | empty |
| `cookies` | Optional. Same as `cookie`, as a browser cookie-export JSON array | – |

## Comments and limits, honestly

- **Public posts only.** Posts that LinkedIn reserves for signed-in members, or that were deleted, are skipped and the run still finishes with the posts it could read.
- **Profile feeds show a preview, not an archive — unless you bring a cookie.** Without one, LinkedIn's logged-out activity page lists only the newest few public posts (usually 8). There is no pagination for visitors who are not signed in. With your own session cookie the actor paginates the profile feed up to `maxPosts`.
- **Comments are the visible pageful.** Logged-out post pages embed roughly the ten most recent comments, and the cookie path adds commenter profile URLs to those same first ~10. LinkedIn's "more comments" pagination is not available to visitors.
- **Likers and reaction-type breakdowns are account-scoped.** LinkedIn shows those lists only to signed-in members. If you paste a session cookie from your own browser, the actor adds the first visible page of likers, the reaction-type breakdown, reposts/views counts and commenter profiles. LinkedIn often rejects sessions coming from a server IP, so treat cookie extras as best-effort. Everything in the output still works with no cookie at all.
- **Repost flags survive both page versions.** Reshares are detected from the full-page `sharedContent` block or from the lighter page's "reposted this" header, whichever LinkedIn serves.

## FAQ

**Do I need a LinkedIn account or cookie?** No. The actor reads the same public pages a logged-out visitor sees.

**How current is the data?** Every run fetches the pages live. `scrapedAt` tells you when each row was captured.

**Which post link formats are accepted?** `linkedin.com/posts/...` share links, `/feed/update/urn:li:activity:...` and `urn:li:ugcPost:...` links, bare activity IDs, and country subdomains such as `uk.linkedin.com`.

**Can it scrape company pages or keyword-search LinkedIn?** Those surfaces are login-gated at the publicly visible layer, so this actor focuses on profiles and individual posts.

**Why did a specific URL return nothing?** Either the post is member-only/removed, or it does not exist. The run log lists each URL with the reason.

**Do I need a proxy?** No. The actor selects its own connection and escalates only when a page request is challenged.
