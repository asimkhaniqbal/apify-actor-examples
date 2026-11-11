# Multi-Site Username Lookup & Profile Finder

Find where a username exists across 400+ social networks, developer platforms, forums, gaming sites and marketplaces, and get a real profile URL instead of a guess. Every result carries a per-site status, HTTP code, detection method and verification flag, so a soft-404 page can never masquerade as a profile.

## What this actor does

- Checks up to **100 usernames per run** with concurrent site checks.
- Uses **22 native probes** with exact API/XML/HTML detection for GitHub, GitLab, Steam, Chess.com, Lichess, Hacker News, Codeforces, Docker Hub, Roblox, YouTube, Mastodon, Dev.to, SoundCloud, Instagram, Pinterest, Twitch, X, LeetCode, Keybase, Stack Exchange, Dailymotion and Kick.
- Runs **decoy verification** on the generic long tail: a site that claims a random handle is marked `unverified` and kept out of `links`.
- Exposes site, category, status, HTTP status, detection method and response time for every check, plus an NSFW flag on adult-site rows.
- Extracts profile fields where the site exposes them: display name, bio, avatar, followers, following, posts, join date, location, website and verified badge.
- Hides **NSFW sites by default** and removes ten sites proved unable to distinguish real profiles from missing ones.
- Supports one row per username (legacy-compatible), one row per site, or one row per claimed profile.
- Needs **no login, cookie, API key or proxy**.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `usernames` | array | `["octocat"]` | Handles to look up (required). A leading `@` is ignored. `{?}` expands to `_`, `-` and `.`. |
| `username` | string | – | Optional single handle merged into `usernames`; the array is still required. |
| `sites` | array | `[]` | Names (`GitHub`), domains (`github.com`) or profile URLs. Empty means every site. |
| `categories` | array | `[]` | `social`, `dev`, `gaming`, `forum`, `media`, `business`, `education`, `adult`, `other`. Combined with `sites` as a union. |
| `includeNsfw` | boolean | `false` | Also check adult sites. |
| `includeAvailable` | boolean | `false` | Also emit rows for free, blocked, unknown and illegal checks. |
| `verifyClaims` | boolean | `true` | Run the decoy re-check on generic claimed sites. |
| `enrichProfiles` | boolean | `true` | Add profile fields where available. |
| `outputMode` | string | `perUsername` | `perUsername`, `perSite` or `perHit`. |
| `maxUsernames` | integer | `100` | Cap after wildcard expansion (1–100). |
| `maxSitesPerUsername` | integer | `0` | Check only the top N priority sites; `0` means all (0–500). |

## Output

### `perUsername` (default, backward compatible)

One row per username. `username` and `links[]` keep their original meaning; `links` now contains only verified claimed profiles. New summary fields and a `profiles[]` array sit beside them.

```json
{
  "username": "octocat",
  "links": ["https://github.com/octocat"],
  "sitesChecked": 428,
  "profilesFound": 1,
  "sitesAvailable": 380,
  "sitesBlocked": 12,
  "sitesUnknown": 15,
  "sitesUnverified": 0,
  "checkedAt": "2026-11-04T12:00:00Z",
  "profiles": [
    {
      "username": "octocat",
      "site": "GitHub",
      "category": "dev",
      "url": "https://github.com/octocat",
      "status": "claimed",
      "httpStatus": 200,
      "detectionMethod": "api_json",
      "verified": true,
      "responseTimeMs": 210,
      "checkedAt": "2026-11-04T12:00:00Z",
      "profile": {
        "userId": "583231",
        "username": "octocat",
        "displayName": "The Octocat",
        "avatarUrl": "https://avatars.githubusercontent.com/u/583231",
        "followers": 18432,
        "following": 9,
        "joinedAt": "2011-01-25T18:44:36Z",
        "location": "San Francisco",
        "website": "https://github.blog",
        "stats": { "repositories": 8, "gists": 8 }
      }
    }
  ]
}
```

### `perSite`

One row per username × site with `username`, `site`, `category`, `url`, `status`, `httpStatus`, `detectionMethod`, `responseTimeMs` and `checkedAt`, plus `verified` on hit rows, `isNsfw` on adult rows and `profile` when enrichment is on. By default only claimed and unverified rows are emitted; `includeAvailable` adds the rest.

### `perHit`

One row per claimed and verified profile, with the same shape as `perSite`. With `verifyClaims=false`, claimed rows are emitted without the verification gate.

### Status vocabulary

| Status | Meaning |
|---|---|
| `claimed` | The site shows a profile for this handle. Native probes and verified generic checks are `verified: true`. |
| `available` | The handle is free on this site. |
| `unverified` | A generic site claimed the handle but could not prove it; kept out of `links`. |
| `waf` | The site returned a bot challenge, 403, 429 or 451. |
| `unknown` | Timeout, 5xx or an unparseable response. |
| `illegal` | The handle does not satisfy the site's own username rule. |

