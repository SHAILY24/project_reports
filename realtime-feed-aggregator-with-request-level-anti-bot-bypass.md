# Real-Time Feed Aggregator With Request-Level Anti-Bot Bypass

A real-time feed had to be assembled from several public RSS entry points and a supplementary API, without depending on the primary paid APIs in that space.

## What I did
Built an aggregator that pulls from RSS entry points and an API, follows through to the full source page, extracts and validates a key identifier, deduplicates across sources, and pushes new items to the browser over websockets.

## Approach
The entry point for each source is its public RSS by category. From there a worker fetches the full page, since the RSS body is truncated. A worker pool polls sources on a short interval. A cleaning step extracts a candidate identifier and validates it against a reference source so a string is only accepted if it actually resolves, and a dedup layer catches the same item republished across sources. Clean items land in a database and a pub/sub layer broadcasts them so the dashboard updates without a refresh. Several sources sit behind bot detection, including a Cloudflare layer, so I built request-level bypasses rather than driving a headless browser, which is easy to detect and slow at this cadence. The design goal was to behave like a normal logged-in browser kept active around the clock so the internal endpoints stay reachable. New sources attach as drop-in connectors without slowing the existing ones.

## Stack
Python for the scrapers and workers, a reference API for identifier validation and supplementary items, a queue for URLs to fetch, a database with deduplication, websockets and pub/sub for real-time push, request-level anti-bot bypass for the protected sources.

## Result
A live feed reachable in a browser with sub-five-second freshness on the integrated sources, with no paid subscriptions in the path. A working proof of concept whose architecture extends to more sources without rework.
