# CollectionHub — Domain Use Cases and Application Services

## 1. Purpose

This document defines the **use cases and application-level orchestration model** of CollectionHub.

It builds on the domain model established in:

- `07_DOMAIN_COMMANDS_AND_EVENTS.md`
- `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`
- `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`
- `10_DOMAIN_SERVICES_AND_POLICIES.md`

The purpose is to answer:

> **What meaningful operations can be performed in CollectionHub, who or what initiates them, and how does the application coordinate the domain objects involved?**

This document establishes the boundary between:

- domain behavior;
- application orchestration;
- infrastructure.

It does not define:

- HTTP endpoints;
- REST resources;
- database repositories;
- ORM implementations;
- message broker configuration;
- UI workflows;
- framework-specific application services.

---

# 2. Application Layer Philosophy

The application layer exists to **orchestrate domain behavior**, not to contain business rules that belong to the domain.

Its responsibility is to coordinate:

```text id="g5q8kd"
External Trigger
      ↓
Application Use Case
      ↓
Load required domain objects
      ↓
Invoke Domain Policies / Services
      ↓
Invoke Aggregate behavior
      ↓
Persist resulting changes
      ↓
Publish / expose resulting domain facts
```

The application layer therefore acts as a boundary between the outside world and the domain.

---

# 3. Domain vs Application Responsibility

The distinction must remain explicit.

## Domain

The domain decides:

- whether an operation is valid;
- whether invariants hold;
- how state changes;
- which business rules apply;
- which domain events result.

## Application

The application decides:

- which domain objects need to participate;
- in what order they are invoked;
- how a use case is coordinated;
- when persistence is requested;
- how the use case result is returned;
- how external concerns are coordinated.

Conceptually:

```text id="x2m7pv"
Application
    |
    | "What needs to happen for this use case?"
    v
Domain
    |
    | "Is this allowed and what does it mean?"
    v
Domain State
```

---

# 4. What Is a Use Case?

A use case represents a meaningful capability of the system from the perspective of an actor or external process.

A use case is not simply a CRUD method.

For example:

```text id="q8v3kc"
Add Item to Collection
```

is a domain-relevant capability.

Whereas:

```text id="r4n7mx"
Update Collection Row
```

is a technical or presentation-oriented operation and should not define the domain.

---

# 5. Use Case Characteristics

A valid CollectionHub use case should have:

- a clear intention;
- a meaningful trigger;
- explicit inputs;
- defined domain participation;
- identifiable success conditions;
- identifiable business failure conditions;
- clear side effects;
- explicit consistency expectations.

---

# 6. Use Case Naming

Use case names should express **business intent**.

Preferred:

```text id="m7q2cx"
Create Collection
Add Item to Collection
Remove Item from Collection
Create Item
Update Item Metadata
```

Avoid technical naming:

```text id="k5r8pn"
POST Collection
Update Collection DTO
Save Item Row
Execute Collection SQL
```

The use case name should remain meaningful even if the technical architecture changes.

---

# 7. Actor Model

A use case may be initiated by:

- a human user;
- an administrator;
- an import process;
- a scheduled process;
- an external system;
- another application process.

The actor is the initiator of the use case, not necessarily the owner of the resulting domain state.

For example:

```text id="c4x8mz"
External Import
      ↓
Create Item
      ↓
Item Aggregate
```

The import process initiates the use case, but Item owns the resulting domain state.

---

# 8. Application Service

An Application Service is a technical application-layer component responsible for executing one or more use cases.

Its responsibility is orchestration.

Conceptually:

```text id="p7m3vk"
Application Service
       |
       +-- load aggregate
       +-- invoke policy
       +-- invoke aggregate
       +-- persist
       +-- publish / return result
```

It must not become a second domain model.

---

# 9. Application Service Anti-Pattern

Avoid:

```text id="h8q4sc"
ApplicationService
    |
    +-- validates business rule
    +-- changes aggregate state directly
    +-- decides lifecycle transition
    +-- calculates business result
```

This creates a domain model split across layers.

Instead:

```text id="t5w2xn"
Application Service
       |
       +-- coordinates
              |
              v
         Domain Model
```

---

# 10. Use Case Inventory

The current model identifies the following primary use cases.

## Collection

