# Evaluation

## Rubric
Write expectations before running the agent.

| Input | Expected finding | Expected severity | Expected reference |
| --- | --- | --- | --- |
| PR-01 — clean | [PLACEHOLDER] | | |
| PR-02 — Go naming | [PLACEHOLDER] | | |
| PR-03 — hardcoded credential | [PLACEHOLDER] | | |
| PR-04 — repository bypass | [PLACEHOLDER] | | |
| PR-05 — mixed | [PLACEHOLDER] | | |

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
