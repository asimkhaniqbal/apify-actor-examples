# X Keyword Search with Mention & Reply Operators

Search **X (Twitter)** by keyword, phrase or hashtag, then narrow results with operator-style filters most keyword scrapers skip — who a post mentions, who posted it, which handle it's replying to, or which conversation thread it belongs to — on top of date, language, engagement, media and verification filters. Every row tells you which keyword found it, so multi-keyword runs are easy to analyze. No login required.

## What this actor does

- **Keyword search with operators:** words, phrases, hashtags or full queries — each keyword runs as its own search
- **Advanced filters:** mentions, posted-by, replying-to, conversation ID and quote-tweets-only, on top of date, language, engagement, media and verification filters
- **Three result orders:** `Top`, `Latest` or `Top + Latest` (both passes merged, de-duplicated by post ID)
- **Up to 1,000 posts per keyword:** `maxItems` is the target per keyword; popular keywords can fill ~500 rows, low-volume keywords return only what exists
- **Row-level attribution:** every post carries `searchTerm` — the keyword that found it
- **Honest results:** a keyword with no data emits one `status` row with a message instead of silently returning nothing
- **Empty fields are omitted** — the dataset never contains `null`, `""`, `[]` or `{}`

## Output per post

- `id` — post ID
- `url` — direct post URL
- `text` — full post text, hashtags and mentions intact
- `createdAt` — ISO 8601 post time
- `language` — post language code when X provides it
- `searchTerm` — the keyword that found this row
- `replyCount`, `retweetCount`, `likeCount`, `quoteCount`, `bookmarkCount`, `viewCount` — engagement metrics
- `engagementRate` — derived `(likes + retweets + replies + quotes) / followers`; present only when the author's follower count is known and greater than zero
- `isReply`, `isRetweet`, `isQuote`, `isNoteTweet` — post type flags
- `conversationId`, `inReplyToTweetId`, `inReplyToUserId`, `inReplyToUsername`
- `hashtags`, `mentions`, `urls`, `cashtags`
- `media` — photo/video/GIF objects (`type`, `url`, `previewImageUrl`, dimensions, `durationSeconds`, `viewCount`, `altText`, `variants`)
- `quotedTweet`, `quotedTweetId`, `poll`, `card`
- `possiblySensitive`, `place`, `postedFrom` — extra labels when X exposes them
- `author` — author profile (`id`, `username`, `name`, `bio`, `followers`, `isBlueVerified`, `verified`, `profileImageUrl`, …)
- `type: "post"`, `collectedAt`

### Sample record

```json
{
  "type": "post",
  "id": "1881234567890123456",
  "url": "https://x.com/i/status/1881234567890123456",
  "text": "AI will change everything in the next 10 years. #AI",
  "createdAt": "2026-04-20T14:32:00+00:00",
  "language": "en",
  "searchTerm": "artificial intelligence",
  "replyCount": 1200,
  "retweetCount": 8500,
  "likeCount": 42000,
  "quoteCount": 310,
  "bookmarkCount": 3100,
  "viewCount": 9800000,
  "isReply": false,
  "isRetweet": false,
  "isQuote": false,
  "hashtags": ["AI"],
  "author": {
    "id": "44196397",
    "username": "elonmusk",
    "name": "Elon Musk",
    "profileImageUrl": "https://pbs.twimg.com/profile_images/.../photo_400x400.jpg",
    "followers": 241000000,
    "isBlueVerified": true
  },
  "collectedAt": "2026-05-05T13:42:18+00:00"
}
```

## Output per status row