```text id="v6m9rp"
UC-COL-001 Create Collection
UC-COL-002 Rename Collection
UC-COL-003 Add Item to Collection
UC-COL-004 Remove Item from Collection
```

Potential future use cases:

```text id="x3q7nk"
Archive Collection
Restore Collection
Update Collection Metadata
Reorder Collection Items
```

These remain provisional until the corresponding domain behavior is fully established.

---

## Item

```text id="a8p4vc"
UC-ITEM-001 Create Item
UC-ITEM-002 Update Item Metadata
UC-ITEM-003 Change Item Classification
```

Potential future use cases:

```text id="j6r2mw"
Archive Item
Restore Item
Merge Items
Split Item
```

These require additional domain analysis before being accepted.

---

# 11. UC-COL-001 — Create Collection

## Intent

Create a new valid Collection.

## Actor

A user or authorized application process.

## Inputs

Conceptually:

```text id="q3x8nk"
CollectionName
CollectionMetadata?
OwnershipContext
```

## Participating Aggregate

```text id="m7v4cz"
Collection
```

## Preconditions

- required collection information is present;
- supplied values satisfy structural requirements;
- ownership context is valid.

## Domain Behavior

The Collection aggregate creates a valid initial state.

## Postconditions

A new Collection exists in its valid initial state.

## Domain Event

```text id="h5q9wp"
CollectionCreated
```

## Consistency

Single aggregate.

---

# 12. UC-COL-002 — Rename Collection

## Intent

Change the name of an existing Collection.

## Actor

User or authorized application process.

## Input

```text id="r6v2mx"
CollectionId
NewCollectionName
```

## Participating Aggregate

```text id="n8k3pd"
Collection
```

## Preconditions

- Collection exists;
- Collection is in a state allowing modification;
- new name satisfies domain rules.

## Domain Behavior

The Collection aggregate validates and applies the new name.

## Postconditions

Collection has the new valid name.

## Domain Event

```text id="c4m7xq"
CollectionRenamed
```

## Consistency

Single aggregate.

---

# 13. UC-COL-003 — Add Item to Collection

This is one of the most important current use cases because it crosses the boundary between Collection and Item.

## Intent

Establish membership of an Item in a Collection.

## Actor

User or application process.

## Inputs

```text id="v7n2kc"
CollectionId
ItemId
MembershipMetadata?
```

## Participating Aggregates

```text id="x5m8qr"
Collection
Item
```

The Item aggregate is referenced by identity.

The Collection aggregate owns the membership change.

---

# 14. Add Item — Preconditions

At application level:

- Collection must be retrievable;
- Item must be retrievable if the domain requires existence verification;
- required command information must be valid.

At domain level:

- Collection must permit modification;
- membership must be valid;
- duplicate membership must not violate domain rules;
- any cross-aggregate compatibility policy must permit the relationship.

---

# 15. Add Item — Orchestration

Conceptually:

```text id="g3x7mp"
AddItemToCollection
        |
        v
Load Collection
        |
        v
Load Item
        |
        v
Evaluate Compatibility Policy
        |
        +---- incompatible ----> reject
        |
        v
Collection.addItem(ItemId)
        |
        +---- invalid ----> reject
        |
        v
Collection changed
        |
        v
ItemAddedToCollection
```

The key principle is:

> **The application coordinates; the Collection aggregate decides whether the membership change itself is valid.**

---

# 16. Add Item — Postconditions

After successful completion:

- Collection contains the membership;
- membership invariants hold;
- Item itself remains unchanged unless another explicit domain rule requires modification;
- `ItemAddedToCollection` is produced.

---

# 17. Add Item — Consistency

The membership invariant must be strongly consistent within Collection.

If compatibility requires information from Item, the policy evaluates the relevant facts before the Collection mutation.

The domain must later determine whether the Item existence check is mandatory or whether an identity reference is sufficient.

---

# 18. UC-COL-004 — Remove Item from Collection

## Intent

Remove an existing membership relationship.

## Inputs

```text id="p5x9md"
CollectionId
ItemId
```

## Participating Aggregate

```text id="w3k7cq"
Collection
```

## Preconditions

- Collection exists;
- membership exists, unless removal is explicitly idempotent;
- Collection permits modification.

## Domain Behavior

Collection removes the membership.

