# Technology Stack and Platform Selection

## 1. Purpose

This document defines the proposed concrete technology stack for CollectionHub based on the architectural constraints and technical decisions established in the previous phase.

It converts the technology-independent architecture into an explicit implementation baseline.

The objective is not to select technologies because they are fashionable or familiar, but because they satisfy the architectural requirements already established.

The decisions in this document must therefore be evaluated against:

- domain independence,
- application isolation,
- infrastructure replaceability,
- persistence requirements,
- integration requirements,
- testability,
- maintainability,
- operational simplicity,
- security,
- deployment requirements.

This document establishes the **default technical baseline for implementation**.

---

# 2. Architectural Context

The selected technology stack must implement the following architectural structure:

```text
┌──────────────────────────────────────┐
│               DOMAIN                 │
│                                      │
│ Entities                             │
│ Value Objects                        │
│ Aggregates                           │
│ Domain Services                      │
│ Specifications                       │
│ Domain Events                        │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│             APPLICATION              │
│                                      │
│ Use Cases                            │
│ Application Services                 │
│ Ports / Contracts                    │
│ Commands / Queries                   │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│            INFRASTRUCTURE            │
│                                      │
│ Persistence                          │
│ External Integrations                │
│ Configuration                        │
│ Runtime                              │
│ Observability                        │
└───────────────────┬──────────────────┘
                    │
                    ▼
┌──────────────────────────────────────┐
│          EXTERNAL SYSTEMS            │
│                                      │
│ PostgreSQL                            │
│ External APIs                        │
│ Container Runtime                    │
└──────────────────────────────────────┘
```

The technology stack must preserve this direction.

---

# 3. Technology Selection Principles

The following principles govern the selected stack.

## 3.1 Mature Technologies

Core infrastructure should use mature, widely supported technologies.

The goal is to reduce:

- operational risk,
- dependency risk,
- knowledge concentration,
- migration cost.

---

## 3.2 Explicit Architecture

The selected technologies must allow CollectionHub to maintain explicit:

- domain boundaries,
- application boundaries,
- infrastructure boundaries,
- dependency relationships,
- persistence mappings.

---

## 3.3 Minimize Accidental Complexity

The initial implementation should avoid introducing infrastructure that is not justified by the domain or application requirements.

In particular, the initial stack should not require:

- microservices,
- distributed messaging,
- Kubernetes,
- distributed caching,
- event streaming,
- multi-region infrastructure,

unless future requirements establish a concrete need.

---

# 4. Selected Technology Stack

The proposed baseline stack is:

| Area | Technology | Status |
|---|---|---|
| Language | C# | Selected |
| Runtime | .NET 10 | Selected |
| Application framework | ASP.NET Core | Selected |
| API style | HTTP/REST | Selected |
| Database | PostgreSQL | Selected |
| Data access | Entity Framework Core | Selected |
| Database migrations | EF Core Migrations | Selected |
| Dependency injection | Microsoft.Extensions.DependencyInjection | Selected |
| Configuration | .NET Configuration / Options | Selected |
| Logging | Microsoft.Extensions.Logging | Selected |
| Testing | xUnit | Selected |
| Assertions | FluentAssertions | Selected |
| Mocking | Moq | Selected |
| Containers | Docker | Selected |
| Local orchestration | Docker Compose | Preferred |
| CI/CD | To be selected | Open |
| Secret management | Environment / platform secret provider | Open |
| Observability | OpenTelemetry-compatible stack | Preferred |
| External HTTP | HttpClient / IHttpClientFactory | Selected |

The stack is intentionally conservative.

It provides a strong ecosystem while allowing the domain and application architecture to remain framework-independent.

---

# 5. Programming Language — C#

## Decision

**C# is selected as the implementation language.**

C# is appropriate for CollectionHub because it provides strong support for:

- object-oriented domain modeling,
- encapsulation,
- immutable types,
- records and value semantics,
- interfaces,
- dependency inversion,
- generics,
- asynchronous programming,
- automated testing,
- mature tooling.

It also integrates naturally with the selected .NET runtime and ASP.NET Core platform.

---

# 6. Runtime — .NET 10

## Decision

**.NET 10 is selected as the application runtime.**

The runtime provides:

- modern C# support,
- long-term platform evolution,
- asynchronous execution,
- dependency injection,
- configuration,
- logging,
- HTTP infrastructure,
- testing ecosystem,
- container support.

