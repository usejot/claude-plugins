---
description: Work through Jot feedback (list, fix one, or loop on new ones)
argument-hint: "[comment id | loop | loop fix]"
---

Arguments: $ARGUMENTS

Jot holds feedback people left on the live app: each comment is pinned to a page element, with a screenshot, browser details and sometimes a session recording. Use the jot MCP tools (fetch_feedback, get_comment, update_status, post_reply). If they are not loaded, the `jot` CLI does the same: `jot list`, `jot show <id>`, `jot status <id> <STATUS> -m "…"`, `jot reply <id> -m "…"`.

- No argument: list the open feedback (fetch_feedback with status OPEN, or `jot list`). Show one line each and ask which to take.
- A comment id: get_comment for the full repro, find the cause in this codebase, fix it, run the project's tests, then update_status RESOLVED and post_reply saying what changed (files, PR link).
- `loop`: repeat until stopped:
  1. Run `jot watch --once` with the Bash tool. It waits for the next new comment and prints it.
  2. update_status IN_PROGRESS on it.
  3. Propose the change as a short plan and wait for the user's OK before editing anything.
  4. Once approved: implement, run the tests, update_status RESOLVED and post_reply with what changed.
  5. Go back to step 1.
- `loop fix`: the same loop, but implement straight away instead of waiting for an OK.

RESOLVED means "please check", not "closed". Approval follows the workspace's policy and the API refuses if you may not approve: set APPROVED only when the user asks, and never say a fix is approved unless the API accepted it.