## Postconditions

The membership no longer exists.

## Event

```text id="n6r2vk"
ItemRemovedFromCollection
```

## Important Distinction

```text id="m4q8xc"
Remove Item from Collection
        ≠
Delete Item
```

Removing membership does not modify the Item aggregate.

---

# 19. UC-ITEM-001 — Create Item

## Intent

Create a new collectible Item.

## Inputs

Conceptually:

```text id="x7c3pn"
ItemIdentityInformation
ItemMetadata
Classification?
Provenance?
```

## Participating Aggregate

```text id="r5m8vq"
Item
```

## Preconditions

- required item information is valid;
- identity rules are satisfied;
- supplied classification is valid when present.

## Postconditions

A valid Item exists.

## Event

```text id="k2p7mx"
ItemCreated
```

---

# 20. UC-ITEM-002 — Update Item Metadata

## Intent

Modify descriptive information belonging to an Item.

## Inputs

```text id="q4v8nc"
ItemId
MetadataChanges
```

## Aggregate

```text id="j7m3px"
Item
```

## Preconditions

- Item exists;
- metadata changes satisfy domain rules;
- Item permits modification.

## Domain Behavior

The Item aggregate applies the metadata change.

## Postconditions

Item metadata is valid.

## Event

```text id="w5r9kc"
ItemMetadataUpdated
```

---

# 21. UC-ITEM-003 — Change Item Classification

## Intent

Change the classification of an Item.

## Inputs

```text id="x6m2pv"
ItemId
NewClassification
```

## Aggregate

```text id="n4q7cs"
Item
```

## Preconditions

- Item exists;
- classification is valid;
- Item permits modification.

## Postconditions

Item has the new classification.

## Event

```text id="h8v3mq"
ItemClassificationChanged
```

---

# 22. Use Case Result

A use case should return an application-level result that is meaningful to its caller.

The result may include:

- aggregate identity;
- resulting state;
- relevant domain information;
- domain event references when appropriate.

However, the application result should not necessarily expose the entire aggregate.

---

# 23. Command vs Use Case

A command and a use case are related but different.

### Command

Represents:

> **An intention to perform an operation.**

### Use Case

Represents:

> **The complete application-level behavior required to fulfill that intention.**

Example:

```text id="q8m3rx"
Command:
    AddItemToCollection

Use Case:
    Add Item to Collection
        |
        +-- load Collection
        +-- load Item
        +-- evaluate policy
        +-- invoke Collection
        +-- persist
        +-- publish resulting fact
```

---

# 24. Command Handler vs Application Service

These concepts should not be conflated.

A Command Handler is one possible technical mechanism for executing a command.

An Application Service is an application-layer abstraction around a use case.

The domain model must not depend on either terminology.

The architectural choice will be made later.

---

# 25. Use Case Input Model

Inputs should represent the information required to perform the use case.

They should not necessarily mirror database records.

For example:

```text id="v4x7pk"
AddItemToCollectionInput
    CollectionId
    ItemId
    MembershipMetadata?
```

is preferable conceptually to:

```text id="f8q2mw"
CollectionDatabaseRow
ItemDatabaseRow
MembershipDatabaseRow
```

---

# 26. Input Validation Responsibility

Input validation should be split.

## Application Boundary

Responsible for:

- malformed input;
- missing required transport fields;
- invalid primitive representation.

## Domain

Responsible for:

- business validity;
- invariant protection;
- lifecycle rules;
- semantic relationships.

The application layer must not become the authoritative owner of domain validation.

---

# 27. Use Case Preconditions vs Domain Preconditions

A use case may have application-level preconditions such as:

```text id="k7m2xc"
CollectionId is present
ItemId is present
```

while the domain has semantic preconditions:

```text id="p5r8vq"
Collection can be modified
Membership is valid
Item is compatible
```

The latter must remain protected by the domain.

---

# 28. Application Orchestration Pattern

The generic pattern is:

```text id="n4c7xm"
1. Receive use case input
2. Validate application-level structure
3. Load required aggregates
4. Gather required domain facts
5. Evaluate policies/services
6. Invoke aggregate behavior
7. Persist changed aggregates
8. Publish resulting domain facts
9. Return use case result
```

The exact technical implementation is intentionally deferred.

---

