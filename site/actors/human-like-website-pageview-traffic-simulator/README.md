# Human-Like Website Pageview Traffic Simulator

Send real Chromium pageviews to a website you own or are authorized to test. Choose how many pages to load, how deep each visit goes, how long it stays, whether it scrolls or follows internal links, which device and country it appears from, and which referrer it reports. Every attempt is written as a dataset row with its measured status, timing, device, referrer, analytics requests and error. A run-level summary is checkpointed while the run progresses, and a graceful stop writes an `aborted` row for an in-flight page.

This is a testing and monitoring tool. It does not solve CAPTCHAs, click ads, inflate YouTube views, or promise that the visits will appear in Google Analytics or improve rankings. Use it on your own sites or sites you have permission to test.

## What this actor does

- **Real browser visits:** each page loads in Chromium with JavaScript, cookies, redirects and lazy content working normally.
- **Pageview volume:** request 1–1,000 pageviews; each visit gets its own browser context, cookies and device profile.
- **Session depth:** attempt to follow one visible same-site link per page, up to 10 pages per visit. A visit can stay on one page when no link is followed or the global pageview budget is used.
- **Human behaviour:** a jittered dwell target, mouse movement, measured scroll depth and optional internal clicks. A failed click attempt can extend the measured time on a page.
- **Device, referrer and geo controls:** desktop, Android mobile, Android tablet or a mixed run; direct, search, social or custom referrer; optional residential country for geo QA.
- **Evidence, not claims:** per-page HTTP status, final URL, load time, actual dwell, actual scroll, links followed, analytics requests observed and a clear error code. A `SUMMARY` record is written to the key-value store and checkpointed during the run.

## Output per pageview

- `url` — the target URL you asked for (before redirects or UTM tagging)
- `finalUrl` — the final URL after redirects
- `status` — `success`, `failed`, `blocked`, `timeout` or `aborted`
- `statusCode` — HTTP response status
- `pageTitle` — HTML title, omitted for non-HTML pages
- `loadTimeMs` — time until the DOM was ready
- `timeOnPageSec` — actual time spent on the page after loading, including click attempts
- `scrollDepthPercent` — measured scroll depth, omitted when scrolling is off or no range is measurable; a very short page with a tiny overflow can report 100
- `linksFound` — same-site links present in the page, not only visible ones
- `linksFollowed` — internal links followed from this page
- `referrer` — the actual `document.referrer` seen by the page
- `userAgent`, `viewport`, `device` — the browser profile used
- `visitId`, `pageviewIndex`, `pageIndex` — traceability for sessions and loops
- `analyticsTags` — analytics requests observed, e.g. `ga4`, `gtm`, `meta_pixel`, `hotjar` (not proof that the platform accepted a hit)
- `errorCode`, `error` — a short code and message for failed, blocked or timed-out pages
- `timestamp` — ISO-8601 UTC time the page attempt started

Empty or unmeasured fields are omitted, not filled with empty strings or fake zeros. A successful page has no `errorCode`. A graceful stop writes `aborted` for pages that have not yet emitted a row; a hard kill can still omit an in-flight row, while pages already written stay `success`.

A `SUMMARY` record contains requested/attempted pageviews, success/failed/blocked/timeout/aborted counts, error rate, median and p95 load time, median dwell, analytics-request rates and per-target counts. It is checkpointed during the run, so a stopped run still has the latest version.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `targetUrls` | array | – | Public pages you own or are authorized to test (the prefill uses `https://example.com`); legacy `startUrls` is accepted in code |
| `pageviews` | integer | `10` | Total page loads to attempt (1–1,000) |
| `pagesPerVisit` | integer | `1` | Maximum pages in one visit (1–10); above 1 attempts same-site link clicks when clicks are enabled and the global pageview budget allows |
| `visitStyle` | string | `human` | `human` for dwell/scroll/click, `quick` for a fast load check |
| `dwellSeconds` | integer | `20` | Target seconds per page for human visits (1–300) |
| `referrerMode` | string | `direct` | `direct`, `search`, `social` or `custom` |
| `customReferrer` | string | – | Full http(s) URL used when `referrerMode` is `custom` |
| `scroll` | boolean | `true` | Scroll during human visits; measured depth is omitted if not measurable |
| `clickInternalLinks` | boolean | `true` | Attempt one visible same-site link per page; never ads, forms or external links |
| `deviceMix` | string | `mixed` | `desktop`, `mobile`, `tablet` or `mixed` (Android Chromium emulation for mobile/tablet) |
| `visitCountry` | string | `none` | Optional ISO country for residential-proxy geo QA; `none` keeps visits direct |
| `blockAds` | boolean | `true` | Block ad domains; analytics requests still load so tag firing can be measured |
| `blockImages` | boolean | `false` | Skip images, fonts and media for faster visits |
| `utmSource` | string | – | Optional `utm_source` label appended to target URLs, e.g. `traffic-test` |

## Example: load check on your own site

```json
{
  "targetUrls": [
    { "url": "https://www.example.com/" },
    { "url": "https://www.example.com/pricing" }
  ],
  "pageviews": 20,
  "pagesPerVisit": 1,
  "visitStyle": "human",
  "dwellSeconds": 15,
  "deviceMix": "mixed",
  "referrerMode": "direct"
}
```

