# morula

A Python email body builder for people who actually have to send reports through Outlook

Python

Email

HTML

Published

April 15, 2026

> **NOTE:**
>
> Design complete, implementation in progress. The SMTP sending layer this pairs with is already running in production at work as part of an internal package. `morula` is the body-construction half, being built as a public library on my own time.

I send a lot of HTML email. Weekly client reports, monthly summaries, ad-hoc notifications. The recipients are on Outlook, which means the renderer is Microsoft Word’s HTML engine, which means most of what you know about modern HTML and CSS does not apply. Tables for layout. Inline styles only. System fonts only. PNG-embedded charts because anything else either won’t render or won’t render reliably.

I miss [blastula](https://rstudio.github.io/blastula/). Back when I was writing R, it was the tool I reached for whenever a report needed to go out by email. Composable blocks, sensible defaults, just worked. The Python ecosystem has plenty of email libraries but most of them focus on the sending side, and none of them feel like blastula. So I’m building one. `morula` is the Python email body builder I wish already existed.

The name is a nod. A morula is the embryonic stage that precedes a blastula, which felt right for a smaller, simpler, more upstream library that owes its lineage to the original.

`morula` is also open about its lineage in a more practical sense. blastula is open source under MIT, and I’ve been reading its source directly to understand the table-nesting patterns, the MSO conditional comments, and the rendering choices that took the blastula maintainers years to land on. The Python implementation is mine, the design owes them a real debt, and the docs will say so.

## The design

The architecture follows blastula’s lead. A tested HTML scaffold filled with composable blocks, theme tokens for styling, and rendering helpers for the parts that need to become bytes. The Python-specific pieces are what’s new.

A [Cerberus](https://github.com/TedGoas/Cerberus)-derived HTML scaffold saved as a Jinja2 template, with slots for header, body, and footer. Cerberus is the battle-tested table-based pattern that works across email clients. Jinja2 is the engine that fills the slots at runtime. Write the scaffold once, never think about it again.

Every section of the email is a Python function that returns an inline-styled HTML fragment. `block_text`, `block_image`, `block_table`, `block_plot`, `block_button`. Blocks compose by ordering. The body is a list of block return values.

A top-level `compose_email()` function accepts lists of block HTML strings for header, body, and footer, renders them into the scaffold, and returns the final HTML string ready to hand to the sending layer.

Rendering helpers for the unsexy stuff that makes the blocks work. Markdown via [markdown-it-py](https://markdown-it-py.readthedocs.io/). Image base64 encoding with MIME detection. Plot rasterization (Plotly via `kaleido`, Altair via `vl-convert-python`, Matplotlib via its built-in PNG export, all producing PNG bytes that get base64-embedded). Polars DataFrame to inline-styled HTML table, first-class.

An `EmailStyle` dataclass holding design tokens (colors, fonts, spacing, button styling). Pass an instance into `compose_email()` and every block inherits the same theme. Per-client theming becomes a matter of swapping the style object.

``` mermaid
flowchart LR
    A[block_text]
    B[block_image]
    C[block_table]
    D[block_plot]
    E[EmailStyle]
    A --> F[compose_email]
    B --> F
    C --> F
    D --> F
    E --> F
    F --> G[Jinja2 fills scaffold.html.j2]
    G --> H[Final HTML string]
    H --> I[Your sending layer]
```

## What’s different in morula

Three updates to the blastula approach, mostly reflecting how email rendering has shifted over the last decade.

Inline styles from the start. blastula uses a `<style>` block and counts on email clients to support it. That worked well in 2017. In 2026, full inline is the safer default. Less elegant in the source, more reliable in the inbox.

No `@media` queries. Outlook ignores them anyway. Designing for 600px fixed and letting mobile clients reflow naturally is simpler and avoids the failure modes.

DataFrame-native table block. blastula was built around general HTML composition. Since most of my email content is “render this DataFrame as a table,” that case deserves to be first-class rather than something I build out of generic blocks.

## What’s done, what’s next

Working: the architecture decisions above are settled. The SMTP sending layer this pairs with is already in production at work in a separate codebase.

In progress: scaffold template, the first few block functions (text, image, table), the `EmailStyle` dataclass and design-token spec.

Planned: plot rasterization helpers, Polars to HTML table renderer, Markdown rendering, public PyPI release once the API is stable enough that I’d be embarrassed to break it.

Watch [the GitHub repo](https://github.com/ozanozbeker) for the eventual release. Feedback on the design above is welcome before I’m too committed to change it.
