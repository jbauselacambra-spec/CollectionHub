# Technical Solution Structure and Project Dependencies

## 1. Purpose

This document defines the concrete .NET solution structure that will implement the architectural and technology decisions established in the previous phases.

It translates:

```text
Architectural Model
        ↓
Technology Stack
        ↓
.NET Solution
        ↓
Projects / Modules
        ↓
Project Dependencies
```

The objective is to ensure that the physical structure of the solution reinforces the architectural boundaries instead of merely documenting them.

This document therefore defines:

- solution organization,
- project responsibilities,
- project dependency direction,
- allowed references,
- forbidden references,
- package ownership,
- test project structure,
- composition boundaries,
- technical module organization,
- dependency validation rules.

No source implementation is introduced by this document.

---

# 2. Architectural Foundation

The solution is based on the modular monolith architecture selected in:

`37_TECHNOLOGY_STACK_AND_PLATFORM_SELECTION.md`

The fundamental dependency direction is:

```text
┌───────────────────────────────┐
│            DOMAIN             │
│                               │
│ CollectionHub.Domain          │
└───────────────┬───────────────┘
                ▲
                │
┌───────────────┴───────────────┐
│          APPLICATION          │
│                               │
│ CollectionHub.Application    │
└───────────────┬───────────────┘
                ▲
                │
┌───────────────┴───────────────┐
│        INFRASTRUCTURE         │
│                               │
│ CollectionHub.Infrastructure │
└───────────────┬───────────────┘
                ▲
                │
┌───────────────┴───────────────┐
│              API              │
│                               │
│ CollectionHub.Api             │
└───────────────────────────────┘
```

The arrows represent compile-time dependency direction.

The critical rule is:

> Outer projects may depend on inner projects, but inner projects must never depend on outer projects.

---

# 3. Proposed Solution Structure

The initial solution structure is:

```text
CollectionHub/
│
├── CollectionHub.sln
│
├── src/
│   │
│   ├── CollectionHub.Domain/
│   │
│   ├── CollectionHub.Application/
│   │
│   ├── CollectionHub.Infrastructure/
│   │
│   └── CollectionHub.Api/
│
├── tests/
│   │
│   ├── CollectionHub.Domain.Tests/
│   │
│   ├── CollectionHub.Application.Tests/
│   │
│   ├── CollectionHub.Infrastructure.Tests/
│   │
│   ├── CollectionHub.Api.Tests/
│   │
│   └── CollectionHub.Integration.Tests/
│
├── docs/
│   └── 02_ARCHITECTURE/
│
└── infrastructure/
    └── local/
```

The exact contents of each directory will be defined by subsequent technical design documents.

---

# 4. Solution Root

The solution root is responsible for organizing the entire CollectionHub codebase.

Expected root structure:

```text
CollectionHub/
├── CollectionHub.sln
├── src/
├── tests/
├── docs/
├── infrastructure/
└── README.md
```

The solution root must not contain application implementation code.

---

# 5. Source Projects

The production solution contains four primary projects:

```text
CollectionHub.Domain
CollectionHub.Application
CollectionHub.Infrastructure
CollectionHub.Api
```

Each project corresponds to a distinct architectural responsibility.

---

# 6. CollectionHub.Domain

## Responsibility

`CollectionHub.Domain` contains the business model.

It is the innermost production project.

It owns:

- entities,
- value objects,
- aggregates,
- domain services,
- domain specifications,
- domain events,
- domain exceptions,
- domain policies,
- domain-facing contracts where justified.

The project must contain no infrastructure implementation.

---

# 7. Domain Project Dependencies

`CollectionHub.Domain` must have:

```text
Project References:
    None
```

The domain project should ideally depend only on the base class library and carefully selected technical primitives that do not compromise domain independence.

The following dependencies are prohibited:

```text
CollectionHub.Domain
    → CollectionHub.Application
    → CollectionHub.Infrastructure
    → CollectionHub.Api
```

The domain must remain independently compilable.

---

# 8. Domain Package Policy

The domain project should have the smallest dependency surface of all production projects.

Third-party packages should only be introduced when they provide clear domain value.

