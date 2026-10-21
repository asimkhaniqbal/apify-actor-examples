# Google Maps Business Lead Finder with Email & Social Links

Turn a local market into a contact-ready lead list. Search Google Maps by phrase ("roofers in Dallas TX", "dentists in Berlin"), then read each business's own website for the email addresses, phone numbers and social profiles it publishes. Two more modes let you enrich a list of websites you already own, or look up specific Google Maps places.

No Google account, cookies or API key is needed.

## What this actor does

- **Search Google Maps** — runs each search phrase and keeps the businesses Google returns for it. Every place comes with its listing details and a link to its Google Maps page.
- **Read the business website** — visits the homepage and a small number of contact, about, legal, privacy or careers pages (including local names such as *kontakt* and *impressum*), and collects public emails, phone numbers and social profiles.
- **Check email domains** — reports which found addresses sit on a domain that actually accepts mail (an MX record), so you can prioritise them before outreach.
- **Filter and de-duplicate** — one row per business, matched by Place ID; optional rating, category, website and closed-place filters.
- **Enrich websites directly** — paste a list of domains when you already have the businesses and only need their contact details.
- **Look up specific places** — paste Google Maps place links or Place IDs and get the same lead rows without running a search.

The actor never guesses an email address from a name or a domain, and never invents fields. If a business does not publish a contact, the field is left out.

## Output per lead

| Field | Type | Description |
|-------|------|-------------|
| `recordType` | string | Always `lead` |
| `searchQuery` | string | The search phrase that found the business (search mode only) |
| `title` | string | Business name |
| `categoryName` | string | Primary Google Maps category |
| `categories` | string[] | All Google Maps categories for the place |
| `address` | string | Full address |
| `phone` | string | Phone number shown on Google Maps |
| `website` | string | The business's website |
| `emails` | string[] | Public email addresses found on the business's own website |
| `emailsMxValid` | string[] | The subset of `emails` whose domain published a mail server record at run time |
| `phonesOnWebsite` | string[] | Phone numbers found on the website (may differ from the Maps number) |
| `contactPageUrl` | string | The contact or legal page that supplied the first website email |
| `totalScore` | number | Google Maps rating (1–5) |
| `reviewsCount` | integer | Number of Google reviews |
| `placeId` | string | Canonical Google Place ID (starts with `ChIJ`) |
| `featureId` | string | Legacy Google feature token (`0x…:0x…`) |
| `kgmid` | string | Google Knowledge Graph identifier |
| `url` | string | Google Maps URL of the place |
| `sourceUrl` | string | Source URL (Maps place URL, or the website in website mode) |
| `location` | object | `{lat, lng}` place coordinates |
| `plusCode` | string | Plus Code computed from the place coordinates |
| `priceLevel` | string | Price range only when Google supplies one; often unavailable |
| `socialLinks` | object | Profile URLs for Facebook, Instagram, LinkedIn, X/Twitter, YouTube, TikTok, Pinterest, Threads, Telegram and WhatsApp |
| `scrapedAt` | string | ISO 8601 timestamp shared by all rows of a run |

## Input

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `mode` | select | `searchPlaces` | What to do: search Google Maps, enrich websites, or scrape specific places |
| `searchQueries` | string[] | – | Search phrases for Google Maps (search mode). Include the place in the phrase or use `locationQuery` |
| `locationQuery` | string | – | Adds a shared location to every search phrase, e.g. `Austin, TX, USA` |
| `websites` | string[] | – | Websites to enrich (website mode); bare domains are accepted |
| `placeUrls` | string[] | – | Google Maps place links (specific-places mode) |
| `placeIds` | string[] | – | Canonical Place IDs starting with `ChIJ` (specific-places mode) |
| `maxPlacesPerSearch` | integer | `20` | Maximum businesses kept per search phrase (1–1000) |
| `language` | select | `en` | Language of the Google Maps interface and listing text (74 options) |
| `extractEmailsFromWebsite` | boolean | `true` | Visit business websites to collect contact details |
| `maxEmailsPerWebsite` | integer | `5` | Maximum emails kept per website (1–20) |
| `maxContactPagesPerWebsite` | integer | `3` | Pages read per website: homepage plus contact/about/legal/privacy/careers pages (1–10) |
| `onlySaveResults` | select | `allPlaces` | Save all places, only places with an email, or only places with any contact detail |
| `placeMinimumStars` | select | Any rating (empty) | Keep only places rated at least the selected value |
| `categoryFilter` | string[] | – | Keep places whose category contains all words of at least one entry |
| `websiteFilter` | select | `allPlaces` | All places, only places with a website, or only places without one |
| `skipClosedPlaces` | boolean | `false` | Check public status and leave out permanently or temporarily closed places |

