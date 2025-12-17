# Multi-Portal Scraping With Anti-Bot Bypass and Fuzzy Matching

Records had to be pulled from several separate portals, normalized to one schema, deduplicated against a master database, and scored so the output is a ranked list rather than a raw dump.

## What I did
Built a pipeline that ingests records from multiple portals, normalizes them to one schema, fuzzy matches new records against a master database, classifies and scores each record, and exposes everything through a web dashboard with export.

## Approach
The first piece was a CSV-to-CSV task: clean and fuzzy match two messy CSVs. I shipped that as both a single-file and a modular version, and it grew into a full system. The matching layer normalizes names (strips suffixes, expands abbreviations, handles plural and singular), phone numbers (digits only), and addresses (region and postcode extraction). Scoring combines RapidFuzz token_set_ratio, token_sort_ratio, and partial_ratio with weighting, plus extra signals: a phone match raises confidence, a region or category mismatch drops it. Records split into auto-match, manual review, and new. Scrapers cover several portals. Some expose clean data through an open API. Others only return detail on per-record pages, so the scraper walks detail pages. One portal caps results to a rolling window, which is a portal limitation. One sits behind an aggressive bot-detection layer, which needed an anti-detect browser, rotating fingerprints, and a proxy provider that does not block the relevant host class. The first proxy vendor silently blocked that host class, so I isolated it with controlled tests and switched providers. On top of the raw data sits a classification layer: a dictionary of known entities plus fingerprint-style matching for variants, and an adaptive queue that promotes an unknown entity once it recurs enough. Each record gets category and type flags, a 0-to-100 score from several signals, and a bucket. Confidence-banded merge rules mean high-confidence matches enrich the master record automatically and the rest go to a review queue. Hash-based change detection means daily runs only touch new or changed records.

## Stack
Python, RapidFuzz, pandas, SQLite for the master and audit tables, an anti-detect Firefox-based browser for the protected portal, residential proxies, a web dashboard with scrape jobs, filtering, a pending-entities queue, and CSV export.

## Result
Collection went from manual lookups to scheduled scraping across several portals with deterministic ingestion. The early CSV matching step cleared the sample data cleanly and locked the schema and scoring before scraping replaced it. The classification layer turns raw records into a ranked list, and every match decision is logged with its score so any merge can be audited.
