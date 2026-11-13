# RescueGroups.org Adoptable Pet Finder by ZIP

RescueGroups.org brings together adoptable animals from thousands of shelters and rescues across the United States and Canada. This actor searches that catalogue the way a Petfinder-style site does — **ZIP + radius, species, breed, sex, age, size, and temperament** — then opens each animal's public page for the shelter contact, description, photos and personality flags.

It also looks up specific animals by ID and fetches rescue organizations by ID. **No cookies, login, or API key are required.** An optional free RescueGroups.org API key adds API-only fields and organization name search.

## Modes

| Mode | What it does |
|---|---|
| `search` (default) | Finds adoptable pets by ZIP/radius and the filters below |
| `byAnimalIds` | Fetches the animals whose IDs (or animal-page URLs) you paste |
| `orgSearch` | Fetches rescue organizations by ID; with an optional API key it can also search organizations by name |

## What you get per animal

- **Identity:** `animalId`, `name`, `status`, `species`, `breedPrimary`, `breedSecondary`, `breedString`
- **Profile:** `sex`, `ageGroup`, `sizeGroup`, `sizePotential`, `colors`, `pattern`, `coatLength`
- **Temperament and care:** `isAltered`, `isHousetrained`, `isCurrentVaccinations`, `isMicrochipped`, `isDeclawed`, `isSpecialNeeds`, `isNeedingFoster`, `isCatsOk`, `isDogsOk`, `isKidsOk`, `isAdultsOk`, `isYardRequired`, `isCrateTrained`, `activityLevel`, `energyLevel`, `exerciseNeeds`, `groomingNeeds`, `indoorOutdoor`, `newPeopleReaction`, `obedienceTraining`, `ownerExperience`, `sheddingLevel`, `vocalLevel`
- **Story:** `descriptionText`, `descriptionHtml`, `adoptionArea`
- **Shelter:** `organizationId`, `organizationName`, `organizationAddress`, `organizationCity`, `organizationState`, `organizationPostalCode`, `organizationEmail`, `organizationPhone`, `organizationUrl`
- **Location:** `locationCity`, `locationState`, `distanceMiles` (radius searches)
- **Links and metadata:** `pictures[]`, `pictureThumbnailUrl`, `sourceUrl`, `adoptionUrl`, `recordType`, `scrapedAt`

With an optional API key the records can also include the API-only fields: `searchString`, `rescueId`, `ageString`, `birthDate`, `isBirthDateExact`, `sizeCurrent`, `colorDetails`, `adoptionFeeString`, `distinguishingMarks`, `specialNeedsDetails`, `qualities`, `summary`, `organizationIds`, `locationCountry`, and the full temperament set.

## What you get per organization

