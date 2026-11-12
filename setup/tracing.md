# Trace the reviewer, not the Go application

Use a free LangSmith Developer personal account and an API key: no card is needed for
up to 5,000 traces/month ([billing](https://docs.langchain.com/langsmith/billing)). Base
traces are retained for 14 days ([pricing](https://www.langchain.com/pricing)); check
these limits before running, and keep the course on the free allowance.
Save evidence promptly: the AI reviewer cannot open links and traces are not permanent.
Only enable tracing in the isolated course workspace: plugins upload conversation and
tool-call content. Do not enable it in a private working conversation.

## Codex CLI

Requirements: **Codex CLI 0.153.4+**, **Node.js 22+**, and enabled/trusted synchronous
`UserPromptSubmit` hooks. Follow the [official integration](https://docs.langchain.com/langsmith/trace-with-codex)
and [plugin README](https://github.com/langchain-ai/langsmith-codex-plugins).

```sh
codex plugin marketplace add langchain-ai/langsmith-codex-plugins
codex plugin add tracing@langsmith-codex-plugins
```

Enable the plugin in your local `.codex/config.toml` (ignored) if installation has not:

```toml
[plugins."tracing@langsmith-codex-plugins"]
enabled = true
```

Use `/hooks` to review and trust the installed plugin hooks. Enabling a plugin alone does
not trust them. In the shell that starts Codex, set `TRACE_TO_LANGSMITH=true`,
`LANGSMITH_CODEX_API_KEY`, `LANGSMITH_CODEX_PROJECT=go-pr-review-agent`, and
`LANGSMITH_CODEX_METADATA` with case, actual PR number, tool and head SHA.
For EU accounts set `LANGSMITH_CODEX_ENDPOINT=https://eu.api.smith.langchain.com`.
The plugin also supports ignored `.codex/langsmith.json`; never commit API keys there.

## Codex Desktop (optional)

The CLI setup above is not proof that the desktop app loads the same plugin or hooks.
Shell exports may not reach an already running desktop app. Check plugin availability,
enablement and hook trust in the actual desktop client; use the installed integration's
supported local configuration (including ignored `.codex/langsmith.json` when supported)
instead of assuming the launch-shell environment is shared.

Before Practices 6/7, run the PR-03 trace check below in the desktop app itself.
Explicit instruction handoff to real children establishes delegation, not trace capture:
the integration must still record those children and their usage. If that check fails,
report the missing capability and use the documented Codex CLI route for Practices 6/7,
recording the client change and rerunning the five final cases there. Do not present a
CLI trace as desktop evidence. Check the saved content as well as span counts: a trace
with `ls_tracing_mode: metadata` or “content omitted because tracing is muted” does not
prove input coverage or returned findings. Confirm synchronous `UserPromptSubmit` hook
execution. The integration's exact `langsmith-tracing:unmute` command changes the next
turn's privacy preference; use it only in a test chat whose content you intend to upload,
then run a fresh review. Send the command as plain text, without backticks or extra
whitespace. Verify the hook/privacy state and actual cloud content: an ordinary
assistant acknowledgement neither proves nor disproves that the hook processed it.
In desktop 26.1016.51333 with plugin 0.3.0, manually submitted messages produced full
cloud content, while programmatic chat creation/message dispatch produced metadata-only
traces. Submit the actual review manually through the desktop composer and verify its
new child runs; a previous full-content turn does not establish the next turn's mode.
Do not manually invoke hooks to manufacture launch evidence.
A full desktop-app restart was not required to obtain full content in this test: full
manual-message traces existed before the restart. After changing project instructions
or roles, open a new chat in the trusted project and verify discovery. Restart only
when a changed setting or the installed plugin explicitly requires it; do not use
restart as a substitute for checking hook trust and trace content.

## Claude Code

Use the [official integration](https://docs.langchain.com/langsmith/trace-claude-code)
with Node.js and the plugin version you record in your results:

```text
/plugin marketplace add langchain-ai/langsmith-claude-code-plugins
/plugin install langsmith-tracing@langsmith-claude-code-plugins
/reload-plugins
```

Set `TRACE_TO_LANGSMITH=true`, `CC_LANGSMITH_API_KEY`,
`CC_LANGSMITH_PROJECT=go-pr-review-agent` and `CC_LANGSMITH_METADATA` in the launching
shell or ignored `.claude/settings.local.json`.
For EU routing use the documented replica configuration: `CC_LANGSMITH_RUNS_ENDPOINTS`
is a JSON array with one object containing `apiUrl: "https://eu.api.smith.langchain.com"`,
your `apiKey` and `projectName: "go-pr-review-agent"`. This overrides other destination
settings; keep the entire value local. Do not assume generic `LANGSMITH_ENDPOINT`
controls this plugin. Use the installed plugin's documented redaction settings.

## Redaction and reproducibility

Keep built-in secret redaction enabled. The synthetic fixture strings may not match
its default patterns: add exact-value redaction for the credential literals in
`samples/pr-03-security.diff` and `samples/pr-05-mixed.diff`, using the installed plugin's
custom redaction mechanism: `LANGSMITH_CODEX_REDACT_EXTRA` for Codex or
`CC_LANGSMITH_REDACT_EXTRA` for Claude, a JSON array of regex `pattern` and `replace`
objects. Use `[REDACTED]` as the replacement; escape regex metacharacters in literal
values. Test sanitized outgoing data locally before uploading;
redact tool inputs/outputs as well as final answers. Replace complete credential values
with `[REDACTED]` while preserving the hardcoded-credential finding.
Never claim local collection proves successful LangSmith cloud delivery.

If you change shell environment variables, exit the running CLI and start it again
from that shell so the new process inherits them; no computer reboot is needed.
Desktop does not automatically inherit exports from an unrelated terminal.
Record CLI, plugin and actual model versions.
Start with PR-03: verify one completed cloud trace has the correct PR, three sequential
subagents as child runs, usage and preserved HIGH output before running all five cases.
Stop and report the concrete limitation if any part is missing. Do not simulate
subagents or fabricate trace identifiers.

## Evidence to save

Fill `results/observability-table.md` with five rows: trace ID, total tokens, the most
expensive span (name or ID and cost/token value), and whether security was called.
Record tool/plugin/model versions and the counting method once above the table.

For PR-03, save a short redacted excerpt showing the security invocation and relevant
input, its returned finding, and the corresponding consolidated output. Explain whether
the finding survived and what caused any failure. A supported no-failure result is valid.
No exports of every reviewer, parent IDs or per-reviewer model/usage records are required.
If a final score fails, save only the additional excerpt needed to explain that failure
in `results/evaluation.md`.

Count model-request usage once. Do not add inclusive parent totals or cached subsets
again. With Codex plugin 0.3.0, UI totals may undercount and wait/LLM events may repeat
usage: use unique request records or exclusive per-agent counters. For resumed chats,
exclude earlier turns. Briefly state the method; copying the raw counters is optional.
Use tokens if currency cost cannot be reconciled. Mark unavailable values `N/A: <reason>`
and report the limitation rather than inventing measurements. Inspect child runs in
LangSmith, but commit only the table and diagnostic excerpt needed for offline assessment.
