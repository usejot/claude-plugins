---
name: jot
description: Work through Jot feedback: bug reports and change requests that people pinned to elements of the live app, with screenshots and recordings. Use when the user mentions Jot, feedback, a bug report, a pin or a comment id, or asks what is open, to fix reported issues, or to reply to reviewers.
---

# Working a Jot queue

Jot holds feedback people left on a live app. Each comment is pinned to the exact element, with a screenshot, the page URL and selector (or the app screen), the browser and OS, and sometimes a session recording with its console errors and failed requests. Use the jot tools (or the `jot` CLI), never curl or the REST API: they already carry the token, the ids and the screenshot.

## The loop

1. **Fetch.** `fetch_feedback` with status OPEN lists the open work. No projectId is needed unless this repo is bound to one.
2. **Read the full repro.** `get_comment` for one comment: the reporter's sentence, where it was left, the thread, the recording summary and the screenshot.
3. **Reproduce.** Find the element from the selector (or the screen) in this codebase and the route from the URL. Read the console errors and failed requests before guessing.
4. **Mark it.** `update_status` IN_PROGRESS when you start, so nobody else picks it up.
5. **Fix and test.** Make the smallest change that fixes what was reported, run the project's tests, and add one when the bug could come back.
6. **Close the loop.** `update_status` RESOLVED, then `post_reply` saying what changed. Never resolve without a reply.

## What a good reply says

- What was wrong, in one sentence the reporter understands.
- What changed: the files, and the PR or commit.
- How to check it: the page, the steps, and the browser when it mattered.

For example: "The Pay button sat under the cookie banner on Safari, so clicks never reached it. Fixed the banner's stacking in src/components/CookieBanner.tsx (PR #204). To check: open /checkout in Safari and press Pay."

## Ask instead of fixing when

- the report is unclear, or you cannot find what it points at;
- it is a design or product decision (copy, layout, pricing, scope) rather than a bug;
- the fix belongs to another repo or project, or is far bigger than the report.

Ask in the thread with `post_reply`, leave the status as it is, and tell the user.

## Using the evidence

- **Selector** (`domSelector`): search the code for its ids, classes and test ids; the closest component that renders it is where to start. A comment from a native app names the screen and the element instead.
- **Screenshot:** what the reporter saw, with their marks on it. Compare it with what the code means to render.
- **Console errors and failed requests** (the recording summary): the first error, or the first 4xx/5xx, is usually the cause. Match a request's path to its handler.
- **Browser, OS and viewport:** reproduce in the same conditions. Many reports happen in one browser or at one width only.

## Never

- Never say a fix is approved. RESOLVED means "please check", not "closed". Approval follows the workspace's policy (by default only the reporter approves): set APPROVED only when the user asks, and say it is approved only when the API accepted it.
- Never resolve a comment without a reply that says what changed.
- Never work on comments outside the project this repo is bound to, unless the user asks.
- Never paste the token anywhere, or call the API by hand.

## Several at once

When one change fixes several comments, move them together with one `update_status` (`commentIds`) and post one `create_batch_note` (`comment_ids`, `message`) instead of a reply in every thread.

## Several accounts

Work and personal Jot accounts are profiles: with the `jot` CLI's server, `profiles` lists the ones on this machine, `use_profile` switches this session to another, and `connect` signs in to one more.

## Examples

### 1. A button that does nothing

The report: "Pay button does nothing on Safari" · https://shop.your-app.example/checkout · button#pay · Safari 18 / macOS · recording: 12s, 1 console error (TypeError: Cannot read properties of undefined (reading 'total') at checkout.ts:88)

```text
get_comment {"commentId":"cm_pay"}
update_status {"commentId":"cm_pay","status":"IN_PROGRESS"}
update_status {"commentId":"cm_pay","status":"RESOLVED"}
post_reply {"commentId":"cm_pay","message":"Pay read the cart total before the cart had loaded, so it threw on Safari and did nothing. It now waits for the cart (src/checkout/checkout.ts, PR #204), with a test. To check: open /checkout in Safari and press Pay."}
```

The reply:

> Pay read the cart total before the cart had loaded, so it threw on Safari and did nothing. It now waits for the cart (src/checkout/checkout.ts, PR #204), with a test. To check: open /checkout in Safari and press Pay.

### 2. One fix for several comments

The report: Three COPY comments: "Sign Up should be Sign up" on /, /pricing and /signup, each pinned to the header button a.signup

```text
fetch_feedback {"status":"OPEN","label":"COPY"}
update_status {"commentIds":["cm_a","cm_b","cm_c"],"status":"IN_PROGRESS"}
update_status {"commentIds":["cm_a","cm_b","cm_c"],"status":"RESOLVED"}
create_batch_note {"comment_ids":["cm_a","cm_b","cm_c"],"message":"The header button now says \"Sign up\" everywhere: one label in src/components/Header.tsx (PR #311). To check: the header on /, /pricing and /signup."}
```

The reply:

> The header button now says "Sign up" everywhere: one label in src/components/Header.tsx (PR #311). To check: the header on /, /pricing and /signup.

### 3. A request to ask about, not fix

The report: "Make the hero more exciting" · https://your-app.example/ · section.hero · label ENHANCEMENT

```text
get_comment {"commentId":"cm_hero"}
post_reply {"commentId":"cm_hero","message":"Happy to change it. What should change: the headline, the image, or adding motion? A link to a hero you like would help."}
```

The reply:

> Happy to change it. What should change: the headline, the image, or adding motion? A link to a hero you like would help.
