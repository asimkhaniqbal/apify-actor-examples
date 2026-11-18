# EU Procurement Notice Finder by CPV & Country

TED (Tenders Electronic Daily) is the EU's official publication for public procurement notices — hundreds of thousands of calls for tender from public buyers across Europe. This actor searches TED by CPV category, buyer country, publication date, and keyword, or fetches specific notices directly by their publication number.

## What this actor does

- Searches TED notices by CPV category (and an optional exact CPV code override), buyer country, a publication-date range, and a free-text keyword.
- Fetches one or more specific notices directly by publication number.
- Scopes a search to notices still open for submission, the latest version of each notice, or every notice and version in the archive.
- Returns title, buyer, CPV codes, deadline, contract value, and links to the notice's HTML, XML, and PDF versions.
- Leaves a field out of a record whenever TED doesn't publish that value for the notice, rather than sending it empty.

## Output per notice

| Field | Description |
|---|---|
| `publicationNumber` | TED publication number, e.g. `463590-2022` |
| `title` | Notice title |
| `buyerName` | Contracting authority or buyer name |
| `buyerCountry`, `buyerCountryName` | Buyer country code and name |
| `placeOfPerformance` | Country/region codes where the contract will be performed |
| `cpvCodes`, `cpvMain` | All CPV classification codes on the notice, and the primary one |
| `noticeType` | TED notice type, e.g. `cn-standard`, `can-standard` |
| `formType` | Form type, e.g. competition, result, planning |
| `contractNature` | Contract nature: works, supplies, or services |
| `procedureType` | Procurement procedure type, e.g. open, restricted |
| `publicationDate` | When the notice was published |
| `deadline` | Earliest tender submission deadline, when published |
| `value`, `valueCurrency` | Total contract value and its currency, when published |
| `htmlUrl` | Direct HTML page of the notice |
| `xmlUrl` | Machine-readable XML version |
| `pdfUrl` | PDF version |
| `sourceUrl` | Canonical TED notice detail page |
| `recordType` | Always `notice` |
| `scrapedAt` | UTC timestamp of the scrape |

## Output when a request fails

If a search query is rejected by TED, a fetch fails outright, or `getNotice` is called without a valid `publicationNumber`, the actor pushes an error record instead of silently returning nothing:

| Field | Description |
|---|---|
| `recordType` | Always `error` |
| `error` | Short error code, e.g. `query_error`, `request_failed`, `missing_input` |
| `message` | Human-readable explanation |
| `input` | The query or input value that triggered the error, when applicable |
| `sourceUrl` | The TED endpoint that was called |
| `scrapedAt` | UTC timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search` (filter-based) or `getNotice` (by publication number). |
| `cpvCategory` | string | – (any) | CPV top-level division, e.g. `45` for construction, `72` for IT services. Matches the division and all its sub-codes. |
| `cpvCode` | string | – | Exact or partial CPV code override, e.g. `45233140`, combined with `cpvCategory`. |
| `country` | string | – (all) | Buyer country, as an ISO-3166 alpha-3 code. |
| `keyword` | string | – | Full-text keyword to match anywhere in the notice. |
| `publicationDateFrom` | string | – | Earliest publication date to include, `YYYY-MM-DD`. |
| `publicationDateTo` | string | – | Latest publication date to include, `YYYY-MM-DD`. |
| `scope` | string | `ACTIVE` | `ACTIVE` (open for submission), `LATEST` (latest version of each notice), or `ALL` (every notice and version). |
| `publicationNumber` | string | – | One or more TED publication numbers to fetch, used in `getNotice` mode. |
| `maxItems` | integer | `50` | Maximum number of notice records to return. |

### Example: open construction tenders in Germany

```json
{
  "mode": "search",
  "cpvCategory": "45",
  "country": "DEU",
  "scope": "ACTIVE",
  "maxItems": 50
}
```

### Example: keyword search across a date range

```json
{
  "mode": "search",
  "keyword": "hospital equipment",
  "publicationDateFrom": "2026-01-01",
  "publicationDateTo": "2026-03-31",
  "maxItems": 100
}
```

### Example: fetch specific notices

```json
{
  "mode": "getNotice",
  "publicationNumber": "463590-2022, 471201-2022"
}
```

## Use cases

- **Bid discovery** — find open tenders in your sector and country before the submission deadline.
- **Market intelligence** — track which buyers are procuring what, where, and at what value.
- **Competitor monitoring** — watch contract award notices and framework agreements by CPV category.
- **Procurement research** — build datasets of public spending across Europe by category and time period.
- **Lead generation** — surface contracting authorities active in a specific sector.

## FAQ

**Do I need an account or API key?**
No. TED's public data is open and doesn't require login, a key, or cookies.

**What's a CPV code, and how does `cpvCategory` relate to `cpvCode`?**
CPV (Common Procurement Vocabulary) is the EU's standard classification for public contracts. `cpvCategory` matches a whole top-level division plus all its sub-codes. `cpvCode` adds another exact/partial CPV code to match against — when both are set, a notice matching either one is returned, so use `cpvCode` on its own for the tightest match on a specific code.

**Which countries are covered?**
The EU member states, EEA/EFTA countries (Iceland, Liechtenstein, Norway, Switzerland), the United Kingdom, and other European countries that publish notices on TED.

**What's the difference between the three `scope` options?**
`ACTIVE` returns only notices still open for submission. `LATEST` returns the newest version of each matching notice, including ones no longer open. `ALL` returns every notice and every version in the archive, which can include duplicates across amendments.

**How many notices can one run return?**
Up to `maxItems`, capped at 1000, retrieved through pagination. Narrowing with CPV, country, date, and keyword filters keeps results focused.

**Why would `value` be missing from a notice?**
Not every notice publishes a contract value — some notice types (like calls for competition before an award) don't have one yet.

**How fresh is the data?**
Each run queries TED's live data at request time, so results reflect what's currently published.

**What happens if my query is invalid or a request fails?**
The actor pushes a single error record describing what went wrong (`recordType: "error"`) instead of failing silently or returning nothing at all.

**Is this affiliated with TED or the EU?**
No. This actor reads TED's public data and is not affiliated with, endorsed by, or connected to the Publications Office of the European Union.
