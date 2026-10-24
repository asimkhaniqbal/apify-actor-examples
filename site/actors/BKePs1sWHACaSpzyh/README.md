# Reddit Subreddit Post & Comment Harvester

Pull posts from any subreddit without an API key or login — hot, new, top, rising, controversial, or best — with 100+ fields per post covering score, upvote ratio, author, flairs, media, awards, and moderation flags. Add comment threads with nested replies when you need the discussion, not just the post. Filter by date range, post type, score/comment thresholds, keywords, authors, and more.

## What this actor does

- **Scrape any subreddit** by bare name (`python`), `r/`-prefixed name (`r/python`) or full URL — scrape multiple subreddits in a single run
- **100+ fields per post**: score, upvotes, upvote ratio, comment count, author + author flair, community + subscriber count, media (images, galleries, video, polls, external embeds), flairs, awards, moderation flags and timestamps
- **Optional comment threads** with nested replies, depth, parent links and OP flags, in any of Reddit's 6 comment sort orders (turn on `includeComments` and set `maxCommentsPerPost` above 0)
- **Rich filtering**: 6 sort orders, time windows, date ranges, post type, score/comment-count/upvote-ratio thresholds, keyword and author allow/deny lists, and more
- **Empty fields are omitted** from the output — no need to defend against `null`/empty values in your downstream pipeline

## Output per post

**Identity & context** — `dataType` (always `post`), `subreddit`, `subreddit_prefixed`, `subreddit_id`, `subreddit_subscribers`, `subreddit_type`, `post_id`, `post_name`, `title`, `url` (canonical reddit.com permalink), `permalink`, `content_url` (the external link, or the permalink for text posts), `domain`, `url_overridden_by_dest`, `post_hint`, `is_self`, `post_type` (self/link/image/video/gallery/poll), `content` (self-post body, markdown), `content_html` (self-post body, HTML)

**Author** — `author`, `author_id`, `author_flair`, `author_flair_background_color`, `author_flair_text_color`, `author_flair_type`, `author_flair_css_class`, `author_flair_richtext`, `author_flair_template_id`, `author_premium`, `author_is_blocked`, `author_patreon_flair`

**Engagement** — `score`, `ups`, `downs` (always 0 — Reddit's public API doesn't expose downvotes), `upvote_ratio`, `num_comments`

**Media** — `media_type` (text/link/image/video/gallery/embed), `has_media`, `images[]`, `gallery_images[]` (full resolution), `gallery_count`, `video_url` (direct MP4 for Reddit-hosted video), `video_duration_seconds`, `video_width`, `video_height`, `video_has_audio`, `video_bitrate_kbps`, `poll_data` (options, vote count, end time), `thumbnail_url`, `thumbnail_width`, `thumbnail_height`, `embed_provider`, `embed_type`, `embed_title`, `embed_author_name`, `embed_thumbnail_url`, `embed_html`, `embed_width`, `embed_height` (oEmbed metadata for link posts to YouTube/Imgur/Twitch/Vimeo/etc., when Reddit has cached it)

**Flair** — `link_flair`, `link_flair_background_color`, `link_flair_text_color`, `link_flair_type`, `link_flair_css_class`, `link_flair_richtext`, `link_flair_template_id`

