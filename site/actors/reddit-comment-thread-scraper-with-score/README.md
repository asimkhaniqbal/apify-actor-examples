# Reddit Comment Thread Scraper with Score & Keyword Filters

Export Reddit comment threads — top-level comments and nested replies, including branches hidden behind "load more replies" and "continue this thread" — as flat, structured rows. Paste post links (or comment links), choose a sort, and get author, text, score, timestamps, reply depth, parent relationships, and moderation/deletion flags. No Reddit account, API key, or login required.

## What this actor does

- **Full thread reconstruction** — follows Reddit's own nested reply loaders and continue-thread pages until `maxComments` is reached or Reddit stops serving comments to logged-out readers
- **6 comment sorts** — Best, Top, New, Controversial, Old, and Q&A, matching Reddit's UI
- **Comment permalink focus** — paste a comment permalink with `focusOnTargetComment` enabled to get that comment plus its ancestor chain, including deeply nested targets
- **Rich client-side filtering** — date window, reply-depth range, score range, author allow/deny, keyword include/exclude, length bounds, award count, OP-only, deleted/removed, collapsed
- **Optional parent post record** — title, author, score, ratio, timestamp, subreddit, and text/link metadata when `includePost` is on
- **Flexible URL input** — post URLs, comment permalinks, share links, `redd.it` short links, bare post IDs
- **Omit-empty output** — a field appears only when Reddit actually exposes a value for it

## Output per comment

- `dataType` — `"comment"`
- `comment_id`, `comment_name` — bare id and Reddit fullname (`t1_...`)
- `post_id`, `post_url`, `post_title`, `link_id` — parent thread context
- `permalink` — direct comment link
- `author` — username (`[deleted]` when the account is gone)
- `avatarUrl` — author avatar image, when available
- `body` — comment text as readable plain text (block breaks preserved)
- `body_html` — comment HTML from Reddit's renderer
- `score` — net score (upvotes minus downvotes) at crawl time
- `created_utc`, `created_at` — posted time (epoch and ISO-8601 UTC)
- `edited`, `edited_at` — whether and when the comment was edited
- `depth` — nesting level, `0` = top-level reply to the post
- `position` — the comment's position among its siblings
- `parentPositions` — the position path from the thread root. The prefix can be incomplete for branches reached through a "continue this thread" page; use `parent_id` for exact parentage
- `parent_id`, `parent_kind` — direct parent fullname (`t3_...` for top-level or `t1_...` for a reply) and kind
- `is_op` — author is the post's original poster
- `collapsed` — Reddit collapses the comment by default
- `isCommentDeleted`, `isAuthorDeleted` — comment content / author account was deleted
- `contentType` — `text`, `image`, or `giphy`
- `isHighlighted`, `hasPaidAward` — Reddit highlight/paid-award markers when present
- `total_awards_received` — total award count (0 when none)
- `subreddit`, `subreddit_prefixed`, `subreddit_id` — community context
- `crawled_at` — when this record was fetched (ISO-8601)
- `source` — `"html"`

A rare deleted-comment placeholder that Reddit no longer gives an id for has no `comment_id`; its `isCommentDeleted`/`isAuthorDeleted` flags and position path still identify it. Its `parent_id` falls back to the post, so use the position path for its place in the tree.

Fields Reddit does not expose to logged-out readers are omitted: comment `author_id`, author flair fields, `ups`/`downs`, `score_hidden`, `controversiality`, `distinguished`, comment-level `stickied`/`locked`/`archived`, `collapsed_reason*`, legacy `gilded`, the award breakdown, and `removed_by_category`. See **Limitations** below.

## Output per post

Emitted once per post as the first record when `includePost` is on. Comment filters do not apply to it.

- `dataType` — `"post"`
- `post_id`, `post_name`, `title`, `author`, `author_id`
- `subreddit`, `subreddit_prefixed`, `subreddit_id`
- `url`, `permalink` — canonical reddit.com link
- `content_url` — external destination for link posts
- `domain`, `post_type`, `is_self`
- `content`, `content_html` — self-post body
- `score`, `upvote_ratio`, `num_comments`
- `created_utc`, `created_at`
- `language`
- `is_nsfw`, `is_stickied`, `is_locked`
- `total_awards_received`
- `crawled_at`, `source`

