# Indie Hackers Startup Revenue & Interview Scraper

Indie Hackers is where bootstrapped founders publish their products alongside self-reported monthly revenue, follower counts, and founder interviews. This actor collects that product catalog, the founder interview index, and community groups, or resolves specific product/interview/group URLs directly.

## What this actor does

- Browses the product catalog, with a revenue-ranked order available and an optional keyword filter (see the `sortBy` FAQ entry for the other sort options' current behavior).
- Lists founder interviews with publish dates and the product/founder they belong to.
- Lists community groups with member counts and descriptions.
- Resolves a list of specific Indie Hackers product, interview, or group URLs into the matching record type.
- Fields a product, interview, or group hasn't filled in (for example no disclosed revenue, or no Twitter handle) are left out of that record.

## Output per product

| Field | Description |
| --- | --- |
| `name` | Product name. |
| `slug` | Product's Indie Hackers ID. |
| `tagline` | One-line product tagline. |
| `description` | Longer product description. |
| `websiteUrl` | Product's own website. |
| `indiehackersUrl` | Link to the product's page on Indie Hackers. |
| `logoUrl` | Product logo/avatar image URL. |
| `monthlyRevenue` | Self-reported monthly revenue, when the founder discloses one. |
| `memberCount` | Number of followers on Indie Hackers. |
| `launchDate` | Reported launch date. |
| `twitterHandle` / `facebookUrl` | Social links, when provided. |
| `commitment` | Founder's stated commitment level (e.g. full-time, side project). |
| `employees` | Reported team size bracket. |
| `founders` | Founder count bracket tags. |
| `funding` | Funding status tags, when disclosed. |
| `revenueModel` | Revenue model tags (e.g. subscription, one-time). |
| `verticals` | Industry/vertical tags. |
| `platforms` | Platform tags (e.g. web, mobile). |
| `founderUsername` / `founderName` / `founderAvatarUrl` | The listed founder's Indie Hackers username, display name, and avatar. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Output per interview

| Field | Description |
| --- | --- |
| `title` | Interview title. |
| `slug` | Interview's Indie Hackers ID. |
| `interviewUrl` | Link to the interview on Indie Hackers. |
| `intervieweeSlug` / `intervieweeName` | Username and display name of the founder interviewed. |
| `productId` / `productUrl` | The product the interview is linked to, when one is set. |
| `publishedDate` / `createdDate` | When the interview was published and originally created. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Output per group

| Field | Description |
| --- | --- |
| `name` | Group name. |
| `slug` | Group's Indie Hackers ID. |
| `description` | Group description. |
| `groupUrl` | Link to the group on Indie Hackers. |
| `iconUrl` | Group icon image URL. |
| `memberCount` | Number of members in the group. |
| `createdDate` | When the group was created. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | String (select) | `products` | `products`, `interviews`, `groups`, or `byUrl`. |
| `sortBy` | String (select) | `revenue-desc` | Product sort order (mode=products): `revenue-desc` reliably ranks by revenue; `foundedDate-desc`, `memberCount-desc`, and `name-asc` currently return Indie Hackers' default catalog order rather than actually re-sorting (see FAQ). |
| `searchQuery` | String | – | Keyword filter for product name/tagline or group name/description. |
| `pageUrls` | Array of strings | – | Indie Hackers product, interview, or group URLs to resolve directly (mode=byUrl). |
| `maxItems` | Integer | 50 | Maximum number of items to return. |

### Example: top products by revenue

```json
{
  "mode": "products",
  "sortBy": "revenue-desc",
  "maxItems": 100
}
```

### Example: top-revenue products matching a keyword

```json
{
  "mode": "products",
  "searchQuery": "newsletter",
  "sortBy": "revenue-desc",
  "maxItems": 30
}
```

### Example: resolve specific pages

```json
{
  "mode": "byUrl",
  "pageUrls": [
    "https://www.indiehackers.com/product/example-product",
    "https://www.indiehackers.com/group/example-group"
  ]
}
```

## Use cases

- **Market researchers** tracking which bootstrapped products report the highest monthly revenue in a vertical.
- **VC and acquisition scouts** sourcing self-funded startups with disclosed traction before they raise.
- **Content curators** pulling founder interviews for newsletters or roundups.
- **Competitive analysts** monitoring how competitors describe their revenue model, team size and funding status.
- **Community managers** mapping Indie Hackers groups by member count and topic.

## FAQ

**Does this need an Indie Hackers account?**
No. It reads the same public product, interview and group listings shown to any visitor.

**Why is `monthlyRevenue` missing from some products?**
Revenue figures on Indie Hackers are self-reported by founders; when a founder hasn't disclosed one, the field is left out rather than shown as zero.

**Does the interview record include the full interview text?**
No, it returns the interview's metadata (title, dates, interviewee, linked product) rather than the transcript body.

**What's the difference between `founders`, `employees`, and `commitment`?**
These come from separate tags the founder sets on their product: `founders` is the founder-count bracket, `employees` is the team-size bracket, and `commitment` is whether they work on it full-time or as a side project.

**Does `sortBy` actually change the order products come back in?**
Only `revenue-desc` is currently guaranteed — it pulls from Indie Hackers' own revenue-ranked index. `foundedDate-desc`, `memberCount-desc`, and `name-asc` are accepted but currently return results in the catalog's default order rather than re-sorting by date, member count, or name.

**Can I look up a specific product, interview or group I already know the URL for?**
Yes, use `mode: "byUrl"` with those URLs in `pageUrls`; the actor detects whether each URL is a product, interview, or group.

**Is this affiliated with Indie Hackers?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by Indie Hackers or Stripe.
