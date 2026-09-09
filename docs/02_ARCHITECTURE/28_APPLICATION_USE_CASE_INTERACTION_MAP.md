# 28 — Application Use Case Interaction Map

## 1. Purpose

This document maps CollectionHub application use cases to the architectural components, domain capabilities, aggregates, ports, events and side effects defined in the previous architecture documents.

It establishes how each application use case traverses the architecture without prescribing implementation details.

The objective is to answer:

> **Given a user or system intention, which application component coordinates it, which domain capabilities participate, which boundaries are crossed, and which side effects may occur?**

This document therefore acts as the bridge between:

```text
Domain Model
      ↓
Application Use Cases
      ↓
Architectural Components
      ↓
Infrastructure Requirements
```

It also provides the basis for later:

- API design;
- command/query definitions;
- transaction design;
- repository contracts;
- event handler design;
- infrastructure adapter design;
- architecture tests;
- integration testing.

---

# 2. Architectural Context

The interaction model established previously is:

```text
Delivery
    │
    ▼
Application Use Case
    │
    ├───────────────┐
    ▼               ▼
Domain          Application Ports
    │               │
    ▼               ▼
Aggregates      Infrastructure
    │
    ▼
Domain Events
    │
    ▼
Application / Integration Handlers
```

The Application layer coordinates.

The Domain decides.

Infrastructure implements.

Delivery exposes.

---

# 3. Use Case Classification

CollectionHub use cases are divided into the following categories:

```text
Collection Management
Item Management
Classification
Acquisition
Valuation
Disposal
Provenance
Media
Search and Retrieval
Import / Synchronization
Cross-Aggregate Workflows
```

Not every category necessarily becomes a separate bounded context.

The categorization is primarily intended to provide architectural organization and traceability.

---

# 4. Interaction Legend

The maps in this document use the following notation:

```text
[CMD]      Command / state-changing use case
[QUERY]    Query / read-only use case
[WF]       Cross-component workflow
[EVENT]    Domain event
[PORT]     Application or domain port
[ASYNC]    Potential asynchronous interaction
[TX]       Transactional boundary
```

---

# 5. Collection Use Cases

## 5.1 Create Collection

```text
CreateCollection [CMD]
        │
        ▼
CreateCollectionUseCase
        │
        ▼
Collection Aggregate
        │
        ├── validate creation invariants
        ├── create Collection
        │
        ▼
CollectionCreated [EVENT]
        │
        ▼
Repository / Event Handling
```

### Primary components

```text
Application:
    CreateCollectionUseCase

Domain:
    Collection Aggregate
    Collection identity
    Collection value objects
    Collection invariants

Ports:
    CollectionRepository
    IdGenerator
    Clock
```

### Transaction

```text
[TX]
Create Collection
Persist Collection
Commit
```

### Potential side effects

```text
[ASYNC]
Create search projection
Publish integration event
Create audit record
```

---

## 5.2 Rename Collection

```text
RenameCollection [CMD]
        │
        ▼
RenameCollectionUseCase
        │
        ▼
CollectionRepository
        │
        ▼
Collection Aggregate
        │
        ▼
rename(...)
        │
        ▼
CollectionRenamed [EVENT]
```

The rename operation must respect collection lifecycle and validation rules defined by the Domain.

---

## 5.3 Get Collection

```text
GetCollection [QUERY]
        │
        ▼
GetCollectionQueryHandler
        │
        ▼
CollectionReadPort
        │
        ▼
Read Model
        │
        ▼
CollectionResult
```

This query does not require mutation of the Collection aggregate.

---

## 5.4 List Collections

```text
ListCollections [QUERY]
        │
        ▼
ListCollectionsQueryHandler
        │
        ▼
CollectionReadPort
        │
        ▼
Read Model
```

Pagination, sorting and filtering remain Application/Infrastructure concerns unless a specific business rule is involved.

---

# 6. Item Use Cases

## 6.1 Create Item

```text
CreateItem [CMD]
        │
        ▼
CreateItemUseCase
        │
        ├── validate application input
        │
        ▼
Item Aggregate
        │
        ├── create
        ├── enforce invariants
        │
        ▼
ItemCreated [EVENT]
        │
        ▼
ItemRepository
```