The following categories should normally be prohibited:

- ORM packages,
- HTTP clients,
- ASP.NET Core packages,
- database providers,
- configuration providers,
- logging infrastructure,
- cloud SDKs.

The goal is to ensure that the domain remains technically portable.

---

# 9. CollectionHub.Application

## Responsibility

`CollectionHub.Application` contains application orchestration.

It owns:

- use cases,
- application services,
- commands,
- queries,
- application DTOs,
- application ports,
- application-level result contracts,
- transaction-facing abstractions where required,
- orchestration logic.

The application project may reference the domain.

---

# 10. Application Project Dependencies

The allowed dependency is:

```text
CollectionHub.Application
        ↓
CollectionHub.Domain
```

The application project must not reference:

```text
CollectionHub.Infrastructure
CollectionHub.Api
```

This prevents application workflows from becoming coupled to implementation details.

---

# 11. Application Package Policy

Application packages should remain lightweight.

The application layer should generally avoid direct dependencies on:

- Entity Framework Core,
- PostgreSQL provider packages,
- ASP.NET Core,
- external SDKs,
- concrete HTTP clients,
- infrastructure configuration providers.

Application interfaces may describe required capabilities without importing their implementation technologies.

---

# 12. CollectionHub.Infrastructure

## Responsibility

`CollectionHub.Infrastructure` implements technical capabilities required by the application.

It owns:

- repository implementations,
- persistence models,
- EF Core configuration,
- database access,
- external integration adapters,
- HTTP client configuration,
- configuration infrastructure,
- technical services,
- observability infrastructure,
- infrastructure-specific error translation.

---

# 13. Infrastructure Project Dependencies

Infrastructure may reference:

```text
CollectionHub.Infrastructure
        ↓
CollectionHub.Application
        ↓
CollectionHub.Domain
```

Therefore:

```text
CollectionHub.Infrastructure
    → CollectionHub.Application
    → CollectionHub.Domain
```

This is the primary implementation of dependency inversion.

Infrastructure implements contracts defined by inner layers.

---

# 14. Infrastructure Package Ownership

Infrastructure is the only production project that should normally contain dependencies such as:

- Entity Framework Core,
- PostgreSQL provider,
- database migration tooling,
- external SDKs,
- HTTP resilience libraries,
- OpenTelemetry infrastructure packages,
- infrastructure-specific configuration providers.

These packages must not leak into inner projects.

---

# 15. CollectionHub.Api

## Responsibility

`CollectionHub.Api` is the transport and application composition boundary.

It owns:

- ASP.NET Core startup,
- HTTP endpoints,
- middleware,
- transport DTOs,
- request/response mapping,
- API-specific validation,
- authentication/authorization integration,
- dependency composition,
- application startup.

It must not own business rules.

---

# 16. API Project Dependencies

The API project may reference:

```text
CollectionHub.Api
      ↓
CollectionHub.Application
CollectionHub.Infrastructure
```

The API indirectly accesses the domain through application contracts.

Direct domain references should be minimized.

The intended dependency structure is:

```text
API
├── Application
└── Infrastructure
       └── Application
              └── Domain
```

---

# 17. Why the API References Infrastructure

The API is the composition root of the initial modular monolith.

It must be able to configure concrete infrastructure implementations:

```text
ICollectionRepository
        ↓
EfCollectionRepository
```

and:

```text
IExternalProvider
        ↓
ExternalProviderAdapter
```

The API therefore needs access to infrastructure registration mechanisms.

This does not mean API endpoint code should instantiate infrastructure objects.

Composition should remain centralized.

---

# 18. Production Dependency Graph

The complete production dependency graph is:

```text
                         ┌─────────────────────┐
                         │  CollectionHub.Api  │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
              ┌──────────────────┐  ┌──────────────────────┐
              │   Application    │  │   Infrastructure     │
              └────────┬─────────┘  └───────────┬──────────┘
                       │                        │
                       ▼                        ▼
              ┌──────────────────┐     ┌──────────────────┐
              │      Domain      │◄────│   Application    │
              └──────────────────┘     └──────────────────┘
```

Simplified:

```text
Api
 ├── Application
 └── Infrastructure
       └── Application
             └── Domain
```

---

# 19. Forbidden Production Dependencies

The following references are explicitly prohibited:

```text
Domain → Application
Domain → Infrastructure
Domain → Api

Application → Infrastructure
Application → Api

Infrastructure → Api
```

The most important forbidden dependency is:

```text
Application → Infrastructure
```

because it would invert dependency direction and make infrastructure implementation details part of application logic.

---

# 20. Dependency Matrix

| From | Domain | Application | Infrastructure | API |
|---|---:|---:|---:|---:|
| Domain | — | ❌ | ❌ | ❌ |
| Application | ✅ | — | ❌ | ❌ |
| Infrastructure | ✅ | ✅ | — | ❌ |
| API | Prefer no direct dependency | ✅ | ✅ | — |

The API may have a limited direct dependency on Domain only where transport mapping requires a domain type and where such dependency does not compromise the boundary.

The preferred strategy is to keep API contracts independent of domain entities.

---

# 21. Test Projects

Each production project receives an associated test project where useful.

```text
tests/
│
├── CollectionHub.Domain.Tests/
├── CollectionHub.Application.Tests/
├── CollectionHub.Infrastructure.Tests/
├── CollectionHub.Api.Tests/
└── CollectionHub.Integration.Tests/
```

Testing projects are not part of the production dependency graph.

---

# 22. Domain Tests

`CollectionHub.Domain.Tests` tests:

- entities,
- value objects,
- aggregates,
- domain services,
- specifications,
- invariants,
- domain events,
- domain exceptions.

These tests must not require:

- PostgreSQL,
- ASP.NET Core,
- Docker,
- external APIs.

The domain test suite should be fast and deterministic.

---

# 23. Application Tests

`CollectionHub.Application.Tests` tests:

- use cases,
- application services,
- orchestration,
- application validation,
- application error handling,
- interaction with application ports.

Infrastructure should normally be replaced with test doubles.

The purpose is to validate application behavior independently of concrete infrastructure.

---

# 24. Infrastructure Tests

`CollectionHub.Infrastructure.Tests` tests:

- repository implementations,
- persistence mappings,
- external adapters,
- configuration infrastructure,
- infrastructure error translation,
- technical services.

Where database behavior is important, real PostgreSQL should be preferred over a fake database.

---

# 25. API Tests

`CollectionHub.Api.Tests` tests:

- endpoint behavior,
- HTTP contracts,
- request validation,
- response mapping,
- middleware,
- authentication/authorization integration where appropriate.

These tests should not duplicate extensive domain tests.

---

# 26. Integration Tests

`CollectionHub.Integration.Tests` validates the system across architectural boundaries.

Potential scenarios include:

```text
API
 ↓
Application
 ↓
Infrastructure
 ↓
PostgreSQL
```

and, where appropriate:

```text
Application
 ↓
Integration Adapter
 ↓
External Test Provider
```

Integration tests are intended to detect wiring and boundary failures that unit tests cannot detect.

---

# 27. Test Dependency Rules

Test projects may depend on production projects according to their test target.

Example:

```text
Domain.Tests
    → Domain

Application.Tests
    → Application
    → Domain

Infrastructure.Tests
    → Infrastructure
    → Application
    → Domain

Api.Tests
    → Api
    → Application
    → Infrastructure

Integration.Tests
    → Api
    → Application
    → Infrastructure
    → Domain
```

Tests must never introduce forbidden dependencies into production projects.

---

# 28. Infrastructure Internal Modules

Although Infrastructure is one .NET project initially, it should be internally organized by technical responsibility.

Conceptually:

```text
CollectionHub.Infrastructure/
│
├── Persistence/
├── Integrations/
├── Configuration/
├── Runtime/
├── Observability/
└── Common/
```

These are logical modules inside the infrastructure project.

They are not necessarily separate assemblies.

---

# 29. Persistence Module

The persistence module should contain:

```text
Persistence/
├── DbContext/
├── Configurations/
├── Models/
├── Mappings/
├── Repositories/
└── Migrations/
```

Responsibilities:

- EF Core configuration,
- persistence entities/models,
- mappings,
- repository implementations,
- migrations,
- database-specific infrastructure.

---

# 30. Integration Module

The integration module should contain:

```text
Integrations/
├── ProviderA/
├── ProviderB/
└── Common/
```

The actual provider names will be defined once external integrations are finalized.

Each provider should remain internally isolated.

Example:

```text
Integrations/
└── ExternalProvider/
    ├── Client/
    ├── Models/
    ├── Mapping/
    ├── Adapter/
    └── Configuration/
```

---

# 31. Configuration Module

The configuration module should contain:

```text
Configuration/
├── Options/
├── Validation/
└── Extensions/
```

It owns:

- configuration binding,
- validation,
- infrastructure registration helpers,
- configuration-specific adapters.

Application configuration contracts may be defined in Application when the application actually requires them.

---

# 32. Runtime Module

The runtime module contains technical startup and composition helpers.

Potential responsibilities include:

- dependency registration,
- infrastructure service registration,
- application service registration extensions,
- health checks,
- runtime initialization.

The runtime module must not become a second application layer.

---

# 33. Observability Module

The observability module may contain:

```text
Observability/
├── Logging/
├── Metrics/
├── Tracing/
└── Extensions/
```

It owns technical instrumentation.

Business decisions must not be implemented here.

---

# 34. Application Internal Structure

The Application project should be organized around use cases and application capabilities.

A conceptual structure is:

```text
CollectionHub.Application/
│
├── Abstractions/
├── UseCases/
├── Commands/
├── Queries/
├── DTOs/
├── Services/
├── Ports/
└── Common/
```

The exact organization should follow the application use-case map defined previously.

---

# 35. Domain Internal Structure

The Domain project should follow domain concepts rather than technical infrastructure concerns.

Conceptually:

```text
CollectionHub.Domain/
│
├── Aggregates/
├── Entities/
├── ValueObjects/
├── Services/
├── Specifications/
├── Events/
├── Exceptions/
└── Common/
```

Where appropriate, aggregates should be the primary organizational unit.

---

# 36. API Internal Structure

The API project should separate transport concerns from composition.

Conceptually:

```text
CollectionHub.Api/
│
├── Endpoints/
├── Contracts/
│   ├── Requests/
│   └── Responses/
├── Middleware/
├── Authentication/
├── Authorization/
├── Configuration/
└── Program.cs
```

The API should not contain persistence logic.

---

# 37. Package Boundary Rules

Package ownership should follow project responsibility.

### Domain

Allowed:

- minimal base runtime dependencies,
- explicitly justified domain libraries.

Forbidden:

- EF Core,
- ASP.NET Core,
- PostgreSQL provider,
- HTTP clients,
- external SDKs.

### Application

Allowed:

- domain reference,
- application-oriented libraries.

Forbidden:

- concrete infrastructure packages,
- EF Core,
- database providers,
- external provider SDKs.

### Infrastructure

Allowed:

- EF Core,
- PostgreSQL provider,
- HTTP infrastructure,
- external SDKs,
- observability packages,
- configuration providers.

### API

Allowed:

- ASP.NET Core,
- application reference,
- infrastructure registration reference,
- API-specific tooling.

---

# 38. Composition Boundary

The composition root should be concentrated in the API/runtime boundary.

Conceptually:

```text
Program
  │
  ├── Load Configuration
  ├── Register Domain Services
  ├── Register Application Services
  ├── Register Infrastructure
  ├── Configure HTTP
  ├── Configure Observability
  └── Start Runtime
```

No domain or application component should perform global service registration.

---

# 39. Assembly-Level Architecture

The four production assemblies represent the primary architectural boundaries:

```text
CollectionHub.Domain.dll
CollectionHub.Application.dll
CollectionHub.Infrastructure.dll
CollectionHub.Api.dll
```

This physical separation provides compile-time enforcement of the conceptual architecture.

---

# 40. Architecture Enforcement

The solution should eventually include automated architecture checks.

Possible mechanisms include:

- dependency validation,
- architecture tests,
- restricted package references,
- static analysis,
- build-time validation.

The goal is to detect violations such as:

```text
Domain references Infrastructure
Application references Infrastructure
Infrastructure references API
```

before they reach code review or production.

---

# 41. Namespace Strategy

Namespaces should follow the project and logical module structure.

Examples:

```text
CollectionHub.Domain.Collections
CollectionHub.Domain.Collections.Entities

CollectionHub.Application.Collections
CollectionHub.Application.Collections.UseCases

CollectionHub.Infrastructure.Persistence
CollectionHub.Infrastructure.Persistence.Repositories

CollectionHub.Infrastructure.Integrations

CollectionHub.Api.Endpoints
CollectionHub.Api.Contracts
```

Namespaces should communicate architectural ownership.

---

# 42. Shared/Common Project Decision

A generic `CollectionHub.Common` production project should **not** be created by default.

A shared project tends to become a dependency dumping ground.

Shared functionality should remain in the owning architectural layer unless it represents a clearly defined cross-cutting concern.

If a shared project becomes necessary later, its responsibility must be explicitly documented.

---

# 43. Domain Common Types

Reusable domain primitives should remain inside the Domain project.

Examples may include:

- domain identifiers,
- domain result abstractions,
- common value object infrastructure,
- domain event abstractions.

These should not be extracted prematurely.

---

# 44. Application Common Types

Application-wide abstractions should remain inside Application.

Examples include:

- application result types,
- pagination contracts,
- application exceptions,
- command/query abstractions,
- transaction-facing abstractions.

They should only be created when multiple use cases genuinely require them.

---

# 45. Infrastructure Common Types

Infrastructure-specific shared components may exist under:

```text
CollectionHub.Infrastructure.Common
```

but only for genuinely technical cross-cutting concerns.

Examples:

- persistence utilities,
- HTTP infrastructure helpers,
- integration error translation,
- technical retry helpers.

Business concepts must not be placed here.

---

# 46. Project Reference Rules

The following rules are mandatory.

### Rule PR-01

`CollectionHub.Domain` references no CollectionHub production project.

### Rule PR-02

`CollectionHub.Application` references only `CollectionHub.Domain`.

### Rule PR-03

`CollectionHub.Infrastructure` may reference `CollectionHub.Application` and `CollectionHub.Domain`.

### Rule PR-04

`CollectionHub.Api` may reference `CollectionHub.Application` and `CollectionHub.Infrastructure`.

### Rule PR-05

No production project may reference a test project.

### Rule PR-06

Test projects may reference the production projects required by their test scope.

---

# 47. Architectural Dependency Diagram

The final intended dependency graph is:

```text
                         ┌──────────────────────┐
                         │   CollectionHub.Api  │
                         │                      │
                         │ Transport / Runtime  │
                         └───────┬───────┬──────┘
                                 │       │
                                 │       │
                                 ▼       ▼
                       ┌────────────┐ ┌──────────────┐
                       │Application │ │Infrastructure│
                       └─────┬──────┘ └──────┬───────┘
                             │               │
                             │               │
                             ▼               │
                         ┌────────┐          │
                         │ Domain │◄─────────┘
                         └────────┘
```

There is no reverse dependency.

---

# 48. Technical Structure Validation

The proposed structure satisfies the following requirements:

| Requirement | Result |
|---|---|
| Domain isolation | Pass |
| Application isolation | Pass |
| Infrastructure isolation | Pass |
| API isolation | Pass |
| Compile-time dependency direction | Pass |
| Persistence isolation | Pass |
| External integration isolation | Pass |
| Test isolation | Pass |
| Framework containment | Pass |
| Package ownership | Pass |
| Composition root | Pass |
| Modular monolith compatibility | Pass |

---

# 49. Risks

## Risk 01 — API Becoming an Application Layer

Because the API references Application and Infrastructure, there is a risk that business logic could accidentally be implemented inside endpoints.

### Mitigation

Endpoints should only:

1. translate transport input,
2. invoke a use case,
3. translate the result.

---

## Risk 02 — Infrastructure Becoming a Generic Utility Project

Infrastructure may accumulate unrelated helper classes.

### Mitigation

Organize infrastructure by technical responsibility and enforce ownership.

