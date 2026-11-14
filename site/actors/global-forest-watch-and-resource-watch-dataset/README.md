# Global Forest Watch & Resource Watch Dataset Explorer

Global Forest Watch, Resource Watch, Aqueduct, PREP, Forest Atlas and the Open Data portal all publish their environmental and geospatial datasets through one shared dataset catalog. This actor browses that catalog, returning each dataset's name, description, provider, connector type, source application(s) and tags — useful for discovering which deforestation, land-use, water-risk or climate datasets exist before you dig into any one of them.

## What this actor does

- Lists every dataset entry in the shared GFW/Resource Watch catalog, paginated automatically
- Filters results to one application at a time: Global Forest Watch, Resource Watch, PREP, Aqueduct, Forest Atlas or Open Data
- Searches dataset name and description for a free-text keyword
- Resolves each dataset's vocabulary entries into a flat list of tags
- Empty fields are omitted — a dataset only carries the fields the catalog actually published for it

## Output per dataset

- `datasetId` — the catalog's unique dataset identifier
- `name` — dataset title
- `description` — dataset description or subtitle, trimmed to 500 characters
- `provider` — the underlying data provider/connector (e.g. `cartodb`, `gee`, `csv`)
- `connectorType` — how the catalog connects to the data (e.g. `rest`, `document`)
- `updatedAt` — when the dataset was last updated in the catalog
- `applications[]` — which application(s) list this dataset (`gfw`, `rw`, `prep`, `aqueduct`, `forest-atlas`, `open-data`)
- `tags[]` — up to 10 vocabulary tags associated with the dataset
- `datasetUrl` — the dataset's own catalog API endpoint
- `recordType: "dataset"`, `scrapedAt`

## Input

| Field | Type | Default | Description |
|---|---|---|---|
| `mode` | string | `datasets` | Only `datasets` (browse environmental datasets) is currently available |
| `searchQuery` | string | – | Keyword matched against dataset name and description |
| `application` | string | – (any) | Restrict to one source application: `gfw`, `rw`, `prep`, `aqueduct`, `forest-atlas`, `open-data` |
| `maxItems` | integer | `20` | Maximum number of datasets to return (1–500) |

### Example: all Global Forest Watch datasets

```json
{
  "mode": "datasets",
  "application": "gfw",
  "maxItems": 100
}
```

### Example: keyword search across every application

```json
{
  "mode": "datasets",
  "searchQuery": "mangrove",
  "maxItems": 50
}
```

### Example: Aqueduct water-risk datasets

```json
{
  "mode": "datasets",
  "application": "aqueduct",
  "maxItems": 30
}
```

## Use cases

- **Environmental researchers** — inventory which forest-cover, land-use or water-risk datasets exist before requesting the underlying data
- **GIS and data engineering teams** — build a catalog of available dataset IDs and providers to wire into a downstream pipeline
- **Journalists and NGOs** — discover relevant climate/deforestation datasets by keyword for a story or report
- **Platform integrators** — compare dataset coverage across Global Forest Watch, Resource Watch, Aqueduct, PREP and Forest Atlas in one pass

## FAQ

**Does this return the actual forest-cover or deforestation data?**
No. It returns the catalog *metadata* about each dataset — name, description, provider, tags and the dataset's own API endpoint — not the underlying geospatial layers or time-series values themselves.

**Why do Global Forest Watch and Resource Watch share one catalog?**
Global Forest Watch, Resource Watch, Aqueduct, PREP, Forest Atlas and Open Data are all built on the same underlying dataset infrastructure; a single dataset can be listed under more than one application, which is why `applications[]` can contain multiple values.

**What goes in `provider` and `connectorType`?**
These describe how the catalog itself fetches the dataset — a backend identifier (e.g. a tile service, spreadsheet or raster provider) and the connection method, not a geographic or topical classification.

**Why are `tags[]` sometimes empty?**
Not every dataset has vocabulary/tag entries defined in the catalog. When none exist, the field is omitted from that record entirely.

**Can I filter by more than one application at a time?**
No — `application` accepts a single value per run. Run it multiple times with a different value to compare coverage across applications.

**How fresh is this data?**
Every run queries the catalog live, so `updatedAt` and the dataset list reflect the current state of the catalog at run time.

**Is this affiliated with Global Forest Watch or the World Resources Institute?**
No. This is an independent, third-party actor that reads the publicly accessible dataset catalog; it is not operated by or affiliated with Global Forest Watch, Resource Watch or the World Resources Institute.
