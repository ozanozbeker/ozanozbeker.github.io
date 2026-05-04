# oxy

Replacing a Selenium-on-Helium-10 setup with an async Python CLI and a portable datalake

Python

CLI

Web Scraping

Datalake

Published

September 1, 2025

The team’s Amazon marketplace data came from a setup that drove a Chrome browser through Selenium, against the Helium 10 chrome-extension UI. Five to eight minutes per scrape, randomized, because Helium 10 was throttling and back-off was the only way to avoid getting flagged. There was a hidden ~100-search daily limit that nobody could find documented anywhere. Chrome version updates would silently break the driver. There was no API, so the weekly and monthly reports built on top of it all required a human to hit play.

By the time it landed on me, the system was failing weekly. Reports going out late. The team spending real hours per week firefighting a tool that was supposed to be invisible. So I went looking for an alternative, found Oxylabs (HTTP API, 50 requests per second, no browser, no bot-detection arms race), and built the replacement.

## The CLI

`oxy` is async over `httpx`, hits Oxylabs at the rate-limit ceiling, and runs as a single Typer-based CLI with two namespaces. `oxy scrape` for the API endpoints, `oxy build` for datalake operations. Defaults baked into the commands. Project-specific overrides live in YAML configs that the CLI loads with one flag.

It started as functions imported from a Python module, the way most internal tools do. After the third project where I was reconfiguring the same scripts I gave up and built the CLI properly. Installed once as a `uv` tool, runs from any project’s `.bat` file under Windows Task Scheduler.

## The datalake

`oxy` needed somewhere to put the data and the team didn’t have a shared layer for it, so I designed one. Four layers, all on a Windows network share, all in formats that move cleanly to S3 or GCS when company-wide cloud access lands.

``` mermaid
flowchart TD
    A[Windows Task Scheduler] --> B[.bat file per pipeline]
    B --> C["oxy scrape --config project.yml"]
    C -->|async httpx, Oxylabs API| D[Raw: JSON, preserved verbatim]
    D --> E["oxy build --config project.yml"]
    E --> F[Processed: Parquet per table per scrape]
    F --> G[Model: joined, conformed, query-ready]
    G --> H[Reports, Shiny apps, ad-hoc analyses]
```

Raw is JSON keyed by scrape ID. Processed is Parquet, one file per logical table per scrape (NDJSON for append-heavy logs). Model is the joined, conformed analytics layer that downstream reports query directly. Mart is reserved for future presentation aggregates. About 4 GB of compressed Parquet in the model layer today, growing weekly.

The architectural point at this scale is that Polars and DuckDB are the right tools. Not Spark, not a warehouse. Free OSS tooling does the job, and the layout is portable, so when GCP access becomes available the migration is a copy operation, not a rewrite.

> **NOTE:**
>
> - Per-scrape: 5 to 8 minutes down to ~50 ms.
> - Bulk workloads: ~5 days down to ~5 minutes.
> - Currently ~20,000 scrapes per month, fully unattended, across 5 projects for 2 client engagements.
> - Daily-limit ceiling: gone. Chrome-version-update breakage: gone.

If I were doing this at a larger company I’d split the scrape and datalake namespaces into separate tools. At our team’s size, one tool was the right answer. The package owns the Oxylabs response schema and the datalake layout that depends on it, so schema changes are atomic.

The bigger lesson was about scope. The right time to build a CLI is the third project where you’re reconfiguring the same scripts. Earlier and you’re guessing at requirements. Later and you’ve already paid the cost the CLI was supposed to save you. The right time to swap browser automation for an HTTP API is when an HTTP API exists. I should have looked sooner.
