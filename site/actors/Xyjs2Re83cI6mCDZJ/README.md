# Reddit Multi-Mode Scraper with Advanced Pro Filters

This actor combines Reddit's three most common scraping jobs — subreddit post listings, a post's comment thread, and a user's profile posts — into one run, with a shared set of quality filters applied before anything reaches your dataset. Point it at as many subreddits, post URLs, or usernames as you like, in any combination of modes, and get back only the records that pass your thresholds.

## What this actor does

- Runs one mode or several (`subreddit`, `comments`, `profile`) in a single execution via the `modes` array, instead of requiring one run per mode.
- Pulls subreddit post listings sorted hot, new, top, rising, or controversial, with a time window for top/controversial.
- Pulls a post's full comment thread, including nesting depth and parent-comment linkage.
- Pulls a user's recent posts from a chosen profile section, tagged with that user's karma and account-creation date.
- Applies Pro filters — minimum score, maximum age, NSFW exclusion, a keyword substring, an author substring, and a maximum comment depth — to every record from every mode before it is saved.
- Fields a given post, comment, or profile does not have (for example, a post with no link flair) are left out of the record rather than returned empty.

## Output per record

Every record carries a `recordType` field (`subreddit_post`, `comment`, `profile`, or `summary`) so mixed-mode runs stay easy to sort through.

### Subreddit post (`recordType: "subreddit_post"`)

`subreddit`, `subreddit_prefixed`, `post_id`, `post_name`, `title`, `author`, `author_fullname`, `content`, `score`, `num_comments`, `url`, `permalink`, `external_url`, `domain`, `link_flair`, `thumbnail_url`, `is_self`, `is_stickied`, `is_nsfw`, `is_spoiler`, `is_promoted`, `is_original_content`, `is_gallery`, `gilded_count`, `num_crossposts`, `crosspost_parent`, `post_hint`, `title_word_count`, `post_age_seconds`, `created_utc`, `created_at`.

### Comment (`recordType: "comment"`)

`comment_id`, `comment_name`, `author`, `author_fullname`, `text`, `body_html`, `score`, `permalink`, `post_url`, `depth`, `parent_comment_id`, `replies_count`, `subreddit`, `subreddit_prefixed`, `is_op`, `is_edited`, `edited_at`, `is_stickied`, `is_collapsed`, `is_deleted`, `is_automod`, `is_likely_bot`, `comment_age_seconds`, `distinguished`, `controversiality`, `user_flair`, `word_count`, `body_quote_count`, `body_code_count`, `body_link_count`, `awards`, `awards_count`, `gilded_count`, `created_utc`, `created_at`.

### Profile post (`recordType: "profile"`)

The same post fields as a subreddit post (`post_id`, `title`, `author`, `subreddit`, `score`, `num_comments`, `url`, `permalink`, `domain`, `created_utc`, `created_at`, and the related boolean/metadata fields above), plus `author_metadata` — an object with the profile's `username`, `total_karma`, `post_karma`, `comment_karma`, and `account_created` date.

### Run summary (`recordType: "summary"`)