Empty strings, arrays and objects are omitted at every depth.

## Examples

Check a batch of handles across every non-NSFW site:

```json
{
  "usernames": ["johndoe", "janedoe", "octocat"],
  "maxUsernames": 3
}
```

Check only developer and gaming sites, one row per hit:

```json
{
  "usernames": ["octocat"],
  "categories": ["dev", "gaming"],
  "outputMode": "perHit",
  "enrichProfiles": true
}
```

Availability check for a new handle on selected sites:

```json
{
  "usernames": ["mynewbrand2026"],
  "sites": ["GitHub", "Instagram", "X (Twitter)", "Twitch", "Reddit"],
  "includeAvailable": true,
  "outputMode": "perSite",
  "verifyClaims": true
}
```

Fast top-20 scan per handle:

```json
{
  "usernames": ["johndoe"],
  "maxSitesPerUsername": 20,
  "enrichProfiles": false
}
```

## Use cases

- **Brand protection** — find impersonating or parked accounts across networks.
- **OSINT and investigations** — build a verified footprint from one handle.
- **Lead enrichment** — attach public profile URLs and display names to a handle list.
- **Username availability** — see where a new brand handle is free before registering.
- **Recruiting and sourcing** — locate developer profiles on GitHub, GitLab, Codeforces, LeetCode and Stack Exchange.
- **Creator discovery** — compare presence across YouTube, Twitch, Instagram, Pinterest and X.

## FAQ

**How do I know a hit is real?** Native probes use exact API/XML/HTML signals. Generic sites are re-checked with a random decoy handle: if the site claims the decoy too, the original row becomes `unverified` and is excluded from `links`. The `detectionMethod` and `verified` fields tell you exactly which path proved each hit.

**Why are there fewer links than the old version?** The old actor reported soft-404 and WAF pages as profiles. Ten such sites were removed from the catalog, and the remaining generic claims now pass the decoy check. Set `verifyClaims=false` to see the raw claimed rows.

**What does `unverified` mean?** The site returned a claimed signal, but a random handle produced the same signal, so the site cannot distinguish real profiles. The row is kept for transparency (and in `perSite` output) but never enters `links`. `profilesFound` counts claimed plus unverified candidates; `sitesUnverified` and `links` show which are proven.

**Why are NSFW sites missing?** They are excluded unless `includeNsfw=true`. The catalog marks 16 adult entries.

**A site I need is missing. Can I add it?** The catalog is pinned and covers 428 sites. Use `sites` to confirm whether a site is supported; unknown values are logged and ignored. The catalog can be refreshed with `scripts/refresh_catalog.py` when upstream adds sites.

**How does `{?}` work?** It expands to three handles using `_`, `-` and `.`. For example `john{?}doe` checks `john_doe`, `john-doe` and `john.doe`. Expansion happens before `maxUsernames` is applied.

**Do I need cookies, an API key or a proxy?** No. The actor is cookieless and anonymous. Sites that hard-block datacenter IPs or serve a bot challenge are reported honestly as `waf` or `unknown` rather than guessed.

**Why do some sites return `unknown` or `waf`?** Cloudflare, Akamai and similar edges sometimes challenge anonymous requests. Retries handle transient 429/5xx responses, but a persistent challenge stays `waf`, and a timeout or 5xx stays `unknown`.

**Can I check 100 handles in one run?** Yes, `maxUsernames` accepts 1–100. Handles are processed four at a time, with up to 50 site checks in flight across the run. Large scans take longer; narrow them with `sites`, `categories` or `maxSitesPerUsername`. GitHub's anonymous API allows 60 requests/hour, so the tail of a 100-handle GitHub-only batch can fall back to HTML or report `waf`.

**Which fields can be enriched?** Display name, bio, avatar, followers, following, posts, join date, location, website and verified badge, where the site's public response exposes them. Long-tail sites return URL and status only.

## Limitations

- **No login-only data.** Private profiles, follower lists, posts and activity feeds are out of scope.
- **Some sites cannot be detected anonymously.** Reddit, TikTok and Spotify are not shipped as claimed paths; they are either excluded upstream or return `waf`/`unknown`. Sites behind a persistent bot challenge behave the same way.
- **Instagram and X throttle HTML.** From datacenter IPs they can serve a login or generic page, so an existing profile may be reported `available` or `unknown`; no hit is fabricated.
- **Native enrichment covers 22 sites.** The long tail provides URL, status, HTTP code and detection method only.
- **GitHub's anonymous API is limited to 60 requests/hour.** Large GitHub-only batches fall back to HTML (fewer profile fields) or report `waf` for the tail.
- **Catalog size is 428 sites.** This is a footprint tool, not a 3,000-site Maigret replacement.
- **Decoy verification trades recall for precision.** A site that claims every handle is reported as `unverified`, even if a real profile exists there.