The application should target a single major runtime version consistently across:

- development,
- CI,
- test,
- production.

Version drift between environments must be avoided.

---

# 7. Web Framework — ASP.NET Core

## Decision

**ASP.NET Core is selected as the application/HTTP framework.**

ASP.NET Core will provide the transport/runtime boundary.

Its responsibilities include:

- HTTP request handling,
- endpoint routing,
- middleware,
- dependency composition,
- authentication integration,
- authorization integration,
- serialization,
- HTTP error handling,
- health endpoints where required.

ASP.NET Core must remain outside the domain model.

---

# 8. API Style — HTTP/REST

## Decision

**HTTP/REST is selected as the initial API style.**

REST provides an appropriate boundary for CollectionHub application use cases.

The API layer should expose application capabilities rather than database operations.

Preferred conceptual flow:

```text
HTTP Request
     ↓
API Endpoint
     ↓
Application Command / Query
     ↓
Use Case
     ↓
Domain
     ↓
Infrastructure
     ↓
HTTP Response
```

The API layer must not directly manipulate repositories.

---

# 9. API Contract Strategy

API contracts should be represented through explicit transport DTOs.

The API should not expose domain entities directly.

Preferred:

```text
HTTP DTO
   ↓
Application Command
   ↓
Domain Model
```

and:

```text
Domain Result
   ↓
Application Result
   ↓
HTTP DTO
```

This protects the domain from API evolution.

---

# 10. Database — PostgreSQL

## Decision

**PostgreSQL is selected as the primary persistence technology.**

PostgreSQL is appropriate because CollectionHub requires strong support for:

- relational consistency,
- transactions,
- foreign keys,
- constraints,
- indexing,
- structured querying,
- schema evolution,
- mature operational tooling.

PostgreSQL becomes the initial authoritative persistence store.

---

# 11. Persistence Strategy — Entity Framework Core

## Decision

**Entity Framework Core is selected as the initial persistence technology.**

EF Core will be used only inside infrastructure.

The architecture must explicitly prevent:

```text
Domain
   ↓
Entity Framework Core
```

The correct relationship is:

```text
Domain
   ↑
Repository Contract
   ↑
Infrastructure Repository
   ↓
Entity Framework Core
   ↓
PostgreSQL
```

---

# 12. EF Core Usage Rules

EF Core must follow these architectural rules:

1. Domain entities must not require EF Core annotations unless explicitly justified.
2. Persistence configuration should be kept in infrastructure.
3. Database mappings should be explicit.
4. Lazy loading should not be introduced by default.
5. Database-specific concerns must remain inside infrastructure.
6. Queries should be encapsulated by repositories or dedicated persistence components.
7. EF Core entities must not automatically become API DTOs.
8. Persistence models should not dictate aggregate design.

---

# 13. Database Migrations

## Decision

**EF Core Migrations are selected for schema versioning.**

Migration files must be:

- committed to source control,
- reviewed as code,
- executed deterministically,
- associated with application versions,
- tested against representative environments.

Manual schema modification must not be part of the normal deployment process.

---

# 14. Transaction Strategy

Transactions will be aligned with application use-case consistency boundaries.

The initial technical implementation should use the transaction capabilities provided by PostgreSQL and EF Core.

Conceptually:

```text
Application Use Case
        │
        ▼
Transaction Boundary
        │
        ├── Repository Operation
        ├── Domain Operation
        ├── Repository Operation
        │
        ▼
Commit / Rollback
```

Transaction coordination must remain an infrastructure concern driven by application-level transactional intent.

---

# 15. Concurrency Strategy

The initial persistence implementation should prefer **optimistic concurrency** where aggregate consistency requires protection against concurrent updates.

A version/concurrency mechanism should be introduced for aggregates where:

- concurrent modification is possible,
- lost updates are unacceptable,
- aggregate state is mutable across multiple operations.

Pessimistic locking should only be introduced when concrete workload analysis demonstrates that optimistic concurrency is insufficient.

---

# 16. Dependency Injection

## Decision

**Microsoft's built-in dependency injection container is selected.**

The default composition mechanism will use constructor injection.

Preferred:

```text
public ApplicationService(
    IRepository repository,
    IExternalService externalService)
```

Dependencies should be explicit.

The service locator pattern should not be used.

The domain should not resolve dependencies from the container.

---

# 17. Composition Root

Dependency composition must occur at the application/infrastructure boundary.

