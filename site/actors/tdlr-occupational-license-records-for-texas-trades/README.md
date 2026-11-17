# TDLR Occupational License Records for Texas Trades

The Texas Department of Licensing and Regulation licenses more than 80 trades — cosmetology, electricians, HVAC technicians, tow truck operators, elevator inspectors, auctioneers, property tax professionals, and more. This actor searches those license records by holder or business name, or looks up a specific license number directly, returning status, expiration, trade category, and contact details.

## What this actor does

- Searches by license holder or business name, matching either field with a case-insensitive partial match.
- Looks up a specific license number directly when you already know it.
- Filters results to one of 84 TDLR trade categories, a Texas county, and/or active-vs-expired status.
- Works out each license's active/expired status itself by comparing its expiration date against today, since TDLR's own data doesn't carry a separate status field.
- Leaves a field out of a record whenever TDLR never published that value for the license, rather than sending it empty.

## Output per license

| Field | Description |
|---|---|
| `licenseNumber` | TDLR license number |
| `licenseType` | TDLR trade category, e.g. `Cosmetology Operator`, `Electrical Contractor` |
| `licenseSubtype` | TDLR's internal subtype/tier code, when published |
| `ownerName` | Individual or business owner name |
| `businessName` | Business or DBA name |
| `businessAddressLine1`, `businessAddressLine2`, `businessCityStateZip`, `businessCounty`, `businessPhone` | Business location and phone |
| `mailingAddressLine1`, `mailingAddressLine2`, `mailingCityStateZip`, `mailingCounty`, `ownerPhone` | Mailing address and owner phone |
| `latitude`, `longitude` | Geocoded business location, when published |
| `expirationDate` | License expiration date |
| `licenseStatus` | `active` or `expired`, computed from `expirationDate` against today |
| `continuingEducationRequired` | Whether TDLR flags this license as requiring continuing education |
| `sourceUrl` | Link to the record on the Texas Open Data Portal |
| `datasetLastUpdated` | When TDLR's published dataset was last refreshed |
| `recordType` | Always `license` |
| `scrapedAt` | UTC timestamp of the scrape |

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `byName` | `byName` (holder/business name search) or `byLicenseNumber` (direct lookup). |
| `nameQuery` | string | `Smith` | License holder or business name to match, used in `byName` mode. Optional if `licenseType` and/or `countyFilter` is set. |
| `licenseNumber` | string | – | TDLR license number to look up, used in `byLicenseNumber` mode. |
| `licenseType` | string | – (all types) | One of 84 TDLR trade categories to restrict results to. |
| `countyFilter` | string | – | Texas county name, matching business or mailing county. |
| `statusFilter` | string | – (both) | `active` or `expired`. |
| `sortBy` | string | `licenseNumber` | `licenseNumber`, `ownerName`, `businessName`, or `expirationDate`. |
| `maxItems` | integer | `20` | Maximum number of license records to return. |

### Example: search by name

```json
{
  "mode": "byName",
  "nameQuery": "Smith",
  "maxItems": 20
}
```

### Example: active cosmetology operators in one county

```json
{
  "mode": "byName",
  "nameQuery": "Garcia",
  "licenseType": "Cosmetology Operator",
  "countyFilter": "Harris",
  "statusFilter": "active",
  "maxItems": 50
}
```

### Example: direct license number lookup

```json
{
  "mode": "byLicenseNumber",
  "licenseNumber": "11484",
  "licenseType": "A/C Contractor"
}
```

### Example: browse a trade category

```json
{
  "mode": "byName",
  "nameQuery": "a",
  "licenseType": "Tow Truck Operator (Consent Tow)",
  "maxItems": 100
}
```

## Use cases

- **Contractor vetting** — confirm an electrician, HVAC technician, or contractor's license is active before hiring them.
- **Compliance checks** — verify cosmetology or barbering staff licenses across multiple locations.
- **Vendor and lead validation** — confirm a business's license number resolves to a real, current license.
- **Industry-specific checks** — look up tow truck operators, elevator inspectors, or property tax professionals by trade category.
- **Market mapping** — enumerate licensed businesses by trade category and county.

## FAQ

**What's the data source, and is this affiliated with TDLR?**
It's TDLR's own official license dataset, published on the Texas Open Data Portal. This is an independent, third-party actor and isn't affiliated with or endorsed by TDLR or the State of Texas.

**How fresh is the data?**
The Open Data Portal dataset is refreshed by TDLR on a regular cadence — the `datasetLastUpdated` field on each record tells you exactly when. It isn't a live, real-time feed, but it reflects the same license records TDLR itself publishes.

**Why do some records have no `licenseType`?**
A small number of rows in TDLR's own dataset don't have a trade category filled in; that field is left off rather than shown blank.

**Why can the same license number return more than one record?**
TDLR assigns license numbers per trade category rather than globally, so one number can belong to several unrelated license types at once (for example, an A/C Technician and a Property Tax Consultant sharing the same number). Set `licenseType` to narrow to the one you mean.

**Do I need a login or proxy?**
No. The Texas Open Data Portal is fully public and doesn't require authentication.

**How is `licenseStatus` worked out?**
By comparing each license's expiration date to the current date: `active` if it expires today or later, `expired` otherwise. TDLR's dataset itself doesn't publish a separate status field.

**Can I search without a name at all?**
Yes — leave `nameQuery` blank and set `licenseType` and/or `countyFilter` instead to browse licenses matching just those filters.
