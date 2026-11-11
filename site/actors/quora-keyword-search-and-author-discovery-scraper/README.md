# Quora Keyword Search & Author Discovery Scraper

Collect public Quora questions, full answers, author profiles, and topics without cookies or logins. Paste question, answer, profile, or topic URLs, or use best-effort keyword discovery with related-question expansion. The default path is HTTP; a residential browser fallback is automatic only after Quora challenges the direct, datacenter, and residential HTTP paths.

## What this actor does

- **Three modes:** `urls` follows pasted Quora links, `search` discovers indexed question links and expands related questions, `authors` builds author profiles from profile or question URLs.
- **Full question context:** title, URL, topics, follower count, answer count, related questions, and the explicit unanswered state.
- **Full answers:** answer text, permalink, author name and profile link, credential, verified badge, upvotes, views, comment count, share count, exact creation date, AI label, and related-answer provenance.
- **Author profiles:** followers, following, answers, questions, posts, content views, join date, credential, location, employment, education, and expertise topics.
- **Honest discovery:** Quora's native search page is gated for anonymous visitors, so keyword mode uses a public web index and reports a clear status row when no indexed matches exist. Related-question expansion is the reliable discovery path.
- **HTTP first, browser last resort:** every page is fetched over HTTP first. Only if Quora challenges the direct, datacenter, and residential HTTP paths does the actor render that page in a real Chromium session through a residential connection. The same parser is used, so the output shape does not change.
- Leaves empty fields out of each row instead of writing guesses or nulls.

## Output per question

- `question_id`, `title`, `question_title`, `url`, `question_url`, `canonical_url`
- `topics[]`, `topic_urls[]`
- `answer_count`, `answer_count_text`, `answer_count_scraped`, `last_answered_at`
- `follow_count`, `follow_count_text`, `follower_count`
- `related_questions[]`, `related_questions_count`
- `question_language`, `answer_pagination_state`
- `discovery_source`, `source_query`, `parent_question_url`
- `access_state`, `scrape_timestamp`, `record_id`

## Output per answer

- `answer_id`, `answer_url`, `answer_text`, `word_count`
- `upvotes` / `upvotes_text`, `view_count` / `view_count_text`
- `comments_count` / `comments_count_text`, `shares_count` / `shares_count_text`
- `created_at`, `answer_created_at`, `answer_timestamp`, `answer_updated_at`
- `author_name`, `author_url`, `author_id`, `author_image_url`, `author_credentials`, `author_is_verified`, `author_is_anonymous`
- `is_ai_answer`, `ai_label_text`, `answer_visibility`, `answer_is_collapsed`
- `answer_is_related`, `source_question_url`, `source_question_title`
- `question_id`, `question_title`, `question_url`
- `media_urls`, `image_urls`, `video_urls`, `outbound_links` (optional)
- `answer_html` (optional), `discovery_source`, `source_url`, `access_state`

## Output per author

- `author_id`, `name`, `profile_url`, `avatar_url`, `credential`, `is_verified`
- `follower_count`, `following_count`, `answer_count`, `question_count`, `post_count`, `content_views`
- `joined_at`, `location`, `employment`, `education`
- `topics[]`, `bio`, `website_url`, `social_links[]`
- `evidence_urls[]`, `evidence_question_urls[]`, `evidence_answer_urls[]`, `evidence_answer_previews[]`

A direct profile URL in `urls` mode emits a `profile` row with the same profile fields. In `authors` mode it emits an `author` row.

## Output per topic

- `topic_id`, `name`, `url`, `follower_count`
- `discovery_source`, `source_url`, `access_state`

## Output per status / manifest

Status rows explain a page that could not be read: `access_state` is one of `not_found`, `removed`, `login_required`, `cloudflare_challenge`, `rate_limited`, `unavailable`, `unsupported_url`, or `discovery_unavailable`. A run-manifest row is written at the end of every run that starts.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `urls` | `urls`, `search`, or `authors` |
| `directUrls` | array | – (prefilled with one Python question) | Question, answer, profile, or topic URLs |
| `searchQueries` | array | – | Keywords or question titles for best-effort discovery (mode=search) |
| `usernames` | array | – | Profile slugs converted to profile URLs |
| `resultTypes` | array | `["question","answer"]` | `question`, `answer`, `profile`, `author`, `topic` |
| `includeAnswers` | boolean | `true` | Emit answers from question pages |
| `maxAnswersPerQuestion` | integer | `20` | Answer cap per question (0–500) |
| `includeRelatedQuestions` | boolean | `false` | Follow related-question links |
| `maxRelatedQuestions` | integer | `15` | Cap per question; 0 keeps all observed |
| `includeAuthorProfiles` | boolean | `true` | Enrich discovered authors with profile stats |
| `maxAuthors` | integer | `100` | Author-row cap |
| `minAnswerUpvotes` | integer | `0` | Drop answers below this explicit upvote count |
| `filterAiAnswers` | string | `include` | `include`, `exclude`, or `only` |
| `answerSortOrder` | string | `observed` | `observed`, `upvotes_desc`, or `newest` |
| `onlyUnanswered` | boolean | `false` | Keep only questions with an explicit zero answer count |
| `includeMedia` | boolean | `false` | Retain answer image/video links |
| `includeOutboundLinks` | boolean | `false` | Retain external links in answers |
| `includeAnswerHtml` | boolean | `true` | Retain a simple HTML rendering of answers |
| `includeProfileSensitiveFields` | boolean | `true` | Include location, employment, education, join date |
| `includeSocialLinks` | boolean | `true` | Include profile website/social links |
| `maxItems` | integer | `200` | Total real-row cap |
| `maxItemsPerSource` | integer | `0` | Per-URL/keyword cap (0 = no cap) |
| `includeFieldEvidence` | boolean | `true` | Add `field_sources` and `fields_present` |

