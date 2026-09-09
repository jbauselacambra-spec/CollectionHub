# CollectionHub — Application Use Cases and Workflows

## 1. Purpose

This document defines the application workflows associated with the use cases identified in:

- `07_DOMAIN_COMMANDS_AND_EVENTS.md`
- `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`
- `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`
- `10_DOMAIN_SERVICES_AND_POLICIES.md`
- `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md`

Its purpose is to describe:

- how each use case starts;
- which inputs it requires;
- which domain objects participate;
- which decisions are made;
- which domain operations are invoked;
- which invariants are protected;
- what happens on success;
- what happens on failure;
- which events result;
- what consistency is required;
- which steps are synchronous or potentially asynchronous.

This document remains implementation-agnostic.

It does not define:

- HTTP endpoints;
- database schemas;
- repository implementations;
- framework-specific handlers;
- message broker configuration;
- UI flows;
- infrastructure retry mechanisms.

---

# 2. Workflow Philosophy

A workflow represents the orchestration required to fulfill a meaningful application use case.

The application layer coordinates the workflow.

The domain remains responsible for deciding whether individual domain operations are valid.

The fundamental relationship is:

    External Trigger
          ↓
    Application Workflow
          ↓
    Domain Operations
          ↓
    Domain State Change
          ↓
    Domain Events

The workflow therefore describes **how the application reaches a domain outcome**, not how the domain itself implements its internal behavior.

---

# 3. Workflow Responsibilities

An application workflow may:

- receive input;
- validate structural input;
- identify required aggregates;
- load domain state;
- retrieve external/domain facts;
- invoke domain policies;
- invoke aggregate behavior;
- coordinate persistence;
- coordinate event publication;
- return an application result.

An application workflow must not:

- bypass aggregate invariants;
- directly mutate domain state;
- duplicate business rules;
- decide domain state transitions;
- embed persistence-specific assumptions.

---

# 4. Workflow Structure

Each workflow follows this conceptual structure:

    Trigger
      ↓
    Input
      ↓
    Preconditions
      ↓
    Load required state
      ↓
    Evaluate required policies
      ↓
    Invoke domain behavior
      ↓
    Persist resulting changes
      ↓
    Publish resulting domain facts
      ↓
    Return result

Failure paths branch from any step where an application or domain constraint can fail.

---

# 5. Workflow Classification

Current workflows are classified into:

## Collection Workflows

- `WF-COL-001 Create Collection`
- `WF-COL-002 Rename Collection`
- `WF-COL-003 Add Item to Collection`
- `WF-COL-004 Remove Item from Collection`

## Item Workflows

- `WF-ITEM-001 Create Item`
- `WF-ITEM-002 Update Item Metadata`
- `WF-ITEM-003 Change Item Classification`

Additional workflows remain provisional until the domain model explicitly requires them.

---

# 6. Workflow States

A workflow can conceptually move through these application states:

    RECEIVED
       ↓
    VALIDATING
       ↓
    LOADING
       ↓
    EVALUATING
       ↓
    EXECUTING
       ↓
    PERSISTING
       ↓
    COMPLETED

Failure can occur from any stage:

    RECEIVED
    VALIDATING
    LOADING
    EVALUATING
    EXECUTING
    PERSISTING
         ↓
       FAILED

These are **application workflow states**, not domain aggregate states.

They must not be confused with domain lifecycle states.

---

# 7. WF-COL-001 — Create Collection

## 7.1 Intent

Create a new Collection in a valid initial state.

## 7.2 Trigger

A user or authorized application process requests creation of a Collection.

## 7.3 Input

Conceptually:

    CollectionName
    CollectionMetadata?
    OwnershipContext

## 7.4 Preconditions

Application-level:

- request exists;
- required fields are present;
- primitive values have valid representation.

Domain-level:

- Collection name satisfies domain rules;
- ownership context is valid;
- initial Collection state is valid.

