# ADR-003: Repository Pattern

- Status: Accepted
- Deciders: Go team (practice fixture)

## Context
We need consistent, testable service boundaries.

## Decision
Services obtain data through small repository interfaces, normally declared near the consumer. Concrete adapters in internal/repository may depend on internal/api or database/sql. Services must not import or call API clients or SQL directly, even when the client is injected. Injecting a client satisfies dependency injection but still bypasses the repository. Report this boundary violation as MEDIUM. Tests may wire adapters to httptest servers.

## Consequences
Keep dependencies explicit and report deviations with changed-code evidence.