# 29. Read vs Write Use Cases

CollectionHub should distinguish between:

### Command Use Cases

Change domain state.

Examples:

```text id="m8q3vx"
Create Collection
Add Item to Collection
Update Item Metadata
```

### Query Use Cases

Retrieve information without changing domain state.

Examples may eventually include:

```text id="r6k2pn"
Get Collection
Get Item
List Collection Items
Search Items
```

Query modeling will be addressed separately because query responsibilities can differ significantly from command orchestration.

---

# 30. Query Use Cases Are Not Yet Fully Defined

The current documents focus primarily on domain-changing behavior.

This is intentional.

Read-side requirements should be modeled after the domain write model is sufficiently stable.

This prevents query convenience from distorting aggregate boundaries.

---

# 31. Use Case Atomicity

Each use case must define its required consistency level.

For example:

```text id="x5p8mc"
Rename Collection
    → single aggregate
    → immediate consistency
```

Whereas:

```text id="q7v3nk"
Add Item to Collection
    → Collection + Item information
    → membership change belongs to Collection
    → compatibility may be evaluated externally
```

The application must not automatically treat every participating aggregate as one atomic unit.

---

# 32. Use Case Transaction Boundary

Transaction boundaries should be derived from aggregate and consistency semantics.

The use case does not automatically imply:

```text id="j8m4qx"
one transaction = all loaded aggregates
```

Instead:

```text id="c3v7kp"
Domain invariants
      ↓
Aggregate boundaries
      ↓
Required consistency
      ↓
Transaction strategy
```

Technical transaction design belongs to later architecture.

---

# 33. Idempotency

Each use case should explicitly identify whether repeated execution is meaningful.

For example:

### Add Item

Potentially:

```text id="p8x4mq"
Duplicate request
    ↓
Reject
```

or:

```text id="r3n7kc"
Duplicate request
    ↓
Idempotent success
```

The domain must decide which interpretation is correct.

---

# 34. Retry Semantics

The application layer may retry an operation for technical reasons.

However, retry safety depends on domain semantics.

For example:

```text id="m6q2vx"
Technical retry
      ↓
same command
      ↓
domain decision
```

The application layer must not assume that all commands are safely repeatable.

---

# 35. Event Publication

A successful use case may result in domain events.

Conceptually:

```text id="x7c4pn"
Aggregate
    |
    +-- state change
    |
    +-- domain event
            |
            v
       Application Boundary
```

The exact event publication mechanism belongs to the infrastructure/application architecture.

The domain only defines the fact.

---

# 36. Event Failure

A key architectural question remains:

> What happens if domain state is successfully persisted but event publication fails?

This document deliberately does not solve the technical mechanism.

Potential later solutions may include:

- transactional outbox;
- reliable event publication;
- event store;
- message retry.

The important domain principle is:

> **Event publication failure must not redefine whether the domain operation itself occurred.**

---

# 37. Authorization Boundary

Authorization should be evaluated before a use case performs unauthorized domain changes.

However, the domain may still need to protect ownership-related invariants if ownership is a business concept.

The distinction will be formalized later.

---

# 38. Use Case Failure Categories

Use case failures should be distinguished conceptually.

## Input Failure

```text id="w5m8qk"
Malformed or incomplete input
```

## Domain Failure

```text id="f7x2pn"
Business rule violation
```

## Not Found

```text id="k4r9vc"
Required domain object cannot be located
```

## Conflict

```text id="n8q3mx"
Requested operation conflicts with current state
```

## Infrastructure Failure

```text id="p6v2kw"
Database unavailable
External service unavailable
```

These categories should not be collapsed into a generic failure.

---

# 39. Use Case Example — Full Flow

Consider:

```text id="r5m8xq"
Add Item to Collection
```

Full conceptual workflow:

```text
Actor
  |
  v
AddItemToCollection
  |
  v
Application Use Case
  |
  +---- validate input
  |
  +---- load Collection
  |
  +---- load Item
  |
  +---- evaluate compatibility policy
  |          |
  |          +---- reject
  |
  +---- Collection.addItem(ItemId)
  |          |
  |          +---- reject
  |
  +---- persist Collection
  |
  +---- publish ItemAddedToCollection
  |
  v
Success
```

