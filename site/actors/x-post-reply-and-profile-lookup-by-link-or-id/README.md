# X Post, Reply & Profile Lookup by Link or ID

Give this actor a post link, a bare post ID, or a profile handle and it returns structured public data: the post's full text, author, media, poll, quoted and parent posts, engagement counts and replies — or that account's public profile. Optional quote collection turns a single post into the conversation around it. No login at any point.

## What this actor does

- **Posts by link or ID:** accepts `x.com` and `twitter.com` links, `x.com/i/web/status` links, bare post IDs, profile links and bare handles
- **Replies:** walks the replies X exposes to logged-out visitors, starting from the post's publication time, and de-duplicates them by reply ID
- **Quotes:** optionally collects the posts that quote the requested post, each with its text, author and metrics (off by default)
- **Retweeters:** optionally collects the handles of accounts that reposted the requested post (off by default; X exposes only a search-index sample of visible reposts)
- **Rich nested data:** optional parent post, quoted post, poll, link-preview card, photo/video variants and the author's public profile on every row
- **Profile input:** a profile link or handle returns one `profile` row with that account's public bio and counters
- **Honest rows:** every row carries a `type` (`post`, `reply`, `quote`, `profile`, `status`); deleted, protected or rate-limited links produce a `status` row with a message instead of failing the run
- **Empty fields are omitted** — the dataset never contains `null`, `""`, `[]` or `{}`

## Output per post or reply

Each row describes one post. `type` is `post` for requested posts and `reply` for replies collected from the conversation.

- `id` — post ID
- `url` — direct post URL
- `text` — full post text, multi-line and links intact
- `createdAt` — ISO 8601 post time
- `language` — post language code when X provides it
- `replyCount`, `retweetCount`, `likeCount`, `quoteCount`, `bookmarkCount`, `viewCount` — engagement metrics (views when X exposes them)
- `engagementRate` — derived `(likes + retweets + replies + quotes) / followers`; present only when the author's follower count is known and greater than zero
- `isReply`, `isQuote`, `isRetweet`, `isNoteTweet` — post type flags
- `conversationId` — the thread root of the post
- `threadRootId`, `threadPosition` — position of the row inside the collected thread
- `inReplyToTweetId`, `inReplyToUserId`, `inReplyToUsername` — where a reply points
- `hashtags`, `mentions`, `urls`, `cashtags` — entities found in the text
- `media` — photo/video/GIF objects
- `poll` — poll choices and status
- `card` — link-preview card (`title`, `description`, `domain`, `url`, `thumbnail`)
- `quotedTweet`, `quotedTweetId` — the quoted post when the input post quotes another (removed, together with `isQuote`, when `includeQuotedTweet` is off)
- `parentTweet` — the post being replied to when the input post is a reply
- `threadAncestors` — the chain of posts above a reply, root first (when `includeThreadAncestors` is on)
- `author` — author profile snapshot
- `authorProfile` — fuller author profile attached to every row
- `possiblySensitive`, `place`, `postedFrom` — extra labels when X exposes them
- `type`, `collectedAt`

**Media item:** `type` (`photo` / `video` / `gif`), `url`, `previewImageUrl`, `width`, `height`, `durationSeconds`, `viewCount`, `altText`, `variants` (each variant has `url`, `contentType`, `bitrate`).

**Poll:** `choices` (label + votes), `countsAreFinal`, `endsAt`, `durationMinutes`.

**Author:** `id`, `username`, `name`, `bio`, `bioLinks`, `bioEmails`, `website`, `location`, `followers`, `following`, `postCount`, `mediaCount`, `likesGiven`, `listedCount`, `isBlueVerified`, `verified`, `verifiedType`, `label`, `isProtected`, `profileImageUrl`, `bannerUrl`, `createdAt`. The last five counters are included when X publishes them for the account.

### Sample record

```json
{
  "type": "post",
  "id": "1519480761749016577",
  "url": "https://x.com/i/status/1519480761749016577",
  "text": "Next I'm buying Coca-Cola to put the cocaine back in",
  "createdAt": "2022-04-28T00:56:58+00:00",
  "language": "en",
  "replyCount": 4056,
  "retweetCount": 4151,
  "likeCount": 91599,
  "quoteCount": 762,
  "viewCount": 246037082,
  "isReply": false,
  "isRetweet": false,
  "isQuote": false,
  "conversationId": "1519480761749016577",
  "threadRootId": "1519480761749016577",
  "threadPosition": 1,
  "author": { "username": "elonmusk", "name": "Elon Musk", "followers": 241741247 },
  "collectedAt": "2026-10-06T00:00:00+00:00"
}
```

## Output per quote row

When quote collection is on, each post that quotes the requested post is emitted as its own row with `type: "quote"` and the same fields as a post row. `quotedTweetId` points at the requested post, so quote rows are easy to separate from replies.

## Output per retweet row

When retweeter collection is on, each reposting account is emitted as a row with `type: "retweet"`. The row carries the retweeter's `id`, `url`, `createdAt` and `author` (with the public profile snapshot), and `retweetedTweetId` points at the post you requested. This list is an approximation: X only exposes a search-index sample of recent, visible reposts, so a post with many reposts can return far fewer handles than its repost count.

## Output per profile row

When you pass a profile link or handle instead of a post, the actor emits one `profile` row:

- `id`, `username`, `name`, `bio`, `bioLinks`, `bioEmails`, `location`, `website`
- `joinedAt`, `birthDate` — when public
- `followers`, `following`, `postCount`, `mediaCount`, `listedCount`, `likesGiven` — the last three when X publishes them
- `isBlueVerified`, `verified`, `verifiedType`, `isProtected`
- `avatarUrl`, `bannerUrl`
- `url` — profile URL
- `type: "profile"`, `collectedAt`

## Output per status row

A `status` row is emitted instead of data when a link cannot be collected. It always has a `message`:

- `type: "status"`
- `status` — `noInput`, `invalidLink`, `notFound`, `rateLimited` or `error`
- `tweetUrl` and `tweetId` (post inputs) or `profileUrl` (profile inputs)
- `collectedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `tweetUrls` | array | – | **Required.** Post links, `x.com/i/web/status` links, bare post IDs, or profile links/handles. `x.com` and `twitter.com` are both accepted. |
| `includeReplies` | boolean | `true` | Also collect the replies X exposes under each post. Turn off for the post only. |
| `maxReplies` | integer | `20` | Replies to collect per post (0–500). Very large threads can return fewer. |
| `includeQuotes` | boolean | `false` | Also collect the posts that quote each requested post, as `quote` rows. |
| `maxQuotes` | integer | `50` | Quote posts to collect per post (0–500). |
| `includeRetweeters` | boolean | `false` | Also collect handles of accounts that reposted each post, as `retweet` rows. Approximate: X exposes only a search-index sample of visible reposts. |
| `maxRetweeters` | integer | `50` | Retweeter handles to collect per post (0–500). |
| `includeQuotedTweet` | boolean | `true` | Include the quoted post's text, author and metrics. |
| `includeParentTweet` | boolean | `true` | When a collected post is a reply, attach the post it answers under `parentTweet`. |
| `includeThreadAncestors` | boolean | `false` | When a requested post is a reply, walk up to 10 posts back to the thread root and attach them under `threadAncestors`, root first. |
| `includeMedia` | boolean | `true` | Include image links, video links and video variants. |
| `includeAuthorProfile` | boolean | `true` | Attach the author's public profile details to every post, reply and quote row under `author` and `authorProfile`. |

### Example: one post with its replies

```json
{
  "tweetUrls": ["https://x.com/NASA/status/2107095768821576019"],
  "includeReplies": true,
  "maxReplies": 100
}
```

### Example: quotes and retweeters of a post

```json
{
  "tweetUrls": ["https://x.com/elonmusk/status/1519480761749016577"],
  "includeReplies": false,
  "includeQuotes": true,
  "maxQuotes": 100,
  "includeRetweeters": true,
  "maxRetweeters": 100
}
```

### Example: quotes of a post, no replies

```json
{
  "tweetUrls": ["https://x.com/elonmusk/status/1519480761749016577"],
  "includeReplies": false,
  "includeQuotes": true,
  "maxQuotes": 100
}
```

### Example: exact reply cap and no media

```json
{
  "tweetUrls": ["2106665276758491194"],
  "includeReplies": true,
  "maxReplies": 500,
  "includeMedia": false,
  "includeAuthorProfile": false
}
```

### Example: mix posts, IDs and a profile

```json
{
  "tweetUrls": [
    "https://x.com/elonmusk/status/1519480761749016577",
    "2106665276758491194"
  ],
  "includeParentTweet": true
}
```

## Use cases

- **Brand and crisis monitoring** — collect a post, its replies and the posts quoting it to follow a conversation
- **Journalism and research** — archive exact text, timestamps, authors and metrics for citations
- **Social listening** — feed post, reply and quote text into sentiment or topic models
- **Customer support** — pull replies to your own posts for response triage
- **Dataset enrichment** — resolve a list of post IDs into structured records with author profiles
- **Compliance** — keep immutable evidence rows for posts you may need to reference later

## FAQ

**What data does this actor read?**  The same public post, profile and thread data any visitor sees on X without signing in. Nothing private is accessed.

**Do I need an X account, login or developer key?**  No. There is no login step and no credential input at all.

**Why do I get fewer replies than `maxReplies` or fewer than the post's reply counter?**  X only exposes part of a conversation to logged-out visitors, and some replies are deleted, protected or hidden. The actor collects everything public it can find, de-duplicated by reply ID, and never invents replies to fill the cap. Setting a larger cap and running again later can pick up more.

**Do quote rows include the post that was quoted?**  Yes. Each quote row is a normal post row, and its `quotedTweetId` points at the post you requested, so you can group quotes by target.

**Why are some profile counters missing?**  X does not publish every counter for every account on the logged-out profile view. The actor merges media, like and list counters from X's public profile widget when they are available; anything the source still omits stays absent rather than being filled with zero.

**Can I get the list of accounts that reposted or liked a post?**  Reposts: yes, approximately. With `includeRetweeters` the actor searches X for visible reposts of the post and returns their handles; X exposes only a recent search-index sample, so the list can be much shorter than the repost count. Likers: no — X does not publish who liked a post to logged-out visitors; the like count is returned instead.

**What happens with deleted, protected or hidden posts?**  You get a `status` row with `"status": "notFound"` and a message instead of data. The rest of the input is still processed.

**How fresh is the data?**  Live — every run reads X at request time. Schedule the actor for hourly or daily refreshes.

**Is this affiliated with X?**  No. This is a third-party actor that reads publicly available post and profile pages.
