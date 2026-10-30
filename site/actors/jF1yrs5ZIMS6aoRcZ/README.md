# HelloWork France Job Listings & Salary Filter Scraper

HelloWork aggregates listings from thousands of French employers and recruitment agencies, and its search UI carries an unusually deep filter set — right down to CSR certifications and télétravail frequency. This actor reads those same search-results pages for you: search by keyword, browse by region/city or by sector/job function, and stack contract type, remote-work frequency, experience level, company type, CSR label, posted-within window, disclosed salary and search radius on top. No login or cookies required.

## What this actor does

- **Three modes:** `search` (keyword), `byLocation` (region/city browse), `bySector` (sector/job-function browse)
- **18 French regions** (including Guadeloupe, Martinique, Guyane, La Réunion, Mayotte), **26 industry sectors**, **34 job functions**, **6 experience-level bands**, **5 company types**, and **10 CSR/sustainability labels** as ready-to-use dropdowns
- **Filters:** contract type (CDI/CDD/Intérim/Stage/Alternance and more), remote-work (télétravail) frequency, employment schedule (full-time/part-time), experience level, company type, CSR label, posted-within window, minimum disclosed salary, search radius (mode=byLocation with `city`), disclosed-salary-only, super-recruiter-only — combinable across all modes
- **Structured salary parsing:** raw display text plus derived min/max/period/currency when the employer discloses a salary range
- **Empty fields are omitted**

## Output per job

