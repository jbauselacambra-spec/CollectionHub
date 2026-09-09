# 29 — Infrastructure Components and Adapters

## 1. Purpose

This document defines the Infrastructure architecture required by CollectionHub based on the domain, application and interaction models established in the previous architecture documents.

The objective is to identify:

- infrastructure components;
- adapters;
- persistence mechanisms;
- repository implementations;
- transaction management;
- event publication;
- event consumption;
- search infrastructure;
- media storage;
- external service integrations;
- identity and time providers;
- observability;
- configuration;
- infrastructure composition;
- infrastructure dependency rules.

This document deliberately defines **architectural responsibilities rather than concrete technologies**.

The technology selection must follow the architectural requirements established here, not redefine them.

---

# 2. Infrastructure Responsibility

Infrastructure provides concrete implementations for capabilities required by the inner layers.

Conceptually:

```text id="jv7r0e"
                    ┌───────────────────────┐
                    │       Delivery        │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │      Application      │
                    │                       │
                    │ Commands / Queries    │
                    │ Ports / Workflows     │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │        Domain         │
                    │                       │
                    │ Aggregates / Rules    │
                    │ Events / Policies     │
                    └───────────────────────┘
                                ▲
                                │
                         Contracts / Ports
                                │
                                │
                    ┌───────────┴───────────┐
                    │    Infrastructure     │
                    │                       │
                    │ Persistence           │
                    │ Messaging             │
                    │ Search                │
                    │ Media                 │
                    │ External APIs         │
                    │ Observability         │
                    └───────────────────────┘
```

Infrastructure is therefore the **outer technical boundary** around the application and domain core.

---

# 3. Fundamental Principle

The primary rule is:

> **Infrastructure implements technical capabilities; it does not define business behavior.**

Infrastructure may determine:

- how data is stored;
- how messages are transported;
- how files are stored;
- how external APIs are called;
- how logs are emitted;
- how metrics are collected.

Infrastructure must not determine:

- whether an item can be disposed;
- whether a classification is valid;
- whether an acquisition is allowed;
- whether a valuation is semantically valid;
- whether a collection operation violates a domain invariant.

Those decisions belong to the Domain.

---

# 4. Infrastructure Component Map

The initial infrastructure architecture is:

```text id="5ppkxr"
infrastructure/
│
├── persistence/
│   ├── repositories/
│   ├── mappings/
│   ├── models/
│   ├── transactions/
│   └── migrations/
│
├── messaging/
│   ├── publishers/
│   ├── consumers/
│   ├── serializers/
│   └── handlers/
│
├── search/
│   ├── index/
│   ├── queries/
│   ├── projections/
│   └── adapters/
│
├── media/
│   ├── storage/
│   ├── metadata/
│   └── adapters/
│
├── external/
│   ├── valuation/
│   ├── synchronization/
│   └── adapters/
│
├── identity/
│   └── providers/
│
├── time/
│   └── providers/
│
├── observability/
│   ├── logging/
│   ├── tracing/
│   └── metrics/
│
├── configuration/
│
└── composition/
```

This structure is organized by infrastructure capability rather than by a single technical framework.

---

# 5. Persistence Infrastructure

Persistence is responsible for storing authoritative transactional domain state.

Its responsibilities include:

- database connectivity;
- repository implementations;
- persistence mapping;
- transaction handling;
- concurrency handling;
- migrations;
- database-specific optimization.

It must not leak persistence concerns into the Domain.

---

# 6. Repository Adapters

Repository interfaces are defined by Application and/or Domain boundaries.

Infrastructure provides their concrete implementations.

Example:

```text id="m6u1t8"
Application
    │
    ▼
ItemRepository
    ▲
    │
    │ implements
    │
SqlItemRepository
    │
    ▼
Database
```

The adapter is responsible for translating between:

```text id="9c0y9n"
Domain Model
     ↕
Persistence Model
```

The two models do not have to be structurally identical.

---

# 7. Repository Components

The initial repository adapter set is:

```text id="qcv0o4"
CollectionRepositoryAdapter
ItemRepositoryAdapter
ClassificationRepositoryAdapter
AcquisitionRepositoryAdapter
ValuationRepositoryAdapter
DisposalRepositoryAdapter
ProvenanceRepositoryAdapter
MediaRepositoryAdapter
```

The final list must be reconciled with the final aggregate boundaries.

A repository should normally correspond to an aggregate or persistence boundary rather than to every database table.

---

# 8. Persistence Models

Infrastructure may define persistence-specific models.

Example:

```text id="a4n0gq"
Domain:
    Item

Infrastructure:
    ItemPersistenceModel
```

The persistence model may contain:

- database identifiers;
- foreign keys;
- indexes;
- denormalized fields;
- technical timestamps;
- optimistic concurrency version;
- persistence metadata.

These should not automatically become Domain concepts.

---

# 9. Persistence Mapping

Mapping occurs at the Infrastructure boundary.

Conceptually:

```text id="w2h0p8"
Domain Entity
      │
      ▼
Repository Adapter
      │
      ▼
Persistence Mapper
      │
      ▼
Persistence Model
```

Reverse mapping:

```text id="o5rjxi"
Persistence Model
      │
      ▼
Persistence Mapper
      │
      ▼
Domain Entity
```

The mapper is responsible for preserving the semantics of the Domain model.

---

# 10. Persistence Source of Truth

The transactional persistence store is the authoritative source for domain state.

Derived stores such as:

- search indexes;
- reporting databases;
- caches;
- projections;

must not become authoritative for transactional business decisions.

Conceptually:

```text id="j8q3m4"
Transactional Domain State
          │
          ├── Search Projection
          ├── Reporting Projection
          └── Cache
```

not:

```text id="4h4lkk"
Search Index
    ↓
Domain Truth
```

---

# 11. Transaction Infrastructure

The Infrastructure layer provides transaction management required by Application use cases.

Conceptual contract:

```text id="k6ak2o"
UnitOfWork
```

Implementation:

```text id="1h70qf"
DatabaseUnitOfWork
```

Typical flow:

```text id="9c2v4r"
Use Case
   │
   ▼
Begin Transaction
   │
   ├── Load aggregate
   ├── Execute domain behavior
   ├── Persist changes
   └── Register events
   │
   ▼
Commit
```

The Domain must not know about transactions.

---

# 12. Transaction Boundary

The default transaction boundary is an Application use case.

Example:

```text id="qfjw3h"
RecordValuation
      │
      ├── load Item
      ├── execute valuation behavior
      ├── persist
      └── commit
```

Cross-aggregate workflows require explicit transaction analysis.

The architecture does not assume that every workflow requires distributed transactions.

---

# 13. Concurrency Control

Persistence must support the concurrency guarantees required by aggregate consistency.

The preferred initial strategy should be evaluated around:

```text id="i0t9i8"
Optimistic Concurrency
        │
        └── Aggregate Version
```

Potential persistence mechanism:

```text id="zq1fbr"
Item
 ├── id
 └── version
```

On update:

```text id="5ylwlf"
UPDATE ...
WHERE id = ?
AND version = ?
```

If the version does not match, the infrastructure reports a concurrency conflict.

The exact mechanism remains a technology decision.

---

# 14. Database Migrations

Database schema evolution belongs to Infrastructure.

Migrations must be:

- versioned;
- reproducible;
- environment-independent;
- backward-compatible where required;
- reviewed alongside domain/application changes.

The Domain must not contain migration logic.

---

# 15. Messaging Infrastructure

Messaging infrastructure handles communication through asynchronous transports.

Responsibilities include:

- event publication;
- event consumption;
- serialization;
- deserialization;
- delivery retries;
- dead-letter handling;
- message correlation;
- idempotency.

The messaging infrastructure must not alter domain semantics.

---

# 16. Event Publisher

The Application layer may depend on an abstraction such as:

```text id="5e6v1z"
EventPublisher
```

Infrastructure implements it:

```text id="o8prn6"
MessageBrokerEventPublisher
```

Flow:

```text id="oh5zrd"
Domain Event
      │
      ▼
Application Event Handling
      │
      ▼
EventPublisher
      │
      ▼
Message Broker
```

The aggregate should not know which broker is being used.

---

# 17. Event Serialization

Domain events should not automatically become transport messages without an explicit boundary.

Recommended flow:

```text id="0h6v4m"
Domain Event
      │
      ▼
Integration Event Mapper
      │
      ▼
Integration Event
      │
      ▼
Serializer
      │
      ▼
Message Broker
```

This allows the internal domain event model to evolve independently from external contracts.

---

# 18. Event Consumers

Infrastructure consumers receive external or asynchronous messages.

Typical flow:

```text id="umz8po"
Message Broker
      │
      ▼
Consumer
      │
      ▼
Deserializer
      │
      ▼
Application Handler
```

The consumer should remain thin.

Business behavior should execute through Application and Domain components.

---

# 19. Event Idempotency

Consumers must assume that messages can be delivered more than once.

Therefore:

```text id="qk4xsp"
Message
   │
   ▼
Idempotency Check
   │
   ├── already processed → ignore
   │
   └── new → process
```

The exact mechanism may use:

- message identifiers;
- processed-message records;
- version checks;
- natural business idempotency.

The chosen mechanism belongs to Infrastructure/Application.

---

# 20. Outbox Consideration

The architecture should strongly consider the Outbox pattern where domain state changes and event publication must remain reliable.

Conceptually:

```text id="fckj8q"
Application Transaction
        │
        ├── Domain State
        │
        └── Outbox Event
                │
                ▼
             Commit
                │
                ▼
        Outbox Dispatcher
                │
                ▼
         Message Broker
```

This prevents the failure scenario:

```text id="b6m5v6"
Database commit succeeds
       ↓
Message publication fails
       ↓
Event lost
```

The final adoption of Outbox should be decided during persistence and messaging design.

---

# 21. Search Infrastructure

Search is a read-optimized infrastructure capability.

The initial architecture is:

```text id="2u7yph"
Domain Events
      │
      ▼
Search Projection Handler
      │
      ▼
Search Index
      │
      ▼
Search Adapter
      │
      ▼
Application Query
```

Search infrastructure should support:

- item search;
- classification filtering;
- valuation filtering;
- collection filtering;
- text search;
- sorting;
- pagination;
- faceting where required.

---

# 22. Search Projection

The search index is a projection of transactional state.

Example:

```text id="7w0r1k"
Item
 ├── identity
 ├── title
 ├── classification
 ├── collection
 ├── current valuation
 ├── lifecycle state
 └── searchable metadata
```

The projection may intentionally denormalize these values.

---

# 23. Search Consistency

Search should normally be considered eventually consistent.

Example:

```text id="k8j8g7"
ItemUpdated
    │
    ▼
Transaction committed
    │
    ▼
Search Projection
    │
    ▼
Index updated
```

A short delay between transactional state and search visibility is acceptable if the use case permits it.

If immediate consistency is required for a particular operation, that requirement must be explicitly identified.

---

# 24. Media Infrastructure

Media infrastructure handles binary content and storage.

Responsibilities include:

- upload;
- retrieval;
- deletion;
- storage addressing;
- metadata persistence;
- content validation;
- storage lifecycle;
- optional thumbnail generation.

The Domain should know about media references, not storage implementation.

---

# 25. Media Storage Port

Conceptual contract:

```text id="9l4jym"
MediaStoragePort
```

Possible implementations:

```text id="hff8qa"
LocalFileStorageAdapter
ObjectStorageAdapter
CloudStorageAdapter
```

The Domain/Application layers remain independent from the concrete storage technology.

---

# 26. Media Metadata

Media metadata should be split carefully.

Domain-level metadata:

```text id="l8g6sl"
MediaId
MediaType
MediaRole
DisplayOrder
Association
```

Infrastructure metadata:

```text id="q6ak3e"
StorageKey
Bucket
Checksum
PhysicalPath
StorageProvider
CompressionFormat
```

The latter must not leak into the Domain unless it has genuine business meaning.

---

# 27. External Service Infrastructure

External integrations should be isolated under:

```text id="5r6qzo"
infrastructure/external/
```

Examples may eventually include:

```text id="6lknz8"
Valuation Provider
Metadata Provider
Collection Synchronization Provider
Image Processing Provider
External Catalog Provider
```

Each integration requires an explicit port and adapter.

---

# 28. External Service Adapter Pattern

Preferred structure:

```text id="1u7qjj"
Application / Domain Port
          ▲
          │
          │ implements
          │
ExternalServiceAdapter
          │
          ▼
External API
```

The adapter is responsible for:

- HTTP;
- authentication;
- retries;
- timeout handling;
- serialization;
- provider-specific errors;
- provider-specific data mapping.

The Domain must not know these details.

---

# 29. External Service Failure

External services are unreliable by nature.

Infrastructure should support:

- timeout;
- retry;
- circuit breaking where justified;
- rate limiting;
- fallback;
- error mapping.

The Domain should receive a meaningful result or domain-relevant failure rather than a raw HTTP exception.

---

# 30. Clock Provider

Time-dependent behavior should use a clock abstraction.

Conceptually:

```text id="rrx2xj"
Clock
  ▲
  │
SystemClock
```

Infrastructure provides the real implementation.

Tests may provide:

```text id="f4fh2f"
FixedClock
```

This allows deterministic testing of:

- acquisition dates;
- valuation dates;
- disposal dates;
- lifecycle rules;
- expiration;
- temporal validity.

---

# 31. Identity Provider

Identity generation may use:

```text id="j3j1hx"
IdGenerator
```

Infrastructure can provide:

```text id="az8f8v"
UuidGenerator
```

or another implementation.

The application and domain layers should not depend on the concrete random/UUID library.

---

# 32. Authentication and Authorization Adapters

Authentication belongs primarily to Delivery and Infrastructure.

Potential architecture:

```text id="f8f40h"
Request
  │
  ▼
Authentication Adapter
  │
  ▼
Authenticated Actor
  │
  ▼
Application Use Case
```

Infrastructure may integrate with:

- identity providers;
- token verification;
- external authentication systems.

The Domain should not depend on the authentication provider.

---

# 33. Configuration Infrastructure

Configuration should be centralized at the composition boundary.

Potential categories:

```text id="nq3m4m"
Database
Messaging
Search
Media Storage
External Services
Authentication
Observability
Runtime
```

The composition root transforms configuration into typed dependencies.

Avoid accessing environment variables throughout the application.

Bad:

```text id="n5lqkb"
Domain
  └── getenv(...)
```

Preferred:

```text id="pdbxpf"
Environment
    ↓
Configuration
    ↓
Composition Root
    ↓
Dependencies
```

---

# 34. Observability Infrastructure

Infrastructure provides technical observability:

```text id="3d7n6m"
Logging
Tracing
Metrics
Health Checks
```

These mechanisms should remain outside the Domain.

---

# 35. Logging

Application boundaries should provide structured logging around:

- use-case execution;
- failures;
- retries;
- external integrations;
- asynchronous processing.

Logs should contain correlation information where available.

Domain entities should not directly emit infrastructure logs.

---

# 36. Distributed Tracing

Tracing should follow the main execution path:

```text id="ykh6qj"
Request
  ↓
Application Use Case
  ↓
Repository
  ↓
Database
  ↓
Event Publisher
  ↓
External Consumer
```

Trace propagation should be implemented by Infrastructure/Delivery.

---

# 37. Metrics

Potential infrastructure/application metrics include:

```text id="3y5vqm"
use_case.execution.duration
use_case.failure.count

repository.operation.duration
repository.failure.count

event.publish.success
event.publish.failure

event.consumer.retry
event.consumer.dead_letter

search.query.duration

external_api.latency
external_api.failure
```

Metrics naming remains implementation-specific but should preserve semantic consistency.

---

# 38. Health Checks

Infrastructure should expose health information for:

- database;
- message broker;
- search;
- media storage;
- required external services.

Health checks must distinguish:

```text id="8d9gqk"
Application availability
vs.
Optional dependency availability
```

An optional external integration being unavailable should not necessarily make the entire application unhealthy.

---

# 39. Infrastructure Composition Root

The composition root is responsible for assembling concrete implementations.

Conceptually:

```text id="w8lqxt"
Composition Root
      │
      ├── Database
      ├── Repositories
      ├── UnitOfWork
      ├── EventPublisher
      ├── SearchAdapter
      ├── MediaStorage
      ├── External Adapters
      ├── Clock
      └── IdGenerator
             │
             ▼
        Application
```

This is where dependency injection should be configured.

---

# 40. Adapter Naming

Adapters should make their technical role explicit.

Examples:

```text id="g0b1if"
SqlItemRepository
PostgresItemRepository

SearchEngineAdapter
ObjectStorageMediaAdapter

ExternalValuationProvider
MessageBrokerEventPublisher
SystemClock
UuidGenerator
```

Avoid generic names such as:

```text id="t4w3uy"
Helper
Manager
Service
Utility
Handler
```

unless the role is genuinely generic and unambiguous.

---

# 41. Infrastructure Module Boundaries

Infrastructure should be divided by capability.

Preferred:

```text id="smj9tj"
persistence/
messaging/
search/
media/
external/
identity/
time/
observability/
configuration/
composition/
```

Avoid a single:

```text id="6d8gjp"
infrastructure/services/
```

directory containing unrelated technical behavior.

---

# 42. Infrastructure Dependency Rules

Infrastructure may depend on:

```text id="e17w8m"
Application contracts
Domain contracts
Domain models
Infrastructure libraries
External SDKs
Database drivers
Messaging libraries
Storage SDKs
```

Infrastructure must not modify the semantic meaning of Domain concepts merely to fit a technical framework.

---

# 43. Infrastructure Must Not Become a Second Application Layer

Avoid placing business decisions in:

```text id="y1u2ph"
Repository
Adapter
Consumer
Projection
External Client
```

For example:

```text id="2wjz9d"
Repository:
    if item.canBeDisposed():
        ...
```

is acceptable only when the call is part of persistence orchestration.

The repository must not independently decide:

```text id="9g8t1h"
"Items over X value cannot be disposed."
```

if that is a business rule.

Such logic belongs in the Domain.

---

# 44. Adapter Isolation

An adapter should isolate provider-specific concepts.

For example:

```text id="kcr1hp"
External Valuation API
       │
       ▼
Provider DTO
       │
       ▼
Valuation Adapter
       │
       ▼
Domain/Application Model
```

Provider-specific DTOs must not leak into the Application or Domain.

---

# 45. Persistence Anti-Corruption Boundary

Persistence may require an anti-corruption mapping layer when database semantics differ substantially from the Domain.

Example:

```text id="y3r2ri"
Database Schema
      │
      ▼
Persistence Model
      │
      ▼
Mapper
      │
      ▼
Domain Model
```

This prevents legacy or optimized database structures from defining the business model.

---

# 46. Legacy Integration

If CollectionHub must integrate with legacy systems, each legacy system should receive its own adapter boundary.

```text id="o8n5ad"
CollectionHub Port
       ▲
       │
Legacy Adapter
       │
       ▼
Legacy System
```

The adapter should act as an Anti-Corruption Layer where necessary.

---

# 47. Caching

Caching is an Infrastructure concern.

Potential architecture:

```text id="cbz1o1"
Application Query
      │
      ▼
Cache Adapter
      │
      ├── hit → result
      │
      └── miss
            │
            ▼
        Read Store
```