`organizationId`, `name`, `about`, `services`, `adoptionProcess`, `adoptionArea`, `address`, `city`, `state`, `postalcode`, `country`, `phone`, `email`, `url`, `facebookUrl`, `donationUrl`, `adoptionUrl`, `isCommonApplicationAccepted`, `sourceUrl`, `recordType`, `scrapedAt`.

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `search` | `search`, `byAnimalIds`, or `orgSearch` |
| `apiKey` | secret | – | Optional free API key; adds API-only fields and org name search |
| `species` | string | `Dog` | Full species list, or `(any species)` |
| `breed` | string | – | Breed name, e.g. `Labrador Retriever` (matches any breed in the animal's breed string) |
| `sex` | string | – | `(any)`, `Male`, `Female` |
| `ageGroup` | string | – | `(any)`, `Baby`, `Young`, `Adult`, `Senior` |
| `sizeGroup` | string | – | `(any)`, `Small`, `Medium`, `Large`, `X-Large` |
| `specialNeedsOnly` | boolean | `false` | Only special-needs animals |
| `goodWithDogsOnly` | boolean | `false` | Only animals flagged good with dogs |
| `goodWithCatsOnly` | boolean | `false` | Only animals flagged good with cats |
| `goodWithKidsOnly` | boolean | `false` | Only animals flagged good with kids |
| `needsFosterOnly` | boolean | `false` | Only animals looking for a foster home |
| `housetrainedOnly` | boolean | `false` | Only house-trained animals |
| `declawedOnly` | boolean | `false` | Only declawed animals |
| `postalCode` | string | – | ZIP/postal code for a radius search; empty searches everywhere |
| `radiusMiles` | integer | `50` | Radius in miles (1–500; requests under 25 use 25, values between buckets round down, and requests above 200 use 200) |
| `includeDetails` | boolean | `true` | Open each animal's detail page (richer, slightly slower) |
| `animalIds` | array | `[]` | Animal IDs or page URLs for `mode=byAnimalIds` |
| `organizationIds` | array | `[]` | Organization IDs for `mode=orgSearch` |
| `orgQuery` | string | – | Organization name (needs the optional key) or a numeric ID / organization-page URL (keyless; legacy inputs supported) |
| `maxItems` | integer | `50` | Hard cap on emitted records (1–2000) |

### Example: dogs within 25 miles of Boston

```json
{
  "mode": "search",
  "species": "Dog",
  "postalCode": "02108",
  "radiusMiles": 25,
  "maxItems": 50
}
```

### Example: senior cats, good with kids

```json
{
  "mode": "search",
  "species": "Cat",
  "ageGroup": "Senior",
  "goodWithKidsOnly": true,
  "maxItems": 30
}
```

### Example: one breed, nationwide

```json
{
  "mode": "search",
  "species": "Dog",
  "breed": "Labrador Retriever",
  "maxItems": 100
}
```

### Example: look up animals by ID

```json
{
  "mode": "byAnimalIds",
  "animalIds": ["22514067", "22813117"]
}
```

### Example: fetch a rescue organization

```json
{
  "mode": "orgSearch",
  "organizationIds": ["6260"]
}
```

## How it works

- **Default, credential-free path.** The actor reads the public National Pet Search toolkit pages (`toolkit.rescuegroups.org`), the same pages RescueGroups.org serves to any website visitor. Search pages return 24 animals at a time; the actor paginates until `maxItems` or the result set ends. Each animal's public detail page supplies the profile, shelter contact, description and photos.
- **Optional API path.** Paste a free RescueGroups.org public API key and the actor uses the official v5 JSON:API instead. It returns the same records plus API-only fields and supports organization name search. No key is required for any core mode.
- **Filters are source-side.** Species, breed, sex, age, size, ZIP/radius, special-needs, foster, house-trained, declawed and good-with filters are applied by the source, not discarded afterwards.
- **Pagination is honest.** The actor stops on `maxItems`, source exhaustion, or a short page — it never cuts a run short with a wall-clock timer.
- **Empty values are omitted.** A field the source does not expose for a particular animal is left out, never sent as `null`, `""`, `[]` or a placeholder.

## FAQ

**Do I need an API key?** No. The default path needs no key, account, cookie or proxy. The optional key is free and only adds fields and organization name search.

**Why is the radius rounded?** The public search page offers 25, 50, 100 and 200 mile buckets. The actor picks the closest supported bucket: values between buckets round down, requests under 25 use the 25-mile minimum, and requests above 200 use the 200-mile maximum instead of widening to a nationwide search.

**Why did a breed return nothing?** Breed names are shelter-entered. Use the exact name from the RescueGroups breed list, optionally prefixed with the species (`Dog: Labrador Retriever`).

**Are all adoptable animals included?** The public search lists currently available animals. Most have photos, but a few listings show only a placeholder, so their `pictures` fields are omitted.

**How fresh is the data?** Every run queries the live source.

**Is this affiliated with RescueGroups.org?** No. This is an independent third-party actor using publicly available pages and, optionally, the public API.

## Limitations

- The public search exposes 25/50/100/200-mile radius buckets; the actor rounds down, using 25 for smaller requests and 200 for larger ones.
- Age and size are single-choice filters on the public path (the source's own checkboxes support multiple values).
- Breed matching is exact after case-folding and matches any breed in the animal's breed string (primary or secondary); near-miss spellings return a clear status message instead of wrong results.
- Without an API key, organization lookup works by numeric ID only; organization name search needs the optional key.
- The source is US/Canada focused; most listings have photos, but some have no public photo and omit the picture fields.
- Some animals and organizations omit fields such as size, contact details, address or phone; those fields are absent from the record rather than guessed.
