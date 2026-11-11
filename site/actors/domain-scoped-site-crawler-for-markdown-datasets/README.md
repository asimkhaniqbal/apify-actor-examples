# Domain-Scoped Site Crawler for Markdown Datasets

Give it one or more start URLs and it crawls the site the way a documentation index should be crawled: it follows same-domain links, isolates the main content of every page, and returns clean **Markdown**, **text**, or **HTML** alongside per-page metadata — title, description, author, publish dates, canonical URL, links, and image URLs. Built for LLM training sets, RAG pipelines, knowledge bases, and content migrations.

## What this actor does

- **Fetches pages with the right engine automatically.** Every page starts with fast raw HTTP; hosts that reject plain clients get a browser TLS fingerprint (`curl_cffi`), and pages that render only with JavaScript are loaded in a headless browser — no engine choice needed.
- **Finds more pages.** Follows links up to your depth limit, and can also enqueue URLs from `sitemap.xml` / robots.txt sitemaps and from `/llms.txt`.
- **Respects robots.txt** by default: discovered URLs that the site disallows are skipped (start URLs are always crawled).
- **Extracts the meaningful content.** An extraction cascade (Trafilatura → Readability → basic cleanup) removes navigation, footers, sidebars, ads, and cookie dialogs, while preserving headings, tables, lists, code blocks, and links.
- **Prepares content for RAG.** Every record carries an approximate token count, and you can switch on heading-aware `chunks` (code blocks and tables never split, context-preserving overlap) that drop straight into an embedding pipeline.
- **Returns rich metadata per page**: canonical URL, author, published/modified dates, keywords, OpenGraph, JSON-LD, image URLs, page links, word count, and a content hash.
- **Handles PDFs**: when a crawled URL serves a PDF, its text is extracted into the record as a file row — never raw binary.
- **Stores Markdown and plain-text files**: pages served as `text/markdown` or plain text (for example the `.md`/`.txt` files listed in an `llms.txt`) are stored with their content in `text` and `markdown`, and their links are followed when depth allows.
- **Skips non-pages cleanly**: RSS/XML feeds, JSON, images, archives and other binary responses are skipped with a reason (see `storeSkippedUrls`).
- **Supports incremental re-crawls**: `crawlOnlyPagesChangedSince` produces a delta of changed pages using HTTP validators, sitemap `lastmod`, and content hashes.
- **Cuts crawl waste**: canonical duplicates are skipped, pages are pushed as they finish, retries and waits are bounded per URL, and an adaptive per-domain pacer slows down (bounded) only when a host actually throttles.
- **Handles late and lazy content**: a bounded wait after load lets client-rendered pages finish, and pages that declare lazy images or infinite scroll are scrolled automatically (bounded) so that content is captured.
- **Optional screenshots**: every stored page can be rendered in the browser and captured as a full-page JPEG with a public URL.
- **Optional request headers**: send extra headers (e.g. a token for a public API or a specific User-Agent) on every request; cookie and transport headers are refused, so the actor stays cookieless.
- **Works cookieless** for any public website — no logins, no API keys.

## Output per page

Every record is pushed with empty values omitted (no `null`, no empty strings). A page record contains:

| Field | Description |
|---|---|
| `url` | URL that was requested |
| `loadedUrl` | Final URL after redirects |
| `title` | OpenGraph title, JSON-LD headline, or `<title>` |
| `description` | Meta description / OpenGraph description |
| `languageCode` | Language from `lang`, OpenGraph locale, or hreflang |
| `author` | JSON-LD author, meta author, citation author, or byline |
| `publishedDate` | Normalized ISO date: JSON-LD `datePublished`, `article:published_time`, `citation_date`, or `<time datetime>` |
| `modifiedDate` | Normalized ISO date: JSON-LD `dateModified`, `article:modified_time`, or `last-modified` |
| `canonicalUrl` | `<link rel="canonical">`, `og:url`, or JSON-LD URL |
| `keywords` | Meta keywords or JSON-LD keywords, as an array |
| `imageUrls` | Every image URL found (OpenGraph, Twitter, `<img>`), absolute and unique |
| `links` | Every link found on the page, absolute and unique |
| `openGraph` | Raw `og:*` and `twitter:*` meta tags as an object |
| `jsonLd` | Parsed `application/ld+json` blocks (including `@graph`) |
| `text` | Clean plain text of the page content |
| `markdown` | Content as Markdown (only when `outputFormat` is `markdown`) |
| `html` | Cleaned content HTML (only when `outputFormat` is `html`) |
| `wordCount` | Word count of `text` |
| `tokenCount` | Approximate LLM tokens in the returned content (characters ÷ 4) |
| `chunks` | Heading-aware Markdown pieces for embeddings (only with `chunkMarkdown`); each has `index`, `text`, `charCount`, and `headingPath` when the piece sits under headings |
| `screenshot` | Public URL of the page's full-page JPEG (only with `takeScreenshots`) |
| `contentHash` | SHA-256 of the normalized text — use it to detect unchanged pages |
| `depth` | Crawl depth (`0` = start URL) |
| `httpStatusCode` | HTTP status of the response |
| `loadedTime` | ISO 8601 UTC timestamp when the page was fetched |
| `referrerUrl` | URL of the page that linked here (start URLs have none) |
| `recordType` | `page` for HTML pages, `file` for extracted documents |
| `fileType` | `pdf` on file records |
| `pageCount` | Number of pages, on PDF records |

