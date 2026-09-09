# 27 — Component Interactions and Dependency Rules

## 1. Purpose

This document defines how the architectural components of CollectionHub are allowed to interact and which dependency relationships are permitted or prohibited.

The objective is to transform the structural model established in:

- `24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md`
- `25_ARCHITECTURAL_COMPONENTS_AND_RESPONSIBILITIES.md`
- `26_APPLICATION_AND_DOMAIN_MODULE_STRUCTURE.md`

into explicit interaction rules that can later be enforced through implementation conventions, code review and automated architecture tests.

The central principle is:

> **Components may collaborate across architectural boundaries, but they must not bypass those boundaries.**

---

# 2. Architectural Interaction Model

The primary interaction direction is:

```text
                         ┌───────────────────┐
                         │      Delivery     │
                         │ API / UI / CLI    │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │    Application    │
                         │ Commands/Queries  │
                         │ Workflows         │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │      Domain       │
                         │ Aggregates        │
                         │ Entities          │
                         │ Value Objects     │
                         │ Domain Services   │
                         │ Specifications    │
                         │ Domain Events     │
                         └───────────────────┘

                         ┌───────────────────┐
                         │  Infrastructure   │
                         │ Persistence       │
                         │ Messaging         │
                         │ External APIs     │
                         │ File Storage      │
                         └─────────▲─────────┘
                                   │
                         implements contracts
                                   │
                         ┌─────────┴─────────┐
                         │ Application Ports│
                         │ Domain Ports      │
                         └───────────────────┘
```

The architecture therefore follows a controlled dependency direction rather than unrestricted component-to-component communication.

---

# 3. Fundamental Dependency Rule

The most important rule is:

```text
Delivery
   ↓
Application
   ↓
Domain
```

Infrastructure exists at the outer boundary and implements contracts required by the inner layers.

Therefore:

```text
Domain
  ← Application
  ← Infrastructure
  ← Delivery
```

but:

```text
Domain ──X──> Application
Domain ──X──> Infrastructure
Domain ──X──> Delivery
```

The Domain is the dependency center of business meaning.

---

# 4. Dependency Direction

## 4.1 Allowed Dependencies

The following dependencies are allowed:

```text
Delivery → Application
Application → Domain
Application → Application.Shared
Domain → Domain.Shared
Infrastructure → Application.Ports
Infrastructure → Domain.Ports
Infrastructure → Domain
Infrastructure → Application
```

The Infrastructure dependencies exist only to implement or adapt contracts and must not cause reverse dependencies.

---

## 4.2 Forbidden Dependencies

The following are prohibited:

```text
Domain → Application
Domain → Infrastructure
Domain → Delivery

Application → Delivery

Domain → Framework
Domain → ORM
Domain → Database
Domain → HTTP Client

Application → Concrete Database Adapter
Application → Concrete Message Broker
Application → Concrete External API Client
```

These restrictions are architectural constraints, not merely coding preferences.

---

# 5. Component Interaction Categories

CollectionHub recognizes four principal interaction categories:

1. synchronous command interaction;
2. synchronous query interaction;
3. domain event interaction;
4. infrastructure/integration interaction.

Each category has different coupling characteristics.

---

# 6. Synchronous Command Interaction

A command represents an explicit intention to modify system state.

Typical flow:

```text
Client
  │
  ▼
Controller / Handler
  │
  ▼
Application Command
  │
  ▼
Application Use Case
  │
  ▼
Aggregate
  │
  ▼
Domain Behavior
```

Example:

```text
POST /items
     │
     ▼
CreateItemCommand
     │
     ▼
CreateItemUseCase
     │
     ▼
Item.create(...)
     │
     ▼
ItemCreated
```

The Delivery layer must not directly mutate aggregates.

---

# 7. Synchronous Query Interaction

Queries follow a different path because they do not modify domain state.

Typical flow:

```text
Client
  │
  ▼
Delivery
  │
  ▼
Application Query
  │
  ▼
Query Handler
  │
  ├── Read Port
  │
  ▼
Read Model / Repository
```

A query does not need to load a complete aggregate when the use case only requires read-oriented information.

This allows optimized read models without compromising domain boundaries.

---

# 8. Command and Query Separation

The architecture should distinguish:

```text
Commands → change state
Queries  → retrieve state
```

Commands may invoke domain behavior.

Queries should normally avoid invoking domain mutation behavior.

Conceptually:

```text
Command
   ↓
Aggregate
   ↓
State change

Query
   ↓
Read model
   ↓
Result
```

This distinction should remain visible in the Application module structure.

---

# 9. Aggregate Interaction Rules

Aggregates are consistency boundaries.

An aggregate may directly enforce invariants over the state it owns.

It must not arbitrarily modify another aggregate.

Avoid:

```text
Item
 └── modifies Collection
      └── modifies Classification
           └── modifies Valuation
```

Prefer:

```text
Application Workflow
      │
      ├── Item aggregate
      │
      ├── Collection aggregate
      │
      └── Classification aggregate
```

The Application layer coordinates cross-aggregate operations.

---

# 10. Cross-Aggregate References

Cross-aggregate references should normally be represented through identity.

Preferred:

```text
Item
 └── collectionId
```

rather than:

```text
Item
 └── Collection aggregate
      └── complete object graph
```

This prevents accidental aggregate coupling and uncontrolled transactional boundaries.

---

# 11. Cross-Aggregate Operations

When an operation requires multiple aggregates, the Application layer coordinates the interaction.

Example:

```text
TransferItem
    │
    ├── load Item
    ├── load SourceCollection
    ├── load TargetCollection
    │
    ├── invoke Item behavior
    ├── invoke SourceCollection behavior
    └── invoke TargetCollection behavior
```

The Application workflow owns orchestration.

The Domain owns the business rules executed by each aggregate.

---

# 12. Domain Services

A Domain Service may coordinate domain concepts when behavior does not naturally belong to a single aggregate.

Example:

```text
Application Use Case
        │
        ▼
Domain Service
        │
        ├── Aggregate A
        └── Aggregate B
```

A Domain Service must remain free from:

- HTTP;
- persistence implementation;
- UI concerns;
- framework APIs;
- message broker APIs.

It must express domain behavior.

---

# 13. Application Services

Application Services coordinate use cases.

Typical responsibility:

```text
Receive command
      ↓
Validate application-level input
      ↓
Load aggregates
      ↓
Invoke domain behavior
      ↓
Persist changes
      ↓
Publish events
      ↓
Return result
```

They should not contain large blocks of business decision logic.

If a rule determines whether an operation is valid according to CollectionHub's business semantics, it should normally live in the Domain.

---

# 14. Repository Interaction

Repositories are accessed through abstractions.

Preferred:

```text
Application
    │
    ▼
ItemRepository
    ▲
    │
Infrastructure
    │
    ▼
SqlItemRepository
```

Not:

```text
Application
    │
    ▼
SqlItemRepository
```

The concrete persistence technology must remain replaceable.

---

# 15. Repository Responsibility

Repositories are responsible for retrieving and persisting aggregates or appropriate domain persistence boundaries.

Repositories must not become:

```text
BusinessRuleRepository
CalculationService
WorkflowManager
ValidationEngine
```

Their primary responsibility is persistence-oriented access.

Business decisions belong elsewhere.

---

# 16. Unit of Work and Transaction Boundary

A transaction boundary should normally align with an Application use case.

Conceptually:

```text
Use Case
   │
   ├── load aggregate(s)
   ├── execute domain behavior
   ├── persist changes
   └── commit
```

The architecture should avoid exposing transaction management directly to the Domain.

The Domain must not know whether persistence uses:

- SQL transactions;
- document transactions;
- distributed transactions;
- event sourcing;
- another persistence mechanism.

