# Building Your Go Agent

Build a Go pull-request reviewer with **Codex CLI or Claude Code**. Codex Desktop is
an optional alternative; use a trusted project and submit tracing exercises manually.
Choose `codex` or `claude` in `review-target.json`; complete only the selected tool's setup.

Follow the **Building Your Go Agent lesson on the course platform** for the assignments,
steps and acceptance criteria. This repository supplies the working files, not a second
copy of the lesson. [Practice overview](practice-description.md) lists the seven stages.

- [GitHub MCP setup](setup/github-mcp.md): the hosted server is the default; Docker is optional.
- [LangSmith setup](setup/tracing.md): CLI, optional desktop and Claude Code configuration.
- [Course PRs](https://github.com/nebius-academy-templates/go-pr-review-practice): five
  independent review inputs, with matching local diffs in `samples/`.
- [Fixture policy](fixture-policy.md): keep the teaching inputs unchanged and redact credentials.

Shared instructions live in `AGENTS.md`, `skills/` and `reviewers/`. Native entry points
live in `.agents/` and `.codex/` for Codex, and `.claude/` plus `CLAUDE.md` for Claude Code.
Use `$review-pr 3` in Codex or `/review-pr 3` in Claude Code once the workflow is configured.

Keep cumulative work and the existing filenames. Save real, redacted evidence in
`results/`, use LF endings, and keep credentials and local configuration out of Git.
For observability, fill the five-row table and save the short PR-03 diagnostic excerpt;
full exports for every reviewer are not required. Automated checks run before an AI
reviewer reads the committed files. It cannot run code or open trace links.