## Example: analytics-tag QA with a test-source label

```json
{
  "targetUrls": [{ "url": "https://www.example.com/" }],
  "pageviews": 10,
  "pagesPerVisit": 2,
  "visitStyle": "human",
  "dwellSeconds": 25,
  "scroll": true,
  "clickInternalLinks": true,
  "referrerMode": "search",
  "utmSource": "traffic-test",
  "blockAds": true
}
```

Use the `analyticsTags` field and the `SUMMARY` rates to confirm which analytics requests the page made. Test traffic may still be filtered by an analytics platform; this actor does not guarantee attribution.

## Example: mobile and geo QA

```json
{
  "targetUrls": [{ "url": "https://www.example.com/" }],
  "pageviews": 12,
  "pagesPerVisit": 1,
  "visitStyle": "human",
  "dwellSeconds": 10,
  "deviceMix": "mobile",
  "visitCountry": "DE",
  "referrerMode": "social"
}
```

Country visits use a residential proxy in that country. If the proxy is unavailable, the page is reported as failed instead of silently falling back to a direct request.

## Example: quick smoke test

```json
{
  "targetUrls": [
    { "url": "https://www.example.com/" },
    { "url": "https://www.example.com/health" }
  ],
  "pageviews": 2,
  "visitStyle": "quick",
  "blockAds": true,
  "blockImages": true
}
```

## Use cases

- **Load and cache warm-up** — load key pages after a deploy with a realistic browser and short dwell.
- **Analytics and tag QA** — check whether GA4, GTM, Meta Pixel, Hotjar or other tags actually send data on each page.
- **Mobile checks** — load pages with Android mobile or tablet profiles and compare status, title and timing.
- **Geo checks** — request visits through a residential country to test redirects, consent banners, prices or language.
- **Uptime and broken-link checks** — schedule a small run and watch for `failed`, `blocked`, `timeout` or non-2xx statuses.

## FAQ

### Do I need a proxy?

No, for new inputs. Visits are direct by default. Set `visitCountry` only when you need a residential exit in a specific country for geo testing; that option adds proxy usage. Old saved inputs that still contain `proxyConfiguration` can also create a proxy; if no group is set, the account's residential group is used and billed. Remove that legacy key for direct visits.

### Will these visits show up in Google Analytics or other analytics tools?

Not necessarily. The actor loads the page in a real browser and records which analytics requests fired, but analytics platforms filter automated traffic and may exclude the visits. Use this actor to test whether tags work, not to inflate audience numbers.

### Why did a page report `blocked`?

The site answered with 401, 403 or 429, or served a recognizable bot-challenge page. A 403 is reported as `blocked`, a 404 as `failed` with `errorCode=not_found`, and 5xx as `failed` with `errorCode=server_error`; the HTTP status is retained even when the body is empty. The actor does not solve CAPTCHAs and does not try to bypass the block. Change the target, test a staging URL, or use a country proxy where appropriate.

### Why is `scrollDepthPercent` missing on some successful pages?

The page had no measurable scrollable range, scrolling was turned off, or the depth could not be measured. A page with no range is never forced to 100%, but a very short page with a few pixels of overflow can still report 100% because a wheel step reaches its bottom.

### Why is the actual dwell or page count different from what I set?

`dwellSeconds` is a target, not a hard wall. Waiting for an internal click to navigate can make the measured `timeOnPageSec` longer, and `pagesPerVisit` is a maximum: the run's total pageview budget can end a visit before it reaches that maximum. Both the actual dwell and the actual pages are recorded per visit.

### Does it click ads or inflate YouTube views?

No. It only attempts visible same-site links; ad elements, forms and external links are skipped. YouTube URLs are treated as ordinary pages, and the actor does not generate views.

### What happens if the run is stopped early?

Rows already written stay in the dataset. A graceful stop writes `aborted` rows for in-flight visits, writes the latest `SUMMARY`, and sets a partial status such as `Stopped 3/20`. A hard kill can still omit the in-flight row, but the summary is checkpointed after each page.

### Are my saved inputs from the old version still supported?

Yes. An input with only `startUrls` is accepted and mapped to `targetUrls`. Legacy `waitOnPage`, `mode=STRESS`, `enableCrawling`/`maxPagesPerUrl`, `enableAdvancedFingerprinting`, `crawlingLinkSelector` and `blockResources` are applied in code, including when the platform supplies the new defaults. `endAfterSeconds` and YouTube mode are ignored with a warning; the run stops after the requested pageviews.

### Is this affiliated with the websites I test?

No. It is an independent browser automation tool. Use it only on sites you own or have permission to test, and follow their terms and your local laws.

## Limitations

- No login, basic-auth, form submission or file upload.
- No CAPTCHA solving or guarantee of bypassing bot protection.
- No ad clicks, ad impressions or YouTube view generation.
- Mobile and tablet profiles use Android Chromium emulation, not iOS Safari.
- Direct visits come from the actor's network; some sites will block them. Country visits depend on residential proxy availability.
- Analytics platforms may filter automated traffic; `analyticsTags` records requests the page made, not hits the platform accepted.
- Target URLs and redirect destinations are checked against the private/loopback guard before navigation.
- A hard kill can omit the in-flight page row; graceful stops write `aborted` and a checkpointed summary.
- The actor does not claim to improve SEO rankings or audience metrics.