Conceptually:

```text
Composition Root
      │
      ├── Domain Services
      ├── Application Services
      ├── Repository Implementations
      ├── External Adapters
      └── Technical Services
```

The composition root determines concrete implementations.

This allows application and domain components to remain unaware of infrastructure technologies.

---

# 18. Configuration

## Decision

The application will use the standard .NET configuration system combined with the Options pattern where appropriate.

Configuration sources may include:

- `appsettings.json`,
- environment-specific configuration,
- environment variables,
- container configuration,
- deployment platform configuration,
- secret providers.

Configuration should be validated at application startup.

Invalid required configuration should normally cause startup failure.

---

# 19. Secrets

Secrets will not be stored in source control.

Development may use local mechanisms appropriate to the environment.

Production secrets should be provided through the deployment platform or a dedicated secret-management service.

The application should consume secrets through configuration abstractions rather than directly coupling to a specific secret provider.

This preserves deployment portability.

---

# 20. HTTP Client Strategy

## Decision

**HttpClient with IHttpClientFactory is selected.**

External integrations should use named or typed HTTP clients.

The selected mechanism provides:

- connection management,
- centralized client configuration,
- timeout configuration,
- handler pipelines,
- observability hooks,
- testability.

External adapters should own the client-specific behavior.

---

# 21. External Integration Pattern

The standard integration structure will be:

```text
Application Port
       ↓
Integration Adapter
       ↓
Typed HTTP Client
       ↓
External API
```

The adapter is responsible for:

- request mapping,
- response mapping,
- authentication integration,
- provider-specific errors,
- timeout policy,
- retry policy where justified.

---

# 22. Retry Strategy

Retries will not be globally enabled for every external operation.

Each integration must determine whether an operation is:

- idempotent,
- safely retryable,
- transient-failure prone.

Retries should be applied selectively.

Potential retry conditions include:

- transient network failure,
- temporary provider unavailability,
- rate limiting where provider semantics permit retry.

Authentication failures and invalid requests should generally not be retried automatically.

---

# 23. Logging

## Decision

**Microsoft.Extensions.Logging is selected as the base logging abstraction.**

Application and infrastructure code should use structured logging.

The architecture should avoid coupling business logic to a specific logging backend.

The selected logging abstraction can later be connected to:

- console output,
- centralized log collection,
- cloud logging,
- OpenTelemetry-compatible pipelines.

---

# 24. Observability

## Decision

**OpenTelemetry-compatible observability is the preferred technical direction.**

The initial observability model should support:

- logs,
- metrics,
- traces.

Instrumentation should focus initially on:

- HTTP requests,
- database operations,
- external integrations,
- application operation boundaries.

Observability infrastructure should not become a dependency of the domain model.

---

# 25. Testing Framework

## Decision

**xUnit is selected as the primary testing framework.**

Tests should be organized according to architectural responsibility.

Example:

```text
Domain.Tests
Application.Tests
Infrastructure.Tests
Api.Tests
Integration.Tests
```

The exact project structure will be defined in the technical implementation blueprint.

---

# 26. Assertion Library

## Decision

**FluentAssertions is selected for test assertions.**

It improves readability of complex domain and application assertions.

Tests should describe behavior rather than implementation details.

---

# 27. Mocking Strategy

## Decision

**Moq is selected where mocking is appropriate.**

Mocking should not become the default testing strategy.

Preference should be given to:

1. real domain objects,
2. in-memory fakes where appropriate,
3. test-specific implementations,
4. mocks only where interaction behavior matters.

The goal is to avoid brittle tests coupled to implementation details.

---

# 28. Integration Testing

Infrastructure integration tests should use real technical components whenever practical.

For persistence, tests should preferably execute against PostgreSQL rather than attempting to emulate PostgreSQL behavior with a different database engine.

This is important because:

- SQL behavior differs between databases,
- constraints differ,
- transaction behavior differs,
- query semantics differ.

Containerized PostgreSQL is therefore a strong candidate for integration testing.

---

# 29. Containers — Docker

## Decision

**Docker is selected as the initial containerization technology.**

Containers will provide reproducible runtime environments.

The application container should contain:

- application binaries,
- required runtime components,
- runtime configuration mechanism.

It must not contain environment-specific secrets.

---

# 30. Local Infrastructure — Docker Compose

## Decision

**Docker Compose is the preferred local orchestration mechanism.**

It can provide local infrastructure such as:

```text
CollectionHub Application
PostgreSQL
Supporting services when required
```

Docker Compose should remain a development convenience and must not dictate the production deployment architecture.

---

# 31. Production Deployment

The exact production platform remains open.

However, the deployment target must support:

- container execution,
- managed PostgreSQL or equivalent,
- secure configuration,
- secret management,
- logging,
- health monitoring,
- automated deployment.

The architecture intentionally avoids selecting a specific cloud provider at this stage.

**Status:** Open.

---

# 32. CI/CD

The CI/CD platform remains open.

Regardless of platform, the pipeline must eventually support:

```text
Source
  ↓
Build
  ↓
Static Analysis
  ↓
Unit Tests
  ↓
Integration Tests
  ↓
Package / Container
  ↓
Security Checks
  ↓
Deployment
```

No deployment should bypass automated validation.

---

# 33. Security Baseline

The selected stack must enforce:

- HTTPS in deployed environments,
- secure secret handling,
- dependency vulnerability scanning,
- database credential protection,
- authentication where required,
- authorization at application boundaries,
- input validation,
- safe serialization,
- controlled error exposure.

Security mechanisms must remain separated from domain business rules.

---

# 34. Technology Dependency Governance

The project should avoid unnecessary third-party dependencies.

Before introducing a dependency, evaluate:

1. Does it solve a real requirement?
2. Does it respect the architecture?
3. Is it actively maintained?
4. Does it introduce unnecessary coupling?
5. Can it be replaced reasonably?
6. Does it increase operational complexity?
7. Does it introduce security or licensing concerns?

A smaller dependency surface is preferred.

---

# 35. Proposed Solution Structure

The selected stack should support a solution structure conceptually similar to:

```text
CollectionHub.sln
│
├── src
│   ├── CollectionHub.Domain
│   ├── CollectionHub.Application
│   ├── CollectionHub.Infrastructure
│   └── CollectionHub.Api
│
└── tests
    ├── CollectionHub.Domain.Tests
    ├── CollectionHub.Application.Tests
    ├── CollectionHub.Infrastructure.Tests
    ├── CollectionHub.Api.Tests
    └── CollectionHub.Integration.Tests
```

The exact project and namespace organization will be refined during the implementation blueprint phase.

---

# 36. Dependency Direction

The solution must enforce the following dependency graph:

```text
CollectionHub.Domain
        ↑
CollectionHub.Application
        ↑
CollectionHub.Infrastructure
        ↑
CollectionHub.Api
```

Testing projects may depend on the components they test.

The inverse dependency must not be introduced.

For example:

```text
Domain → Infrastructure
```

is prohibited.

---

# 37. Technology-to-Architecture Mapping

| Architecture Responsibility | Technology |
|---|---|
| Domain modeling | C# |
| Application services | C# / .NET |
| HTTP transport | ASP.NET Core |
| Dependency injection | Microsoft DI |
| Configuration | .NET Configuration / Options |
| Persistence | EF Core |
| Database | PostgreSQL |
| Migrations | EF Core Migrations |
| External HTTP | HttpClient / IHttpClientFactory |
| Logging | Microsoft.Extensions.Logging |
| Observability | OpenTelemetry-compatible |
| Unit testing | xUnit |
| Assertions | FluentAssertions |
| Mocking | Moq |
| Containerization | Docker |
| Local orchestration | Docker Compose |

This mapping must remain traceable to the architectural responsibilities defined in phase 2.3.

---

# 38. Selected Stack Rationale

The selected stack is intentionally coherent rather than maximally sophisticated.

The combination:

```text
C#
+
.NET 10
+
ASP.NET Core
+
PostgreSQL
+
EF Core
+
Docker
```

provides a mature foundation for implementing the previously defined architecture.

The principal advantage is ecosystem consistency.

A single platform provides:

- application runtime,
- HTTP stack,
- dependency injection,
- configuration,
- logging,
- testing ecosystem,
- container support.

PostgreSQL provides the persistence capabilities required by the current architectural model without introducing a distributed persistence architecture.

---

# 39. Technologies Explicitly Not Selected

The initial architecture does not require:

- MongoDB,
- Redis,
- Kafka,
- RabbitMQ,
- Kubernetes,
- microservices,
- serverless functions,
- distributed caches,
- event-streaming infrastructure.

Their absence is deliberate.

They may be introduced later if concrete requirements justify them.

---

