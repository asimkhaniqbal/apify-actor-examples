# US County & City Unclaimed Funds Scraper

Many US counties and cities publish their own unclaimed-funds ledgers as open data, separately from (and often more accessible than) their state's official Treasury search. This actor searches those ledgers by owner name, or lets you browse a state's register outright, returning amount, holder, property type and dates wherever the source discloses them.

## What this actor does

- **Two modes:** `search` (owner name) and `browse` (list a state's register)
- **6 states covered, 9 government sources:** California, Maryland, Massachusetts,
  Minnesota, Rhode Island, Texas
- **Multi-state in one run** — set `state` to `ALL` to search every source at once
- **Jurisdiction filter** — narrow to one specific government source (useful when a
  state has more than one, e.g. California or Massachusetts)
- **Sorting** — highest/lowest amount, most/least recent, or source order
- **Minimum- and maximum-amount filters** to bracket a balance range
- **Empty fields are omitted** — a source is never forced to report a field it doesn't disclose

## Data source

This actor aggregates **public, unauthenticated Socrata (SODA) open-data APIs**
published directly by 9 US county/city treasuries as their official unclaimed-funds /
uncashed-check registers:

| State | Jurisdiction | Property type |
|---|---|---|
| California | Marin County | Uncashed county checks |
| California | City of Roseville | Stale-dated municipal checks |
| Maryland | Montgomery County | Unclaimed vendor payments |
| Massachusetts | Town of Framingham | Unclaimed payroll checks |
| Massachusetts | Town of Framingham | Unclaimed resident/vendor/AP checks |
| Minnesota | Ramsey County | Unclaimed money |
| Rhode Island | City of Providence | Uncashed municipal checks |
| Texas | City of Austin | Unclaimed property |
| Texas | City of Corpus Christi | Unclaimed property |

**Why not a state-level Treasury / MissingMoney.com?** We researched this thoroughly
before building. The obvious "one-stop" target — MissingMoney.com, the NAUPA-endorsed
multi-state search — is blocked behind three layered anti-bot systems (a Cloudflare
Turnstile challenge injected into every page, AWS WAF Bot Control 403s, and a
`__cf_bm` bot-detection cookie); this holds for plain HTTP clients, TLS-impersonating
clients, and full headless browsers alike. The same problem exists further down the
list: FDIC's national unclaimed-funds search requires a validated Google reCAPTCHA
token; PBGC's pension search requires the claimant's own Social Security number;
Texas's own state-level open-data unclaimed-property dataset returns `403` to
anonymous requests and requires emailing the Comptroller's office for an access
grant. In short, **every state's live name-search portal is CAPTCHA/WAF-gated** —
that's exactly why third-party "finder" sites merely deep-link into those portals
instead of scraping them.

What *is* reliably open is the layer beneath the search UI: many county and city
treasuries publish their own unclaimed-funds ledgers as public Socrata datasets with
anonymous API access (no key, no login, no CAPTCHA) — the same open-data
infrastructure many US cities use for budgets, permits, and 311 data. This actor
aggregates 9 such datasets across 6 states into a single multi-state search/browse
tool. It complements, and does not duplicate, this repo's `CAUnclaimedPropertyScraper`
(which is single-state, sourced from the California State Controller's own bulk CSV
export of its statewide database — a different tier of source entirely).

## Output per record

| Field | Description |
|---|---|
| `ownerName` | Owner/payee name on the unclaimed funds |
| `amountUsd` | Dollar amount, where the source discloses one |
| `reportedDate` | Check/transaction date (`YYYY-MM-DD`), where disclosed |
| `address` | Last-known street address, where disclosed (joins multi-line addresses, e.g. Providence's suite/floor lines, into one string) |
| `city` | City, where disclosed |
| `postalCode` | ZIP code, where disclosed |
| `recordId` | Source's own record/check ID, where disclosed |
| `vendorId` | Source's own vendor/payee ID, where disclosed — currently only Montgomery County, MD |
| `invoiceDate` | Underlying invoice date (`YYYY-MM-DD`), where disclosed separately from `reportedDate` — currently only Montgomery County, MD |
| `state` | 2-letter state code |
| `jurisdiction` | The specific county/city government holding the funds |
| `holderName` | Same as `jurisdiction` — the entity holding the money |
| `propertyType` | Category, e.g. "Unclaimed Vendor Payment (County)" |
| `paymentStatus` | Bank/ERP status of the payment (e.g. "Negotiable", "Void"), where disclosed — currently only Montgomery County, MD |
| `sourceUrl` | The government's own open-data page for this dataset |
| `recordType` | Always `"unclaimedProperty"` |
| `scrapedAt` | UTC timestamp of the scrape |

Not every jurisdiction discloses every field — e.g. the City of Austin's dataset lists
owner names only, with no amount or date. Fields that aren't disclosed are simply
omitted, never faked.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search` (by owner name) or `browse` (no name filter) |
| `state` | select | `MN` | `ALL`, `CA`, `MA`, `MD`, `MN`, `RI`, or `TX` |
| `jurisdiction` | select | `ALL` | Narrow to one specific government source's dataset (e.g. just Marin County, not all of CA) |
| `ownerName` | string | `Johnson` | Required for `mode=search`; case-insensitive substring match |
| `minAmountUsd` | int | `0` | Only include records at least this large (where amount is disclosed) |
| `maxAmountUsd` | int | unset | Only include records at most this large (where amount is disclosed) |
| `sortBy` | select | `amountDesc` | `amountDesc` / `amountAsc` / `dateDesc` / `dateAsc` / `relevance` |
| `maxItems` | int | `20` | Hard cap on emitted records (1–1000) |

### Example: search by name across all states

```json
{
  "mode": "search",
  "state": "ALL",
  "ownerName": "Smith",
  "maxItems": 50
}
```

### Example: browse California's highest-value unclaimed funds

```json
{
  "mode": "browse",
  "state": "CA",
  "sortBy": "amountDesc",
  "minAmountUsd": 1000,
  "maxItems": 100
}
```

### Example: recent unclaimed payroll checks in Massachusetts

```json
{
  "mode": "browse",
  "state": "MA",
  "sortBy": "dateDesc",
  "maxItems": 25
}
```

## Use cases

- **Asset-recovery researchers** — screen multiple states' unclaimed-funds ledgers by name in one run
- **Genealogy / heir-search firms** — sweep county-level registers for a family surname
- **Finance / fraud investigators** — cross-reference vendor payment records with known entities
- **Local journalism** — surface large uncashed municipal payments for public-interest reporting
- **Personal finance tools** — let users check multiple jurisdictions for money owed to them

## FAQ

**Does this cover my state's full unclaimed-property database?** No — this actor
covers county/city government ledgers that publish open Socrata data, not each
state's complete Treasury database (those require going through the state's own
CAPTCHA-protected search, which this project does not bypass). For California
specifically, see this repo's separate `CAUnclaimedPropertyScraper`, sourced from the
CA State Controller's full statewide bulk CSV export.

**Why is `amountUsd` missing on some records?** The source jurisdiction (currently
only the City of Austin, TX) publishes owner names without a disclosed dollar amount.
We never fabricate a placeholder value — the field is simply omitted.

**What does `paymentStatus` mean?** Montgomery County, MD is the only jurisdiction
that discloses this — it's the bank/ERP status of the payment (e.g. "Negotiable"
means the check can still be cashed; "Void" or "Reconciled" means it can't). No
other source in this actor publishes an equivalent field.

**How do I claim money I find?** This actor surfaces public record data only; it does
not file claims. Contact the `jurisdiction`/`holderName` government office directly,
or visit the `sourceUrl` dataset page for the office's own contact information.

**How often is the underlying data refreshed?** Each government body refreshes its
own Socrata dataset on its own schedule (typically daily-to-monthly); the actor always
queries the live API, so results reflect whatever is currently published.

**Can I search all 6 states in a single run?** Yes — set `state` to `ALL`; the actor
queries all 9 sources concurrently and merges/sorts the results.
