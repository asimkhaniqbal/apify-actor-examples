# Reddit Subreddit Post & Comment Harvester

Harvest posts from any public subreddit — hot, new, top, rising, controversial or best — with score, author, flair, selftext, images, galleries, video, polls and crossposts. Add full nested comment threads when you need the discussion too. Rich date, score, comment-count, upvote-ratio, keyword and author filters; no API key, no login, no proxy setup.

## What this actor does

- **Any public subreddit:** bare name (`python`), `r/`-prefixed name (`r/python`) or a full Reddit URL; multiple subreddits in one run
- **Six listing modes:** `hot`, `new`, `top`, `rising`, `controversial`, `best`; `top` and `controversial` support Reddit's six time windows
- **Deep history:** follows Reddit's listing pages to the practical ceiling (roughly 1,000 posts for hot/new, about 500 for top/controversial)
- **Posts with media:** score, upvote ratio, comment count, author, flair, selftext, images, gallery images (full resolution), video (HLS + MP4), poll options and votes, crosspost parent
- **Optional comment threads:** full nested trees with 6 comment sorts, depth, parent links, OP flag, edited/deleted flags and awards
- **Rich filtering:** date range, score, comment count, upvote ratio, post type, flair, title/body keywords, authors, domain, awards, content length and moderation flags
- **Empty fields are omitted** from the output — no need to defend against `null`/empty values downstream

## Output per post

- `dataType` — always `post`
- `subreddit`, `subreddit_prefixed`, `subreddit_id`
- `post_id`, `post_name`, `title`
- `author`, `author_id`
- `url`, `permalink`, `content_url`, `domain`
- `is_self`, `post_type` (`self`/`link`/`image`/`video`/`gallery`/`poll`), `post_type_raw`
- `score`, `upvote_ratio`, `num_comments`
- `link_flair`, `link_flair_background_color`, `flair_url`
- `content` (self-post text), `content_html`
- `thumbnail_url`
- `created_utc`, `created_at` (ISO-8601 UTC)
- `is_stickied`, `is_nsfw`, `is_spoiler`, `is_crosspost`
- `is_archived` — when the comment mode loads the post page
- `total_awards_received`, `top_award_id`, `top_award_icon_url`
- `media_type`, `has_media`
- `images[]`, `gallery_images[]`, `gallery_count`
- `video_url` (direct MP4), `video_hls_url` (HLS)
- `poll_data` — options, option ids, total votes, closing time
- `crosspost_parent_id`, `crosspost_parent_subreddit`, `crosspost_parent_url`
- `language`, `crawled_at`, `source`

For crossposts, `content` is the parent text Reddit renders in the listing, while `post_type_raw` is `crosspost` and the `crosspost_parent_*` fields identify the source post.

## Output per comment

Returned when `includeComments` is on and `maxCommentsPerPost` is above 0. The parent post is emitted first, then its comments.

- `dataType` — always `comment`
- `subreddit`, `subreddit_prefixed`, `subreddit_id`
- `post_id`, `post_url`, `post_title`, `link_id`
- `comment_id`, `comment_name`, `parent_id`, `parent_kind`
- `author`, `avatarUrl`
- `body`, `body_html`
- `score`, `total_awards_received`
- `depth` (0 = top-level), `position`, `parentPositions`
- `is_op`, `collapsed`, `contentType`
- `isCommentDeleted`, `isAuthorDeleted`, `isHighlighted`, `hasPaidAward` — present only when true
- `created_utc`, `created_at`, `edited`, `edited_at`
- `permalink`, `crawled_at`, `source`

## Fields not available anonymously

