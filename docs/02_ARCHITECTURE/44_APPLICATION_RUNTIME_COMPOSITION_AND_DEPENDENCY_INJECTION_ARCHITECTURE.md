# 44 — Application Runtime Composition and Dependency Injection Architecture

## 1. Purpose

This document defines the runtime composition architecture for CollectionHub.

It establishes how the Domain, Application, Infrastructure, Persistence, and External Integration components are assembled into an executable application without compromising the architectural boundaries established in previous documents.

The objective is to define:

- the Composition Root;
- dependency injection;
- module registration;
- service lifetimes;
- configuration binding;
- persistence registration;
- domain service registration;
- application service registration;
- infrastructure adapter registration;
- domain event dispatching;
- background processing;
- health checks;
- observability;
- startup and shutdown;
- runtime dependency rules;
- testing implications.

This document defines **runtime composition**, not business behavior.

---

# 2. Architectural Context

The runtime composition must preserve the dependency direction established by the architecture.

```text
                    +----------------+
                    |     Domain     |
                    +----------------+
                           ^
                           |
                    +----------------+
                    |   Application  |
                    +----------------+
                           ^
                           |
                    +----------------+
                    | Infrastructure |
                    +----------------+
                           |
             +-------------+-------------+
             |                           |
       Persistence                External Integrations
             |                           |
             +-------------+-------------+
                           |
                     SQL Server /
                    External Systems
```

The Composition Root is responsible for assembling these components.

The components themselves must not be responsible for discovering or constructing their own infrastructure dependencies.

---

# 3. Composition Root

## 3.1 Definition

The Composition Root is the single architectural location where application dependencies are assembled.

It is responsible for:

- registering services;
- configuring infrastructure;
- binding configuration;
- configuring persistence;
- registering external adapters;
- configuring observability;
- configuring hosted services;
- validating startup configuration.

The Composition Root belongs to the executable/host layer.

---

## 3.2 Composition Root Rule

Application and Domain code must never behave as a Composition Root.

The following are prohibited:

```text
Domain entity -> creates repository
Application service -> creates DbContext
Domain service -> creates HTTP client
Use case -> new InfrastructureAdapter()
```

Instead:

```text
Composition Root
      |
      v
Dependency Injection Container
      |
      v
Application Services
      |
      v
Abstractions
      |
      v
Infrastructure Implementations
```

---

# 4. Runtime Layer Structure

The runtime is organized into the following logical layers:

```text
Host
 |
 +-- Configuration
 |
 +-- Dependency Injection
 |
 +-- Application
 |
 +-- Domain
 |
 +-- Infrastructure
       |
       +-- Persistence
       |
       +-- External Integrations
       |
       +-- Messaging
       |
       +-- Observability
```

The Host is the composition boundary.

---

# 5. Recommended Solution Structure

The previously defined solution structure should evolve toward:

```text
src/
    CollectionHub.Domain/
    CollectionHub.Application/
    CollectionHub.Infrastructure/
    CollectionHub.Persistence/
    CollectionHub.Host/

tests/
    CollectionHub.Domain.Tests/
    CollectionHub.Application.Tests/
    CollectionHub.Infrastructure.Tests/
    CollectionHub.Persistence.Tests/
    CollectionHub.IntegrationTests/
    CollectionHub.ArchitectureTests/
```

The exact project names may vary, but the architectural responsibilities must remain equivalent.

---

# 6. Dependency Direction

The dependency graph must remain acyclic.

Expected direction:

```text
CollectionHub.Domain
        ^
        |
CollectionHub.Application
        ^
        |
CollectionHub.Infrastructure
        ^
        |
CollectionHub.Host
```

If Persistence is separated as an independent project:

```text
Domain
  ^
Application
  ^
Infrastructure
  ^
Persistence
  ^
Host
```

However, the exact layering must respect the project dependency rules established previously.

Infrastructure implementations may depend on Application abstractions.

Application must not depend on Infrastructure.

Domain must not depend on either.

---

# 7. Dependency Injection Container

The initial implementation should use the native .NET dependency injection abstraction:

```text
Microsoft.Extensions.DependencyInjection
```

The container is an infrastructure/runtime concern.

