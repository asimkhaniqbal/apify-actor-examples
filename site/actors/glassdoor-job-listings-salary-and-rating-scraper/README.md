# Glassdoor Job Listings Salary & Rating Scraper

Scrape **Glassdoor** job listings with salary percentiles, employer ratings and company profiles, full descriptions, and the site's own search filters. Search by keyword and location, paste search or company URLs, or refresh known job IDs across 20 Glassdoor country sites.

## What this actor does

- **Two modes:** `search` for keyword + location searches, and `byUrl` for search pages, company jobs pages, job URLs, and job IDs
- **Full filter surface:** posting date, employment type, seniority, company rating, salary range, Easy Apply, company size, industry, job function, radius, and sort order
- **Salary percentiles:** p10, median, and p90 with currency, pay period, and whether the employer or Glassdoor supplied the figure
- **Company context:** overall rating, logo, profile URL, size, revenue, headquarters, website, founded year, CEO, review count, and sub-ratings when available
- **Detail enrichment:** full description, exact posting and expiry dates, benefits, education and experience requirements, coordinates, and the Glassdoor apply link
- **Multi-country:** United States, United Kingdom, Canada, Ireland, France, Germany, Spain, Italy, Netherlands, Belgium, Austria, Switzerland, Australia, New Zealand, India, Singapore, Hong Kong, Brazil, Mexico, and Argentina
- **Omit-empty output:** fields Glassdoor does not provide are left out instead of returned as null, zero, or an empty list

## Output per job listing

