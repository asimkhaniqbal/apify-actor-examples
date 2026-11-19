# GSMArena Device Database & Spec Sheet Extractor

GSMArena maintains one of the most complete historical records of mobile phones, from early-2000s feature phones to the newest flagships. This actor reads that catalog in two depths: a lighter summary record for browsing/searching many devices at once, and a full spec-sheet record for one device at a time.

## What this actor does

- Searches the catalog by keyword with optional OS, availability and sort filters.
- Lists every device for a given brand, or the current top devices by popularity/price/weight/camera/battery.
- Pulls a complete specification sheet — display, chipset, cameras, battery, connectivity, body and sensors — for one device by URL or slug.
- Normalizes shorthand specs (RAM, storage, battery capacity, display size) out of the brief text shown in listings.
- Fields GSMArena doesn't publish for a device (e.g. no listed price, no memory card slot) are left out of that record.

## Output per device (search, byBrand, topDevices modes)

| Field | Description |
| --- | --- |
| `url` | GSMArena device page URL. |
| `slug` | GSMArena's URL slug for the device. |
| `deviceName` | Full device name as listed. |
| `brand` | Manufacturer, inferred from the device name. |
| `imageUrl` | Listing thumbnail image URL. |
| `os` | Operating system, when shown in the listing's brief specs. |
| `released` | Announcement or release period, when shown in the brief specs. |
| `displaySize` | Screen size, when shown in the brief specs. |
| `chipset` | Chipset/SoC, when shown in the brief specs. |
| `battery` | Battery capacity, when shown in the brief specs. |
| `storage` | Internal storage, when shown in the brief specs. |
| `ram` | RAM capacity, when shown in the brief specs. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Output per device (deviceDetails mode)

| Field | Description |
| --- | --- |
| `url` / `slug` | Device page URL and GSMArena slug. |
| `deviceName` / `brand` | Full device name and manufacturer. |
| `imageUrl` | Product image URL. |
| `released` / `status` | Release date and current availability status. |
| `displaySize` / `displayResolution` / `displayType` / `displayProtection` | Screen size, resolution, panel technology, and protective glass/coating. |
| `chipset` / `cpu` / `gpu` | Processor, CPU core layout, and graphics processor. |
| `os` | Operating system and version/skin. |
| `ram` / `storage` / `memorySlot` | RAM capacity, internal storage, and memory-card slot support. |
| `rearCamera` / `rearCameraDetails` | Main rear camera resolution and the full rear camera module specs. |
| `frontCamera` / `frontCameraDetails` | Selfie camera resolution and full front camera specs. |
| `battery` / `batteryDetails` | Battery capacity and full battery/charging details. |
| `network` / `networkTechnology` | Highest supported network generation and the full network band list. |
| `wlan` / `bluetooth` / `nfc` / `usb` / `gps` / `radio` | Connectivity specs. |
| `dimensions` / `weight` / `build` | Physical size, weight, and build materials. |
| `sim` / `bodyOther` | SIM configuration and other body notes. |
| `sensors` | List of onboard sensors. |
| `colors` / `models` | Available color options and regional model numbers. |
| `price` | Suggested or observed retail price, when GSMArena lists one. |
| `scrapedAt` | ISO timestamp of when the record was captured. |

## Input

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `mode` | String (select) | `search` | `search`, `deviceDetails`, `byBrand`, or `topDevices`. |
| `searchQuery` | String | `Samsung Galaxy` | Keyword to search for (mode=search). |
| `osFilter` | String (select) | – | Filter by operating system: Android, iOS, feature phones, KaiOS, Windows Phone, Symbian, BlackBerry, Bada, or Firefox OS (mode=search). |
| `availability` | String (select) | – | Filter by availability: Available, Coming soon, Discontinued, or Rumored (mode=search). |
| `sortBy` | String (select) | `0` (Popularity) | Sort order: Popularity, Price, Weight, Camera resolution, or Battery capacity (mode=search and topDevices). |
| `deviceUrl` | String | – | Full device URL or slug (mode=deviceDetails). |
| `brandSlug` | String | – | Brand slug to list, e.g. `apple`, `xiaomi`, `google` (mode=byBrand). |
| `maxItems` | Integer | 20 | Maximum number of device records to return. |

### Example: search Android phones sorted by battery capacity

```json
{
  "mode": "search",
  "searchQuery": "Galaxy",
  "osFilter": "2",
  "sortBy": "4",
  "maxItems": 30
}
```

### Example: full spec sheet for one device

```json
{
  "mode": "deviceDetails",
  "deviceUrl": "samsung_galaxy_s25_ultra-12610"
}
```

### Example: every current device from a brand

```json
{
  "mode": "byBrand",
  "brandSlug": "xiaomi",
  "maxItems": 100
}
```

## Use cases

- **Tech comparison sites** pulling spec sheets for review articles and comparison tables.
- **E-commerce catalogs** enriching phone listings with authoritative display, camera and battery specs.
- **Market researchers** tracking which chipsets, display types or camera resolutions ship across a brand's lineup.
- **App and accessory developers** building device-compatibility matrices from dimensions, SIM type and connectivity data.
- **Price trackers** monitoring suggested retail prices across a large set of devices.

## FAQ

**Is GSMArena's data public?**
Yes. This actor reads the same specification pages GSMArena publishes openly to any visitor, without an account or login.

**Why does a `search` or `byBrand` record have fewer fields than `deviceDetails`?**
Search and brand-listing pages only show a brief spec summary per device. To get the complete spec sheet for a device, run `deviceDetails` on that device's URL or slug.

**How current is the device catalog?**
GSMArena typically adds new devices within days of their announcement, and this actor reads the live catalog on each run.

**Does this cover older or discontinued phones?**
Yes, GSMArena's archive spans from early feature phones to current flagships; use the `availability` filter to narrow to Discontinued or Rumored devices specifically.

**What's the best way to collect every device from one manufacturer?**
Use `byBrand` with that brand's slug and a high `maxItems`; large manufacturers can have over a thousand devices across their history.

**Why might `price` be missing from a device record?**
GSMArena only lists a price when one has been reported for that device; without one, the field is omitted.

**Is this affiliated with GSMArena?**
No. This is an independent tool and is not affiliated with, endorsed by, or sponsored by GSMArena.
