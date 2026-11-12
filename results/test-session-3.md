# Practice 3 — actual session evidence

Tool/model/version and date: Claude Code 2.1.322, model Claude Opus 5.5 (`claude-opus-5-5`), 2026-11-12
Target, mode and head SHA when live: `nebius-academy-templates/go-pr-review-practice`, `github` mode (per `review-target.json`)
- PR #2: head `6a01fd4b95c55923ff33993c7a48a940f9e9f8f8` (branch `pr-02`)
- PR #4: head `baa1692dbb1ee9e1c1cb91e2428aebbe38ec1a95` (taken from the file `blob_url`s in `get_pull_request_files`; `get_pull_request` was not called in this run)
- PR #1: head `2d551765be07e61e70f70245c9f7be2943f901f4` (branch `pr-01`)

## Recorded conversation and tool events

> Notes:
> - Tool names: the connected GitHub MCP server has no `pull_request_read` tool.
>   PR reads used `mcp__github__get_pull_request` and
>   `mcp__github__get_pull_request_files`, which return the same metadata and diff.
> - Skill loading: in runs 1 and 2, no convention skill files were read. Before
>   those runs, earlier in this session, `skills/app-conventions.md`,
>   `skills/test-conventions.md` and `skills/review-pr.md` had been read when they
>   were still `[PLACEHOLDER]`, so those reviews used general Go style knowledge.
>   Run 3 is the first run that read the filled-in `review-pr.md`,
>   `app-conventions.md` and `test-conventions.md`.
> - No credential values appeared in any output. No remote write was made in any
>   run.

---

### Run 1: PR #2, application files only, style only

**User:** Review only the application files in PR #2 from
https://github.com/nebius-academy-templates/go-pr-review-practice for style
issues.

**Skill files read in this run:** none.

**Tool events:**
- `mcp__github__get_pull_request_files(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=2)` → 1 file: `internal/notifications/manager.go` (added, +13/-0). No `_test.go` files.
- `mcp__github__get_pull_request(..., pull_number=2)` → title "Add notification scheduling manager", state `open`, head `pr-02` @ `6a01fd4b95c55923ff33993c7a48a940f9e9f8f8`, base `main` @ `cb7a691656d4881e5d045a2308dbaf0d5de52c69`

Returned patch for `internal/notifications/manager.go`:

```go
package notifications

type notification_manager struct {
	pending_notifications []string
}

func (m *notification_manager) Schedule_notification(Title string) {
	m.pending_notifications = append(m.pending_notifications, Title)
}

func (m *notification_manager) Get_pending() []string {
	return append([]string(nil), m.pending_notifications...)
}
```

**Agent:**

> Review of **PR #2, "Add notification scheduling manager"**, application files
> only, style only. The PR has one application file and no test files.
>
> **Target:** `nebius-academy-templates/go-pr-review-practice`, PR #2, head
> `6a01fd4`, posted as a review with event `COMMENT`
>
> ### `internal/notifications/manager.go`
>
> **MEDIUM** (lines 3–4, 7, 11): identifiers use snake_case. Go uses MixedCaps.
> - `notification_manager` should be `notificationManager`, or `Manager` if it's
>   meant to be exported.
> - `pending_notifications` should be `pendingNotifications`, or just `pending`.
> - `Schedule_notification` should be `ScheduleNotification`, or just `Schedule`.
> - `Get_pending` should be `Pending`.
>
> **MEDIUM** (lines 3, 7, 11): the exported methods sit on an unexported type,
> and there's no constructor, so the methods can't be called from outside the
> package. Either export the type or make the methods unexported.
>
> **LOW** (line 11): `Get_pending` uses a `Get` prefix. Getters leave it off, so
> this should be `Pending()`.
>
> **LOW** (line 7): the parameter `Title` is capitalized. It should be `title`.
>
> **LOW** (lines 1, 3, 7, 11): there are no doc comments, and no package comment.
>
> (Included a suggested corrected `Manager` / `Schedule` / `Pending` version.)
>
> ### Summary
> - Files reviewed: 1 (application)
> - Findings: 0 HIGH, 2 MEDIUM, 3 LOW
> - Highest severity: MEDIUM
>
> I haven't posted anything. Do you want me to post this review to PR #2? Reply
> **yes/post** to post it, or send edits.

