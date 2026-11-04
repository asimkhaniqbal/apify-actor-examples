# Multi-Site Username Lookup & Profile Link Finder

Check whether a username exists across **400+ social networks and websites** in one run. Give it one or more usernames and get back every site where a matching profile was found, with the direct profile URL for each hit.

## What this actor does

- Checks one or more usernames against 400+ sites — GitHub, Twitter/X, Instagram, Reddit, TikTok, LinkedIn, YouTube, and hundreds more
- Returns a confirmed profile URL for every site where the username exists
- Processes multiple usernames in a single run, each with its own result
- Expands a `{?}` wildcard into common separator variations (`_`, `-`, `.`) so you can check several spellings of a handle at once
- No login or API key required — only publicly accessible pages are checked

## Output per username

- `username` — the username that was searched
- `links` — array of URLs, one per site where a matching profile was found

```json
{
  "username": "johndoe",
  "links": [
    "https://github.com/johndoe",
    "https://www.reddit.com/user/johndoe",
    "https://www.instagram.com/johndoe",
    "https://twitter.com/johndoe",
    "https://www.tiktok.com/@johndoe"
  ]
}
```

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `usernames` | array | – | Usernames to search for; supports a `{?}` wildcard for variations |

### Example: search two usernames

```json
{
  "usernames": ["johndoe", "alice_smith"]
}
```

### Example: wildcard variations of one handle

```json
{
  "usernames": ["john{?}doe"]
}
```

This expands to search `john_doe`, `john-doe`, and `john.doe`.

## Sites checked

Among the 400+ sites checked:

- **Social media:** Twitter/X, Instagram, Facebook, TikTok, Snapchat, Reddit, Mastodon, Bluesky
- **Professional:** LinkedIn, GitHub, GitLab, Stack Overflow, Kaggle, LeetCode
- **Content:** YouTube, Twitch, SoundCloud, Spotify, Medium, DeviantArt, Behance
- **Gaming:** Steam, Roblox, Chess.com, Lichess
- **Communication:** Discord, Telegram, Signal
- And 350+ more

## Use cases

- **OSINT investigations** — discover every online account associated with a username
- **Social media auditing** — map a person's or brand's public footprint across platforms
- **Username availability checks** — see which major platforms a handle is already taken on
- **Digital forensics** — locate accounts tied to a handle for security research
- **Brand monitoring** — track where a brand name or handle is being used

## FAQ

**How many sites get checked per username?**  Over 400 social, professional, content, gaming, and communication sites.

**How long does a run take?**  A single username typically takes 1-2 minutes to check; multiple usernames are processed one after another.

**Does it find private or deleted accounts?**  No — only publicly accessible profiles are reported. Private accounts, deleted accounts, and accounts with restricted visibility won't appear in results.

**What does the `{?}` wildcard do?**  It expands a username into three variants using common separators — underscore, hyphen, and period. `john{?}doe` becomes `john_doe`, `john-doe`, and `john.doe`.

**Can I check multiple usernames in one run?**  Yes — add as many as you like to the `usernames` array; each gets its own result with its own list of found profiles.

**Why might a site that actually has the username be missing from results?**  Some sites temporarily block automated requests when they suspect non-human traffic, which can cause that one site to be absent from a given run even though the username exists there.

**Does this require a login or API key?**  No — every site is checked using only publicly accessible information.

**Can a result be wrong?**  Occasionally. A small number of sites return a normal-looking page for a username that doesn't actually belong to anyone, or belongs to someone else, which can register as a false match. Treat results as a strong starting point and verify anything important by opening the link.
