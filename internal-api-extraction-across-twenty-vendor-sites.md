# Internal-API Extraction Across Twenty Vendor Sites

A set of about twenty vendor sites had to be read by code for a list of item codes. Each site had its own layout, some with login walls and dropdown variant selectors.

## What I did
Built a CLI tool that takes a list of sites and item codes and returns the current values, one extraction module per site, designed to run unattended on a schedule with structured logging.

## Approach
Instead of rendering pages and clicking through variant dropdowns, I went after each site's underlying JSON endpoints, so one request returns the value and all variants at once. That removes the dropdown crawl entirely and keeps proxy use low, helped further by not loading images or other page assets. Login-gated sites authenticate with stored cookies. The parser is deterministic with no model in the data path. One site sat behind a Cloudflare anti-bot layer, so I built a request-level bypass for it rather than skip it. A couple of sites need proxies to avoid IP bans from a process that queries every code. One module per site behind a shared skeleton means a broken site is an isolated fix, not a rewrite. CLI-first so it sits on a server as a scheduled job.

## Stack
Python with a fast HTTP client, per-site request modules, cookie-based auth, residential proxies for the sites that ban aggressively, a Cloudflare bypass for the one protected site, structured logging, packaged for scheduled unattended runs.

## Result
Manual checking across the full set of sites became one scheduled command. Direct-API extraction also surfaced malformed values in the source data that a manual spot-check would miss, and the per-site structure means a layout change is a contained patch.
