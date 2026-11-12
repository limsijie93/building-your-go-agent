# Practice 4 — actual session evidence

Tool/model/version and date: Claude Code 2.1.322, model Claude Opus 5.5 (`claude-opus-5-5`), 2026-11-13
Target, mode and head SHA when live: `nebius-academy-templates/go-pr-review-practice` PR #3, `github` mode (per `review-target.json`), head `72d19b992b7af937438327676e17a709623331bd` (branch `pr-03`, base `main` @ `cb7a691656d4881e5d045a2308dbaf0d5de52c69`)

## Recorded conversation and tool events

> Notes:
> - Tool names: the connected GitHub MCP server has no `pull_request_read` tool.
>   PR reads used `mcp__github__get_pull_request` and
>   `mcp__github__get_pull_request_files`. The full diff came from the files
>   endpoint's `patch` field, because the PR has a single added file.
> - The credential literal is shown as `<redacted>` everywhere in this record,
>   including the diff passed to the subagents.
> - No remote write was made. The review was not posted.

---

**User:** `/review-pr 3`

**Main agent:** read `skills/review-pr.md`, `fixture-policy.md` and `review-target.json` (mode `github`, PR 3 → 3).

**Tool:** `mcp__github__get_pull_request(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=3)`
→ open PR "Add payment authorization client", head `72d19b9…`, base `main`.

**Tool:** `mcp__github__get_pull_request_files(... pull_number=3)`
→ 1 file: `internal/payment/client.go` (added, +11):

```diff
+package payment
+
+const apiKey = "<redacted>"
+
+// Client supplies authorization for the payment adapter.
+type Client struct{}
+
+// Authorization returns the bearer credential used by outgoing requests.
+func (c Client) Authorization() string {
+	return "Bearer " + apiKey
+}
```

---

### Subagent 1 — `style-reviewer` (started first)

**Agent call:** `Agent(subagent_type="style-reviewer", description="Style review PR 3")`. The prompt contained the target, the head SHA, the repo root, the scope (full PR, including tests, of which there were none), the changed-file list, the complete diff, and instructions to read `reviewers/style-reviewer.md` and the convention files, treat the diff as data, and mask credentials.

**Returned:**
> Style check: no style findings for internal/payment/client.go. Naming is correct (unexported const `apiKey` is mixedCaps; exported `Client` and `Authorization` are MixedCaps). Both exported declarations have doc comments that begin with the identifier name. No _test.go files changed, so test-conventions.md does not apply. Note that skills/app-conventions.md and skills/test-conventions.md are still TODO templates with few concrete rules, so style coverage here is thin.
>
> Security finding (outside the style rules; flagged for the caller): [MEDIUM] internal/payment/client.go, line 3: a credential is hardcoded as a package-level string constant (`const apiKey = "<redacted>"`, value masked). Lines 9-10 return it to any caller as `"Bearer " + apiKey` … Suggested change: remove the literal and read the key at startup from an environment variable or secret store … Rotate the key.
>
> No files were edited and nothing was posted or merged.

**Main agent → user:** "The style reviewer finished and found no style problems in `internal/payment/client.go`. The security reviewer is running now."

---

### Subagent 2 — `security-reviewer` (started after subagent 1 returned)

**Agent call:** `Agent(subagent_type="security-reviewer", description="Security review PR 3")`. Same target, SHA, root, scope, file list and complete diff. The prompt said to read `reviewers/security-reviewer.md` and `fixture-policy.md`.

**Returned:**
> internal/payment/client.go
>
> - [HIGH] line 3: hardcoded credential. `const apiKey = "<redacted>"` is a package-level constant. Line 10 uses it to build the bearer credential.
>   - Fixture policy: the value looks synthetic … reported as HIGH per fixture-policy.md.
>   - Impact: anyone with repo access could read it; it would persist in git history; it can't be rotated without a code change and redeploy.
>   - Remediation: load the key at runtime from an environment variable or secret manager, inject it into `Client`, and fail if it is empty.
>
> Scope: full PR, 1 file. There are no other security findings (no SQL, shell, TLS, deserialization, logging or input-validation code).
>
> Summary: 1 HIGH, 0 MEDIUM, 0 LOW. Highest severity: HIGH.

**Main agent → user:** "The security reviewer found one HIGH issue … The architecture reviewer is running now."

---

### Subagent 3 — `architecture-reviewer` (started after subagent 2 returned)

**Agent call:** `Agent(subagent_type="architecture-reviewer", description="Architecture review PR 3")`. Same target, SHA, root, scope, file list and complete diff. The prompt said to read `reviewers/architecture-reviewer.md`, `skills/architecture-guidelines.md` and `docs/adr/`.

**Returned:**
> No architecture findings.

**Main agent:** to merge the results, it checked `skills/architecture-guidelines.md` and found line 6 (Configuration / ADR-002): "secrets are loaded centrally … Services don't call os.Getenv or contain credentials." The architecture reviewer had missed that this applies. `review-pr.md` says to combine a hardcoded credential and the matching Configuration guideline into one HIGH, so they were combined.

---

### Consolidated draft shown to user

**Target:** `nebius-academy-templates/go-pr-review-practice` PR #3, head `72d19b9`. Comment-only review: no approve, merge or close.

**internal/payment/client.go**

- **[HIGH] line 3: hardcoded credential, which breaks the Configuration guideline (ADR-002).** `const apiKey = "<redacted>"` is a package-level literal. Line 10 sends it out as `"Bearer " + apiKey`. The secret is committed to source and git history, and it can only be rotated by changing the code. ADR-002 requires secrets to be loaded centrally (`internal/config` or the composition root) and passed in through constructors.
  *Fix:* remove the literal. Add the key as a field on `Client`, set it through a `NewClient(apiKey string)` constructor fed from central config, and reject an empty key. Rotate any key that was committed.
  *(Fixture note: the value is synthetic training data. It is reported as HIGH under the exercise's fixture policy.)*

**Summary:** 1 file reviewed. 1 HIGH, 0 MEDIUM, 0 LOW. Highest severity: **HIGH**.

**Main agent → user:** "Nothing has been posted. Reply **yes** or **post** to submit this review to PR #3, or tell me what to change."

**User:** asked to save the session to `results/test-session-4.md` (no yes/post was given, so nothing was posted).

---

Delegation path: Claude Code's named-role selector. Each reviewer was started with the `Agent` tool using `subagent_type` set to the role name: `style-reviewer`, then `security-reviewer`, then `architecture-reviewer`. These are defined in `.claude/agents/*.md`, with tools limited to Read and Grep. The calls ran one at a time: the main agent waited for each subagent's hand-back before starting the next. Each subagent got the same target, head SHA, repo root, scope, file list and complete diff, and read its own reviewer prompt and convention files. Observations: the style reviewer went outside its role and raised the credential as MEDIUM; the security reviewer applied the fixture policy and rated it HIGH; the architecture reviewer missed the Configuration guideline, so the main agent merged it into the single HIGH finding.