### Main dependencies

```text
ItemRepository
CollectionRepository / CollectionReference
IdGenerator
Clock
```

If the item must belong to an existing collection, the Application layer verifies the relevant collection boundary according to the domain rules.

---

## 6.2 Update Item

```text
UpdateItem [CMD]
        │
        ▼
UpdateItemUseCase
        │
        ▼
ItemRepository
        │
        ▼
Item Aggregate
        │
        ▼
update(...)
        │
        ▼
ItemUpdated [EVENT]
```

The Application layer should not directly assign arbitrary fields to an entity if doing so bypasses domain invariants.

---

## 6.3 Get Item

```text
GetItem [QUERY]
        │
        ▼
GetItemQueryHandler
        │
        ▼
ItemReadPort
        │
        ▼
Read Model
```

The result may include denormalized information such as:

```text
Item
Collection
Classification
Current Valuation
Current Status
Media Summary
```

without requiring the query to reconstruct the entire aggregate graph.

---

## 6.4 List Collection Items

```text
ListCollectionItems [QUERY]
        │
        ▼
CollectionItemsQueryHandler
        │
        ▼
ItemReadPort
        │
        ▼
Collection Item Read Model
```

This is a read-oriented interaction.

No collection aggregate mutation is required.

---

# 7. Classification Use Cases

## 7.1 Classify Item

```text
ClassifyItem [CMD]
        │
        ▼
ClassifyItemUseCase
        │
        ├── load Item
        ├── validate Classification
        │
        ▼
Item / Classification Domain Behavior
        │
        ▼
ClassificationChanged [EVENT]
```

The exact aggregate ownership of the classification relationship must follow the aggregate model established in the domain design.

The Application layer must not invent a second classification rule.

---

## 7.2 Create Classification

```text
CreateClassification [CMD]
        │
        ▼
CreateClassificationUseCase
        │
        ▼
Classification Aggregate
        │
        ▼
ClassificationCreated [EVENT]
```

---

## 7.3 Get Classification Tree

```text
GetClassificationTree [QUERY]
        │
        ▼
ClassificationQueryHandler
        │
        ▼
ClassificationReadPort
        │
        ▼
Classification Read Model
```

The query may use a specialized hierarchical projection.

---

# 8. Acquisition Use Cases

## 8.1 Record Acquisition

```text
RecordAcquisition [CMD]
        │
        ▼
RecordAcquisitionUseCase
        │
        ├── load Item
        ├── validate acquisition data
        │
        ▼
Acquisition Domain Behavior
        │
        ├── update lifecycle state
        ├── record acquisition
        │
        ▼
AcquisitionRecorded [EVENT]
```

### Potential dependencies

```text
ItemRepository
AcquisitionRepository
Clock
IdGenerator
```

### Transaction

If acquisition and item lifecycle state are part of the same consistency boundary:

```text
[TX]
Item change
+
Acquisition record
+
Commit
```

Otherwise, the workflow must explicitly define the consistency model.

---

## 8.2 Get Acquisition History

```text
GetAcquisitionHistory [QUERY]
        │
        ▼
AcquisitionHistoryQueryHandler
        │
        ▼
AcquisitionReadPort
        │
        ▼
Read Model
```

Historical queries should preferably use a dedicated read model where the data volume or query complexity justifies it.

---

# 9. Valuation Use Cases

## 9.1 Record Valuation

```text
RecordValuation [CMD]
        │
        ▼
RecordValuationUseCase
        │
        ├── load Item
        ├── validate valuation
        │
        ▼
Valuation Domain Service / Aggregate
        │
        ▼
ValuationRecorded [EVENT]
```

The Domain owns rules such as:

- valid valuation dates;
- valid monetary values;
- allowed valuation state transitions;
- valuation source requirements.

---

## 9.2 Get Current Valuation

```text
GetCurrentValuation [QUERY]
        │
        ▼
ValuationQueryHandler
        │
        ▼
ValuationReadPort
        │
        ▼
Current Valuation Projection
```

---

## 9.3 Get Valuation History

