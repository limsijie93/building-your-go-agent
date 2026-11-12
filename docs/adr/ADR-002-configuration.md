# ADR-002: Configuration

- Status: Accepted
- Deciders: Go team (practice fixture)

## Context
We need consistent, testable service boundaries.

## Decision
Load runtime configuration and secrets centrally in internal/config or the composition root; inject them into constructors. Services do not call os.Getenv ad hoc or embed credentials. Ordinary algorithm constants are allowed. Hardcoded credential usage is HIGH security with this configuration reference; avoid duplicate findings.

## Consequences
Keep dependencies explicit and report deviations with changed-code evidence.