The saved prefill is one question with `maxAnswersPerQuestion: 2` and `maxItems: 3`, so the first run returns three real rows.

## Examples

### A question with many answers

```json
{
  "mode": "urls",
  "directUrls": ["https://www.quora.com/What-is-Python-primarily-used-for"],
  "resultTypes": ["question", "answer"],
  "maxAnswersPerQuestion": 50
}
```

### A profile and its stats

```json
{
  "mode": "authors",
  "directUrls": ["https://www.quora.com/profile/Adam-DAngelo"],
  "resultTypes": ["author"]
}
```

### Expand related questions from one seed

```json
{
  "mode": "search",
  "directUrls": ["https://www.quora.com/What-is-machine-learning"],
  "includeRelatedQuestions": true,
  "maxRelatedQuestions": 10,
  "maxItems": 100
}
```

### Best-effort keyword discovery

```json
{
  "mode": "search",
  "searchQueries": ["remote work productivity"],
  "includeRelatedQuestions": true,
  "maxItems": 50
}
```

## Use cases

- **Market and product research** — collect complete answer threads with engagement metrics.
- **Expert sourcing** — turn answer authors into a profile list with follower and answer counts.
- **Content gap analysis** — find questions with zero answers in a niche.
- **AI-answer auditing** — separate explicitly AI-labelled answers from human answers.
- **Knowledge-base building** — export clean question/answer pairs with source URLs and dates.

## FAQ

### Does this actor need cookies or a login?

No. It reads public question, answer, profile, and topic pages without an account. There is no cookie or proxy field to configure.

### Why can't I search Quora by keyword reliably?

Quora's own search page is gated for anonymous visitors, and the actor never calls Quora's `/search?q=` endpoint. It uses a public web index for keyword discovery and expands related questions from any question it reaches. When no indexed matches exist, it writes a `discovery_unavailable` status row and does not invent results.

### How many answers can I get?

All answers that Quora exposes through its answer feed, up to `maxAnswersPerQuestion`. The actor follows the answer cursor until your cap or the end of the feed. Popular questions can contain thousands of answers; set a cap for the first run.

### Why is an answer marked as related?

Quora sometimes shows answers written for a merged or equivalent question. Those rows carry `answer_is_related: true` and `source_question_url` / `source_question_title`, so the answer is not misattributed to the page you opened.

### Why are some fields missing on some rows?

Quora only publishes a credential, location, employment, education, or bio for some users. Missing values are omitted, never guessed. Question creation dates and question view counts are not published on the public question page and are not emitted.

### Can I get comment text?

No. Quora publishes the comment count with each answer, but not the comment bodies on the public page. The actor returns the count only.

### What does "best-effort" mean for keyword discovery?

It means the actor tries a public index first and tells you when that index has no matches. It never re-labels related questions or unrelated pages as keyword search results.

### What happens if Quora challenges the page?

The actor first tries its fast HTTP path, then datacenter and residential HTTP. If all three are challenged, it renders that page in a real browser through its residential connection and parses the same data. If even that is blocked, the page is reported as a `cloudflare_challenge` status row. The browser fallback is slower, uses more memory, and incurs residential data transfer for the pages that need it; it only runs after direct, datacenter, and residential HTTP are challenged.

### Is this affiliated with Quora?

No. This is an independent, third-party actor that reads public Quora pages. It is not affiliated with or endorsed by Quora.

## Limitations

- Native Quora keyword search is unavailable anonymously; keyword discovery depends on a public web index.
- Comment bodies, question creation dates, question view counts, topic question feeds, and full profile answer lists are not available from the public page.
- Topic pages expose the topic name and follower count; descriptions and question counts are omitted when Quora does not publish them.
- Very large answer threads are bounded by `maxAnswersPerQuestion` and `maxItems`.
- A page that is removed, private, or behind a login wall is reported as a status row, not fabricated data.
- Challenge recovery uses a real browser as a last resort, so it is slower, uses more memory, and incurs residential data transfer for the pages that need it.