**Flags & moderation** — `is_stickied`, `is_locked`, `is_archived`, `is_pinned`, `is_nsfw`, `is_spoiler`, `is_original_content`, `is_created_from_ads_ui` (posted via Reddit's Ads Manager, even though it looks organic), `is_crosspost`, `is_crosspostable`, `num_crossposts`, `num_duplicates`, `crosspost_parent_id`, `media_only`, `is_reddit_media_domain`, `quarantine`, `allow_live_comments`, `send_replies`, `no_follow`, `hide_score`, `content_categories`, `contest_mode`, `suggested_sort`, `is_meta`, `is_robot_indexable`, `gilded`, `total_awards_received`, `awards` (award-type breakdown: name, count, coin price, icon URL), `removed_by_category` (who removed it: moderator/reddit/author/etc.), `distinguished` (moderator/admin badge), `hidden`

**Timestamps & metadata** — `created_utc`, `created_at` (ISO-8601), `edited`, `edited_at`, `crawled_at`, `source` (`json` normally; `dom` when Reddit's data feed was unavailable and the posts were read from the listing page instead)

Posts with `source: "dom"` come from a single listing page and carry a reduced field set: `subreddit`, `subreddit_prefixed`, `post_id`, `post_name`, `title`, `author`, `score`, `upvote_ratio`, `num_comments`, `url`, `content_url`, `domain`, `post_type`, the `is_nsfw`/`is_stickied`/`is_locked`/`is_crosspost` flags, `gilded`, `total_awards_received`, `created_at`, `created_utc` and `crawled_at`.

## Output per comment

Returned when `includeComments` is enabled and `maxCommentsPerPost` is above 0. Comments follow their post in the dataset; replies Reddit only loads on demand ("more replies") are fetched afterwards and added at the end of that post's comments.

**Identity** — `dataType` (always `comment`), `subreddit`, `subreddit_prefixed`, `subreddit_id`, `subreddit_type`, `post_id`, `comment_id`, `comment_name`, `link_id` (fullname of the parent post), `parent_id` (fullname of the parent post or comment), `parent_kind`, `depth` (0 = top-level), `is_op` (author is also the post's author)

**Author** — `author`, `author_id`, `author_flair`, `author_flair_background_color`, `author_flair_text_color`, `author_flair_type`, `author_flair_css_class`, `author_flair_richtext`, `author_flair_template_id`, `author_premium`, `author_is_blocked`, `author_patreon_flair`

**Content** — `body` (markdown), `body_html`

**Engagement** — `score`, `ups`, `downs` (always 0), `score_hidden`, `controversiality`

**Flags & moderation** — `is_stickied`, `is_locked`, `archived` (locked from voting/replies after 6 months), `collapsed`, `collapsed_reason`, `collapsed_reason_code`, `collapsed_because_crowd_control`, `can_gild`, `comment_type`, `unrepliable_reason`, `no_follow`, `send_replies`, `gilded`, `total_awards_received`, `awards`, `removed_by_category`, `distinguished`

**Parent post context** — `post_url`, `post_title`

**Timestamps & metadata** — `permalink`, `created_utc`, `created_at`, `edited`, `edited_at`, `crawled_at`, `source`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `subreddits` | array | – | **Required.** Subreddit names, `r/`-prefixed names or full URLs |
| `maxPosts` | int | `25` | Max posts per subreddit after filtering (1–5,000; raise for full-subreddit mode) |
| `maxItems` | int | `10,000` | Global safety cap on total records (posts + comments) for the whole run (1–100,000) |
| `sort` | enum | `hot` | `hot`, `new`, `top`, `rising`, `controversial`, `best` (ignored when `postedAfter` or `fullSubreddit` is set, which use `new`) |
| `timeFilter` | enum | `day` | `hour`, `day`, `week`, `month`, `year`, `all` (for top/controversial) |
| `includeComments` | bool | `false` | Collect comments for every scraped post (needs `maxCommentsPerPost` above 0) |
| `maxCommentsPerPost` | int | `0` | Max comments per post (0–100,000). At `0` no comments are collected, even with `includeComments` on |
| `commentSort` | enum | `confidence` | `confidence`, `top`, `new`, `controversial`, `old`, `qa` |
| `postedAfter` | date | – | Only posts created on/after this date (UTC); switches the listing to `new` and stops at the first older post |
| `postedBefore` | date | – | Only posts created on/before this date (UTC) |
| `includeNSFW` | bool | `false` | Include 18+ posts |
| `onlyWithFlair` | bool | `false` | Drop posts without a flair tag |
| `postType` | enum | – | Keep only `self`, `link`, `image`, `video`, `gallery` or `poll` posts. Leave unset to keep all types |
| `minScore` | int | – | Keep only posts with a score at or above this value (-100,000–1,000,000) |
| `maxScore` | int | – | Keep only posts with a score at or below this value (-100,000–1,000,000) |
| `minComments` | int | – | Keep only posts with at least this many comments (0–1,000,000) |
| `maxComments` | int | – | Keep only posts with at most this many comments (0–1,000,000) |
| `minUpvoteRatio` | number | – | Keep only posts with an upvote ratio at or above this value (0–1) |
| `excludeStickied` | bool | `false` | Drop mod-stickied/pinned posts |
| `excludeRemoved` | bool | `false` | Drop posts removed by a moderator/Reddit or deleted by their author |
| `excludeCrossposts` | bool | `false` | Drop crossposted posts, keeping only original submissions |
| `excludeSpoilers` | bool | `false` | Drop posts tagged as a spoiler by their author |
| `titleContains` | string | – | Keep only posts whose title contains this text (case-insensitive) |
| `contentContains` | string | – | Keep only self posts whose body contains this text (case-insensitive) |
| `flairContains` | string | – | Keep only posts whose flair text contains this text (case-insensitive) |
| `authorFlairContains` | string | – | Keep only posts whose author flair text contains this text (case-insensitive) |
| `distinguished` | enum | – | `moderator`, `admin`, `special` (official posts only) or `none` (exclude all distinguished posts) |
| `authors` | array | – | Keep only posts submitted by these usernames |
| `domain` | string | – | Keep only posts whose link domain contains this text (e.g. `youtube.com`) |
| `minAwards` | int | – | Keep only posts with at least this many total awards (0–1,000,000) |
| `onlyOriginalContent` | bool | `false` | Keep only posts tagged by their author as original content (OC) |
| `excludeLocked` | bool | `false` | Drop posts whose comments were locked by a moderator |
| `excludeDeletedAuthor` | bool | `false` | Drop posts whose author account was deleted |
| `excludeArchived` | bool | `false` | Drop posts Reddit has archived (locked after 6 months) |
| `excludeAuthors` | array | – | Drop posts submitted by these usernames (e.g. bots like AutoModerator) |
| `excludeKeywords` | array | – | Drop posts whose title, body or flair contains any of these keywords |
| `minContentLength` | int | – | Keep only self posts whose body is at least this many characters (0–1,000,000) |
| `maxContentLength` | int | – | Keep only self posts whose body is at most this many characters (0–1,000,000) |
| `fullSubreddit` | bool | `false` | Use the `new` listing and keep paging until `maxPosts` is reached or Reddit's listing ends |

### Example: hot posts from a single subreddit

```json
{
  "subreddits": ["python"],
  "maxPosts": 50,
  "sort": "hot"
}
```

### Example: full comment threads with a keyword filter

```json
{
  "subreddits": ["technology"],
  "maxPosts": 20,
  "includeComments": true,
  "maxCommentsPerPost": 50,
  "commentSort": "top",
  "titleContains": "AI"
}
```

### Example: date-range collection with engagement filters

```json
{
  "subreddits": ["worldnews"],
  "postedAfter": "2025-06-01",
  "postedBefore": "2025-06-30",
  "minScore": 500,
  "minUpvoteRatio": 0.8,
  "excludeStickied": true
}
```

### Example: multi-subreddit brand monitoring

```json
{
  "subreddits": ["startups", "Entrepreneur", "SaaS"],
  "maxPosts": 100,
  "sort": "new",
  "postType": "self",
  "excludeAuthors": ["AutoModerator"],
  "excludeKeywords": ["megathread", "giveaway"]
}
```

## Use cases

- **Market research** — analyze authentic community opinions and pain points
- **Brand monitoring** — track mentions of your product across relevant subreddits (schedule daily runs)
- **Content ideation** — find viral topics and trending questions in your niche
- **Training data** — build large, clean Reddit corpora for NLP / LLM fine-tuning
- **Competitive intelligence** — monitor competitor discussions and launches
- **Academic & social research** — export a subreddit's history for sentiment or discourse analysis

## FAQ

**Do I need a Reddit API key or login?**
No. The actor uses Reddit's public web data — no API key, no OAuth, no cookies, no account.

**What happens with private, banned or misspelled subreddits?**
They return no records, and the run carries on with the remaining subreddits. If nothing at all is found, the run finishes with an empty dataset and a status message suggesting you check the names or loosen filters.

**Do I need to configure a proxy?**
No. There's no proxy setting to configure; the actor handles access to Reddit's public data on its own.

**Is Reddit scraping legal?**
Scraping publicly available data is generally allowed, but you must comply with Reddit's terms of service and use the data responsibly. Don't exceed reasonable request rates, and respect the site's usage policies.

**How many posts can I get from one subreddit?**
Up to `maxPosts` (max 5,000), but Reddit serves roughly 1,000 posts per listing, so that is the practical ceiling for one subreddit and sort. `fullSubreddit` pages through the `new` listing until it runs out; combining it with `postedAfter` lets you target a specific date window. Filters are applied after posts are loaded, so strict filters can return fewer posts than `maxPosts`.

**Why didn't I get any comments?**
`maxCommentsPerPost` defaults to `0`, which collects none. Set it to the number of comments you want per post (for example `100`) alongside `includeComments: true`. If `maxItems` is tight, the actor keeps every post and trims comments first.

**What is `maxItems` for?**
It's a global safety cap on the total number of records (posts + comments) pushed for the run — useful for bounding the size of large multi-subreddit jobs. Collection stops once the cap is reached.

**What does `removed_by_category` mean?**
It tells you who removed a post or comment: `moderator`, `automod_filtered`, `reddit`, `content_takedown` (legal), `author`/`deleted`. Live posts simply don't have this field.

**Can I get subreddit moderator lists or follower/member growth history?**
No. Only the current subscriber count (`subreddit_subscribers`) is part of the public data; moderator lists and historical follower counts aren't exposed.

**Why is `downs` always 0?**
Reddit's public API doesn't expose raw downvote counts to anonymous users — only `score` and `upvote_ratio` reflect real voting data.

**Can I schedule recurring runs?**
Yes — set up a schedule in Apify Console for daily/weekly brand monitoring or trend tracking.

**How do I integrate the data with other tools?**
Export to JSON, CSV, Excel, XML or HTML; call via REST API; connect to n8n, Zapier or Make; or expose it to AI agents via the [Apify MCP server](https://docs.apify.com/integrations/mcp).