Application code should depend only on abstractions and not on the container itself.

The following patterns are prohibited:

```text
IServiceProvider.GetService<T>()
```

inside ordinary application services.

Service locator usage must be avoided.

---

# 8. Service Registration Strategy

Service registration should be organized by architectural module rather than as one large registration method.

Preferred conceptual structure:

```text
AddApplication(...)
AddInfrastructure(...)
AddPersistence(...)
AddIntegrations(...)
AddObservability(...)
```

The Host invokes these registration modules.

Example composition:

```text
Host
 |
 +-- AddApplication()
 +-- AddInfrastructure()
 +-- AddPersistence()
 +-- AddIntegrations()
 +-- AddObservability()
```

Each module owns registration of its own implementations.

---

# 9. Registration Ownership

| Component | Registration Owner |
|---|---|
| Domain services | Application/Infrastructure composition |
| Application services | Application module |
| Repository abstractions | Application |
| Repository implementations | Persistence |
| DbContext | Persistence |
| External adapters | Infrastructure |
| Integration clients | Infrastructure |
| Configuration options | Corresponding module |
| Health checks | Host/Infrastructure |
| Background workers | Host/Infrastructure |
| Logging | Host |
| Telemetry | Host/Infrastructure |

The Composition Root orchestrates registration but should not contain implementation-specific details for every individual service.

---

# 10. Application Service Registration

Application services implement use-case orchestration.

They should normally be registered as:

```text id="rjzt1e"
Scoped
```

when they depend on:

- DbContext;
- repositories;
- scoped services;
- unit-of-work infrastructure.

Application services must not maintain mutable state between requests or use-case executions.

---

# 11. Domain Service Registration

Domain services are stateless by design unless the domain model explicitly requires otherwise.

Stateless domain services may use:

```text id="2ot8o5"
Singleton
```

only when they have no scoped dependencies and no mutable state.

Where uncertainty exists, `Transient` is the safer default.

Domain services must not depend directly on:

- DbContext;
- HTTP clients;
- configuration providers;
- infrastructure adapters.

If a domain rule requires external information, that dependency must be expressed through an appropriate application/infrastructure abstraction.

---

# 12. Repository Registration

Repositories are infrastructure implementations of application-facing abstractions.

Expected lifetime:

```text id="w7w7x4"
Scoped
```

This aligns repository lifetime with the DbContext.

Example conceptual mapping:

```text
ICollectionRepository
        |
        v
CollectionRepository
```

The application layer references `ICollectionRepository`.

The infrastructure layer provides `CollectionRepository`.

---

# 13. DbContext Lifetime

The EF Core DbContext should normally be:

```text id="h3f0tx"
Scoped
```

The scope should correspond to:

- one HTTP request;
- one application execution scope;
- one explicitly controlled background operation.

A DbContext must not normally be registered as Singleton.

---

# 14. DbContext Concurrency Rule

A DbContext instance must not be used concurrently by multiple operations.

This means background workers must create an explicit scope for each unit of work.

Conceptually:

```text id="j7k7tr"
Background Worker
      |
      v
Create Scope
      |
      v
Resolve DbContext
      |
      v
Execute Unit of Work
      |
      v
Dispose Scope
```

---

# 15. Transient Services

Transient lifetime is appropriate for:

- stateless lightweight services;
- pure adapters without expensive initialization;
- formatting/mapping components;
- strategy implementations where no shared state is required.

Transient services must not accidentally capture scoped dependencies in long-lived objects.

---

# 16. Singleton Services

Singleton lifetime is restricted to services that are:

- stateless;
- thread-safe;
- independent of scoped services;
- safe to share across the application lifetime.

Examples may include:

- immutable configuration-derived services;
- pure algorithmic services;
- static metadata providers.

Singleton should not be used merely because a service appears efficient to share.

---

# 17. Lifetime Compatibility

The following rule is mandatory:

```text id="lpxqci"
Singleton
    X
    -> Scoped
```

A Singleton must never directly depend on a Scoped service.

Similarly, a long-lived hosted service must not capture a scoped DbContext.

Instead:

```text id="8y1x5m"
Singleton Worker
      |
      v
IServiceScopeFactory
      |
      v
Scoped Dependencies
```