```text
GetValuationHistory [QUERY]
        │
        ▼
ValuationHistoryQueryHandler
        │
        ▼
ValuationReadPort
        │
        ▼
Historical Valuation Model
```

---

# 10. Disposal Use Cases

## 10.1 Dispose Item

```text
DisposeItem [CMD]
        │
        ▼
DisposeItemUseCase
        │
        ▼
ItemRepository
        │
        ▼
Item Aggregate
        │
        ├── validate disposal
        ├── change lifecycle state
        │
        ▼
ItemDisposed [EVENT]
        │
        ├── Search Projection [ASYNC]
        ├── Reporting Projection [ASYNC]
        └── Integration Event [ASYNC]
```

### Critical domain rule

The Application layer may initiate disposal.

The Domain decides whether the item is legally and semantically disposable according to the domain invariants.

---

## 10.2 Get Disposal History

```text
GetDisposalHistory [QUERY]
        │
        ▼
DisposalHistoryQueryHandler
        │
        ▼
DisposalReadPort
        │
        ▼
Read Model
```

---

# 11. Provenance Use Cases

## 11.1 Add Provenance Record

```text
AddProvenanceRecord [CMD]
        │
        ▼
AddProvenanceRecordUseCase
        │
        ├── load Item
        ├── validate provenance
        │
        ▼
Provenance Domain Behavior
        │
        ▼
ProvenanceRecorded [EVENT]
```

Potential domain concerns include:

- chronology;
- ownership transitions;
- provenance completeness;
- source validation;
- contradictory historical records.

---

## 11.2 Get Provenance History

```text
GetProvenanceHistory [QUERY]
        │
        ▼
ProvenanceQueryHandler
        │
        ▼
ProvenanceReadPort
        │
        ▼
Historical Read Model
```

---

# 12. Media Use Cases

## 12.1 Attach Media

```text
AttachMedia [CMD]
        │
        ▼
AttachMediaUseCase
        │
        ├── validate item
        ├── create MediaReference
        │
        ▼
MediaStoragePort
        │
        ▼
Infrastructure
```

The Domain owns the media association.

Infrastructure owns binary storage.

---

## 12.2 Remove Media

```text
RemoveMedia [CMD]
        │
        ▼
RemoveMediaUseCase
        │
        ├── validate removal
        ├── update media association
        │
        ▼
MediaRemoved [EVENT]
        │
        ▼
MediaStoragePort
```

Physical deletion may be immediate or asynchronous depending on the final storage architecture.

---

## 12.3 List Item Media

```text
ListItemMedia [QUERY]
        │
        ▼
MediaQueryHandler
        │
        ▼
MediaReadPort
        │
        ▼
Media Read Model
```

---

# 13. Search Use Cases

## 13.1 Search Items

```text
SearchItems [QUERY]
        │
        ▼
SearchItemsQueryHandler
        │
        ▼
SearchPort
        │
        ▼
Search Infrastructure
        │
        ▼
Search Results
```

Search is intentionally separated from transactional persistence.

The search engine is an optimization and retrieval mechanism.

---

## 13.2 Search by Classification

```text
SearchByClassification [QUERY]
        │
        ▼
Search Query
        │
        ▼
SearchPort
        │
        ▼
Search Projection
```

Domain specifications may define what a valid classification relationship means, while the search infrastructure determines how the query is executed.

---

## 13.3 Search by Valuation

```text
SearchByValuation [QUERY]
        │
        ▼
SearchItemsQueryHandler
        │
        ▼
SearchPort
```

The search projection may contain denormalized valuation information.

The authoritative valuation remains the domain state.

---

# 14. Cross-Aggregate Workflow — Acquire Item

A complete acquisition workflow may be modeled as:

```text
AcquireItemIntoCollection [WF]
        │
        ▼
Application Workflow
        │
        ├── validate Collection
        ├── create/load Item
        ├── execute Acquisition behavior
        ├── persist affected aggregates
        │
        ▼
Domain Events
        │
        ├── ItemCreated
        └── AcquisitionRecorded
```

The exact aggregate boundaries determine whether these operations share a single transaction.

---

