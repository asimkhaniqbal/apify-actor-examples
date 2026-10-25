# Facebook Group Feed & Engagement Extractor

Point this actor at any public Facebook group and pull its feed as structured data — no login, password, or cookies required. Each record carries the full post text, author details, photos and videos, a reaction breakdown, top comments, shares, and reshared content, plus optional group metadata like member count, admins, and rules.

## What this actor does

- **Scrapes any public Facebook group** by pasting a group URL, slug, or numeric group ID
- **No Facebook login, password, or cookies needed** — works entirely on public data
- **Captures full post content**: text, hashtags, photos/videos, timestamps, and author info
- **Full engagement data**: reaction breakdown by type (Like, Love, Haha, Wow, Sad, Angry, Care), total reactions, comment counts, shares, and top comments
- **Reshares included**: when a post shares another post, the original post's full content is captured too
- **Four sort orders**: top posts, chronological (newest first), recent activity, and buy/sell listings
- **Optional group profile data**: description, member count, admin names, group rules, and creation date
- **Monitor mode**: schedule recurring runs that return only newly published posts
- **Empty fields are omitted** — every record only contains data that was actually found, never `null` or blank placeholders

## Output per post

| Field | Type | Description |
|-------|------|--------------|
| `id` | string | Facebook's internal opaque post ID |
| `postId` | string | Real numeric post ID |
| `permalinkUrl` | string | Direct link to the post |
| `groupId` | string | Numeric ID of the group the post belongs to |
| `groupUrl` | string | Normalized URL of the group |
| `text` | string | Full post text |
| `hashtags` | array | Facebook hashtag page URLs found in the post text |
| `creationTime` | integer | Unix timestamp the post was created |
| `authorId` | string | Numeric ID of the post author |
| `authorName` | string | Name of the post author |
| `authorUrl` | string | URL of the post author's profile |
| `authorUrlIsFallback` | boolean | `true` when Facebook didn't provide a direct profile link and the URL was built from the author's numeric ID instead |
| `attachments` | array | Photos/videos attached to the post, including direct media URLs and auto-generated image captions where available |
| `reactionsTotal` | integer | Total reaction count |
| `reactionBreakdown` | array | Reaction counts by type, e.g. Like/Love/Haha/Wow/Sad/Angry/Care |
| `commentsCount` | integer | Total number of comments (including replies) |
| `sharesCount` | integer | Total number of shares |
| `topComments` | array | Top comments, each with text, author, reactions, and permalink |
| `sharedPost` | object | The original post's full data if this post is a reshare, otherwise omitted |
| `group` | object | Group profile data (only present when "Enrich with group info" is enabled) — see below |
| `inputUrl` | string | The input URL that produced this record |
| `scrapedAt` | string | ISO 8601 timestamp of when the record was scraped |

If a group can't be reached or has no posts in the chosen sort order, one diagnostic record is pushed for it instead: `_status` (the access/error state), `_inputUrl`, `_groupUrl`, `_message`, `scrapedAt`.

### `group` object (when "Enrich with group info" is enabled)

| Field | Type | Description |
|-------|------|--------------|
| `id` | string | Numeric group ID |
| `url` | string | Normalized group URL |
| `description` | string | Full group description |
| `memberCount` | integer | Exact member count |
| `memberCountFormatted` | string | Abbreviated member count, e.g. `"75.9K"` |
| `createdAt` | integer | Unix timestamp the group was created |
| `admins` | array | Group admin names |
| `rules` | array | Group rules, each with a title and description |
| `privacy` | string | Group privacy setting, e.g. `"Anyone can see who's in the group and what they post."` |
| `discoverability` | string | Group discoverability setting |
| `tags` | array | Group category tags |

## Input

| Field | Type | Default | Description |
|-------|------|---------|--------------|
| `startUrls` | array | – | Facebook group URLs, bare slugs, or numeric group IDs (required) |
| `sortBy` | string | `top` | `top` / `chronological` / `recent_activity` / `listings` |
| `maxItems` | integer | `50` | Maximum posts to return per group (1–500) |
| `includeTopComments` | boolean | `true` | Include each post's top comments |
| `enrichGroupInfo` | boolean | `false` | Add group description, member count, admins, and rules to every post |
| `monitoringMode` | boolean | `false` | Only return posts not seen in a previous run for the same group |

### Supported group URL formats

- **Group URL**: `https://www.facebook.com/groups/pythondevelopers/`
- **Group with a numeric ID**: `https://www.facebook.com/groups/123456789012345/`
- **Bare slug**: `pythondevelopers`

### Example: basic scrape of a group's top posts

```json
{
    "startUrls": ["https://www.facebook.com/groups/pythondevelopers/"],
    "sortBy": "top",
    "maxItems": 50,
    "includeTopComments": true
}
```

### Example: scrape with group profile data included

```json
{
    "startUrls": ["https://www.facebook.com/groups/pythondevelopers/"],
    "sortBy": "chronological",
    "maxItems": 20,
    "enrichGroupInfo": true
}
```

### Example: monitor a group for new posts only

```json
{
    "startUrls": ["https://www.facebook.com/groups/pythondevelopers/"],
    "maxItems": 50,
    "monitoringMode": true
}
```

### Example: multiple groups in one run

```json
{
    "startUrls": [
        "https://www.facebook.com/groups/pythondevelopers/",
        "https://www.facebook.com/groups/javascript/"
    ],
    "sortBy": "recent_activity",
    "maxItems": 100
}
```

