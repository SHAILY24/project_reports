# Parallel-Fetch and Frequency-Normalized Ranking Script

A large list of items had to be pulled from an upstream data API and ranked, where a raw most-recent figure is not comparable across items because each reports on a different cadence.

## What I did
Built a script that collects the full list, pulls the current value and the most recent periodic figure for each, computes a normalized annualized figure, and writes a sorted spreadsheet so the strongest candidates surface first.

## Approach
The subtlety was cadence. Items report annually, quarterly, monthly, or weekly, so a raw most-recent figure cannot be compared directly. The script annualizes by cadence so every item sits on the same basis, and it keeps a cadence column so the underlying period stays visible. It pulls from the data source already in use so the figures stay consistent with what is trusted, with a fallback source for coverage. Fetching runs in parallel across the list. It runs on demand or on a schedule and logs each run, so an item with missing upstream data is visible rather than silently dropped.

## Stack
Python, an upstream data API with a fallback source, parallel fetching across the list, spreadsheet output, scheduled-run friendly with logging.

## Result
A manual review of a large list became one script run that returns a ranked short list, with cadence normalized so the ranking is meaningful. The same approach extends to a much larger universe, where the open technical question is reliable coverage at that scale.