---

# 17. Domain Event Interaction

Domain events represent facts produced by domain behavior.

Example:

```text
Item
 │
 └── dispose()
       │
       ▼
 ItemDisposed
```

The event may then be consumed by Application or Infrastructure components.

```text
ItemDisposed
      │
      ├── update search projection
      ├── update reporting model
      ├── publish integration event
      └── trigger another application workflow
```

The aggregate itself should not perform these side effects.

---

# 18. Domain Events vs Integration Events

These concepts must remain separate.

### Domain Event

Represents a fact meaningful inside CollectionHub's domain model.

Example:

```text
ItemDisposed
ValuationRecorded
AcquisitionRecorded
ClassificationChanged
```

### Integration Event

Represents a fact published to an external system or bounded context.

Example:

```text
CollectionItemDisposed
CollectionItemValuated
```

Integration events may be derived from domain events.

They should not force external transport concerns into the Domain.

---

# 19. Event Flow

Preferred event flow:

```text
Aggregate
    │
    ▼
Domain Event
    │
    ▼
Application Event Dispatcher
    │
    ├── Application Handler
    ├── Projection Handler
    └── Integration Publisher
```

Not:

```text
Aggregate
    │
    ├── Kafka
    ├── RabbitMQ
    ├── HTTP
    └── Email
```

The latter would couple business behavior directly to infrastructure.

---

# 20. Eventual Consistency

Some cross-component reactions may be eventually consistent.

For example:

```text
Item updated
     │
     ▼
ItemUpdated
     │
     ├── transaction committed
     │
     └── search index updated asynchronously
```

The authoritative business state remains the transactional domain state.

Derived projections must not become authoritative merely because they are optimized for queries.

---

# 21. Infrastructure Interaction

Infrastructure components may communicate with:

- databases;
- object storage;
- search engines;
- external APIs;
- message brokers;
- file systems;
- monitoring systems.

These interactions must remain outside the Domain.

Example:

```text
Application
     │
     ▼
MediaStoragePort
     ▲
     │
     ▼
ObjectStorageAdapter
```

---

# 22. External Service Interaction

External services must be hidden behind explicit ports.

Preferred:

```text
Application
    │
    ▼
MarketValuationPort
    ▲
    │
    ▼
ExternalMarketAdapter
```

Not:

```text
ValuationService
    │
    ▼
HttpClient
    │
    ▼
External API
```

The latter would embed infrastructure concerns into business behavior.

---

# 23. Time Dependency

Business logic that depends on current time must not directly call system time APIs.

Preferred:

```text
Domain/Application
       │
       ▼
Clock
       ▲
       │
SystemClock
```

This enables:

- deterministic tests;
- reproducible business behavior;
- explicit temporal rules.

---

# 24. Identity Generation

Identity generation should follow the same principle.

Preferred:

```text
Application
    │
    ▼
IdGenerator
    ▲
    │
Infrastructure
```

or, where appropriate, deterministic domain identity creation.

The Domain must not become coupled to a concrete UUID library or infrastructure-specific identity provider merely for convenience.

---

# 25. Validation Interaction

Validation occurs at different levels.

```text
Delivery validation
        ↓
Application validation
        ↓
Domain invariants
```

Each level has a different purpose.

### Delivery validation

Checks transport correctness.

Examples:

- malformed JSON;
- missing HTTP field;
- invalid request encoding.

### Application validation

Checks use-case input.

Examples:

- referenced identifier missing;
- unsupported command combination;
- authorization requirement.

### Domain validation

Checks business invariants.

Examples:

- item cannot be disposed in current state;
- valuation cannot use an invalid date;
- classification is not permitted.

Domain invariants must remain authoritative.

---

# 26. Authorization Interaction

Authorization is primarily an Application/Delivery concern.

The Domain should not depend on:

```text
User
Role
JWT
HTTP Session
OAuth Token
```