- `offerId` — HelloWork internal offer ID
- `title` — job title
- `company` — hiring company name
- `companyLogoUrl` — company logo image URL
- `location` — free-text location (e.g. `Limonest - 69`)
- `department` — French department code, when derivable from `location` (e.g. `69`, `2A`)
- `contractType` — e.g. `CDI`, `CDD`, `Intérim`, `Stage`, `Alternance`
- `workTime` — employment schedule: `Temps Plein` (full-time) / `Temps Partiel` (part-time) / `Temps Occasionnel` (occasional), when disclosed
- `remoteWork` — remote-work (télétravail) frequency: `Télétravail Complet` (fully remote) / `Télétravail Partiel` (hybrid) / `Télétravail Occasionnel` (occasional remote), when disclosed on the listing card
- `salary` — raw disclosed salary text (e.g. `50 000 - 55 000 € / an`), when disclosed
- `salaryMin`, `salaryMax`, `salaryPeriod`, `salaryCurrency` — parsed salary amounts, when parseable
- `postedDate` — approximate UTC timestamp derived from HelloWork's relative posting text
- `postedDateRelative` — HelloWork's own relative posting text (e.g. `il y a 3 jours`)
- `hasApplicationTracking` — true when the recruiter shares application-progress tracking with candidates
- `isSuperRecruiter` — true for HelloWork-verified "super recruiter" employers
- `url` — canonical hellowork.com job detail page
- `recordType: "job"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` / `byLocation` / `bySector` |
| `searchQuery` | string | – | Free-text keyword (mode=search); combinable with all filters below |
| `region` | string | – | French administrative region (mode=byLocation, unless `city` is set) |
| `city` | string | – | French city/commune name (mode=byLocation, unless `region` is set); takes priority over `region` |
| `sector` | string | – | Industry sector (mode=bySector, unless `jobFunction` is set) |
| `jobFunction` | string | – | Job function / métier (mode=bySector, unless `sector` is set) |
| `contractTypes` | array | – | One or more contract types |
| `remoteWork` | array | – | Remote-work (télétravail) frequency: fully / hybrid / occasional / none disclosed |
| `employmentType` | array | – | Employment schedule: full-time / part-time / unspecified |
| `experienceLevels` | array | – | Years of experience the employer asks for |
| `companyTypes` | array | – | Direct employer / recruitment agency / IT services company / temp agency / training center |
| `csrLabels` | array | – | CSR / sustainability certification held by the employer |
| `minSalary` | int | `0` | Minimum disclosed annual salary floor in EUR (0–100,000, matches HelloWork's own slider) |
| `radiusKm` | string | `20` | Search radius around `city` in km (0/5/10/20/50); ignored when only `region` is set |
| `postedWithin` | string | `all` | Any time / last 24h / last 3 days / last week / last month |
| `sortBy` | string | `relevance` | `relevance` or `date` (newest first) |
| `salaryOnly` | boolean | `false` | Only offers that disclose a salary range |
| `superRecruiterOnly` | boolean | `false` | Only HelloWork-verified "super recruiter" employers |
| `maxItems` | int | `50` | Hard cap on emitted job listings (1–1000) |

### Example: keyword search

```json
{
  "mode": "search",
  "searchQuery": "developpeur",
  "sortBy": "date",
  "maxItems": 50
}
```

### Example: browse by region + contract type, full-time only

```json
{
  "mode": "byLocation",
  "region": "Bretagne",
  "contractTypes": ["CDI"],
  "employmentType": ["ft"],
  "postedWithin": "w",
  "maxItems": 100
}
```

### Example: browse by sector across France, salary disclosed only

```json
{
  "mode": "bySector",
  "sector": "Inform_SSII",
  "jobFunction": "Informatique_dev",
  "salaryOnly": true,
  "sortBy": "date",
  "maxItems": 100
}
```

### Example: browse by city with multiple contract types and a 10km radius

```json
{
  "mode": "byLocation",
  "city": "Lyon",
  "radiusKm": 10,
  "contractTypes": ["CDI", "CDD", "Alternance"],
  "maxItems": 60
}
```

## Use cases

- **Recruitment agencies** — monitor new listings in a target French region or sector
- **Labor market research** — track job posting volume and salary disclosure by sector over time
- **HR tech / job aggregators** — feed structured French job data into your own platform
- **Salary & demand analysis** — combine sector/function filters with disclosed-salary data to study hiring trends
- **Relocation & recruiting platforms** — surface opportunities in a specific French region or city
- **Remote-work / ESG sourcing** — combine `remoteWork` and `csrLabels` to find fully-remote roles at certified socially-responsible employers

## FAQ

**What is HelloWork?** One of France's largest job boards, aggregating listings from thousands of French employers and recruitment agencies. See [hellowork.com](https://www.hellowork.com).

**Is this affiliated with HelloWork?** No — this is an independent third-party actor that reads HelloWork's public search-results pages. No login, cookies, or paid API is required.

**Can I combine keyword search with a location or sector filter?** Yes — `region`/`city` and `sector`/`jobFunction` work as additional filters on `search` mode too, not just their own dedicated modes.

**What's the difference between `workTime` and `remoteWork`?** `workTime` is the employment *schedule* (full-time / part-time / occasional), shown on every card as "en temps plein/partiel/occasionnel". `remoteWork` is a separate, independent axis — how much of the job is done remotely ("Télétravail complet/partiel/occasionnel") — shown as its own optional clause only when the recruiter discloses it. A job can be full-time (`workTime`) AND hybrid-remote (`remoteWork`) at the same time; filter on whichever axis matches your need (`employmentType` input filters `workTime`, `remoteWork` input filters `remoteWork`).

**Why do some jobs have no `salary`?** Most French employers don't disclose a salary range publicly; the field is only populated when HelloWork shows one on the listing.

**Why do some jobs have no `department`?** `department` is only derived when `location` follows the standard "City - NN" pattern. Broader locations (a region name, "France", or a foreign country) don't have a single department to extract.

**Why do some jobs have no `remoteWork`?** The télétravail clause only appears on the card when the recruiter explicitly discloses a remote-work frequency; a missing value doesn't necessarily mean "no remote work" — it may simply be undisclosed at the search-results level.

**How fresh is the data?** HelloWork re-indexes continuously as employers post and remove listings; running the actor gives you the current live listing set for your filters.

**Does `maxItems` guarantee that many results?** No — it's a cap. If fewer jobs match your filters, you'll get fewer records.

**What does `postedDate` mean if HelloWork only shows relative text like "3 days ago"?** `postedDate` is an approximate UTC timestamp computed from that relative text at scrape time; `postedDateRelative` preserves HelloWork's original wording.

**Why do `experienceLevels`, `companyTypes`, and `csrLabels` filter results but not appear as output fields?** HelloWork only shows these attributes on each job's individual detail page, not on the search-results card the actor reads. The filters still narrow which jobs are returned — just visit a result's `url` to see the matching attribute on the live listing.

**Does `radiusKm` work with a `region` filter?** No — the radius only applies around a specific `city`. A region has no single center point to measure a radius from, so `radiusKm` is ignored unless `city` is set.

**Does `minSalary` exclude jobs that don't disclose a salary?** No — it only excludes jobs that disclose a salary *below* the floor. Jobs with no disclosed salary still pass through. Combine with `salaryOnly: true` if you want disclosed-only, above-floor results.

**Does this actor fetch each job's full description page?** No — all output fields come from the search-results listing cards, which keeps runs fast. Fields only shown on a job's detail page (full description text, required diploma, application instructions) are not included; use the `url` field to open the full listing.
