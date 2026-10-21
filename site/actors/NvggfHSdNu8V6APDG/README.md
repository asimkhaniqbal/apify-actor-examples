# LinkedIn Post Reactor Identity & Reaction Type Scraper

Find out exactly who reacted to a LinkedIn post and how. Give it a post URL (or just the activity ID) plus your own LinkedIn session cookie, and it returns each reactor's name, headline, profile link, and the specific reaction they left — like, celebrate, love, support, insightful, funny, or curious.

## What this actor does

- Scrapes the reactor list for one or more LinkedIn posts, identified by URL or bare activity ID, up to `maxReactorsPerPost` per post
- Returns each reactor's name, headline, profile URL, and avatar image
- Reports the exact reaction type each person left, not just a generic "liked" count
- Flags whether a reactor is a person or a company page, and whether LinkedIn shows them as verified
- Filters reactors by reaction type, and by a minimum reaction date when LinkedIn supplies one
- Fields LinkedIn doesn't expose for a given reactor are left out of that record rather than filled with a placeholder

## Output per reactor

- `reactorName` — the reactor's full name
- `reactorProfileUrl` — link to the reactor's LinkedIn profile or company page
- `reactorHeadline`, `authorHeadline` — the reactor's current headline/job title
- `reactorAvatar` — the reactor's profile photo URL
- `reactionType` — one of `like`, `celebrate`, `love`, `support`, `insightful`, `funny`, `curious`
- `authorType` — `PERSON` or `COMPANY`
- `authorFollowersCount` — the reactor's follower count, when LinkedIn exposes it
- `verifiedStatus` — present (as `true`) only when LinkedIn shows the reactor as a verified account
- `reactedAt` — when the reaction was left, when LinkedIn provides a timestamp
- `mutualConnections` — mutual-connection text LinkedIn shows for that reactor, when available
- `postUrl` — the canonical LinkedIn post URL
- `postId` — the post's LinkedIn activity ID
- `inputPostUrl` — the exact URL or ID you supplied for this post
- `scrapedAt` — when the actor started on that post

### Notice record when the cookie doesn't work

If `cookie` is empty, or LinkedIn rejects it or reports the session as expired, the run ends without reactor data and the dataset holds a single notice record instead:

- `_status` — always `no_data`
- `_reason` — `cookie-missing` or `cookie-invalid`
- `_message` — a short description of the problem
- `_help` — how to export a fresh `li_at` cookie or full cookie JSON
- `scrapedAt` — when the notice was written

If the cookie text can't be read at all, the run fails with an error message instead. If LinkedIn rejects the session part-way through, the run stops at that point and keeps the reactors already collected.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `postUrls` | array of strings | – | LinkedIn post URLs or bare activity IDs to scrape reactors from. Required. |
| `cookie` | string | – | Your LinkedIn session cookie — either the `li_at` value or a full cookie JSON export. Required. |
| `maxReactorsPerPost` | integer | `100` | Maximum reactors to collect per post (1–5000); LinkedIn typically exposes up to about 1,000. |
| `reactionTypeFilter` | array of strings | – | Only return reactors with these friendly reaction types (`like`, `celebrate`, `love`, `support`, `insightful`, `funny`, `curious`). Combined with `reactionTypes` (see below). |
| `reactionTypes` | array of strings | `["LIKE", "PRAISE", "EMPATHY", "INTEREST", "APPRECIATION", "ENTERTAINMENT"]` | Filter using LinkedIn's own raw reaction codes (`LIKE`, `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION`, `ENTERTAINMENT`). The default lists six of the seven types, so it acts as a filter that leaves out `curious` reactions; set it to `[]` to stop filtering by it. |
| `minDate` | string | – | Only return reactions newer than this date (`YYYY-MM-DD`). Reactions without a date from LinkedIn are kept. |
| `language` | string | `en_US` | Intended LinkedIn locale for the session; the current version does not apply it, so it has no effect on the output. |
| `proxyConfiguration` | object | – | Optional Apify proxy configuration; a residential proxy is recommended. |

### Reaction type reference

| Friendly name | LinkedIn's raw code |
|---|---|
| like | LIKE |
| celebrate | PRAISE |
| love | EMPATHY |
| support | APPRECIATION |
| insightful | INTEREST |
| funny | ENTERTAINMENT |
| curious | MAYBE |

## Example: all reactors on one post, every reaction type

```json
{
  "postUrls": ["https://www.linkedin.com/feed/update/urn:li:activity:7234567890123456789/"],
  "cookie": "YOUR_LI_AT_COOKIE",
  "reactionTypes": [],
  "maxReactorsPerPost": 1000
}
```

## Example: only "insightful" and "love" reactions since a given date

```json
{
  "postUrls": ["7234567890123456789"],
  "cookie": "YOUR_LI_AT_COOKIE",
  "reactionTypeFilter": ["insightful", "love"],
  "reactionTypes": [],
  "minDate": "2026-01-01"
}
```

## Example: several posts in one run

```json
{
  "postUrls": [
    "https://www.linkedin.com/posts/username_slug-7234567890123456789-xxxx/",
    "https://www.linkedin.com/feed/update/urn:li:activity:7111111111111111111/"
  ],
  "cookie": "YOUR_LI_AT_COOKIE",
  "maxReactorsPerPost": 200
}
```

## Use cases

- **Warm lead generation** — pull everyone who reacted to a competitor's or industry leader's post into a prospect list
- **Community insight** — see the mix of titles, companies, and reaction types your own posts attract
- **Recruiting** — spot professionals already engaging with content about your industry or role
- **Influencer vetting** — check whether reactions on a sponsored post look like real engagement from real accounts
- **Sales intelligence** — filter to the reaction types that signal real interest (`insightful`, `love`) rather than a passive `like`

## FAQ

**Why does this actor need a LinkedIn cookie?**
LinkedIn only shows a post's reactor list to signed-in members, so a valid `li_at` cookie (or full cookie export) from your own session is required for every run. Without a working cookie you get the notice record described above instead of reactors.

**How many reactors can I get per post?**
Up to `maxReactorsPerPost` (5,000 at most). The actor stops earlier when LinkedIn reports no more reactors, keeps rate-limiting the requests, or returns empty pages, and LinkedIn may not list every reactor on very popular posts.

**What's the difference between `reactionTypeFilter` and `reactionTypes`?**
They filter the same thing two ways: `reactionTypeFilter` uses the friendly names (`like`, `love`, etc.), `reactionTypes` uses LinkedIn's own raw codes (`LIKE`, `EMPATHY`, etc.). When both are set, a reactor is kept if its type appears in either list. Because `reactionTypes` defaults to six codes, a `reactionTypeFilter` only narrows the results if you also set `reactionTypes` to `[]` (as in the example above). There is no raw code for `curious`, so use `reactionTypeFilter` for it.

**Why is `authorFollowersCount`, `reactedAt` or `mutualConnections` missing on some reactors?**
LinkedIn doesn't expose those fields for every reactor in every response; when a field has no data, it's left off the record instead of being filled with a zero or empty string.

**Can I scrape reactions from multiple posts in one run?**
Yes, add multiple entries to `postUrls` — the actor processes each post and tags every reactor record with its `postId` and `inputPostUrl`.

**Does this work on posts from company pages, not just people?**
Yes, as long as you can access the post with your account, and reactors who are company pages are flagged with `authorType: "COMPANY"`.

**Is this affiliated with LinkedIn?**
No. This is an independent, third-party actor that uses your own LinkedIn session to read publicly visible reaction data; it is not affiliated with or endorsed by LinkedIn.
