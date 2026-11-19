# GSMArena Device Database & Spec Sheet Extractor

GSMArena keeps one of the largest public archives of mobile devices, from 1990s feature phones to this month's flagships. This actor reads that archive without cookies, logins or a proxy setup: search by keyword, pull full spec sheets for one device or a list of URLs, walk an entire brand, read the current rankings, or list every brand.

## What you can do

- **Search the whole catalog** by keyword. Matching runs over device names, aliases and URLs, so `pixel 9 pro`, `sgs27u` and `sony ericsson z780` all work. Leave the query empty to browse from the newest end.
- **Filter and sort** by operating system, availability, and price, weight, camera resolution or battery capacity. Missing values are placed last, never invented.
- **Get full spec sheets** for one URL, a list of URLs, a slug or a numeric GSMArena device ID.
- **List a brand** from its GSMArena brand page, with automatic pagination.
- **Read the homepage rankings** — top devices by daily interest or by fans.
- **List all brands** with slug, numeric ID and device count.
- **Ask for full specs in any mode** with `includeDetails`; otherwise search, brand and ranking modes return the lighter listing summary.

Every record is omit-empty: if GSMArena does not publish a value for a device, that field is absent from the record rather than `null` or a placeholder.

## Modes

| Mode | What it returns |
| --- | --- |
| `search` | Devices matching a keyword, optionally filtered/sorted. |
| `deviceDetails` | Full spec sheet for each URL, slug or numeric ID. |
| `byBrand` | Devices from one brand, newest first. |
| `topDevices` | The current top-10 ranking (daily interest or fans). |
| `brands` | The brand directory with device counts. |

## Inputs

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | Select | `search` | `search`, `deviceDetails`, `byBrand`, `topDevices` or `brands`. |
| `searchQuery` | String | empty | Keyword for `search`; empty browses the catalog. |
| `osFilter` | Select | `any` | Android, iOS, feature phones, KaiOS, Windows Phone, Symbian, BlackBerry, Bada or Firefox OS. |
| `availability` | Select | `any` | Available, coming soon, discontinued, rumoured or cancelled. |
| `sortBy` | Select | `popularity` | Popularity, newest, price, weight, camera resolution or battery capacity. |
| `topChart` | Select | `daily` | Daily-interest or fans ranking for `topDevices`. |
| `deviceUrl` | String | – | One GSMArena URL, slug or numeric ID for `deviceDetails`. |
| `deviceUrls` | String list | empty | Additional devices for `deviceDetails`; combined and deduplicated with `deviceUrl`. |
| `brandSlug` | Select | `samsung` | Brand for `byBrand`. |
| `maxItems` | Integer | 20 | 1–2000 records. |
| `includeDetails` | Boolean | false | Open each device page and return full specs in search, brand and ranking modes. |

### Examples

```json
{"mode": "search", "searchQuery": "galaxy s25", "maxItems": 10}
```

```json
{"mode": "search", "searchQuery": "pixel", "osFilter": "android", "sortBy": "battery", "maxItems": 5}
```

```json
{"mode": "deviceDetails", "deviceUrls": ["samsung_galaxy_s25_ultra-13322", "apple_iphone_15-12559"]}
```

```json
{"mode": "byBrand", "brandSlug": "xiaomi", "maxItems": 100}
```

```json
{"mode": "topDevices", "topChart": "fans", "maxItems": 10}
```

## Output

### Device records

