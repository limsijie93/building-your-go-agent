# GitHub MCP setup

Use [GitHub's hosted MCP server](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
from the course walkthrough. Docker is not required. Keep a GitHub personal access token
in `GITHUB_PERSONAL_ACCESS_TOKEN` in the environment of the client process, never in Git.
For the required read-only exercises, use repository Contents and Pull requests read access.

## Codex CLI

```sh
codex mcp add github --url https://api.githubcopilot.com/mcp/ --bearer-token-env-var GITHUB_PERSONAL_ACCESS_TOKEN
```

If `github` is already registered, update its existing entry instead of adding a second
server. In your local `~/.codex/config.toml`, the entry for the course reads should be:

```toml
[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_PERSONAL_ACCESS_TOKEN"
http_headers = { "X-MCP-Readonly" = "true" }
```

Start a new CLI session from the environment containing the token and use `/mcp` to
check the connection. `bearer_token_env_var` stores the variable name, not the token.
Codex Desktop is optional: use the same HTTP configuration and make the token available
to the app process. An app already running does not inherit exports from another terminal.

## Claude Code

Add the hosted server to the ignored project `.mcp.json` (merge with existing entries):

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}",
        "X-MCP-Readonly": "true"
      }
    }
  }
}
```

Claude Code expands the environment variable at startup. Approve the project server
and check `/mcp`. Keep `.mcp.json` and `.claude/settings.local.json` out of the submission.

## Check the connection

Follow the platform lesson using `pull_request_read` with `get`, `get_diff` and, when
needed, paginated `get_files`. Set `review-target.json` to `github` mode and retain the
course repository and PR 1–5 mapping. Save the requested MCP call, review and declined
confirmation in `results/test-session-2.md`. Record the endpoint and advertised tool
names; hosted releases are managed by GitHub. Client prefixes may differ.

The exercises need no remote writes. Optional posting uses `add_issue_comment` and
requires suitable write access, a connection that permits writes, and explicit approval
of the displayed review and target. Never approve, merge or close a PR.

## Optional local server

If hosted access is unavailable, use the [official Docker setup](https://github.com/github/github-mcp-server#local-github-mcp-server),
with `GITHUB_READ_ONLY=1`, the token passed by environment, and a pinned release tag.
Record the tag and transport. Existing Docker-based run evidence remains valid.

References: [Codex MCP](https://learn.chatgpt.com/docs/extend/mcp),
[GitHub Codex setup](https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-codex.md),
[Claude Code environment expansion](https://code.claude.com/docs/en/mcp#environment-variable-expansion-in-mcpjson).