This diagram establishes responsibilities without committing to implementation technology.

---

# 40. Use Case Example — Simple Aggregate Operation

Consider:

```text id="q7c3mv"
Rename Collection
```

The workflow is simpler:

```text
Actor
  |
  v
RenameCollection
  |
  v
Load Collection
  |
  v
Collection.rename(...)
  |
  v
CollectionRenamed
  |
  v
Persist
  |
  v
Success
```

This is a useful contrast with cross-aggregate use cases.

---

# 41. Use Case Example — Item Metadata

```text id="x4m7pn"
UpdateItemMetadata
        |
        v
Load Item
        |
        v
Item.updateMetadata(...)
        |
        v
ItemMetadataUpdated
        |
        v
Persist
        |
        v
Success
```

No Domain Service is required.

This reinforces the principle that the aggregate owns its own behavior.

---

# 42. Application Service Responsibilities

An Application Service may be responsible for:

- receiving use case input;
- coordinating aggregate loading;
- invoking domain policies;
- invoking aggregates;
- managing application-level transaction scope;
- coordinating persistence;
- collecting resulting events;
- returning use case results.

---

# 43. Application Service Non-Responsibilities

An Application Service should not:

- implement collection membership rules;
- decide lifecycle transitions;
- mutate aggregate state directly;
- duplicate domain validation;
- calculate business outcomes that belong to domain objects;
- define domain terminology;
- become a repository;
- become an external API client.

---

# 44. Use Case to Aggregate Mapping

| Use Case | Primary Aggregate | Other Domain Participants |
|---|---|---|
| Create Collection | Collection | — |
| Rename Collection | Collection | — |
| Add Item to Collection | Collection | Item, optional Policy |
| Remove Item from Collection | Collection | — |
| Create Item | Item | — |
| Update Item Metadata | Item | — |
| Change Item Classification | Item | Classification Policy, if required |

The **primary aggregate** is the aggregate whose state change defines the use case outcome.

---

# 45. Use Case to Rule Mapping

| Use Case | Relevant Rules |
|---|---|
| Create Collection | Collection invariants |
| Rename Collection | Collection name rules, lifecycle |
| Add Item | Membership rules, compatibility policy |
| Remove Item | Membership rules, lifecycle |
| Create Item | Item invariants |
| Update Metadata | Metadata rules |
| Change Classification | Classification rules |

This matrix should evolve as the domain becomes more precise.

---

# 46. Use Case to Event Mapping

| Use Case | Expected Domain Event |
|---|---|
| Create Collection | CollectionCreated |
| Rename Collection | CollectionRenamed |
| Add Item to Collection | ItemAddedToCollection |
| Remove Item from Collection | ItemRemovedFromCollection |
| Create Item | ItemCreated |
| Update Item Metadata | ItemMetadataUpdated |
| Change Item Classification | ItemClassificationChanged |

A successful state-changing use case should have an identifiable domain fact when the domain semantics require one.

---

# 47. Use Case Traceability

Each use case should eventually be traceable through:

```text id="w8k3qp"
Actor
  ↓
Use Case
  ↓
Command
  ↓
Aggregate / Policy
  ↓
Invariant
  ↓
State Change
  ↓
Domain Event
```

This provides a strong mechanism for detecting incomplete modeling.

---

# 48. Missing Use Case Detection

If a domain event exists without a plausible use case or domain operation causing it, investigate.

For example:

```text id="c7m2vx"
ItemAddedToCollection
       ↓
What operation caused this?
```

If no operation exists, the model may be incomplete.

Likewise, if a command exists but no use case can execute it:

```text id="q4x8pn"
Command
   ↓
No use case
```

the command may be premature or incorrectly modeled.

---

# 49. Use Case Granularity

A use case should represent a meaningful business capability.

Avoid excessively granular technical use cases such as:

```text id="n7m3kc"
SetCollectionNameField
SetCollectionDescriptionField
SetCollectionImageField
```

when the domain considers these one conceptual operation.

At the same time, avoid giant workflows that combine unrelated business intentions.

---

# 50. Composite Use Cases

Some future operations may involve several domain intentions.

For example:

```text id="p8v4xm"
Import Collection
```

might eventually involve:

```text id="j6r2qn"
Create Items
Resolve Metadata
Create Memberships
```

