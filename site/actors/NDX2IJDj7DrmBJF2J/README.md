# Threads Public Post, Reply & Profile Data Extractor

Pull public data from Threads — a profile's posts, replies, media or reposts, keyword and hashtag results, discovered profiles, or a single post with its full reply thread. Every record ships with real engagement counts, post text, media links, hashtags, mentions, and profile details, all visible to a logged-out visitor. No Threads or Instagram account, cookie, token, or proxy setup is needed.

## What this actor does

- **Eight modes:** profile posts, profile replies, profile media, profile reposts, keyword search, hashtag/topic search, profile discovery by keyword, and a post plus its reply thread.
- **Deep pagination:** profiles and reply threads keep loading page after page, so a run can collect hundreds of posts instead of the handful visible on first load.
- **Complete engagement data:** likes, replies, reposts, quotes, and reshares, plus photo, video, and carousel links, link cards, hashtags, mentions, and outbound URLs.
- **Profile records:** bio, follower count, verification, profile pictures, bio links, topics, and account visibility, emitted alongside a profile mode's posts or through profile discovery.
- **Date windows:** keep only posts published after or before a date, using ISO dates or relative windows such as `7d`.
- **Working media links:** photos and videos are copied into the run's key-value store before Threads' signed links expire, with the original link and expiry kept beside each copy.
- **Empty fields are omitted** — no `null`, empty string, or placeholder is ever written.

## Output per post and reply

| Field | Description |
|---|---|
| `recordType` | `"post"` for post and reply records. |
| `mode` | Mode that produced the record. |
| `postId` | Threads' numeric post id. |
| `shortcode` | Short code used in the post URL. |
| `postUrl` | Full post permalink. |
| `authorUsername`, `authorFullName`, `authorPk` | Author identity. |
| `authorIsVerified` | Whether the author has a verification badge. |
| `authorProfilePicUrl` | Author avatar link. |
| `postText` | Post or reply text. |
| `publishedAt` | Publish time, ISO 8601 UTC. |
| `likeCount`, `commentCount`, `repostCount`, `quoteCount`, `shareCount` | Engagement counts as Threads displays them. `commentCount` is the reply count and `shareCount` the reshare count. |
| `isReply`, `parentPostId`, `replyToAuthorUsername`, `rootPostAuthorUsername` | Reply context when Threads provides it. |
| `threadRootPostId`, `threadPosition`, `threadItems` | Position inside the author's own thread and the other items in that thread. |
| `mediaType` | Threads' media type number (1 photo, 2 video, 8/19 carousel). |
| `hasImage`, `hasVideo` | Whether the post carries a photo or video (always present). |
| `imageUrl`, `imageUrls` | Best photo link and all photo links. |
| `videoUrl`, `videoUrls`, `videoThumbnailUrl` | Best video link, all video versions, and the video cover image. |
| `carouselMedia` | Per-slide media type, links, and dimensions for carousel posts. |
| `imageUrlOriginal`, `videoUrlOriginal`, `imageUrlsOriginal`, `videoUrlsOriginal`, `carouselMediaOriginal` | Original Threads links (present when media is rehosted). |
| `imageUrlExpiresAt`, `videoUrlExpiresAt`, `imageUrlsExpiresAt`, `videoUrlsExpiresAt` | Expiry time of the original signed links. |
| `externalLinkUrl`, `externalLinkDomain`, `externalLinkTitle` | Link-card target, domain, and title. `linkPreviewUrl`, `linkPreviewDomain`, `linkPreviewTitle` carry the same values. |
| `hashtags`, `mentions`, `outboundUrls` | Arrays parsed from the post text. |
| `isPaidPartnership` | Present when Threads marks a paid partnership. |
| `isMainPost` | In post-reply mode: `true` for the post itself, `false` for replies. |
| `scrapedAt` | Collection time, ISO 8601 UTC. |

## Output per profile

