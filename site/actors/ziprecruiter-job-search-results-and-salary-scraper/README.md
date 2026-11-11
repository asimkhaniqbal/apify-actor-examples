# ZipRecruiter Jobs & Salary Scraper

Collect **ZipRecruiter job postings and salary benchmarks** in one run. Search by keyword across locations, browse a single employer, paste search/job/company URLs, or ask for salary percentiles by role and city. Job rows carry the full description, structured pay with a source flag, benefits, company profile, dates and the real apply destination.

## What this actor does

- **Three modes:** search jobs, scrape exact URLs, or read salary benchmarks.
- **Multi-query × multi-location:** every keyword is searched in every location, deduplicated by ZipRecruiter's own job identity.
- **Company-scoped search:** set a company name or paste a `/co/<company>/Jobs` URL to get only that employer's postings.
- **Rich job rows:** full description text and HTML, structured pay (min/max, period, currency, annualized values, provided vs predicted), benefits, workplace and employment types, posting dates, education, skills, company industry/website/size/HQ/rating, and apply links.
- **Salary directory:** median, 10th/25th/75th/90th percentile, average annual and hourly pay, related titles and top-paying cities — something plain job-board scrapers usually skip.
- **Filters that work:** employment type, remote/hybrid/on-site, experience level, posted within N days, radius, and minimum salary (maximum is best effort).
- **HTTP-first and cheap:** search, detail and salary pages are fetched directly with a Safari TLS profile. A headed browser with a US residential exit is used automatically only when the source challenges the request.
- **No setup:** no cookies, account, API key or proxy configuration. Omit-empty everywhere: a value the source does not show is left out, never sent as `null`.

## Modes

| Mode | What it reads | Example input |
|---|---|---|
| `search` | Keyword × location searches, or a company name | `searchQueries: ["registered nurse"]`, `locations: ["Chicago, IL"]` |
| `byUrls` | Search, job, company or salary URLs you already have | `startUrls: ["https://www.ziprecruiter.com/c/..."]` |
| `salary` | Salary pages for role × location, or exact salary URLs | `salaryRoles: ["data analyst"]`, `salaryLocations: ["Austin, TX"]` |

## Output

Job records use `type: job_ziprecruiter` and `recordType: job`.

Identity: `id`, `jid`, `listingKey`, `openSeatId`, `matchId`, `url`, `title`.

Company: `company`, `companyId`, `companyUrl`, `companyLogo`, `companyWebsite`, `companyIndustries`, `companySize`, `companyHqLocation`, `companyRating`, `companyAbout`, `companyFoundedYear`.

Location: `location`, `city`, `state`, `country`, `countryCode`, `postalCode`, `county`, `locationId`, `workplaceTypes`, `isRemote`, `isHybrid`.

Pay: `salary`, `salaryMin`, `salaryMax`, `salaryPeriod`, `salaryCurrency`, `salaryMinAnnual`, `salaryMaxAnnual`, `salarySource` (`provided`, `parsed`, `predicted` or `missing`), `isEstimatedSalary`.

Job: `description`, `descriptionHtml`, `shortDescription`, `employmentTypes`, `benefits`, `educationLevels`, `experienceMonths`, `postedAt`, `rollingPostedAt`, `postedAgo`, `validThrough`, `isActive`, `isNew`, `skills`, `occupationalCategory`, `industry`.

Apply and meta: `applyUrl`, `applyType`, `directApply`, `shareUrl`, `inputUrl`, `searchQuery`, `descriptionFetchFailed`, `scrapedAt`.