However, the Domain may receive domain-relevant concepts when authorization affects a genuine business rule.

Example:

```text
Application
    │
    ├── determines actor
    │
    ▼
Domain operation
    │
    └── evaluates business constraint
```

This keeps technical identity mechanisms outside the Domain.

---

# 27. Error Propagation

Errors should flow outward without exposing infrastructure details unnecessarily.

Example:

```text
Domain Error
    ↓
Application Error Mapping
    ↓
Delivery Error
```

An infrastructure exception such as:

```text
SqlConnectionTimeout
```

must not leak directly into an API contract.

Instead:

```text
Infrastructure Failure
    ↓
Application Boundary
    ↓
Appropriate Application/Transport Error
```

---

# 28. Dependency Injection

Dependency injection may be used at composition boundaries.

The Domain itself should not construct Infrastructure services.

Bad:

```text
ItemService
    └── new SqlRepository()
```

Preferred:

```text
Composition Root
    │
    ├── SqlItemRepository
    ├── EventPublisher
    ├── Clock
    └── UseCase
```

The composition root is responsible for assembling the application.

---

# 29. Module-to-Module Interaction

Within the Domain, modules should communicate through explicit concepts.

Example:

```text
Item
 │
 ├── ClassificationId
 ├── CollectionId
 └── ProvenanceId
```

Within Application:

```text
Item Use Case
      │
      ├── ItemRepository
      ├── CollectionRepository
      └── ClassificationRepository
```

This makes cross-module interaction visible and reviewable.

---

# 30. Shared Module Restrictions

`shared` modules are subject to strict rules.

A component may move into `shared` only if:

1. it represents a genuinely shared concept;
2. it has stable semantics;
3. it is used by multiple modules;
4. its ownership cannot reasonably belong to one business capability.

Avoid using:

```text
shared/
    Utils
    Helpers
    CommonService
    GenericManager
```

as dumping grounds.

---

# 31. Circular Dependency Prevention

Circular dependencies are prohibited.

Forbidden:

```text
Collection
   ↓
Item
   ↓
Collection
```

Also forbidden:

```text
Application.Items
   ↓
Application.Collections
   ↓
Application.Items
```

If a circular relationship emerges, introduce an explicit orchestration boundary or reconsider the module ownership.

---

# 32. Dependency Inversion Rule

When a higher-level component requires functionality supplied by a lower-level technical component, the abstraction must belong to the higher-level architectural boundary.

Example:

```text
Application
    │
    ▼
ItemRepository
    ▲
    │
Infrastructure
```

The abstraction expresses what the Application needs.

Infrastructure merely provides an implementation.

This preserves dependency inversion.

---

# 33. Component Interaction Matrix

| From | To | Allowed | Interaction |
|---|---|---:|---|
| Delivery | Application | ✓ | Commands / Queries |
| Delivery | Domain | ✗ | Never direct |
| Delivery | Infrastructure | Restricted | Through composition/adapters |
| Application | Domain | ✓ | Use cases / domain behavior |
| Application | Infrastructure | Indirect | Through ports |
| Domain | Application | ✗ | Never |
| Domain | Infrastructure | ✗ | Never |
| Domain | Delivery | ✗ | Never |
| Infrastructure | Application Ports | ✓ | Implement |
| Infrastructure | Domain Ports | ✓ | Implement |
| Infrastructure | External Systems | ✓ | Adapters |

---

# 34. Interaction Sequence — Command

A canonical CollectionHub command should follow this sequence:

```text
1. Client
      ↓
2. Delivery Adapter
      ↓
3. Application Command
      ↓
4. Application Use Case
      ↓
5. Repository / Port
      ↓
6. Aggregate
      ↓
7. Domain Behavior
      ↓
8. Domain Event
      ↓
9. Persistence
      ↓
10. Commit
      ↓
11. Event Handling
      ↓
12. Application Result
      ↓
13. Delivery Response
```

The exact ordering of event dispatch and persistence will be refined by the transaction/eventing architecture.

