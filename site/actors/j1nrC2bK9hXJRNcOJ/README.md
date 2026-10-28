# GulfTalent Job Listings Across 16 Middle East Markets

Search GulfTalent, a job board covering the UAE, Saudi Arabia, Qatar, Kuwait, Bahrain, Oman, Egypt and more, by keyword, or browse by category, industry, job type, nationality preference, career level, or a single employer's own careers page — with full salary, description and preference details on every job.

## What this actor does

- Searches by job-title keyword, optionally narrowed to a country or city
- Browses by job category, company industry, job type, Gulf-nationality preference, or seniority/career level
- Pulls every open role listed on one company's own GulfTalent careers page
- Fetches a list of specific job URLs directly
- Visits each job's own page to collect salary, full description, nationality/gender preference and Arabic-fluency requirement, and omits any of these fields a posting doesn't disclose

## Output per job

| Field | Description |
|---|---|
| `jobId` | GulfTalent's internal numeric job ID |
| `title` | Job title |
| `companyName` / `companyUrl` / `companyLogoUrl` | Hiring company details |
| `country` / `countrySlug` / `city` / `location` | Job location: full country name, GulfTalent's own country slug, city, and the raw "City, Country" string |
| `area` | Finer neighbourhood/district within the city, when disclosed, e.g. "Downtown Dubai - Dubai" |
| `salaryMin` / `salaryMax` / `salaryCurrency` / `salaryPeriod` | Posted salary range, when disclosed |
| `salaryLabel` | Free-text salary note, e.g. "Negotiable", only when GulfTalent shows text instead of a min/max range |
| `employmentType` | Full Time, Part Time, Contract, Temporary, or Internship |
| `jobFunction` | GulfTalent's job-function category, e.g. "Accounting & Audit" |
| `careerLevel` | Seniority level, when listed |
| `vacancyCount` | Number of open vacancies, when listed |
| `isRemote` | `true` only when GulfTalent flags the posting as remote |
| `companyIndustry` | Hiring company's industry sector |
| `nationalityPreference` | Nationalities the employer is open to, if specified |
| `genderPreference` | Gender preference, if specified |
| `arabicFluency` | Arabic-fluency requirement, if specified |
| `description` / `descriptionHtml` | Full job description, as plain text and original HTML |
| `companyDescription` | "About the company" blurb, when shown |
| `similarJobUrls` | Related job postings GulfTalent surfaces on the same page, when shown |
| `referenceNumber` | Recruiter's internal job reference |
| `postedAt` / `validThrough` | Posting date and application deadline |
| `sourceUrl` | Canonical GulfTalent job URL |
| `recordType` | Always `"job"` |
| `scrapedAt` | UTC timestamp when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | search | search, byCategory, byIndustry, byJobType, byNationality, byCareerLevel, byCompany, or byUrls |
| `titleKeyword` | string | accountant | Free-text job title to search for — used in `search` mode |
| `country` | string | `uae` | One of 16 markets, or `""` (Any country) — used in search, byCategory, byIndustry, byJobType |
| `city` | string | (any) | One of 29 major cities; overrides `country` to match the city's own country — used in search, byCategory |
| `category` | string | – | Job category to browse — used in `byCategory` mode |
| `industry` | string | – | Hiring company's industry sector — used in `byIndustry` mode |
| `jobType` | string | – | Contract, Internship, Part-Time, or Temporary — used in `byJobType` mode |
| `nationality` | string | – | Bahraini, Kuwaiti, Saudi, or UAE nationals — used in `byNationality` mode |
| `careerLevel` | string | – | Fresh Graduate, Junior, Experienced, Manager, or Director — used in `byCareerLevel` mode |
| `companyUrl` | string | – | A GulfTalent company-careers page URL or slug, e.g. "hays-careers" — used in `byCompany` mode |
| `startUrls` | array of strings | `[]` | Direct GulfTalent job URLs — used in `byUrls` mode |
| `minSalary` / `maxSalary` | integer | – | Drop jobs outside this posted salary range; jobs with no posted salary are kept either way |
| `employmentType` | string | (any) | Keep only jobs of this employment type; jobs with an unknown type are kept |
| `containsKeyword` | string | – | Keep only jobs whose title, description, or job function contains this text |
| `fetchFullDetails` | boolean | true | Visit each job's detail page for salary, full description, and preference fields. Turn off for a faster run limited to listing-card fields. Automatically re-enabled when `minSalary`, `maxSalary`, or `employmentType` is set. |
| `maxItems` | integer | 25 | Maximum number of job records to return (1-1000) |

### Example: search a role in a specific city

```json
{
  "mode": "search",
  "titleKeyword": "accountant",
  "city": "dubai",
  "maxItems": 50
}
```

### Example: browse one industry across all markets

```json
{
  "mode": "byIndustry",
  "industry": "it",
  "maxItems": 100
}
```

### Example: all open jobs at one employer

```json
{
  "mode": "byCompany",
  "companyUrl": "https://www.gulftalent.com/companies/hays-careers",
  "maxItems": 100
}
```

### Example: fetch specific job postings

```json
{
  "mode": "byUrls",
  "startUrls": [
    "https://www.gulftalent.com/uae/jobs/accountant-614857",
    "https://www.gulftalent.com/uae/jobs/cashier-620007"
  ]
}
```

## Use cases

- **Recruitment market monitoring** — track live vacancy volume by country, industry, or job function across the Gulf
- **Salary benchmarking** — compare posted salary ranges for a role across markets
- **Job feed aggregation** — fold fresh Middle East listings into a regional job board
- **Employer tracking** — watch which companies and industries are actively hiring
- **Candidate-fit screening** — filter by nationality preference, career level, or Arabic fluency requirements

## FAQ

**Do I need a GulfTalent account or API key?**
No. The actor reads GulfTalent's public job listing pages; no login, cookies, or key are required.

**Why do some jobs have no salary?**
Recruiters choose whether to disclose a salary on GulfTalent. When one isn't disclosed, `salaryMin`/`salaryMax` are simply left out of that record.

**Can I combine `country` and `city`?**
Yes — setting `city` automatically applies the matching country; if the two disagree, `city` takes precedence.

**What's the difference between `category` and `industry`?**
`category` is the job's function, like "Accounting & Audit"; `industry` is the hiring company's business sector, like "Oil & Gas". A finance role can appear under the finance category at an oil-and-gas-industry company.

**Why is `fetchFullDetails` on by default?**
It's required to get salary, full description, and preference fields at all. Turn it off only when you need a faster run with just title, company, location and URL.

**Can I filter by nationality or career level together with a country?**
Nationality-preference and career-level browsing run across all markets at once, since GulfTalent doesn't offer a per-country URL for those two axes. Combine the results with `containsKeyword` if you need to narrow them further, or use `search`/`byCategory`/`byIndustry`/`byJobType` when geography plus one of those filters is what you need.

**Is this affiliated with GulfTalent?**
No. This actor reads GulfTalent's public pages and is not affiliated with, endorsed by, or sponsored by GulfTalent.
