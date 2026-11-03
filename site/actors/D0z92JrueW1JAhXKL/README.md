# General Osteopathic Council Register Scraper

Verify UK-registered osteopaths against the **General Osteopathic Council (GOsC)** official public register at `osteopathy.org.uk`. Search by postcode/town, surname, or practice features (disabled access, home visits, Welsh spoken) — or look up an exact GOsC registration number — to confirm whether an osteopath is currently registered and legally able to practise in the UK. No login, no cookies, no paid proxy required.

## What this actor does

- **Two lookup modes:** browse/filter the register, or exact registration-number lookup
- **Search by any combination of:** postcode or town (with a configurable radius for postcode searches), surname, disabled access, home visits, Welsh spoken
- **Browse the entire register** — leave all filters blank to page through every currently-registered osteopath
- **Full result-set pagination** — follows the register's own paging so large matches are returned in full (bounded only by `maxItems`)
- **Multiple practice addresses per osteopath** — many osteopaths practise at more than one clinic; every practice they are listed at is captured (address, opening hours, home visits, disabled access, Welsh spoken, website)
- **Empty fields are never included** — only real, extracted data is emitted

## Output per osteopath

- `registrantId` — GOsC internal registrant identifier
- `registrationNumber` — unique GOsC registration number
- `name` — full name as shown on the register (with courtesy title)
- `gender`
- `practisingStatus` — e.g. `Fully Registered`
- `dateOfRegistration` — date first joined the register (ISO `YYYY-MM-DD`)
- `qualifications` — post-nominal qualifications (e.g. `DO`, `BSc (Hons) Ost`)
- `placeOfTraining` — institution where the osteopath trained
- `qualificationDate` — month/year of qualification (ISO `YYYY-MM`)
- `contactNumber` — personal contact phone number, when published on the register
- `email` — personal contact email address, when published on the register
- `practiceCount` — number of practices this osteopath is listed at
- `practices[]` — one entry per practice: `name`, `practiceUrl`, `address`, `postcode`, `openingHours`, `email`, `phone`, `website`, `homeVisits` (boolean), `disabledAccess` (boolean), `welshSpoken` (boolean)
- `sourceUrl` — direct link to the official GOsC register entry
- `recordType`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | `search` | `search` (browse/filter) or `byRegistrationNumbers` (exact lookup) |
| `location` | string | – | UK postcode or town/city name (mode=search). Leave blank to browse the whole register |
| `radius` | select | `10mi` | `5mi` / `10mi` / `25mi` / `50mi` — only applies when `location` is a postcode |
| `surname` | string | – | Filter to osteopaths whose surname contains this text |
| `disabilityAccess` | boolean | `false` | Only return osteopaths practising where disabled access is available |
| `homeVisit` | boolean | `false` | Only return osteopaths who offer home visits |
| `welshSpoken` | boolean | `false` | Only return osteopaths practising where Welsh is spoken |
| `registrationNumbers` | array | – | Exact GOsC registration numbers (mode=byRegistrationNumbers) |
| `maxItems` | integer | `20` | Maximum number of osteopath records to return (1–2000) |

### Example: search osteopaths near a postcode

```json
{
  "mode": "search",
  "location": "SW1A 1AA",
  "radius": "10mi",
  "maxItems": 25
}
```

### Example: filter by surname and practice features

```json
{
  "mode": "search",
  "surname": "Smith",
  "homeVisit": true,
  "disabilityAccess": true,
  "maxItems": 25
}
```

### Example: look up exact registration numbers

```json
{
  "mode": "byRegistrationNumbers",
  "registrationNumbers": ["1766", "7065", "8596"]
}
```

## Use cases

- **Employers & clinics** verifying a candidate osteopath's registration status before hiring
- **Patients** confirming their osteopath is legitimately registered with the GOsC
- **Insurers** cross-checking practitioner registration numbers
- **Researchers** analysing the geographic distribution, qualifications, or training institutions of UK osteopaths
- **Directory builders** enriching a UK health-practitioner dataset with official register data

## FAQ

**Is this official data?**
Yes — every record is scraped live from the General Osteopathic Council's own public register search at `osteopathy.org.uk/register-search/`. This is the same register the GOsC itself uses to let the public verify osteopaths.

**Do I need an account, login, or cookies?**
No. The register search is fully public; no login or paid proxy is required.

**What if my search returns 0 results?**
Check the spelling of the surname/postcode, widen the `radius`, or remove filters. The actor sets a clear status message when a search or lookup finds no matches.

**Why do some osteopaths have more than one entry in `practices[]`?**
Many osteopaths work across multiple clinics. Every practice location currently listed against that osteopath on the register is captured in the `practices[]` array.

**Can I look up multiple registration numbers at once?**
Yes — set `mode` to `byRegistrationNumbers` and pass an array of registration numbers.

**Why do `sourceUrl` and `practices[].practiceUrl` return a blank/challenge page with a plain HTTP request (e.g. `curl`)?**
The GOsC register-search pages sit behind a JavaScript security check. They render correctly and instantly in any real browser (which is exactly how this actor reads them) but a plain HTTP client without JavaScript will see an interstitial instead of the page content. This does not affect the actor's own output — only manual `curl`/`wget` checks of these specific URLs.

**How long does a run take?**
Each osteopath record requires solving a JavaScript security check, so expect roughly 5-10 seconds per osteopath. For runs with a large `maxItems`, allow a generous run timeout (the platform default is usually sufficient, but very large runs — 100+ records — may need the timeout increased in Actor run settings).