Salary records use `type: salary_ziprecruiter` and `recordType: salary`: `salaryRole`, `salaryLocation`, `salaryMedian`, `salaryP10`, `salaryP25`, `salaryP75`, `salaryP90`, `salaryAverage`, `salaryAverageHourly`, `salaryCurrency`, `salaryPeriod`, `salaryUrl`, `relatedTitles`, `topPayingCities`, `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | Select | `search` | `search`, `byUrls` or `salary` |
| `searchQueries` | String list | `[]` | Keywords searched in every location |
| `locations` | String list | `["United States"]` | Cities, states, ZIPs or nationwide |
| `companyName` | String | – | Only this employer's jobs; empty keywords browse all of them |
| `startUrls` | String list | `[]` | Search, job, company or salary URLs for `byUrls` |
| `jobType` | Select | `any` | `full_time`, `part_time`, `contract`, `contract_to_hire`, `temporary`, `as_needed`, `other`, `internship` |
| `workplace` | Select | `any` | `remote`, `hybrid`, `onsite` |
| `experienceLevels` | Multi-select | `[]` | `no_experience`, `junior`, `mid`, `senior` |
| `daysPosted` | Integer | `0` | Posted within N days (0–30); 0 = any |
| `radiusMiles` | Integer | `0` | Radius around each location (0–100); 0 = default |
| `salaryMin` | Integer | `0` | Annual minimum; 0 = any |
| `salaryMax` | Integer | `0` | Annual maximum (best effort); 0 = any |
| `fetchDescriptions` | Boolean | `true` | Fetch detail pages for full descriptions and pay |
| `includeCompanyProfile` | Boolean | `true` | Add industry, website, size, HQ, rating and about |
| `maxItems` | Integer | `50` | Total records across all searches |
| `maxSearches` | Integer | `100` | Cap on keyword × location searches |
| `salaryRoles` | String list | `[]` | Role titles for `salary` mode |
| `salaryLocations` | String list | `["United States"]` | Locations for `salary` mode |
| `salaryUrls` | String list | `[]` | Exact `/Salaries/...` URLs; overrides role/location pairs |

### Example: keyword search with filters

```json
{
  "mode": "search",
  "searchQueries": ["registered nurse"],
  "locations": ["Chicago, IL"],
  "jobType": "full_time",
  "daysPosted": 7,
  "maxItems": 50
}
```

### Example: several queries and locations

```json
{
  "mode": "search",
  "searchQueries": ["data engineer", "analytics engineer"],
  "locations": ["Texas", "Florida"],
  "workplace": "remote",
  "experienceLevels": ["senior"],
  "salaryMin": 120000,
  "maxItems": 200
}
```

### Example: one company's openings

```json
{
  "mode": "search",
  "companyName": "Visa",
  "locations": ["United States"],
  "fetchDescriptions": true,
  "maxItems": 100
}
```

### Example: exact URLs

```json
{
  "mode": "byUrls",
  "startUrls": [
    "https://www.ziprecruiter.com/jobs-search?search=fire+alarm+technician&location=New+York%2C+NY&days=7",
    "https://www.ziprecruiter.com/c/Company/Job/Title/-in-Chicago,IL?jid=abcdef1234567890",
    "https://www.ziprecruiter.com/co/Visa/Jobs"
  ],
  "maxItems": 100
}
```

### Example: salary benchmarks

```json
{
  "mode": "salary",
  "salaryRoles": ["software engineer", "data analyst"],
  "salaryLocations": ["New York, NY", "Austin, TX"],
  "maxItems": 10
}
```

## Use cases

- **Talent intelligence:** follow hiring volume and role mix for a company, city or industry.
- **Compensation research:** compare provided versus predicted pay and build salary bands from percentiles.
- **Recruiting pipelines:** export clean rows with real apply links and workplace types.
- **Remote-work tracking:** measure remote, hybrid and on-site mixes across markets.
- **Job-board products:** refresh an internal board with scheduled runs and deduplicate on `jid`.

## Limitations

- ZipRecruiter's public data has no latitude/longitude, so this actor does not invent coordinates.
- Search results arrive in ZipRecruiter's own order; the public site ignores sort parameters, so no sort input is offered.
- One search is capped at roughly 10,000 results. Split broad topics by keyword, location or filter for more.
- The salary maximum filter is best effort because ZipRecruiter's ceiling parameter does not narrow reliably; the minimum filter is server-side and is also checked on each row.
- `salarySource: predicted` means ZipRecruiter estimated the range, not that the employer published it. Values are kept and flagged rather than silently mixed with provided pay.
- US and Canada listings on ziprecruiter.com are covered. The separate UK and Ireland sites are not.
- A very broad run can hit detail-page rate limits; the actor pauses, backs off and keeps card rows with `descriptionFetchFailed` instead of failing the whole run.

## FAQ

**Do I need a ZipRecruiter account, cookies or a proxy?**
No. Only public pages are read, and the actor chooses its own network path. There is nothing to configure.

**What is the difference between the three modes?**
`search` discovers jobs from keywords, locations or a company. `byUrls` reads exact search, job, company or salary URLs. `salary` reads salary benchmark pages and returns percentile/average pay.

**Why do some rows have no salary?**
ZipRecruiter does not publish pay for every posting. When it shows an estimated range, the row is marked `salarySource: predicted` and `isEstimatedSalary: true`; when it shows nothing, the pay fields are omitted rather than filled with zero.

**Do I get the full job description?**
Yes when `fetchDescriptions` is on. It returns both plain text and HTML. Turn it off for faster, cheaper card-only runs.

**What does `descriptionFetchFailed: true` mean?**
The detail page could not be read for that posting, so the row was kept from search-card data. Re-run those job URLs later if you need the description.

**Why is the same job missing from a second search?**
Jobs are deduplicated by ZipRecruiter's job id across every query and location in the run, so a posting found twice is delivered once. Running the same search on another day can return different rows as ZipRecruiter re-ranks and refreshes listings.

**How current is the salary data?**
Salary pages are ZipRecruiter's own estimates and are returned as shown, including median, percentiles, average annual/hourly pay, related titles and top-paying cities. They are benchmarks, not offers.

**Can I filter for 1-Click Apply jobs?**
There is no reliable server-side filter for it, so it is not exposed as an input. `applyType` on each row tells you whether the application is external, internal or quick-apply.

**How many results can I get?**
`maxItems` supports up to 10,000 per run, and that also matches ZipRecruiter's per-search ceiling. For larger datasets, split the work into multiple runs by keyword or location.

**Is this affiliated with ZipRecruiter?**
No. It is an independent tool that reads ZipRecruiter's public job and salary data.
