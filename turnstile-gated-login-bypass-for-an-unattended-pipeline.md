# Turnstile-Gated Login Bypass for an Unattended Pipeline

An automation that uploads a file to a third-party portal and pulls back generated output stalled at the login screen. A Cloudflare Turnstile challenge blocked the browser-automation library it ran on.

## What I did
Built a working login path that clears the challenge without any paid captcha-solving service, and moved the existing automation off the detected library onto one that survives the challenge. The manual login and solve steps are now automated, so the run goes end to end with no human at the keyboard.

## Approach
The block was library fingerprinting, not the challenge itself. The automation library's traffic is already fingerprinted at the network edge, so tuning the browser does not help. I showed the login also works as a direct signed POST to the portal's internal endpoint, which returns the session cookies. Those cookies can be injected into the browser, so the challenge has to clear once and the session stays warm after that. Where the browser path had to stay, I swapped the detected library for a drop-in anti-detect replacement so the rest of the code did not change, and removed every wait-for-human step from the flow. I delivered it on a branch with a written guide so the same swap could be applied across the wider codebase.

## Stack
Node browser automation moved to an anti-detect drop-in, a reverse-engineered login endpoint with cookie injection, session persistence to avoid re-solving, no third-party captcha solver.

## Result
The login path that had blocked the pipeline now works with no paid solver in the loop. The challenge clears once per session instead of every run, and the library swap is documented so it generalizes across the codebase.
