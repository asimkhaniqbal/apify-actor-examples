# Amazon Autocomplete Keyword Expander

Discover **Amazon's own search-autocomplete keyword suggestions** for any seed keyword or phrase, across 23 Amazon marketplaces. Feed it one or more seeds and it returns the exact completions Amazon's search box would suggest, in the order Amazon returns them. Expand coverage past Amazon's ~10-suggestion-per-query cap with A-Z / 0-9 / custom-suffix querying, and recursively re-query newly discovered suggestions to build out a full suggestion graph.

## What this actor does

- **Any number of seed keywords** — pass a list, get suggestions for each
- **23 marketplaces** — from `amazon.com` to `amazon.co.jp`
- **Suffix expansion** — append A-Z, 0-9, or your own custom suffixes to a seed to pull more suggestions than a single raw query returns
- **Recursive expansion** — treat each newly discovered suggestion as a fresh seed, up to 2 levels deep
- **Deduplication** — skip suggestions already seen earlier in the same run
- **Empty fields are omitted**

## Output per suggestion

- `marketplace` — `countryCode`, `domain`, `marketplaceId`, `currency`, `language`
- `prefix` — the exact query string that produced this result (may include an expansion suffix, e.g. `wireless earbuds a`)
- `suggestion` — the suggested keyword text Amazon returned
- `position` — 1-based order within that specific API response
- `suggestionType` — Amazon's own suggestion-type label when present (e.g. `KEYWORD`), or `SEED` for an echoed seed-keyword record
- `expansionDepth` — 0 for a direct query of an original seed keyword, 1/2 for recursive re-queries
- `expansionParent` — the prior suggestion that was used as the seed for this query (absent at depth 0)
- `refTag` — Amazon's own click-tracking reference tag for this suggestion (e.g. `nb_sb_ss_i_1_12`)
- `candidateSources` — where the API sourced this suggestion (e.g. `local`)
- `strategyId` — the suggestion-generation strategy Amazon used (e.g. `organic`)
- `prior` — Amazon's own numeric prior/confidence score for this suggestion, when present
- `ghost` / `help` — Amazon's own booleans marking a suggestion as placeholder ("ghost") or help text, when present
- `responseId` — Amazon's own per-request correlation id for the API call that returned this suggestion
- `shuffled` — whether Amazon randomized this response's suggestion order (when `true`, `position` is even less meaningful as a ranking signal)
- `sourceUrl` — the exact `completion.amazon.{domain}` autocomplete API URL that produced this record
- `recordType: "suggestion"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `keywords` | array | – | Seed keywords/phrases to expand |
| `marketplace` | string | `US` | Which Amazon marketplace's autocomplete API to query |
| `expandWith` | string | `none` | `none` / `alphabet` / `digits` / `customSuffixes` |
| `customSuffixes` | array | – | Suffixes to append (expandWith=customSuffixes) |
| `maxDepth` | int | `0` | Recursion depth for re-querying new suggestions (0-2) |
| `maxSuggestions` | int | `50` | Hard cap on emitted records (1-2000) |
| `dedupe` | bool | `true` | Skip suggestions already emitted this run |
| `includeSeed` | bool | `false` | Also emit a record for the literal seed keyword |

### Example: quick single-keyword lookup

```json
{
  "keywords": ["wireless earbuds"],
  "marketplace": "US",
  "maxSuggestions": 20
}
```

### Example: full A-Z expansion for keyword research

```json
{
  "keywords": ["yoga mat"],
  "marketplace": "US",
  "expandWith": "alphabet",
  "maxSuggestions": 300
}
```

### Example: recursive 2-level expansion, UK marketplace

```json
{
  "keywords": ["running shoes"],
  "marketplace": "GB",
  "expandWith": "alphabet",
  "maxDepth": 2,
  "maxSuggestions": 500
}
```

### Example: custom suffixes for a niche

```json
{
  "keywords": ["phone case"],
  "customSuffixes": ["iphone 15", "samsung s24", "waterproof", "wallet"],
  "expandWith": "customSuffixes",
  "maxSuggestions": 100
}
```

## Use cases

- **SEO / PPC keyword research** — discover the exact long-tail phrases Amazon shoppers' search box completes to
- **Listing optimization** — find related phrasing to include in titles and bullet points
- **Competitive intelligence** — see what completions surface for a competitor's product category
- **Content planning** — build topic clusters from a seed niche keyword
- **International expansion** — compare suggestion coverage across marketplaces for the same seed

## FAQ

**Is this a search-volume or popularity tool?**
No. This is a *suggestion-discovery* actor. Amazon's autocomplete API returns an ordered list of completions for its own UX purposes — `position` reflects that response's order at the moment of the request, not a numeric popularity or search-volume estimate. Don't treat it as one.

**Why does the same suggestion sometimes appear from different queries?**
Amazon's autocomplete can return the same completion for multiple related prefixes (e.g. both `wireless ear` and `wireless earbuds a` might surface `wireless earbuds bluetooth`). Leave `dedupe` on (the default) to keep only the first occurrence.

**Why is `expandWith` needed if I already give a full-phrase seed?**
Amazon caps each raw autocomplete response at roughly 10 suggestions per query, no matter how you phrase the seed. Appending a letter/digit/custom suffix (`{seed} a`, `{seed} b`, …) is the standard technique to surface additional completions beyond that per-request cap.

**What does `maxDepth` actually recurse on?**
Each *newly discovered* suggestion (not already seen) can itself become a new seed keyword, which is then queried (and optionally suffix-expanded) the same way. `maxDepth=1` re-queries suggestions found from your original seeds; `maxDepth=2` goes one level further.

**Is authentication or a proxy required?**
No. The autocomplete endpoint is a public, unauthenticated JSON API and responds directly to a plain HTTP request — no browser rendering, cookies, or proxy needed.

**Which marketplaces are supported?**
23 Amazon marketplaces, from the US to Japan — see the `marketplace` dropdown for the full list.

**How fresh are the suggestions?**
Live — every run queries Amazon's autocomplete API directly at request time.

**Is this affiliated with Amazon?**
No, this is a third-party actor using Amazon's public autocomplete API.

**What happens if a run returns zero suggestions?**
The run's status message tells you why. If every request to Amazon's autocomplete API failed outright (repeated network/5xx errors, even after retries) — which happens occasionally for specific marketplaces during a temporary outage on Amazon's side — the message says so explicitly and suggests retrying later or trying a different marketplace. If the requests succeeded but genuinely returned no completions (e.g. an obscure or misspelled seed keyword), you'll instead see a message suggesting a shorter, more common keyword.
