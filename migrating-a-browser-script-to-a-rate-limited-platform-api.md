# Migrating a Browser Script to a Rate-Limited Platform API

A fragile browser-driven script counted activity on a third-party messaging platform and dumped it to a spreadsheet. It only ran while someone kept a browser window open, could not take a date range, and could not run on its own.

## What I did
Reworked it to query through the platform API instead of a browser session, take any chosen date range, and run a scheduled mode that produces recurring reports automatically and can email them with no one running anything.

## Approach
The original code drove a logged-in browser session, which is why it died the moment the window closed and could not run on a server. I moved it to API requests so it can sit on a server around the clock, generate the reports, and send them. I added a date-range mode so a historical pull could run immediately against a near-term need, then layered scheduled mode on top with fixed recurring runs. The platform rate-limited the bulk queries, so I added pacing and retry handling so a large pull slows down instead of failing. The counting logic runs against a configurable keyword list and matched the previous manual counts on a verification run.

## Stack
Python, platform API with token-based session auth, rate-limit pacing and retry, scheduled mode for recurring runs, spreadsheet output, email delivery.

## Result
A script that needed a babysat browser window became a server job that emails recurring reports on its own. The verification pull matched the prior known-correct count, and the date-range mode covered the time-critical pull before the schedule took over.
