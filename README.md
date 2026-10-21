# Vexaloom Labs Actor examples

Runnable Python examples and reference documentation for [Vexaloom Labs's Apify Actors](https://apify.com/vexaloom-labs). Maintained by the publisher.

Use these examples to collect data through the Apify API and turn results into local tables, review summaries, or research files. Python 3.10+ and the standard library are sufficient. Scraper implementations run on Apify and are not included in this repository.

## Start with a working example

Each guide includes an offline fixture, a small input, validation instructions, and an optional billable execution step.

| Example | Audience |
| --- | --- |
| [Website pages to a Markdown corpus for RAG](examples/website-to-markdown-corpus/README.md) | RAG and knowledge-base developers |
| [Google Trends results to an analysis table](examples/google-trends-to-csv/README.md) | SEO analysts and market researchers |
| [Trustpilot reviews to a rating summary](examples/trustpilot-rating-summary/README.md) | Customer research and product teams |

Clone this repository, open an example directory, and follow its README. Start with the offline transformation; it needs no account or API key. For live execution, use your own Apify account and review current Actor pricing.

## Find an Actor

Browse the [Actor catalog](CATALOG.md) for purpose, categories, published README, input schema, and Apify links. [actors.json](actors.json) provides a machine-readable index with stable Actor IDs. Documentation is synchronized from default published builds; draft changes are not included.

## Reference updates and support

Reference docs update automatically when new public Actors or published documentation changes are detected. Example code is maintained separately and is never overwritten by the sync. Current availability and pricing are shown on Apify.

See [SUPPORT.md](SUPPORT.md) for help, [CONTRIBUTING.md](CONTRIBUTING.md) for improvements, and [NOTICE.md](NOTICE.md) for documentation attribution and license scope. Fixtures are synthetic; live extraction success and complete source coverage are not guaranteed.

## Documentation website

[Browse the Actor catalog, schemas, and Python guides](https://asimkhaniqbal.github.io/apify-actor-examples/).
