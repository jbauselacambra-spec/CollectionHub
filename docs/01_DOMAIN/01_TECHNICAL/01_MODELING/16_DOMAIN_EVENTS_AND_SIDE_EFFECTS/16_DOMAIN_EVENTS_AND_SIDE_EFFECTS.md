# CollectionHub — Domain Events and Side Effects

> **Phase:** 2.2 — Domain Modeling  
> **Artifact:** 16 — Domain Events and Side Effects  
> **Status:** Draft / Design Baseline  
> **Scope:** Domain Layer / Application Boundary  
> **Depends on:** Domain Concept Inventory, Domain Glossary, Domain Invariants, Domain Aggregates, Domain Use Cases, Application Workflows, Domain Policies and Decision Tables

---

# 1. Purpose

This document defines the **domain events** emitted by CollectionHub and the **side effects** that may result from those events.

The objective is to establish a clear distinction between:

- domain state changes;
- facts that become true as a consequence of those changes;
- domain events representing those facts;
- application-level reactions;
- infrastructure-level side effects.

The design must prevent external concerns from leaking into the domain model.

The domain should express:

> **What happened.**

The application layer should determine:

> **What the system should do because it happened.**

Infrastructure should determine:

> **How that reaction is technically executed.**

---

# 2. Core Principle

A Domain Event represents a fact.

It should therefore be expressed in the past tense.

Examples:

```text
CollectionCreated
ItemAddedToCollection
ItemRemovedFromCollection
CollectionArchived
CollectionReactivated
CollectionDeleted
CollectionMetadataUpdated
```

The event does not represent an instruction.

Prefer:

```text
ItemAddedToCollection
```

over:

```text
AddItemToCollection
```

The first is a fact.

The second is a command.

---

# 3. Domain Event Definition

A domain event should contain enough information to identify and understand the domain occurrence without requiring the consumer to reconstruct the event from persistence state.

Conceptually:

```text
DomainEvent
├── eventId
├── occurredAt
├── aggregateId
├── aggregateType
├── eventType
├── aggregateVersion
└── domainPayload
```

The exact technical representation is intentionally deferred.

The domain model must not depend on a specific event transport.

---

# 4. Event Characteristics

A CollectionHub domain event should satisfy the following properties.

## 4.1 Immutable

Once emitted, an event represents a historical fact and must not be modified.

---

## 4.2 Past-tense

The event describes something that has already happened.

---

## 4.3 Domain-oriented

The event should use domain terminology.

Avoid technical terminology such as:

```text
DatabaseRowInserted
CollectionTableUpdated
ORMEntitySaved
```

Prefer:

```text
CollectionCreated
```

---

## 4.4 Self-descriptive

A consumer should understand the event's meaning without needing access to internal implementation details.

---

## 4.5 Stable

Event names and semantic meaning should not change casually.

Changing an event contract can affect multiple consumers.

---

# 5. Event Categories

CollectionHub events can be classified into several categories.

| Category | Purpose |
|---|---|
| Lifecycle Events | Represent creation, activation, archival or deletion |
| Membership Events | Represent changes to collection membership |
| Metadata Events | Represent relevant collection metadata changes |
| Classification Events | Represent classification changes |
| Synchronization Events | Represent external synchronization outcomes |
| Conflict Events | Represent unresolved domain conflicts |
| Ownership Events | Represent ownership changes |
| Access Events | Represent relevant visibility/access changes |

Not every category necessarily requires events in the first implementation.

Events should exist because they represent meaningful domain facts, not because event-driven architecture is fashionable.

---

# 6. Aggregate Event Ownership

Domain events should originate from the aggregate that owns the state transition.

Conceptually:

```text
Collection
    │
    ├── addItem()
    │      └── ItemAddedToCollection
    │
    ├── removeItem()
    │      └── ItemRemovedFromCollection
    │
    ├── archive()
    │      └── CollectionArchived
    │
    └── delete()
           └── CollectionDeleted
```

The aggregate is responsible for ensuring that its invariants are satisfied before recording the event.

