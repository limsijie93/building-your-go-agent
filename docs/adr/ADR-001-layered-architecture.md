# ADR-001: Layered Architecture

- Status: Accepted
- Deciders: Go team (practice fixture)

## Context
We need consistent, testable service boundaries.

## Decision
Handlers and cmd/ entrypoints parse input and format output. Services contain business logic. Entrypoints do not query persistence directly. This separates transport from decisions and makes business logic independently testable.

## Consequences
Keep dependencies explicit and report deviations with changed-code evidence.
