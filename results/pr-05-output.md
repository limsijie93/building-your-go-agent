# PR-05 — final-configuration run

Tool/model/version, date, repository/PR/head and trace ID: Claude Code 2.1.323 (CLI), model claude-opus-5-5 (orchestrator and all three reviewer subagents), 2026-11-13, nebius-academy-templates/go-pr-review-practice PR #5 "Add checkout service", head feedb3b27ee87b4823dc1ae441ae58f05c33a36c, LangSmith trace 01a1d329-33b8-70e6-a004-3bb136ebf886 (spans: style 01a1d32c-b36d-7f65-82af-cd9978aa4068, security 01a1d32d-4797-7557-bc6b-620deaf3c831, architecture 01a1d32d-836e-7466-9775-0214f93ffe9a)

## Complete recorded review and declined publication

Reviewers run in order: style-reviewer → security-reviewer → architecture-reviewer, each given the same complete diff (2 files) fetched via GitHub MCP, with the credential literal masked before hand-off. The credential was reported by all three (style HIGH, security HIGH, architecture MEDIUM under ADR-002); merged into one HIGH. The architecture note on the live-service test is merged into the test-isolation finding.

### internal/service/checkout_service.go
- [HIGH] Lines 10, 19: hardcoded client secret. `const clientSecret = "<redacted>"` is a non-empty credential-like literal, and the exported `Authorization()` returns `"Bearer " + clientSecret`. Impact: the secret is committed to source control and can't be rotated without a code change. Fix: remove the literal, load the secret centrally in `internal/config` or the composition root, and inject it through the constructor (or move auth into the API adapter). Rule: security rules (hardcoded credential, reported HIGH per fixture-policy.md); violates ADR-002 Configuration.
- [MEDIUM] Lines 7, 13, 24: service bypasses the repository layer. `CheckoutService` imports `internal/api`, holds `client *api.Client`, and calls `s.client.RecentRides(ctx)` directly; injection does not excuse the bypass. Fix: take a small `RideRepository` interface and put the `api.Client` adapter in `internal/repository`. Rule: ADR-003 Repository Pattern.
- [MEDIUM] Lines 21–22: exported method `Submit_order` uses snake_case. Rename to `SubmitOrder`, update the doc comment and the test call site. Rule: app-conventions (MixedCaps for exported names).

### internal/service/checkout_service_test.go
- [MEDIUM] Lines 16–19: test depends on a real external service. It builds `api.NewClient` against `https://api.example.com` and asserts on live data; the `RUN_LIVE_TESTS` gate only skips it, it doesn't isolate it. Fix: test the service with a fake repository, or test the adapter against a local `httptest.Server` with `t.Cleanup`; keep the live test disabled (fixture-policy.md). Rule: test-conventions (isolation: no real external services).

### Summary
- Files reviewed: 2
- Findings: 1 HIGH, 3 MEDIUM, 0 LOW
- Highest severity: HIGH

## Publication
Target: nebius-academy-templates/go-pr-review-practice PR #5 (head feedb3b27ee87b4823dc1ae441ae58f05c33a36c). User confirmed posting ("Yes"). Post attempted via GitHub MCP add_issue_comment and failed with "Authentication Failed: Requires authentication"; nothing was written to GitHub.
