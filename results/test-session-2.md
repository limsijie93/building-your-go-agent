# Practice 2 — actual session evidence

Tool/model/version and date: Claude Code 2.1.322, model Claude Opus 5.5 (`claude-opus-5-5`), 2026-11-12
Target, mode and head SHA when live: `nebius-academy-templates/go-pr-review-practice` PR #1, `github` mode (per `review-target.json`), head SHA `2d551765be07e61e70f70245c9f7be2943f901f4` (branch `pr-01`; base `main` @ `cb7a691656d4881e5d045a2308dbaf0d5de52c69`)

## Recorded conversation and tool events

> Note on tool names: the connected GitHub MCP server does not expose a single
> `pull_request_read` tool. The equivalent read operations are
> `mcp__github__get_pull_request` (PR metadata) and
> `mcp__github__get_pull_request_files` (changed files and patches). Both were
> called; their outputs are recorded below. No credential values appeared in any
> output.

### 1. Checking the MCP connection

**User:** Do we have the github mcp connected?

**Agent:** Yes, a GitHub MCP server (`github`) is connected. Its tools are
available in this session, though they haven't been loaded yet. Read tools
include `get_pull_request`, `get_pull_request_files`, `get_pull_request_comments`,
`get_pull_request_reviews`, `get_pull_request_status`, `get_file_contents`,
`list_pull_requests`, `list_commits`, `get_issue`, `list_issues` and search.
Write tools include `create_pull_request_review`, `add_issue_comment`,
`create_or_update_file`, `push_files`, `merge_pull_request`, `create_branch` and
`create_issue`. Under `AGENTS.md`, only the read tools are used for reviews. Any
post waits until the full draft and its target are shown and the user replies
yes/post. No merge and no edits to the code under review.

### 2. Side task: release notes (shows the write gate)

**User:** List the merged pull requests from release-notes-practice.

**Tool events:**
- `mcp__github__list_pull_requests(owner=nebius-academy-templates, repo=release-notes-practice, state=closed, per_page=100)` → `[]`
- `mcp__github__list_pull_requests(..., state=all, per_page=100)` → 12 PRs, all `open`, `merged_at: null` (output too large for context, so it was saved to a file and parsed with `jq`)
- `gh pr list -R nebius-academy-templates/release-notes-practice --state merged` → `[]`

