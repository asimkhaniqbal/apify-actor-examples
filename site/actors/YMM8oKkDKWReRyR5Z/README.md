# Lever Job Board Postings by Company Slug

Extract job postings from any [Lever.co](https://www.lever.co/) company board. Get every open position for one or more companies, or fetch specific postings by ID — no account or authentication needed.

## What this actor does

Lever is an applicant tracking system used by companies like Netflix, Shopify, Duolingo, Coinbase, and many more. This actor lets you:

- **Scrape all open postings** from one or more Lever company boards in a single run
- **Fetch specific postings** by company slug and posting UUID for targeted lookups
- **Filter by team, location, or commitment type** (Full-time, Part-time, Contract, Internship)
- Collect **rich job metadata**: title, location, team/department, salary range, requirements, responsibilities, and apply links

## Output Data

Each posting record contains:

| Field | Description |
|-------|-------------|
| `id` | Lever posting UUID |
| `title` | Job title |
| `state` | Posting state (e.g., `published`) |
| `companySlug` | Lever company board slug |
| `companyName` | Company name, derived from the slug (e.g. `stripe` → `Stripe`) — a best-effort guess, not a field Lever's API publishes directly |
| `categories` | Object with `commitment`, `location`, `team`, `department` |
| `descriptionPlain` | Plain-text job description |
| `lists` | Structured lists (requirements, responsibilities) |
| `additional` | Additional information text |
| `salaryRange` | Salary object with `min`, `max`, `currency`, `interval` |
| `applyUrl` | Direct application URL |
| `hostedUrl` | Full Lever job listing URL |
| `sourceUrl` | Constructed Lever board URL |
| `createdAt` | ISO 8601 UTC timestamp when the posting was created |
| `recordType` | Always `"job"` |
| `scrapedAt` | ISO 8601 UTC timestamp of scrape |

### Sample Output

```json
{
  "id": "abc123-def456-ghi789",
  "title": "Senior Software Engineer",
  "state": "published",
  "companySlug": "stripe",
  "companyName": "Stripe",
  "categories": {
    "commitment": "Full-time",
    "location": "San Francisco, CA",
    "team": "Engineering",
    "department": "Product Engineering"
  },
  "descriptionPlain": "We are looking for a Senior Software Engineer...",
  "salaryRange": {
    "min": 150000,
    "max": 200000,
    "currency": "USD",
    "interval": "per year"
  },
  "applyUrl": "https://jobs.lever.co/stripe/abc123-def456-ghi789/apply",
  "hostedUrl": "https://jobs.lever.co/stripe/abc123-def456-ghi789",
  "sourceUrl": "https://jobs.lever.co/stripe/abc123-def456-ghi789",
  "createdAt": "2024-06-01T08:00:00+00:00",
  "recordType": "job",
  "scrapedAt": "2026-05-30T12:00:00+00:00"
}
```

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mode` | String (select) | `getJobs` | `getJobs` — all postings from boards; `getJob` — specific posting by ID |
| `companySlugs` | Array | – | Lever company slugs to scrape when `mode=getJobs` (e.g. `["netflix", "stripe"]`) |
| `companySlug` | String | – | Single company slug for fetching a specific posting when `mode=getJob` |
| `postingIds` | Array | – | List of Lever posting UUIDs to fetch when `mode=getJob` |
| `team` | String | – | Filter postings by team name (e.g. `"Engineering"`) |
| `location` | String | – | Filter postings by location string (e.g. `"San Francisco"`, `"Remote"`) |
| `commitment` | String (select) | `""` (All types) | Filter by job type: Full-time, Part-time, Contract, Internship |
| `maxItems` | Integer | `500` | Maximum number of posting records to emit (1–10,000) |

### How to Find a Company Slug

The company slug appears in the Lever job board URL. For example:
- `https://jobs.lever.co/stripe` → slug is `stripe`
- `https://jobs.lever.co/netflix` → slug is `netflix`

## Usage Examples

### Get all jobs from multiple companies

```json
{
  "mode": "getJobs",
  "companySlugs": ["stripe", "netflix", "figma"],
  "maxItems": 1000
}
```

### Get only engineering roles

```json
{
  "mode": "getJobs",
  "companySlugs": ["coinbase", "duolingo"],
  "team": "Engineering",
  "commitment": "Full-time"
}
```

### Fetch specific postings by ID

```json
{
  "mode": "getJob",
  "companySlug": "stripe",
  "postingIds": ["abc123-def456-ghi789", "xyz987-uvw654"]
}
```

### Get remote jobs only

```json
{
  "mode": "getJobs",
  "companySlugs": ["remote-company"],
  "location": "Remote",
  "maxItems": 200
}
```

## Use Cases

- **Job market research** — track hiring trends across high-growth tech companies
- **Competitive intelligence** — monitor which teams and roles competitors are building
- **Job aggregation** — power job boards or career sites with real-time Lever data
- **Recruitment analytics** — analyze salary ranges, team growth, and location trends
- **HR benchmarking** — compare job requirements and compensation across companies

## Frequently Asked Questions

**Is authentication required?**
No. Lever's public postings are completely open — no account or key needed.

**How do I find a company's Lever slug?**
Visit the company's job listings page. If it's hosted on Lever, the URL will contain `lever.co`. The slug is the part after `jobs.lever.co/`.

**Can I filter by team, location, and commitment type together?**
Yes — all three filters can be combined, and a posting only comes through when it matches every filter you've set.

**What is a posting UUID?**
Each Lever job posting has a unique UUID identifier (e.g., `abc123-def456-ghi789`). You can find it in the posting's URL on the company's Lever board.

**Is salary data always available?**
No — `salaryRange` is only included when the company has actually published it on the posting. Many postings omit salary entirely.

**How many postings can I scrape?**
Up to 10,000 postings per run, across as many company slugs as you list.

**How fresh is the data?**
Every run reads the company board live, so results reflect whatever is currently posted. Each record also includes a `createdAt` timestamp set by Lever when the posting was created.

**Is this affiliated with Lever?**
No. This is an independent actor that reads Lever's public job board data; it is not endorsed by or affiliated with Lever Inc.