## Example output

```json
{
    "id": "UzpfSTEwMDA3NjkwNTU4ODg0MDpWSzoyODQ0OTkzNTA4Nzk3OTA4MQ==",
    "postId": "28449935087979081",
    "permalinkUrl": "https://www.facebook.com/groups/pythondevelopers/posts/28449935087979081/",
    "groupId": "527595400639758",
    "groupUrl": "https://www.facebook.com/groups/pythondevelopers/",
    "text": "PYTHON #60 CODING CHALLENGE - can you solve it?",
    "hashtags": ["https://www.facebook.com/hashtag/python"],
    "creationTime": 1789471995,
    "authorId": "100076905588840",
    "authorName": "Dhiraj Kumar",
    "authorUrl": "https://www.facebook.com/profile.php?id=100076905588840",
    "authorUrlIsFallback": true,
    "attachments": [
        {
            "type": "Photo",
            "url": "https://scontent.xx.fbcdn.net/v/...",
            "width": 1080,
            "height": 1920,
            "accessibilityCaption": "May be an image of text that says \"PYTHON #60 CODING CHALLENGE...\""
        }
    ],
    "reactionsTotal": 796,
    "reactionBreakdown": [
        {"type": "Like", "count": 450},
        {"type": "Love", "count": 310},
        {"type": "Wow", "count": 20},
        {"type": "Care", "count": 15},
        {"type": "Haha", "count": 1}
    ],
    "commentsCount": 42,
    "sharesCount": 3,
    "topComments": [
        {
            "id": "Y29tbWVudDoxMjM0NQ==",
            "text": "Nice challenge!",
            "authorName": "Jari Abbas",
            "authorId": "100001234567890",
            "authorUrl": "https://www.facebook.com/jari.abbas.691553",
            "createdAt": 1789472500,
            "reactionsTotal": 5,
            "replyCount": 1,
            "permalinkUrl": "https://www.facebook.com/groups/pythondevelopers/posts/28449935087979081/?comment_id=12345"
        }
    ],
    "inputUrl": "https://www.facebook.com/groups/pythondevelopers/",
    "scrapedAt": "2026-09-16T12:00:00+00:00"
}
```

## Use cases

- **Community monitoring** — track new posts and discussions in Facebook groups relevant to your brand, product, or industry
- **Market research** — analyze engagement patterns, sentiment, and popular topics inside niche communities
- **Content aggregation** — collect group discussions for newsletters, digests, or dashboards
- **Lead generation** — identify active members and recurring topics in professional or interest-based groups
- **Trend tracking** — use monitor mode on a schedule to catch new activity as it happens
- **Competitor and brand monitoring** — watch what's being said in industry-specific or competitor-run Facebook groups

## FAQ

**Do I need to log in to Facebook to use this actor?**
No. This actor works entirely without a Facebook login, password, cookies, or any authentication. It only accesses data that's publicly visible to anyone, logged in or not.

**Is this affiliated with Facebook or Meta?**
No. This is an independent, third-party tool that reads publicly available Facebook group data. It is not created, endorsed, or affiliated with Meta Platforms, Inc.

**Can I scrape private or closed Facebook groups?**
No. Private and closed groups require Facebook group membership to view posts, and this actor cannot bypass that requirement. It only works with public groups.

**What happens if a group is private, invalid, or doesn't exist?**
The run does not fail. Instead, the actor returns a diagnostic record noting that no posts could be retrieved for that specific group, so you can tell "group is private or invalid" apart from "group genuinely has zero posts" — and any other groups in the same run are unaffected.

**Can I search Facebook groups by keyword?**
Not directly — Facebook doesn't offer true keyword search across a group's full post history. This actor fetches a group's feed in the sort order you choose (top, chronological, recent activity, or listings) as Facebook itself renders it. If you need posts about a specific topic, scrape the feed and filter the results for your keyword afterward.

**How is the member count calculated?**
Member count is Facebook's own reported number from the group's About page, returned both as an exact figure and an abbreviated string (e.g. `"75.9K"`). It's only included when "Enrich with group info" is enabled.

**What does monitor mode do?**
When enabled, the actor remembers which post IDs it has already returned for each group, across separate runs. The next run only pushes posts that weren't seen before — ideal for scheduled runs where you just want new activity.

**Why is `authorUrl` sometimes a `profile.php?id=...` link instead of a normal profile URL?**
Facebook doesn't always expose a normal vanity profile URL for a post's author. When that happens, the actor builds a valid link from the author's numeric ID so the field is never left empty. Check `authorUrlIsFallback` to tell the two cases apart.

**How many posts can I scrape per group?**
Up to 500 posts per group in a single run, controlled by the "Max posts per group" input field.

**How fresh is the data?**
Every run fetches Facebook's live group feed at the moment the actor runs — there's no caching or delay, so results reflect what's currently visible on Facebook.

**Can I export the results to Excel, CSV, or Google Sheets?**
Yes. Every Apify actor run's dataset can be exported to JSON, CSV, Excel (XLSX), XML, or pushed directly to Google Sheets from the run's results page.

**Can I scrape multiple Facebook groups in one run?**
Yes. Add as many group URLs, slugs, or IDs as you like to the input list — each is scraped in the same run and tagged with its own `groupUrl` in the output.