# 15. Cross-Aggregate Workflow — Transfer Item

```text
TransferItem [WF]
        │
        ├── load Item
        ├── load SourceCollection
        ├── load TargetCollection
        │
        ├── Item.transfer(...)
        ├── SourceCollection.remove(...)
        └── TargetCollection.add(...)
                │
                ▼
        Domain Events
```

The workflow must explicitly define:

- transaction scope;
- failure behavior;
- consistency requirements;
- event ordering.

---

# 16. Cross-Aggregate Workflow — Dispose Item

```text
DisposeItem [WF]
        │
        ▼
Item
        │
        ▼
Disposal
        │
        ▼
ItemDisposed
        │
        ├── Search Projection
        ├── Reporting
        └── Integration
```

The disposal decision itself belongs to the Domain.

The Application layer coordinates the workflow and persistence.

---

# 17. Import Collection

CollectionHub may eventually support importing collection data.

A conceptual import workflow is:

```text
ImportCollection [WF]
        │
        ▼
Import Application Service
        │
        ├── parse input
        ├── validate structural data
        ├── map to commands
        │
        ▼
Domain Use Cases
        │
        ├── CreateCollection
        ├── CreateItem
        ├── ClassifyItem
        ├── RecordAcquisition
        └── AttachMedia
```

Import parsing belongs to Application/Infrastructure.

Business validation remains in the Domain.

---

# 18. Bulk Operations

Bulk operations should not bypass domain rules merely for performance.

For example:

```text
BulkClassifyItems
```

must not simply execute:

```text
UPDATE items SET classification = ...
```

if classification is governed by domain invariants.

Instead:

```text
Bulk Operation
      ↓
Application orchestration
      ↓
Domain behavior
```

Infrastructure-level bulk optimization may be introduced only when it preserves the required domain guarantees.

---

# 19. Application Port Map

The following ports are likely required by the mapped use cases.

```text
Persistence Ports
-----------------
CollectionRepository
ItemRepository
ClassificationRepository
AcquisitionRepository
ValuationRepository
DisposalRepository
ProvenanceRepository
MediaRepository

Read Ports
----------
CollectionReadPort
ItemReadPort
ClassificationReadPort
AcquisitionReadPort
ValuationReadPort
DisposalReadPort
ProvenanceReadPort
MediaReadPort
SearchPort

Infrastructure Services
-----------------------
MediaStoragePort
EventPublisher
Clock
IdGenerator
UnitOfWork
```

The final list must be refined during infrastructure modeling.

---

# 20. Port Ownership

Ports should be owned by the layer that needs the capability.

For example:

```text id="k1q8o2"
Application needs persistence
        │
        ▼
Application defines repository contract
        │
        ▼
Infrastructure implements it
```

This avoids a generic Infrastructure API dictating application architecture.

---

# 21. Use Case to Port Matrix

| Use Case | Primary Port(s) |
|---|---|
| CreateCollection | CollectionRepository, IdGenerator, Clock |
| RenameCollection | CollectionRepository |
| GetCollection | CollectionReadPort |
| ListCollections | CollectionReadPort |
| CreateItem | ItemRepository, CollectionRepository, IdGenerator |
| UpdateItem | ItemRepository |
| GetItem | ItemReadPort |
| ListCollectionItems | ItemReadPort |
| ClassifyItem | ItemRepository, ClassificationRepository |
| CreateClassification | ClassificationRepository, IdGenerator |
| GetClassificationTree | ClassificationReadPort |
| RecordAcquisition | ItemRepository, AcquisitionRepository |
| GetAcquisitionHistory | AcquisitionReadPort |
| RecordValuation | ItemRepository, ValuationRepository |
| GetCurrentValuation | ValuationReadPort |
| GetValuationHistory | ValuationReadPort |
| DisposeItem | ItemRepository, DisposalRepository |
| GetDisposalHistory | DisposalReadPort |
| AddProvenanceRecord | ItemRepository, ProvenanceRepository |
| GetProvenanceHistory | ProvenanceReadPort |
| AttachMedia | ItemRepository, MediaRepository, MediaStoragePort |
| RemoveMedia | MediaRepository, MediaStoragePort |
| ListItemMedia | MediaReadPort |
| SearchItems | SearchPort |
| SearchByClassification | SearchPort |
| SearchByValuation | SearchPort |

