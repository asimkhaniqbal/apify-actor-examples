# LinkedIn People Finder by Name & Company (No Login)

Turn a list of names into LinkedIn profiles — with the current employer, location, headline, photo and followers — without a LinkedIn account, cookie or proxy. Give the actor names plus an optional company or location, and it returns the people who match.

## What this actor does

- Looks up each name in LinkedIn's public people directory and returns the matching profiles, up to 50 candidates per name.
- Narrows the list with strict **current company** and **location** filters: every candidate's public profile page is checked before a row is kept, so a person whose company only appears in a directory summary is verified or dropped.
- Opens each matched profile to fill the public details: current employer, location, followers, "500+" connection indicator, about text, education and profile photo.
- Understands full names, single names, middle names, and one-sided searches (`first:Madonna`, `last:Smith`).
- Removes duplicates, so the same profile is never returned twice in one run.
- Works with zero credentials and zero proxy setup. No LinkedIn login is used at any point.

## Output per profile

| Field | Description |
|---|---|
| `name` | The person's name as shown on their profile. |
| `profileUrl` | Canonical `https://www.linkedin.com/in/...` profile URL. |
| `headline` | Real headline when publicly visible or recoverable from an indexed public result (`headlineSource: "page"` or `"search_engine"`); omitted when LinkedIn hides it everywhere. |
| `currentTitle` | Job title parsed from a real headline, when available. |
| `location` | The location shown on the profile. |
| `currentCompany` | Current employer as `{name, slug, url}`. |
| `companyText` | The employer summary shown in the directory card, kept for traceability. |
| `companyRedacted` | `true` when LinkedIn hides the employer and would otherwise show the job title in its place. |
| `about` | The public About text (`aboutSource`: `page`, `dom` or `og`). |
| `aboutTruncated` | `true` when LinkedIn only exposes a truncated About excerpt to logged-out visitors. |
| `followersCount` | Follower count. |
| `connectionsText` | Public connection indicator, usually `"500+"`. |
| `currentSchool` | School shown in the profile's top card. |
| `education` / `educationCount` | Public education entries (profile data, with an `og:description` fallback) and their count. |
| `profilePicture` / `profileImageUrl` | Profile photo URL when the member has a real photo. |
| `searchName` | The input name that produced this row. |
| `discoveredVia` | `directory`, `directory_profile` (the directory page was the profile itself) or `search_engine`. |
| `searchEngine` | `yahoo` or `duckduckgo` when the row came from the search-engine fallback. |
| `scrapedAt` | ISO 8601 UTC timestamp. |

Empty fields are omitted, never emitted as `null`. The legacy keys `connectionDegree`, `verifiedStatus`, `openToWork`, `mutualConnections` and `commonInterests` remain part of the record shape but are not available without a logged-in session, so they do not appear.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `names` | List of strings | – | Names to look up. Full names, single names, or `first:` / `last:` for a one-sided search. Required. |
| `maxResultsPerName` | Integer (1–50) | `10` | Profiles to return per name. |
| `currentCompany` | String | – | Keep only people whose current employer contains this text. |
| `location` | String | – | Keep only people whose location contains this text. |
| `enrichProfiles` | Boolean | `true` | Open each matched profile for photo, about, followers, education and a verified employer. Off = directory cards only. |
| `includeHeadlinesFromSearchEngines` | Boolean | `true` | Best-effort extra matches and indexed headlines from public search engines, including a company-specific headline fallback derived from each profile. Also drives the company-augmented rescue query. |

### Example: name plus company

```json
{
  "names": ["Satya Nadella", "Sundar Pichai"],
  "currentCompany": "Microsoft",
  "maxResultsPerName": 5
}
```

### Example: location filter

```json
{
  "names": ["John Smith"],
  "location": "London",
  "enrichProfiles": true
}
```

### Example: fastest mode

```json
{
  "names": ["Ada Lovelace", "Grace Hopper"],
  "enrichProfiles": false,
  "maxResultsPerName": 10
}
```

## Use cases

- **Lead research** — turn a list of names into current employer, role and profile links.
- **Recruiting** — confirm a candidate's current company and location before outreach.
- **CRM cleanup** — refresh LinkedIn URLs and employers for existing contacts.
- **Name disambiguation** — use company or location to pick the right person among namesakes.

## FAQ

**Do I need a LinkedIn cookie or account?** No. The actor reads only LinkedIn's public directory and logged-out profile pages. It never asks for a login and does not use your account.

**Why is a person missing from the results?** LinkedIn's public directory shows at most 50 candidates per name. Very common names ("John Smith") can push the person you want outside that window. Add a company or location filter: the actor then also runs a company-augmented public search-engine query, which recovers many common-name cases, and verifies every candidate against its profile.

**Are the company and location filters exact?** They are case-insensitive text matches, verified against the profile the actor opens. They keep people whose current employer or location contains your text. They are not LinkedIn's own server-side filters.

**Why is the headline missing for some people?** LinkedIn hides most logged-out headlines and replaces them with asterisks or the current company. The actor returns a real headline when it is visible on the page or recoverable from an indexed public result (name, vanity and name+company query forms); otherwise the field is left out rather than guessed.

**Why do some rows have `companyRedacted: true`?** On some profiles LinkedIn hides the employer and puts the job title where the company should be. The actor detects that pattern, never reports the job title as a company, and marks the record instead.

**Why does the connection count say "500+"?** LinkedIn caps the public connection indicator at "500+". An exact number is not exposed to logged-out visitors.

**Can I get email addresses, skills or full work history?** No. Those fields require a logged-in LinkedIn session and are outside this actor's public-data scope.

**What happens when a name has no matches?** No row is emitted for that name; the run continues with the remaining names.

**Is this actor affiliated with LinkedIn?** No. It is an independent tool and is not affiliated with, endorsed by or sponsored by LinkedIn Corporation.

## Data source & limitations

The actor reads LinkedIn's public people directory (`/pub/dir/`) and the public, logged-out profile pages it links to. It selects its own access path automatically; no proxy configuration is needed.

Honest limits: the directory window is about 50 candidates per name; exact connections, skills, full experience history, education degrees and contact details are not public; and headline visibility depends on what LinkedIn and public search indexes expose to logged-out visitors. The search-engine fallback is best-effort and may pause itself when the engines rate-limit. About texts are often truncated for anonymous visitors (flagged with `aboutTruncated`).
