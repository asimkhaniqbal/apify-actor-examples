# Google Ads Transparency Creative Lookup by Domain

Scrape live and recent ads shown through Google — search by landing-page domain or exact advertiser ID. Get ad format, preview creative, advertiser identity, and first/last shown dates from the public [Ads Transparency Center](https://adstransparency.google.com). No login, no API key, no cookies.

## What this actor does

- **Two modes:** `byQuery` (landing-page domain — most reliable) and `byAdvertiserId` (one or more exact IDs)
- **Filters:** region/market (89 to choose from), ad format (text/image/video), platform (Search/Maps/Shopping/Play/YouTube), last-shown date range
- **One page per run:** each run reads a single page of up to `maxItems` (at most 100) creatives; format and date filters are then applied to that page
- **Preview assets:** static thumbnail image URL (and dimensions) when available, and/or a live preview script URL
- **Empty fields are omitted**

## Output per ad

- `advertiserId`, `advertiserName`
- `creativeId`, `creativeUrl` — deep link to the ad's page on Ads Transparency Center (with your `region`)
- `advertiserUrl` — deep link to the advertiser's page
- `format` — `Text`, `Image`, or `Video`
- `previewImageUrl`, `previewWidth`, `previewHeight` — static thumbnail and its size, when Google stores one
- `previewContentUrl` — live preview script URL, when Google stores one
- `firstShownAt`, `lastShownAt` — ISO-8601 UTC
- `firstShownTimestamp`, `lastShownTimestamp` — Unix epoch seconds
- `region` — the region code you selected
- `recordType: "ad"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byQuery` | `byQuery` / `byAdvertiserId` |
| `query` | string | `nike.com` | Landing-page domain (mode=byQuery). The upstream search endpoint reliably matches domains (e.g. `nike.com`) — free-text advertiser names (e.g. `Nike`) frequently return zero results, so a domain is strongly recommended. Use `mode=byAdvertiserId` for a name-independent, always-stable lookup once you have an `advertiserId`. |
| `advertiserIds` | array | `[]` | Exact advertiser IDs, e.g. `AR16735076323512287233` (mode=byAdvertiserId). All IDs are looked up together, so `maxItems` is shared across them |
| `region` | string | `US` | Market to search; it is sent with the lookup and written on every record |
| `adFormat` | string | `""` (all formats) | `TEXT` / `IMAGE` / `VIDEO` |
| `platform` | string | `""` (all platforms) | `SEARCH` / `MAPS` / `SHOPPING` / `PLAY` / `YOUTUBE` |
| `dateFrom` | string | – | Only ads last shown on/after this date (`YYYY-MM-DD`) |
| `dateTo` | string | – | Only ads last shown on/before this date (`YYYY-MM-DD`) |
| `maxItems` | int | `40` | Number of creatives requested (1–100); filters can leave fewer |
| `proxyConfiguration` | object | `{"useApifyProxy": true}` | Apify Proxy settings; requests that get temporarily blocked are retried through a fresh proxy IP |

### Example: recent ads for a domain

```json
{
  "mode": "byQuery",
  "query": "nike.com",
  "region": "US",
  "maxItems": 40
}
```

### Example: video ads only, by exact advertiser ID

```json
{
  "mode": "byAdvertiserId",
  "advertiserIds": ["AR16735076323512287233"],
  "adFormat": "VIDEO",
  "maxItems": 50
}
```

## Use cases

- **Competitive intelligence** — see what ad creatives a competitor is running right now
- **Ad compliance monitoring** — audit an advertiser's messaging and formats
- **Creative research** — collect a swipe file of live ad copy and imagery by niche
- **Brand safety** — verify what ads are (and aren't) associated with a domain

## FAQ

**Is this the official Google Ads API?**  No. It reads the same public search results you get on adstransparency.google.com, without needing a Google Ads account or developer access.

**Do I need a Google account or API key?**  No. The Ads Transparency Center is a public tool; no login is required to browse or search it.

**Why is `maxItems` capped at 100?**  The Transparency Center returns at most 100 creatives per lookup, and the actor doesn't page beyond that first batch. Larger advertisers therefore can't be fully listed in one run.

**Some ads have `previewImageUrl` and others have `previewContentUrl` — why?**  It depends on how Google stores the creative: some come with a static thumbnail image, others only with a script that renders the ad live, and some with both. The actor passes on whichever Google provides.

**Does `region` actually filter results?**  It's passed to the upstream API the same way the website's region selector does. For some advertisers the set of ads returned does not change much across regions, since global advertisers run largely the same catalog everywhere — this mirrors what you'd see switching regions on the website itself.

**How do I find an advertiser ID?**  Run `mode=byQuery` first with the advertiser's domain — the `advertiserId` field in the results is reusable as input to `mode=byAdvertiserId` for a stable, unambiguous lookup. You can also copy it from an `adstransparency.google.com/advertiser/AR...` URL.

**What if nothing comes back?**  The run ends with an empty dataset and a status message explaining why: no ads matched, the filters removed everything, or Google kept blocking the lookup after several retries (try again shortly).

**How fresh is the data?**  Each run reads what the Ads Transparency Center shows at that moment; `lastShownAt` tells you when Google last recorded each ad running.

**Why does `dateTo` (an older cutoff) sometimes return 0 results for popular advertisers?**  Because only one page of up to 100 creatives is fetched (see above), and for an advertiser running continuous campaigns that page is usually filled with creatives whose `lastShownAt` is very recent. A `dateTo` cutoff further in the past than that will correctly exclude all of them — that's not a bug, it means none of the currently-fetchable creatives are that stale. `dateTo` is most useful for advertisers whose campaigns have genuinely paused/ended, or combined with `dateFrom` for a recent window (e.g. "last shown in the past week").

**Is this affiliated with Google?**  No. It is an independent tool that reads the public Ads Transparency Center and is not run or endorsed by Google.