Caching must not become authoritative for business state.

Cache invalidation should be driven by explicit application/domain events where appropriate.

---

# 48. File Processing

File processing should remain outside the Domain.

Examples:

```text id="aj0v1d"
Image resizing
Thumbnail generation
Metadata extraction
Format conversion
Virus scanning
Checksum generation
```

The Application layer may request such operations through ports.

Infrastructure implements them.

---

# 49. Background Processing

Long-running infrastructure operations may be executed asynchronously.

Examples:

```text id="2m5jue"
Search reindexing
Media processing
External synchronization
Large collection import
Report generation
```

The architecture should model these as Application workflows triggered through appropriate scheduling or messaging infrastructure.

---

# 50. Retry Policy

Retry policies belong to Infrastructure/Application boundaries.

Retryable operations may include:

```text id="lqk37n"
External API call
Message publication
Message consumption
Search indexing
Media storage operation
```

Domain operations should not contain retry loops.

---

# 51. Dead-Letter Handling

Asynchronous infrastructure should support a dead-letter mechanism for messages that cannot be processed successfully after configured retries.

Flow:

```text id="f5e6yq"
Message
  │
  ▼
Consumer
  │
  ├── success
  │
  └── failure
       │
       ▼
     Retry
       │
       └── repeated failure
               │
               ▼
          Dead Letter
```

Dead-letter handling is an Infrastructure responsibility.

---

# 52. Infrastructure Security Boundary

Infrastructure is responsible for protecting technical credentials and secrets.

Examples:

```text id="1h3vpe"
Database credentials
API keys
Storage credentials
Message broker credentials
External provider tokens
```

Secrets must not be represented as Domain concepts.

---

# 53. Infrastructure Testing Strategy

Infrastructure should be tested at several levels.

### Adapter unit tests

Validate:

- mapping;
- serialization;
- error translation;
- configuration.

### Integration tests

Validate:

- repository behavior;
- database schema;
- message broker integration;
- search indexing;
- media storage.

### Contract tests

Validate:

- external APIs;
- message contracts;
- integration events.

Infrastructure tests should not replace Domain tests.

---

# 54. Infrastructure Test Isolation

The Domain must remain testable without Infrastructure.

Application tests should normally use:

```text id="n4k2r7"
FakeItemRepository
FakeCollectionRepository
FakeEventPublisher
FixedClock
```

Infrastructure integration tests can then validate the real implementations separately.

---

# 55. Preliminary Port-to-Adapter Map

| Port | Infrastructure Adapter |
|---|---|
| CollectionRepository | CollectionRepositoryAdapter |
| ItemRepository | ItemRepositoryAdapter |
| ClassificationRepository | ClassificationRepositoryAdapter |
| AcquisitionRepository | AcquisitionRepositoryAdapter |
| ValuationRepository | ValuationRepositoryAdapter |
| DisposalRepository | DisposalRepositoryAdapter |
| ProvenanceRepository | ProvenanceRepositoryAdapter |
| MediaRepository | MediaRepositoryAdapter |
| CollectionReadPort | CollectionReadAdapter |
| ItemReadPort | ItemReadAdapter |
| ClassificationReadPort | ClassificationReadAdapter |
| ValuationReadPort | ValuationReadAdapter |
| SearchPort | SearchEngineAdapter |
| MediaStoragePort | ObjectStorageAdapter |
| EventPublisher | MessageBrokerEventPublisher |
| Clock | SystemClock |
| IdGenerator | UuidGenerator |
| UnitOfWork | DatabaseUnitOfWork |

This table is an architectural baseline rather than a final implementation contract.

---

# 56. Infrastructure Dependency Matrix