## 7.5 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Create Collection aggregate
      ↓
    Validate domain invariants
      ↓
    Persist Collection
      ↓
    Collect CollectionCreated
      ↓
    Publish / dispatch event
      ↓
    Return Collection identity

## 7.6 Domain Operation

The Collection aggregate owns creation.

The application layer does not construct an invalid Collection and then repair it.

Creation should produce a valid aggregate immediately.

## 7.7 Success

Postconditions:

- Collection exists;
- Collection satisfies all initial invariants;
- Collection has a stable identity;
- `CollectionCreated` exists as the resulting domain fact.

## 7.8 Failure

Possible failures:

- invalid input;
- invalid Collection name;
- invalid ownership context;
- persistence failure.

## 7.9 Consistency

Single aggregate.

Strong consistency is required for Collection creation.

---

# 8. WF-COL-002 — Rename Collection

## 8.1 Intent

Change the name of an existing Collection.

## 8.2 Input

    CollectionId
    NewCollectionName

## 8.3 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Load Collection
      ↓
    Collection.rename(...)
      ↓
    Validate invariants
      ↓
    Persist Collection
      ↓
    Collect CollectionRenamed
      ↓
    Publish / dispatch event
      ↓
    Return success

## 8.4 Preconditions

- Collection exists;
- Collection allows modification;
- new name is structurally valid;
- new name satisfies domain rules.

## 8.5 Domain Responsibility

The Collection aggregate decides whether the rename is valid.

The application workflow must not reproduce Collection naming rules.

## 8.6 Success

Collection contains the new valid name.

Event:

    CollectionRenamed

## 8.7 Failure

Possible failures:

- Collection not found;
- invalid name;
- Collection cannot currently be modified;
- persistence failure.

## 8.8 Consistency

Single Collection aggregate.

---

# 9. WF-COL-003 — Add Item to Collection

This workflow is currently the most important cross-aggregate workflow.

## 9.1 Intent

Create a valid membership relationship between an Item and a Collection.

## 9.2 Input

    CollectionId
    ItemId
    MembershipMetadata?

## 9.3 Participants

Primary aggregate:

    Collection

Supporting domain object:

    Item

Potential policy:

    CollectionItemCompatibilityPolicy

## 9.4 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Load Collection
      ↓
    Load Item / obtain required Item facts
      ↓
    Evaluate compatibility policy
      ↓
       ┌───────────────┐
       │ compatible?   │
       └───────┬───────┘
               │
          NO   │   YES
          ↓    │    ↓
        Reject │ Collection.addItem(...)
               │        ↓
               │    Validate invariants
               │        ↓
               │    Persist Collection
               │        ↓
               │    ItemAddedToCollection
               │        ↓
               │    Publish / dispatch
               │
               └──────────────→ Success

## 9.5 Preconditions

Application:

- CollectionId exists in the request;
- ItemId exists in the request;
- metadata has valid structural representation.

Domain:

- Collection can be modified;
- Item is compatible when compatibility is a domain rule;
- membership does not violate uniqueness;
- membership-specific invariants hold.

## 9.6 Aggregate Ownership

The membership belongs conceptually to the Collection boundary.

Therefore:

    Collection.addItem(ItemId)

is the authoritative state-changing operation.

The application must not create membership directly in persistence.

## 9.7 Cross-Aggregate Rule

If Item information is required to evaluate membership validity:

    Item
      ↓
    facts
      ↓
    Compatibility Policy
      ↓
    Collection.addItem(...)

The policy evaluates compatibility.

The Collection aggregate protects its own membership invariants.

## 9.8 Success

Postconditions:

- membership exists;
- Collection invariants hold;
- Item remains unchanged unless an explicit domain rule states otherwise;
- `ItemAddedToCollection` is produced.

## 9.9 Failure

Possible failures:

- Collection not found;
- Item not found;
- incompatible Item;
- duplicate membership;
- Collection cannot be modified;
- invalid membership metadata;
- persistence failure.

## 9.10 Consistency

The membership change must be strongly consistent within Collection.

The Item does not become part of the Collection aggregate.

---

