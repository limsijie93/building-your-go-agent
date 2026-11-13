# Practice 5 — actual session evidence

Tool/model/version and date: Claude Code 2.1.323, model Claude Opus 5.5 (claude-opus-5-5), 2026-11-13
Target, mode and head SHA when live: nebius-academy-templates/go-pr-review-practice, github mode (review-target.json `"mode": "github"`), fetched via GitHub MCP
- PR #1 "Add ride history service" — head 2d551765be07e61e70f70245c9f7be2943f901f4
- PR #2 "Add notification scheduling manager" — head 6a01fd4b95c55923ff33993c7a48a940f9e9f8f8
- PR #3 "Add payment authorization client" — head 72d19b992b7af937438327676e17a709623331bd
- PR #4 "Add trip loading service" — head baa1692dbb1ee9e1c1cb91e2428aebbe38ec1a95

## Recorded conversation and tool events

Credential values are redacted (`training-***`). Nothing was posted to GitHub in this session; all four drafts are still awaiting an explicit yes/post.

### 1. History question

**User:** Were the reviews appended to review_history.md?

**Tool event:** `git status --short; cat review_history.md` →

```
# Review history

<!-- Practice 5: append real completed runs. Clean runs use NONE. -->
```

**Agent:** No. `review_history.md` still contains only its heading and the Practice 5 placeholder comment. Nothing told me to write it: `AGENTS.md` says "introduce history writes in Practice 5", and `skills/review-pr.md` has no history step. Offered to append the three runs.

**User:** Yes lets add the reviews for PR 1 2 and 3 into the review_history.md

**Tool events:**
- `git diff AGENTS.md` showed the new uncommitted history rule: one line per completed review, `YYYY-MM-DD | PR-0N | top finding short description | severity`; clean reviews written as `No findings | NONE`; no credential values.
- Appended three lines to `review_history.md`.

**Agent:** Appended one line per review in the format from `AGENTS.md`. Corrected an earlier slip: PR #1 is the clean run, not PR #2.

**User:** We should be following the guardrails used in @AGENTS.md

**Tool event:** Edit to `review_history.md`. The PR-02 line had combined two findings, so it was narrowed to the single top finding.

**Agent:** The PR-02 line broke the "top finding" rule and is now fixed. PR-01 (`NONE`) and PR-03 (one HIGH, key value left out) already followed the rules. From the next `/review-pr` on, the history line is appended as soon as all three reviewers finish, with no extra line when the user answers the posting question.

Resulting history:

```
2026-11-13 | PR-01 | No findings | NONE
2026-11-13 | PR-02 | snake_case type name notification_manager | MEDIUM
2026-11-13 | PR-03 | Hardcoded API credential in payment client (ADR-002 Configuration) | HIGH
```

### 2. `/review-pr 4` run

**Fetch (GitHub MCP):**
- `get_pull_request(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=4)` → head `baa1692dbb1ee9e1c1cb91e2428aebbe38ec1a95`, base `main` `cb7a691656d4881e5d045a2308dbaf0d5de52c69`
- `get_pull_request_files(... pull_number=4)` → `internal/service/trip_service.go` (+24), `internal/service/trip_service_test.go` (+33)

**Delegation (sequential, each waited for):** each subagent received the same target, head SHA, repository root, full scope and complete diff.

1. `style-reviewer` → "No style findings."
2. `security-reviewer` → both files: "No security findings." (no credentials, SQL, shell, TLS, logging or deserialization; the test uses a local `httptest` server)
3. `architecture-reviewer` → read `reviewers/`, the architecture-guidelines skill and `docs/adr/` (ADR-001 through ADR-004). It reported one MEDIUM finding.

**ADR search and citation:**
- `ls docs/adr` → `ADR-001-layered-architecture.md`, `ADR-002-configuration.md`, `ADR-003-repository-pattern.md`, `ADR-004-dependency-injection.md`
- `grep -n -i "services must not" docs/adr/*` →
  `docs/adr/ADR-003-repository-pattern.md:10: ... Services must not import or call API clients or SQL directly, even when the client is injected. Injecting a client satisfies dependency injection but still bypasses the repository. Report this boundary violation as MEDIUM. Tests may wire adapters to httptest servers.`
- The architecture reviewer cited ADR-003 (Repository Pattern) for the finding. It noted that the constructor injection satisfies ADR-004 (Dependency Injection), and it found no violations of ADR-001 or ADR-002.
- The reviewer also reported that `fixture-policy.md` was not found. The main agent checked with `ls -l fixture-policy.md`, which shows the file exists at the repository root. That doesn't affect the result, because the diff has no credentials.

**History write (before the posting question):**

```
2026-11-13 | PR-04 | TripService calls api.Client directly, bypassing repository (ADR-003) | MEDIUM
```

**Draft shown to the user:**

> **Target:** nebius-academy-templates/go-pr-review-practice PR #4, "Add trip loading service" (head baa1692dbb1ee9e1c1cb91e2428aebbe38ec1a95), to be posted as a PR review comment, not an approval.
>
> **internal/service/trip_service.go**
> - [MEDIUM] L8, L12, L15, L19: the service calls the API client directly instead of going through a repository. This breaks ADR-003 (Repository Pattern): "Services must not import or call API clients or SQL directly, even when the client is injected." Injecting the client satisfies ADR-004 but still skips the repository.
>   Fix: declare `type RideRepository interface{ RecentRides(ctx context.Context) ([]model.Ride, error) }` near the service, have `TripService`/`NewTripService` take it, put the `api.Client` adapter in `internal/repository`, and connect it where the app is assembled.
>
> **internal/service/trip_service_test.go**
> No findings. After the fix, test the service with a fake repository and move the `httptest` test to the adapter in `internal/repository`.
>
> **Summary:** 0 HIGH, 1 MEDIUM, 0 LOW. Highest severity: MEDIUM.
>
> Reply post 4 to post this one, or send edits.

**Posting gate:** no yes/post has been received, so nothing was written to GitHub.
