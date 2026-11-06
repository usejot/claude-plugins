# Jot for Claude Code

Work through Jot feedback: bug reports and change requests that people pinned to elements of the live app, with screenshots and recordings. Use when the user mentions Jot, feedback, a bug report, a pin or a comment id, or asks what is open, to fix reported issues, or to reply to reviewers.

```
/plugin marketplace add usejot/claude-plugins
/plugin install jot@usejot
```

Then ask Claude to connect Jot (it opens your browser), or paste an agent token when Claude Code asks.

- `/jot:jot`: open feedback
- `/jot:jot <id>`: fix one
- `/jot:jot loop`: propose a fix for each new comment as it lands (`loop fix` applies it)

Generated from the Jot contract 0.9.0. Docs: https://docs.usejot.co/docs/ai/mcp
