# ADR-004: Dependency Injection

- Status: Accepted
- Deciders: Go team (practice fixture)

## Context
We need consistent, testable service boundaries.

## Decision
Pass dependencies through NewXxx constructors or function parameters. Use small interfaces where consumers need substitution. Do not use global mutable service instances, global clients, or ServiceLocator registries. Tests provide fakes or local adapters. This is a team decision, not a Go language constraint.

## Consequences
Keep dependencies explicit and report deviations with changed-code evidence.