### Example output row (truncated)

```json
{
    "url": "https://example.com/blog/release-notes",
    "loadedUrl": "https://example.com/blog/release-notes",
    "title": "Release notes — Example",
    "description": "What changed in the latest release.",
    "author": "Jane Developer",
    "publishedDate": "2026-09-12T00:00:00Z",
    "canonicalUrl": "https://example.com/blog/release-notes",
    "languageCode": "en",
    "imageUrls": ["https://example.com/media/hero.png"],
    "links": ["https://example.com/blog", "https://example.com/download"],
    "text": "What changed in the latest release...",
    "markdown": "## What changed\n\n- Faster startup\n",
    "depth": 1,
    "httpStatusCode": 200,
    "loadedTime": "2026-10-16T12:00:05.123Z",
    "referrerUrl": "https://example.com/blog",
    "wordCount": 412,
    "contentHash": "8f2c1d…",
    "recordType": "page"
}
```

PDF record example:

```json
{
    "url": "https://example.com/whitepaper.pdf",
    "loadededUrl": "https://example.com/whitepaper.pdf",
    "title": "Whitepaper 2026",
    "text": "Executive summary ...",
    "fileType": "pdf",
    "recordType": "file",
    "wordCount": 5321
}
```

## Input

Picking a fetch engine, parallelism, timeouts, retries, rate limits, and proxies are auto-decided per page and per domain — no tuning needed.

| Field | Type | Default | Description |
|---|---|---|---|
| `startUrls` | array | – | Pages where the crawl begins. Required. Accepts URL objects or plain URL strings. |
| `maxCrawlDepth` | integer | `10` | How many links deep to follow (`0` = start URLs only). |
| `maxCrawlPages` | integer | `20` | Total pages and files to store; the crawl stops at this limit. |
| `useSitemaps` | boolean | `false` | Enqueue URLs from robots.txt sitemaps and `/sitemap.xml` (sitemap indexes are followed). |
| `useLlmsTxt` | boolean | `false` | Enqueue the files linked from `/llms.txt`. |
| `respectRobotsTxtFile` | boolean | `true` | Skip discovered URLs disallowed by robots.txt. Start URLs are always crawled. |
| `crawlOnlyPagesChangedSince` | string | – | Date (`YYYY-MM-DD` or full ISO) for incremental crawls; pages unchanged since then are not stored and the output is a delta. |
| `includeUrlGlobs` | array | `[]` | Only crawl discovered URLs matching these glob patterns. |
| `excludeUrlGlobs` | array | `[]` | Skip discovered URLs matching these glob patterns. |
| `includeUrlRegexps` | array | `[]` | Only crawl discovered URLs matching at least one regex. |
| `excludeUrlRegexps` | array | `[]` | Skip discovered URLs matching any regex. |
| `keepUrlFragments` | boolean | `false` | Treat `#fragment` variants as separate pages (useful for fragment-routed SPAs). |
| `ignoreCanonicalUrl` | boolean | `false` | When off, a page whose canonical URL was already crawled is skipped as a duplicate. |
| `outputFormat` | string | `markdown` | `markdown`, `text`, or `html`. |
| `htmlTransformer` | string | `auto` | `auto` picks Trafilatura or Readability per page; `trafilatura`, `readability`, `basic`, and `none` force a strategy. |
| `includeImages` | boolean | `false` | Add image references to Markdown (`imageUrls` is filled regardless). |
| `keepElementsCssSelector` | string | – | Keep only matching elements before extraction (e.g. `article, main`). |
| `removeElementsCssSelector` | string | – | Remove matching elements before extraction (e.g. `.sidebar, .ad`). |
| `parsePdfText` | boolean | `true` | Extract text from linked PDFs into file records instead of skipping them. |
| `chunkMarkdown` | boolean | `false` | Emit a `chunks` array per page for RAG/embeddings. |
| `chunkMaxChars` | integer | `2000` | Target chunk size in characters (200–10000); code blocks and tables stay whole. |
| `chunkOverlapChars` | integer | `200` | Overlap between consecutive chunks (0–1000). |
| `takeScreenshots` | boolean | `false` | Render every stored page in the browser, store a full-page JPEG, and return its URL in `screenshot`. |
| `customHttpHeaders` | array | `[]` | Extra request headers as `{"key", "value"}` pairs, applied to every engine. Cookie/transport headers are refused. |
| `clickElementsCssSelector` | string | – | Click these elements first (e.g. `[aria-expanded="false"]`) to expand collapsed content. |
| `storeSkippedUrls` | boolean | `false` | Save a `SKIPPED_URLS` key-value record with skipped URLs grouped by reason. |

