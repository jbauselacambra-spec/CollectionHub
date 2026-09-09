# 26 — Application and Domain Module Structure

## 1. Purpose

This document defines the logical module structure for the **Application** and **Domain** layers of CollectionHub.

The objective is to establish a stable architectural organization before implementation begins, ensuring that:

- domain concepts remain independent from technical infrastructure;
- application orchestration is separated from business rules;
- dependencies point inwards toward the domain;
- aggregates, domain services, specifications, events and policies have explicit ownership;
- application use cases remain discoverable and cohesive;
- module boundaries reflect the architectural boundaries established in the previous phase;
- future infrastructure and delivery mechanisms can evolve without forcing changes into the core domain model.

This structure is a logical architectural structure. It does not prescribe a programming language or framework-specific package layout, although it is intentionally suitable for translation into a conventional modular codebase.

---

## 2. Architectural Principle

CollectionHub follows a dependency direction in which the **Domain is the innermost business layer** and the **Application layer coordinates domain capabilities**.

```text
┌───────────────────────────────────────────────────────────────┐
│                         Delivery                              │
│              API / UI / CLI / External Adapters              │
└───────────────────────────────┬───────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                        Application                            │
│                                                               │
│  Commands / Queries / Use Cases / Workflows / DTOs / Ports   │
└───────────────────────────────┬───────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                           Domain                              │
│                                                               │
│  Aggregates / Entities / Value Objects / Services             │
│  Specifications / Policies / Domain Events / Invariants      │
└───────────────────────────────────────────────────────────────┘
                                ▲
                                │
                     implemented through ports
                                │
┌───────────────────────────────┴───────────────────────────────┐
│                       Infrastructure                          │
│                                                               │
│ Persistence / Messaging / External APIs / File Systems        │
└───────────────────────────────────────────────────────────────┘
```

The essential rule is:

> **Application coordinates business capabilities; Domain defines business meaning and business rules.**

The Application layer must not become a second domain layer, and the Domain layer must not depend on Application or Infrastructure concerns.

---

## 3. Top-Level Module Structure

The logical structure is organized around the following modules:

```text
CollectionHub
│
├── domain
│   ├── shared
│   ├── collection
│   ├── item
│   ├── classification
│   ├── valuation
│   ├── acquisition
│   ├── disposal
│   ├── provenance
│   ├── media
│   ├── metadata
│   ├── search
│   └── events
│
├── application
│   ├── collections
│   ├── items
│   ├── classification
│   ├── valuation
│   ├── acquisition
│   ├── disposal
│   ├── provenance
│   ├── media
│   ├── search
│   └── shared
│
├── infrastructure
│   └── ...
│
└── delivery
    └── ...
```

The exact bounded contexts and aggregate names remain governed by the domain model defined in the preceding Phase 2.2 documents.

The structure above therefore represents the **architectural module map**, not an assertion that every named directory must necessarily become a separate runtime component.

---

# 4. Domain Module

## 4.1 Responsibility

The Domain module contains the business model of CollectionHub.

It owns:

- entities;
- aggregate roots;
- value objects;
- domain invariants;
- domain services;
- domain specifications;
- domain policies;
- domain events;
- domain-level errors;
- business concepts and terminology.

It must be possible to reason about the Domain module without knowing:

- which database is used;
- which HTTP framework is used;
- which UI exists;
- which message broker is used;
- which ORM is used;
- how authentication is implemented;
- how external integrations are connected.

---

# 5. Domain Internal Structure

A recommended logical structure is:

```text
domain/
│
├── shared/
│   ├── entity/
│   ├── value-object/
│   ├── domain-error/
│   ├── domain-event/
│   └── types/
│
├── collection/
│   ├── entities/
│   ├── aggregates/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   ├── policies/
│   └── events/
│
├── item/
│   ├── entities/
│   ├── aggregates/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   ├── policies/
│   └── events/
│
├── classification/
│   ├── entities/
│   ├── aggregates/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   └── events/
│
├── valuation/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   └── events/
│
├── acquisition/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   └── events/
│
├── disposal/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   └── events/
│
├── provenance/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── specifications/
│   └── events/
│
├── media/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   └── events/
│
├── metadata/
│   ├── entities/
│   ├── value-objects/
│   ├── specifications/
│   └── services/
│
├── search/
│   ├── specifications/
│   ├── policies/
│   └── services/
│
└── events/
    ├── integration/
    └── internal/
```

This structure is intentionally organized by **business capability first**, rather than by technical artifact first.

---

# 6. Domain Modules by Business Capability

