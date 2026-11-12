# Observability table

Tool version: [PLACEHOLDER]
Plugin version: [PLACEHOLDER]
Model: [PLACEHOLDER]
Token method: [PLACEHOLDER] — briefly explain how totals count each model request once.
Expense metric: [PLACEHOLDER] — measured cost, or tokens if cost is unavailable.

| Input | Trace ID | Total tokens | Most expensive span | security-reviewer called? |
| --- | --- | --- | --- | --- |
| PR-01 | [PLACEHOLDER] | | | |
| PR-02 | [PLACEHOLDER] | | | |
| PR-03 | [PLACEHOLDER] | | | |
| PR-04 | [PLACEHOLDER] | | | |
| PR-05 | [PLACEHOLDER] | | | |

Identify the most expensive span by name or ID and its cost/token value. Record unavailable
metrics as `N/A: <reason>`. No separate model, parent ID or usage dump per reviewer is needed.

## PR-03 diagnostic
- Security reviewer called: [PLACEHOLDER]
- Received the correct PR-03 diff: [PLACEHOLDER]
- Returned finding: [PLACEHOLDER]
- Finding preserved in final output: [PLACEHOLDER]
- Supported diagnosis, including no failure when appropriate: [PLACEHOLDER]

## Saved span evidence
Paste only the short, redacted PR-03 excerpt needed to support the diagnosis: security
invocation and relevant input, returned finding, and corresponding consolidation output.
Identify the case and connect it to the table's trace. A trace link alone is insufficient.

[PLACEHOLDER]
