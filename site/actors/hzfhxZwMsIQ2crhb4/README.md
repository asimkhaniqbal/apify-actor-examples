# Reddit Comment Thread Scraper with Score & Keyword Filters

Scrape **Reddit comment threads in depth** — top-level comments and nested replies, including the ones Reddit hides behind "load more" links, up to the comment limit you set. Paste one or more post URLs (or a specific comment permalink) and get back structured comment data with author, votes, timestamps, moderation flags, and thread position, optionally alongside the full parent post record. No Reddit API key or login required.

## What this actor does

- **Deep thread resolution** — walks top-level comments and nested replies, including replies hidden behind Reddit's "load more" placeholders, until `maxComments` (default 100) is reached or the thread runs out
- **6 comment sort modes** — Best, Top, New, Controversial, Old, Q&A, matching Reddit's own UI
- **Rich filtering** — date windows, reply-depth range, score thresholds, author allow/deny lists, keyword include/exclude, comment length bounds, controversial-only, and more; filters apply to comments only, and `maxComments` counts the comments that pass them
- **Optional parent post record** — full post metadata (title, score, media, embeds) emitted first when enabled
- **Flexible URL input** — accepts post URLs, comment permalinks, share links, `redd.it` short links, or bare post IDs
- **Empty fields are omitted** — a field only appears on a record when Reddit actually returned a value for it

## Output per comment

- `dataType` — always `"comment"` for these records
- `comment_id` — comment identifier without the `t1_` prefix
- `comment_name` — full comment name in Reddit's format (`t1_...`)
- `post_id`, `post_url`, `post_title` — identifiers and title of the parent post this comment belongs to
- `link_id` — fullname of the parent post (`t3_...`)
- `permalink` — direct link to the comment on reddit.com
- `author` — commenter's username (`[deleted]` if the account was deleted)
- `author_id` — commenter's fullname (`t2_...`); absent when the author is deleted
- `author_is_blocked` — whether the author has blocked the scraper's account
- `author_flair`, `author_flair_css_class`, `author_flair_template_id`, `author_flair_type`, `author_flair_text_color`, `author_flair_background_color`, `author_flair_richtext` — the author's community flair, when they have one
- `author_premium`, `author_patreon_flair` — whether the author has Reddit Premium / a Patreon badge in this community
- `body` — comment text as markdown
- `body_html` — comment text rendered as Reddit HTML
- `score` — net score (upvotes minus downvotes)
- `ups`, `downs` — raw upvote / downvote counts
- `score_hidden` — whether the score is currently hidden by the community
- `controversiality` — `1` when the up/down split is near-even, else `0`
- `subreddit`, `subreddit_prefixed`, `subreddit_id`, `subreddit_type` — the community the comment belongs to
- `created_utc`, `created_at` — when the comment was posted (Unix epoch and ISO-8601)
- `edited`, `edited_at` — whether and when the comment was last edited
- `depth` — nesting level, `0` = top-level comment
- `parent_id` — fullname of the direct parent (`t3_...` for a post, `t1_...` for another comment)
- `parent_kind` — `post` if it's a top-level reply to the post, `comment` if it's a reply to another comment
- `is_op` — whether the commenter is also the post's original author
- `is_stickied` — whether the comment is pinned by moderators
- `is_locked` — whether the comment thread is locked
- `archived` — whether the comment is archived (voting/commenting closed)
- `distinguished` — `moderator` or `admin` when the author posted in an official capacity
- `collapsed` — whether Reddit collapses the comment by default
- `collapsed_reason`, `collapsed_reason_code` — human-readable and machine-readable reason for the collapse
- `collapsed_because_crowd_control` — whether the collapse was caused by crowd control
- `can_gild` — whether the comment could receive legacy awards
- `comment_type` — optional type marker (e.g. `live`); absent on regular comments
- `unrepliable_reason` — why the comment can't be replied to, when that's the case
- `no_follow` — whether the comment is tagged nofollow for search engines
- `send_replies` — whether the author receives reply notifications for this comment
- `gilded` — number of legacy (gold-style) awards received
- `total_awards_received` — total award count
- `awards` — award breakdown (name, count, coin price, icon URL); omitted when there are no awards
- `removed_by_category` — who removed the comment, when removed: `moderator`, `automod_filtered`, `reddit`, `author`/`deleted`
- `crawled_at` — when the actor fetched this record (ISO-8601)
- `source` — always `json`

Deleted/removed comments keep their `[deleted]`/`[removed]` markers in `body` so removals stay auditable.

## Output per post

Emitted once per post (duplicate URLs for the same post are merged), as the first record of that thread, when `includePost` is enabled. The comment filters don't apply to it.