This matrix is a preliminary architectural map and should be refined as the domain model evolves.

---

# 22. Use Case to Aggregate Matrix

| Use Case | Collection | Item | Classification | Acquisition | Valuation | Disposal | Provenance | Media |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| CreateCollection | W | | | | | | | |
| RenameCollection | W | | | | | | | |
| CreateItem | R | W | | | | | | |
| UpdateItem | | W | | | | | | |
| ClassifyItem | | W | R/W | | | | | |
| CreateClassification | | | W | | | | | |
| RecordAcquisition | R | W | | W | | | | |
| RecordValuation | | W | | | W | | | |
| DisposeItem | | W | | | | W | | |
| AddProvenanceRecord | | W | | | | | W | |
| AttachMedia | | R/W | | | | | | W |
| RemoveMedia | | R/W | | | | | | W |
| SearchItems | | R | R | R | R | R | R | R |

Legend:

```text
R = Read
W = Write
R/W = Read and Write
```

The matrix makes aggregate coupling visible before implementation.

---

# 23. Use Case Transaction Matrix

| Use Case | Transaction Required | Consistency |
|---|---:|---|
| CreateCollection | Yes | Strong |
| RenameCollection | Yes | Strong |
| CreateItem | Yes | Strong |
| UpdateItem | Yes | Strong |
| ClassifyItem | Yes | Strong |
| CreateClassification | Yes | Strong |
| RecordAcquisition | Yes | Strong within defined boundary |
| RecordValuation | Yes | Strong |
| DisposeItem | Yes | Strong |
| AddProvenanceRecord | Yes | Strong |
| AttachMedia | Domain transaction + storage coordination | Defined explicitly |
| RemoveMedia | Domain transaction + storage coordination | Defined explicitly |
| SearchItems | No | Eventual/read consistency |
| SearchByClassification | No | Eventual/read consistency |
| SearchByValuation | No | Eventual/read consistency |

The final transaction architecture must be established before infrastructure implementation.

---

# 24. Event Map

Representative domain events include:

```text
CollectionCreated
CollectionRenamed

ItemCreated
ItemUpdated
ItemClassified
ItemTransferred
ItemDisposed

ClassificationCreated
ClassificationChanged

AcquisitionRecorded
ValuationRecorded
DisposalRecorded
ProvenanceRecorded

MediaAttached
MediaRemoved
```

Events should be generated by domain behavior rather than by infrastructure repositories.

---

# 25. Event Consumer Map

Potential consumers include:

```text
ItemCreated
    ├── Search Projection
    └── Audit Projection

ItemUpdated
    ├── Search Projection
    └── Audit Projection

ItemClassified
    └── Search Projection

ItemDisposed
    ├── Search Projection
    ├── Reporting Projection
    └── Integration Publisher

ValuationRecorded
    ├── Search Projection
    └── Reporting Projection

ProvenanceRecorded
    └── Historical Projection
```

These are architectural candidates, not mandatory implementations.

---

# 26. Side-Effect Classification

Side effects are classified as:

### Immediate transactional

Required for the correctness of the current use case.

Example:

```text
Persist Item state
Persist Acquisition state
```

### Post-transaction asynchronous

Can occur after successful commit.

Example:

```text
Update search index
Publish integration event
Generate reporting projection
```

### External best-effort

May fail independently and require retry mechanisms.

Example:

```text
External notification
External synchronization
```

The classification must be explicit for each future integration.

---

# 27. Failure Boundaries

A failure in a core domain operation should normally abort the current transaction.

Example:

```text
DisposeItem
    │
    ├── domain rule fails
    │
    └── transaction aborted
```

A failure in a secondary projection should not normally invalidate the already committed domain transaction.

Example:

```text
ItemDisposed
    │
    ▼
Search Projection
    │
    └── failure
         │
         ▼
      Retry
```

This separation is critical for resilient architecture.

---

# 28. Idempotency

Operations that may be retried asynchronously should have explicit idempotency considerations.

