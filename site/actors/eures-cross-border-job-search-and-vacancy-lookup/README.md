# EURES Cross-Border Job Search & Vacancy Lookup

EURES is the European Union's official portal for cross-border employment, aggregating job vacancies posted by national employment services across every EU member state plus Iceland, Liechtenstein, Norway and Switzerland. This actor searches that live vacancy index by keyword and more than a dozen filters, or pulls full detail — salary, benefits, contact info, application link — for specific vacancy IDs you've already found.

## What this actor does

- **Two modes**: `search` (keyword + filters over the whole index) and `byIds` (full detail for vacancy IDs you supply)
- **Geographic filtering** at two levels: 31 EEA country codes, and finer-grained EU NUTS region codes that can be combined with country filters
- **More than a dozen filters**: business sector (NACE), contract type, work schedule, education level, required experience, required languages, ESCO occupation/skill codes, advertised benefits, and minimum open-post count
- **Detail-mode enrichment**: application links and deadlines, salary figures, named benefits, remote-work and travel flags, and recruiter contact details when the employer discloses them
- **Empty fields are omitted** — a listing only carries the fields EURES actually published for it

## Output per job vacancy

- `jobId` — EURES vacancy identifier, usable with `mode=byIds`
- `title`, `description` — job title and full description text
- `publicationDate`, `lastModifiedDate` — when first posted and when EURES's index last synced it
- `reference`, `documentId` — the source employment service's own reference/document numbers
- `numberOfPosts` — how many open posts this single vacancy covers
- `euresFlag` — whether the employer explicitly welcomes cross-border EEA candidates
- `countries[]`, `countryNames[]` — job location country code(s) and name(s)
- `nutsRegionCodes[]` — EU NUTS statistical region code(s) covering the job location, feedable back into the `regionCodes` input
- `locations[]`, `postalCodes[]` — location names and postal codes (`mode=byIds`)
- `positionOfferingCode` — contract type (e.g. `directhire`, `temporary`, `internship`)
- `positionScheduleCodes[]` — `fulltime` / `parttime` / `flextime`
- `employerName`, `employerWebsite`, `employerDescription`, `employerSectorCodes[]` — employer identity and NACE sector(s)
- `employerLegalId`, `employerSizeCode` — employer registration ID and organisation-size band, when disclosed (`mode=byIds`)
- `occupationUris[]` — ESCO occupation classification URIs
- `requiredSkills[]` — ESCO skill URIs required for the role (`mode=byIds`)
- `requiredYearsOfExperience`, `requiredExperienceDescription` — numeric or free-text experience requirement
- `requiredEducationLevelCode` — ISCED-based required education level
- `availableLanguages[]` — language(s) the listing is available in (`mode=search`)
- `workingLanguages[]`, `requiredLanguageSkills[]` — working language(s) and per-language proficiency requirements (`mode=byIds`)
- `drivingLicensesRequired[]` — required driving license categories, when specified
- `employmentPeriod`, `employmentStartDate`, `employmentEndDate` — contract duration and dates, when fixed-term
- `applicationUrl`, `applicationInstructions`, `applicationDeadline` — how and by when to apply (`mode=byIds`)
- `remoteWorkAllowed`, `immediateStart` — flags the employer explicitly set, when disclosed
- `travelRequired`, `travelPercentage`, `travelDescription` — travel expectations, when the employer specified them
- `salaryMin`, `salaryMax`, `salaryReference`, `salaryCurrency`, `salaryPeriod`, `salaryDescription` — pay figures and notes, only when the employer discloses pay
- `benefits[]`, `benefitsDescription` — named extra benefits (accommodation, meals, relocation, insurance types, etc.) and free-text perks, when disclosed
- `contactName`, `contactEmail`, `contactPhone`, `contactAddress` — recruiter contact details, when published (`mode=byIds`)
- `contentLanguage` — the language the returned `title`/`description` text is actually written in
- `source`, `sourceUrl` — originating employment service and the canonical EURES vacancy page URL
- `scrapedAt` — ISO 8601 timestamp of the run

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` or `byIds` |
| `searchQuery` | string | `developer` | Free-text keyword (mode=search) |
| `searchIn` | string | `EVERYWHERE` | `EVERYWHERE` / `TITLE` / `DESCRIPTION` / `EMPLOYER` — where the keyword is matched |
| `jobIds` | array | – | Vacancy IDs to fetch (mode=byIds) |
| `countries` | array | – | One or more of the 31 EEA country codes |
| `regionCodes` | array | – | EU NUTS region codes; combines with `countries` as a union |
| `sectors` | array | – | NACE Rev.2 business sector codes |
| `positionOfferingCodes` | array | – | Contract types (direct hire, temporary, internship, etc.) |
| `positionScheduleCodes` | array | – | `fulltime` / `parttime` / `flextime` |
| `educationLevels` | array | – | Required education level (basic to doctoral) |
| `experienceLevels` | array | – | Required-experience band |
| `languageSkills` | array | – | Required working language(s) |
| `occupationUris` | array | – | ESCO occupation URIs (from a prior result's `occupationUris`) |
| `skillUris` | array | – | ESCO skill URIs (from a prior `byIds` result's `requiredSkills`) |
| `benefitCodes` | array | – | Advertised extra-benefit codes |
| `euresFlagOnly` | boolean | `false` | Only cross-border-flagged vacancies |
| `publicationPeriod` | string | `` (any time) | `LAST_DAY` / `LAST_THREE_DAYS` / `LAST_WEEK` / `LAST_MONTH` |
| `minNumberOfPosts` | integer | – | Only vacancies with at least this many open posts |
| `sortBy` | string | `BEST_MATCH` | `BEST_MATCH` or `MOST_RECENT` (keyword searches always sort by relevance) |
| `maxItems` | integer | `50` | Hard cap on returned records (1–2000) |

### Example: keyword search in one country

```json
{
  "mode": "search",
  "searchQuery": "warehouse",
  "countries": ["de"],
  "maxItems": 100
}
```

### Example: bulk-hiring full-time roles in a region, posted recently

```json
{
  "mode": "search",
  "searchQuery": "",
  "regionCodes": ["deb"],
  "positionScheduleCodes": ["fulltime"],
  "minNumberOfPosts": 5,
  "publicationPeriod": "LAST_WEEK",
  "maxItems": 200
}
```

### Example: full detail for known vacancy IDs

```json
{
  "mode": "byIds",
  "jobIds": ["MzQwNDYwIDU"]
}
```

### Example: nurse roles requiring specific experience and languages

```json
{
  "mode": "search",
  "searchQuery": "nurse",
  "sectors": ["q"],
  "experienceLevels": ["between_1_and_2_years"],
  "languageSkills": ["de", "en"],
  "maxItems": 150
}
```

## Use cases

- **Cross-border recruitment agencies** — source candidates by sector, language and region across the EEA in one search
- **Labour-market research** — track vacancy volume and contract-type mix by country or NACE sector over time
- **HR tech integrations** — feed EURES vacancies into a job-aggregator or ATS pipeline by ESCO occupation
- **Relocation and mobility platforms** — filter for EURES-flagged jobs that explicitly welcome candidates relocating from another EEA country
- **Public-policy and academic research** — analyse posted salary, benefit and experience requirements across the single market

## FAQ

**Do I need an account or API key for EURES?**
No. EURES's job search is public; this actor queries the same public search index that powers the EURES website.

**How large is the EURES vacancy index?**
It typically holds well over a million active postings aggregated from national employment services across 31 EEA countries at any given time. Use filters and `maxItems` to target a workable slice.

**What's the difference between `search` and `byIds`?**
`search` runs a keyword/filter query and returns a page of matching vacancies with the fields EURES's listing index exposes. `byIds` fetches the full detail page for specific vacancy IDs — collect `jobId` values from a `search` run first, then feed them into a `byIds` run to get salary, benefits, contacts and the application link.

**Why do some records have far fewer fields than others?**
EURES aggregates postings from dozens of national employment services, each publishing a different level of detail. A field like `salaryMin` or `contactEmail` only appears when the originating service actually disclosed it — there's no fallback value.

**Can I filter below the country level?**
Yes — `regionCodes` accepts EU NUTS region codes (e.g. `deb` for Berlin), which you can find in the `nutsRegionCodes` field of any prior result. It combines with `countries` as a union, not an intersection.

**What does `sortBy=MOST_RECENT` actually sort on?**
It sorts by `lastModifiedDate`, the timestamp of the vacancy's last sync into EURES's index — not always the same as `publicationDate`, since a national service occasionally re-syncs an unchanged listing. This sort order is ignored whenever `searchQuery` is set, because keyword matches are always ranked by relevance.

**Will every vacancy include salary or benefits?**
No. Most EURES postings don't disclose pay or extra benefits at all. Setting `benefitCodes` narrows results to only vacancies that explicitly advertise one of the benefits you choose.

**Is this affiliated with EURES or the European Union?**
No. This is an independent, third-party actor built against EURES's own public job-search index; it is not operated by or affiliated with EURES or the European Commission.
