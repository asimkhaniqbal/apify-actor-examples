# Reddit Research Scraper — Posts, Comments, Search, Users & Communities

One run, five ways to read Reddit: subreddit feeds, complete comment trees, keyword search, user profiles, and community details. Apply the filters you need before anything reaches your dataset, so you can go from a broad topic to a clean, analysis-ready table without stitching several tools together.

## What this actor does

- **Subreddit posts** — collect a community's hot, new, top, rising, controversial, or best posts, with native Reddit pagination and optional post body/media details.
- **Comment trees** — pull a post's comments with nesting depth, parent-comment linkage, scores, awards, edited/deleted flags, and optional expansion of collapsed branches.
- **Keyword search** — search Reddit posts, comments, or both, globally or inside one community, and sort by relevance, hot, top, new, or comment count.
- **User profiles** — read a user's profile summary (karma breakdown, cake day, account age, avatar, moderated communities, trophies) plus their posts and/or comments.
- **Community details** — get a subreddit's description, subscriber and activity counts, creation date, type, icon/banner, and optionally its public rules.
- **One shared filter layer** — score, age, comment count, upvote ratio, content type, flair, author, domain, keyword, NSFW, and comment depth filters run before records are saved.
- **Mixed-mode runs** — combine any modes in one execution; every row carries a `recordType` so you can split the output at a glance.

## Modes

| Mode | What it reads | Best for |
|---|---|---|
| `posts` | A subreddit's post listing | Community monitoring, trend discovery |
| `comments` | Comment trees from post URLs or IDs | Sentiment, moderation, discussion analysis |
| `search` | Reddit keyword search results | Topic research across Reddit |
| `users` | A user's profile and activity | Author research, influencer vetting |
| `subreddits` | Community about pages | Community discovery and comparison |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `posts` | One mode: `posts`, `comments`, `search`, `users`, or `subreddits`. |
| `modes` | array | `["posts"]` | Run several modes in one execution; overrides `mode`. |
| `subreddits` | array | `["python"]` | Communities for posts, comments, or subreddit details. Accepts names, `r/` prefixes, or URLs. |
| `searchQueries` | array | `["python"]` | Keywords for search mode. |
| `searchType` | string | `posts` | `posts`, `comments`, or `both`. |
| `searchSort` | string | `relevance` | `relevance`, `hot`, `top`, `new`, or `comments`. |
| `withinCommunity` | string | — | Restrict search to one subreddit. |
| `maxSearchResults` | integer | `50` | Maximum search results per query. |
| `enrichSearchResults` | boolean | `true` | Open the first few result posts to fill score, comment count, body, and media. |
| `postUrls` | array | — | Post URLs or IDs for comments mode. |
| `usernames` | array | — | Users for users mode. Accepts names, `u/` prefixes, or URLs. |
| `section` | string | `submitted` | Profile section: `overview`, `submitted`, or `comments`. |
| `sort` | string | `hot` | `hot`, `new`, `top`, `rising`, `controversial`, or `best`. |
| `timeFilter` | string | `day` | `hour`, `day`, `week`, `month`, `year`, or `all`. |
| `commentSort` | string | `confidence` | `confidence`, `top`, `new`, `controversial`, `old`, or `qa`. |
| `maxPosts` | integer | `25` | Posts per community or user. |
| `maxComments` | integer | `100` | Comments per post. |
| `maxItems` | integer | `1000` | Global cap across the whole run. |
| `expandThreads` | boolean | `true` | Load collapsed comment branches and additional profile pages. |
| `includePost` | boolean | `false` | Include the parent post record in comments mode. |
| `includeSubredditInfo` | boolean | `false` | Emit a community record alongside posts. |
| `includeRules` | boolean | `false` | Add public rules to subreddit records. |
| `minScore` | integer | — | Minimum post/comment score. |
| `maxScore` | integer | — | Maximum post/comment score. |
| `minComments` | integer | — | Minimum comment count on a post. |
| `maxPostComments` | integer | — | Maximum comment count on a post. |
| `minUpvoteRatio` | number | — | Minimum post upvote ratio (0–1). |
| `maxAgeDays` | integer | — | Drop records older than this many days. |
| `postedAfter` | date | — | Keep posts created on or after this date. |
| `postedBefore` | date | — | Keep posts created on or before this date. |
| `postType` | string | `all` | `all`, `self`, `link`, `image`, `video`, `gallery`, `poll`, `crosspost`, `gif`, or `multi_media`. |
| `excludeStickied` | boolean | `false` | Drop pinned/stickied posts. |
| `excludeNsfw` | boolean | `false` | Drop posts flagged NSFW. |
| `keywordFilter` | string | — | Keep records containing this text (title, body, or comment text). |
| `authorFilter` | string | — | Keep records whose author name contains this text. |
| `authorAllowlist` | array | — | Keep only these exact authors. |
| `authorBlocklist` | array | — | Exclude these exact authors. |
| `flairFilter` | string | — | Keep posts whose link flair contains this text. |
| `domainFilter` | string | — | Keep posts whose link domain contains this text. |
| `maxDepth` | integer | — | Maximum comment nesting depth. |

