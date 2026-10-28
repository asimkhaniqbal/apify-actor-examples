# JobStreet Southeast Asia Job Search by Keyword & Country

Search JobStreet, a leading job portal across Singapore, Malaysia, the Philippines, Indonesia and Thailand, by keyword, browse all listings for one country, or pull featured/premium jobs — with salary, job type and a direct apply link on every job.

## What this actor does

- Searches job listings by keyword within a chosen country
- Browses every listing for a country without a keyword filter
- Fetches featured/premium job listings
- Filters `searchJobs` results by city/location within the country and by employment type
- Omits any field JobStreet doesn't show for a given job rather than returning a null or placeholder

## Output per job

| Field | Description |
|---|---|
| `jobId` | JobStreet's job identifier |
| `title` | Job title |
| `company` | Employer name |
| `location` | City or area within the country |
| `country` | Full country name |
| `salary` | Salary as shown, in local currency |
| `jobType` | Employment type, e.g. FULL_TIME |
| `postedDate` | When the job was posted |
| `jobUrl` | Direct link to the listing on JobStreet |
| `recordType` | Always `"job"` |
| `scrapedAt` | UTC timestamp when the record was scraped |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string (required) | searchJobs | searchJobs (keyword search), searchByCountry (browse all listings), or featuredJobs (featured/premium listings) |
| `keyword` | string | data analyst | Job title or keyword to search for — used in `searchJobs` mode |
| `country` | string | sg | Country to search: sg (Singapore), my (Malaysia), ph (Philippines), id (Indonesia), or th (Thailand) |
| `location` | string | – | City within the country to filter jobs by — used in `searchJobs` mode |
| `jobType` | string | (any) | Employment type: full-time, part-time, contract, internship, or permanent — used in `searchJobs` mode |
| `maxItems` | integer | 20 | Maximum number of jobs to return (1-500) |

### Example: keyword search in Singapore

```json
{
  "mode": "searchJobs",
  "keyword": "data analyst",
  "country": "sg",
  "maxItems": 20
}
```

### Example: browse all listings in Malaysia

```json
{
  "mode": "searchByCountry",
  "country": "my",
  "jobType": "full-time",
  "maxItems": 100
}
```

### Example: featured jobs in the Philippines

```json
{
  "mode": "featuredJobs",
  "country": "ph",
  "maxItems": 30
}
```

## Use cases

- **Regional recruitment research** — compare job titles, salaries and demand across five Southeast Asian markets
- **Salary benchmarking** — gather posted salary ranges for a role in its local currency
- **Job feed aggregation** — fold JobStreet listings into a broader Southeast Asia job board
- **Talent sourcing** — find companies actively hiring for a given role and location
- **Featured-listing tracking** — monitor which employers are paying for premium placement

## FAQ

**Which countries are covered?**
Singapore, Malaysia, the Philippines, Indonesia and Thailand, each on its own country-specific JobStreet domain.

**Are salaries shown in local currency?**
Yes — salary text reflects each country's own currency (SGD, MYR, PHP, IDR, THB) exactly as JobStreet displays it.

**Can I search across several countries in one run?**
Each run targets a single `country`; run it once per country and combine the resulting datasets if you need multi-country coverage.

**Why is `salary` missing from some jobs?**
Not every employer publishes a salary on JobStreet. When it's not shown, the field is left out of the record.

**What's the difference between `searchJobs` and `searchByCountry`?**
`searchJobs` filters by your `keyword`; `searchByCountry` returns whatever JobStreet is currently showing for that country without a keyword filter.

**Do I need a JobStreet account?**
No. The actor reads JobStreet's public listing pages; no login is required.

**Is this affiliated with JobStreet?**
No. This actor reads JobStreet's public pages and is not affiliated with, endorsed by, or sponsored by JobStreet or SEEK.