### Examples

Crawl a documentation site to Markdown with metadata:

```json
{
    "startUrls": [{ "url": "https://docs.example.com" }],
    "maxCrawlPages": 500,
    "maxCrawlDepth": 5,
    "outputFormat": "markdown"
}
```

Only one section of a site, using glob and regex filters:

```json
{
    "startUrls": [{ "url": "https://example.com" }],
    "includeUrlGlobs": ["https://example.com/docs/*"],
    "excludeUrlRegexps": ["/v[0-9]+/", "/changelog/"]
}
```

Weekly incremental re-crawl (delta of changed pages):

```json
{
    "startUrls": [{ "url": "https://example.com/blog" }],
    "useSitemaps": true,
    "crawlOnlyPagesChangedSince": "2026-10-01"
}
```

Find pages through the sitemap only, without following links:

```json
{
    "startUrls": [{ "url": "https://example.com" }],
    "useSitemaps": true,
    "maxCrawlDepth": 0,
    "maxCrawlPages": 2000
}
```

Crawl a documentation site straight into embedding-ready chunks:

```json
{
    "startUrls": [{ "url": "https://docs.example.com" }],
    "maxCrawlPages": 500,
    "chunkMarkdown": true,
    "chunkMaxChars": 1500,
    "chunkOverlapChars": 200
}
```

## Use cases

- **LLM training and fine-tuning data** — turn documentation, blogs, and help centers into structured Markdown with source metadata
- **RAG pipelines and semantic search** — per-page content with optional heading-aware `chunks` and `tokenCount`, plus canonical URLs, dates, and hashes for clean indexing and re-indexing
- **Knowledge-base and chatbot grounding** — keep a site's content current with scheduled incremental crawls
- **Content migration and archival** — export a whole site to Markdown while keeping authorship and publication dates
- **Competitive and market research** — compare content across sites with consistent fields per page

## FAQ

**Do I need to choose a crawler engine?**
No. The actor starts every page with fast raw HTTP, retries blocked hosts with a browser TLS fingerprint (`curl_cffi`), and loads pages that render only with JavaScript in a headless browser. Parallelism, timeouts, retries, and pacing are also chosen automatically per page and per domain, so the input surface only contains options that change which data you get.

**How does the extraction cascade pick content?**
With `htmlTransformer` set to `auto`, the actor runs Trafilatura and Readability on each page, keeps the candidate that preserves the most structure (headings, tables, lists, images), and falls back to full basic-cleaned content when both would drop most of the page. Use `none` to keep the page almost untouched.

**Does it respect robots.txt?**
Yes, `respectRobotsTxtFile` is on by default: discovered URLs that robots.txt disallows are skipped and listed under the `robots` skip reason. Start URLs are always crawled. Crawl-delay directives are not applied; the adaptive per-domain pacer keeps request rates polite automatically.

