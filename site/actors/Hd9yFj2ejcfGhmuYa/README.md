# Himalayas Remote Job Finder with Salary & Tech Stack

Himalayas is a job board for remote roles. This actor searches its public listings by keyword, pulls the open roles at one company, or looks up a list of job URLs you already have — returning salary, tech stack and job links for each one.

## What this actor does

- Searches Himalayas job listings by free-text keyword, most recent first
- Fetches the currently open jobs posted by a single company, given its slug (up to `maxItems`)
- Turns a list of specific Himalayas job page URLs into structured records, as long as the job is still listed
- Filters by employment type and a minimum salary floor in every mode
- Omits any field Himalayas doesn't provide for a job, so a record never shows a null or placeholder value

## Output per job

| Field | Description |
|---|---|
| `jobId` | Unique job identifier taken from the job URL |
| `jobUrl` | Direct link to the job listing on Himalayas |
| `title` | Job title |
| `company` | Hiring company name |
| `companyUrl` | Link to the company's Himalayas profile |
| `location` | Countries applicants must be in (first five, then "(+more)"), or "Worldwide" when there are none |
| `remotePolicy` | Always `"remote"` — every job on Himalayas is a remote role |
| `salaryMin` / `salaryMax` | Annual salary range, when published |
| `salaryCurrency` | Currency code for the salary range, when a salary is published |
| `employmentType` | full-time, part-time, contract, or internship |
| `techStack` | Skill and technology tags attached to the listing |
| `categories` | Broader function areas the job belongs to, e.g. Developer, Product, Design |
| `postedAt` | UTC timestamp of when the job was posted |
| `description` | Job description HTML, or Himalayas' short excerpt when the full text isn't provided |
| `recordType` | Always `"job"` |
| `scrapedAt` | UTC timestamp of when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | searchJobs | `searchJobs` runs a keyword search with filters; `byCompany` returns every job for a company slug; `byJobUrls` enriches a list of specific job URLs |
| `searchQuery` | string | – | Free-text keyword query, e.g. "python backend", "product manager" — used in `searchJobs` mode |
| `companySlug` | string | – | Canonical company slug from Himalayas, e.g. "stripe", "gitlab" — used only in `byCompany` mode |
| `jobUrls` | array | `[]` | Himalayas job page URLs (`/companies/<slug>/jobs/<job>`) — used only in `byJobUrls` mode |
| `remotePolicy` | string | `""` (any) | Leave blank or set to "remote"; both return the same jobs, because every record is marked remote |
| `employmentType` | string | `""` (any) | Keep full-time, part-time, contract, or internship jobs. Jobs with no stated employment type are kept |
| `minSalary` | integer | – | Exclude jobs whose maximum salary (or minimum, when only that is given) is below this value; jobs with no published salary are also excluded once this is set. The number is compared to the listing's own salary figures in whatever currency they use |
| `maxItems` | integer | 20 | Hard cap on the number of job records returned (1–5000) |

### Example: keyword search with a salary floor

```json
{
  "mode": "searchJobs",
  "searchQuery": "data scientist",
  "employmentType": "contract",
  "minSalary": 120000,
  "maxItems": 30
}
```

### Example: all open roles at one company

```json
{
  "mode": "byCompany",
  "companySlug": "gitlab",
  "maxItems": 100
}
```

### Example: enrich a list of job URLs

```json
{
  "mode": "byJobUrls",
  "jobUrls": [
    "https://himalayas.app/companies/stripe/jobs/backend-engineer-123"
  ]
}
```

## Use cases

- **Remote job search** — filter by keyword, employment type and minimum salary to shortlist roles
- **Startup hiring trends** — track which companies are actively posting and in what roles
- **Salary benchmarking** — compare published pay ranges for a role or tech stack across companies
- **Recruiting and sourcing** — monitor what a target company is hiring for over time
- **Job board aggregation** — fold Himalayas listings into a broader remote-jobs feed

## FAQ

**Do I need a Himalayas account or API key?**
No. The actor reads Himalayas's public job listings; no login or key is required.

**What does `remotePolicy` mean if every job is remote anyway?**
Himalayas only lists remote roles, so the field is always `"remote"`. Any geographic restriction on who can apply shows up in `location` instead, for example "United States, Canada" or "Worldwide".

**How do I find a company's slug for `byCompany` mode?**
It's the part of the company's Himalayas URL after `/companies/`. For `https://himalayas.app/companies/stripe`, the slug is `stripe`.

**Why are `salaryMin`, `salaryMax` or `salaryCurrency` missing on some jobs?**
Not every company publishes a salary range on Himalayas. When a listing has no salary figures, those fields are left out of the record entirely.

**What happens if I set `minSalary` on a search?**
Jobs whose maximum salary is below that value are dropped, and jobs with no published salary at all are dropped too, since there's nothing to compare against your floor. There's no currency conversion: a listing paid in EUR or GBP is compared using its own numbers, so check `salaryCurrency` in the results.

**Can I pull every listing on Himalayas?**
Leave `searchQuery` blank in `searchJobs` mode to page through the newest listings, up to `maxItems` (at most 5,000 per run). The number returned also depends on how many jobs Himalayas has live at the time.

**Why was one of my job URLs skipped in `byJobUrls` mode?**
The actor finds each URL among that company's currently listed jobs (checking up to 200 of them). A job that has been closed, or a URL that isn't in the `/companies/<slug>/jobs/<job>` form, is skipped.

**Is this affiliated with Himalayas?**
No. This actor reads Himalayas's public job pages and is not affiliated with, endorsed by, or sponsored by Himalayas.
