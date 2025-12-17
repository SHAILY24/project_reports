# Reverse-Engineering a Signed-Request API for Headless Extraction

A large paginated catalog had to be pulled off a site and turned into a searchable internal app, but the site's API signed every request with an encrypted token generated in the browser.

## What I did
Built a scraper that reverse-engineers the signed request header so the internal API can be called directly instead of driving a browser, then a recoded version that puts completeness over speed, then planned and built a search app on top of the collected data.

## Approach
The API rejects any request without a valid signed header that the frontend builds per call, including an encryption step. I worked out how that token is constructed and reproduced it, which made the whole catalog reachable as plain API calls and exported to spreadsheets with each section's extra attribute columns kept separate rather than flattened. The first pass missed the second page of every paginated request and a block of records at the tail. I rebuilt the pagination and rerun logic so the scraper resumes from where it stopped and treats "miss nothing" as more important than throughput, because the site blocks proxies hard and runs need to survive IP rotation and restarts. The work then expanded across many more sections plus a web app: a browse page, search with input sanitization, a basket with session persistence and bulk spreadsheet upload, and an on-prem deployment with a tunnel for remote access and a LAN fallback. The data model collapses to three lean tables with a single lookup path so search stays fast on a large database.

## Stack
Python for the scraper, reverse-engineered request signing, residential proxies with rotation and resume, a normalized relational schema, a Next.js search app, on-prem deployment with a tunnel plus LAN fallback.

## Result
A site that could not be touched without a browser became fully scriptable through its own API. One full section was captured with no missed records after the pagination fix, the approach generalized to many more sections, and the data fed a self-hosted search app used internally.
