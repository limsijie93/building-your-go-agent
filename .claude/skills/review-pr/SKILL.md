---
name: review-pr
description: "Run a complete Go PR review of a course PR. Explicit invocation only, via /review-pr <PR number> (e.g. /review-pr 3)."
disable-model-invocation: true
---

Read `skills/review-pr.md` relative to the repository root and follow its steps in order.
The shared workflow determines whether this practice uses subagents.

Before delegation is introduced (Practice 3), review in the main agent. Load
`app-conventions` only for changed application `.go` files and `test-conventions`
only for changed `_test.go` files, respecting the requested scope.

When the shared workflow calls for delegation (Practice 4 onward), use the Agent tool
with the named `style-reviewer`, `security-reviewer`, and `architecture-reviewer`
subagents. Run them one at a time and wait for each result before starting the next.
Pass all three the same target, head SHA, repository root, review scope and complete
diff. Let the subagents read their shared reviewer prompts and convention files.
If a named agent is unavailable, report the missing agent instead of simulating it.

PR number supplied by Claude Code: $ARGUMENTS.