**Posting outcome:** the user did not reply yes/post. Not posted.

---

### Run 2: PR #4, only `internal/service/trip_service_test.go`, style only

**User:** Review only internal/service/trip_service_test.go in PR #4 from
https://github.com/nebius-academy-templates/go-pr-review-practice for style
issues.

**Skill files read in this run:** none.

**Tool events:**
- `mcp__github__get_pull_request_files(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=4)` → 2 files: `internal/service/trip_service.go` (+24, out of scope and not reviewed) and `internal/service/trip_service_test.go` (+33, in scope). Blob URLs at `baa1692dbb1ee9e1c1cb91e2428aebbe38ec1a95`.
- `mcp__github__get_file_contents(owner=nebius-academy-templates, repo=go-pr-review-practice, path=go.mod, branch=pr-04)` → `module example.com/go-pr-review-practice` / `go 1.22`. This was read to decide whether `t.Context()` (Go 1.24+) applies.

Returned patch for `internal/service/trip_service_test.go`:

```go
package service

import (
	"context"
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"

	"example.com/go-pr-review-practice/internal/api"
)

func TestLoadTripsReturnsRecentTrips(t *testing.T) {
	server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.URL.Path != "/rides" {
			http.NotFound(w, r)
			return
		}
		w.Header().Set("Content-Type", "application/json")
		if _, err := fmt.Fprint(w, `[{"id":"ride-1"}]`); err != nil {
			t.Errorf("write response: %v", err)
		}
	}))
	t.Cleanup(server.Close)
	svc := NewTripService(api.NewClient(server.Client(), server.URL))
	rides, err := svc.LoadTrips(context.Background())
	if err != nil {
		t.Fatalf("LoadTrips() error = %v", err)
	}
	if len(rides) != 1 || rides[0].ID != "ride-1" {
		t.Fatalf("LoadTrips() = %v, want ride-1", rides)
	}
}
```

**Agent:**

> Review of **`internal/service/trip_service_test.go`** in **PR #4**, style only.
> I didn't review `internal/service/trip_service.go`.
>
> **Target:** `nebius-academy-templates/go-pr-review-practice`, PR #4, head
> `baa1692`, posted as a review with event `COMMENT`
>
> ### `internal/service/trip_service_test.go`
>
> **LOW** (lines 30–31): the failure message doesn't show what the test
> expected, and `%v` hides field names. Build the expected value and compare all
> of it with `reflect.DeepEqual(rides, want)` and `%+v`.
>
> **LOW** (lines 13–33): only the success path is tested. `LoadTrips` wraps
> client errors with `load trips: %w`, but no test checks that. Add a test with a
> handler that returns `http.StatusInternalServerError`.
>
> The rest follows standard practice: the test name says what it checks,
> `httptest.NewServer` keeps it isolated, `t.Cleanup(server.Close)` shuts it
> down, the handler uses `t.Errorf` rather than `t.Fatalf` (correct for the
> server goroutine), and `context.Background()` is right for Go 1.22.
>
> ### Summary
> - Files reviewed: 1 (test)
> - Findings: 0 HIGH, 0 MEDIUM, 2 LOW
> - Highest severity: LOW
>
> I haven't posted anything. Do you want me to post this review to PR #4? Reply
> **yes/post** to post it, or send edits.

**Posting outcome:** the user did not reply yes/post. Not posted.

---

### Run 3: `/review-pr 1`

**User:** `/review-pr 1`

**Skill files read in this run:**
- `.claude/skills/review-pr/SKILL.md`: loaded by Claude Code when the slash command ran. It says: "Read `skills/review-pr.md` relative to the repository root and follow its steps in order… Load `app-conventions` only for changed application `.go` files and `test-conventions` only for changed `_test.go` files."
- Tool event: `Bash: cat skills/review-pr.md skills/app-conventions.md skills/test-conventions.md`. This read all three files in one command. It returned the filled-in steps and rules shown below. The `git status` from the same command showed all three as locally modified.