Potential candidates:

```text
ImportCollection
PublishIntegrationEvent
UpdateSearchProjection
AttachExternalMedia
SynchronizeExternalData
```

The Application and Infrastructure layers must define idempotency strategies.

The Domain should not be polluted with infrastructure retry mechanisms.

---

# 29. Concurrency Considerations

Commands modifying the same aggregate may execute concurrently.

The persistence architecture must eventually define mechanisms such as:

- optimistic concurrency;
- version numbers;
- conflict detection;
- retry policies.

The Domain model should expose enough identity/state information to support these mechanisms without becoming dependent on a particular persistence technology.

---

# 30. Authorization Interaction Map

A typical command flow is:

```text
Request
  │
  ▼
Authentication
  │
  ▼
Authorization
  │
  ▼
Application Use Case
  │
  ▼
Domain
```

Authorization should happen before expensive domain processing where appropriate.

However, business-level authorization rules must remain distinguishable from technical access-control mechanisms.

---

# 31. Audit Interaction

Audit information should preferably be derived from explicit application/domain events.

Example:

```text
ItemUpdated
     │
     ▼
Audit Handler
     │
     ▼
Audit Store
```

The aggregate should not contain code such as:

```text
auditRepository.save(...)
```

because that would couple business behavior to infrastructure.

---

# 32. Observability Interaction

Application boundaries provide natural points for:

- tracing;
- metrics;
- execution timing;
- failure measurement.

Example:

```text
Command
   │
   ▼
Use Case
   │
   ├── start trace
   ├── execute
   └── record result
```

The Domain remains independent from the observability mechanism.

---

# 33. Use Case Naming Rules

Use case names should express user/system intent.

Preferred:

```text
CreateItem
RecordAcquisition
RecordValuation
DisposeItem
TransferItem
ClassifyItem
SearchItems
```

Avoid technical names:

```text
InsertItem
UpdateItemTable
ExecuteItemCommand
PersistItem
CallSearchDatabase
```

Intent-oriented naming improves traceability between:

```text
Requirement
   ↓
Use Case
   ↓
Domain Behavior
   ↓
Event
```

---

# 34. Traceability Model

Each use case should eventually be traceable to:

```text
Requirement
    ↓
Application Use Case
    ↓
Domain Rule
    ↓
Aggregate / Domain Service
    ↓
Domain Event
    ↓
Infrastructure Reaction
    ↓
Observable Outcome
```

Example:

```text
Requirement:
An item may only be disposed when it is in a disposable state.

        ↓

DisposeItem

        ↓

Item.canBeDisposed()

        ↓

Item aggregate invariant

        ↓

ItemDisposed

        ↓

Search projection update

        ↓

Item no longer appears as actively owned
```

This is the level of traceability expected from the architecture model.

---

# 35. Application Boundary Rules

Every application use case should:

1. have one clearly identified entry point;
2. define its input;
3. define its output;
4. identify required ports;
5. identify participating domain components;
6. define transaction scope;
7. define domain events;
8. define relevant side effects;
9. define failure behavior.

This makes each use case independently understandable.

---

# 36. Use Case Complexity Rule

A use case should remain small enough that its orchestration can be understood as a sequence.

If a use case starts containing:

```text
complex business calculations
many conditional business rules
state transition rules
historical validation
classification logic
valuation logic
```

then those responsibilities should be reconsidered for extraction into:

- Aggregate behavior;
- Domain Service;
- Specification;
- Policy;
- Value Object.

The Application layer is not a replacement for the Domain.

---

# 37. Dependency Hotspots

The following areas are expected to have higher architectural coupling:

```text
Item
Collection
Classification
Acquisition
Disposal
Search
```

They should receive additional architectural scrutiny because they participate in multiple use cases.

In particular:

```text
Item
  ↕
Collection
  ↕
Classification
  ↕
Acquisition
  ↕
Disposal
```

should not evolve into a single giant aggregate or service merely because these concepts interact frequently.

---

# 38. Architectural Risk Areas

The following risks should be monitored:

### R1 — Application logic becoming business logic

Mitigation:

Move invariant/business decisions into Domain.

