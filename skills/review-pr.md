<!-- TODO: Fetch via GitHub MCP or explicit local mode, route file types, review, consolidate, and ask before posting. Add sequential real delegation in Practice 4; when Codex has no named-role selector, pass the corresponding TOML developer_instructions to a real child explicitly. -->

## Steps

Fetch the complete PR diff and changed-file list with the GitHub MCP server (pull_request_read with get, get_diff, and get_files), keeping the scope you ask for.
Delegate to style-reviewer, then security-reviewer, then architecture-reviewer, in this order, waiting for each one to finish before starting the next.
Pass each subagent the same complete in-scope diff and changed-file list, including test files.
Stop and report an incomplete review if any subagent fails.
Merge the three results into one review, organized by file, without duplicates. A hardcoded credential and the matching Configuration guideline become one [HIGH] finding.
Mask credential values.
Show the complete review and the target, then ask for confirmation before posting.
Post only after a later, explicit yes or post.