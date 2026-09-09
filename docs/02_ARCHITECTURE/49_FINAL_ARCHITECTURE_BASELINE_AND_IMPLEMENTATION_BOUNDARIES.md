# 49 — Final Architecture Baseline and Implementation Boundaries

## 1. Purpose

This document establishes the **Final Architecture Baseline** for CollectionHub.

It consolidates the architectural decisions produced throughout the Domain, Application, Infrastructure, Persistence, Runtime, and Observability phases into a single implementation-oriented baseline.

Its purpose is to define:

- what is architecturally decided,
- what boundaries are considered stable,
- which dependencies are permitted,
- which responsibilities belong to each layer,
- which implementation structures are expected,
- which decisions remain open,
- and where architectural design ends and implementation begins.

This document is not intended to replace the detailed architectural documents that precede it.

Instead, it acts as the **authoritative architectural baseline** against which implementation decisions can be evaluated.

---

# 2. Architectural Status

The current CollectionHub architecture is considered:

```text
┌─────────────────────────────────────────────────┐
│             ARCHITECTURAL BASELINE              │
│                                                 │
│ Status: DEFINED                                 │
│ Consistency: PASSED                             │
│ Traceability: PASSED                            │
│ Major blockers: NONE                             │
│ Implementation readiness: READY WITH CONDITIONS │
└─────────────────────────────────────────────────┘
```

The architecture is sufficiently mature to begin implementation planning and controlled implementation.

This does not mean that every implementation detail has been predetermined.

It means that implementation should now occur **inside stable architectural boundaries** rather than continuing to redesign the architecture implicitly through code.

---

# 3. Architectural Baseline Principle

The principal rule for the implementation phase is:

> Implementation may refine the architecture, but must not silently redefine it.

Any implementation decision that changes:

- dependency direction,
- aggregate boundaries,
- application responsibilities,
- persistence boundaries,
- external integration boundaries,
- transaction semantics,
- runtime composition,
- or security architecture

must be treated as an architectural change rather than an ordinary coding decision.

---

# 4. Final Architectural Model

The approved architectural model is:

```text
┌───────────────────────────────────────────────────────────┐
│                       HOSTING                             │
│                                                           │
│ HTTP / Process / Runtime / Configuration / Lifecycle      │
│ Middleware / Health / Entry Points                        │
└───────────────────────────────┬───────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────┐
│                    INFRASTRUCTURE                         │
│                                                           │
│ Persistence │ External Integrations │ Configuration       │
│ Observability │ Resilience │ Runtime Services             │
└───────────────────────────────┬───────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────┐
│                      APPLICATION                          │
│                                                           │
│ Use Cases │ Workflows │ Application Services │ Ports      │
└───────────────────────────────┬───────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────┐
│                         DOMAIN                             │
│                                                           │
│ Entities │ Aggregates │ Value Objects │ Invariants         │
│ Domain Services │ Specifications │ Domain Events           │
└───────────────────────────────────────────────────────────┘
```

The architecture follows a dependency direction from outer technical concerns toward inner business concerns.

---

# 5. Dependency Rule

The definitive dependency rule is:

```text
Hosting
   ↓
Infrastructure
   ↓
Application
   ↓
Domain
```

The reverse direction is prohibited.

In particular:

```text
Domain ─────X────> Infrastructure
Domain ─────X────> Hosting
Domain ─────X────> Persistence
Domain ─────X────> External APIs

Application ──X──> Hosting
Application ──X──> Concrete Persistence
Application ──X──> Vendor SDKs
```

Application abstractions may be implemented by Infrastructure.

---

# 6. Domain Boundary

The Domain is the innermost architectural boundary.

It owns business meaning and business correctness.

The Domain contains concepts such as:

- entities,
- aggregates,
- value objects,
- domain invariants,
- domain services,
- specifications,
- domain events.

The Domain must remain independent of:

- EF Core,
- ASP.NET Core,
- HTTP,
- database providers,
- telemetry providers,
- configuration systems,
- dependency injection frameworks,
- external SDKs.

---

# 7. Domain Responsibilities

The Domain is responsible for:

1. Representing business concepts.
2. Protecting invariants.
3. Defining aggregate consistency boundaries.
4. Controlling valid state transitions.
5. Expressing reusable domain rules.
6. Representing meaningful domain events.
7. Coordinating domain behavior where it cannot belong to a single aggregate.

The Domain is not responsible for:

- persistence,
- HTTP transport,
- user interfaces,
- external API calls,
- process lifecycle,
- logging implementation,
- configuration loading.

---

# 8. Aggregate Boundary Baseline

Aggregates remain the primary consistency boundaries defined by the Domain model.

An aggregate:

- protects its invariants,
- controls state changes,
- exposes behavior rather than uncontrolled mutation,
- is persisted as an intentional consistency unit.

Application code must not bypass aggregate behavior merely to simplify persistence.

The database schema must reflect the persistence requirements of these boundaries rather than independently redefine them.

---

# 9. Application Boundary

The Application layer represents executable business capabilities.

It owns:

- use cases,
- application services,
- workflows,
- orchestration,
- application-level policies,
- ports required to execute use cases.

The Application layer coordinates the Domain.

It does not redefine the Domain.

---

# 10. Application Use Case Model

Each externally meaningful application capability should be represented by an explicit use case or equivalent application operation.

The general execution model is:

```text
Entry Point
     │
     ▼
Application Use Case
     │
     ├── Load Aggregate
     ├── Execute Domain Behavior
     ├── Coordinate Supporting Services
     ├── Persist Changes
     └── Publish / Trigger Required Side Effects
     │
     ▼
Result
```

The use case is the principal application-level orchestration boundary.

---

# 11. Application Port Model

Application code must communicate with infrastructure through explicit abstractions where infrastructure details are not part of the application model.

Examples include:

- repository interfaces,
- external service ports,
- application infrastructure services,
- persistence abstractions.

The Application must not instantiate concrete infrastructure classes.

---

# 12. Infrastructure Boundary

Infrastructure contains technical implementations required by the Application and Runtime.

The infrastructure baseline includes:

```text
Infrastructure
├── Persistence
├── External Integrations
├── Configuration
├── Runtime Services
├── Observability
└── Resilience
```

Infrastructure must implement technical concerns without becoming a hidden business layer.

---

# 13. Persistence Boundary

Persistence remains isolated within Infrastructure.

The expected structure is:

```text
Application
    │
    ▼
Repository / Persistence Abstraction
    ▲
    │
Infrastructure.Persistence
    │
    ▼
EF Core
    │
    ▼
Database
```

Persistence owns:

- EF Core configuration,
- entity mappings,
- database access,
- transactions,
- persistence-specific queries,
- migrations,
- database constraints,
- indexes,
- concurrency mechanisms.

---

# 14. Database Boundary

The database is a persistence integrity boundary.

It is responsible for enforcing appropriate structural guarantees such as:

- primary keys,
- foreign keys,
- uniqueness,
- referential integrity,
- appropriate check constraints,
- indexes,
- concurrency-related persistence mechanisms.

The database does not replace domain validation.

The rule remains:

> Business correctness belongs to the Domain; persistence integrity belongs to the Database.

---

# 15. Migration Boundary

Database schema evolution must occur through controlled migrations.

Migrations must be:

- deterministic,
- reviewable,
- versioned,
- reproducible,
- aligned with the persistence model.

Schema changes must not be introduced manually as an undocumented part of implementation.

---

# 16. External Integration Boundary

External systems are isolated through Infrastructure adapters.

The model is:

```text
Application
     │
     ▼
External Capability Port
     ▲
     │
Infrastructure Adapter
     │
     ▼
External System
```

Infrastructure owns:

- transport,
- serialization,
- protocol handling,
- authentication transport,
- timeout configuration,
- retry policies,
- provider-specific error handling.