Whether this becomes one use case or several coordinated processes depends on whether "Import Collection" is itself a meaningful domain capability.

This remains open.

---

# 51. Long-Running Use Cases

Some processes may eventually be long-running.

Examples:

```text id="x5m9vc"
Import external catalog
Synchronize metadata
Reconcile conflicts
```

These should not be modeled as ordinary synchronous use cases if the domain semantics require a process lifecycle.

A future process/workflow model may be necessary.

---

# 52. Application Process vs Domain Aggregate

A long-running process must not automatically become an aggregate.

The distinction is:

```text id="r3q7nk"
Aggregate
    protects immediate consistency

Process
    coordinates behavior over time
```

This distinction may become important if CollectionHub introduces asynchronous ingestion.

---

# 53. Use Case Security

The application layer should establish whether the actor is allowed to invoke a use case.

However, domain ownership invariants must remain protected by the domain if they are business rules.

This creates two potentially separate checks:

```text id="v8m2qc"
Application
    |
    +-- "May this actor invoke this use case?"

Domain
    |
    +-- "Would this operation be valid?"
```

Both may be necessary.

---

# 54. Observability

Observability is not part of domain semantics.

Application-level telemetry may record:

- use case started;
- use case succeeded;
- use case failed;
- duration;
- technical correlation information.

However, instrumentation must not alter domain behavior.

---

# 55. Logging

Similarly, logging belongs to application/infrastructure layers.

The domain should not depend on logging infrastructure to make decisions.

---

# 56. Transaction Coordination

The application layer may define transaction boundaries according to the architecture.

The domain defines what must be consistent.

Therefore:

```text id="m4q8xp"
Domain:
    "These invariants must hold."

Application:
    "These changes must be coordinated."

Infrastructure:
    "This is how the coordination is implemented."
```

This separation must remain explicit.

---

# 57. Application Boundary and Domain Events

Application orchestration may observe domain events after aggregate operations.

However, the event's meaning remains domain-owned.

For example:

```text id="x7p3mc"
Domain:
    ItemAddedToCollection

Application:
    persist / publish / react

Infrastructure:
    message broker / outbox / transport
```

---

# 58. Query Side — Preliminary Direction

Queries should not require aggregate mutation.

For example:

```text id="q8m5vn"
Get Collection
```

may eventually use a read-oriented model rather than loading the complete Collection aggregate.

This should not be decided prematurely.

The important principle is:

> **Read requirements must not distort aggregate boundaries.**

---

# 59. Application Service Naming

Application services should use use-case-oriented names where possible.

Prefer:

```text id="n4x7qp"
CreateCollection
RenameCollection
AddItemToCollection
RemoveItemFromCollection
CreateItem
UpdateItemMetadata
```

over:

```text id="m6r2vc"
CollectionManager
ItemManager
CollectionService
```

This keeps the application layer aligned with actual capabilities.

---

# 60. Application Service Granularity

One application service may coordinate multiple related use cases.

Alternatively, each use case may have its own handler.

Both are valid architectural strategies.

The domain model does not require one particular technical organization.

The important requirement is:

> **Each use case must have an explicit orchestration boundary.**

---

# 61. Use Case Contracts

Each use case should eventually have a stable conceptual contract:

```text id="r7c2mx"
Input
    ↓
Use Case
    ↓
Success Result
    |
    +---- Domain Failure
    +---- Not Found
    +---- Conflict
    +---- Technical Failure
```

These contracts will later influence API and integration design.

---

# 62. Current Use Case Model

The current write-side model can be summarized as:

```text id="k5x8pn"
COLLECTION

Create Collection
      ↓
CollectionCreated

Rename Collection
      ↓
CollectionRenamed

Add Item to Collection
      ↓
ItemAddedToCollection

Remove Item from Collection
      ↓
ItemRemovedFromCollection


ITEM

Create Item
      ↓
ItemCreated

Update Item Metadata
      ↓
ItemMetadataUpdated

Change Item Classification
      ↓
ItemClassificationChanged
```

---

# 63. Current Application Orchestration Model

The preferred orchestration pattern is:

```text id="v3m7cq"
                    +----------------------+
                    |   Application Layer  |
                    |                      |
Actor ─────────────>| Use Case             |
                    |        |             |
                    |        v             |
                    |   Load Aggregates    |
                    |        |             |
                    |        v             |
                    |   Invoke Policies   |
                    |        |             |
                    |        v             |
                    |   Invoke Aggregate  |
                    |        |             |
                    |        v             |
                    |   Persist / Publish |
                    +--------+-------------+
                             |
                             v
                    +----------------------+
                    |     Domain Layer     |
                    +----------------------+
```

---

# 64. Open Questions

## OPEN-UC-001

What is the complete actor model?

---

## OPEN-UC-002

Which use cases are user-facing versus system/integration initiated?

---

## OPEN-UC-003

Does CollectionHub require collection archival?

---

## OPEN-UC-004

Does CollectionHub require item archival?

---

## OPEN-UC-005

Is item deletion a valid business operation?

---

## OPEN-UC-006

Does adding an Item require Item existence validation?

---

## OPEN-UC-007

Should duplicate Add Item operations be rejected or idempotent?

---

## OPEN-UC-008

Does Item classification require a separate policy?

---

## OPEN-UC-009

Does metadata reconciliation constitute a synchronous use case or an asynchronous process?

---

## OPEN-UC-010

Does CollectionHub require explicit long-running import/synchronization use cases?

---

## OPEN-UC-011

What ownership and authorization rules belong to the domain?

---

## OPEN-UC-012

Which query capabilities are core product use cases?

---

# 65. Anti-Patterns

## UC-ANTI-001 — CRUD-First Use Cases

Do not model the application around database CRUD operations.

---

## UC-ANTI-002 — Business Logic in Application Services

Do not place invariant decisions in application services.

---

## UC-ANTI-003 — Application Service as God Object

Avoid:

```text id="x8m4pc"
CollectionApplicationService
    |
    +-- collections
    +-- items
    +-- imports
    +-- metadata
    +-- search
    +-- notifications
    +-- synchronization
```

Use-case responsibilities should remain explicit.

---

## UC-ANTI-004 — Aggregate Mutation From Application Layer

Do not modify aggregate state directly.

Application code invokes domain behavior.

---

## UC-ANTI-005 — Event as Workflow

A domain event is not an application workflow.

---

## UC-ANTI-006 — Query Distorting Domain Model

Do not introduce aggregate relationships solely to make queries convenient.

---

# 66. Traceability to Previous Documents

The current model provides the following chain:

```text id="p6w2qm"
07 Commands & Events
        |
        v
08 Invariants & Business Rules
        |
        v
09 Aggregates
        |
        v
10 Services & Policies
        |
        v
11 Use Cases
```

Each layer adds a different dimension.

The resulting model is:

```text id="m3q8vx"
Command
   ↓
Use Case
   ↓
Application Coordination
   ↓
Policy
   ↓
Aggregate
   ↓
Invariant
   ↓
State Change
   ↓
Event
```

---

# 67. Completion Criteria

This document is considered sufficiently mature when:

- all important write-side capabilities are identified;
- every command maps to a use case or is explicitly marked provisional;
- every state-changing use case has an identifiable domain owner;
- aggregate ownership is explicit;
- cross-aggregate coordination is explicit;
- application responsibilities are separated from domain responsibilities;
- domain failures are distinguishable from technical failures;
- idempotency expectations are identified;
- consistency expectations are identified;
- open questions are recorded.

---

# 68. Final Principle

CollectionHub's application layer should remain intentionally thin.

Its purpose is not to contain the domain.

Its purpose is to **coordinate the domain**.

The desired relationship is:

```text id="q8m4vx"
             APPLICATION
                  |
                  | orchestrates
                  v
+-----------------------------------+
|              DOMAIN               |
|                                   |
|  Policies → Aggregates → Events   |
|       ↑          ↑                |
|       |          |                |
|    decisions   invariants         |
+-----------------------------------+
                  |
                  v
             INFRASTRUCTURE
```

The application layer therefore acts as the **orchestration boundary**, while the domain remains the **authority for business meaning**.

The next modeling step should formalize the workflows behind these use cases in:

`12_APPLICATION_USE_CASES_AND_WORKFLOWS.md`

where each important use case will be decomposed into explicit preconditions, steps, decisions, postconditions, consistency boundaries, failure paths, and resulting domain events.