---

## Risk 03 — Application Becoming a Service Dump

Application may accumulate generic services unrelated to use cases.

### Mitigation

Organize around application capabilities and use cases.

---

## Risk 04 — Shared Project Explosion

A generic common project could become a dependency escape hatch.

### Mitigation

Do not introduce `CollectionHub.Common` without explicit architectural justification.

---

## Risk 05 — Direct Domain Usage by API

API DTOs could accidentally expose domain entities.

### Mitigation

Use explicit transport contracts.

---

# 50. Implementation Rules

Before writing production code, the following rules should be treated as mandatory:

- [ ] Domain project contains no infrastructure package.
- [ ] Application project contains no infrastructure package.
- [ ] Infrastructure implements application contracts.
- [ ] API endpoints invoke use cases rather than repositories.
- [ ] Persistence models remain inside Infrastructure.
- [ ] External provider models remain inside Infrastructure.
- [ ] API transport DTOs remain inside API.
- [ ] Configuration loading remains outside Domain.
- [ ] Dependency composition occurs at the composition root.
- [ ] Tests do not introduce production dependency violations.

---

# 51. Recommended Initial Directory Structure

The initial physical structure should be approximately:

```text
CollectionHub/
│
├── src/
│   ├── CollectionHub.Domain/
│   │   ├── Aggregates/
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Services/
│   │   ├── Specifications/
│   │   ├── Events/
│   │   └── Exceptions/
│   │
│   ├── CollectionHub.Application/
│   │   ├── Abstractions/
│   │   ├── UseCases/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   ├── DTOs/
│   │   ├── Ports/
│   │   └── Common/
│   │
│   ├── CollectionHub.Infrastructure/
│   │   ├── Persistence/
│   │   ├── Integrations/
│   │   ├── Configuration/
│   │   ├── Runtime/
│   │   ├── Observability/
│   │   └── Common/
│   │
│   └── CollectionHub.Api/
│       ├── Endpoints/
│       ├── Contracts/
│       ├── Middleware/
│       ├── Authentication/
│       ├── Authorization/
│       └── Program.cs
│
├── tests/
│   ├── CollectionHub.Domain.Tests/
│   ├── CollectionHub.Application.Tests/
│   ├── CollectionHub.Infrastructure.Tests/
│   ├── CollectionHub.Api.Tests/
│   └── CollectionHub.Integration.Tests/
│
├── docs/
│   └── 02_ARCHITECTURE/
│
└── infrastructure/
    └── local/
```

This is the structural baseline, not yet the final implementation tree.

---

# 52. Exit Criteria

The solution structure is considered defined when:

- [x] Production projects are identified.
- [x] Test projects are identified.
- [x] Project responsibilities are defined.
- [x] Project dependencies are defined.
- [x] Forbidden dependencies are defined.
- [x] Package ownership is defined.
- [x] Composition root is defined.
- [x] Internal technical modules are identified.
- [x] Namespace strategy is defined.
- [x] Shared project policy is defined.
- [x] Architecture enforcement requirements are defined.

---

# 53. Final Decision

CollectionHub will use a **four-project modular monolith**:

```text
CollectionHub.Domain
CollectionHub.Application
CollectionHub.Infrastructure
CollectionHub.Api
```

with dedicated test projects for each architectural concern and cross-boundary integration testing.

The dependency direction is:

```text
Api
 ├── Application
 └── Infrastructure
       ├── Application
       └── Domain

Application
 └── Domain

Domain
 └── nothing
```

This structure converts the previously documented architecture into a physical .NET dependency model.

The critical architectural guarantee is:

> The project structure itself must make it difficult to violate the architecture.

The next technical design step is therefore to define the **concrete persistence architecture inside `CollectionHub.Infrastructure`**, including EF Core structure, `DbContext`, persistence models, mappings, repository implementations, transaction handling, migrations, and database boundaries.

**Technical solution structure: ESTABLISHED.**

**Project dependency direction: ESTABLISHED.**

**Next step:** `39_TECHNICAL_PERSISTENCE_DESIGN_AND_EF_CORE_ARCHITECTURE.md`