---

# 7. Event Lifecycle

The conceptual lifecycle is:

```text
Command
   ↓
Domain Policy
   ↓
Aggregate
   ↓
State Mutation
   ↓
Invariant Validation
   ↓
Domain Event Recorded
   ↓
Application Boundary
   ↓
Side Effects
```

The important ordering is:

```text
State change
    +
Invariant satisfaction
    ↓
Domain event
```

A domain event must not represent a state transition that did not successfully occur.

---

# 8. Event: CollectionCreated

## Meaning

A new collection has been successfully created.

## Aggregate

`Collection`

## Trigger

Successful collection creation.

## Payload

Conceptually:

```text
collectionId
ownerId
name
initialState
occurredAt
```

Only domain-relevant information should be included.

## Possible Side Effects

| Side effect | Layer |
|---|---|
| Persist collection | Application / Infrastructure |
| Update search index | Infrastructure |
| Create audit entry | Application / Infrastructure |
| Notify owner | Application / Infrastructure |
| Update projections | Application / Infrastructure |

The domain event itself must not perform these operations.

---

# 9. Event: CollectionMetadataUpdated

## Meaning

Relevant collection metadata has changed.

## Aggregate

`Collection`

## Trigger

A valid metadata update satisfying all applicable policies and invariants.

## Payload

Conceptually:

```text
collectionId
changedAttributes
occurredAt
```

The event should not necessarily contain the entire collection.

Only the information required by consumers should be included.

## Possible Side Effects

- update search projections;
- update read models;
- invalidate caches;
- create audit information;
- synchronize relevant external representations.

---

# 10. Event: ItemAddedToCollection

## Meaning

An item has successfully become a member of a collection.

## Aggregate

`Collection`

## Trigger

Successful membership addition.

## Payload

Conceptually:

```text
collectionId
itemId
position
occurredAt
```

Additional attributes should only be included if they have explicit domain meaning.

## Possible Side Effects

| Side effect | Responsibility |
|---|---|
| Persist collection | Infrastructure |
| Update collection projection | Application |
| Update item/collection read model | Application |
| Update search index | Infrastructure |
| Generate audit entry | Application |
| Trigger external synchronization | Application |

---

# 11. Event: ItemRemovedFromCollection

## Meaning

An item has ceased to be a member of a collection.

## Aggregate

`Collection`

## Payload

Conceptually:

```text
collectionId
itemId
previousPosition
occurredAt
```

## Possible Side Effects

- update read models;
- update search projections;
- update audit information;
- synchronize external representations where applicable.

---

# 12. Event: ItemReorderedInCollection

## Meaning

The ordering of collection membership has changed.

## Aggregate

`Collection`

## Payload

Conceptually:

```text
collectionId
itemId
previousPosition
newPosition
occurredAt
```

## Important Constraint

This event should only exist if ordering has meaningful domain semantics.

If ordering exists purely for UI presentation, an event may not be necessary at the domain level.

This distinction must be preserved.

---

# 13. Event: CollectionArchived

## Meaning

A collection has successfully transitioned to the archived state.

## Aggregate

`Collection`

## Payload

```text
collectionId
previousState
occurredAt
```

## Possible Side Effects

- update read models;
- remove collection from active indexes;
- notify relevant consumers;
- update analytics;
- initiate external synchronization.

---

# 14. Event: CollectionReactivated

## Meaning

An archived collection has successfully returned to an active state.

## Aggregate

`Collection`

## Payload

```text
collectionId
previousState
occurredAt
```

## Possible Side Effects

- restore active projections;
- update search index;
- synchronize external representations;
- invalidate cached state.

---

# 15. Event: CollectionDeleted

## Meaning

A collection has successfully transitioned to its deleted domain state.

## Aggregate

`Collection`

## Payload

```text
collectionId
previousState
occurredAt
```

## Important Distinction

This event means:

```text
The collection is logically deleted.
```

It does not necessarily mean:

```text
All physical records have been permanently erased.
```

