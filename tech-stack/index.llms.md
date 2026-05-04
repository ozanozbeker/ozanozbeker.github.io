# Tech Stack

These are the tools I actually use, with notes on why. Updated when something changes.

## The Data Stack

Most of my work flows through Python and [Polars](https://pola.rs/), with [Parquet](https://parquet.apache.org/) as my storage format and [ADBC](https://arrow.apache.org/adbc/) as the database bridge ([`dbc`](https://columnar.tech/dbc/) shout-out). [Arrow](https://arrow.apache.org/) and ADBC are the foundation under most of the tools I reach for. I’m a user, not a database or systems engineer, but the experience as a user is excellent: column-oriented, fast pulls from sources, and direct compatibility with Polars frames and Parquet files. It just works, and it works fast.

Polars is my DataFrame tool of choice. I came from [dplyr](https://dplyr.tidyverse.org/) in R (my original GOAT), and Polars feels like something between dplyr and SQL. Method chaining clicks in my head, and lazy frames with `sink_*` methods are memory-efficient in a way that scales past what fits in RAM. The thing I most like about the Polars API over dplyr is how `pl.col()` explicitly states what column you mean. dplyr mixes helper functions and bare-name references, which gets awkward once you start writing functions around it. I also find it easier to build method chains off an object than to remember which package each function came from, but that’s personal preference.

I still reach for [DuckDB](https://duckdb.org/) at the command line. For 99% of use cases I find Polars and DuckDB neck and neck for performance, I just prefer writing Polars. Both are heavy hitters and either can do the job well. Storing analytical data as Parquet is what makes them interchangeable: a teammate who prefers Python, Polars, [pandas](https://pandas.pydata.org/), or SQL can all work against the same files without conversion. I’m interested in learning more about [ducklake](https://ducklake.select) in the future.

A couple of libraries make the Python side of this stack feel even more solid: [dataframely](https://github.com/Quantco/dataframely) for tabular validation. I haven’t used [pandera](https://pandera.readthedocs.io/) much, but my impression is most of its features are pandas-only right now. dataframely is Polars-native and handles cross-table validation, like the kind you’d get from foreign-key constraints in a database, but on flat files. The maintainers (QuantCo) clearly care about the project, which I appreciate. I’ve also tried out [Pydantic](https://docs.pydantic.dev/latest/), but honestly for what I do, Python’s own dataclasses fills the void. Now, [Pydantic Settings](https://docs.pydantic.dev/latest/) I’m a huge fan of.

[dlt](https://dlthub.com/) for ingestion. I’m new to it but already impressed by the use case: cleaning nested, inconsistently-shaped JSON (Oxylabs responses are a great example) and handling schema evolution gracefully. Schema evolution is something I manage “by hand”, usually by registry-timestamped functions, but I’m excited to learn more about how dlt works in this regard.

The center of all of this is [marimo](https://marimo.io/), the new notebook on the block. The UI is what stood out to me first, plus the [uv](https://docs.astral.sh/uv/) and [Ruff](https://docs.astral.sh/ruff/) compatibility is phenomenal. The fact that notebooks save as plain Python files, that you can build internal tools out of them, that execution is reactive, and that the UI keeps getting better. It’s an honest pleasure to work in. You can tell the team behind it really cares about their work. (That’s a recurring theme in my picks: I gravitate toward tools whose maintainers are clearly thoughtful about what they’re building. Not a knock on others, just where I land.)

## Building, Code Quality, & Shipping

At the heart of this is [Zed](https://zed.dev/), another new editor on the block that’s lightweight and super snappy. Zed comes configured with Ruff and [basedpyright](https://docs.basedpyright.com/) (which are also my tools of choice), and dropping in your config TOML files per project is a breeze. I’m not going to name any other editors, the speed and cleanliness of Zed has made it easily my go-to for scripts and production code, and I go back to marimo for exploration and data-heavy work. Both can use the same `ruff.toml` (and maybe `ty.toml` in my near future), so they play along with each other very well.

basedpyright is my choice because it’s the default in Zed, which is how it ended up in my stack. I’m interested in trying [ty](https://github.com/astral-sh/ty) and [pyrefly](https://pyrefly.org/) when I get a chance, but basedpyright works well enough that the upgrade hasn’t been a priority.

uv handles project, dependency, and Python-version management. The speed alone makes it worth the switch and the PEP-based defaults and the `[project.scripts]` block in `pyproject.toml` are why it’s stuck. Defining script entry points in `pyproject.toml` means a fresh teammate can clone a repo, run `uv sync`, and have working CLI commands without me writing setup docs.

Most of my work eventually needs to run unattended, which where I work means Windows Task Scheduler kicking off `.bat` files. One `.bat` file per pipeline, each calling a defined entry point, so every pipeline is a named, importable, runnable unit rather than a folder of loose scripts. That’s a big part of why I’ve been building CLIs around my work.

[Typer](https://typer.tiangolo.com/) is my CLI framework of choice. It’s a sister project to FastAPI, sharing the same type-hint-driven philosophy. I prefer using type hinting wherever I can in production code, and Typer is built around it.

Speaking of sister projects, I’ve been delving into [FastAPI](https://fastapi.tiangolo.com/) as we try to move some of our models and functions to be services. I like it for the same reasons as Typer, plus the docs between the two packages are wonderful.

Git for version control, GitHub for hosting (personal) and GitLab at work. No surprises here.

## Reporting and Reproducibility

[Quarto](https://quarto.org/) is the centerpiece. It renders this website, my client-facing reports, and a free public textbook for the course I teach (with hopes for more books down the line). One tool, three very different outputs, all from plain markdown with code that executes during rendering. Writing once and rendering as a webpage, a PDF, and a slide deck saves real time.

For charts and tables, I lean on a few different libraries depending on the use-case:

- [Plotly](https://plotly.com/python/) looks the nicest and is the default in Shiny, but it’s too heavy for Quarto HTML files I’m sending over email.
- [Altair](https://altair-viz.github.io/) fills that gap. It’s easy to write, has great built-in interactivity in marimo (assuming the data isn’t huge), and stays light enough for embedded HTML.
- [Plotnine](https://plotnine.org/) and [great-tables](https://posit-dev.github.io/great-tables/) (shout out Posit) are my go-tos for flat charts and tables, especially in email-friendly reports.
- [py-reactable](https://machow.github.io/reactable-py/) for interactive HTML tables when I need them, though sending HTML-with-JavaScript files through Outlook has been hit-or-miss for me. I haven’t tracked down the root cause, I just pick the best tool for the job.

I work in whatever the client uses. Quarto reports with figures and tables are my preference for summaries and daily/weekly/monthly cadence. Excel and CSV are the way when the client wants larger or raw data, which is most business people most of the time. I’m not a fan of any of the dashboarding services in particular, but if a client is on Tableau or Power BI, I’m on Tableau or Power BI. [Shiny](https://shiny.posit.co/) (Python and R) is what I reach for when interactive exploration is part of the deliverable.

## Databases

[PostgreSQL](https://www.postgresql.org/) can do it all, but rarely do you actually have a choice in the matter. Most of my work has been against [SQL Server](https://www.microsoft.com/en-us/sql-server), [IBM DB2](https://www.ibm.com/products/db2), and [SAP HANA](https://www.sap.com/products/technology-platform/hana.html), all of which exist because someone made a corporate-purchasing decision a decade ago and the data hasn’t moved since. They each have their quirks. PostgreSQL is the exception. PostgreSQL can do no wrong.

DuckDB I covered above, it’s my local default.

## Supporting Cast

[httpx](https://www.python-httpx.org/) is my HTTP client of choice. Async-first, requests-compatible API, and the HTTP client I reach for whenever something needs to hit an API at scale.

[LightGBM](https://lightgbm.readthedocs.io/) when modeling is the answer. Tabular data, native categorical support, fast to train, and well-supported in Python. I’ve used it to train an in-house tabular model that replaced a third-party SaaS dependency at work.

[SQLAlchemy](https://www.sqlalchemy.org/) when ADBC doesn’t have a driver. I don’t build web apps so I don’t really have a use for ORMs. My preferred pattern is raw SQL files loaded into Python and executed with Arrow via `from adbc_driver_manager import dbapi`. SQLAlchemy is the fallback for private databases like DB2 where ADBC coverage is thinner.

[Selenium](https://www.selenium.dev/) when an API doesn’t exist and a browser has to be in the loop. Less and less of my work goes through it now that I’ve moved most of my browser-automation work over to async HTTP clients, but it’s still in the toolbox.

Linux and Bash are the operating environment. WSL on my work Windows machine, native Linux on my home server, and bash scripts whenever something needs to glue tools together.

[Docker](https://www.docker.com/) when I need to package something for deployment, but not daily. Most of what I ship runs as scripts, packages, or Quarto documents that don’t need containerization.

## Currently Learning

[Dagster](https://dagster.io/) and [dbt](https://www.getdbt.com/), with dlt covered above. Together they’re the declarative side of the modern data stack: Dagster for orchestration, dbt for transformations, dlt for ingestion. I’ve prototyped Dagster at work but haven’t shipped it, dbt I’m working through Fundamentals, dlt is the one that’s clicked fastest because the use case is one I run into all the time (nested JSON cleanup).

I’ll have more to say about each as I get more reps in.

### Outside the data realm

I’m interested in the rest of the engineering stack too: dev ops, backend engineering, systems work. Not because I want to switch tracks. The way playing guitar helped me learn piano, picking up the surrounding stack helps me think more clearly about the data work. I’m aiming to be a full-stack data scientist, and full-stack means getting comfortable wearing more than one hat.

A few things I’m actively working through:

- [boot.dev](https://www.boot.dev/) for structured curriculum, currently working through the Backend and DevOps tracks. Go, Docker, Linux, HTTP, networking, and CI/CD are the through-lines.
- Self-hosting on a home server: Ubuntu under the hood, Docker Compose for the apps, with a slowly-growing list of services (Home Assistant, dashboards, a Quarto-rendered version of this website as the test case before it goes live).
- Working through technical books on my own time. Linux and the shell, database management, advanced Python, data engineering, and machine learning.
- [Google Cloud Platform](https://cloud.google.com/) at work, where my largest engagement runs. Mostly observing as the team gets credentialed in, but the credentials are coming.
- Logging and observability with [Prometheus](https://prometheus.io/) and similar tools. I’m not running them in production yet but I’m reading and prototyping.

None of this is to claim I’m a backend engineer or a sysadmin. It’s to say the data work I do sits on top of these layers, and understanding the layers makes the work better.

Also, I’m really trying to get back into the guitar. Those bad boys have been hanging on the wall for too long.