## 6.1 Collection

The Collection module owns concepts related to the logical organization of collected objects.

Potential responsibilities include:

- collection identity;
- collection lifecycle;
- collection metadata;
- collection-level rules;
- collection membership;
- collection-specific invariants.

It must not contain persistence concerns.

---

## 6.2 Item

The Item module represents the collectible object and its lifecycle.

Potential responsibilities include:

- item identity;
- item state;
- item attributes;
- item ownership within the domain;
- item lifecycle rules;
- item-specific invariants;
- relationships with classification and provenance concepts.

The Item aggregate must remain responsible for rules that require transactional consistency within its boundary.

---

## 6.3 Classification

The Classification module represents the domain concepts required to categorize collection items.

Potential responsibilities include:

- categories;
- classification schemes;
- classification assignments;
- hierarchical classification rules;
- classification validity.

Classification should not become a generic tagging subsystem unless the domain explicitly requires such behavior.

---

## 6.4 Valuation

The Valuation module owns concepts associated with monetary or non-monetary valuation.

Potential responsibilities include:

- valuation records;
- valuation sources;
- valuation dates;
- valuation methods;
- valuation history;
- valuation-related domain rules.

External market data providers belong outside the Domain layer.

---

## 6.5 Acquisition

The Acquisition module represents how an item enters the collection.

Potential responsibilities include:

- acquisition records;
- acquisition methods;
- acquisition dates;
- acquisition costs;
- acquisition provenance;
- acquisition-related invariants.

---

## 6.6 Disposal

The Disposal module represents the lifecycle transition through which an item leaves the collection.

Potential responsibilities include:

- disposal records;
- disposal reasons;
- disposal dates;
- disposal values;
- disposal rules;
- lifecycle constraints.

---

## 6.7 Provenance

The Provenance module represents historical information about the origin and ownership history of an item.

Potential responsibilities include:

- provenance records;
- historical ownership;
- source information;
- provenance events;
- provenance validation rules.

Because provenance may require historical reconstruction rather than simple CRUD semantics, it must remain modeled as a domain concept rather than being reduced to database metadata.

---

## 6.8 Media

The Media module represents domain-level concepts related to media associated with collection objects.

The Domain owns concepts such as:

- media identity;
- media association;
- media type;
- media ordering;
- media role;
- media lifecycle state.

Actual binary storage belongs to Infrastructure.

---

## 6.9 Metadata

The Metadata module contains domain-specific descriptive information that does not belong directly to the structural identity of an aggregate.

This module must be carefully controlled to prevent it from becoming an unstructured "miscellaneous fields" container.

---

## 6.10 Search

Search is primarily an Application/Infrastructure concern, but certain reusable domain specifications may originate from the Domain.

For example:

```text
ItemIsCurrentlyOwned
ItemBelongsToCollection
ItemMatchesClassification
ItemHasValuation
ItemHasProvenance
```

The Domain defines reusable business predicates.

The Application layer composes them into use-case-specific queries.

Infrastructure determines how those queries are executed efficiently.

---

# 7. Shared Domain Module

The `domain/shared` module contains only concepts that are genuinely shared across multiple bounded domain areas.

Typical contents include:

```text
domain/shared/
├── entity/
├── value-object/
├── domain-event/
├── domain-error/
└── types/
```

Examples:

- Entity identity;
- domain event abstractions;
- common domain errors;
- common value-object primitives;
- common domain types.

The shared module must remain deliberately small.

A business concept should not be moved into `shared` merely because several modules currently use it.

> **Shared code is not automatically shared domain meaning.**

---

# 8. Application Module

## 8.1 Responsibility

The Application module coordinates the execution of business use cases.

It is responsible for:

- application use cases;
- commands;
- queries;
- workflows;
- transaction orchestration;
- invoking domain behavior;
- coordinating multiple aggregates;
- application-level authorization checks where appropriate;
- mapping application input to domain concepts;
- mapping domain results to application outputs;
- defining ports required by application workflows.

It is **not** responsible for implementing core business invariants that belong to the Domain.

---

# 9. Application Internal Structure

Recommended logical structure:

```text
application/
│
├── shared/
│   ├── commands/
│   ├── queries/
│   ├── results/
│   ├── errors/
│   ├── ports/
│   └── transaction/
│
├── collections/
│   ├── commands/
│   ├── queries/
│   ├── workflows/
│   └── results/
│
├── items/
│   ├── commands/
│   ├── queries/
│   ├── workflows/
│   └── results/
│
├── classification/
│   ├── commands/
│   ├── queries/
│   └── results/
│
├── valuation/
│   ├── commands/
│   ├── queries/
│   ├── workflows/
│   └── results/
│
├── acquisition/
│   ├── commands/
│   ├── queries/
│   ├── workflows/
│   └── results/
│
├── disposal/
│   ├── commands/
│   ├── queries/
│   ├── workflows/
│   └── results/
│
├── provenance/
│   ├── commands/
│   ├── queries/
│   └── results/
│
├── media/
│   ├── commands/
│   ├── queries/
│   └── results/
│
└── search/
    ├── queries/
    ├── filters/
    └── results/
```

---

# 10. Application Commands

Commands represent an explicit request to change application state.

Examples:

```text
CreateCollection
RenameCollection
AddItemToCollection
UpdateItem
ClassifyItem
RecordAcquisition
RecordValuation
RecordDisposal
AddProvenanceRecord
AttachMedia
```

A command should represent **intent**, not persistence mechanics.

Bad:

```text
InsertItemRow
UpdateItemTable
SaveCollectionRecord
```

Better:

```text
CreateItem
UpdateItem
MoveItemToCollection
RecordAcquisition
```

---

# 11. Application Queries

Queries retrieve information without expressing a domain state transition.

Examples:

```text
GetCollection
GetItem
ListCollectionItems
SearchItems
GetItemHistory
GetItemValuationHistory
GetItemProvenance
GetClassificationTree
```

Queries may use optimized read models where appropriate.

The use of a read model does not imply that the domain model should be weakened.

---

# 12. Application Workflows

Workflows coordinate operations that cross multiple domain boundaries or require multiple application steps.

For example:

```text
AcquireItemIntoCollection
DisposeItemFromCollection
ImportCollection
TransferItem
ReclassifyCollection
```

A workflow may:

1. validate application input;
2. load one or more aggregates;
3. invoke domain behavior;
4. coordinate multiple domain services;
5. persist changes through ports;
6. publish resulting events;
7. construct the application result.

The workflow must not recreate domain invariants procedurally.

---

# 13. Application Ports

Application ports define dependencies required by use cases.

Examples:

```text
CollectionRepository
ItemRepository
ClassificationRepository
ValuationRepository
ProvenanceRepository
MediaRepository
UnitOfWork
EventPublisher
Clock
IdGenerator
```

These are interfaces/contracts from the perspective of the Application layer.

Infrastructure implements them.

Dependency direction:

```text
Application
    │
    ├── defines port
    │
    ▼
Infrastructure
    │
    └── implements port
```

The Application layer therefore does not depend on a concrete database adapter.

---

# 14. Domain Services vs Application Services

The distinction must remain explicit.

### Domain Service

A Domain Service:

- expresses domain behavior;
- operates on domain concepts;
- exists because the behavior does not naturally belong to one entity or aggregate;
- contains business logic.

Example:

```text
ValuationPolicy
ProvenanceConsistencyService
CollectionClassificationService
```

### Application Service

An Application Service:

- orchestrates a use case;
- loads aggregates;
- invokes domain behavior;
- coordinates repositories and ports;
- controls transaction boundaries;
- produces application results.

Example:

```text
RecordAcquisition
DisposeItem
SearchCollection
```

A useful rule is:

> If removing the database and HTTP API would still leave the logic meaningful as business behavior, it probably belongs to the Domain.

---

# 15. Dependency Rules

The following dependency rules are mandatory architectural constraints.

## Rule 1 — Domain has no Infrastructure dependency

```text
Domain ──X──> Infrastructure
```

The Domain must never import:

- ORM entities;
- database clients;
- HTTP clients;
- framework controllers;
- message brokers;
- filesystem APIs;
- cloud SDKs.

---

## Rule 2 — Domain has no Delivery dependency

```text
Domain ──X──> API / UI / CLI
```

The Domain must not know how users interact with the system.

---

## Rule 3 — Application may depend on Domain

```text
Application ──> Domain
```

This is the principal dependency.

---

## Rule 4 — Infrastructure implements Application/Domain ports

```text
Infrastructure ──> Application contracts
Infrastructure ──> Domain contracts
```

Concrete adapters belong outside the core.

---

## Rule 5 — Delivery depends on Application

```text
Delivery ──> Application
```

Controllers and handlers should invoke application use cases rather than manipulating domain objects directly.

---

# 16. Aggregate Ownership

Each aggregate must have a clear owning module.

Example conceptual structure:

```text
domain/
├── collection/
│   └── aggregates/
│       └── Collection
│
├── item/
│   └── aggregates/
│       └── Item
│
├── classification/
│   └── aggregates/
│       └── Classification
│
└── ...
```

Cross-aggregate operations must not bypass aggregate boundaries.

The Application layer coordinates multiple aggregates when necessary.

---

# 17. Cross-Module Domain Dependencies

Cross-module dependencies are allowed only when they represent genuine domain relationships.

For example:

```text
Item
 ├── references Collection
 ├── references Classification
 └── references Provenance
```

However, the dependency must not automatically imply object ownership.

A reference to another aggregate should normally be represented through:

- aggregate identity;
- domain value object;
- explicit domain reference.

Avoid embedding entire aggregates inside other aggregate roots merely for convenience.

---

# 18. Domain Events

Domain events should live close to the domain concept that produces them.

Example:

```text
domain/item/events/
    ItemCreated
    ItemUpdated
    ItemClassified

domain/acquisition/events/
    AcquisitionRecorded

domain/disposal/events/
    ItemDisposed

domain/valuation/events/
    ValuationRecorded
```

Events represent facts that have occurred.

They should not encode technical commands such as:

```text
SendEmail
UpdateSearchIndex
InsertDatabaseRecord
```

Those are reactions or infrastructure concerns.

---

# 19. Application Event Handling

Application-level handlers may react to domain events.

Conceptually:

```text
Domain Aggregate
       │
       ▼
 Domain Event
       │
       ▼
Application Event Handler
       │
       ├── update read model
       ├── invoke another use case
       ├── publish integration event
       └── trigger infrastructure adapter
```

This prevents infrastructure side effects from leaking into aggregate behavior.

---

# 20. Specifications

Specifications belong in the Domain when they express reusable business predicates.

Examples:

```text
ItemBelongsToCollection
ItemIsActive
ItemCanBeDisposed
ItemHasValidClassification
ValuationIsCurrent
ProvenanceIsConsistent
```

Application-specific filtering should remain in Application or Infrastructure.

The distinction is:

```text
Business rule
    → Domain Specification

Screen/filter/search requirement
    → Application Query

Database optimization
    → Infrastructure
```

---

# 21. Error Ownership

Errors should be defined according to the layer that owns the violated rule.

### Domain errors

Examples:

```text
InvalidItemState
InvalidAcquisition
InvalidDisposal
ClassificationNotAllowed
InvariantViolation
```

### Application errors

Examples:

```text
ItemNotFound
CollectionNotFound
UseCaseValidationError
UnauthorizedOperation
Conflict
```

### Infrastructure errors

Examples:

```text
DatabaseUnavailable
ExternalServiceUnavailable
MessageBrokerUnavailable
FileStorageFailure
```

Infrastructure failures must not be disguised as domain errors.

---

# 22. DTO Boundaries

Application DTOs define the boundary between external consumers and the application layer.

Example:

```text
CreateItemCommand
    ↓
Application
    ↓
Item aggregate
```

and:

```text
Item aggregate
    ↓
Application result
    ↓
ItemResponse
```

Domain entities should not be exposed directly as API contracts.

This protects the Domain from transport concerns and allows external contracts to evolve independently.

---

# 23. Persistence Boundary

Persistence mapping must remain outside the Domain.

Conceptually:

```text
Domain Entity
      │
      │ repository abstraction
      ▼
Application / Domain Port
      │
      ▼
Infrastructure Adapter
      │
      ▼
Persistence Model
      │
      ▼
Database
```

A persistence model may differ from a domain entity.

This is intentional.

Database normalization, indexing and query optimization must not dictate the shape of the domain model.

---

# 24. Recommended Package Naming Convention

The preferred convention is:

```text
<layer>/<business-capability>/<artifact-type>
```

For example:

```text
application/items/commands
application/items/queries
domain/item/aggregates
domain/item/value-objects
domain/item/specifications
```

Avoid a global technical structure such as:

```text
entities/
services/
repositories/
controllers/
utils/
```

because that structure groups artifacts by implementation type rather than business responsibility.

---

# 25. Forbidden Structural Patterns

The following structures should be explicitly avoided.

### 25.1 Anemic domain model

```text
Entity
  ↓
ApplicationService
  ↓
All business rules
```

Business rules should not accumulate inside generic application services.

---

### 25.2 Generic service dumping ground

Avoid:

```text
services/
    CollectionService
    ItemService
    CommonService
    UtilityService
```

where unrelated business behavior accumulates.

---

### 25.3 Domain depending on repositories implemented by Infrastructure

Avoid concrete dependencies such as:

```text
Item → SqlItemRepository
```

Prefer:

```text
Application/Domain → ItemRepository
Infrastructure → SqlItemRepository
```

---

### 25.4 Framework-driven domain structure

Avoid allowing framework conventions to determine domain boundaries.

The desired direction is:

```text
Domain model
      ↓
Architectural modules
      ↓
Application use cases
      ↓
Framework adapters
```

not:

```text
Framework
      ↓
Database
      ↓
Entities
      ↓
Retrofit business rules
```

---

# 26. Module Dependency Matrix

| Module | Domain | Application | Infrastructure | Delivery |
|---|---:|---:|---:|---:|
| Domain | ✓ | ✗ | ✗ | ✗ |
| Application | ✓ | ✓ internal | contracts only | ✗ |
| Infrastructure | contracts | ✓ | ✓ internal | ✗ |
| Delivery | ✗ | ✓ | adapters where required | ✓ |

The critical architectural dependency is:

```text
Delivery
    ↓
Application
    ↓
Domain

Infrastructure
    ↑
implements contracts
```

---

# 27. Module Cohesion Principle

A module should contain concepts that change together because they belong to the same business responsibility.

A strong module should answer:

> "What business capability does this module own?"

If the answer is primarily technical:

> "It contains repositories."

or:

> "It contains services."

the module boundary is probably implementation-oriented rather than domain-oriented.

---

# 28. Module Coupling Principle

Coupling between modules should be minimized.

Preferred:

```text
Item
  │
  └── ClassificationId
```

Less desirable:

```text
Item
  │
  └── complete Classification aggregate
```

Even less desirable:

```text
Item
  │
  └── ClassificationRepository
```

The first expresses a domain relationship without creating unnecessary structural coupling.

---

# 29. Evolution Strategy

The module structure should support gradual extraction.

Initially:

```text
CollectionHub
└── modular monolith
    ├── domain
    ├── application
    ├── infrastructure
    └── delivery
```

If future requirements justify service decomposition, modules can evolve toward:

```text
Collection Service
Item Service
Classification Service
Valuation Service
Media Service
```

without redesigning the domain concepts from scratch.

Therefore:

> **Modularity is established before distribution.**

CollectionHub should not introduce microservices merely because the domain contains multiple modules.

---

# 30. Architectural Testability

The structure must make it possible to test each layer independently.

### Domain tests

Test:

- invariants;
- aggregate behavior;
- value objects;
- specifications;
- domain services;
- domain events.

No database or HTTP server should be required.

### Application tests

Test:

- use-case orchestration;
- workflow behavior;
- transaction boundaries;
- repository interactions;
- event publication;
- application errors.

Infrastructure should normally be replaced by test doubles.

### Infrastructure tests

Test:

- persistence mappings;
- repository implementations;
- external integrations;
- messaging;
- storage adapters.

### Delivery tests

Test:

- request mapping;
- response mapping;
- authentication integration;
- transport-level behavior.

---

# 31. Architectural Decision

CollectionHub adopts a **business-capability-oriented modular structure** for Domain and Application.

The resulting principle is:

```text
Business capability
      ↓
Domain module
      ↓
Application use cases
      ↓
Infrastructure adapters
      ↓
Delivery mechanisms
```

Technical concerns must not be allowed to redefine business boundaries.

---

# 32. Final Structural View

The target architecture can therefore be represented as:

```text
CollectionHub
│
├── Domain
│   │
│   ├── Shared
│   │
│   ├── Collection
│   ├── Item
│   ├── Classification
│   ├── Valuation
│   ├── Acquisition
│   ├── Disposal
│   ├── Provenance
│   ├── Media
│   ├── Metadata
│   └── Search Specifications
│
├── Application
│   │
│   ├── Collection Use Cases
│   ├── Item Use Cases
│   ├── Classification Use Cases
│   ├── Valuation Use Cases
│   ├── Acquisition Use Cases
│   ├── Disposal Use Cases
│   ├── Provenance Use Cases
│   ├── Media Use Cases
│   └── Search Queries
│
├── Infrastructure
│   │
│   ├── Persistence
│   ├── External Services
│   ├── Messaging
│   ├── File Storage
│   └── Other Adapters
│
└── Delivery
    │
    ├── API
    ├── UI
    └── Other Entry Points
```

The key architectural constraint is that **Domain and Application boundaries are defined by CollectionHub's business model, not by the chosen implementation technology**.

This structure provides the foundation required for the next architectural decisions concerning component interaction, dependency rules, persistence boundaries, infrastructure adapters and deployment structure.