Physical deletion remains an infrastructure concern unless explicitly modeled as a domain behavior.

---

# 16. Event: CollectionOwnershipTransferred

If CollectionHub supports ownership transfer, the corresponding domain event is:

```text
CollectionOwnershipTransferred
```

## Payload

```text
collectionId
previousOwnerId
newOwnerId
occurredAt
```

## Preconditions

The ownership transition must satisfy all applicable ownership policies and invariants.

## Possible Side Effects

- update access projections;
- invalidate authorization caches;
- notify previous owner;
- notify new owner;
- update audit records.

---

# 17. Event: CollectionVisibilityChanged

If visibility is part of the domain model, changes should produce:

```text
CollectionVisibilityChanged
```

## Payload

```text
collectionId
previousVisibility
newVisibility
occurredAt
```

## Possible Side Effects

- update access projections;
- update search indexing;
- invalidate caches;
- trigger notifications;
- synchronize external representations.

---

# 18. Event: ItemClassificationChanged

If item classification is a domain concern:

```text
ItemClassificationChanged
```

## Payload

```text
itemId
previousClassification
newClassification
occurredAt
```

Depending on the classification model, the event may instead contain:

```text
addedClassifications
removedClassifications
```

The final shape should follow the domain semantics rather than persistence structure.

---

# 19. Event: ExternalItemLinked

If CollectionHub associates a domain item with an external identity:

```text
ExternalItemLinked
```

## Meaning

A domain item has been successfully linked to an external representation.

## Payload

```text
itemId
externalSource
externalIdentity
occurredAt
```

## Possible Side Effects

- update synchronization mappings;
- schedule synchronization;
- update external-source projections;
- generate audit information.

---

# 20. Event: SynchronizationCompleted

If synchronization is a meaningful domain process, the domain may represent its completion through:

```text
SynchronizationCompleted
```

## Payload

Conceptually:

```text
synchronizationId
source
entityId
result
occurredAt
```

Possible result values:

```text
NO_CHANGES
UPDATED
IMPORTED
MERGED
```

The exact result model must be aligned with the synchronization domain model.

---

# 21. Event: SynchronizationConflictDetected

When synchronization cannot be resolved automatically:

```text
SynchronizationConflictDetected
```

## Payload

```text
conflictId
entityId
source
conflictType
occurredAt
```

The event indicates:

> A conflict exists.

It does not prescribe how the conflict must be resolved.

---

# 22. Event: SynchronizationConflictResolved

When an existing conflict is successfully resolved:

```text
SynchronizationConflictResolved
```

## Payload

```text
conflictId
resolution
resolvedAt
```

Possible resolution strategies:

```text
LOCAL_WINS
EXTERNAL_WINS
MERGED
MANUAL_DECISION
```

Only values supported by the actual domain model should be retained.

---

# 23. Event Naming Convention

Domain event names should follow:

```text
<DomainConcept><PastTenseAction>
```

Examples:

```text
CollectionCreated
CollectionArchived
CollectionDeleted
ItemAddedToCollection
ItemRemovedFromCollection
CollectionOwnershipTransferred
CollectionVisibilityChanged
SynchronizationConflictDetected
```

Avoid:

```text
CreateCollectionEvent
CollectionCreateEvent
HandleCollectionCreated
PublishCollectionCreated
CollectionWasCreatedMessageV2
```

Technical suffixes should not contaminate domain terminology.

---

# 24. Commands vs Events

This distinction must remain explicit.

| Command | Domain Event |
|---|---|
| Intent | Fact |
| Future-oriented | Past-oriented |
| Requests an action | Reports completed action |
| May be rejected | Represents successful occurrence |
| Usually initiated by actor/system | Produced by domain behavior |
| Mutable input | Immutable fact |

Example:

```text
AddItemToCollection
```

is a command.

```text
ItemAddedToCollection
```

is an event.

---

# 25. Domain Event vs Integration Event

These concepts must not be conflated.

## Domain Event

Represents a meaningful occurrence inside the domain model.

Example:

```text
ItemAddedToCollection
```