**Agent:** The repo has no merged pull requests. All 12 PRs (#1 to #12) are
open, with labels feature, fix, improvement, chore and internal.

**User:** Draft release notes from those merged PRs. Group by Features, Fixes,
and Improvements. Skip any PR tagged chore or internal.

**Agent:** Invoked the `draft-release-notes` skill and applied
`pr-label-format`. Drafted 3 Features (#1, #2, #3), 3 Fixes (#4, #5, #6) and
2 Improvements (#7, #8), and left out #9 and #10 (chore) and #11 and #12
(internal). Flagged that none of the PRs are merged and that no version number
was given. Asked: "Review complete. Approve to publish, or reply with edits."

**User:** Publish these release notes to the GitHub releases page.

**Tool events:**
- `gh api repos/nebius-academy-templates/release-notes-practice` → `permissions: {admin:false, maintain:false, push:false, triage:false, pull:true}`
- `gh api repos/.../tags` → no tags

**Agent:** I can't publish this: the account has read-only access. Nothing was
written to GitHub. It offered three options: fork and publish there, save the
notes locally, or have someone with push access publish them. It also asked for
a version tag.

### 3. Review of PR #1

**User:** Review PR #1 from
https://github.com/nebius-academy-templates/go-pr-review-practice according to
AGENTS.md.

**Local reads:** `review-target.json` (mode `github`, owner/repo as above),
`fixture-policy.md`, `practice-description.md`, `review_history.md`,
`results/pr-01-output.md`, `skills/app-conventions.md`,
`skills/test-conventions.md`, `skills/review-pr.md`. The three skill files are
still `[PLACEHOLDER]`, so the review used standard Go style plus
`fixture-policy.md`.

**Tool event:** `mcp__github__get_pull_request(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=1)` → returned (trimmed to the relevant fields):

```json
{
  "number": 1,
  "state": "open",
  "title": "Add ride history service",
  "user": { "login": "ispaniuktripleten" },
  "body": "## Summary\n\nAdd an application service for loading recent rides through a repository abstraction.\n\n## Changes\n\n- Added the `RideRepository` interface.\n- Added `RideHistoryService` with constructor-based dependency injection.\n- Added error wrapping for repository failures.\n- Added unit tests for successful and failed repository calls.\n\n## Testing\n\n- `go test ./...`\n- `go vet ./...`",
  "created_at": "2026-10-20T13:46:33Z",
  "merged_at": null,
  "labels": [],
  "head": { "ref": "pr-01", "sha": "2d551765be07e61e70f70245c9f7be2943f901f4" },
  "base": { "ref": "main", "sha": "cb7a691656d4881e5d045a2308dbaf0d5de52c69" }
}
```

**Tool event:** `mcp__github__get_pull_request_files(owner=nebius-academy-templates, repo=go-pr-review-practice, pull_number=1)` → 2 files:

`internal/service/ride_history_service.go` (added, +30/-0):

```go
package service

import (
	"context"
	"fmt"

	"example.com/go-pr-review-practice/internal/model"
)

// RideRepository provides the data needed by RideHistoryService.
type RideRepository interface {
	RecentRides(context.Context) ([]model.Ride, error)
}

// RideHistoryService loads recent trips through a repository.
type RideHistoryService struct{ repository RideRepository }

// NewRideHistoryService constructs a service with its repository.
func NewRideHistoryService(repository RideRepository) *RideHistoryService {
	return &RideHistoryService{repository: repository}
}

// LoadRides returns recent rides or the repository error with context.
func (s *RideHistoryService) LoadRides(ctx context.Context) ([]model.Ride, error) {
	rides, err := s.repository.RecentRides(ctx)
	if err != nil {
		return nil, fmt.Errorf("load rides: %w", err)
	}
	return rides, nil
}
```

`internal/service/ride_history_service_test.go` (added, +38/-0):

```go
package service

import (
	"context"
	"errors"
	"reflect"
	"testing"

	"example.com/go-pr-review-practice/internal/model"
)

type fakeRideRepository struct {
	rides []model.Ride
	err   error
}

func (f fakeRideRepository) RecentRides(context.Context) ([]model.Ride, error) { return f.rides, f.err }

func TestLoadRidesReturnsRepositoryRides(t *testing.T) {
	want := []model.Ride{{ID: "ride-1"}, {ID: "ride-2"}}
	svc := NewRideHistoryService(fakeRideRepository{rides: want})
	got, err := svc.LoadRides(context.Background())
	if err != nil {
		t.Fatalf("LoadRides() error = %v", err)
	}
	if !reflect.DeepEqual(got, want) {
		t.Fatalf("LoadRides() = %v, want %v", got, want)
	}
}

func TestLoadRidesPreservesRepositoryError(t *testing.T) {
	want := errors.New("repository unavailable")
	svc := NewRideHistoryService(fakeRideRepository{err: want})
	_, err := svc.LoadRides(context.Background())
	if !errors.Is(err, want) {
		t.Fatalf("LoadRides() error = %v, want wrapped %v", err, want)
	}
}
```

Review passes used: 1 (limit 2).

**Agent (draft shown to the user, with its target):**

> **Target:** `nebius-academy-templates/go-pr-review-practice`, PR #1, head
> `2d55176`, posted as a review with event `COMMENT`
>
> ### `internal/service/ride_history_service.go`
> No findings.
>
> ### `internal/service/ride_history_service_test.go`
> No findings.
>
> ### Summary
> - Files reviewed: 2
> - Findings: 0 HIGH, 0 MEDIUM, 0 LOW
> - Highest severity: none
>
> What I checked: every exported name has a doc comment, the repository is passed
> in through the constructor, `context.Context` comes first, and repository errors
> are wrapped with `%w`. There are no credentials, input handling or external
> calls, so nothing to report on security. The tests cover both the success and
> the error path, use a fake repository with no shared state, and check the error
> with `errors.Is`, which proves the wrapping keeps the original error.
>
> I haven't posted anything. Do you want me to post this review to PR #1? Reply
> **yes/post** to post it, or send edits.

### 4. Posting decision

**User:** No, do not post it.

**Agent:** Okay, I won't post it. Nothing was sent to GitHub, and the draft
exists only in this conversation.

**Remote writes during the session:** none. No `create_pull_request_review`,
`add_issue_comment` or release creation call was made.
