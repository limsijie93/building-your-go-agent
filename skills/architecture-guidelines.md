<!-- TODO: Summarize the four team ADRs in docs/adr/. State the severity for repository bypass. -->

## Rules

Layered architecture: handlers and cmd/ entry points parse input and format output; services contain business logic; entry points don't query storage directly.
Configuration: runtime configuration and secrets are loaded centrally (internal/config or the composition root) and passed through constructors. Services don't call os.Getenv or contain credentials.
Repository pattern: services get data through small repository interfaces. Only adapters in internal/repository may use internal/api or database/sql.
Dependency injection: dependencies come through NewXxx constructors or function parameters, with no global services, global clients or service locators.
Also state that:

Injecting a concrete API client follows dependency injection but still bypasses the repository layer.
Tests may connect adapters to a local httptest.Server.
A hardcoded credential is mainly a security finding, so it isn't reported again as a separate architecture finding.
These are this team's decisions, not general Go rules.