---

# 35. Interaction Sequence — Query

A canonical query should follow:

```text
1. Client
      ↓
2. Delivery Adapter
      ↓
3. Application Query
      ↓
4. Query Handler
      ↓
5. Read Port
      ↓
6. Read Model / Repository
      ↓
7. Application Result
      ↓
8. Delivery Response
```

No domain mutation should occur.

---

# 36. Interaction Sequence — Cross-Aggregate Workflow

Example:

```text
TransferItem
      │
      ▼
Application Workflow
      │
      ├── load Item
      ├── load Source Collection
      ├── load Target Collection
      │
      ├── Item.transfer()
      ├── SourceCollection.removeItem()
      └── TargetCollection.addItem()
              │
              ▼
         Domain Events
              │
              ▼
         Commit / Handlers
```

The workflow coordinates.

The aggregates decide whether their own operations are valid.

---

# 37. Interaction Sequence — External Integration

Example:

```text
Application Use Case
        │
        ▼
Domain Behavior
        │
        ▼
Domain Event
        │
        ▼
Application Handler
        │
        ▼
Integration Port
        │
        ▼
Infrastructure Adapter
        │
        ▼
External System
```

The external system must never become a direct dependency of the aggregate.

---

# 38. Synchronous vs Asynchronous Interaction

Use synchronous interaction when:

- immediate consistency is required;
- the result is required to complete the use case;
- the dependency is part of the current transaction boundary.

Use asynchronous interaction when:

- the operation can be eventually consistent;
- the work is expensive;
- the consumer is independently scalable;
- the operation is a side effect rather than core transactional behavior.

The architecture must not introduce asynchronous processing merely to make the design appear distributed.

---

# 39. Transactional Consistency Rule

A single business invariant that must hold immediately must be enforced inside the same consistency boundary.

If enforcing the rule requires multiple aggregates, reconsider:

1. aggregate boundaries;
2. domain service design;
3. application workflow;
4. eventual consistency requirements.

Do not solve an incorrect aggregate boundary by introducing distributed transactions by default.

---

# 40. Side-Effect Isolation

Business operations should produce explicit facts.

Side effects should react to those facts.

Preferred:

```text
Business operation
      ↓
Domain state change
      ↓
Domain event
      ↓
Side effect
```

Avoid:

```text
Business operation
      ↓
Database
      ↓
Email
      ↓
Search
      ↓
External API
      ↓
More business logic
```

The latter creates hidden coupling and makes transactional behavior difficult to reason about.

---

# 41. Read Model Independence

Read models may be optimized independently from the domain model.

For example:

```text
Domain Model
    │
    ▼
Domain Event
    │
    ▼
Projection
    │
    ▼
Search / Read Model
```

A read model may denormalize information for efficient queries.

It must not become the source of truth for transactional domain invariants.

---

# 42. Search Interaction

Search should preferably interact through an Application query boundary.

Example:

```text
SearchItems
    │
    ▼
SearchQueryService
    │
    ▼
SearchPort
    │
    ▼
Search Infrastructure
```

Domain specifications may contribute business semantics to the query, but the Domain must not know the concrete search engine.

---

# 43. Media Interaction

Media metadata may belong to the Domain.

Binary storage does not.

Preferred:

```text
Item
 │
 └── MediaReference
          │
          ▼
   MediaStoragePort
          │
          ▼
   Object Storage
```

The Domain knows that a media resource exists.

It does not know how or where its bytes are physically stored.

---

# 44. Logging and Observability

Logging, tracing and metrics are infrastructure concerns.

The Domain should not directly invoke:

```text
Logger
Tracer
MetricsClient
```

If observability requires business facts, Domain Events or Application Events should provide the necessary semantic information.

---

# 45. Configuration

Configuration must not leak into the Domain.

Avoid:

```text
DomainRule
    └── EnvironmentVariable
```

Instead:

```text
Configuration
    ↓
Application Composition
    ↓
Domain Policy / Value
```

Business policies that require configurable parameters should receive those parameters explicitly.

---

# 46. Feature Flags

Feature flags are generally Application/Delivery concerns.

The Domain should not depend directly on a feature flag SDK.

If a business rule genuinely changes based on a policy, provide the Domain with an explicit domain policy rather than a technical feature flag mechanism.

---

# 47. Architectural Invariants

The following rules are considered architectural invariants:

### A1 — Domain isolation

The Domain cannot depend on Infrastructure or Delivery.

### A2 — Application orchestration

Application coordinates use cases and cross-aggregate workflows.

### A3 — Aggregate integrity

Aggregates protect their own invariants.

### A4 — No direct cross-aggregate mutation

One aggregate cannot directly mutate another aggregate.

### A5 — Port-based infrastructure

Infrastructure dependencies are accessed through explicit contracts.

### A6 — Domain event isolation

Domain events represent business facts, not technical commands.

### A7 — Query separation

Read operations do not implicitly perform domain mutations.

### A8 — No circular module dependencies

Modules must form an acyclic dependency graph.

### A9 — Shared minimization

Shared modules contain only genuinely shared concepts.

### A10 — Delivery isolation

Delivery mechanisms invoke Application use cases rather than domain behavior directly.

---

# 48. Architecture Enforcement

These rules should eventually be enforceable through automated architecture tests.

Potential checks include:

```text
Domain must not reference Infrastructure
Domain must not reference Delivery
Domain must not reference Application
Application must not reference Delivery
Application must not reference concrete Infrastructure adapters
Modules must not contain circular dependencies
Aggregates must not directly depend on other aggregate implementations
Shared modules must not depend on business-specific modules
```

The exact mechanism will depend on the implementation technology selected later.

---

# 49. Dependency Graph Target

The final dependency graph should remain approximately:

```text
                         Delivery
                            │
                            ▼
                       Application
                            │
                            ▼
                          Domain
                            ▲
                            │
                     Domain/Application
                         Contracts
                            ▲
                            │
                     Infrastructure
```

The arrows represent dependency direction.

The important characteristic is that **Infrastructure points inward through contracts**, rather than becoming a dependency of the core.

---

# 50. Architectural Review Checklist

Before accepting an implementation component, verify:

- [ ] Does the component have a clearly identified architectural owner?
- [ ] Does it belong to a business capability?
- [ ] Does its dependency direction respect the layer boundaries?
- [ ] Does it introduce a dependency toward Infrastructure?
- [ ] Does it introduce a dependency toward Delivery?
- [ ] Does it bypass the Application layer?
- [ ] Does it bypass an aggregate boundary?
- [ ] Does it introduce a circular dependency?
- [ ] Does it contain business rules that belong in the Domain?
- [ ] Does it introduce unnecessary shared code?
- [ ] Does it create an implicit transaction boundary?
- [ ] Does it create hidden side effects?
- [ ] Could the dependency be expressed through a port?
- [ ] Could the interaction be represented more explicitly through a domain event?

---

# 51. Architectural Decision

CollectionHub adopts **explicit component interaction rules based on dependency inversion, aggregate boundaries, application orchestration and event-driven side-effect isolation**.

The architecture therefore establishes the following fundamental model:

```text
                 ┌──────────────┐
                 │   Delivery   │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │ Application  │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │    Domain    │
                 └──────────────┘
                        ▲
                        │
                 ┌──────┴───────┐
                 │    Ports     │
                 └──────┬───────┘
                        ▲
                        │
                 ┌──────┴───────┐
                 │Infrastructure│
                 └──────────────┘
```

The essential architectural rule is:

> **Business behavior flows through the Domain, use-case orchestration flows through the Application layer, and technical concerns remain at the architectural perimeter.**

This establishes the interaction contract required before mapping the concrete CollectionHub use cases onto the architectural components.