### R2 — Aggregate coupling

Mitigation:

Use aggregate identities and application orchestration.

### R3 — Search becoming authoritative

Mitigation:

Maintain transactional domain state as source of truth.

### R4 — Infrastructure leakage

Mitigation:

Use ports and adapters.

### R5 — Event proliferation

Mitigation:

Create events only for meaningful domain facts.

### R6 — Shared module growth

Mitigation:

Require explicit justification for shared concepts.

### R7 — Distributed transaction pressure

Mitigation:

Review aggregate boundaries before introducing distributed transactions.

---

# 39. Preliminary Complete Use Case Map

```text
COLLECTION
├── CreateCollection
├── RenameCollection
├── GetCollection
└── ListCollections

ITEM
├── CreateItem
├── UpdateItem
├── GetItem
└── ListCollectionItems

CLASSIFICATION
├── CreateClassification
├── ClassifyItem
└── GetClassificationTree

ACQUISITION
├── RecordAcquisition
└── GetAcquisitionHistory

VALUATION
├── RecordValuation
├── GetCurrentValuation
└── GetValuationHistory

DISPOSAL
├── DisposeItem
└── GetDisposalHistory

PROVENANCE
├── AddProvenanceRecord
└── GetProvenanceHistory

MEDIA
├── AttachMedia
├── RemoveMedia
└── ListItemMedia

SEARCH
├── SearchItems
├── SearchByClassification
└── SearchByValuation

WORKFLOWS
├── AcquireItemIntoCollection
├── TransferItem
├── DisposeItem
└── ImportCollection
```

This list is a working architectural baseline and should be reconciled with the final domain model before implementation.

---

# 40. Architecture Verification Checklist

Before implementation, each use case should be reviewed against:

- [ ] Application entry point identified.
- [ ] Command or Query classification established.
- [ ] Participating aggregate identified.
- [ ] Aggregate ownership respected.
- [ ] Cross-aggregate interactions identified.
- [ ] Domain Service involvement identified.
- [ ] Specifications identified.
- [ ] Required ports identified.
- [ ] Transaction boundary identified.
- [ ] Domain events identified.
- [ ] Asynchronous side effects identified.
- [ ] Failure boundary identified.
- [ ] Idempotency requirement identified where applicable.
- [ ] Read model requirement identified where applicable.
- [ ] Authorization boundary identified.
- [ ] Infrastructure dependency isolated.
- [ ] Delivery dependency isolated.
- [ ] Traceability to domain rules established.

---

# 41. Architectural Decision

CollectionHub adopts a **use-case-centric interaction model** in which every application operation explicitly defines its path through the architecture.

The resulting pattern is:

```text
                     USER / SYSTEM INTENT
                              │
                              ▼
                    ┌─────────────────┐
                    │ Application     │
                    │ Use Case        │
                    └────────┬────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
          Domain Behavior          Application Ports
                 │                       │
                 ▼                       ▼
            Aggregates             Infrastructure
                 │
                 ▼
          Domain Events
                 │
                 ▼
      Secondary / External Effects
```

The most important architectural distinction is:

> **The Application layer determines how a use case is executed; the Domain determines whether the requested business behavior is valid.**

This map establishes the interaction baseline required for the next architectural stage.

---

# 42. Next Architectural Step

The next document should define the concrete infrastructure requirements derived from this interaction map:

```text
29_INFRASTRUCTURE_COMPONENTS_AND_ADAPTERS.md
```

That document will translate the ports and interaction requirements identified here into the infrastructure architecture, including:

- persistence adapters;
- repository implementations;
- transaction management;
- event publishing;
- message handling;
- search infrastructure;
- media storage;
- external service adapters;
- clock and identity providers;
- infrastructure boundaries;
- adapter responsibilities;
- infrastructure dependency rules.

The sequence is therefore:

```text
24  Architectural Boundaries
25  Architectural Components
26  Application / Domain Modules
27  Component Interactions
28  Application Use Case Interaction Map
29  Infrastructure Components and Adapters
```

This preserves the intended architecture-first progression: **boundaries → components → modules → interactions → use cases → infrastructure**.