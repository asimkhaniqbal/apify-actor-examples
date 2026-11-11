# Lever Job Board Scraper — Global & EU Boards

Lever hosts the public job boards used by thousands of companies, from early-stage startups to public technology firms. This actor reads those boards directly and returns open roles with their full description, location list, workplace type, country, employment type, salary data when the employer publishes it, and direct apply link. It accepts bare company slugs as well as board, API and posting URLs, and finds EU-only boards automatically.

## What this actor does

- **Reads global and EU Lever boards** from a company slug, `jobs.lever.co` URL, `api.lever.co` URL or a single posting URL
- **Auto-detects the EU instance** when the global board is missing or empty
- **Filters every posting client-side** by team, department, location, commitment, workplace type, remote-only, country, title words, posting date and salary
- **Returns the complete posting**, including all locations, workplace type, country, normalized employment type, salary range and readable salary text, description sections and requirement lists
- **Supports monitoring** with an only-new-since-last-run option that skips postings seen by a previous run
- **Never emits empty fields** — a field is present only when Lever published a real value

## Output per posting

- `id` — Lever posting UUID
- `title` — job title
- `companySlug` — board slug from Lever
- `companyName` — display name derived from the slug (a readable guess; Lever's API does not publish the official company name)
- `boardRegion` — `global` or `eu`
- `boardUrl` — the company's Lever board URL for that region
- `categories` — nested `commitment`, `location`, `team` and `department`
- `commitment`, `department`, `team`, `location` — the same values as top-level fields
- `locations[]` — every location on the posting; falls back to the primary location
- `workplaceType` — `remote`, `hybrid`, `onsite` or `unspecified`
- `isRemote` — true when the posting is remote or has a remote location
- `country` — ISO 3166-1 alpha-2 country code
- `employmentType` — normalized from the raw commitment: `full-time`, `part-time`, `contract`, `internship`, `temporary` or `other`
- `salaryRange` — structured `min`, `max`, `currency` and `interval` when published
- `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryInterval`, `salaryText` — flattened salary values and a readable range such as `USD 75,000–85,000 per year`
- `salaryDescriptionPlain` — salary text when the employer wrote one but did not publish a structured range
- `descriptionPlain` — combined plain-text description
- `openingPlain`, `descriptionBodyPlain`, `additionalPlain` — separate description sections
- `additional` — closing or benefits text (same as `additionalPlain`)
- `lists[]` — requirement/responsibility groups as `{text, content}`
- `descriptionHtml`, `additionalHtml`, `lists[].contentHtml` — original HTML when `includeHtml` is enabled
- `applyUrl`, `hostedUrl`, `sourceUrl` — application link and public posting links
- `createdAt` — ISO 8601 UTC posting timestamp
- `recordType: "job"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `getJobs` | `getJobs` for whole boards, or `getJob` for specific postings |
| `companySlugs` | array | – | Slugs, board URLs, API URLs or posting URLs (mode=`getJobs`) |
| `companySlug` | string | – | Company slug, board URL or posting URL (mode=`getJob`) |
| `postingIds` | array | – | Posting UUIDs or posting/apply URLs (mode=`getJob`) |
| `boardRegion` | string | `auto` | `auto`, `global` or `eu`; auto falls back to EU on an empty or missing global board |
| `team` | string | – | Team contains this text |
| `department` | string | – | Department contains this text |
| `location` | string | – | Any location on the posting contains this text |
| `commitment` | string | – | Commitment text or normalized employment type matches |
| `workplaceType` | string | – | `remote`, `hybrid`, `onsite` or `unspecified` |
| `remoteOnly` | boolean | `false` | Keep remote postings only |
| `country` | string | – | ISO alpha-2 code, e.g. `US` or `DE` |
| `titleKeywords` | array | – | Every word must appear in the title |
| `excludeTitleKeywords` | array | – | Drop titles containing any of these words |
| `postedAfter` | string | – | Keep postings created on or after `YYYY-MM-DD` |
| `hasSalaryOnly` | boolean | `false` | Keep postings with a structured salary range |
| `salaryMin` | integer | – | Minimum structured salary value |
| `maxItems` | integer | `500` | Global record cap (1–10,000) |
| `maxItemsPerCompany` | integer | `0` | Per-board cap; `0` means no limit |
| `includeDescription` | boolean | `true` | Include description text and lists |
| `includeHtml` | boolean | `false` | Include original HTML fields |
| `onlyNewSinceLastRun` | boolean | `false` | Emit only postings unseen by a previous matching run |

## Examples

### Every open role from several companies

```json
{
  "mode": "getJobs",
  "companySlugs": ["wealthfront", "cologix", "runlaminar"],
  "maxItems": 500
}
```

### Board and posting URLs, including an EU-only company

```json
{
  "mode": "getJobs",
  "companySlugs": [
    "https://jobs.lever.co/gohighlevel",
    "https://jobs.eu.lever.co/olx",
    "https://jobs.lever.co/wealthfront/78d6f6d5-2c6b-4f7f-9f0e-000000000000"
  ]
}
```

### Remote engineering roles posted this year

```json
{
  "mode": "getJobs",
  "companySlugs": ["agile-defense", "contentsquare"],
  "team": "Engineering",
  "remoteOnly": true,
  "titleKeywords": ["engineer"],
  "postedAfter": "2026-01-01",
  "hasSalaryOnly": true
}
```

### Look up specific postings by URL

```json
{
  "mode": "getJob",
  "companySlug": "wealthfront",
  "postingIds": [
    "https://jobs.lever.co/wealthfront/78d6f6d5-2c6b-4f7f-9f0e-000000000000",
    "https://jobs.lever.co/wealthfront/78d6f6d5-2c6b-4f7f-9f0e-111111111111/apply"
  ]
}
```

## Use cases

- **Job aggregation** — feed a niche job board with complete postings from selected Lever employers
- **Competitive hiring intelligence** — see which teams and locations a competitor is growing
- **Recruiting outreach** — export remote engineering roles with published salary ranges
- **Market research** — compare workplace arrangements, countries and salary ranges across companies
- **Scheduled monitoring** — run daily with only-new mode to capture fresh postings for alerts

## FAQ

### Do I need a Lever account, API key or cookies?

No. Lever's public postings API is open, and this actor reads only public board data.

### Do I need to configure a proxy?

No. The actor connects to Lever directly and needs no proxy setup.

### How do I find a company's Lever slug?

Open the company's careers page. If it is hosted by Lever, the URL looks like `https://jobs.lever.co/company-slug`; the slug is the last path segment. The actor also accepts the whole URL.

### Why do some postings have no salary fields?

Lever only returns salary data when the employer chooses to publish it, either as a structured range or as salary text. The actor never estimates or fabricates a range, so salary fields are simply absent on postings without published pay.

### What is the difference between `commitment` and `employmentType`?

`commitment` is the employer's raw text, which can vary (`Full-Time`, `Full time`, `EE Full-Time`, `Contract`). `employmentType` is a normalized value for filtering and analysis while the raw value is kept for traceability.

### Why is `companyName` sometimes not the official brand name?

Lever's API does not return a display name, so the actor derives one from the company slug. Treat it as a readable label rather than an authoritative company name.

### How does auto region detection work?

The actor queries the global Lever host first. If the board is missing there, or returns no postings but the company uses Lever's EU host, the actor retries on `api.eu.lever.co` and labels those records `boardRegion: "eu"`. An EU board URL goes straight to the EU host.

### Can I search every Lever company at once?

No. Lever has no public cross-company search endpoint; boards are read per company. Add as many company slugs or URLs as you need to one run.

### Does the actor report closed jobs?

No. Lever only exposes currently published postings, so a posting disappears when it is closed and there is no closed-job or update timestamp. Only-new monitoring reports postings that were not seen on an earlier run.

### Is this actor affiliated with Lever?

No. It is an independent third-party actor that reads Lever's public job board data and is not endorsed by or affiliated with Lever Inc.