## Integration Event

Represents information intentionally exposed to another bounded context or external system.

Example:

```text
CollectionItemAddedIntegrationEvent
```

The integration event may be derived from a domain event.

Conceptually:

```text
Domain Event
     ↓
Application / Integration Mapping
     ↓
Integration Event
     ↓
Message Broker / External System
```

The domain must not depend on the integration transport.

---

# 26. Domain Events and Aggregate Boundaries

An aggregate should primarily emit events about state changes that it owns.

For example:

```text
Collection
    └── ItemAddedToCollection
```

rather than:

```text
Collection
    └── ExternalSearchIndexUpdated
```

The second event is infrastructure-oriented and does not belong to the domain aggregate.

---

# 27. Side Effects

A side effect is an operation that occurs because of a domain event but is not itself the domain state transition.

Examples:

```text
Domain Event
    ↓
Send notification
Update search index
Update projection
Invalidate cache
Publish integration message
Write audit record
Schedule synchronization
```

These operations must remain outside the core domain model.

---

# 28. Side Effect Classification

| Side Effect | Domain | Application | Infrastructure |
|---|---:|---:|---:|
| Validate invariant | ✓ | | |
| Change aggregate state | ✓ | | |
| Record domain event | ✓ | | |
| Coordinate workflow | | ✓ | |
| Trigger integration | | ✓ | ✓ |
| Persist aggregate | | ✓ | ✓ |
| Send email | | ✓ | ✓ |
| Publish message | | ✓ | ✓ |
| Update search index | | ✓ | ✓ |
| Cache invalidation | | ✓ | ✓ |
| HTTP request | | | ✓ |
| SQL execution | | | ✓ |

---

# 29. Critical Rule — Event Emission Must Follow Successful State Change

Incorrect:

```text
Receive command
   ↓
Publish ItemAddedToCollection
   ↓
Attempt state mutation
   ↓
Mutation fails
```

This creates a false domain fact.

Correct:

```text
Receive command
   ↓
Evaluate policy
   ↓
Mutate aggregate
   ↓
Validate invariants
   ↓
Record event
   ↓
Persist state + event
   ↓
Process side effects
```

The event must never claim that something happened when the domain state did not successfully transition.

---

# 30. Atomicity Consideration

When an aggregate mutation and its domain events are persisted, CollectionHub should preserve consistency between:

```text
Domain State
+
Domain Events
```

Conceptually:

```text
Transaction
├── Persist aggregate state
└── Persist domain events
```

Both must succeed together from the perspective of the domain operation.

If an event is lost after the state change, downstream side effects may never occur.

If an event is published without the state change, consumers may observe a false fact.

This problem must be addressed by the application/infrastructure architecture.

---

# 31. Transactional Outbox Consideration

If CollectionHub eventually requires reliable asynchronous event publication, a **transactional outbox** is a strong candidate.

Conceptually:

```text
Application Transaction
│
├── Aggregate state
│
└── Outbox event
        ↓
   Outbox Processor
        ↓
   Message Broker
        ↓
   External Consumers
```

The domain should not know that an outbox exists.

The outbox is an infrastructure/application mechanism for reliably delivering domain-derived integration messages.

---

# 32. Event Ordering

Consumers may require ordering guarantees.

CollectionHub should define ordering semantics explicitly when required.

Possible ordering scope:

```text
Per aggregate
Per collection
Per item
Global
```

Global ordering should not be assumed unless explicitly required.

A reasonable default is:

> Events belonging to the same aggregate should preserve their logical order.

---

# 33. Event Versioning

Events are contracts.

If an event changes incompatibly, a new version should be considered.

Example:

```text
ItemAddedToCollection.v1
ItemAddedToCollection.v2
```

However, versioning should not be introduced mechanically.

First evaluate whether:

- the existing event can remain backward-compatible;
- a new event type is clearer;
- the event can remain domain-internal;
- only the integration representation needs versioning.

---

# 34. Event Payload Design

Event payloads should contain:

1. Event identity.
2. Aggregate identity.
3. Occurrence timestamp.
4. Aggregate version where useful.
5. Relevant domain facts.

Avoid including:

- database-specific identifiers;
- ORM state;
- HTTP request objects;
- authentication tokens;
- UI state;
- infrastructure metadata;
- complete serialized aggregates unless explicitly required.

The payload should communicate meaning rather than implementation.

---

# 35. Event Metadata

Potential technical metadata includes:

```text
eventId
occurredAt
aggregateId
aggregateType
aggregateVersion
correlationId
causationId
```

The first five have clear architectural value.

`correlationId` and `causationId` are primarily application/infrastructure concerns and should not become domain concepts merely because they are useful operationally.

---

# 36. Event Correlation

For workflows involving several domain events, correlation may be required.

Example:

```text
CreateCollection
    ↓
CollectionCreated
    ↓
ScheduleExternalSynchronization
    ↓
SynchronizationCompleted
```

The application layer may correlate these events as part of a larger workflow.

The domain itself should remain unaware of workflow orchestration infrastructure.

---

# 37. Side Effect Reliability

Side effects must be classified according to their reliability requirements.

## Critical

Failure may compromise business correctness.

These should generally be part of the same transactional boundary or represented through durable mechanisms.

## Recoverable

Failure can be retried without changing the domain truth.

Examples:

- search indexing;
- cache invalidation;
- notifications.

## Optional

Failure does not affect domain correctness.

Examples:

- analytics;
- telemetry;
- non-critical notifications.

---

# 38. Idempotency of Side Effects

Event consumers should preferably be idempotent.

For example:

```text
ItemAddedToCollection
```

may be delivered twice.

The side effect should not produce:

```text
two duplicate notifications
```

unless duplicate delivery is intentionally meaningful.

Possible strategies include:

```text
eventId-based deduplication
aggregateVersion checks
idempotency keys
consumer-side state
```

This belongs to the application/infrastructure architecture.

---

# 39. Failure Model

A side effect failure must not automatically imply that the original domain event was false.

Example:

```text
CollectionCreated
    ↓
Email notification fails
```

The collection was still created.

Therefore:

```text
Domain fact ≠ Side effect success
```

The architecture must provide retry/recovery semantics for the side effect.

---

# 40. Event Processing Policy

A generic event-processing flow is:

```text
Domain Event
    ↓
Identify interested handlers
    ↓
Execute side effect
    ↓
Success?
 ┌──┴──┐
Yes   No
 ↓     ↓
Done  Retry / Dead Letter / Recovery
```

The specific mechanism remains outside the domain layer.

---

# 41. Event-to-Side-Effect Matrix

| Domain Event | Primary Side Effects | Criticality |
|---|---|---|
| CollectionCreated | Persist / projections / indexing | High |
| CollectionMetadataUpdated | Projections / indexing / cache | Medium |
| ItemAddedToCollection | Projections / indexing / synchronization | High |
| ItemRemovedFromCollection | Projections / indexing | Medium |
| ItemReorderedInCollection | Projection update | Medium |
| CollectionArchived | Projection / indexing | Medium |
| CollectionReactivated | Projection / indexing | Medium |
| CollectionDeleted | Projection / indexing / cleanup | High |
| CollectionOwnershipTransferred | Access projection / notifications | High |
| CollectionVisibilityChanged | Access projection / indexing | High |
| ItemClassificationChanged | Classification projection / indexing | Medium |
| ExternalItemLinked | Synchronization scheduling | Medium |
| SynchronizationCompleted | Projection / audit | Medium |
| SynchronizationConflictDetected | Conflict workflow / notification | High |
| SynchronizationConflictResolved | Projection / synchronization | High |

The exact criticality classification should be validated against operational requirements.

---

# 42. Event-to-Use-Case Traceability

Domain events should be traceable back to the use cases that can produce them.

