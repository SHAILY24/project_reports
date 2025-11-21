# Portfolio: project write-ups

A few software projects I've worked on, written up with the metrics, the architecture decisions, and the parts that were hard. Client names and sensitive details are stripped; the technical story is intact.

## What's here

Each write-up covers a real project that shipped to production. They include before/after numbers, the architecture choices I made and why, and the bugs that ate the most time.

Things I tend to work on:

- Performance work. Several of these involved 10x to 30x speedups from rethinking the architecture, not micro-tuning.
- API work, including reverse-engineering undocumented endpoints when no SDK exists.
- Automation that runs unattended and recovers from its own failures.
- Full stack, backend through to the interface users actually touch.

Languages and tools I reach for: Python, JavaScript/TypeScript, Go, Rust. Async with Trio and asyncio. HTTP/REST, WebSockets, browser automation. CI/CD, containers, the usual deployment plumbing. Git on Linux, Windows, and macOS.

## Featured project

### [Slack analytics automation](./slack_report_generator.md)

20-30x faster, fully automated, runs against a 228-user workspace.

A volunteer org had a slow manual Slack analytics script. I rebuilt it. Report generation dropped from 6-10 minutes to 15-20 seconds, and the whole thing now runs on a weekly/monthly schedule with no one watching it.

Stack: Python, Playwright, httpx, Trio, async/await, rate limiting with exponential backoff. The interesting bits:

- Reverse-engineered Slack's internal search API by watching network traffic, instead of driving the UI.
- Structured concurrency with Trio for parallel queries.
- API-first with a browser-automation fallback when a request fails.
- Works the same on Windows, macOS, and Linux.

## How I work

Every write-up here leads with the numbers because that's what mattered to the client: before/after timings, friction removed, downtime avoided, room to grow.

On the engineering side I care about modular code with clear boundaries, error handling that degrades gracefully instead of crashing, and documentation a non-developer can follow. The approach is usually the same: find the root cause before touching anything, weigh a few designs, build in small steps with validation, then hand off with notes someone else can act on.

## By the numbers

Across these projects:

- 20-30x performance gains from architectural changes
- Manual workflows taken to 100% automated
- ~100% success rates once retry and fallback logic was in place
- 228+ users supported in production
- thousands of end-user hours saved

## Links

[![GitHub](https://img.shields.io/badge/GitHub-SHAILY24-181717?style=for-the-badge&logo=github)](https://github.com/SHAILY24)
[![Portfolio](https://img.shields.io/badge/Portfolio-project__reports-blue?style=for-the-badge)](https://github.com/SHAILY24/project_reports)

New write-ups get added as projects wrap. Each one has the problem statement, the architecture and the decisions behind it, code samples, the results, and what went wrong along the way.

All projects keep client confidentiality. Organization names, URLs, and personal identities are removed; the technical and business detail stays.

## License

Project documentation here is available under the MIT License. Code samples are for demonstration.