---

# 18. Configuration Architecture

Configuration is divided into:

```text
Application configuration
Infrastructure configuration
Persistence configuration
Integration configuration
Observability configuration
Host configuration
```

Configuration objects should use strongly typed options.

---

# 19. Options Pattern

The preferred configuration mechanism is the .NET Options pattern.

Conceptually:

```text id="t7xq0w"
Configuration
     |
     v
Options Binding
     |
     v
Strongly Typed Options
     |
     v
Service
```

Services must not repeatedly query raw configuration keys.

Avoid:

```text id="w8e8b5"
configuration["Some:Random:Key"]
```

throughout application code.

Prefer strongly typed options.

---

# 20. Configuration Validation

Critical configuration must be validated during startup.

Examples:

- database connection configuration;
- external provider configuration;
- required endpoints;
- integration identifiers;
- mandatory operational settings.

Invalid configuration should prevent the application from starting in environments where the setting is mandatory.

---

# 21. Secret Management

Secrets must not be represented as ordinary source-controlled configuration.

Examples:

- database passwords;
- API keys;
- client secrets;
- signing keys;
- external service credentials.

The application should receive secrets through the environment's approved secret-management mechanism.

---

# 22. Environment Configuration

The runtime must support environment-specific configuration without changing the application architecture.

Typical environments:

```text id="f5z6oe"
Development
Test
Staging
Production
```

Environment configuration may change:

- endpoints;
- credentials;
- logging levels;
- telemetry configuration;
- feature availability.

It must not change domain semantics.

---

# 23. Application Startup Sequence

The startup sequence should follow this conceptual order:

```text
1. Create Host
2. Load configuration
3. Register services
4. Validate configuration
5. Build service provider
6. Initialize infrastructure
7. Validate database readiness
8. Initialize hosted services
9. Start application endpoints
10. Report healthy state
```

The exact order depends on the hosting model.

---

# 24. Database Initialization

Database initialization must be explicit.

The application should distinguish between:

- checking connectivity;
- applying migrations;
- initializing reference data;
- normal runtime operation.

Production applications should not automatically apply arbitrary migrations during startup unless explicitly approved as part of the deployment architecture.

---

# 25. Application Shutdown

The application must support graceful shutdown.

Shutdown sequence:

```text id="grz0id"
Stop accepting new work
        |
        v
Finish active operations
        |
        v
Stop background workers
        |
        v
Flush telemetry/logging
        |
        v
Dispose scoped resources
        |
        v
Terminate process
```

Timeouts must be configurable.

---

# 26. Background Processing

Background work must not be implemented through uncontrolled fire-and-forget tasks.

Prohibited:

```text id="1n1qsz"
_ = DoSomethingAsync();
```

when the operation requires lifecycle management.

Background processing should use hosted services or an appropriate application-level scheduling mechanism.

---

# 27. Background Worker Scope

Every background unit of work must create its own dependency injection scope.

Conceptually:

```text id="0bq1ml"
BackgroundService
      |
      +-- CreateScope()
             |
             +-- Application Service
             +-- Repository
             +-- DbContext
```

The scope must be disposed after completion.

---

# 28. Background Worker Failure Handling

A background worker must distinguish between:

- recoverable transient errors;
- permanent business failures;
- configuration failures;
- infrastructure failures.

Retry policies must be bounded and observable.

A worker must not enter an infinite retry loop.

---

# 29. External Integration Registration

External adapters should be registered behind application-facing abstractions.

Example:

```text id="3hplvk"
IExternalCollectionProvider
        |
        v
ExternalCollectionProvider
```

The application should not instantiate HTTP clients or SDK clients directly.

---

# 30. HTTP Client Architecture

External HTTP integrations should use the .NET HTTP client infrastructure.

Typed or named clients should be preferred over constructing `HttpClient` manually.

Conceptually:

```text id="j4h5v7"
Application Port
      |
      v
Integration Adapter
      |
      v
Typed HttpClient
      |
      v
External API
```

This enables centralized:

- timeout;
- retry;
- authentication;
- telemetry;
- connection management.

---

# 31. Integration Configuration

Each external integration should have its own strongly typed configuration section.

Conceptually:

```text id="5k1zcd"
ExternalProviders
    ProviderA
        Endpoint
        Timeout
        Credentials
        Options
```

Integration-specific configuration must not leak into domain entities.

---

# 32. Domain Events

Domain events represent business-significant occurrences generated by the domain model.

They must remain independent from infrastructure implementation.

The runtime composition must provide a mechanism for dispatching them.

Conceptually:

```text id="s2f83o"
Aggregate
   |
   v
Domain Event
   |
   v
Event Dispatcher
   |
   +-- Handler A
   +-- Handler B
   +-- Handler C
```

---

# 33. Domain Event Dispatcher

The event dispatcher belongs to the application/infrastructure boundary.

It must not require domain objects to know:

- DI container;
- event bus;
- database;
- message broker.

The dispatcher resolves handlers through dependency injection.

---

# 34. Domain Event Handler Lifetime

Domain event handlers should normally be:

```text id="h7n7o0"
Scoped
```

when they depend on:

- repositories;
- DbContext;
- application services.

Handlers must not hold mutable state between events.

---

# 35. Domain Events and Transactions

The architecture must distinguish:

### In-process domain events

Executed within the local application boundary.

### Integration events

Published externally.

These must not be treated as identical mechanisms.

If reliable external delivery is required, the architecture should introduce an outbox mechanism rather than relying on an in-memory event dispatch alone.

---

# 36. Outbox Decision

The need for an outbox remains an architectural decision dependent on external integration reliability requirements.

If external operations require guaranteed eventual publication:

```text id="d7qk48"
Aggregate Change
      |
      v
Local Transaction
      |
      +-- Domain State
      |
      +-- Outbox Message
      |
      v
Commit
      |
      v
Background Publisher
      |
      v
External System
```

The outbox mechanism must be introduced before production integration workflows that require this guarantee.

---

# 37. Health Checks

The runtime should expose health information appropriate to the hosting environment.

Health checks should distinguish:

### Liveness

Is the process running?

### Readiness

Can the application serve requests safely?

### Dependency health

Are critical dependencies available?

---

# 38. Database Health

Database health checks should verify connectivity where required.

A failed database connection should make the application report an unhealthy or unready state depending on the hosting strategy.

Health checks must not perform expensive business queries.

---

# 39. External Dependency Health

External provider health checks must be used selectively.

A remote provider being temporarily unavailable should not necessarily make the entire CollectionHub application unavailable.

Dependency criticality must determine readiness semantics.

---

# 40. Observability Composition

Observability should be configured centrally.

The runtime should integrate:

- structured logging;
- metrics;
- tracing;
- correlation;
- health checks.

The implementation must use the chosen .NET observability stack consistently.

---

# 41. Logging

Logging must be:

- structured;
- contextual;
- searchable;
- environment-aware.

Logs should contain useful identifiers such as:

- correlation ID;
- operation ID;
- aggregate ID where appropriate;
- external operation ID where applicable.

Sensitive data must never be logged.

---

# 42. Correlation

Application operations should support correlation across:

```text id="v9t2kp"
Incoming Request
      |
      v
Application Use Case
      |
      v
Persistence
      |
      v
Integration
      |
      v
External System
```

Correlation identifiers must be infrastructure concerns.

The domain model should not depend on them.

---

# 43. Dependency Graph Validation

The architecture should include automated tests that verify dependency rules.

Examples:

```text id="u8wqk0"
Domain
  X-> Infrastructure

Application
  X-> Persistence Implementation

Application
  X-> SQL Server

Domain
  X-> EF Core
```

Architecture tests should fail if forbidden dependencies are introduced.

---

# 44. Circular Dependency Prevention

Circular dependencies are prohibited.

Examples:

```text id="4l0n7h"
Application -> Infrastructure -> Application
```

must be avoided.

If Infrastructure needs to implement an Application abstraction, that is valid:

```text id="p3q1mc"
Application
    |
    | abstraction
    v
Infrastructure
    |
    +-- implementation
```

Infrastructure must not force Application to depend back on its implementation.

---

# 45. Composition Root Testing

The Composition Root must be testable.

Startup tests should verify:

- all required services resolve;
- configuration validation works;
- required dependencies are registered;
- no invalid lifetime relationships exist;
- DbContext resolves;
- repositories resolve;
- integration adapters resolve.

---

# 46. Service Registration Validation

The application should validate the service graph during tests and, where appropriate, during startup.

Common failures to detect:

- missing registration;
- duplicate conflicting registration;
- invalid lifetime;
- unresolved constructor dependency;
- configuration missing.

---

# 47. Service Registration Conventions

Registration methods should follow a predictable naming convention:

```text id="q1z8i8"
AddApplication(...)
AddPersistence(...)
AddInfrastructure(...)
AddIntegrations(...)
AddObservability(...)
```

Each method should be cohesive and focused on one architectural responsibility.

---

# 48. Avoiding Registration Sprawl

The Host must not become a 1,000-line service registration file.

Instead:

```text id="7uw3yk"
Program / Host
    |
    +-- Application registration
    +-- Persistence registration
    +-- Infrastructure registration
    +-- Integration registration
    +-- Observability registration
```

The Host orchestrates modules.

Each module owns its internal registration knowledge.

---

# 49. Feature Registration

Where useful, registration may be grouped by feature rather than technical type.

For example:

```text id="h2f5po"
AddCollectionFeature()
AddSynchronizationFeature()
AddClassificationFeature()
```

This is appropriate when the feature boundary corresponds to a real application capability.

Technical organization must not create artificial module boundaries.

---

# 50. Cross-Cutting Services

Cross-cutting infrastructure includes:

- clock abstraction;
- ID generation;
- serialization;
- logging;
- telemetry;
- validation;
- transaction management;
- resilience.

These services must be registered centrally.

The domain should depend only on abstractions where a cross-cutting service is genuinely part of domain behavior.

---

# 51. Clock Abstraction

Where deterministic time is required for domain logic or testing, the architecture should provide an abstraction such as:

```text id="qcnz8p"
IClock
```

or an equivalent time abstraction.

Infrastructure provides the real implementation.

Tests provide deterministic implementations.

The domain must not directly call infrastructure-specific system time APIs where doing so would compromise deterministic behavior.

---

# 52. Identifier Generation

Technical identifier generation should be centralized where necessary.

The system must avoid multiple inconsistent ID-generation mechanisms.

The chosen strategy must remain compatible with the persistence model.

---

# 53. Validation

Validation should be separated into:

### Domain validation

Business invariants.

### Application validation

Command/request/use-case validation.

### Infrastructure validation

Configuration and dependency validation.

### Database validation

Relational constraints.

Each layer validates what it owns.

---

# 54. Error Handling

Infrastructure exceptions must not leak indiscriminately through application boundaries.

Examples:

```text id="y7q3vw"
SqlException
HttpRequestException
DbUpdateConcurrencyException
```

must be translated or handled at the appropriate infrastructure/application boundary.

Domain exceptions remain domain concepts.

---

# 55. Retry Policies

Retries belong to infrastructure boundaries.

Appropriate targets include:

- transient database connectivity;
- external HTTP calls;
- messaging infrastructure.

Retries must not be applied blindly to domain operations.

A retry must be safe with respect to idempotency.

---

# 56. Idempotency

Any retryable application/integration operation must define whether it is idempotent.

This is especially important for:

- synchronization;
- external updates;
- background processing;
- message handling.

Infrastructure retry policy must not create duplicate business effects.

---

# 57. Runtime Feature Flags

Feature flags, if introduced, belong to application/infrastructure configuration.

The domain should not depend directly on a specific feature-flag provider.

Where a business rule genuinely depends on a configurable policy, that policy should be expressed as an application/domain abstraction.

---

# 58. Startup Failure Policy

The application should fail fast when critical configuration is invalid.

Examples:

- missing database connection;
- invalid mandatory integration configuration;
- invalid cryptographic configuration;
- impossible runtime options.

Non-critical optional integrations should not necessarily prevent startup.

---

# 59. Shutdown Failure Policy

Shutdown should be best-effort and bounded.

A component that fails to stop within the configured timeout must not prevent indefinite process termination.

Telemetry/logging should attempt to flush before shutdown completes.

---

# 60. Runtime Configuration Matrix

