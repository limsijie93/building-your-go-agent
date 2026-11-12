<!-- TODO: Fetch via GitHub MCP or explicit local mode, route file types, review, consolidate, and ask before posting. Add sequential real delegation in Practice 4; when Codex has no named-role selector, pass the corresponding TOML developer_instructions to a real child explicitly. -->

## Steps

1. Read the case number from the request and look up the PR in review-target.json.
2. Fetch the PR with the GitHub MCP server (pull_request_read with get and get_diff).
3. Review only the files in the requested scope.
4. Apply app-conventions to .go files that don't end in _test.go, and test-conventions to files ending in _test.go. Load each skill only if the scope has that kind of file.
5. Check the security rules from AGENTS.md.
6. Compile the findings in your output format.
7. Show the complete review and the target, and ask for confirmation before posting.
8. Post only after a later, explicit yes or post. Never approve, merge or close a PR.