A `status` row is emitted when a keyword cannot return data. It carries `type: "status"`, `status`, `message`, `searchTerm` (the keyword it applies to) and `collectedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keywords` | array | – | **Required.** Words, phrases, hashtags or full queries. Each is searched separately. |
| `maxItems` | integer | `20` | Posts to collect per keyword (1–1000). In `Top + Latest` mode the cap is shared across both passes. |
| `maxTweets` | integer | – | Alias of `maxItems`. Either name is accepted. |
| `sort` | string | `Top` | `Top` — most relevant and popular; `Latest` — newest first; `Top + Latest` — both passes combined without duplicates. |
| `searchType` | string | – | Alias of `sort`. |
| `language` | string | `Any language` | Only posts in this language (40 common X languages, e.g. `en`, `es`, `ja`). |
| `dateFrom` | date | – | Only posts published on or after this date. |
| `dateTo` | date | – | Only posts published on or before this date. |
| `excludeRetweets` | boolean | `false` | Drop retweets/reposts. |
| `excludeReplies` | boolean | `false` | Drop replies and keep top-level posts. |
| `onlyVerifiedUsers` | boolean | `false` | Only posts from verified accounts. |
| `onlyImages` | boolean | `false` | Only posts with at least one image. |
| `onlyVideos` | boolean | `false` | Only posts with at least one video or GIF. |
| `onlyQuotes` | boolean | `false` | Only quote-tweets. |
| `minimumLikes` | integer | – | Only posts with at least this many likes. |
| `minimumRetweets` | integer | – | Only posts with at least this many retweets. |
| `minimumReplies` | integer | – | Only posts with at least this many replies. |
| `mentioning` | string | – | Only posts that mention this handle (with or without `@`). |
| `author` | string | – | Only posts published by this handle (with or without `@`). |
| `inReplyTo` | string | – | Only posts that reply to this handle (with or without `@`). |
| `conversationId` | string | – | Only posts inside this conversation (root post ID). |
| `proxyConfiguration` | object | – | Optional. Recommended for large or repeated runs from the same IP. |
| `startDate` / `endDate` | string | – | Legacy aliases of `dateFrom` / `dateTo` (`YYYY-MM-DD`). |
| `tweetLanguage` | string | – | Legacy alias of `language`. |

### Example: posts about your brand that mention a handle

```json
{
  "keywords": ["pricing"],
  "mentioning": "yourbrand",
  "maxItems": 200,
  "sort": "Latest",
  "language": "en"
}
```

### Example: quote-tweets from a specific account

```json
{
  "keywords": ["launch"],
  "author": "openai",
  "onlyQuotes": true,
  "dateFrom": "2026-01-01",
  "dateTo": "2026-06-30"
}
```

### Example: everything inside one conversation

```json
{
  "keywords": ["q4 earnings"],
  "conversationId": "1881234567890123456",
  "sort": "Top + Latest",
  "maxItems": 500,
  "excludeRetweets": true
}
```

## Use cases

- **Brand and reputation monitoring** — mentions, replies and quote-tweets around your brand, with author and reply context
- **Competitive intelligence** — track what a specific competitor's account posts around a topic
- **Crisis and event tracking** — follow a conversation thread or a tight date window with `Latest`
- **Media and PR analysis** — find the highest-engagement posts about a campaign
- **Academic research** — build filtered datasets of public discourse by language, date and engagement
- **Community management** — surface replies to a handle that need response

## FAQ

**Do I need an X account, login or developer key?**  No. The actor reads the same public search results any visitor sees on X without signing in, and there is no credential input at all.

**What makes the operator filters different from a plain keyword search?**  `mentioning`, `author`, `inReplyTo`, `conversationId` and `onlyQuotes` let you narrow a keyword search the same way X's own advanced search operators do, without writing the operator syntax yourself. Combine any of them with the standard date/language/engagement filters and the `Top + Latest` merged order.

**How many posts can I get per keyword?**  You can set `maxItems` up to 1,000. Popular keywords can fill around 500 rows; niche or low-volume keywords return only the posts that exist. `maxItems` is a target, not a guarantee, and in `Top + Latest` mode it is shared across both passes.

**Why are results different between runs?**  X's public search surface answers intermittently and does not expose full pagination to logged-out visitors. Coverage varies by keyword, network and time. Running again or using smaller date windows usually adds rows.

**`Top`, `Latest` or `Top + Latest`?**  `Latest` for real-time monitoring, `Top` for the highest-engagement posts first, `Top + Latest` when you want both merged without duplicates.

**Do I need a proxy?**  No — a proxy is optional. It helps when large or repeated runs hit IP-based limits.

**How fresh is the data?**  Live — every run searches X at request time. Schedule the actor for hourly or daily monitoring.

**How do I integrate the results?**  Use the dataset from the run page, the Apify API, webhooks, or a no-code connection such as Zapier, Make or Google Sheets.

**Is this affiliated with X?**  No. This is a third-party actor that reads publicly available search results and is not affiliated with or endorsed by X Corp.