# 10. WF-COL-004 — Remove Item from Collection

## 10.1 Intent

Remove a membership relationship without deleting the Item.

## 10.2 Input

    CollectionId
    ItemId

## 10.3 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Load Collection
      ↓
    Collection.removeItem(ItemId)
      ↓
    Validate invariants
      ↓
    Persist Collection
      ↓
    ItemRemovedFromCollection
      ↓
    Publish / dispatch
      ↓
    Success

## 10.4 Preconditions

- Collection exists;
- Collection permits modification;
- membership exists, unless the domain later defines removal as idempotent.

## 10.5 Important Semantic Rule

    Remove Item from Collection
              ≠
           Delete Item

The Item aggregate remains independent.

## 10.6 Success

Membership no longer exists.

Event:

    ItemRemovedFromCollection

## 10.7 Failure

Possible failures:

- Collection not found;
- membership not found;
- Collection cannot be modified;
- persistence failure.

## 10.8 Consistency

Single Collection aggregate.

---

# 11. WF-ITEM-001 — Create Item

## 11.1 Intent

Create a new valid Item.

## 11.2 Input

Conceptually:

    ItemIdentityInformation
    ItemMetadata
    Classification?
    Provenance?

## 11.3 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Create Item aggregate
      ↓
    Apply initial domain rules
      ↓
    Validate invariants
      ↓
    Persist Item
      ↓
    ItemCreated
      ↓
    Publish / dispatch
      ↓
    Success

## 11.4 Preconditions

- required identity information exists;
- metadata is structurally valid;
- classification is valid when supplied;
- provenance is valid when supplied.

## 11.5 Success

A valid Item exists.

Event:

    ItemCreated

## 11.6 Failure

Possible failures:

- invalid identity;
- invalid metadata;
- invalid classification;
- invalid provenance;
- identity conflict;
- persistence failure.

## 11.7 Consistency

Single Item aggregate.

---

# 12. WF-ITEM-002 — Update Item Metadata

## 12.1 Intent

Change descriptive information belonging to an Item.

## 12.2 Input

    ItemId
    MetadataChanges

## 12.3 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Load Item
      ↓
    Item.updateMetadata(...)
      ↓
    Validate invariants
      ↓
    Persist Item
      ↓
    ItemMetadataUpdated
      ↓
    Publish / dispatch
      ↓
    Success

## 12.4 Preconditions

- Item exists;
- Item permits modification;
- metadata changes satisfy domain rules.

## 12.5 Success

Item contains valid updated metadata.

Event:

    ItemMetadataUpdated

## 12.6 Failure

Possible failures:

- Item not found;
- invalid metadata;
- Item cannot be modified;
- persistence failure.

## 12.7 Consistency

Single Item aggregate.

---

# 13. WF-ITEM-003 — Change Item Classification

## 13.1 Intent

Change the classification associated with an Item.

## 13.2 Input

    ItemId
    NewClassification

## 13.3 Workflow

    Trigger
      ↓
    Validate input
      ↓
    Load Item
      ↓
    Evaluate classification policy if required
      ↓
    Item.changeClassification(...)
      ↓
    Validate invariants
      ↓
    Persist Item
      ↓
    ItemClassificationChanged
      ↓
    Publish / dispatch
      ↓
    Success

## 13.4 Preconditions

- Item exists;
- Item can be modified;
- classification is valid;
- any required classification policy permits the transition.

## 13.5 Success

Item has the new valid classification.

Event:

    ItemClassificationChanged

## 13.6 Failure

Possible failures:

- Item not found;
- invalid classification;
- prohibited classification transition;
- Item cannot be modified;
- persistence failure.

---

# 14. Common Workflow Pattern

The current workflows reveal a common structure:

    Receive
      ↓
    Validate
      ↓
    Load
      ↓
    Evaluate
      ↓
    Execute
      ↓
    Persist
      ↓
    Publish
      ↓
    Return

However, not every workflow requires every stage.

For example:

    Rename Collection

does not necessarily require an external policy evaluation.