# 40. Technical Trade-offs

## 40.1 EF Core vs Direct SQL

EF Core provides:

- productivity,
- mapping support,
- migrations,
- integration with .NET,
- transaction support.

Direct SQL may provide more control for specialized queries.

The initial choice is EF Core.

Direct SQL remains available inside infrastructure for justified performance or query-complexity cases.

---

## 40.2 Monolith vs Distributed Architecture

The initial deployment should be a modular monolith.

Reasons:

- simpler deployment,
- simpler transactions,
- simpler debugging,
- lower operational overhead,
- strong internal architectural boundaries.

The architecture must preserve the possibility of later extraction if a real scaling or organizational requirement appears.

---

## 40.3 Synchronous vs Asynchronous Processing

The initial architecture favors synchronous application workflows.

Asynchronous processing should only be introduced when:

- operation duration requires it,
- external integration reliability requires it,
- workload characteristics require it,
- user experience requires it.

---

# 41. Initial Technical Baseline

The following baseline is therefore established:

```text
Language:
    C#

Runtime:
    .NET 10

Web:
    ASP.NET Core

API:
    HTTP/REST

Database:
    PostgreSQL

Persistence:
    Entity Framework Core

Migrations:
    EF Core Migrations

Dependency Injection:
    Microsoft.Extensions.DependencyInjection

Configuration:
    .NET Configuration + Options

HTTP:
    HttpClient + IHttpClientFactory

Logging:
    Microsoft.Extensions.Logging

Observability:
    OpenTelemetry-compatible

Testing:
    xUnit + FluentAssertions + Moq

Container:
    Docker

Local Infrastructure:
    Docker Compose

Deployment:
    TBD
```

---

# 42. Open Technical Decisions

The following decisions remain open despite the baseline stack selection:

| ID | Decision | Status |
|---|---|---|
| TD-01 | Production hosting platform | Open |
| TD-02 | Secret management provider | Open |
| TD-03 | CI/CD provider | Open |
| TD-04 | Concrete OpenTelemetry backend | Open |
| TD-05 | External providers required by CollectionHub | Open |
| TD-06 | Authentication provider | Open |
| TD-07 | Authorization implementation | Open |
| TD-08 | Production PostgreSQL hosting | Open |
| TD-09 | Backup/recovery strategy | Open |
| TD-10 | Production scaling strategy | Open |

These decisions should be resolved when the corresponding infrastructure requirements are specified in greater detail.

---

# 43. Technology Stack Acceptance Criteria

The technology stack is considered accepted when:

- [x] The language is selected.
- [x] The runtime is selected.
- [x] The HTTP framework is selected.
- [x] The database is selected.
- [x] The persistence mechanism is selected.
- [x] The migration mechanism is selected.
- [x] The dependency injection mechanism is selected.
- [x] The configuration mechanism is selected.
- [x] The HTTP client strategy is selected.
- [x] The testing baseline is selected.
- [x] Containerization technology is selected.
- [ ] Production hosting is selected.
- [ ] CI/CD platform is selected.
- [ ] Secret-management provider is selected.
- [ ] Concrete observability backend is selected.

The remaining unchecked decisions do not block the definition of the core application architecture but must be resolved before production deployment.

---

# 44. Final Decision

CollectionHub will initially be implemented as a:

> **Modular monolithic .NET 10 application using C#, ASP.NET Core, PostgreSQL, Entity Framework Core, explicit application/domain boundaries, infrastructure adapters, Docker-based local infrastructure, and a test-first architectural validation strategy.**

The architecture deliberately avoids distributed infrastructure until justified by concrete requirements.

The selected technology stack is therefore:

```text
                 CollectionHub
                       │
              ┌────────┴────────┐
              │                 │
          ASP.NET Core       Application
              │                 │
              └────────┬────────┘
                       │
                    Domain
                       │
             Infrastructure Ports
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     EF Core       HTTP Clients    Runtime
        │              │              │
   PostgreSQL      External APIs   .NET 10
```

This stack provides the technical foundation required to proceed from architectural definition into concrete technical design.

**Technology stack baseline: ESTABLISHED.**

**Architecture style: MODULAR MONOLITH.**

**Primary runtime: .NET 10.**

**Primary persistence: PostgreSQL.**

**Primary data access: Entity Framework Core.**

**Next step:** define the concrete solution/project structure, dependency graph, package boundaries, and technical module organization that will implement this selected stack.