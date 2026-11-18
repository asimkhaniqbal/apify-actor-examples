# Arbeitnow Global Job Board Extractor

Arbeitnow is a European job board with a strong English-speaking, remote and visa-friendly
section. This actor reads the site's public job feed and its own search pages, then returns one
clean row per current listing: company, location, remote flag, tags, employment labels, full
description, posting date, expiry date and apply link.

The actor is cookieless. It needs no Arbeitnow account, API key, proxy or browser session.

## What it can do

- Search by keyword across job title, company and description.
- Filter with Arbeitnow's own category, city and collection pages: Python, Backend, Berlin,
  Munich, Remote, English speaking, Visa sponsorship, Jobs with relocation, 4 day work week and
  more.
- Require one or more tags (all must match), filter by company, location text, remote status and
  job types.
- Choose a 24-hour, 3-day or 7-day posting window and sort by newest or relevance.
- Scrape specific Arbeitnow job URLs instead of searching.
- Optionally open each selected job page to add the employer logo, expiry date, employment type
  and curated benefits such as visa sponsorship or relocation support.
- Deduplicate listings across pages and stop exactly at your row limit.

Arbeitnow's feed holds roughly the last seven days of listings. Older jobs are not available
from the source, so this actor does not claim an archive.

## Input

| Field | Type | Default | What it does |
| --- | --- | --- | --- |
| `mode` | Select | Search jobs | `search` finds jobs with the filters below; `byUrls` reads the job URLs you paste. |
| `searchQuery` | String | – | Keyword matched by Arbeitnow across title, company and description. Leave blank for all jobs. |
| `category` | Select | Any | One of Arbeitnow's topic, city or collection pages (Python, Berlin, Remote, Visa sponsorship, English speaking, …). |
| `tags` | Array | – | Jobs must carry every tag listed, e.g. `["Software Development", "Remote"]`. |
| `companies` | Array | – | Company names, company slugs or company-page URLs. Any match is kept. |
| `location` | String | – | Keep jobs whose location text contains this phrase, e.g. `Berlin` or `Germany`. |
| `urls` | Array | – | Full Arbeitnow job links used by `byUrls` mode. |
| `remote` | Boolean | `false` | Keep only jobs Arbeitnow flags as remote. |
| `jobTypes` | Array | – | Keep jobs whose employment labels contain any listed phrase, e.g. `Full Time`, `Internship`, `Werkstudent`. |
| `datePosted` | Select | `7d` | Posting window: last 24 hours, last 3 days or last 7 days. |
| `sortBy` | Select | Newest first | Newest first or Arbeitnow's relevance ranking. |
| `includeDetails` | Boolean | `false` | Open each selected job page for logo, expiry, employment type and curated benefits. |
| `maxItems` | Integer | 50 | Maximum rows to return (1–5,000). The daily prefill uses 3. |

### Example: remote Python roles from the last week

```json
{
  "mode": "search",
  "searchQuery": "python",
  "remote": true,
  "datePosted": "7d",
  "sortBy": "newest",
  "maxItems": 50
}
```

### Example: visa-sponsorship collection in Berlin

```json
{
  "mode": "search",
  "category": "visa sponsorship",
  "location": "Berlin",
  "datePosted": "7d",
  "maxItems": 100
}
```

### Example: company jobs with detail extras

```json
{
  "mode": "search",
  "companies": ["Zattoo", "https://www.arbeitnow.com/jobs/companies/n8n"],
  "includeDetails": true,
  "maxItems": 25
}
```

### Example: specific job URLs

```json
{
  "mode": "byUrls",
  "urls": [
    "https://www.arbeitnow.com/jobs/companies/zattoo/senior-backend-engineer-python-berlin-435689"
  ]
}
```

## Output

Each row contains the fields below. Fields the source does not provide for a particular job are
left out rather than filled with a guess.

| Field | Description |
| --- | --- |
| `slug` | Stable Arbeitnow job identifier, useful for deduplication. |
| `title` | Job title as posted by the employer. |
| `company` | Hiring company name. |
| `companySlug` | Company identifier from the listing URL. |
| `companyUrl` | Arbeitnow company page. |
| `companyLogoUrl` | Employer logo (detail extras or URL mode). |
| `location` | Location text exactly as published. |
| `remote` | Arbeitnow's remote flag. |
| `jobTypes` | Employment and experience labels such as `Full Time` or `Werkstudent`. |
| `tags` | Category and tech labels such as `Software Development` or `Remote`. |
| `description` | Full job description as plain text. |
| `postedAt` | ISO 8601 posting datetime. |
| `validThrough` | ISO 8601 expiry datetime (detail extras or URL mode). |
| `employmentType` | Human-readable employment type from the job page. |
| `benefits` | Curated benefits from the job page, e.g. `English speaker friendly`. |
| `visaSponsorship` | `true` only when the curated benefits mention visa sponsorship. |
| `englishSpeakingFriendly` | `true` when the curated benefits mark the role English-speaking friendly. |
| `relocationSupport` | `true` when the curated benefits mention relocation support. |
| `applyUrl` | Arbeitnow's apply/company-portal link. |
| `sourceUrl` | Direct link to the listing on Arbeitnow. |
| `scrapedAt` | ISO 8601 datetime when the row was collected. |

## How the modes work

**Search jobs** uses Arbeitnow's own search endpoint, so keywords, categories, tags, date windows
and sorting are applied by the site. The actor then reads the public feed to attach the full
structured record (remote flag, job types, tags, description, posting date). If a listing is
still in the 7-day search window but has just left the feed, its detail page is read instead.

**Scrape these job URLs** reads each page directly. When the same listing is still in the feed,
the structured feed fields are merged in; otherwise the page's own labels and JSON-LD supply the
record. Filters are a search-mode feature: URL mode returns the URLs you paste (up to `maxItems`),
and invalid or expired URLs are skipped with a warning.

## Limitations and FAQ

**Does it need an Arbeitnow account or API key?** No. Every surface this actor reads is public.

**How far back does it go?** About seven days. Arbeitnow removes older listings from its public
feed; the actor reports this plainly instead of returning stale rows.

**Why is `remote` missing or sparse?** It is Arbeitnow's own flag. Many remote-friendly jobs are
not flagged. For broad remote coverage, use the `Remote` category or add remote-related keywords;
the actor does not invent a flag from description text.

**Why are `companyLogoUrl`, `validThrough`, `benefits` or `employmentType` often absent?** They
come from optional detail-page enrichment. Turn on `includeDetails` (or use `byUrls`) to fetch
them. Arbeitnow does not publish them for every listing.

**Does Arbeitnow publish salary or seniority?** No reliable structured field. Some descriptions
mention pay and some labels mention experience, but the actor does not fabricate a salary or a
seniority level.

**What does the visa flag mean?** `visaSponsorship: true` only appears when Arbeitnow's curated
benefits explicitly say visa sponsorship is possible. To browse the site's own visa collection,
set `category` to `Visa sponsorship`.

**Are saved tasks from the old version still supported?** Yes. `searchQuery`, `tags`, `remote` and
`maxItems` keep working with the same meaning. Older alias keys such as `remoteOnly`, `keywords`,
`maxJobs` and `jobUrls` are mapped in code even though they are not shown on the form.

**Is this affiliated with Arbeitnow?** No. This is an independent tool and is not endorsed by or
sponsored by Arbeitnow.

## Use cases

- Recruiters building a daily shortlist of current EU tech roles.
- Relocation and visa teams tracking the site's visa-sponsorship collection.
- Job-board operators and newsletters that need deduplicated, structured listings.
- Researchers monitoring which cities, tags and companies are hiring this week.