The application layer must remain proportional to the domain behavior.

---

# 15. Workflow Decision Points

Decision points should be explicit.

Example:

    Add Item
        |
        v
    Item exists?
       / \
     NO   YES
     |      |
    fail   continue
              |
              v
       compatible?
          / \
        NO   YES
        |      |
       fail   continue
                 |
                 v
           membership exists?
               / \
             YES  NO
             |     |
            fail  continue

This makes business and application assumptions visible.

---

# 16. Failure Classification

Workflow failures are divided into:

## Validation Failure

The request cannot be interpreted correctly.

Example:

    missing CollectionId

---

## Not Found

Required domain state cannot be located.

Example:

    CollectionId does not identify an existing Collection

---

## Domain Rule Violation

The requested operation is semantically invalid.

Example:

    duplicate membership

---

## Conflict

The requested change conflicts with the current state or another concurrent operation.

---

## Infrastructure Failure

The domain operation may be valid but the technical execution fails.

Examples:

    persistence unavailable
    event transport unavailable
    external dependency unavailable

---

# 17. Failure Propagation

The application workflow should preserve the semantic distinction between failures.

Conceptually:

    Domain Error
        ↓
    Application Error Boundary
        ↓
    External Contract

The mapping to HTTP status codes, messaging responses, or UI errors belongs to later layers.

---

# 18. Transaction Boundaries

Transaction boundaries are derived from consistency requirements.

For current simple workflows:

    Create Collection
        → Collection transaction

    Rename Collection
        → Collection transaction

    Remove Item
        → Collection transaction

    Create Item
        → Item transaction

    Update Item
        → Item transaction

The more interesting case is:

    Add Item to Collection

where Item information participates in decision-making but Collection owns the state mutation.

The Item itself should not automatically become part of the same transaction.

---

# 19. Aggregate Loading

Application workflows may load an aggregate because:

- its current state is required;
- its behavior must be invoked;
- its invariants must be evaluated by the aggregate.

The workflow should not load aggregates merely to expose their database representation.

---

# 20. Aggregate Persistence

After successful domain behavior:

    Aggregate
       ↓
    changed state
       ↓
    persistence boundary

The application layer decides when persistence must occur.

The domain does not know how persistence happens.

---

# 21. Domain Events Within Workflows

A domain event represents a fact resulting from successful domain behavior.

Example:

    Collection.addItem(...)
          ↓
    state change
          ↓
    ItemAddedToCollection

The event is not:

    "application requested AddItem"

It is:

    "the Item was successfully added to the Collection."

This distinction is fundamental.

---

# 22. Event Publication Boundary

The conceptual flow is:

    Aggregate
       ↓
    Domain Event
       ↓
    Application Boundary
       ↓
    Event Publication Mechanism

The exact technical mechanism is intentionally deferred.

Potential mechanisms include:

- transactional outbox;
- event dispatcher;
- event store;
- message broker.

No mechanism is selected by this document.

---

# 23. Event Publication Failure

The system must distinguish:

    Domain operation failed

from:

    Domain operation succeeded
    but event publication failed

These are semantically different situations.

A later architecture document must define how reliable event publication is achieved.

---

# 24. Idempotency Matrix

Current assumptions:

| Workflow | Idempotency Status |
|---|---|
| Create Collection | Not inherently idempotent |
| Rename Collection | Potentially repeatable |
| Add Item to Collection | Requires explicit domain decision |
| Remove Item from Collection | Requires explicit domain decision |
| Create Item | Not inherently idempotent |
| Update Item Metadata | Depends on command semantics |
| Change Item Classification | Potentially repeatable |

These are not implementation decisions.

They represent domain questions that must be resolved.

---

# 25. Retry Model

Application retries must not silently change domain semantics.

For a retryable operation:

    Original request
         ↓
    domain operation
         ↓
    uncertain technical result
         ↓
    retry
         ↓
    same command semantics

The system must define whether the second execution:

- succeeds;
- is rejected as duplicate;
- is treated as already completed.

