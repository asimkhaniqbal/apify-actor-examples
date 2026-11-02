# Jobs.ch Swiss Job Listings by Region & Industry

Scrape **jobs.ch**, Switzerland's largest job board, through its public search and company APIs. Run classic keyword searches, browse by filters, resolve employers by name or company ID, or paste jobs.ch search/job URLs. Listings can be enriched with the full description, apply URL and method, jobs.ch salary estimate, contacts, language requirements, occupational categories, regions, locations, application deadline, benefits and employer profile/ratings.

No login, cookies, API key or proxy is needed.

## Modes

| Mode | What it does |
|---|---|
| `search` | Classic keyword search across one or more queries. With `searchMode: "semantic"` it uses jobs.ch's fuzzy, cross-language search page instead. |
| `browse` | Filter-only browsing with no keyword. |
| `byCompany` | Resolves company names through jobs.ch's company directory and searches all openings. Company IDs are also accepted and are the most reliable option. |
| `byUrls` | Accepts jobs.ch search URLs (their filters are preserved) and job-detail URLs. Detail URLs report `jobStatus`: `online`, `offline`, `expired` or `unknown`. |
| `byRegion` | Legacy mode: requires at least one value in `regions`; filters only. |
| `byIndustry` | Legacy mode: requires at least one value in `industries`; filters only. |
| `byEmploymentType` | Legacy mode: requires at least one value in `employmentTypes`; filters only. |
| `byCategory` | Legacy mode: requires at least one value in `categories`; filters only. |

## Filters (all multi-value)

- `regions` — 24 Swiss canton/language-region values.
- `industries` — 24 employer-industry values.
- `categories` — all **165** jobs.ch occupational categories (job function, distinct from employer industry).
- `employmentTypes` — temporary, freelance, internship, supplementary income, permanent, apprenticeship.
- `positions` — executive, specialist, employee.
- `languageSkills` — German, English, French, Italian.
- `companySegments` — large company, SME, personnel-service provider.
- `location` — free-text city/place.
- `employmentGradeMin` / `employmentGradeMax` — workload range in percent.
- `publishedSince` — last 1, 3, 7 or 30 days.
- `sortBy` — relevance or newest-first (classic search).

Multiple values in one filter are combined with OR; different filters are combined with AND. Jobs.ch caps every search at **2,000 rows / 100 pages of 20**. Use narrower filters, date windows or several queries to get past the cap.

## Output fields

Every record includes the job identity and listing fields:

- `jobId`, `title`, `searchQuery` (when a keyword was used), `jobStatus` (URL mode only)
- `companyId`, `companyName`, `companySlug`, `companyUrl`, `companyLogoUrl`
- `location`, `locationDetails[]` (street, city, postal-code, canton, country, latitude, longitude)
- `workloadMin`, `workloadMax`, `workloadText`
- `contractType`, `contractTypes[]`, `seniorityLevel`
- `tags[]`, `benefitTags[]`, `easyApply`, `quickApply`, `homeOfficeOffered`
- `postedDate`, `initialPostedDate`, `postedRelative`, `initialPostedRelative`
- `isNew`, `isBoosted`, `isPaid`, `isActive`, `headhunterApplicationAllowed`
- `jobUrl`, `recordType`, `scrapedAt`

With `includeDetails: true`:

- `description` (plain text), `descriptionHtml` (original HTML), `descriptionSnippet`
- `applyUrl`, `applyMethod` (`external` or `form`), `applicationDeadline`
- `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`, `salaryIsEstimate`
- `contacts[]` (`name`, `firstName`, `lastName`, `role`, `phone`), plus `contactName` / `contactPhone`
- `languageSkills[]` (`language`, `level`), `skills[]`, `benefits[]`
- `category` (full taxonomy path), `categories[]`, `regions[]`, `cantonCode`
- `roleSummary`, `industry`, `workHoursText`, `totalOpenings`, `startDate`, `jobImmediateStart`
- `companyWebsite`, `companyOverview`, `companyDescription`
- `companySize`, `companySegment`, `companyIndustry`, `companyRating`, `companyReviewCount`,
  `companyOpenJobs`, `companySocialLinks[]`, `companyAddress`
- `applicantLocationRequirement`, `streetAddress`, `jobImageUrl`, `directApply`

Empty values are omitted at every depth. `0` and `false` are preserved when they are real values.

