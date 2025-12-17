# Incremental Three-Way Sync Scraper With Cross-Source Dedup

A hosted database aggregated records from several sources. One more source had to be scraped under a fixed filter set, with the database kept in step with that source rather than only appended to.

## What I did
Built a proof of concept that pulls records matching the filters, maps them to the target schema, and reconciles against the existing database so each run inserts new records, removes records that disappeared from the source, and skips ones already stored.

## Approach
The reconciliation is the real work, not the extraction. Every run diffs the live results against what is stored: insert what is new, delete what no longer appears at the source, skip what is present and unchanged. It also avoids clashing with records already in the database from other sources, so a record counts as a duplicate when its identifying fields match, deliberately ignoring timestamp because the same item appears at different times across sources. Each run reports inserts, deletes, and skips so drift is visible. Records map straight into the existing schema with the full target column set so it drops into the current pipeline.

## Stack
Python scraper into the existing hosted database, a runnable batch entry point, three-way diff for insert, delete, and skip, cross-source dedup keyed on a stable identity tuple rather than timestamp.

## Result
A working proof of concept that scrapes the source under the filters and keeps the database in sync each run, including dedup against records already sourced elsewhere. It demonstrated the full sync behavior end to end against the live target.