### Example: search Google Maps

```json
{
  "mode": "searchPlaces",
  "searchQueries": ["roofing contractors in Dallas TX", "coffee shops in Austin TX"],
  "maxPlacesPerSearch": 50,
  "extractEmailsFromWebsite": true,
  "onlySaveResults": "onlyWithContacts"
}
```

### Example: enrich websites you already have

```json
{
  "mode": "enrichWebsites",
  "websites": ["example-bakery.com", "example-dentist.de"],
  "maxContactPagesPerWebsite": 3
}
```

### Example: specific places

```json
{
  "mode": "specificPlaces",
  "placeUrls": ["https://www.google.com/maps/place/…"],
  "placeIds": ["ChIJ…"],
  "skipClosedPlaces": true
}
```

## How complete is the email data?

Google Maps has no email field — the address lives on the business's own website, or nowhere. In a 399-business census across restaurants, contractors, dentists, salons, auto repair and shops in the US and Europe, **66% of businesses published an email within the homepage plus two contact pages**, and **70% when up to nine contact pages were checked**. Homepages supplied most of them; contact, about, legal, privacy and careers pages added the rest, and theme/store configuration files supplied a further slice. German, Austrian and Swiss businesses are often legally required to publish an Impressum page with a contact address, so those markets tend to score higher (86% in the London restaurant sample).

Of the rest, about 22% publish only a contact form or no address at all, and a small share of sites are unreachable (dead, archived or behind bot protection). Those rows keep their Google Maps details, but no website contacts. Nothing is fabricated to fill the gap.

## Limitations

- Google Maps returns a ranked window of about 160–220 places per search phrase, not every business in a city. Use several related phrases to widen coverage.
- A business with no website, or one that publishes only a contact form, cannot yield a public email. Raising `maxContactPagesPerWebsite` finds addresses that only appear on deeper privacy or careers pages.
- `emailsMxValid` confirms that the email domain publishes a mail-server (MX) record; it cannot detect a catch-all mailbox or an abandoned inbox. A domain that publishes only an A record (no MX) is not included.
- `priceLevel` is usually unavailable from public listing data and is omitted rather than guessed.
- `plusCode` is computed from the place coordinates, so it carries the code but not Google's locality suffix next to it.
- Closed-place checking reads Google's public status; a place whose status cannot be read is kept.
- Results depend on Google Maps' search relevance for your phrase.

## FAQ

**How many businesses can I get per search phrase?**
Up to what Google Maps exposes for that phrase — usually 160–220 unique places, and you can keep up to 1000. Use several related phrases to cover more of a market.

**Does this actor need cookies, a login or an API key?**
No. It reads public Google Maps pages and public business websites.

**Why does a row have no email?**
Not every business has a website, and not every website publishes an email. Some use a contact form or a booking widget instead. Rows without contacts are kept unless you choose `onlySaveResults`.

**What does `emailsMxValid` mean?**
It is the part of `emails` whose domain published a mail-server (MX) record when the run executed. A domain that publishes only an A record is not included. It is a deliverability hint, not a guarantee that a person reads that inbox.

**Can I search in languages other than English?**
Yes. `language` offers 74 Google Maps languages, and contact-page discovery understands common local page names such as kontakt, impressum, contacto, contatti and contactez-nous.

**Can I use the actor on a list of websites without Google Maps?**
Yes. Choose `enrichWebsites` and paste the domains.

**Are both email addresses and phone numbers included?**
Yes. `phone` is the Google Maps number; `phonesOnWebsite` lists numbers found on the business's site.

**Is this affiliated with Google?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Google.

Please use this public data lawfully and in line with Google's terms.
