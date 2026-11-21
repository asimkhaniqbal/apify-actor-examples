# Guru Freelance Jobs Scraper

Scrape live freelance job postings from [Guru.com](https://www.guru.com/d/jobs/) — one of the largest freelance marketplaces. Browse jobs by category, keyword, and budget type, or pull full job detail pages (description, skills, employer stats) from a list of job URLs. No login, no API key, no cookies required.

## What this actor does

- **Two modes:** `browse` (category listing) and `byJobId` (full job detail pages)
- **7 top-level categories** covering the whole marketplace: Programming & Development, Design & Art, Writing & Translation, Sales & Marketing, Administrative & Secretarial, Education & Training, Engineering & Architecture
- **Optional subcategory drill-down** (e.g. "Web Development & Design")
- **Keyword filter** — best-effort client-side match on title/description/skills
- **Budget-type filter** — Fixed Price / Hourly / Fixed Price or Hourly
- **Rich per-job data**: skills, budget range, hours/week, project duration, location preference, employer country, quotes received, application deadline
- **Detail mode** adds full job description, employer payment-verification status, feedback score, total spend, and jobs-posted history
- Automatically engages the free Apify AUTO datacenter proxy if Guru.com temporarily rate-limits direct requests
- Empty fields are always omitted — every field in the output is backed by real scraped data

## Output per job (browse mode)

- `jobId`, `title`, `jobUrl`
- `category`, `subcategory`, `skills[]`
- `description`
- `budgetType`, `budgetRange`, `hoursPerWeek`, `duration`, `locationPreference`
- `postedText`, `deadline`, `quotesReceived`
- `employerName`, `employerCountry`
- `scrapedAt`

## Output per job (byJobId mode)

`jobId`, `jobUrl`, `title`, `category`, `subcategory`, `skills[]`, `description`, `postedText`, `deadline`, `quotesReceived`, `employerName`, `employerCountry` (where available), plus:

- `budgetType` — the budget text shown on the job's detail page; unlike browse mode, this is not split into separate `budgetRange`/`hoursPerWeek`/`duration`/`locationPreference` fields, so those four fields don't appear on `byJobId` records
- `employerPaymentVerified` — `true` when Guru.com shows the "Payment Method Verified" badge for the employer
- `employer<Stat>` — one field per row of the employer's stats table on the job page (e.g. `employerFeedback`, `employerJobsPosted`, `employerTotalSpent`); the exact set of stats present depends on what Guru.com publishes for that employer

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `browse` | `browse` / `byJobId` |
| `category` | select | – | Top-level category, or `all` |
| `subcategory` | string | – | Optional subcategory name/slug |
| `keyword` | string | – | Best-effort keyword filter |
| `budgetType` | select | `any` | `any` / `Fixed Price` / `Hourly` / `Fixed Price or Hourly` |
| `jobUrls` | array | – | Full job URLs (mode=byJobId) |
| `maxItems` | int | `40` | Hard cap on emitted records (1–500) |
| `maxPages` | int | `10` | Hard cap on listing pages crawled (1–50) |
| `proxyConfiguration` | object | AUTO | Only engaged automatically if Guru.com rate-limits |

### Example: browse Programming & Development jobs

```json
{
  "mode": "browse",
  "category": "programming-development",
  "budgetType": "any",
  "maxItems": 40
}
```

### Example: fetch full detail for specific jobs

```json
{
  "mode": "byJobId",
  "jobUrls": [
    "https://www.guru.com/jobs/dentist-website-in-wordpress/2119938"
  ]
}
```

## Use cases

- **Freelance market research** — track how many jobs post per category and at what budget type
- **Lead generation for freelancers** — pull fresh postings in a skill category to find new clients
- **Rate benchmarking** — compare `budgetRange` and `budgetType` across jobs in the same subcategory
- **Employer due diligence** — pull full detail pages to check an employer's payment-verification status and posting history before bidding
- **Category trend tracking** — run `category=all` on a schedule to see which categories are most active over time

## FAQ

**Do I need a Guru.com account?** No. All data comes from public listing and job-detail pages.

**Why did I get 0 results?** Either the chosen category/subcategory combination has no active postings right now, or your keyword filter didn't match anything on the fetched pages — try a broader category or remove the keyword.

**Why is `keyword` described as "best-effort"?** Guru.com's own search box is client-side only, so there's no server-side search endpoint to query. The actor instead fetches the category/subcategory listing pages and filters the fetched jobs by keyword locally, which means a keyword can only match jobs that were already within `maxPages` of listings.

**What's the difference between `category` and `subcategory` browsing?** `category` is a true server-side filter on Guru.com's listing pages. `subcategory` narrows further within that category, but is matched against the subcategory text the actor already retrieved — pick the exact subcategory name shown on Guru.com for best results.

**Why do some jobs lack `employerCountry` or `quotesReceived`?** These only appear when Guru.com shows them on the listing or detail page; some employers don't display location, and newly posted jobs may not yet show a quote count.

**When would I need the proxy configuration?** Only if Guru.com temporarily rate-limits direct requests from Apify's infrastructure — the default free datacenter group is usually enough.

**Is this affiliated with Guru.com?** No, this is an independent third-party actor that reads Guru.com's public job pages. It is not affiliated with, endorsed by, or sponsored by Guru.com, Inc.