- `jobListingId` — Glassdoor's internal listing ID
- `jobTitle`, `normalizedJobTitle` — posted title and Glassdoor's normalized role title
- `companyName`, `companyId` — employer and its Glassdoor ID
- `companyRating`, `companyLogo`, `companyProfileUrl` — overall rating, logo, and profile link
- `location`, `city`, `state`, `country`, `postalCode` — location as displayed plus structured parts when available
- `latitude`, `longitude` — coordinates when Glassdoor provides them
- `salary_min`, `salary_median`, `salary_max` — 10th percentile, median, and 90th percentile pay
- `salaryCurrency`, `salaryPeriod`, `salarySource` — currency, pay period, and pay source
- `datePosted`, `validThrough` — ISO posting and expiry dates when details are enabled
- `ageInDays` — listing age as shown in search results
- `employmentType`, `employmentTypes`, `jobTypes` — normalized and site-specific employment types
- `easyApply`, `directApply`, `isSponsored` — apply and placement flags
- `applyUrl` — Glassdoor's apply link for the listing
- `description` — the search-result description snippet
- `fullDescription`, `fullDescriptionHtml` — full posting text and HTML when details are enabled
- `benefits`, `educationRequirements`, `experienceMonths`, `experienceDescription` — structured requirements and benefits
- `industry`, `occupation`, `occupationId` — employer industry and Glassdoor occupation
- `skills` — skills Glassdoor extracted from the posting
- `companySize`, `companyRevenue`, `companyWebsite`, `companyHeadquarters`, `companyType`, `companyFoundedYear`, `companyCeo`, `companyReviewCount`, `companySubRatings` — employer profile fields
- `jobUrl` — direct link to the listing
- `searchQuery`, `searchLocation`, `searchUrl`, `sourceDomain` — search context
- `recordType`, `scrapedAt` — record metadata

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` or `byUrl` |
| `searchQueries` | array | `[]` | Job titles, skills, or company names to search for (mode=search) |
| `location` | string | – | City, state, or country; leave empty for the country's whole market |
| `country` | string | `united-states` | Glassdoor country site |
| `startUrls` | array | `[]` | Search, job, or company jobs page URLs (mode=byUrl) |
| `jobIds` | array | `[]` | Glassdoor job IDs to refresh (mode=byUrl) |
| `postedWithin` | string | – | Any time, 24 hours, 3, 7, 14, or 30 days |
| `employmentType` | string | `all` | All, full-time, part-time, or contract |
| `seniority` | string | `all` | All, entry, mid-senior, director, or executive |
| `minRating` | string | `0.0` | Minimum employer rating: 1.0 through 4.0 |
| `minSalary` | integer | – | Minimum yearly pay in the country's currency |
| `maxSalary` | integer | – | Maximum yearly pay in the country's currency |
| `easyApplyOnly` | boolean | `false` | Only Easy Apply jobs |
| `remoteOnly` | boolean | `false` | Prefer remote jobs; the site filter is best-effort |
| `companySize` | string | `any` | Employer size band |
| `industry` | string | `any` | Employer industry |
| `jobFunction` | string | `any` | Role function |
| `radius` | string | – | Distance around the location in miles |
| `sortBy` | string | `relevance` | Relevance, newest first, or oldest first |
| `maxItems` | integer | `100` | Hard cap across all queries and slices (1–1000) |
| `includeDetails` | boolean | `false` | Fetch each job page for full details |
| `maximizeCoverage` | boolean | `false` | Run extra date, type, rating, and salary slices to go beyond one search's depth |

## Example: US search with rating and Easy Apply filters

```json
{
  "mode": "search",
  "searchQueries": ["software engineer"],
  "location": "New York, NY",
  "country": "united-states",
  "minRating": "4.0",
  "easyApplyOnly": true,
  "maxItems": 50,
  "includeDetails": true
}
```

## Example: UK search for recent contract roles

```json
{
  "mode": "search",
  "searchQueries": ["data analyst"],
  "location": "London",
  "country": "united-kingdom",
  "postedWithin": "7",
  "employmentType": "contract",
  "minSalary": 50000,
  "maxItems": 30
}
```

## Example: all current roles at one company

```json
{
  "mode": "byUrl",
  "startUrls": [
    { "url": "https://www.glassdoor.com/Jobs/Google-Jobs-E9079.htm" }
  ],
  "maxItems": 100,
  "includeDetails": true
}
```

## Example: refresh known job IDs

```json
{
  "mode": "byUrl",
  "jobIds": ["1010224897915", "1010283907867"],
  "country": "united-states",
  "includeDetails": true
}
```

## Use cases

- **Salary benchmarking** — compare Glassdoor's 10th, 50th, and 90th percentile pay for a role across cities, companies, and countries
- **Recruiting intelligence** — see which employers are hiring, for what, and how they are rated
- **Competitor hiring monitoring** — schedule a daily or weekly run for a company or job family
- **Lead generation for staffing agencies** — build lists of companies with open requisitions in a skill area
- **Job-board aggregation** — feed a niche board or newsletter with fresh listings
- **Labor-market research** — measure demand by industry, function, seniority, and location

## FAQ

### Do I need a Glassdoor account, cookies, or a proxy?

No. The actor reads public Glassdoor pages without logging in and chooses a working connection automatically. Just fill in the input and run it.

### Are salaries employer-reported or estimated?

Both happen. Check `salarySource`: `EMPLOYER_PROVIDED` means the posting states the pay; `ESTIMATED` means Glassdoor estimated it. When a listing publishes no pay, the salary fields are omitted.

### Why did I get fewer results than `maxItems`?

Glassdoor reports a large match count but stops serving new pages well before that number. The actor stops when the cursor chain is exhausted or several pages bring no new listings, and reports the site's own match count in the run status. Turn on `maximizeCoverage` to split the search by date, job type, rating, and salary for more rows.

### Why are some fields missing from a record?

Glassdoor does not provide every field for every listing. Company profiles, benefits, ratings, coordinates, and salary are the most common gaps. Fields with no value are omitted instead of returned as null. If `includeDetails` is on but `fullDescription`, dates, or company profile fields are absent, Glassdoor blocked the job page for this run; the record still contains every search-result field.

### What does `includeDetails` change?

With details on, each job page is opened for the full description, exact dates, benefits, requirements, coordinates, apply link, and company profile. With details off, the actor uses the search results only: title, company, location, salary percentiles, rating, snippet, and flags. The daily test runs with details off.

### Can I search more than one country in one run?

Each run uses one country site. Run the actor once per country to compare markets; `sourceDomain` records which site produced each row.

### Does the remote filter always work?

Remote-only searches run market-wide. Glassdoor's remote filter can report matches but serve no rows when it is combined with a city, so the actor deliberately drops the city filter for remote-only runs and searches the selected country.

### Can I scrape a company's open roles?

Yes. Use `mode=byUrl` with the company's Glassdoor jobs page URL, or search the company name as a keyword.

### Why did a job ID or job URL return no record?

Job IDs and direct job URLs need Glassdoor's job detail page. If that page is blocked for the current network, the actor cannot rebuild the record; keyword/company searches and company jobs pages still work. Try a search by title or company instead of a bare job ID.

### Is this affiliated with Glassdoor?

No. This is an independent, third-party actor that reads Glassdoor's public job pages. It has no connection to Glassdoor.
