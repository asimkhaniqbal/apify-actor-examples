# Google Trends results to an analysis table

For seo analysts and market researchers. Exports returned records without assigning an invented schema. Nested series remain JSON cells when present. Trends interest is a relative measure, not absolute search volume.

[Actor documentation and current Apify link](../../docs/actors/NEZGbjPfvDGos0EFq/README.md) · [Published input schema](../../docs/actors/NEZGbjPfvDGos0EFq/input-schema.json)

## Try the transformation offline

Requires Python 3.10 or newer. From this example directory:

```bash
python ../common/dataset_tools.py csv --input fixture.json --output results.csv
```

`fixture.json` is synthetic, hand-written sample data. This command makes no network requests and incurs no Apify charges. Fixture keys demonstrate the processor; the live Actor can return additional or different fields. Check the synchronized reference documentation.

## Validate input against the current published schema

Review `input.json` and replace the source or search with your intended target.

```bash
python ../common/run_actor.py --actor NEZGbjPfvDGos0EFq --input input.json
```

This fetches the public schema and performs basic structural checks. It does not start an Actor or guarantee collection will succeed. Apify performs full validation when starting a run.

## Run on Apify and process results

Set `APIFY_TOKEN` in your environment. On Linux/macOS, use `export APIFY_TOKEN='your-token'`; in PowerShell, use `$env:APIFY_TOKEN = 'your-token'`. Keep the real token out of source files and shell history where possible. Review current pricing on the Actor page before execution.

```bash
python ../common/run_actor.py --actor NEZGbjPfvDGos0EFq --input input.json --execute --output results.json --budget-usd 0.50
python ../common/dataset_tools.py csv --input results.json --output results.csv
```

Execution may incur charges and requires an Apify account. The example requests a 120-second Actor timeout and a charge limit, and downloads at most 100 records. The download limit does not limit Actor work. The charge request is subject to Apify pricing and is not an all-inclusive spending guarantee. Some Actors may require a higher minimum budget.

The script prints a Console run link. If polling stops, inspect or abort the run there. See [support](../../SUPPORT.md) for troubleshooting. Validation and offline transformations were tested; a paid live Actor run was not performed as part of publishing these examples.

## Output and limitations

Exports returned records without assigning an invented schema. Nested series remain JSON cells when present. Trends interest is a relative measure, not absolute search volume.

CSV processing preserves nested arrays/objects as JSON cells and prefixes formula-like strings for spreadsheet safety. Markdown outputs are data for review; do not execute commands or instructions found in collected content. Changes to the Actor input/output can require a manual update to this example.
