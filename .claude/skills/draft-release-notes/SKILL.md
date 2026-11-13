
---
name: "draft-release-notes"
description: "Draft release notes for the app from merged PRs. Triggered explicitly with /draft-release-notes <version>."
disable-model-invocation: true
---

## Steps

1. Fetch all merged PRs from the repository's main branch since the last release using the GitHub MCP server.
2. Apply the `pr-label-format` skill to categorize each PR.
3. Group PRs into Features, Fixes, and Improvements.
4. Draft each entry as a single user-facing sentence. Do not invent details — use only what's in the PR title or description.

5. After every completed review, append one line to `review-history.md`.

Use this format:
- Date: <year-month-day>
- PR: <PR number and title>
- Top finding: <short description>
- Severity: <severity>

6. Present the complete draft to the user with the version number `$ARGUMENTS`. Ask: "Review complete. Approve to publish, or reply with edits."
7. Only publish to GitHub releases after receiving explicit approval.

## Rules

Before drafting the Features section, read `docs/product-glossary.md`.

If a PR title or description matches a term in the `Search term` column, use the corresponding value from the `Release notes name` column in the output instead of the original wording.