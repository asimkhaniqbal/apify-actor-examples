# Facebook Political & Commercial Ad Transparency Extractor

Read Facebook's public Ad Library directly and export one clean JSON record per ad — creative copy, carousel cards, images and videos, placements, advertiser identity, run dates, and the transparency numbers Meta only publishes for political and EU-delivered ads (spend ranges, impressions, reach, payer/beneficiary, age/country/gender reach). Works with a plain guest session: no Facebook login, no cookies, no Meta developer app.

## What you get

- **Four ways to search** — keywords, advertiser Pages, exact Ad Library IDs, or the funding entity on a "Paid for by" disclaimer, all in one run if you want
- **Meta's own transparency payloads, re-emitted whole** — political spend and impression ranges; EU total reach plus the per-country age/gender reach breakdown; declared payer and beneficiary; audience age/gender/location distributions returned as Meta publishes them
- **Creative in full** — primary media URL (copied to a permanent URL before it can expire), every image and video URL, carousel cards with their own text/CTA/link, CTA type, caption, title and display format
- **Advertiser context** — Page URL, profile picture, likes, categories, verification, and the linked Instagram handle with follower count and About text
- **Scale signals** — `collation_count` (variants sharing the same creative), `days_running`, placements, and the special-ad categories
- **Honest gaps** — a field Meta does not disclose for an ad is simply left out; nothing is estimated or fabricated

## Data source

The actor talks to the same internal JSON endpoint the Ad Library web page uses (`facebook.com/api/graphql/`, guest session). The page's own query identifiers are re-learned from the live site when they rotate, so the actor keeps working across Meta releases. No credential of any kind is required. Requests run through a built-in connection ladder (direct first, an Apify datacenter pool, then residential as a last resort) chosen automatically per request — you never configure a proxy.

## Modes

| Mode | What it does |
|---|---|
| `search` | Keyword search across all advertisers (any-order or exact-phrase matching) |
| `byPage` | Every ad an advertiser Page is running or has run |
| `byAdId` | Fetch one or more specific ads by their Ad Library ID |
| `byByline` | Political/issue ads by their "Paid for by" funding entity |
| (empty) | Processes every target list you filled in, in a single run |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | select | All provided targets | Which target list to use; empty processes all of them |
| `searchTerms` | array | – | Keywords, one search each |
| `pageIds` | array | – | Numeric Page IDs or URLs containing `view_all_page_id=` / `?id=` |
| `adIds` | array | – | Numeric Ad Library IDs or ads-library URLs containing `?id=` |
| `bylines` | array | – | "Paid for by" funding-entity names (exact string matching; always searched in the political & issue archive, where this filter exists) |
| `country` | select | `ALL` | Single country code; ignored when `countries` is set |
| `countries` | multi-select | – | Several country codes searched together (true union) |
| `adActiveStatus` | select | `active` | `all` / `active` / `inactive` |
| `adType` | select | `all` | `all` / `political_and_issue_ads` / `housing` / `employment` / `credit` |
| `mediaType` | select | `all` | `all` / `image` / `video` / `meme` / `none` |
| `publisherPlatforms` | multi-select | – | FACEBOOK / INSTAGRAM / MESSENGER / AUDIENCE_NETWORK / WHATSAPP / THREADS |
| `contentLanguages` | array | – | ISO 639-1 codes of the ad creative (e.g. `en`, `de`) |
| `searchType` | select | `keyword_unordered` | Any-order or exact-phrase keyword matching |
| `startDate` | date | – | Ads with impressions on/after this date (delivery window) |
| `endDate` | date | – | Ads with impressions on/before this date |
| `sortBy` | select | `relevance` | `relevance` or `total_impressions` |
| `resultsPerSearch` | integer | `50` | Cap per search target (1–100000) |
| `maxItems` | integer | `1000` | Global cap across everything (1–100000) |
| `includeEuDetails` | boolean | `true` | Fetch EU transparency, payer/beneficiary and advertiser profile per ad |
| `rehostMedia` | boolean | `true` | Copy the primary media into a permanent URL (originals kept in `media_url_original`) |
| `onlyTotal` | boolean | `false` | Return just the total result count per target, read from the Ad Library page in one lightweight request (no browser, no pagination) — respects every filter including the date window; Facebook caps displayed totals at 50,001 (50,000+) |

## Output fields

Every record is omit-empty (a field that does not apply is absent, never null).