- `dataType` — always `"post"` for these records
- `post_id`, `post_name` — bare id and full Reddit name (`t3_...`) of the post
- `title` — post title
- `content`, `content_html` — self-post body as markdown / Reddit HTML (text posts only)
- `content_url` — external destination URL for link posts
- `url`, `permalink` — canonical reddit.com link to the post
- `url_overridden_by_dest` — Reddit's resolved destination URL, when it differs from `content_url`
- `domain` — `self.<subreddit>` for text posts, otherwise the linked domain
- `post_hint` — Reddit's own content hint (`self`, `link`, `image`, `hosted:video`, …)
- `is_self` — whether the post is a text post
- `post_type` — `self`, `link`, `image`, `video`, `gallery`, or `poll`
- `media_type` — `text`, `link`, `image`, `video`, `gallery`, or `embed`
- `has_media` — whether the post carries image/video/gallery/poll/embed media
- `author`, `author_id` — the poster's username and fullname
- `author_flair`, `author_flair_css_class`, `author_flair_template_id`, `author_flair_type`, `author_flair_text_color`, `author_flair_background_color`, `author_flair_richtext` — the poster's community flair, when they have one
- `author_premium`, `author_is_blocked`, `author_patreon_flair` — poster's premium status, block status, and Patreon badge
- `subreddit`, `subreddit_prefixed`, `subreddit_id`, `subreddit_subscribers`, `subreddit_type` — the community the post belongs to
- `score`, `ups`, `downs`, `upvote_ratio` — vote totals and the up/down ratio
- `num_comments` — total comment count on the post at crawl time
- `num_crossposts`, `num_duplicates` — how often the post was crossposted, and how many other posts link the same destination (link posts only)
- `link_flair`, `link_flair_css_class`, `link_flair_richtext`, `link_flair_background_color`, `link_flair_text_color`, `link_flair_type` — the post's own flair, when set
- `thumbnail_url`, `thumbnail_width`, `thumbnail_height` — thumbnail image and dimensions, when a real thumbnail exists
- `created_utc`, `created_at` — when the post was submitted (Unix epoch and ISO-8601)
- `edited`, `edited_at` — whether and when the post was last edited
- `is_stickied`, `is_pinned` — whether the post is pinned by moderators / on the subreddit
- `is_locked`, `is_archived` — whether commenting/voting is closed
- `is_nsfw`, `is_spoiler` — content warnings
- `is_original_content` — whether the post is flagged OC
- `is_crosspost`, `crosspost_parent_id` — whether this is a crosspost, and the original post's id
- `is_crosspostable`, `is_meta`, `is_robot_indexable`, `is_reddit_media_domain`, `media_only`, `quarantine` — additional Reddit-assigned post attributes
- `no_follow`, `send_replies`, `allow_live_comments`, `contest_mode`, `hide_score`, `suggested_sort`, `content_categories` — post configuration flags Reddit reports
- `hidden` — whether the post is hidden
- `images` — image URLs from the post preview
- `gallery_images`, `gallery_count` — full-resolution gallery image URLs and item count (gallery posts)
- `video_url` — direct video file URL (video posts)
- `poll_data` — poll options and vote totals (poll posts)
- `embed_type`, `embed_provider`, `embed_title`, `embed_author_name`, `embed_author_url`, `embed_thumbnail_url`, `embed_html` — auto-embed data for link posts Reddit embeds from YouTube, Imgur, Gfycat, Twitch, etc.; omitted when the post has no external embed
- `gilded`, `total_awards_received`, `awards` — legacy and modern award counts, plus the award breakdown
- `removed_by_category` — who removed the post, when removed
- `distinguished` — `moderator` or `admin` when the poster acted in an official capacity
- `crawled_at` — when the actor fetched this record (ISO-8601)
- `source` — always `json`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `postUrls` | array | – (required) | Post URLs, comment URLs, share links, `redd.it` short links, or bare post IDs — see accepted forms below |
| `maxComments` | integer | `100` | Maximum comments to scrape per post, including nested replies (1–100,000) |
| `commentSort` | select | `confidence` | `confidence` (Best), `top`, `new`, `controversial`, `old`, `qa` |
| `includePost` | boolean | `false` | Emit the parent post as a separate first record of each thread |
| `postedAfter` | date | – | Only keep comments created on/after this date (UTC) |
| `postedBefore` | date | – | Only keep comments created on/before this date (UTC; a bare date means end of that day) |
| `minDepth` | integer | – | Only keep comments at or deeper than this nesting level (0–100) |
| `maxDepth` | integer | – | Only keep comments at or above this nesting level (0–100) |
| `minCommentScore` | integer | – | Only keep comments with score ≥ this value |
| `maxCommentScore` | integer | – | Only keep comments with score ≤ this value |
| `onlyOP` | boolean | `false` | Keep only the original poster's own comments |
| `excludeDeletedRemoved` | boolean | `false` | Drop comments whose body/author shows as `[deleted]`/`[removed]` |
| `excludeStickied` | boolean | `false` | Drop moderator-pinned comments (e.g. AutoModerator notices) |
| `excludeCollapsed` | boolean | `false` | Drop comments Reddit collapses by default |
| `distinguishedFilter` | select | `any` | `any`, `moderator` (mod comments only), `admin` (admin comments only), `none` (regular users only) |
| `authors` | array | – | Only keep comments from these usernames (case-insensitive) |
| `excludeAuthors` | array | – | Drop comments from these usernames (case-insensitive), e.g. bots |
| `keywords` | array | – | Only keep comments containing at least one of these keywords/phrases |
| `excludeKeywords` | array | – | Drop comments containing any of these keywords/phrases |
| `minAwards` | integer | – | Only keep comments with at least this many total awards |
| `minCommentLength` | integer | – | Only keep comments whose body is at least this many characters (0–10,000) |
| `maxCommentLength` | integer | – | Only keep comments whose body is at most this many characters (0–10,000) |
| `controversialOnly` | boolean | `false` | Keep only comments Reddit flags as controversial (near-even up/downvote split) |
| `focusOnTargetComment` | boolean | `false` | When a `postUrls` item is a specific comment permalink, fetch only that comment plus its ancestor chain instead of the whole thread |
| `commentContext` | integer | `3` | Ancestor levels to include above the target comment (0–8; used only with `focusOnTargetComment`) |