| Infrastructure Component | Application | Domain | External Technology |
|---|---:|---:|---:|
| Repository Adapter | ✓ | ✓ | ✓ |
| Unit of Work | ✓ | ✗ | ✓ |
| Event Publisher | ✓ | ✗ | ✓ |
| Event Consumer | ✓ | ✗ | ✓ |
| Search Adapter | ✓ | ✗ | ✓ |
| Media Adapter | ✓ | ✗ | ✓ |
| External Service Adapter | ✓ | ✓ where port requires | ✓ |
| Clock | ✓ | ✓ where required | ✓ |
| Identity Generator | ✓ | ✓ where required | ✓ |
| Observability | ✓ | ✗ | ✓ |
| Configuration | ✓ | ✗ | ✓ |

---

# 57. Infrastructure Deployment Independence

The architecture should not require every infrastructure capability to become a separate deployable service.

For example:

```text id="d3n7os"
Modular Monolith
│
├── Application
├── Domain
├── Persistence
├── Search Adapter
├── Messaging
└── Media Storage Adapter
```

may initially be preferable to:

```text id="3g8s4j"
Item Service
Collection Service
Search Service
Media Service
Valuation Service
Messaging Service
```

Infrastructure modularity does not imply distributed deployment.

---

# 58. Scaling Boundaries

Potential future scaling candidates include:

```text id="1v1kjj"
Search
Media Processing
Import Processing
External Synchronization
Reporting
```

These should be extracted only if actual operational requirements justify it.

The architecture is intentionally designed so that extraction can happen later without redefining the Domain.

---

# 59. Infrastructure Failure Domains

The architecture should distinguish failure domains:

```text id="j4l6k7"
Core Transaction
    │
    └── Database

Derived State
    │
    └── Search

Asynchronous Processing
    │
    └── Message Broker

External Dependencies
    │
    ├── Valuation Provider
    └── External Catalog

Binary Storage
    │
    └── Media Store
```

A failure in one domain should not unnecessarily compromise unrelated capabilities.

---

# 60. Architectural Resilience Principle

The Infrastructure architecture should follow:

> **A secondary infrastructure failure should not automatically invalidate authoritative business state.**

For example:

```text id="gqgnyd"
Database
    ✓ commit

Search
    ✗ unavailable

Result:
Domain state remains valid.
Search catches up later.
```

This principle will strongly influence event publication, outbox design and retry mechanisms.

---

# 61. Infrastructure Observability Requirements

Every infrastructure adapter should expose enough information to diagnose:

- operation;
- duration;
- success/failure;
- retry count;
- correlation identifier;
- external dependency;
- failure category.

Sensitive information must never be logged indiscriminately.

---

# 62. Architectural Invariants

The following Infrastructure invariants are established:

### I1 — No infrastructure dependency from Domain

The Domain never imports concrete Infrastructure components.

### I2 — Adapter isolation

External technologies are hidden behind adapters.

### I3 — Port ownership

Ports belong to the inner layer requiring the capability.

### I4 — Persistence isolation

Database models do not define the Domain model.

### I5 — Event isolation

Transport messages do not define Domain Events.

### I6 — Search is derived

Search is not authoritative for transactional state.

### I7 — Media storage is externalized

Binary storage does not belong inside Domain entities.

### I8 — Infrastructure failures are translated

Technical exceptions do not leak directly across architectural boundaries.

### I9 — Configuration is centralized

Infrastructure configuration is assembled at the composition root.

### I10 — Observability is technical

Logging, metrics and tracing do not become Domain dependencies.

---

# 63. Infrastructure Architecture Checklist

Before implementing an infrastructure component:

- [ ] Is its responsibility explicitly defined?
- [ ] Does it implement an identified port?
- [ ] Is the port owned by the correct inner layer?
- [ ] Does it contain provider-specific logic?
- [ ] Is provider-specific logic isolated?
- [ ] Does it leak infrastructure types into Domain?
- [ ] Does it introduce business rules?
- [ ] Does it require transactions?
- [ ] Does it require retries?
- [ ] Does it require idempotency?
- [ ] Does it produce or consume events?
- [ ] Does it require observability?
- [ ] Does it introduce a new failure domain?
- [ ] Can it be tested independently?
- [ ] Can the underlying technology be replaced without changing the Domain?

---