## Input reference

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | See modes above. |
| `searchMode` | select | `classic` | `classic` or `semantic`. |
| `searchQueries` | string list | `[]` | One or more keyword queries; deduplicated by job ID. |
| `companyNames` | string list | `[]` | Company names for `byCompany`. |
| `companyIds` | string list | `[]` | jobs.ch company IDs for `byCompany`. |
| `startUrls` | URL list | `[]` | jobs.ch search or job-detail URLs for `byUrls`. |
| `regions` | string list | `[]` | Region/canton IDs. |
| `industries` | string list | `[]` | Industry IDs. |
| `categories` | string list | `[]` | Occupational category IDs (all 165). |
| `employmentTypes` | string list | `[]` | Contract-type IDs. |
| `positions` | string list | `[]` | Seniority IDs. |
| `languageSkills` | string list | `[]` | `de`, `en`, `fr`, `it`. |
| `companySegments` | string list | `[]` | `gu`, `kmu`, `pdl`. |
| `location` | text | `""` | City/place refinement. |
| `employmentGradeMin` | integer | `0` | Minimum workload %. |
| `employmentGradeMax` | integer | `100` | Maximum workload %. |
| `publishedSince` | select | `""` | `1`, `3`, `7`, `30` days. |
| `sortBy` | select | `relevance` | `relevance` or `date`. |
| `includeDetails` | boolean | `false` | Fetch job detail and company profile. |
| `descriptionFormat` | select | `text` | `text`, `html` or `both`. |
| `maxDescriptionLength` | integer | `0` | Truncate descriptions; `0` = no limit. |
| `maxItems` | integer | `20` | 1–2,000. |
| `maxPages` | integer | `100` | 1–100 pages per query/filter combination. |
| `locale` | select | `en` | `en`, `de`, `fr`. |
| `incrementalMode` | boolean | `false` | Push only new/changed jobs. |
| `stateKey` | text | `""` | Key-value store key for incremental state. |

## Examples

### Classic keyword search

```json
{
  "mode": "search",
  "searchQueries": ["software engineer", "data engineer"],
  "regions": ["19"],
  "employmentGradeMin": 80,
  "maxItems": 100
}
```

### Semantic, cross-language search

```json
{
  "mode": "search",
  "searchMode": "semantic",
  "searchQueries": ["pflege"],
  "location": "Genève",
  "maxItems": 50
}
```

### Company openings with full profiles

```json
{
  "mode": "byCompany",
  "companyNames": ["Swiss Medical Network"],
  "includeDetails": true,
  "maxItems": 25
}
```

### Paste search and detail URLs

```json
{
  "mode": "byUrls",
  "startUrls": [
    {"url": "https://www.jobs.ch/en/vacancies/?term=pflege&region=19"},
    {"url": "https://www.jobs.ch/en/vacancies/detail/e5ce43b5-b97a-46cc-a783-93eff2bf064f/"}
  ],
  "includeDetails": true
}
```

### Browse by occupational field, newest first

```json
{
  "mode": "browse",
  "categories": ["175", "176"],
  "publishedSince": "7",
  "sortBy": "date",
  "maxItems": 200
}
```

## Use cases

- Recruiting and labour-market monitoring by canton, industry or occupational field.
- Company hiring-tracker dashboards built on stable `jobId` values.
- Job-board aggregation with full descriptions and apply links.
- Salary research using jobs.ch's own annual CHF estimates (always labelled as estimates).
- Incremental monitoring of new or changed postings.

## Data source

The actor reads jobs.ch's public search API, its server-rendered search pages for semantic mode, its company directory/profile API, and the job-detail pages' embedded React Query state plus schema.org `JobPosting` JSON-LD. All requests are anonymous, cookieless and direct.

## Limitations

- **2,000-row search ceiling.** jobs.ch returns at most 100 pages of 20 rows per search. Partition by filters, date windows or queries to collect more.
- **Salary is an estimate.** The salary fields come from jobs.ch's own salary-estimation endpoint, are annual CHF, and are marked `salaryIsEstimate: true`. They are not employer-disclosed salaries.
- **Contacts are partial.** Job-level contacts appear on a minority of postings (about 7 of 48 in testing); phone numbers are rarer and email addresses are not exposed.
- **Apply links are external only.** About three-quarters of sampled jobs had an external apply URL. The rest use jobs.ch's on-site form; for those, `applyMethod` is `form` and `jobUrl` is the apply entry point.
- **Company matching is best-effort.** Exact company names resolve first; otherwise prefix/contains matching is used. Company IDs are the reliable option.
- **Semantic search is fuzzy by design.** It mirrors jobs.ch's own relevance engine, does not support date sorting, and can return off-topic rows for an unmatched query.

## FAQ

**Do I need a proxy, cookie or API key?** No. Every request is anonymous and direct. Existing tasks that still send a proxy object keep running; the value is ignored.

**What is the difference between `industry` and `category`?** `industry` is the employer's business sector; `category` is the job's occupational function. A bank can post an IT-category job, so the two taxonomies are independent.

**Why did I get fewer rows than `maxItems`?** Either jobs.ch had no more matches, the 2,000-row cap was reached, or `maxPages` stopped the crawl. The run's status message says which.

**Why are some fields missing on some jobs?** jobs.ch does not include every field on every posting, and the actor omits empty values rather than inventing placeholders. Detail fields require `includeDetails: true`.

**Are the salary figures employer-disclosed?** No. They are jobs.ch's annual CHF estimate and are always marked `salaryIsEstimate: true`.

**How do I get past 2,000 rows?** Split the search into narrower slices — one region, category, company or publication window at a time — and combine the datasets. Deduplicate on `jobId`.

**Does semantic search support sorting?** No. jobs.ch's semantic results page is relevance-only. Use classic search with `sortBy: "date"` for newest-first ordering.

**What does incremental mode do?** With `incrementalMode: true`, the actor stores a signature per `jobId` under `stateKey` and pushes only new or changed records. Disable it for a full snapshot.

**Can I use job URLs from any jobs.ch language?** Yes. English, German and French detail/search URLs are accepted, and the record URLs follow the URL's language.

**Is this affiliated with jobs.ch?** No. It is an independent third-party actor that reads publicly available jobs.ch data.