This is especially relevant to asynchronous integration.

---

# 26. Concurrency

Workflows must assume that aggregate state may change between reads and writes.

For example:

    Load Collection
         ↓
    another process changes Collection
         ↓
    current workflow attempts modification

The architecture must eventually define optimistic concurrency or an equivalent consistency mechanism.

The domain requirement is:

> An aggregate must not be persisted in a way that silently violates a more recent valid state.

---

# 27. Add Item Concurrency Example

Two requests:

    Request A ── Add Item X
    Request B ── Add Item X

Both load the same Collection state.

Both see:

    Item X not present

Both attempt to add it.

The domain invariant requires that the final state contain at most one membership.

Therefore the persistence/consistency architecture must protect this invariant under concurrency.

This will be addressed later.

---

# 28. Workflow Atomicity

Atomicity should be defined according to business meaning.

For:

    Rename Collection

the state change is atomic within Collection.

For:

    Add Item to Collection

the membership creation must be atomic within Collection.

Event publication is a separate architectural concern unless the event itself participates in domain persistence semantics.

---

# 29. Authorization

Authorization should occur before unauthorized domain mutation.

Conceptually:

    Actor
      ↓
    Authorization
      ↓
    Use Case
      ↓
    Domain

However, authorization and domain validity remain separate concepts.

An authorized actor can still request an invalid domain operation.

---

# 30. Ownership

If CollectionHub introduces ownership as a domain concept, ownership rules may participate in:

- authorization;
- aggregate invariants;
- domain policies.

The final ownership model remains an open domain question.

---

# 31. Synchronous vs Asynchronous Workflows

Current CRUD-like domain operations are conceptually synchronous:

    Request
      ↓
    Domain operation
      ↓
    Result

Future processes such as:

    Import catalog
    Synchronize metadata
    Reconcile external information

may become asynchronous.

These should not be forced into the synchronous workflow model.

---

# 32. Long-Running Process Candidate

A future import process may look like:

    Start Import
       ↓
    Import Requested
       ↓
    External acquisition
       ↓
    Item resolution
       ↓
    Collection updates
       ↓
    Import Completed

This is a **process workflow**, not necessarily one atomic use case.

It will require separate modeling if introduced.

---

# 33. Query Workflows

Query workflows are intentionally outside the main write workflow model.

Potential examples:

    Get Collection
    Get Item
    List Collection Items
    Search Items

A query:

- should not mutate domain state;
- should not require aggregate mutation;
- may use a read model;
- may have different consistency requirements.

A dedicated query model will be defined later.

---

# 34. Workflow Traceability

Each workflow should be traceable to the domain model.

Example:

    WF-COL-003
         ↓
    AddItemToCollection
         ↓
    Collection
         ↓
    Membership Invariant
         ↓
    Compatibility Policy
         ↓
    ItemAddedToCollection

This creates a direct connection between application behavior and domain semantics.

---

# 35. Workflow Matrix

| Workflow | Aggregate | Policy | Event | Consistency |
|---|---|---|---|---|
| Create Collection | Collection | — | CollectionCreated | Strong |
| Rename Collection | Collection | — | CollectionRenamed | Strong |
| Add Item | Collection | Compatibility? | ItemAddedToCollection | Strong within Collection |
| Remove Item | Collection | — | ItemRemovedFromCollection | Strong |
| Create Item | Item | — | ItemCreated | Strong |
| Update Metadata | Item | — | ItemMetadataUpdated | Strong |
| Change Classification | Item | Classification? | ItemClassificationChanged | Strong |

---

# 36. Workflow Dependency Graph

Current dependencies:

    Create Collection
          |
          v
      Collection

    Create Item
          |
          v
        Item

    Add Item
       / \
      /   \
 Collection  Item
      |
      v
 Compatibility Policy

    Remove Item
          |
          v
      Collection

    Update Metadata
          |
          v
         Item

    Change Classification
          |
          v
         Item

---

# 37. Workflow Invariants

The following invariants are particularly important:

## Collection Membership

    A Collection cannot contain the same Item membership more than once.

## Aggregate Validity

Every successful workflow must leave the affected aggregate valid.

## Item Independence

Removing an Item from a Collection does not delete or modify the Item.

## Domain Ownership

Only the appropriate aggregate can perform its state mutation.

## Event Semantics

Events represent successful domain facts, not merely requests.

---

# 38. Workflow Anti-Patterns

## 38.1 Direct Persistence Mutation

Bad:

    AddItemWorkflow
        ↓
    INSERT membership

Preferred:

    AddItemWorkflow
        ↓
    Collection.addItem(...)
        ↓
    persist Collection

---

## 38.2 Business Rules in Workflow

Bad:

    if collection.items.contains(item):
        reject

if this is the authoritative membership invariant.

Preferred:

    Collection.addItem(...)
        ↓
    aggregate enforces invariant

---

## 38.3 Aggregate Bypass

Bad:

    workflow
       ↓
    change aggregate fields

Preferred:

    workflow
       ↓
    aggregate behavior

---

## 38.4 Technical Workflow Naming

Avoid:

    SaveCollectionWorkflow

Prefer:

    RenameCollection

---

## 38.5 Giant Workflow

Avoid a single workflow that performs:

    create item
    create collection
    add item
    update metadata
    publish notification
    synchronize external service

unless the business concept explicitly defines this as one operation.

---

# 39. Application Workflow Boundary

The conceptual boundary is:

    +------------------------------------------+
    | APPLICATION                             |
    |                                          |
    | Trigger                                 |
    |   ↓                                      |
    | Use Case                                |
    |   ↓                                      |
    | Workflow                                |
    |   ↓                                      |
    | Coordination                            |
    +-------------------+----------------------+
                        |
                        v
    +------------------------------------------+
    | DOMAIN                                   |
    |                                          |
    | Policies                                |
    | Aggregates                              |
    | Invariants                              |
    | Domain Events                           |
    +-------------------+----------------------+
                        |
                        v
    +------------------------------------------+
    | INFRASTRUCTURE                           |
    |                                          |
    | Persistence                             |
    | Messaging                               |
    | External Systems                        |
    +------------------------------------------+

---

# 40. Open Questions

## OPEN-WF-001

Should Add Item to Collection require the Item aggregate to exist synchronously?

## OPEN-WF-002

Should duplicate Add Item requests be rejected or treated as idempotent success?

## OPEN-WF-003

Should removing a missing membership be an error or an idempotent success?

## OPEN-WF-004

Which domain policies require external information?

## OPEN-WF-005

Which workflows can safely be retried?

## OPEN-WF-006

What optimistic concurrency strategy will protect aggregate invariants?

## OPEN-WF-007

What mechanism guarantees reliable event publication?

## OPEN-WF-008

Which future operations are long-running processes?

## OPEN-WF-009

Which query workflows belong to the core application model?

## OPEN-WF-010

Which operations require authorization versus domain ownership validation?

---

# 41. Completion Criteria

This document is considered sufficiently mature when:

- every important write-side use case has a workflow;
- each workflow has explicit inputs;
- preconditions are identified;
- participating aggregates are known;
- policies are identified;
- success paths are explicit;
- failure paths are explicit;
- resulting events are known;
- consistency requirements are documented;
- idempotency questions are visible;
- concurrency-sensitive workflows are identified;
- technical implementation remains outside the document.

---

# 42. Final Principle

The purpose of an application workflow is not to make the domain simpler by moving its rules elsewhere.

Its purpose is to make **orchestration explicit**.

The desired separation is:

    APPLICATION
        |
        | coordinates
        v
    DOMAIN
        |
        | decides
        v
    BUSINESS OUTCOME

Therefore:

> **Application workflows coordinate the execution of domain behavior; aggregates, policies, and domain services remain authoritative for business meaning and invariants.**

The next step is to formalize the error semantics that these workflows can produce in:

`13_DOMAIN_ERROR_MODEL.md`