# ZipRecruiter Job Search Results & Salary Scraper

Pull job listings straight from **ZipRecruiter.com** search-result pages: titles, companies, locations, parsed salary ranges, and remote/hybrid flags. Point it at a search URL or a keyword + location pair and it pages through the results for you.

## What this actor does

- Scrapes ZipRecruiter's public job-search result pages (not the detail pages)
- Accepts either a ready-made ZipRecruiter search URL (with your own filters baked in) or a plain `search` keyword + `location`
- Pages through results up to `maxItems`, de-duplicating cards (ZipRecruiter repeats each job in both the list and preview panes)
- Parses free-text salary strings (e.g. `$80K - $180K/yr`) into numeric `salaryMin`/`salaryMax` and a `salaryPeriod`
- Empty fields are omitted from records

## Output per job

- `id` — ZipRecruiter job-card identifier
- `jid` — short job ID parsed out of the listing URL
- `title` — job title
- `url` — full job posting URL
- `company` — hiring company name
- `companyUrl` — company profile URL on ZipRecruiter
- `companyLogo` — company logo image URL, when the card shows one
- `location` — location text as shown on the card (may include a `Remote`/`Hybrid` suffix)
- `city`, `state` — parsed out of `location`
- `isRemote` — present and `true` only when the location text mentions "remote"
- `isHybrid` — present and `true` only when the location text mentions "hybrid"
- `salary` — the raw salary text as shown on the card (e.g. `$80K - $180K/yr`)
- `salaryMin`, `salaryMax` — parsed numeric salary bounds in full dollars (not thousands)
- `salaryPeriod` — `year` / `hour` / `month` / `week`, parsed from the salary text
- `type` — `job_ziprecruiter` for a normal listing, or `job_ziprecruiter_blocked` for a run that produced no listings (see FAQ)
- `scrapedAt` — ISO 8601 timestamp of when the record was captured

When a run can't return any listings, it produces one `job_ziprecruiter_blocked` record instead, with `requestedUrls` (the URLs/keyword that were attempted), `blockedUrls` (the ones that were actually blocked, when any were reached), and a `message` explaining why.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | ZipRecruiter search-result URLs; any filters already in the URL (location, radius, salary, employment type) are kept as-is |
| `search` | string | – | Keyword shortcut, combined with `location` to build a search URL when `startUrls` is empty |
| `location` | string | – | City/state used with `search`; ignored when `startUrls` is provided |
| `maxItems` | integer | `50` | Maximum number of job listings to return (1–500) |
| `proxyConfiguration` | object | RESIDENTIAL (US) | Required — ZipRecruiter blocks requests that don't come through a residential US IP, so this is fixed and should not be disabled |

### Example: direct search URL

```json
{
  "startUrls": [
    "https://www.ziprecruiter.com/jobs-search?search=software+engineer&location=New+York%2C+NY"
  ],
  "maxItems": 100
}
```

### Example: keyword + location shortcut

```json
{
  "search": "data scientist",
  "location": "San Francisco, CA",
  "maxItems": 50
}
```

### Example: a search URL with ZipRecruiter's own filters already applied

```json
{
  "startUrls": [
    "https://www.ziprecruiter.com/jobs-search?search=registered+nurse&location=Austin%2C+TX&refine_by_employment=employment_type%3Aemployment_type%3Afull_time&days=5"
  ],
  "maxItems": 200
}
```

## Use cases

- **Talent intelligence** — see how many roles and what titles a company is actively posting
- **Compensation benchmarking** — collect the salary ranges ZipRecruiter displays for a role or region
- **Remote-work tracking** — filter results where `isRemote` is present to watch remote-hiring trends
- **Regional market scans** — compare hiring volume for the same keyword across different cities
- **Lead generation** — build a list of companies currently hiring in a given field

## FAQ

**What exactly gets scraped?**  The search-results page for the URL or keyword/location pair you give it — the same cards you'd see scrolling a ZipRecruiter search in a browser.

**Does it include the full job description?**  No. Only the information shown on the search-result card itself (title, company, location, salary snippet) is extracted; the detail page is not visited.

**Why are `isRemote`/`isHybrid`/`salaryMin`/`salaryMax` sometimes missing?**  They're only added to a record when ZipRecruiter's card actually shows that information — a listing with no visible salary simply has no salary fields, rather than a zero or placeholder value.

**Do I need to configure a proxy?**  No — `proxyConfiguration` ships preset to a residential US proxy because ZipRecruiter blocks plain datacenter traffic. Leave it as the default.

**Why does a run sometimes return a single `job_ziprecruiter_blocked` record instead of jobs?**  If every attempt for that run gets blocked, the actor returns one record describing which URLs were blocked instead of silently returning nothing, so you can tell a blocked run apart from a search with genuinely zero results.

**Why is the same job only returned once even though the page shows it twice?**  ZipRecruiter's own markup renders each listing in both the results list and a preview pane; this actor deduplicates by the card's internal ID so each job appears once.

**Can I pass any ZipRecruiter search filter?**  Yes — build the URL on ziprecruiter.com with whatever filters you want (location radius, days posted, employment type, salary range, etc.) and pass that full URL in `startUrls`; the filters are preserved exactly as ZipRecruiter encoded them.

**Is this affiliated with ZipRecruiter?**  No. This actor reads ZipRecruiter's public search-result pages and is not affiliated with or endorsed by ZipRecruiter.