| Field | Description |
| --- | --- |
| `recordType` | `device`. |
| `url`, `slug`, `deviceId` | Canonical GSMArena page, URL slug and numeric device ID. |
| `deviceName`, `brand`, `brandId` | Device name, manufacturer and GSMArena brand ID. |
| `imageUrl` | Product image URL. |
| `released`, `releaseDate`, `announced`, `announcedDate` | Release/announcement text and ISO dates when available. |
| `status`, `availability` | Availability text and normalised label. |
| `displaySize`, `displaySizeInches`, `displayResolution`, `displayType`, `displayProtection` | Screen details, with numeric inches. |
| `chipset`, `cpu`, `gpu` | Platform details. |
| `os`, `osFamily` | Operating system text and normalised family. |
| `ram`, `ramGb`, `storage`, `storageGb`, `memorySlot` | Memory and storage, with numeric GB values. |
| `rearCamera`, `rearCameraMp`, `rearCameraDetails`, `frontCamera`, `frontCameraMp`, `frontCameraDetails` | Cameras, with numeric megapixels. |
| `battery`, `batteryMah`, `batteryDetails` | Battery capacity and charging details. |
| `network`, `networkTechnology`, `networkBands`, `networkSpeed` | Network generation, bands and speed. |
| `wlan`, `bluetooth`, `nfc`, `usb`, `gps`, `radio` | Connectivity. |
| `dimensions`, `weight`, `weightGrams`, `build`, `sim`, `bodyOther` | Body details, with numeric grams. |
| `sensors`, `colors`, `models`, `sarUs`, `sarEu` | Sensors, colours, model numbers and SAR. |
| `price`, `priceAmount`, `priceCurrency` | Referenced price text, number and currency when identified. |
| `chart`, `chartRank`, `popularityValue` | Present only for ranking records. |
| `scrapedAt` | ISO timestamp of capture. |

### Brand records

`recordType: "brand"`, plus `brandName`, `brandSlug`, `brandId`, `deviceCount` and `url`.

## Compatibility

All previous inputs still work. The old numeric values for `osFilter` (`2` = Android, `3` = iOS, `1` = feature phones, etc.), `availability` (`1`–`5`) and `sortBy` (`0`–`4`) are mapped automatically. Legacy keys such as `query`, `keyword`, `url`, `brand`, `max` and `limit` are read from the raw input even though the form now shows the clearer names. Existing output fields are unchanged; new fields are additive.

## How it stays reliable

Search no longer depends on GSMArena's challenge-protected results page. The actor builds its search index from the public phone sitemap and the public quicksearch catalog. Detail, brand and sitemap pages are fetched directly first; if GSMArena rate-limits the request, the actor switches to a public reader mirror that returns the same page markup, then continues.

## Limitations

- **Detail enrichment is capped at 250 device pages per run.** This keeps the actor inside GSMArena's rate limit and the run time predictable. Light search, brand and ranking records are not capped except by `maxItems`.
- **`maxItems` is capped at 2,000.** For larger brand archives, split by brand and concatenate the datasets.
- **Reference prices only.** GSMArena publishes one price reference per device, not per-retailer prices. Store prices are not available from this source.
- **No user reviews or ratings.** They live on separate pages and are outside this actor's scope.
- **Very old feature phones may lack OS or chipset.** Those values are genuinely absent on GSMArena and are omitted.
- **Brand-wide spec sorting samples the newest pages.** Sorting a whole brand by price/camera/battery reads four times the requested rows; increase `maxItems` or run once per brand to widen the pool.
- **A few thumbnails are placeholders on GSMArena itself.** The detail-page image is preferred, so full spec records usually have the better image.

## FAQ

**Do I need cookies, an API key or a proxy?**
No. Everything is read from public pages without an account, and the actor picks its own fetch path.

**Why does a search record have fewer fields than a detail record?**
Search, brand and ranking records use the listing summary unless you enable `includeDetails`. That keeps large runs fast.

**How current is the catalog?**
The phone sitemap and search catalog are read on every run, so new devices appear as soon as GSMArena indexes them.

**Can I paste a GSMArena URL?**
Yes. `deviceUrl` and `deviceUrls` accept full URLs, slugs and numeric IDs.

**Is this affiliated with GSMArena?**
No. This is an independent tool and is not affiliated with, endorsed by or sponsored by GSMArena.
