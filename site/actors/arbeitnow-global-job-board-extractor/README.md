# Arbeitnow Global Job Board Extractor

Arbeitnow lists remote, full-time, part-time and contract roles posted by companies worldwide, with a strong lean toward tech and visa-friendly positions. This actor reads the live Arbeitnow job feed and returns each listing as a structured record you can filter by keyword, category tag or remote status.

## What this actor does

- Pulls current job listings from Arbeitnow's public job feed.
- Filters by a free-text keyword matched against the job title and description.
- Filters by one or more category/tech tags, requiring a listing to match all of them.
- Optionally restricts results to remote-friendly roles only.
- Fields with no data for a given listing (for example a missing job type) are omitted from that record.

## Output per job

| Field | Description |
| --- | --- |
| `slug` | Unique job identifier on Arbeitnow. |
| `title` | Job title as posted by the employer. |
| `company` | Name of the hiring company. |
| `location` | Geographic location of the role (e.g. "Berlin, Germany"). |
| `remote` | Whether the position is remote-friendly. |
| `jobTypes` | Employment types for the role (e.g. "Full-time", "Contract"). |
| `tags` | Tech-stack and category tags attached to the listing. |
| `description` | Plain-text job description with HTML markup removed. |
| `postedAt` | ISO 8601 datetime the job was posted. |
| `sourceUrl` | Direct link to the listing on arbeitnow.com. |
| `scrapedAt` | ISO 8601 datetime when this record was collected. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `searchQuery` | String | – | Keyword matched case-insensitively against the job title and description. Leave blank to return all jobs. |
| `tags` | Array of strings | – | One or more category tags a job must match (e.g. `["Software Development", "IT"]`). A job must match every tag listed. |
| `remote` | Boolean | `false` | When enabled, only remote-friendly jobs are returned. |
| `maxItems` | Integer | 10 | Maximum number of job listings to return (1–500). |

### Example: remote Python roles

```json
{
  "searchQuery": "python",
  "remote": true,
  "maxItems": 50
}
```

### Example: software development jobs by category tag

```json
{
  "tags": ["Software Development"],
  "maxItems": 100
}
```

### Example: all current listings, unfiltered

```json
{
  "maxItems": 20
}
```

## Use cases

- **Job seekers** building a personal feed of remote or tag-matched openings without visiting the site daily.
- **Recruiters and sourcers** pulling fresh listings to cross-reference against their own candidate pipelines.
- **Market researchers** tracking which categories and tech tags are most active on a given day.
- **Job board aggregators** mirroring Arbeitnow postings into their own listing feed.
- **HR analysts** monitoring remote-vs-onsite and employment-type mix across incoming postings.

## FAQ

**Do I need an Arbeitnow account or API key?**
No. The feed this actor reads is public and requires no credentials.

**How does `searchQuery` matching work?**
It's a case-insensitive substring match against each job's title and description, so "engineer" matches both "Senior Engineer" roles and listings that mention "engineer" in the body text.

**What exactly do `tags` filter on?**
Tags are the category/tech labels Arbeitnow itself attaches to a listing (e.g. "Software Development", "python"), not free-text keywords. When you list more than one tag, a job must carry all of them to be returned.

**Why is a listing missing `jobTypes` or `tags`?**
Not every posting on Arbeitnow specifies an employment type or tags; when that data isn't present on the source listing, the field is left out rather than filled in with a guess.

**How current are the results?**
Each run reads Arbeitnow's live listing feed at the time it runs, so results reflect whatever is currently posted.

**Is this affiliated with Arbeitnow?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Arbeitnow.
