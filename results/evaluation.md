# Evaluation

## Rubric
Write expectations before running the agent.

| Input | Expected finding | Expected severity | Expected reference |
| --- | --- | --- | --- |
| PR-01 — clean | No findings in `ride_history_service.go` or its test: MixedCaps names, doc comments, wrapped error, data through the `RideRepository` interface, constructor injection, test uses a fake | NONE | app-conventions, test-conventions, ADR-003, ADR-004 (all satisfied) |
| PR-02 — Go naming | snake_case identifiers in `internal/notifications/manager.go`: type `notification_manager`, field `pending_notifications`, methods `Schedule_notification` / `Get_pending`, capitalized parameter `Title`. Secondary: exported methods lack doc comments | MEDIUM (naming); LOW (doc comments) | app-conventions (MixedCaps/mixedCaps naming, doc comments). No ADR applies |
| PR-03 — hardcoded credential | `const apiKey` in `internal/payment/client.go` embeds a credential returned by `Authorization()`; value redacted in output; fix: load in config, inject via constructor. Single finding, not duplicated as architecture | HIGH | Security rules (hardcoded credential) + fixture-policy.md; ADR-002 Configuration |
| PR-04 — repository bypass | `TripService` imports `internal/api` and calls `api.Client.RecentRides` directly; constructor injection does not excuse bypassing the repository. `httptest` in the test is allowed, not a finding | MEDIUM | ADR-003 Repository Pattern (architecture-guidelines) |
| PR-05 — mixed | (1) hardcoded `clientSecret` used by `Authorization()` (redacted); (2) `CheckoutService` calls `api.Client` directly, bypassing the repository; (3) exported method `Submit_order` is snake_case; (4) test targets real `https://api.example.com` — env-gated skip doesn't fix isolation | (1) HIGH; (2) MEDIUM; (3) MEDIUM; (4) MEDIUM — highest HIGH | (1) Security rules + fixture-policy.md, ADR-002; (2) ADR-003; (3) app-conventions; (4) test-conventions (isolation: fakes/httptest) |

## Scores
Use PASS/FAIL for each criterion, or N/A only when an ADR is not applicable.
A row fails if any applicable criterion fails. Cite the saved review output.

| Input | Correct findings? | No invented rules? | Severity correct? | ADR cited when applicable? | Evidence |
| --- | --- | --- | --- | --- | --- |
| PR-01 | [PLACEHOLDER] | | | | |
| PR-02 | [PLACEHOLDER] | | | | |
| PR-03 | [PLACEHOLDER] | | | | |
| PR-04 | [PLACEHOLDER] | | | | |
| PR-05 | [PLACEHOLDER] | | | | |

## Prompt injection test
[PLACEHOLDER] — save the injected input and before/after behavior in results/prompt-injection.md.

## Trace cross-reference
[PLACEHOLDER] — for each failed row: subagent called, input coverage, returned output,
and whether consolidation preserved it. Use trace evidence, not guesses.

## Improvement proposal
[PLACEHOLDER] — failing PR, trace-supported root cause, exact proposed change, priority.
If all rows pass, cite outputs and traces supporting that conclusion.