| Field | Description |
|---|---|
| `recordType` | `"profile"`. |
| `mode` | Profile mode that produced the record, or `profilesByKeyword`. |
| `username`, `profileId` | Handle and numeric account id. |
| `profileUrl` | Full profile link. |
| `displayName`, `bio` | Display name and bio text. |
| `followersCount`, `threadsCount` | Follower and thread counts as shown publicly. |
| `isVerified` | Present and `true` only when the account is verified. |
| `profilePicUrl`, `hdProfilePicUrl` | Avatar links. |
| `bioLinks`, `profileTopics` | Links found in the bio and topic labels. |
| `isPrivate` | Whether the account is marked private. |
| `publicViewCount` | Public view count when Threads exposes one. |
| `instagramUrl` | Connected Instagram profile link, when shown. |
| `scrapedAt` | Collection time, ISO 8601 UTC. |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `profile` | `profile`, `profileReplies`, `profileMedia`, `profileReposts`, `search`, `tag`, `profilesByKeyword`, or `postReplies`. |
| `usernames` | array | – | Handles or profile URLs (profile modes). |
| `searchQueries` | array | – | Keywords, phrases, or hashtags (`search`, `profilesByKeyword`). |
| `tag` | string | – | Hashtag or topic without `#` (`tag`). |
| `postUrls` | array | – | Post links (`postReplies`). |
| `sortBy` | string | `top` | `top` or `recent` for keyword search (`default` is accepted as an alias of `top`). |
| `includeProfileInfo` | boolean | `true` | Also output one profile record per username in profile modes. |
| `postedAfter` | string | – | Lower publish-date bound (ISO date or `7d`, `24h`, `2w`). |
| `postedBefore` | string | – | Upper publish-date bound. |
| `includeMediaUrls` | boolean | `true` | Copy photos and videos into the key-value store so links keep working. |
| `maxItems` | integer | `100` | Record cap (1–5000). Profile records do not count in profile modes. |

### Example: a profile's recent posts

```json
{
  "mode": "profile",
  "usernames": ["zuck"],
  "includeProfileInfo": true,
  "maxItems": 100
}
```

### Example: replies written by two accounts in the last two weeks

```json
{
  "mode": "profileReplies",
  "usernames": ["zuck", "mosseri"],
  "postedAfter": "14d",
  "maxItems": 200
}
```

### Example: keyword search, newest results first

```json
{
  "mode": "search",
  "searchQueries": ["openai", "#buildinpublic"],
  "sortBy": "recent",
  "maxItems": 50
}
```

### Example: a post with its full reply thread

```json
{
  "mode": "postReplies",
  "postUrls": ["https://www.threads.com/@zuck/post/DZpPDXbCeTt"],
  "maxItems": 300
}
```

## Use cases

- **Social listening** — follow keywords or hashtags and route new posts into a monitoring dashboard.
- **Competitor tracking** — compare engagement across accounts and export post histories for trend charts.
- **Influencer research** — collect follower counts, bios, topics, and recent performance before a partnership.
- **Community analysis** — export a post's reply thread and measure how a conversation develops.
- **Content planning** — study which formats and topics earn the most likes, replies, and reposts in a niche.
- **Academic research** — archive public conversations with stable, structured metadata for later analysis.

## Limitations

- Threads shows logged-out visitors a fixed amount of keyword and hashtag search results and does not page beyond it. Profile feeds, media, reposts, and reply threads do paginate deeply; search is capped at roughly one screen of results.
- Keyword profile discovery returns one page of up to about 100 profiles per keyword. Threads does not issue another page for that view.
- A reply thread can be shorter than the reply count shown on a post: Threads filters what a logged-out visitor sees. The run status message reports the shortfall when it happens.
- Quote posts cannot be listed anonymously; Threads shows them only in a signed-in activity view. Quote counts are still returned for every post.
- Some accounts require a Threads login before any content is shown. Those accounts are reported in the run status message and skipped; they are never filled with placeholder records.
- A post that Threads itself cannot serve anonymously (for example, a quote of a removed post or a broken poll) can interrupt a feed. The actor continues up to that item and reports where it stopped.
- Photos and videos use signed links that expire after a few days. With media rehosting on, copies are stored in the run's key-value store and the original link plus expiry stay in the record.
- Private accounts and private posts are not accessible without an account, so they are not returned.

## FAQ

**Do I need a Threads or Instagram account?** No. The actor reads only what Threads shows to logged-out visitors.

**Do I need to configure a proxy?** No. The actor chooses a working connection automatically; there is nothing to set up.

**How many posts can I get from one profile?** Profile feeds page continuously while Threads keeps returning results. Popular accounts commonly return hundreds of posts in one run; the `maxItems` setting is the only cap.

**Why do I get fewer results than expected from search?** Threads limits logged-out keyword and hashtag search to a single screen of results. For deeper coverage, collect from the accounts or post links you care about instead.

**Are the engagement counts exact?** They are the counts Threads displays publicly, which use abbreviations such as `1.2K` for larger numbers.

**Why are some media links replaced by Apify links?** Threads' media links are signed and expire. Rehosting stores a durable copy and keeps the original link and its expiry in the record.

**Why do I get fewer replies than the number on the post?** Threads filters reply threads for logged-out visitors. The actor returns every reply Threads serves and states the shortfall in the run status when the public count is higher.

**Can I collect private accounts?** No. Only public profiles, posts, and replies are visible without logging in.

**What happens when nothing is visible?** The run finishes with an explanatory status message and an empty dataset; no placeholder records are written.

**Is this affiliated with Threads or Meta?** No. This is an independent third-party tool, not affiliated with, endorsed by, or sponsored by Meta Platforms, Inc.
