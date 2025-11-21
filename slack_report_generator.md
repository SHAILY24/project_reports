# Slack analytics automation: performance work

## Overview

A volunteer organization needed activity stats pulled from their Slack workspace on a regular basis. The script they had was slow, broke often, and someone had to run it by hand every time. I came in to make it fast, modernize it, and get it running on its own.

## Results

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Processing time | 6-10 minutes | 15-20 seconds | 20-30x faster |
| User interaction | manual run each time | unattended | fully automated |
| Reliability | failed often | retries plus fallback | ~100% success |
| Date ranges | one hard-coded range | multiple modes | flexible |

## The problem

The client had inherited a Python script that drove Slack's UI with Playwright, typing each search into the web app by hand. It walked 228+ users one at a time, needed the browser window open and visible, and broke whenever Slack changed a UI element. A full report took 6-10 minutes. There was no retry logic and only one hard-coded date range.

What they asked for: make it fast. Run weekly and monthly on a schedule without anyone touching it. Support custom date ranges. Handle failures and retry. Keep the commands simple enough for non-technical users. Work on Windows, macOS, and Linux.

## Solution

### Phase 1: move off the browser, onto the API

Instead of automating clicks, I reverse-engineered Slack's internal search API:

```python
# Before: browser automation (slow, fragile)
page.click(SEARCH_BAR)
page.type(SEARCH_INPUT, query)
page.keyboard.press("Enter")
sleep(1.2)
count = page.inner_text(RESULTS_COUNT)

# After: direct API calls (fast, reliable)
response = await client.post(
    api_url,
    content=captured_request_data,
    headers=authenticated_headers,
    cookies=session_cookies
)
total_count = response.json()['pagination']['total_count']
```

I intercepted the network requests to find the real endpoint and its parameters, pulled auth tokens out of the browser session cookies, and kept the session around so it didn't have to log in every run.

### Phase 2: parallel processing with Trio

Concurrent async processing so multiple users get queried at once:

```python
async def process_users_parallel(self):
    # Process 5 users concurrently with rate limiting
    max_concurrent = 5
    semaphore = trio.Semaphore(max_concurrent)
    
    async with httpx.AsyncClient() as client:
        async with trio.open_nursery() as nursery:
            for user_id, user_info in users.items():
                nursery.start_soon(
                    search_user_async,
                    client, user_id, semaphore
                )
```

This is where the rate limiting came in: exponential backoff on 429s, jitter so the requests don't all retry in lockstep, and a browser fallback for any request the API couldn't satisfy. Windows also needed its Trio signal-handling warnings dealt with.

### Phase 3: error handling

A three-layer recovery path. The API mode retries up to five times. If that still fails, it falls back to browser automation. If that fails too, it writes "N/A" and moves on rather than crashing the run.

```python
for attempt in range(retries):
    try:
        count = await search_via_api_async(client, query)
        break
    except HTTPStatusError as e:
        if e.response.status_code == 429:
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            await trio.sleep(wait_time)
        else:
            raise
```

### Phase 4: scheduled runs

A scheduler that stays up and fires reports on its own:

- Weekly: every Friday at 12:01 AM, covering the previous 7 days
- Monthly: 1st of the month at 12:00 AM, covering the previous calendar month

```python
def run_scheduled(self):
    while True:
        now = datetime.now()
        
        # Check for monthly report (1st at midnight)
        if now.day == 1 and now.hour == 0 and now.minute < 5:
            generate_monthly_report()
        
        # Check for weekly report (Friday at midnight)
        if now.weekday() == 4 and now.hour == 0 and now.minute < 5:
            generate_weekly_report()
        
        sleep(60)  # Check every minute
```

### Phase 5: the CLI

A command line with several time-period modes:

```bash
# Quick presets
uv run python WEEKLY.py              # Current week
uv run python MONTHLY.py             # Current month

# Specific months (new feature!)
uv run python main.py --mode 01-2025 --api    # All of January 2025
uv run python main.py --mode 02-2025 --api    # All of February 2025

# Rolling windows
uv run python main.py --mode last-7-days --api
uv run python main.py --mode last-30-days --api

# Custom ranges
uv run python main.py --date 09-07-2025 --end-date 11-08-2025 --api

# Scheduled mode
uv run python main.py --scheduled --api --headless --open
```

## Stack

- Python 3.8+
- Playwright for session capture and the browser fallback
- httpx for async HTTP
- Trio for the structured concurrency
- termcolor for CLI output

Async/await throughout, exponential backoff with jitter for rate limiting, session persistence via browser storage state, and `uv` for package management. Runs on Windows, macOS, and Linux.

## Performance numbers

The speedup, roughly:

```
Browser Mode (Sequential):
228 users × 1.5s each = 342 seconds (5.7 minutes)

API Mode (Parallel, 5 concurrent):
228 users ÷ 5 × 0.3s each = 13.7 seconds

Speedup: ~25x faster
```

Rate limiting in practice: five users at a time, backoff at 1s/2s/4s/8s/16s, random jitter so retries don't synchronize, and 200-300ms between user requests (300-500ms between terms). API mode lands above 95% on its own; the browser fallback catches the rest, so no data is lost even under throttling.

## Session handling

Browser cookies are captured on first login and stored in `slack_session.json` (gitignored), then reused on later runs. No credentials live in code.

Platform notes: Windows needed UTF-8 console encoding fixes and the Trio signal-handling warnings suppressed. Path handling and file operations are kept portable.

## Documentation delivered

The end users were not developers, so the docs are plain:

1. README.md, quick start and common cases
2. HOW_TO_RUN_COMMANDS.txt, step by step
3. SCHEDULED_MODE_SETUP.txt, automation setup
4. Helper scripts: `open_cmd.bat` opens a prompt in the right directory, `start_scheduler.bat` launches scheduled mode

## Notable bug fixes

### F-string syntax error

Nested f-strings with conflicting quotes:

```python
# Before (syntax error)
f"Found cookies: {', '.join(found_important)}"

# After
cookies_str = ", ".join(found_important)
f"Found cookies: {cookies_str}"
```

### Build configuration

Hatchling couldn't find the package files:

```toml
# Added to pyproject.toml
[tool.hatch.build.targets.wheel]
packages = ["."]
```

### Browser fallback crash

`self.page` didn't exist yet when the fallback triggered:

```python
# Added in __init__
self.page = None

# Set when page is created
page = self.context.new_page()
self.page = page
```

## Before and after, in code

The old `main.old.py` was 162 lines: one execution mode, a hard-coded date calculation, no error handling, sequential only, no session management.

The new `main.py` is 1,916 lines and does a lot more: 10+ time-period modes, full error handling, parallel async processing, session persistence, scheduled runs, retry with backoff, cross-platform support, and detailed progress logging.

## What I took away from it

- Watching browser network traffic is often the fastest way to find an undocumented API.
- Trio's nurseries make parallel error propagation tractable instead of a guessing game.
- Backoff alone isn't enough; jitter is what stops the retry storm.
- The hybrid design traded a little speed for not losing data, which the client cared about more.
- Non-developers will use a tool only if the commands and docs assume nothing.

## Business impact

The client can now run a report in 15 seconds instead of 10 minutes, leave the weekly and monthly ones to the scheduler, pull a custom range on demand, and grow the user base without the runtime falling over. The docs mean someone else can change it later.

---

**Note**: This was done for a volunteer organization. The organization name, workspace URLs, and user identities are omitted to protect client privacy.