`skills/review-pr.md` (Steps as read):

```
1. Read the case number from the request and look up the PR in review-target.json.
2. Fetch the PR with the GitHub MCP server (pull_request_read with get and get_diff).
3. Review only the files in the requested scope.
4. Apply app-conventions to .go files that don't end in _test.go, and test-conventions to files ending in _test.go. Load each skill only if the scope has that kind of file.
5. Check the security rules from AGENTS.md.
6. Compile the findings in your output format.
7. Show the complete review and the target, and ask for confirmation before posting.
8. Post only after a later, explicit yes or post. Never approve, merge or close a PR.
```

`skills/app-conventions.md` Rules: MixedCaps/mixedCaps naming with initialisms
(ID, URL, HTTP, API); doc comments on exported declarations; gofmt; explicit
error handling with `fmt.Errorf("…: %w", err)`.

`skills/test-conventions.md` Rules: descriptive `TestXxx(t *testing.T)` names;
check errors before using results and assert the observable result; isolation
(no real external services, shared state or order dependence; use fakes,
httptest or `t.TempDir`; clean up with `t.Cleanup`).

> Process note: step 4 asks for each convention file to be loaded only when the
> scope has that file type. Both file types were in scope for PR #1, so both were
> needed. They were read in one command together with `review-pr.md`, before the
> diff was fetched, not loaded one at a time after routing.

**Tool events:**
- Step 1: case `1` → `review-target.json` `pr_numbers["1"] = 1`, `mode: github`. This mapping was read earlier in the session and was unchanged.
- Step 2: `mcp__github__get_pull_request_files(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=1)` → `internal/service/ride_history_service.go` (+30) and `internal/service/ride_history_service_test.go` (+38). The patches were identical to those recorded in `results/test-session-2.md`.
- Step 2: `mcp__github__get_pull_request(..., pull_number=1)` → title "Add ride history service", state `open`, head `pr-01` @ `2d551765be07e61e70f70245c9f7be2943f901f4`, base `main` @ `cb7a691656d4881e5d045a2308dbaf0d5de52c69`
- Delegation: none. `skills/review-pr.md` does not call for subagents yet.
- Review passes: 1 (limit 2).

**Agent:**

> I ran `/review-pr 1` by following `skills/review-pr.md`. Step 2 names
> `pull_request_read`, which this GitHub MCP server doesn't have, so I used
> `get_pull_request` and `get_pull_request_files`.
>
> **Target:** `nebius-academy-templates/go-pr-review-practice`, PR #1, head
> `2d55176`, posted as a review with event `COMMENT`
>
> ### `internal/service/ride_history_service.go` (app-conventions)
> No findings.
> - **Naming:** MixedCaps throughout, and `ID` is used correctly as an initialism.
> - **Doc comments:** every exported declaration has one that starts with its
>   name. The code is gofmt-clean.
> - **Errors:** the repository error is checked and wrapped with
>   `fmt.Errorf("load rides: %w", err)`, and none are discarded.
>
> ### `internal/service/ride_history_service_test.go` (test-conventions)
> No findings.
> - **Names:** `TestLoadRidesReturnsRepositoryRides` and
>   `TestLoadRidesPreservesRepositoryError` say what behavior they check.
> - **Assertions:** `err` is checked before the result is used, the whole slice
>   is compared with `reflect.DeepEqual`, and `errors.Is` proves the wrapping keeps
>   the original error.
> - **Isolation:** the in-memory `fakeRideRepository` uses no external services or
>   shared state, and nothing needs cleanup.
>
> ### Security (AGENTS.md)
> No findings. There are no credentials, input handling, external calls, or
> logging of sensitive data.
>
> ### Summary
> - Files reviewed: 2 (1 application, 1 test)
> - Findings: 0 HIGH, 0 MEDIUM, 0 LOW
> - Highest severity: none
>
> I haven't posted anything. Do you want me to post this review to PR #1? Reply
> **yes/post** to post it, or send edits.

**Posting outcome:** the user did not reply yes/post. Not posted.