**How do sitemaps work?**
With `useSitemaps` enabled, the actor reads the `Sitemap:` entries from robots.txt and `/sitemap.xml`, follows sitemap indexes, and enqueues matching same-domain URLs at depth 1. Combined with `maxCrawlDepth: 0` this crawls exactly the sitemap's URL set.

**What happens with JavaScript-heavy sites?**
Pages whose raw HTML is an empty SPA shell are automatically rendered in a headless browser. After load the actor waits a bounded couple of seconds for late content, waits again briefly if the DOM still looks empty, and scrolls (bounded) only when the page declares lazy images or infinite scroll, so client-rendered and lazy content is captured. `clickElementsCssSelector` expands collapsed sections when you need it.

**What happens with PDFs, feeds, and other files?**
PDFs are converted to text and stored as `recordType: "file"` rows with `fileType: "pdf"`. Markdown and plain-text files (`text/markdown`, `text/plain`) are stored as pages with their raw content in `text`/`markdown` (JSON/XML payloads served as text are still skipped). RSS/XML feeds, JSON, and binary responses (images, archives) are not stored as pages; each skip is counted and visible in the run log, and in the `SKIPPED_URLS` record when `storeSkippedUrls` is enabled.

**How do incremental crawls work?**
Set `crawlOnlyPagesChangedSince` to a date. The actor keeps ETag/Last-Modified validators and content hashes in a persistent key-value store, filters sitemap entries by `lastmod`, and stores only pages that changed — the output is a delta, not a full snapshot. Keep previous results if you need the complete site.

**How does chunking work?**
With `chunkMarkdown` on, each page also gets a `chunks` array: heading-aware Markdown pieces up to `chunkMaxChars` characters (200–10000) with `chunkOverlapChars` (0–1000) of overlap so context survives the boundaries. Code fences and tables are never split, every piece knows its `headingPath` (`Guide > Install > Linux`) and `charCount`, and the pieces are in reading order. `tokenCount` on the page is an approximation (characters ÷ 4) — good for budgeting, not a tokenizer.

**Can I take screenshots?**
Yes, with `takeScreenshots`. Every stored page is rendered in the headless browser and gets a full-page JPEG (quality 80) in the run's key-value store; the record's `screenshot` field is its public URL. Expect roughly 0.3–1 MB and a fraction of a second per page; the bytes are held only until the page is pushed.

**Can I send custom headers?**
`customHttpHeaders` accepts a list of `{"key", "value"}` pairs and applies them to page requests and to robots.txt/sitemap/llms.txt lookups on every engine — useful for public APIs that want a token or sites that ask for a contact User-Agent. Cookie and transport/framing headers (`Cookie`, `Host`, `Content-Length`, `Connection`, ...) are refused on purpose: the actor never carries logins.

**What happens when a site throttles the crawl?**
The actor watches each domain: after connection timeouts or 429/503 responses it halves the parallel requests for that domain (down to 1) and adds a small delay (up to 2 seconds), then heals as pages succeed again. Throttled pages are requeued a bounded number of times, so a hostile host can't stall the run.

**Which fields are filled for every page?**
Content-derived fields (`url`, `loadedUrl`, `depth`, `httpStatusCode`, `loadedTime`, `recordType`, and one of `text`/`markdown`/`html`) are always present. Metadata fields (`author`, `publishedDate`, `keywords`, and so on) appear only when the page declares them, and empty values are never pushed.

**When do results appear in the dataset?**
Pages are pushed as soon as they finish, so you can watch results grow while the crawl is running. If a run is interrupted, everything already stored is kept.

**Are image and link URLs usable directly?**
`imageUrls` and `links` contain absolute `http(s)` URLs exactly as the page declares them. No rehosting is done.

**Can it crawl pages behind a login?**
No. The actor only works with publicly accessible pages and never requires credentials.

**What is out of scope?**
Login-protected content and AI summaries are intentionally not implemented — crawling behind a login needs credentials this actor refuses to require, and summaries need an external LLM key. Chunking and token counting are built in (`chunkMarkdown`, `tokenCount`); for anything more, the dataset stays atomic per page with a `contentHash` you can diff.

**Are the links in Markdown absolute?**
Yes. Relative links and images are resolved against the page URL before converting to Markdown.

**Is this affiliated with the websites it crawls?**
No. This is an independent, third-party actor that crawls publicly accessible pages; it isn't operated by or affiliated with any site it's pointed at. Check the target site's terms before scraping.