| Configuration | Owner | Required | Validation |
|---|---|---:|---:|
| Database | Persistence | Yes | Startup |
| External provider | Integration | Conditional | Startup |
| Logging | Host | Yes | Startup |
| Telemetry | Host | Optional/Environment | Startup |
| Background workers | Host | Conditional | Startup |
| Feature flags | Application | Conditional | Startup |
| Migration execution | Deployment | Conditional | Deployment |

---

# 61. Lifetime Matrix

| Service | Lifetime |
|---|---|
| DbContext | Scoped |
| Repository | Scoped |
| Application Service | Scoped |
| Domain Service | Transient by default |
| Stateless immutable service | Singleton when justified |
| Domain Event Handler | Scoped |
| Typed HTTP Client | Managed by HttpClientFactory |
| Background Worker | Singleton hosted service |
| Configuration Options | Options-managed |
| Clock | Singleton/Scoped depending implementation |
| Mapper | Stateless appropriate lifetime |

The lifetime must always reflect dependency compatibility.

---

# 62. Runtime Composition Example

Conceptually:

```text id="q2xj0m"
Host
 |
 +-- Configuration
 |
 +-- AddApplication()
 |      |
 |      +-- Use Cases
 |      +-- Domain Event Handlers
 |
 +-- AddPersistence()
 |      |
 |      +-- DbContext
 |      +-- Repositories
 |
 +-- AddInfrastructure()
 |      |
 |      +-- Clock
 |      +-- Serialization
 |      +-- Resilience
 |
 +-- AddIntegrations()
 |      |
 |      +-- External Clients
 |      +-- Adapters
 |
 +-- AddObservability()
        |
        +-- Logging
        +-- Metrics
        +-- Tracing
        +-- Health Checks
```

---

# 63. Application Execution Flow

A normal request/use-case execution should conceptually follow:

```text id="n1y8xm"
Entry Point
    |
    v
Host / Endpoint
    |
    v
Application Use Case
    |
    v
Domain Model
    |
    v
Repository / Infrastructure Port
    |
    v
Persistence
    |
    v
Database
```

External integration:

```text id="g8qv5k"
Application Use Case
    |
    v
Integration Port
    |
    v
Infrastructure Adapter
    |
    v
External System
```

---

# 64. Background Execution Flow

```text id="5x2zq7"
Hosted Service
     |
     v
Create Scope
     |
     v
Application Use Case
     |
     v
Domain
     |
     +-- Persistence
     |
     +-- Integration
     |
     v
Complete Scope
```

This ensures that background processing follows the same architectural rules as interactive execution.

---

# 65. Runtime Anti-Patterns

The following are prohibited:

### Service locator

```text id="j5s4ck"
IServiceProvider.GetService(...)
```

inside business services.

### Direct infrastructure construction

```text id="7o2l5j"
new SqlConnection(...)
new HttpClient(...)
new Repository(...)
```

inside application services.

### Static mutable service state

```text id="l9g2e0"
static MutableState
```

for request/application state.

### Singleton DbContext

Not permitted.

### Fire-and-forget business operations

Not permitted without explicit lifecycle management.

### Domain dependency on configuration

Not permitted.

### Domain dependency on HTTP/database APIs

Not permitted.

---

# 66. Architecture Testing Requirements

Architecture tests should validate:

- dependency direction;
- namespace boundaries;
- forbidden project references;
- absence of infrastructure references in Domain;
- absence of persistence implementation references in Application;
- Composition Root location;
- repository abstraction placement;
- configuration boundary.

These tests become part of the architectural safety net.

---

# 67. Implementation Readiness Checklist

Before implementation:

- [ ] Composition Root identified.
- [ ] Service registration modules defined.
- [ ] Dependency direction validated.
- [ ] DbContext lifetime defined.
- [ ] Repository lifetime defined.
- [ ] Application service lifetime defined.
- [ ] Domain service lifetime defined.
- [ ] Background worker scope strategy defined.
- [ ] Configuration options defined.
- [ ] Startup validation defined.
- [ ] External client registration defined.
- [ ] Domain event dispatcher strategy defined.
- [ ] Health check strategy defined.
- [ ] Observability composition defined.
- [ ] Retry boundaries defined.
- [ ] Error translation boundaries defined.
- [ ] Architecture tests defined.
- [ ] Secret-management boundary defined.
- [ ] Production migration execution separated from runtime startup.

