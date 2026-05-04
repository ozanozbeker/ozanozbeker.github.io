# Computational Workflows for Engineers (cw4e)

A free, open-source Quarto textbook for the computing course I teach at WVU

Teaching

Python

SQL

DuckDB

Quarto

Published

January 15, 2026

I took IENG 331 as an undergrad, was a TA for it, and came back as the instructor of record in January 2025. The course had been on the books for years but the curriculum was basic Python alone. Students would leave knowing language features without the surrounding context that makes Python actually useful for engineering work. There was no textbook that fit the redesign I had in mind, so I wrote one.

The book is also a working artifact for how I think about technical documentation. Every chapter is a runnable Quarto document. Code blocks execute as part of the build. The rendering pipeline is the same Quarto stack I use for client deliverables at OneMagnify. The course doubles as a long-running test case for the documentation discipline I bring to production work. If something is unclear in a chapter, it’s unclear when 75 students hit it the same week, and I find out fast.

## The redesign

Quarto for everything. One source format, three outputs. HTML book for the web, PDF for students who want offline copies, slides for lecture days, all from the same markdown. Code executes during the build, so any example that ships is an example that ran on the most recent compile.

Modern tooling from day one. `uv` for environment management instead of Anaconda. Ruff and basedpyright for code quality. DuckDB instead of a hosted SQL server. marimo for the data-heavy chapters. Students leave the course with a working development environment they can keep using, not a sandbox they have to abandon when the semester ends.

Industrial engineering students go on to do operations, supply chain, manufacturing, and consulting work. Environments where the boring stuff (reproducibility, dependency management, version control) actually matters more than syntactic depth in any one language feature.

The curriculum: basic Python, then advanced Python, then terminals and CLIs, then SQL with DuckDB, then tying it together. The “tying it together” piece is the part most computing courses skip and the part students need most. Knowing Python and knowing SQL is one thing. Knowing how a Python script reads a file, queries a database, and writes a report is the actual job.

Built in the open on GitHub, hosted on the same domain as the rest of my work. No paywall, no LMS, no login. If someone at another university wants to fork it and adapt it for their course, they can. I’d consider that a win.

> **NOTE:**
>
> - Sole instructor of record across two academic years, ~150 students total.
> - Free and public at [ozanozbeker.com/cw4e](https://ozanozbeker.com/cw4e).
> - Same Quarto pipeline drives the book and my client deliverables. The discipline transfers in both directions.

## What I learned

Teaching the same material to ~75 students at once is the most ruthless documentation review process I’ve ever been through. Stuff that makes sense to me and three colleagues will fall apart against an actual cohort with mixed backgrounds and no incentive to be charitable. Every semester surfaces dozens of small things. An instruction that assumes a step I forgot to mention. An error message that means something different on Windows than on macOS. A paragraph that scans cleanly to me but parses ambiguously to a 19-year-old hitting the concept for the first time. The book is better than my client documentation is, and it’s because the feedback loop is faster and louder.

The other thing this clarified. The gap between “knows Python” and “can ship something” is not a Python gap. It’s a tooling, environments, terminals, and version-control gap. Students who fight Python concepts catch up by midterms. Students who don’t have a working dev environment fight every assignment for the whole semester. Front-loading the boring stuff is the single highest-leverage curriculum decision I’ve made.