One row emitted at the end of every run with `modes` (the modes that ran), `modeStats` (pushed/dropped/error counts per mode), `totalPushed`, `totalErrors`, `totalDropped`, and `scrapedAt`. If every mode produced nothing, a second summary row is also emitted, carrying `reason` and `message` explaining why.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `subreddit` | Single mode to run: `subreddit`, `comments`, or `profile`. Ignored when `modes` is set. |
| `modes` | array | – | Run several modes in one execution, e.g. `["subreddit", "profile"]`. Overrides `mode` when set. |
| `subreddits` | array | – | Subreddit names, without `r/`, for `subreddit` mode. |
| `maxPosts` | integer | `25` | Maximum posts per subreddit (`subreddit` mode) or per profile (`profile` mode), 1-1000. |
| `sort` | string | `hot` | Post sort order: `hot`, `new`, `top`, `rising`, or `controversial`. |
| `timeFilter` | string | `day` | Time window for `top`/`controversial` sorts: `hour`, `day`, `week`, `month`, `year`, or `all`. |
| `postUrls` | array | – | Reddit post URLs to fetch comments from, for `comments` mode. |
| `maxComments` | integer | `100` | Maximum comments per post, 1-10000. |
| `expandThreads` | boolean | `true` | Click "load more comments" to expand collapsed comment threads. |
| `usernames` | array | – | Reddit usernames for `profile` mode — plain, `u/name`, or full profile URLs. |
| `section` | string | `submitted` | Profile section to read: `submitted`, `comments`, or `gilded`. |
| `minScore` | integer | – | Drop any post or comment with a score below this value. |
| `maxAgeDays` | integer | – | Drop any post or comment older than this many days. |
| `excludeNsfw` | boolean | `false` | Drop posts flagged NSFW. |
| `keywordFilter` | string | – | Keep only records whose title, content, or comment text contains this text (case-insensitive). |
| `authorFilter` | string | – | Keep only records by an author whose name contains this text (case-insensitive). |
| `maxDepth` | integer | – | In `comments` mode, drop comments nested deeper than this level. |

### Example: subreddit posts, filtered

```json
{
  "mode": "subreddit",
  "subreddits": ["python", "programming"],
  "maxPosts": 50,
  "sort": "top",
  "timeFilter": "week",
  "minScore": 20,
  "excludeNsfw": true
}
```

### Example: comments on specific posts, shallow threads only

```json
{
  "mode": "comments",
  "postUrls": [
    "https://www.reddit.com/r/programming/comments/1abc123/interesting_discussion/"
  ],
  "maxComments": 200,
  "maxDepth": 2
}
```

### Example: two modes in one run

```json
{
  "modes": ["subreddit", "profile"],
  "subreddits": ["python"],
  "usernames": ["spez"],
  "maxPosts": 30,
  "keywordFilter": "release"
}
```

## Use cases

- Build a single dataset that mixes a subreddit's top posts with the comment threads on the posts you care about most.
- Track a list of users' recent activity alongside the communities they post in.
- Pre-filter noisy subreddits down to only well-scored, recent, non-NSFW posts before any downstream processing.
- Feed shallow-only comment threads (via `maxDepth`) into a quick sentiment pass without walking deep reply chains.
- Monitor mentions of a keyword or a specific author across posts and comments in one run.

## FAQ

### Do I need a Reddit account or API key?

No. All three modes read Reddit's public pages without logging in.

### What's the difference between `mode` and `modes`?

`mode` runs one mode and exists for backward compatibility. `modes` is a list and lets you run several modes — each using its own relevant inputs (`subreddits`, `postUrls`, or `usernames`) — in the same execution. When `modes` is set, it takes priority over `mode`.

### Why did one of my modes produce nothing?

A mode is skipped if its required input is empty (e.g. `comments` mode with no `postUrls`), or every one of its records was removed by your Pro filters. Check the `summary` record's `modeStats` for a per-mode pushed/dropped/error count.

### What do the Pro filters apply to?

`minScore`, `maxAgeDays`, `excludeNsfw`, `keywordFilter`, and `authorFilter` apply to posts and comments alike, across every mode in the run. `maxDepth` only affects `comments` mode.

### Why is a field like `link_flair` or `thumbnail_url` missing from some records?

Those fields only appear when Reddit's page actually renders that element for the specific post; posts without a flair or thumbnail simply omit the field.

### What does `author_metadata` on a profile record contain?

The scraped user's `username`, `total_karma`, and `account_created` date — the karma and account-age figures Reddit's profile page actually displays, not a full karma breakdown.

### Is this affiliated with Reddit?

No. This is an independent tool that reads Reddit's public pages; it is not operated or endorsed by Reddit.