Reddit no longer exposes these to logged-out readers of the public pages, so they are omitted rather than guessed: author flair, raw upvotes/downvotes, post `distinguished` (mod/admin badge), `removed_by_category`, post `locked`, original-content marker, `gilded`, the award-type breakdown, video duration/bitrate/dimensions, and oEmbed title/author/HTML for external links. Link posts still carry their target URL and preview image.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `subreddits` | array | – | **Required.** Subreddit names, `r/`-prefixed names or full URLs |
| `maxPosts` | int | `25` | Max posts per subreddit after filters (1–5,000) |
| `maxItems` | int | `10,000` | Global safety cap on total records (posts + comments) |
| `sort` | enum | `new` | `hot`, `new`, `top`, `rising`, `controversial`, `best` |
| `timeFilter` | enum | `day` | `hour`, `day`, `week`, `month`, `year`, `all` (top/controversial) |
| `includeComments` | bool | `false` | Collect comments for every harvested post |
| `maxCommentsPerPost` | int | `0` | Max comments per post (0–100,000); needs `includeComments` |
| `commentSort` | enum | `confidence` | `confidence`, `top`, `new`, `controversial`, `old`, `qa` |
| `postedAfter` | date | – | Only posts created on/after this date (UTC); switches the listing to `new` |
| `postedBefore` | date | – | Only posts created on/before this date (UTC) |
| `includeNSFW` | bool | `false` | Include posts explicitly flagged NSFW |
| `onlyWithFlair` | bool | `false` | Drop posts without a link flair |
| `postType` | enum | – | `text`, `link`, `image`, `video`, `gallery`, `poll`, `crosspost`, `multi_media` |
| `minScore` | int | – | Minimum post score |
| `maxScore` | int | – | Maximum post score |
| `minComments` | int | – | Minimum comment count |
| `maxComments` | int | – | Maximum comment count |
| `minUpvoteRatio` | number | – | Minimum upvote ratio (0–1) |
| `excludeStickied` | bool | `false` | Drop pinned/stickied posts |
| `excludeCrossposts` | bool | `false` | Drop crossposts |
| `excludeSpoilers` | bool | `false` | Drop spoiler-tagged posts |
| `titleContains` | string | – | Keep posts whose title contains this text |
| `contentContains` | string | – | Keep self posts whose body contains this text |
| `flairContains` | string | – | Keep posts whose flair contains this text |
| `authors` | array | – | Keep posts by these usernames |
| `excludeAuthors` | array | – | Drop posts by these usernames |
| `excludeKeywords` | array | – | Drop posts whose title/body/flair contains any keyword |
| `domain` | string | – | Keep posts whose link domain contains this text |
| `minAwards` | int | – | Minimum total awards |
| `excludeDeletedAuthor` | bool | `false` | Drop posts with a deleted/removed author |
| `minContentLength` | int | – | Minimum self-post body length |
| `maxContentLength` | int | – | Maximum self-post body length |
| `fullSubreddit` | bool | `false` | Walk the `new` listing as deep as Reddit allows |

### Example: 25 new posts from r/Python

```json
{
  "subreddits": ["python"],
  "maxPosts": 25,
  "sort": "new"
}
```

### Example: top image posts with media

```json
{
  "subreddits": ["pics"],
  "maxPosts": 50,
  "sort": "top",
  "timeFilter": "week",
  "postType": "gallery"
}
```

### Example: posts plus full comment threads

```json
{
  "subreddits": ["technology"],
  "maxPosts": 20,
  "sort": "hot",
  "includeComments": true,
  "maxCommentsPerPost": 100,
  "commentSort": "top"
}
```

### Example: brand monitoring with filters

```json
{
  "subreddits": ["startups", "Entrepreneur", "SaaS"],
  "maxPosts": 100,
  "sort": "new",
  "postedAfter": "2026-10-01",
  "minScore": 5,
  "excludeAuthors": ["AutoModerator"],
  "excludeKeywords": ["megathread", "giveaway"]
}
```

## Use cases

- **Market research** — collect authentic community opinions and pain points
- **Brand monitoring** — track product mentions across subreddits on a schedule
- **Content ideation** — find trending questions and high-engagement topics
- **Training data** — build clean Reddit corpora with full comment context for NLP/LLM work
- **Competitive intelligence** — monitor launches, sentiment and competitor discussions
- **Academic and social research** — export a subreddit's reachable history for discourse analysis

## FAQ

**Do I need a Reddit API key or login?**
No. The actor reads Reddit's public web data — no API key, no OAuth, no cookies, no account.

**Do I need to configure a proxy?**
No. The actor chooses a working connection to Reddit automatically. Just fill in the input and run it.

**What happens with private, banned or misspelled subreddits?**
They return no records and the run continues with the remaining subreddits. If nothing is found, the run finishes with a status message suggesting you check the names or loosen filters.

**How many posts can I get from one subreddit?**
Up to `maxPosts` (5,000), but Reddit serves roughly 1,000 posts per hot/new listing and about 500 for top/controversial. `fullSubreddit` walks the `new` listing to that ceiling. Filters are applied after posts are loaded, so strict filters can return fewer rows than `maxPosts`.

**Why did I get no comments?**
`maxCommentsPerPost` defaults to `0`, which collects none. Set it (for example `100`) and enable `includeComments`. If `maxItems` is tight, the actor keeps every requested post and trims comments first.

**Why are some fields missing from a record?**
The actor omits values Reddit does not expose publicly (author flair, raw upvotes/downvotes, post distinguished/removed flags, award breakdown, video duration). Empty values are omitted rather than emitted as `null`, so records stay clean.

**Why is the video URL a low-resolution MP4?**
Reddit serves a direct low-resolution MP4 plus an HLS playlist. The actor provides both: `video_url` for durable downloads and `video_hls_url` for the adaptive stream when Reddit's signed playlist is available. HLS links can expire; `video_url` is the stable choice.

**Why are some NSFW or spoiler posts still included?**
`includeNSFW` filters posts Reddit explicitly flags. Posts in 18+ communities are served to the actor's adult-content session and cannot be distinguished at community level; use `excludeSpoilers` or keyword filters for tighter control.

**Can I schedule recurring runs?**
Yes — create a schedule in Apify Console for daily or weekly monitoring. Use `postedAfter` or `sort: "new"` to collect only recent posts.

**How do I export or integrate the data?**
Download JSON, CSV, Excel or HTML from the run, call the dataset API, or connect to n8n, Make, Zapier or the Apify MCP server.