| Use Case | Domain Event |
|---|---|
| Create Collection | CollectionCreated |
| Update Collection | CollectionMetadataUpdated |
| Add Item | ItemAddedToCollection |
| Remove Item | ItemRemovedFromCollection |
| Reorder Item | ItemReorderedInCollection |
| Archive Collection | CollectionArchived |
| Reactivate Collection | CollectionReactivated |
| Delete Collection | CollectionDeleted |
| Transfer Ownership | CollectionOwnershipTransferred |
| Change Visibility | CollectionVisibilityChanged |
| Classify Item | ItemClassificationChanged |
| Link External Item | ExternalItemLinked |
| Synchronize | SynchronizationCompleted |
| Resolve Conflict | SynchronizationConflictResolved |

Only use cases that actually exist in the finalized application model should remain in this table.

---

# 43. Event-to-Invariant Relationship

An event should only be emitted after the relevant invariants have been satisfied.

Example:

```text
ItemAddedToCollection
```

implies that, at the moment of emission:

```text
Collection is valid
Item is eligible
Collection is mutable
Actor is authorized
Duplicate rule is satisfied
Capacity rule is satisfied
```

The event is therefore a consequence of successful domain validation.

---

# 44. Events Must Not Become Hidden Commands

A dangerous design is:

```text
ItemAddedToCollection
    ↓
"Automatically add another item"
```

unless that behavior is an explicitly modeled business rule.

Events should not become an uncontrolled mechanism for hidden state mutation.

If a business process intentionally reacts to an event and causes another domain command, that relationship should be explicit in the application workflow.

---

# 45. Event Storming Alignment

The event inventory should be compatible with Event Storming concepts:

```text
Command
   ↓
Policy / Decision
   ↓
Aggregate
   ↓
Domain Event
   ↓
Read Model / Policy / External Process
```

This document formalizes the event side of that model.

---

# 46. Initial Event Inventory

The initial CollectionHub event vocabulary is:

```text
CollectionCreated
CollectionMetadataUpdated
ItemAddedToCollection
ItemRemovedFromCollection
ItemReorderedInCollection
CollectionArchived
CollectionReactivated
CollectionDeleted
CollectionOwnershipTransferred
CollectionVisibilityChanged
ItemClassificationChanged
ExternalItemLinked
SynchronizationCompleted
SynchronizationConflictDetected
SynchronizationConflictResolved
```

This is a **candidate vocabulary**, not an irreversible implementation contract.

Events should be removed if they do not represent meaningful domain facts.

Events should be added when the domain model identifies a significant state transition not represented here.

---

# 47. Events That Should NOT Exist

The following are examples of events that should not belong to the domain event model:

```text
DatabaseRecordInserted
DatabaseRecordUpdated
CollectionRepositorySaved
SearchIndexUpdated
CacheInvalidated
HttpRequestCompleted
EmailSent
KafkaMessagePublished
```

These are technical or application/infrastructure facts.

They should not pollute the domain vocabulary.

---

# 48. Side Effects That Must Remain Outside the Domain

The domain must not directly execute:

```text
HTTP requests
SQL queries
emails
push notifications
message broker publishing
search indexing
cache invalidation
file storage
external API calls
background jobs
```

The domain can produce facts that justify those operations.

The application/infrastructure layers execute them.

---

# 49. Recommended Architectural Boundary

The target architecture is:

```text
                    DOMAIN
┌──────────────────────────────────────────┐
│                                          │
│  Aggregates                              │
│  Value Objects                           │
│  Domain Services                         │
│  Policies                                │
│  Invariants                              │
│  Domain Events                           │
│                                          │
└──────────────────┬───────────────────────┘
                   │
                   │ Domain Events
                   ▼
              APPLICATION
┌──────────────────────────────────────────┐
│                                          │
│  Event Handlers                          │
│  Application Workflows                   │
│  Transaction Coordination                │
│  Integration Coordination                │
│                                          │
└──────────────────┬───────────────────────┘
                   │
                   ▼
             INFRASTRUCTURE
┌──────────────────────────────────────────┐
│                                          │
│  Database                                │
│  Message Broker                          │
│  Search Engine                           │
│  Cache                                   │
│  External APIs                            │
│  Email / Notifications                   │
│                                          │
└──────────────────────────────────────────┘
```