---

# 68. Architectural Decisions

The following decisions are established:

| Decision | Status |
|---|---|
| Composition Root belongs to Host | Accepted |
| Application does not construct infrastructure | Accepted |
| Domain does not depend on infrastructure | Accepted |
| DbContext is Scoped | Accepted |
| Repositories are Scoped | Accepted |
| Application services are Scoped | Accepted |
| Stateless domain services are Transient by default | Accepted |
| Singleton requires thread-safety and dependency compatibility | Accepted |
| Strongly typed configuration is preferred | Accepted |
| Secrets remain external to source code | Accepted |
| Background workers create scopes | Accepted |
| Hosted services manage background execution | Accepted |
| EF Core is isolated within persistence/infrastructure | Accepted |
| External clients are registered through infrastructure | Accepted |
| Architecture tests enforce dependency rules | Accepted |
| Production schema migration is separated from normal startup | Accepted |

---

# 69. Open Decisions

The following remain implementation-dependent:

1. Exact hosting model.
2. Exact endpoint technology.
3. Final external provider client implementations.
4. Whether an outbox is required.
5. Exact health-check dependency classification.
6. Final telemetry provider.
7. Background processing technology if requirements exceed native hosted services.
8. Final feature-flag mechanism if required.

These decisions must not alter the core dependency architecture.

---

# 70. Readiness Assessment

## Runtime composition status

**READY FOR IMPLEMENTATION PLANNING**

The architecture defines:

- where dependencies are assembled;
- how dependencies flow;
- how services are registered;
- how lifetimes are controlled;
- how configuration enters the system;
- how persistence is connected;
- how integrations are connected;
- how background work obtains scoped dependencies;
- how health and observability are integrated;
- how architectural rules are tested.

---

# 71. Relationship With Previous Architecture

This document builds directly upon:

```text id="qz1z4d"
24 — Architectural Boundaries and Layers
25 — Architectural Components and Responsibilities
26 — Application and Domain Module Structure
27 — Component Interactions and Dependency Rules
28 — Application Use Case Interaction Map
29 — Infrastructure Components and Adapters
30 — Persistence Architecture and Data Boundaries
31 — Persistence Components and Repository Implementations
32 — Configuration and Runtime Infrastructure
33 — External Integration Infrastructure
34 — Infrastructure Architecture Consistency Review
38 — Technical Solution Structure and Project Dependencies
39 — Technical Persistence Design and EF Core Architecture
40 — Database Schema and Domain Persistence Mapping
41 — Database Schema Definition and Table Catalog
42 — Database Constraints, Indexes and Migration Strategy
43 — Persistence Architecture Implementation Readiness Review
```

It translates those architectural boundaries into runtime composition rules.

---

# 72. Phase Status

The persistence phase is now architecturally closed.

The runtime composition phase is now defined at architectural level.

The project has therefore progressed from:

```text
Domain Architecture
        |
        v
Application Architecture
        |
        v
Infrastructure Architecture
        |
        v
Persistence Architecture
        |
        v
Runtime Composition Architecture
```

The next work should continue with the remaining runtime and hosting concerns without reopening the already-established persistence architecture unless a concrete contradiction is discovered.

---

# 73. Next Step

The next document should define the **runtime configuration, application hosting, startup lifecycle, environment configuration, health model, and operational execution model** in greater detail.

The proposed next file is:

```text
45_APPLICATION_HOSTING_CONFIGURATION_AND_RUNTIME_LIFECYCLE.md
```

That document will establish the boundary between the architectural Composition Root defined here and the concrete runtime environment in which CollectionHub executes.

---

# 74. Final Status

**Application Runtime Composition: DEFINED**

**Dependency Injection Architecture: DEFINED**

**Service Lifetimes: DEFINED**

**Configuration Boundary: DEFINED**

**Background Processing Boundary: DEFINED**

**Observability Boundary: DEFINED**

**Architecture Testing Requirements: DEFINED**

**Persistence Architecture: CLOSED**

**Overall Runtime Architecture: READY FOR NEXT STAGE**