Post fields that require Reddit's legacy API data (flair, media/gallery URLs, poll data, crosspost counts, removal category, ups/downs, edited timestamp) are not available to logged-out readers and are omitted.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `postUrls` | array | – (required) | Post URLs, comment URLs, share links, `redd.it` short links, or bare post IDs |
| `maxComments` | integer | `100` | Maximum comments to keep per post, after filters (1–100,000) |
| `commentSort` | select | `confidence` | `confidence` (Best), `top`, `new`, `controversial`, `old`, `qa` |
| `includePost` | boolean | `false` | Emit the parent post as the first record of each thread |
| `postedAfter` | date | – | Keep comments created on/after this date (UTC) |
| `postedBefore` | date | – | Keep comments created on/before this date (UTC; a bare date means end of day) |
| `minDepth` | integer | – | Keep comments at or deeper than this nesting level (0–100) |
| `maxDepth` | integer | – | Keep comments at or above this nesting level (0–100) |
| `minCommentScore` | integer | – | Keep comments with score ≥ this value |
| `maxCommentScore` | integer | – | Keep comments with score ≤ this value |
| `onlyOP` | boolean | `false` | Keep only the original poster's comments |
| `excludeDeletedRemoved` | boolean | `false` | Drop comments Reddit marks deleted or author-deleted |
| `excludeCollapsed` | boolean | `false` | Drop comments Reddit collapses by default |
| `authors` | array | – | Keep only these usernames (case-insensitive) |
| `excludeAuthors` | array | – | Drop these usernames (case-insensitive), e.g. bots |
| `keywords` | array | – | Keep comments containing at least one keyword/phrase |
| `excludeKeywords` | array | – | Drop comments containing any keyword/phrase |
| `minAwards` | integer | – | Keep comments with at least this many total awards |
| `minCommentLength` | integer | – | Minimum body length in characters (0–10,000) |
| `maxCommentLength` | integer | – | Maximum body length in characters (0–10,000) |
| `focusOnTargetComment` | boolean | `false` | For a comment permalink, return the target plus its ancestor chain. Works for deeply nested targets; the target must still be publicly visible |
| `commentContext` | integer | `3` | Ancestor levels above the target comment when Reddit returns the focus view (0–8) |

**Accepted `postUrls` forms:**
- Full post URL: `https://www.reddit.com/r/programming/comments/1abc123/some_title/`
- Short URL: `https://www.reddit.com/comments/1abc123` (also with `?sort=top`)
- Comment URL: `https://www.reddit.com/r/x/comments/1abc123/title/comment/xyz987/`
- Share link: `https://www.reddit.com/r/x/s/<token>`
- Short link: `https://redd.it/1abc123`
- Bare post ID: `1abc123` or `t3_1abc123`
- `old.reddit.com` URLs also work

Malformed items (empty strings, garbage URLs, non-strings) are skipped without failing the run.

**Removed inputs:** `distinguishedFilter`, `controversialOnly`, and `excludeStickied` were removed because Reddit no longer exposes those markers to logged-out readers. Old saved tasks that still send them keep running; the actor logs one warning and ignores them.

### Example: scrape a whole thread sorted by Top

```json
{
  "postUrls": ["https://www.reddit.com/r/technology/comments/1abc123/some_title/"],
  "maxComments": 500,
  "commentSort": "top",
  "includePost": true
}
```

### Example: recent, high-signal comments only

```json
{
  "postUrls": ["https://www.reddit.com/r/investing/comments/1abc123/some_title/"],
  "maxComments": 1000,
  "postedAfter": "2026-06-01",
  "minCommentScore": 5,
  "excludeDeletedRemoved": true
}
```

### Example: NLP dataset — clean comment bodies

```json
{
  "postUrls": [
    "https://www.reddit.com/r/AskReddit/comments/1abc123/some_title/",
    "https://www.reddit.com/r/AskReddit/comments/1def456/other_title/"
  ],
  "maxComments": 5000,
  "excludeDeletedRemoved": true,
  "excludeAuthors": ["AutoModerator"],
  "minCommentLength": 20,
  "excludeKeywords": ["I am a bot"]
}
```

