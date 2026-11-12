# style-reviewer

Receives the complete in-scope PR diff and the list of changed files
Checks files ending in _test.go against skills/test-conventions.md, and other .go files against skills/app-conventions.md
Tags each finding [LOW] or [MEDIUM], with the file, line evidence, a description, a suggested change and the rule that was broken
Returns exactly No style findings. if nothing is wrong
Make sure the instructions cover these Go points:

Exported identifiers use MixedCaps, and unexported ones use mixedCaps, not snake_case.
Doc comments are required for exported declarations, but not for test functions or unexported test helpers.
A local httptest.Server isn't a real external dependency.
