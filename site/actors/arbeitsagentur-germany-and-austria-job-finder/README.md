# Arbeitsagentur Germany & Austria Job Finder

Arbeitsagentur.de is the online job board of Germany's federal employment agency (Bundesagentur für Arbeit), carrying one of the largest pools of German- and Austrian-market job postings. This actor searches that listing index by keyword and location, returning a fixed, fully-typed set of fields for every job so downstream tools never have to guess which keys exist.

## What this actor does

- Searches current job postings by keyword, with an optional location and search radius.
- Filters by contract type (permanent/fixed-term), working-time arrangement (full-time, part-time, shift, home office, minijob), and how recently a job was published.
- Returns geocoordinates and distance-from-search for each job location, useful for mapping or radius analysis.
- Includes Austrian postings alongside German ones, since Arbeitsagentur cooperates with Austria's AMS employment service.
- Every record always carries the same 30 fields; when Arbeitsagentur has no data for a field, it comes back as an empty string, `0`, or `false` instead of being left out — so no field is ever missing from a record, only empty.

## Output per job

### Identity
| Field | Description |
| --- | --- |
| `id` | Arbeitsagentur reference number (Refnr) for the posting. |
| `title` | Job title. |
| `occupation` | Primary occupation classification (Beruf). |
| `occupations` | All occupation classifications listed for the role. |
| `employer` | Company or organization name. |

### Location
| Field | Description |
| --- | --- |
| `city` | Job city. |
| `postalCode` | Postal code. |
| `street` | Street address, when disclosed. |
| `region` | Federal state (Bundesland). |
| `country` | Country — Deutschland or Österreich. |
| `latitude` / `longitude` | Coordinates of the job location (`0` when Arbeitsagentur doesn't publish a location for the posting). |
| `distanceKm` | Distance from the searched location, in kilometers (`0` when no `location` was searched or no distance was returned). |

### Dates
| Field | Description |
| --- | --- |
| `publishedDate` | Date the posting first went live. |
| `modifiedTimestamp` | When the posting was last updated. |
| `startDate` | Intended job start date. |

### Employment details
| Field | Description |
| --- | --- |
| `contractDuration` | Permanent, fixed-term, or not specified. |
| `salary` | Annual gross salary in EUR, when the employer discloses it (`0` otherwise). |
| `salaryType` | How the salary is expressed (e.g. annual figure, not specified). |
| `workingTimeFullTime` / `workingTimePartTime` / `workingTimeShift` | Working-time arrangement flags. |
| `homeOfficePossible` | Whether home office is possible. |
| `homeOfficeType` | Home office arrangement detail, when specified. |
| `isMinijob` | Whether the role is a minijob (marginal employment). |
| `quereinstiegSuitable` | Whether the role is marked suitable for career changers. |
| `jobType` | Offer type as classified by Arbeitsagentur. |

### Links & metadata
| Field | Description |
| --- | --- |
| `externalUrl` | The employer's own apply link, when the posting provides one. |
| `portalUrl` | Link to the posting on the Arbeitsagentur job portal. |
| `scrapedAt` | ISO 8601 timestamp of when the record was collected. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `search` | String | – | Keyword to search for (matched against title and description), e.g. "Softwareentwickler". |
| `location` | String | – | City, postal code, or region. Leave empty to search all of Germany. |
| `radius` | Integer | 25 | Search radius in kilometers around `location` (only used when `location` is set). |
| `contractType` | String (select) | `all` | `all`, `permanent`, `fixed-term`, or `any` contract. |
| `workingTime` | String (select) | `all` | `all`, `full-time`, `part-time`, `shift`, `homework` (home office), or `minijob`. |
| `publishedSince` | Integer | 0 | Only include jobs published within the last N days (e.g. 1, 3, 7, 14, 28). `0` returns all. |
| `maxItems` | Integer | 50 | Maximum number of jobs to return. |

### Example: software developer jobs in Berlin

```json
{
  "search": "Softwareentwickler",
  "location": "Berlin",
  "radius": 25,
  "contractType": "permanent",
  "workingTime": "full-time",
  "maxItems": 100
}
```

### Example: recent nursing jobs nationwide

```json
{
  "search": "Krankenpfleger",
  "publishedSince": 7,
  "maxItems": 200
}
```

### Example: minijobs near a postal code

```json
{
  "search": "Lagerhelfer",
  "location": "10115",
  "radius": 15,
  "workingTime": "minijob"
}
```

## Use cases

- **Recruiters and sourcers** scanning for newly published openings in a specific occupation or region.
- **Labor-market researchers** tracking contract type and working-time mix (full-time vs. part-time vs. minijob) by region.
- **Salary benchmarking teams** aggregating disclosed salary figures by occupation.
- **Relocation services** mapping job density and distance from a target city using the geocoordinates.
- **Career coaches** compiling relevant, recently-published openings for a job seeker's search terms.

## FAQ

**Do I need an Arbeitsagentur account to use this?**
No. It reads the same public job search index that arbeitsagentur.de serves to any visitor.

**Why do some jobs show `salary` as `0`?**
Most employers don't disclose a salary figure on Arbeitsagentur. `0` means "not disclosed", not that the job is unpaid.

**Why are fields like `street` or `homeOfficeType` sometimes empty strings?**
Every record always contains the same fixed set of fields. When Arbeitsagentur doesn't have a value for a field on a given posting, it's returned as an empty string, `0`, or `false` rather than omitted.

**Are Austrian jobs mixed in with German ones?**
Yes. Arbeitsagentur's index includes postings submitted through Austria's AMS employment service; check the `country` field to tell them apart.

**What does `publishedSince` actually filter on?**
It limits results to jobs whose first publication date falls within the chosen number of days before the run. Setting it to `0` disables the filter.

**How fresh are the results?**
Each run searches the live Arbeitsagentur listing index at the time it runs, so results reflect whatever is currently posted.

**Is this affiliated with the Bundesagentur für Arbeit?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by the Bundesagentur für Arbeit or Arbeitsagentur.de.
