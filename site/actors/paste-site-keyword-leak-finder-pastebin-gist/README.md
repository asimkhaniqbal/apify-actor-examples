# Paste Site Keyword Leak Finder (Pastebin, Gist & More)

Check whether a keyword — an API key, an email domain, a company name — has surfaced on public paste sites. Search **Pastebin**, **GitHub Gist**, **Ideone**, **Paste.org**, and **paste.gg** in one run, or fetch a known paste URL, and get back the paste link, author, dates, syntax, views, raw link, and an optional full-text excerpt. Built for security researchers, threat-intel teams, and compliance monitoring.

## What this actor does

- **Three modes:** `searchKeywords` for keyword hunting, `byUrls` for checking specific public pastes, and `recentPastes` for scanning the newest public posts
- **Five live sources:** Pastebin archive and raw text, GitHub Gist search and metadata, Ideone recent codes, Paste.org trending, and paste.gg's public list
- **Rich metadata:** author, posted date, expiry, syntax, category, views, language, file list, and direct raw link where the source exposes them
- **Optional full text and regex extraction:** return the paste body up to a character limit and pull out emails, keys, URLs, or any pattern you define
- **No credentials or proxy setup:** public data only; empty fields are omitted rather than returned as null

## Output per paste

- `source` — `pastebin`, `gist`, `ideone`, `paste_org`, or `paste_gg`
- `url` — canonical public paste URL
- `title` — paste title, gist description, or first file name
- `snippet` — keyword-centred excerpt, or the first part of the text when no keyword applies
- `matchedKeyword`, `matchedKeywords[]` — first and all keywords that matched
- `sourceQuery` — keyword search that discovered the record
- `recordType` — `paste` or `gist`
- `pasteId`, `gistId` — native identifiers where available
- `author` — public author or owner name
- `postedAt`, `postedAtText` — creation time in ISO 8601 UTC and as shown on the site
- `expiresAt`, `expiryText` — expiry where the source reports one
- `syntax`, `category` — Pastebin syntax and category labels
- `views` — public view count
- `sizeBytes`, `language`, `visibility` — content and listing metadata
- `rawUrl` — direct raw-text link where available
- `files[]`, `fileCount` — gist or paste.gg file names, sizes, languages, and raw links
- `description`, `stars`, `forks`, `comments` — gist and paste.gg metadata
- `content`, `contentLength`, `contentTruncated` — only when `includeContent` is enabled
- `regexMatches[]`, `regexMatchCount` — only when `extractPatterns` is set
- `scrapedAt` — ISO 8601 collection timestamp

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `searchKeywords` | `searchKeywords`, `byUrls`, or `recentPastes` |
| `searchKeywords` | array | – | Keywords to search (mode=searchKeywords) |
| `startUrls` | array | – | Public paste URLs to fetch (mode=byUrls) |
| `sources` | array | `pastebin`, `gist`, `ideone`, `paste_org`, `paste_gg` | Which sites to search |
| `maxItems` | integer | `50` | Overall record limit (1–1000) |
| `maxItemsPerSource` | integer | `10` | Record limit per site (1–500) |
| `maxPastesToScan` | integer | `250` | Recent pastes to inspect per site; Pastebin archive scans cap at 250 keys |
| `syntaxes` | array | `[]` | Extra Pastebin syntax archives to scan; empty uses the main archive |
| `includeContent` | boolean | `false` | Return paste text in addition to metadata |
| `maxContentChars` | integer | `5000` | Text character limit when content is enabled |
| `extractPatterns` | array | `[]` | Regular expressions to extract from paste text |

### Example: search two keywords on Gist and Pastebin

```json
{
  "mode": "searchKeywords",
  "searchKeywords": ["api_key", "password"],
  "sources": ["gist", "pastebin"],
  "maxItems": 40,
  "maxItemsPerSource": 10
}
```

### Example: inspect specific public pastes

```json
{
  "mode": "byUrls",
  "startUrls": [
    {"url": "https://pastebin.com/xdGpC3wH"},
    {"url": "https://gist.github.com/fagoner/71c75995cf81fdc9e32fd17c1b5a43a0"}
  ],
  "includeContent": true,
  "maxContentChars": 3000
}
```

### Example: extract emails from the newest Gists

```json
{
  "mode": "recentPastes",
  "sources": ["gist"],
  "maxItems": 20,
  "maxItemsPerSource": 20,
  "includeContent": true,
  "extractPatterns": ["[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"]
}
```

### Example: scan older Pastebin syntax archives

```json
{
  "mode": "searchKeywords",
  "searchKeywords": ["internal-config"],
  "sources": ["pastebin"],
  "syntaxes": ["python", "javascript", "bash"],
  "maxPastesToScan": 250,
  "maxItemsPerSource": 25
}
```

## Use cases

- **Threat intelligence** — watch for company credentials, internal hostnames, or codenames appearing in public pastes
- **Breach detection** — check whether employee email addresses or customer domains have surfaced
- **Incident response** — fetch a suspicious paste and inspect its metadata and text in one step
- **Security research** — collect public proof-of-concept snippets and leaked configuration samples
- **Compliance monitoring** — keep an audit trail of public paste hits tied to your organization
- **Brand protection** — spot public mentions of a product, domain, or internal project name

## FAQ

**Which paste sites are covered?**
Pastebin, GitHub Gist, Ideone, Paste.org, and paste.gg. Pastebin is searched through its public archive and raw paste pages, Gist through its public search and gist metadata, and the other three through their newest public listings.

**Does Pastebin search cover all of history?**
No. Pastebin's archive shows recent public pastes and selected syntax archives, and it does not offer free pagination through its full history. A scan reaches up to 250 archive keys per run; older or already-deleted pastes may not appear.

**How many Gist results can I get?**
Gist search returns up to 10 results per page and up to 100 pages, so a single keyword is capped at roughly 1,000 search results. Use several specific keywords for broader coverage.

**Why do Ideone, Paste.org, and paste.gg return fewer hits?**
Those sites do not expose a public keyword search. The actor scans their newest public listings and checks the paste text for your keywords, so coverage depends on how many recent pastes are scanned. paste.gg's public list API returns at most 100 entries per request, and unauthenticated Gist API calls are rate-limited.

**Does this need cookies, an account, or a proxy?**
No. Only public pages are read, and the actor chooses a working connection automatically. You do not need to configure a proxy, log in, or supply an API key.

**Does it return private or deleted pastes?**
No. Private, unlisted, password-protected, and deleted pastes are out of scope. If a paste disappears between the archive listing and the content fetch, it is skipped.

**Is the full paste text always returned?**
Only when `includeContent` is enabled, and only up to `maxContentChars`. Paste text can contain sensitive material; handle it according to your organization's data-handling rules and report real leaks responsibly.

**Can I extract only certain values, such as emails or keys?**
Yes. Add regular expressions to `extractPatterns` and the actor returns every match in `regexMatches` without filtering out other pastes.

**Is this affiliated with Pastebin, GitHub, or the other sites?**
No. This is an independent actor that reads publicly accessible pages. It is not endorsed by or affiliated with any of the paste sites.