Application code owns the decision to use the external capability.

---

# 17. Runtime Boundary

Runtime infrastructure is responsible for constructing and executing the application.

It owns:

- dependency injection,
- configuration,
- service registration,
- startup,
- shutdown,
- lifecycle,
- health,
- runtime diagnostics.

The runtime must not become an alternative application orchestration layer.

---

# 18. Composition Root

The Composition Root is the authoritative location for concrete dependency construction.

Conceptually:

```text
Composition Root
│
├── Domain Services
├── Application Services
├── Persistence
├── External Integrations
├── Configuration
├── Observability
├── Health Checks
├── Resilience
└── Hosting
```

The composition root decides **which implementation** satisfies each dependency.

Consumers decide **which capability** they require.

---

# 19. Dependency Injection Baseline

Dependency Injection is the standard mechanism for connecting architectural components.

The implementation must avoid:

- service locator patterns,
- global mutable dependencies,
- hidden static dependencies,
- manual construction of infrastructure inside Application code.

Dependencies should be explicit and discoverable.

---

# 20. Configuration Baseline

Configuration is an external runtime concern.

The application must distinguish between:

- configuration structure,
- configuration values,
- secrets,
- environment-specific settings.

Required configuration should be validated at startup.

Secrets must never be committed to source control.

Configuration models should avoid leaking environment-specific mechanisms into the Domain.

---

# 21. Hosting Baseline

Hosting owns the process and transport lifecycle.

For a web-hosted application, the conceptual pipeline is:

```text
Incoming Request
      │
      ▼
Runtime Middleware
      │
      ▼
Application Entry Point
      │
      ▼
Application Use Case
      │
      ▼
Infrastructure
      │
      ▼
Response
```

The exact middleware technology and endpoint framework remain implementation details.

---

# 22. Lifecycle Baseline

Startup follows:

```text
Process Start
     ↓
Configuration Load
     ↓
Configuration Validation
     ↓
Service Registration
     ↓
Runtime Construction
     ↓
Infrastructure Initialization
     ↓
Application Ready
```

Shutdown follows:

```text
Shutdown Signal
     ↓
Stop Accepting Work
     ↓
Cancellation
     ↓
Graceful Completion
     ↓
Resource Disposal
     ↓
Process Exit
```

---

# 23. Observability Baseline

Observability surrounds the runtime.

It includes:

- structured logging,
- correlation,
- tracing,
- metrics,
- health checks,
- runtime diagnostics.

The conceptual model is:

```text
┌───────────────────────────────────────────┐
│             Observability                 │
│                                           │
│ Logs │ Metrics │ Traces │ Health          │
│                                           │
│      ┌───────────────────────────┐        │
│      │       Application         │        │
│      │                           │        │
│      │ Domain + Use Cases        │        │
│      └───────────────────────────┘        │
│                                           │
└───────────────────────────────────────────┘
```

Observability must not alter business semantics.

---

# 24. Logging Baseline

Logging must be:

- structured,
- contextual,
- correlation-aware,
- configurable,
- safe.

Sensitive data must not be logged.

Business events and operational logs are distinct concepts and must not be conflated.

---

# 25. Health Baseline

The runtime distinguishes:

### Liveness

Indicates whether the process can continue operating.

### Readiness

Indicates whether the application is capable of serving requests correctly.

Mandatory dependencies may affect readiness.

Optional dependencies may result in degraded behavior without necessarily making the application unavailable.

---

# 26. Resilience Baseline

Resilience belongs primarily to infrastructure boundaries.

Possible mechanisms include:

- timeout,
- retry,
- circuit breaker,
- fallback.

Policies must be explicitly associated with the relevant dependency.

Retries must respect idempotency.

No retry policy may be introduced merely because a framework provides one.

---

# 27. Exception Boundary

Internal exceptions must not leak across architectural boundaries without deliberate translation.

The expected model is:

```text
Domain / Infrastructure Failure
          │
          ▼
Application Boundary
          │
          ▼
Application Failure Model
          │
          ▼
External Representation
```

Infrastructure-specific details must remain internal.

---

# 28. Background Processing Boundary

Background processing must use the same architectural model as synchronous operations.

It must participate in:

- Dependency Injection,
- Application use cases,
- configuration,
- logging,
- correlation,
- cancellation,
- resilience,
- error handling.

Background workers must not directly manipulate persistence or domain state outside the established application boundaries.

---

# 29. Cross-Cutting Responsibility Matrix

| Concern | Domain | Application | Infrastructure | Hosting |
|---|---:|---:|---:|---:|
| Business rules | Primary | Coordinates | No | No |
| Use cases | No | Primary | No | Entry point |
| Persistence | No | Abstraction | Primary | No |
| External integrations | No | Abstraction | Primary | No |
| Configuration | No | Consumes | Primary | Primary |
| Dependency Injection | No | Consumes | Implements | Composes |
| Logging | No | Indirect | Primary | Primary |
| Metrics | No | Indirect | Primary | Primary |
| Tracing | No | Indirect | Primary | Primary |
| Health | No | No | Primary | Primary |
| Resilience | No | Policy awareness | Primary | Supporting |
| Process lifecycle | No | No | Supporting | Primary |
| HTTP lifecycle | No | No | Supporting | Primary |

---

# 30. Technical Solution Structure

The implementation structure should reflect the architectural model rather than forcing the architecture to adapt to an arbitrary project layout.

The expected logical solution organization is:

```text
CollectionHub
│
├── Domain
│   ├── Entities
│   ├── Aggregates
│   ├── ValueObjects
│   ├── Services
│   ├── Specifications
│   ├── Events
│   └── Exceptions
│
├── Application
│   ├── UseCases
│   ├── Services
│   ├── Ports
│   ├── DTOs
│   ├── Behaviors
│   └── Common
│
├── Infrastructure
│   ├── Persistence
│   ├── Integrations
│   ├── Configuration
│   ├── Observability
│   ├── Resilience
│   └── Runtime
│
└── Hosting
    ├── Endpoints
    ├── Middleware
    ├── Configuration
    ├── Composition
    └── Lifecycle
```

The exact namespaces and project names may be refined during implementation, but the responsibility boundaries must remain intact.

---

# 31. Project Dependency Baseline

The permitted project dependency direction is:

```text
Hosting
   │
   └──► Infrastructure
            │
            └──► Application
                     │
                     └──► Domain
```

Infrastructure may additionally depend on external technical libraries.

Hosting may depend on the runtime framework.

Domain should minimize external dependencies and remain technology-independent.

---

# 32. Dependency Graph

The conceptual dependency graph is:

```text
                 ┌──────────────┐
                 │    Domain    │
                 └──────▲───────┘
                        │
                 ┌──────┴───────┐
                 │ Application  │
                 └──────▲───────┘
                        │
              ┌─────────┴─────────┐
              │ Infrastructure    │
              └─────────▲─────────┘
                        │
                 ┌──────┴───────┐
                 │    Hosting    │
                 └──────────────┘
```

No dependency should bypass an architectural boundary without explicit justification.

---

# 33. Implementation Boundaries

The following boundaries are considered stable.

## Boundary A — Domain

Implementation may refine internal domain structures.

It must not introduce infrastructure dependencies.

## Boundary B — Application

Implementation may refine use-case orchestration.

It must not become dependent on concrete infrastructure.

## Boundary C — Infrastructure

Implementation may select concrete technologies.

It must preserve Application and Domain boundaries.

## Boundary D — Hosting

Implementation may select transport/runtime mechanisms.

It must not contain business rules.

---

# 34. Stable Decisions

The following architectural decisions are considered stable:

1. Domain-centered architecture.
2. Explicit Domain/Application separation.
3. Infrastructure isolation.
4. Dependency inversion.
5. Explicit use cases.
6. Aggregate consistency boundaries.
7. Repository/persistence boundary.
8. External integration adapters.
9. Centralized Dependency Injection.
10. Configuration as a runtime concern.
11. Hosting lifecycle separation.
12. Structured observability.
13. Liveness/readiness distinction.
14. Controlled database migrations.
15. Database integrity constraints.
16. Explicit exception boundaries.
17. Infrastructure-level resilience.
18. Background processing through application boundaries.

These decisions should not be casually revisited during coding.

---

# 35. Decisions Deliberately Deferred

The following remain implementation or deployment decisions:

- exact logging provider,
- exact telemetry backend,
- production metrics backend,
- tracing backend,
- concrete authentication provider,
- concrete authorization implementation,
- secret-management platform,
- deployment topology,
- containerization details,
- CI/CD implementation,
- production alert thresholds,
- telemetry retention,
- infrastructure scaling parameters.

These decisions can be made without changing the core architecture.

---

# 36. Implementation-Safe Decisions

The following decisions may generally be made by developers without requiring architectural review, provided they remain within the baseline:

- class naming,
- method naming,
- private helper methods,
- internal DTO organization,
- test naming,
- internal mapping organization,
- internal utility extraction,
- implementation-level refactoring,
- dependency registration ordering where semantically irrelevant,
- framework configuration details that do not change architectural behavior.

---

# 37. Architecture-Change Decisions

The following require explicit architectural review:

- changing aggregate boundaries,
- adding a new architectural layer,
- changing dependency direction,
- introducing infrastructure dependencies into Domain,
- bypassing Application use cases,
- changing persistence ownership,
- introducing distributed transactions,
- changing external integration ownership,
- changing transaction semantics,
- changing domain invariants,
- introducing cross-cutting business logic into infrastructure,
- changing runtime execution boundaries.

---

# 38. Definition of Done for Architecture

The architecture baseline is considered complete when:

- all major requirements have architectural owners,
- all major domain concepts are modeled,
- use cases are identified,
- infrastructure responsibilities are defined,
- persistence boundaries are defined,
- external integrations are isolated,
- runtime composition is defined,
- hosting lifecycle is defined,
- observability is defined,
- dependency direction is stable,
- traceability is established,
- no blocking architectural contradiction remains.

CollectionHub currently satisfies these criteria.

---

# 39. Implementation Readiness

The architecture is now ready for controlled implementation.

The readiness state is:

```text
Architecture Definition
        │
        ▼
Architecture Review
        │
        ▼
Traceability
        │
        ▼
Baseline
        │
        ▼
┌────────────────────────────┐
│ IMPLEMENTATION READY       │
└────────────────────────────┘
```

Implementation should now proceed from the most foundational structures outward.

---

# 40. Recommended Implementation Order

Implementation should follow dependency direction.

A recommended sequence is:

```text
1. Domain primitives
        ↓
2. Domain entities / aggregates
        ↓
3. Domain invariants / services
        ↓
4. Domain events
        ↓
5. Application abstractions
        ↓
6. Application use cases
        ↓
7. Persistence mappings
        ↓
8. Database configuration / migrations
        ↓
9. Infrastructure adapters
        ↓
10. Runtime composition
        ↓
11. Hosting
        ↓
12. Observability
        ↓
13. Integration / end-to-end verification
```

This order minimizes premature infrastructure decisions.

---

# 41. Implementation Feedback Loop

Implementation must remain iterative.

The expected process is:

```text
Architecture
     ↓
Implementation
     ↓
Automated Tests
     ↓
Architectural Validation
     ↓
Implementation Refinement
     │
     └───────► Architecture Change
                    │
                    ▼
              Explicit Review
```

The architecture should not be considered immutable.

It should instead be considered **controlled**.

---

# 42. Architectural Change Protocol

When implementation reveals a problem, the following sequence should be followed:

1. Identify the problem.
2. Determine whether it is implementation-specific or architectural.
3. Check the existing architectural documents.
4. Determine whether an existing boundary already solves it.
5. If not, propose an architectural change.
6. Update the relevant architecture document.
7. Update the traceability matrix.
8. Re-run consistency validation.
9. Only then update implementation.

This prevents architecture drift.

---

# 43. Traceability Maintenance

The end-to-end traceability established in:

`48_END_TO_END_ARCHITECTURAL_TRACEABILITY_AND_CONSISTENCY_MATRIX.md`

must remain synchronized with implementation.

When a new major capability is added:

```text
Requirement
   ↓
Domain
   ↓
Use Case
   ↓
Component
   ↓
Infrastructure
   ↓
Persistence / Integration
   ↓
Runtime
```

must be updated accordingly.

---

# 44. Architectural Baseline Versioning

This document represents the initial final architecture baseline.

Conceptually:

```text
Architecture Baseline
        │
        ▼
Implementation
        │
        ├── Minor implementation refinement
        │
        └── Architectural change
                 │
                 ▼
          New baseline revision
```

The baseline should therefore be versioned alongside the implementation.

---

# 45. Final Boundary Statement

The definitive architectural boundaries are:

```text
┌─────────────────────────────────────────────┐
│ HOSTING                                     │
│ Process / HTTP / Lifecycle / Entry Points   │
├─────────────────────────────────────────────┤
│ INFRASTRUCTURE                              │
│ Persistence / Integrations / Runtime        │
├─────────────────────────────────────────────┤
│ APPLICATION                                 │
│ Use Cases / Workflows / Ports                │
├─────────────────────────────────────────────┤
│ DOMAIN                                      │
│ Business Model / Rules / Invariants          │
└─────────────────────────────────────────────┘
```

The direction of dependency is inward.

The direction of execution is outward-to-inward and then back outward through abstractions and adapters.

---

# 46. Final Architectural Principles

The CollectionHub implementation must preserve these principles:

1. **Business rules belong to the Domain.**
2. **Use cases belong to the Application layer.**
3. **Technical implementations belong to Infrastructure.**
4. **Process and transport concerns belong to Hosting.**
5. **Dependencies point inward.**
6. **Concrete implementations are selected by the Composition Root.**
7. **Persistence is isolated behind technical boundaries.**
8. **External systems are isolated through adapters.**
9. **Cross-cutting runtime services remain infrastructure concerns.**
10. **Database constraints complement rather than replace domain invariants.**
11. **Observability must not contaminate business logic.**
12. **Architectural changes must be explicit and traceable.**

---

# 47. Final Architecture Baseline Statement

CollectionHub has reached the point where the architecture can be treated as an implementation baseline rather than an exploratory design.

The architecture is:

```text
DEFINED
   +
TRACEABLE
   +
CONSISTENT
   +
IMPLEMENTATION-READY
```

subject to the explicit conditions and deferred decisions documented above.

The next stage should therefore focus on **implementation preparation and execution**, not further unrestricted architectural exploration.

---

# 48. Final Conclusion

The CollectionHub architecture establishes a coherent path from business intent to executable software:

```text
Business Intent
      ↓
Domain Model
      ↓
Application Use Cases
      ↓
Application Workflows
      ↓
Infrastructure Ports & Adapters
      ↓
Persistence / External Integrations
      ↓
Runtime Composition
      ↓
Hosting Lifecycle
      ↓
Observability
      ↓
Executable System
```

The architecture has:

- defined boundaries,
- explicit responsibilities,
- controlled dependencies,
- persistence isolation,
- integration isolation,
- runtime composition,
- operational visibility,
- end-to-end traceability,
- and a documented implementation boundary.

No major architectural blocker remains.

**Final status: ARCHITECTURE BASELINE ESTABLISHED — IMPLEMENTATION MAY BEGIN WITH CONTROLLED ARCHITECTURAL GOVERNANCE.**