| Field | Meaning |
|---|---|
| `ad_id`, `ad_snapshot_url` | Ad Library ID and its public snapshot URL |
| `page_id`, `page_name`, `page_url`, `page_profile_picture_url` | Advertiser identity |
| `page_like_count`, `page_categories`, `page_verification`, `page_is_deleted` | Advertiser page signals |
| `advertiser_ig_username`, `advertiser_ig_followers`, `advertiser_ig_verified`, `advertiser_about` | Linked Instagram profile and About text |
| `ad_text`, `title`, `caption`, `link_description`, `display_format` | Creative text (as Meta serves it) |
| `template_body`, `template_title`, `template_tokens`, `resolved_ad_text`, `resolved_title` | Dynamic-creative handling: raw `{{…}}` placeholders are kept in `template_body`/`template_title`, their token names in `template_tokens`, and a token-free `resolved_ad_text`/`resolved_title` is attached when the payload's carousel cards carry the served variant |
| `cta_text`, `cta_type`, `link_url` | Call to action and destination |
| `media_type`, `media_url`, `media_url_original`, `media_url_expires_at` | Primary creative media |
| `image_urls`, `video_urls`, `video_preview_urls` | All creative media URLs |
| `cards` | Carousel cards (title, body, caption, link, CTA, image, videos) |
| `start_date`, `end_date`, `days_running`, `status`, `is_active` | Delivery window and live status |
| `publisher_platform` | Placements (Facebook, Instagram, Messenger, …) |
| `collation_count`, `collation_id` | Creative-reuse group size and ID |
| `categories`, `gated_type`, `regional_regulation_data` | Special categories and regulation flags |
| `spend`, `spend_lower_bound`, `spend_upper_bound`, `currency` | Spend range (political & EU-regulated ads) |
| `impressions`, `impressions_index`, `impressions_lower_bound`, `impressions_upper_bound`, `reach_estimate` | Impression/reach signals (same gating) |
| `byline`, `disclaimer_label` | "Paid for by" funding entity |
| `eu_total_reach`, `age_country_gender_reach_breakdown` | EU DSA total reach and per-country age/gender breakdown |
| `target_age_audience`, `target_gender_audience`, `target_location_audience` | Declared EU/UK targeting |
| `audience_age_gender`, `audience_location`, `audience_single_country` | Audience distribution as published |
| `payer_beneficiary`, `violation_types`, `is_violating_eu_siep` | EU payer/beneficiary declarations and violations |
| `contains_digital_created_media`, `contains_sensitive_content`, `has_user_reported`, `is_aaa_eligible`, `total_active_time`, `hide_data_status`, `state_media_run_label` | Additional Meta flags |
| `search_term`, `scraped_at` | Which search produced the row, and when |

Diagnostic rows use `type: "facebook_ads_library_diagnostic"` with `searchTerm`, `outcome`, `message` and `scrapedAt`; `onlyTotal` runs push `type: "facebook_ads_library_total"` with `total_results`. Ad-ID lookups that neither the search endpoint nor the ad's own page can resolve produce a `detail_unavailable` diagnostic that reports Facebook's own reason (`NO_AD_FOUND`) — no record is ever fabricated.

## Example

```json
{
    "mode": "search",
    "searchTerms": ["coffee"],
    "country": "US",
    "adActiveStatus": "active",
    "includeEuDetails": true,
    "maxItems": 50
}
```

## FAQ

**Do I need a Facebook account or cookies?** No. The actor works with an anonymous guest session by default.

**Why is spend missing on some ads?** Meta publishes spend and impressions ranges only for political/issue ads and EU-regulated advertising. The actor returns those fields when the source has them and omits them otherwise — it never fills a gap with a guess.

**Why did my political search for an EU country return all ad types?** Meta's political/issue archive does not cover EU/EEA countries; EU political ads live in the general transparency stream, so the actor automatically searches all types there and says so in the run log.

**Will image and video links keep working?** Facebook's media links are signed and expire within days. With `rehostMedia` on, the primary media is copied to a permanent URL first and the original is preserved in `media_url_original`; the arrays keep Meta's URLs, each with its expiry in `media_url_expires_at`.

**How far back does the data go?** Whatever the public Ad Library serves for your query: seven years for political/issue ads, one year for ads delivered to the EU, and live archive coverage for commercial ads.

**Can I scrape several countries or targets at once?** Yes — `countries` yields a union, and every target list can run together in one job.

**What is `collation_count`?** The number of ad variants Meta groups under the same creative and text — a quick way to spot heavily scaled campaigns.

**Why does `ad_text` sometimes contain `{{product.brand}}`?** Dynamic-creative (DCO/DPA) ads are stored with template placeholders; Meta resolves them only when the ad is delivered. The actor keeps the served text untouched in `ad_text`, records the placeholder in `template_body`/`template_tokens`, and attaches `resolved_ad_text`/`resolved_title` when the payload's cards contain the resolved creative instead of inventing one.

**Is the total count exact?** It is the number the Ad Library page itself displays for your filters. Meta caps the display at 50,001, meaning "50,000 or more"; below that it is exact. Totals are not available for byline searches because the Ad Library UI has no byline filter.
