# Translation-API Integration With Edge-Case Handling

A small internal tool around a third-party translation API: paste text, pick languages, get the result, with history and basic accounts. The deliverable was a built and deployed working version to evaluate against.

## What I did
Built and deployed an app with the core flow working end to end: text input, source and target selection with auto-detection, the API call, and the translated result, with error handling for bad keys, quota, and timeouts, a character counter, and a keyboard shortcut, all containerized behind SSL.

## Approach
The work was in the translation API's edge cases. Sending an explicit auto value as the source language is rejected, so auto-detection means omitting the field entirely. Regional variants work only as targets, not as sources, so source codes have to be normalized down to the base language. The build also handles same-language dialect pairs so a regional pair does not silently fail. CORS is configured separately for development and production. The login system, persistent per-user history, per-user rate limiting, and export were scoped as a clear next phase rather than crammed into the proof of concept, with the demo showing the intended direction for the login and history screens.

## Stack
FastAPI backend with async HTTP, React frontend, Docker deployment with SSL, third-party translation API.

## Result
A clickable, deployed app covering the full translate flow with the API's auto-detect and regional-variant traps already handled, with the production scope laid out as the next phase.
