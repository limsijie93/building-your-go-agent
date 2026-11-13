# Observability table

Tool version: Claude Code 2.1.323 (CLI)
Plugin version: langsmith-tracing 0.6.5 (langsmith-claude-code-plugins)
Model: claude-opus-5-5 (orchestrator and all three reviewer subagents)
Token method: sum of `total_tokens` over the unique `llm` runs in each trace. Each model request is counted once. Inclusive parent counters (`Claude Code Turn`, `Agent`) are not added again. The figures include cached prompt tokens as LangSmith reports them. They come from LangSmith project "Release Notes Agent", session 2026-11-13, pulled with the runs query API.
Expense metric: tokens. Currency cost could not be reconciled from the plugin data.

| Input | Trace ID | Total tokens | Most expensive span | security-reviewer called? |
| --- | --- | --- | --- | --- |
| PR-01 | 01a1d306-5360-71f5-8b8a-c95e41783238 (shared batch trace for PRs 1, 2, 4, 5) | N/A: the batch trace mixes 4 PRs. All 4 batch traces total 1,158,650. The only span captured for this PR is style-reviewer at 23,338 | style-reviewer Subagent, 23,338 | Yes in session. N/A in trace: the security span was not captured |
| PR-02 | 01a1d306-5360-71f5-8b8a-c95e41783238 (shared), plus 01a1d307-9508-7cd9-80ba-280f1f1601f4 | N/A: shared batch (see PR-01). Spans captured for this PR: style 23,444 + architecture 36,093 | architecture-reviewer Subagent, 36,093 | Yes in session. N/A in trace: the security span was not captured |
| PR-03 | 01a1d301-ea93-7083-bc46-5c2bcf704f0d | 670,502 (19 llm runs) | llm 01a1d303-1753-7eb7-bb01-246b423b8bfb (orchestrator consolidation), 73,554. Largest reviewer span: architecture-reviewer Subagent, 35,453 | Yes. Span 01a1d302-8b73-7d0c-bbe6-b029c8036e91, 20,124 tokens |
| PR-04 | 01a1d306-5360-71f5-8b8a-c95e41783238 (shared), plus 01a1d307-9508-7cd9-80ba-280f1f1601f4 | N/A: shared batch (see PR-01). Spans captured for this PR: style 37,568 + architecture 36,994 | style-reviewer Subagent, 37,568 | Yes in session. N/A in trace: the security span was not captured |
| PR-05 | 01a1d306-5360-71f5-8b8a-c95e41783238 (shared) | N/A: shared batch (see PR-01). The only span captured for this PR is style at 51,787. The architecture Agent span was recorded with 0 tokens | style-reviewer Subagent, 51,787 | Yes in session. N/A in trace: the security span was not captured |

Identify the most expensive span by name or ID and its cost/token value. Record unavailable
metrics as `N/A: <reason>`. No separate model, parent ID or usage dump per reviewer is needed.

**How PRs 1, 2, 4 and 5 were run.** They were reviewed in one parallel turn. Each PR still ran style → security → architecture in order, but the four PRs overlapped. The plugin recorded one root trace for that turn, plus one extra trace for each subagent hand-back. It captured all 4 style-reviewer spans and the architecture spans for PR-02 and PR-04. It did not capture any of the 4 security-reviewer spans, the PR-01 architecture span, or the PR-05 architecture span's usage (recorded with 0 tokens). The session transcript shows all 12 reviewer runs completed. The trace does not prove this. To get one trace per PR with all three child runs, rerun them one PR per turn.

## PR-03 diagnostic
- Security reviewer called: Yes. `security-reviewer Subagent` run `01a1d302-8b73-7d0c-bbe6-b029c8036e91` is a child of trace `01a1d301-ea93-…-bc46-5c2bcf704f0d`. It ran after style-reviewer and before architecture-reviewer.
- Received the correct PR-03 diff: Yes. The input names PR #3 with head `72d19b992b7af937438327676e17a709623331bd` and contains the full 11-line `internal/payment/client.go` diff. The credential literal was masked by the orchestrator before the hand-off.
- Returned finding: `[HIGH] Line 3: hardcoded credential … Line 10 uses it to build the bearer credential`. It reported 1 HIGH, and the highest severity was HIGH.
- Finding preserved in final output: Yes. In the consolidated review, it was merged with the style reviewer's matching finding into a single `[HIGH] Hardcoded API credential (lines 3, 10). Violates ADR-002 Configuration`. The architecture reviewer deferred to security under ADR-002, so nothing was duplicated.
- Supported diagnosis: no failure. All three reviewers appear as sequential child runs with usage, and the HIGH finding survived consolidation.

## Saved span evidence
Case PR-03, trace `01a1d301-ea93-7083-bc46-5c2bcf704f0d`. Excerpts are redacted.

Security invocation (Agent → `security-reviewer Subagent` 01a1d302-8b73-7d0c-bbe6-b029c8036e91), input:
```
Target: nebius-academy-templates/go-pr-review-practice PR #3 "Add payment authorization client"
Head SHA: 72d19b992b7af937438327676e17a709623331bd
+const apiKey = "[MASKED]"
+func (c Client) Authorization() string {
+	return "Bearer " + apiKey
```
Returned finding:
```
[HIGH] Line 3: hardcoded credential. `const apiKey = "<redacted>"` … Line 10 uses it to build
the bearer credential … Summary: 1 HIGH, 0 MEDIUM, 0 LOW. Highest severity: HIGH.
```
Consolidated output (orchestrator, final turn of the same trace):
```
**[HIGH] Hardcoded API credential (lines 3, 10). Violates ADR-002 Configuration.**
Line 3 stores a credential directly in the code as a package-level constant, `const apiKey = "***"` …
Findings: 1 (HIGH 1, MEDIUM 0, LOW 0) · Highest severity: HIGH
```