### Example: jump straight to one comment

```json
{
  "postUrls": ["https://www.reddit.com/r/x/comments/1abc123/title/comment/xyz987/"],
  "focusOnTargetComment": true,
  "commentContext": 5
}
```

## Use cases

- **Sentiment & NLP datasets** — build large, structured corpora from real discussions
- **Market & product research** — read what customers say in the replies, not just the top comment
- **Community & moderation analysis** — study conversation depth, controversy, and OP engagement
- **Launch monitoring** — track reaction to announcements across subreddits on a schedule
- **Academic & social research** — export discussion trees for network and discourse analysis
- **Support mining** — surface recurring complaints buried deep in a thread

## FAQ

**Do I need a Reddit account, API key, or login?**
No. The actor reads Reddit's public pages anonymously and stores no credentials.

**Why are some fields missing compared with older versions?**
Reddit stopped serving its anonymous JSON endpoints in 2026 and does not render author IDs, vote splits, flair, moderator badges, or award details to logged-out readers. The actor emits every field the public pages still expose and omits the rest instead of inventing values.

**Does it really capture the whole thread?**
Up to `maxComments`, yes. The actor follows Reddit's nested reply loaders and continue-thread pages until the thread is exhausted or your limit is reached. Reddit's displayed comment count includes deleted and removed comments, so a complete export can be smaller than that number.

**What is the difference between `parent_id` and `link_id`?**
`link_id` always points to the parent post (`t3_...`). `parent_id` points to the comment's direct parent — the post for a top-level comment or another comment for a reply. `parent_kind` tells you which, and `parentPositions` gives the position path (with the continue-page caveat above).

**How do I scrape just one comment instead of the whole thread?**
Paste its permalink into `postUrls` and enable `focusOnTargetComment`. You get that comment plus up to `commentContext` levels of parent comments. The target must still be publicly visible; deleted/removed targets return no records.

**Why does a deleted comment still appear?**
Deleted comments are part of the thread structure. They carry `isCommentDeleted` / `isAuthorDeleted` and usually no body; enable `excludeDeletedRemoved` to drop them.

**What happens for a private subreddit, banned post, or deleted post?**
The actor returns 0 records for that URL and continues with the other URLs. A status message explains when nothing could be read.

**Do I need to configure a proxy?**
No. The actor selects the network route it needs automatically.

**Is this affiliated with Reddit?**
No. This is an independent third-party actor that reads public web pages; it is not endorsed by or affiliated with Reddit, Inc.

## Cost

Measured all-in run cost is roughly **$0.72–1.58 per 1,000 comments**, depending on thread depth, how many continue-thread branches are loaded, and how many deleted comments Reddit still counts. Small or heavily filtered runs have a higher fixed cost per 1,000. Store pricing is set separately.

## Limitations

- Reddit's own displayed comment count includes deleted, removed, and filtered comments that are not served to logged-out readers. The actor reports what is actually served.
- Comment author IDs, vote up/down splits, flair, moderator/admin badges, controversy flags, award names/icons, and removal categories are not exposed anonymously. They are omitted rather than guessed. (The post record still carries the post author's ID when Reddit renders it.)
- Some comments arrive from Reddit's lazy-loading fragments with `body_html` but without a plain-text `body` (and occasionally without either) even though they are not deleted. Use `body_html` as a fallback; the omission rate measured in cloud runs was ~15–18%.
- `parentPositions` can lose the continue-target's position for branches loaded through "continue this thread" pages, so paths for those descendants may not match the tree exactly. `parent_id` remains correct.
- Deleted-comment placeholders that Reddit renders without an id may carry the post as `parent_id` even when nested; their position path is the only tree hint for those rows.
- Very large threads take longer because every hidden branch is loaded through Reddit's own pagination; there is no branch pruning, so cost scales with the served tree as well as `maxComments`.
- The parent post record is intentionally thin: media, gallery, poll, crosspost, and flair details are not available on the public post page.