**Accepted `postUrls` forms:**
- Full post URL: `https://www.reddit.com/r/programming/comments/1abc123/some_title/`
- Short URL: `https://www.reddit.com/comments/1abc123` (also with `?sort=top`)
- Comment URL: `https://www.reddit.com/r/x/comments/1abc123/title/comment/xyz987/`
- Share link: `https://www.reddit.com/r/x/s/<token>`
- Short link: `https://redd.it/1abc123`
- Bare post ID: `1abc123` or `t3_1abc123`
- `old.reddit.com` URLs also work

Malformed items (empty strings, garbage URLs, non-strings) are skipped without failing the run.

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
  "excludeDeletedRemoved": true,
  "excludeStickied": true
}
```

### Example: NLP dataset — clean, filtered comment bodies

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
  "excludeKeywords": ["I am a bot"],
  "includePost": false
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

- **Sentiment & NLP datasets** — build large, labeled corpora from real Reddit discussions
- **Market & product research** — see what customers actually say about a product or brand in the replies
- **Community & moderation analysis** — study conversation structure, controversy, and OP engagement in a subreddit
- **Launch monitoring** — track reaction to announcements across subreddits with scheduled runs
- **Academic & social research** — export full, structured discussion threads for analysis
- **Customer support mining** — surface recurring complaints or questions buried in comment threads

## FAQ

**Do I need a Reddit account, API key, or login?**
No. The actor reads Reddit's public web data; no authentication is required.

**Does it really capture the whole thread, including deeply nested replies, and why might the count differ from `num_comments`?**
Up to `maxComments`, yes — the default of 100 only covers part of a busy thread, so raise it (up to 100,000) for a whole conversation. Reddit hides deeper replies behind "load more" placeholders, and the actor keeps expanding them, top-level comments first, until the limit is hit or the thread runs out. A placeholder that keeps failing after several retries is skipped, and the post's own `num_comments` also counts comments that have since been deleted, so the total can land a little below that counter.

**What's the difference between `parent_id` and `link_id`?**
`link_id` always points to the parent post (`t3_...`). `parent_id` points to the comment's direct parent, which can be the post (`t3_...`, for a top-level comment) or another comment (`t1_...`, for a reply) — check `parent_kind` to tell which.

**How do I scrape just one comment instead of the whole thread?**
Paste that comment's permalink URL (the one containing `/comment/{id}/`) into `postUrls` and enable `focusOnTargetComment`. You'll get that comment plus up to `commentContext` levels of its parent comments, the same view Reddit shows when you click a comment's "permalink".

**What happens for a private subreddit, banned post, or deleted post?**
The actor returns 0 records for that URL along with a status message, instead of failing the whole run — other URLs in the same run are unaffected.

**Is this affiliated with Reddit?**
No. This is an independent, third-party actor that reads Reddit's public web pages; it is not endorsed by or affiliated with Reddit, Inc.