### Example: top posts from two communities

```json
{
  "mode": "posts",
  "subreddits": ["python", "programming"],
  "sort": "top",
  "timeFilter": "week",
  "maxPosts": 50,
  "minScore": 25,
  "excludeNsfw": true
}
```

### Example: a post's comment tree, shallow only

```json
{
  "mode": "comments",
  "postUrls": ["https://www.reddit.com/r/programming/comments/1abc123/interesting_discussion/"],
  "maxComments": 200,
  "maxDepth": 2,
  "commentSort": "top"
}
```

### Example: search posts and comments, then profile a user

```json
{
  "modes": ["search", "users"],
  "searchQueries": ["vector database"],
  "searchType": "both",
  "maxSearchResults": 30,
  "usernames": ["spez"],
  "section": "overview",
  "maxPosts": 20
}
```

### Example: community details with rules

```json
{
  "mode": "subreddits",
  "subreddits": ["python", "MachineLearning"],
  "includeRules": true
}
```

## Output

Every row includes `recordType` (`subreddit_post`, `comment`, `profile`, `user_profile`, `subreddit`, or `summary`).

### Post records

`post_id`, `post_name`, `title`, `author`, `author_id`, `author_fullname`, `author_flair`, `subreddit`, `subreddit_prefixed`, `subreddit_id`, `subreddit_subscribers`, `subreddit_type`, `content`, `content_html`, `content_url`, `url`, `permalink`, `external_url`, `url_overridden_by_dest`, `domain`, `post_hint`, `post_type`, `media_type`, `score`, `ups`, `upvote_ratio`, `num_comments`, `link_flair`, `thumbnail_url`, `created_utc`, `created_at`, `edited`, `edited_at`, `is_self`, `is_stickied`, `is_locked`, `is_archived`, `is_pinned`, `is_nsfw`, `is_spoiler`, `is_original_content`, `is_crosspost`, `is_gallery`, `total_awards_received`, `awards`, `gilded_count`, `num_crossposts`, `crosspost_parent`, `language`, `has_media`, `images`, `gallery_images`, `gallery_count`, `video_url`, `poll_data`, `title_word_count`, `post_age_seconds`, `search_term`, `crawled_at`, `source`.

### Comment records

`comment_id`, `comment_name`, `post_id`, `post_title`, `post_url`, `author`, `author_id`, `author_fullname`, `text`, `body_html`, `score`, `permalink`, `parent_id`, `parent_comment_id`, `parent_kind`, `parent_positions`, `depth`, `position`, `replies_count`, `subreddit`, `subreddit_prefixed`, `subreddit_id`, `is_op`, `is_edited`, `edited_at`, `is_stickied`, `is_collapsed`, `is_deleted`, `is_automod`, `is_likely_bot`, `comment_age_seconds`, `distinguished`, `controversiality`, `user_flair`, `word_count`, `body_quote_count`, `body_code_count`, `body_link_count`, `awards_count`, `total_awards_received`, `gilded_count`, `content_type`, `avatar`, `created_utc`, `created_at`, `search_term`, `crawled_at`, `source`.