# 64. Preliminary Infrastructure Structure

The resulting target structure is:

```text id="0m52m9"
infrastructure/
│
├── persistence/
│   ├── repositories/
│   ├── mappings/
│   ├── models/
│   ├── transactions/
│   └── migrations/
│
├── messaging/
│   ├── publishers/
│   ├── consumers/
│   ├── serializers/
│   ├── handlers/
│   └── outbox/
│
├── search/
│   ├── adapters/
│   ├── projections/
│   ├── indexing/
│   └── queries/
│
├── media/
│   ├── storage/
│   ├── processing/
│   └── adapters/
│
├── external/
│   ├── valuation/
│   ├── catalog/
│   ├── synchronization/
│   └── adapters/
│
├── identity/
│   └── providers/
│
├── time/
│   └── providers/
│
├── observability/
│   ├── logging/
│   ├── metrics/
│   └── tracing/
│
├── configuration/
│
└── composition/
```

---

# 65. Infrastructure Architectural View

The complete infrastructure interaction can now be represented as:

```text id="0e1p8u"
                       ┌──────────────────┐
                       │     Delivery     │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │   Application    │
                       │                  │
                       │ Use Cases        │
                       │ Ports            │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │      Domain      │
                       │                  │
                       │ Aggregates       │
                       │ Rules            │
                       │ Events           │
                       └────────┬─────────┘
                                │
                         domain/application
                              contracts
                                │
             ┌──────────────────┼───────────────────┐
             │                  │                   │
             ▼                  ▼                   ▼
      ┌────────────┐     ┌────────────┐      ┌────────────┐
      │ Persistence│     │ Messaging  │      │   Search   │
      └────────────┘     └────────────┘      └────────────┘
             │                  │                   │
             ▼                  ▼                   ▼
         Database          Broker/Bus          Search Store

             ┌──────────────────┼───────────────────┐
             │                  │                   │
             ▼                  ▼                   ▼
        Media Store       External APIs       Observability
```

This is the infrastructure architecture baseline for CollectionHub.

---

# 66. Architectural Decision

CollectionHub adopts an **adapter-oriented Infrastructure architecture** based on:

- explicit ports;
- dependency inversion;
- persistence isolation;
- event-driven secondary processing;
- search projection;
- media storage abstraction;
- external service adapters;
- centralized configuration;
- infrastructure-level observability;
- explicit failure and resilience boundaries.

The key architectural decision is:

> **Infrastructure is replaceable technology surrounding a stable Application and Domain core.**

The concrete database, search engine, message broker, storage provider and external APIs remain implementation decisions until the architectural constraints require their selection.

---

# 67. Relationship with Previous Architecture Documents

The current architecture now forms the following chain:

```text id="q8d3jr"
24
Architectural Boundaries
        │
        ▼
25
Architectural Components
        │
        ▼
26
Application / Domain Modules
        │
        ▼
27
Component Interactions
        │
        ▼
28
Application Use Case Interaction Map
        │
        ▼
29
Infrastructure Components and Adapters
```

Each document refines the previous one without prematurely introducing implementation technology.

---

# 68. Next Architectural Step

The next architectural document should define:

```text id="9j3h0m"
30_PERSISTENCE_ARCHITECTURE_AND_DATA_BOUNDARIES.md
```

That document will take the persistence requirements identified here and define in greater detail:

- authoritative data ownership;
- aggregate persistence boundaries;
- database responsibilities;
- transactional boundaries;
- persistence models;
- mapping strategy;
- concurrency;
- consistency;
- migrations;
- read models;
- projection storage;
- cache boundaries;
- data ownership between modules;
- persistence anti-corruption boundaries;
- future evolution strategy.

At that point, CollectionHub will have progressed from:

```text
Domain Model
    ↓
Application Architecture
    ↓
Component Interaction
    ↓
Infrastructure Architecture
    ↓
Persistence Architecture
```

which is the correct level of architectural precision before selecting concrete persistence technologies or writing implementation code.