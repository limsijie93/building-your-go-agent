<!-- TODO: Use the *_test.go trigger, testing.T naming, result/error assertions, helpers/cleanup, and isolated dependencies. Include good and bad Go test snippets. -->

## Rules

Descriptive TestXxx(t *testing.T) names that say what behavior is tested
Checking errors before using results, and asserting the observable result
Isolation: no real external services, shared state or test order dependencies; use fakes, httptest or t.TempDir, and clean up with t.Cleanup

## Template

One compliant and one non-compliant Go test.