### Profile post records

Post fields above, plus `author_metadata` with the profile summary.

### User profile records

`username`, `user_id`, `profile_url`, `icon_img`, `snoovatar_img`, `description`, `total_karma`, `post_karma`, `comment_karma`, `awardee_karma`, `awarder_karma`, `created_utc`, `created_at`, `account_created`, `account_age_years`, `is_mod`, `is_gold`, `has_verified_email`, `verified`, `moderated_subreddits`, `trophies`.

### Subreddit records

`subreddit`, `subreddit_prefixed`, `subreddit_id`, `url`, `title`, `description`, `public_description`, `subscribers`, `active_user_count`, `weekly_active_users`, `weekly_contributions`, `created_utc`, `created_at`, `subreddit_type`, `over18`, `quarantine`, `community_icon`, `icon_img`, `banner_img`, `primary_color`, `key_color`, `lang`, `rules`.

### Summary record

One row at the end of every run with `modes`, `modeStats` (pushed/dropped/errors per mode), `totalPushed`, `totalErrors`, `totalDropped`, and `scrapedAt`. If nothing is emitted, a second summary row explains why.

## Use cases

- Track a topic across several subreddits and keep only high-scoring, recent posts.
- Pull shallow comment trees for a quick sentiment pass without walking every reply.
- Search Reddit for a product or phrase, then compare how different communities discuss it.
- Vetting an author: see their profile summary, recent posts, and comment history together.
- Compare community size, activity, and rules before launching a campaign or research project.

## Limitations

- Reddit may challenge individual post/comment pages on some cloud networks even when feed, search, profile, and community pages load. When that happens, comments mode emits no rows and records a mode error (visible in the summary and run status) rather than partial or fabricated data; retry from another network or use a saved copy of the thread.
- Reddit caps listings and search at a source-side maximum; very large `maxItems` requests may stop before that number when Reddit stops paginating. Comment search returns Reddit's first server-rendered result page per query (typically 8-10 rows); post search paginates further.
- Private, quarantined, banned, and login-only communities cannot be read anonymously.
- Search results from older archive data may not include the newest posts; live Reddit search is used first when available.

## FAQ

### Do I need a Reddit account, API key, or login?

No. The actor reads public Reddit pages and public search feeds. No credentials are required.

### How many records can I get?

Set `maxItems` for a global cap, `maxPosts` per community/user, `maxComments` per post, and `maxSearchResults` per query. Reddit itself limits how far a listing or search can paginate; very large requests may stop at the source's own ceiling.

### Why is a field missing from some records?

Fields that Reddit does not expose for a particular record are omitted rather than filled with empty values. For example, a text post has no external link domain, and a search-result comment has no post score.

### Do the filters apply to every mode?

`minScore`, `maxScore`, `maxAgeDays`, `keywordFilter`, `authorFilter`, `authorAllowlist`, `authorBlocklist`, `excludeNsfw`, and the post-level filters apply wherever the corresponding field exists. `maxDepth` applies to comments. `postedAfter` and `postedBefore` apply to posts.

### Can I search inside one subreddit?

Yes. Set `searchQueries` and put the community name in `withinCommunity`.

### What is the difference between `mode` and `modes`?

`mode` runs one mode. `modes` is a list and can run several modes in one execution. When `modes` is set, it takes priority.

### Why did a mode produce nothing?

Its required input was empty (for example, comments mode without `postUrls`), the community/post is private or unavailable, or your filters removed every row. Check the `summary` record's `modeStats` for counts.

### Is this affiliated with Reddit?

No. This is an independent tool that reads Reddit's public pages; it is not operated or endorsed by Reddit.