Dependency direction must remain inward.

---

# 50. Event Design Rules

The following rules become architectural constraints for CollectionHub.

### Rule E-001

Domain events represent completed domain facts.

### Rule E-002

Domain events are immutable.

### Rule E-003

Domain events use domain terminology.

### Rule E-004

Domain events are emitted only after successful state mutation and invariant validation.

### Rule E-005

Aggregates emit events concerning state they own.

### Rule E-006

Domain events must not execute side effects themselves.

### Rule E-007

Integration events are distinct from domain events.

### Rule E-008

Infrastructure-specific events must not pollute the domain vocabulary.

### Rule E-009

Event consumers should be idempotent whenever duplicate delivery is possible.

### Rule E-010

Event contracts must be treated as stable architectural contracts.

---

# 51. Open Questions

The following decisions remain candidates for later refinement:

1. Which domain events must be persisted?
2. Which events are internal-only?
3. Which events become integration events?
4. Which events require transactional guarantees?
5. Which events require ordering guarantees?
6. What is the event retention policy?
7. Are events used for audit purposes?
8. Which events require versioning?
9. What is the retry policy for side effects?
10. Which side effects are critical versus recoverable?
11. Which event consumers must be idempotent?
12. Is an outbox required?
13. Which events cross bounded-context boundaries?
14. Which events are suitable for rebuilding projections?
15. Which events should be considered part of the long-term public integration contract?

These questions should be resolved before finalizing the event-driven infrastructure architecture.

---

# 52. Definition of Done

This artifact is considered sufficiently mature when:

- [ ] Significant domain state transitions have candidate events.
- [ ] Events are named in domain language.
- [ ] Commands and events are clearly distinguished.
- [ ] Domain events and integration events are clearly separated.
- [ ] Event ownership is mapped to aggregates.
- [ ] Event payloads contain domain facts rather than persistence details.
- [ ] Event emission occurs only after successful domain state changes.
- [ ] Side effects are separated from domain behavior.
- [ ] Side effects are classified by responsibility.
- [ ] Event processing failure semantics are identified.
- [ ] Idempotency requirements are identified.
- [ ] Event ordering requirements are identified.
- [ ] Event versioning requirements are identified.
- [ ] Event-to-use-case traceability exists.
- [ ] Event-to-invariant relationships are understood.
- [ ] Infrastructure concerns remain outside the domain.
- [ ] Open event architecture questions are explicitly recorded.

---

# 53. Relationship With Previous Artifacts

This artifact extends the behavioral model established previously:

```text
08_DOMAIN_INVARIANTS
        ↓
09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES
        ↓
11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES
        ↓
12_APPLICATION_USE_CASES_AND_WORKFLOWS
        ↓
15_DOMAIN_POLICIES_AND_DECISION_TABLES
        ↓
16_DOMAIN_EVENTS_AND_SIDE_EFFECTS
```

The resulting chain is:

```text
Invariant
    ↓
Aggregate
    ↓
Policy
    ↓
Decision
    ↓
State Change
    ↓
Domain Event
    ↓
Application Reaction
    ↓
Infrastructure Side Effect
```

This chain provides a complete behavioral explanation of how CollectionHub moves from **business rule** to **observable consequence**.

---

# 54. Final Principle

The most important architectural distinction established by this artifact is:

> **The domain owns facts and decisions. The application owns orchestration. Infrastructure owns technical side effects.**

Therefore:

```text
Domain
  → decides and records what happened

Application
  → decides what to do because it happened

Infrastructure
  → executes how that reaction is technically performed
```

This separation is essential for keeping CollectionHub maintainable, testable and independent of infrastructure choices.

---

**Status:** Domain Events and Side Effects baseline established.

**Next step:** continue with the next domain-model artifact, using the accumulated concepts, invariants, aggregates, policies and events to identify the remaining domain boundaries and architectural contracts before implementation begins.