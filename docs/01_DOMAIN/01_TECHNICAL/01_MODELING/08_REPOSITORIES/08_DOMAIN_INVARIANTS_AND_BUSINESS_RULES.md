# CollectionHub — Domain Invariants and Business Rules

## 1. Purpose

This document defines the **invariants and business rules of the CollectionHub domain**.

Its purpose is to establish the conditions that must remain true throughout the lifecycle of the domain model and the rules that govern valid domain behavior.

This document deliberately stays at the **domain level**.

It does not define:

- database schemas;
- persistence mechanisms;
- API contracts;
- UI behavior;
- framework-specific validation;
- infrastructure concerns;
- authentication or authorization mechanisms;
- implementation-specific exception types;
- programming-language constructs.

The objective is to answer:

> **What must always be true in CollectionHub, and under what conditions is a domain operation considered valid?**

---

# 2. Domain Rule Philosophy

CollectionHub follows a domain-first approach.

Business rules are therefore treated as first-class domain knowledge rather than incidental validation logic.

A rule belongs to the domain when violating it would make the business state conceptually invalid, regardless of how that state is persisted or presented.

For example:

- a collection cannot contain the same item twice if uniqueness is part of the collection semantics;
- an item cannot transition from one lifecycle state to another when that transition is not supported by the domain;
- an event must describe something that actually occurred;
- an aggregate must not be left in a state that violates one of its invariants.

The domain must not depend on whether these rules are triggered through:

- an HTTP request;
- a command handler;
- an import process;
- a background job;
- an administrative operation;
- a future integration;
- or any other external mechanism.

The same domain rules apply regardless of entry point.

---

# 3. Definitions

## 3.1 Invariant

An **invariant** is a condition that must always hold for a valid domain state.

An invariant is not merely a convenient validation.

If an invariant is violated, the domain state is considered invalid.

Conceptually:

```text
Valid State
    |
    +-- invariant holds --> valid domain state
    |
    +-- invariant violated --> invalid domain state
```

---

## 3.2 Business Rule

A **business rule** defines a condition governing whether a domain operation or state transition is allowed.

A business rule may depend on:

- the current state;
- the requested operation;
- relationships between domain objects;
- temporal conditions;
- domain policies;
- previously established facts.

---

## 3.3 Aggregate Invariant

An aggregate invariant is an invariant whose consistency boundary belongs to a particular aggregate.

The aggregate is responsible for protecting the invariant.

External actors must not be able to bypass that protection.

---

## 3.4 State Transition Rule

A state transition rule defines which changes between lifecycle states are valid.

Not every state should necessarily be reachable from every other state.

---

# 4. Global Domain Invariants

The following invariants apply across CollectionHub regardless of the specific aggregate involved.

## INV-GLOBAL-001 — Domain Identity Uniqueness

Every domain entity must possess a stable identity within its defined identity boundary.

Two distinct domain entities must not be treated as the same entity merely because they currently share descriptive attributes.

---

## INV-GLOBAL-002 — Identity Stability

Once an entity has acquired its identity, that identity must not change during its lifecycle.

Changing descriptive information must not implicitly create a different domain identity.

---

## INV-GLOBAL-003 — Value Object Equality

Value objects are identified by their values rather than by independent identity.

Two value objects containing the same semantically relevant values must be considered equivalent.

---

## INV-GLOBAL-004 — Valid Value Objects

A value object must never exist in a semantically invalid state.

Validation belongs to the construction or creation boundary of the value object.

Invalid values must not be allowed to become part of valid domain state.

---

## INV-GLOBAL-005 — Explicit State

Every stateful domain entity must always have one well-defined lifecycle state.

An entity cannot simultaneously occupy incompatible states.

---

## INV-GLOBAL-006 — Explicit State Transitions

State changes must occur through explicitly defined domain transitions.

A state must not change merely because an unrelated property has changed.

---

## INV-GLOBAL-007 — No Impossible Domain State

The domain must never intentionally produce a state that contradicts its own invariants.

If an operation would necessarily produce an invalid state, the operation must not succeed.

---

## INV-GLOBAL-008 — Domain Events Represent Facts

A domain event represents something that actually happened in the domain.

Events must not be emitted merely because an operation was requested.

For example:

```text
Command requested
    ≠
Domain fact occurred
```

A failed operation must not produce an event representing successful completion.

---

## INV-GLOBAL-009 — Event Immutability

Once a domain event represents a historical domain fact, its meaning must not be altered.

Historical facts are append-oriented domain information.

---

## INV-GLOBAL-010 — Domain Rules Are Entry-Point Independent

The validity of a domain operation must not depend on the external mechanism used to invoke it.

The following must obey the same domain rules:

- user interaction;
- API call;
- import;
- scheduled operation;
- integration;
- administrative command.

---

# 5. Collection Invariants

A collection represents a domain-level grouping of collectible items according to CollectionHub semantics.

The following rules apply to collection consistency.

## INV-COLLECTION-001 — Collection Identity

Every collection must have a stable identity.

The identity of a collection must not change when its metadata changes.

---

## INV-COLLECTION-002 — Collection Ownership Context

Every collection must have an unambiguous ownership or responsibility context according to the domain model.

A collection must not exist without knowing which domain context governs it.

---

## INV-COLLECTION-003 — Collection Name Validity

A collection name must satisfy the domain's semantic requirements.

At minimum:

- it must exist when required by the domain;
- it must not consist solely of meaningless whitespace;
- it must respect the domain's length constraints;
- normalization rules must be applied consistently.

Presentation-specific formatting is outside the scope of this invariant.

---

## INV-COLLECTION-004 — Collection Membership Consistency

An item associated with a collection must have a valid membership relationship.

A collection must not contain a membership reference that the domain considers invalid.

---

## INV-COLLECTION-005 — No Duplicate Membership

Where collection semantics define membership as unique, the same domain item must not occupy the collection more than once under the same membership identity.

The uniqueness rule must be based on domain identity, not merely display attributes.

---

## INV-COLLECTION-006 — Membership Is Explicit

Being related to a collection must be represented by an explicit domain relationship.

An item must not be considered part of a collection merely because:

- its name matches;
- its metadata matches;
- it appears in a search result;
- it shares an external identifier;
- or it happens to be displayed alongside other items.

---

## INV-COLLECTION-007 — Collection Cannot Contain Invalid Members

An invalid or non-existent domain object must not become a valid collection member.

---

## INV-COLLECTION-008 — Collection Membership Removal

Removing an item from a collection must remove the domain membership relationship.

Removal must not silently alter the identity of either the collection or the item.

---

# 6. Collectible Item Invariants

CollectionHub treats collectible items as domain objects whose identity and descriptive information must remain conceptually distinct.

## INV-ITEM-001 — Item Identity

Every collectible item must possess a stable domain identity.

---

## INV-ITEM-002 — Identity Is Not Metadata

Changes to descriptive information must not automatically change the item's domain identity.

For example, changes to:

- title;
- description;
- classification;
- image;
- external metadata;

must not implicitly represent the creation of another domain item unless the domain explicitly defines such behavior.

---

## INV-ITEM-003 — Required Item Information

An item must contain all information that the domain considers mandatory for it to be valid.

Optional metadata must not be confused with required identity-defining information.

---

## INV-ITEM-004 — External Identifiers Are Not Automatically Domain Identity

An external identifier must not automatically become the domain identity unless the domain explicitly defines it as such.

External systems may:

- change their identifiers;
- provide conflicting identifiers;
- omit identifiers;
- expose multiple identifiers for the same conceptual item.

The domain must therefore distinguish external identity from internal domain identity.

---

## INV-ITEM-005 — Item Classification Validity

An item must only be classified using valid domain concepts.

Unknown or unsupported classifications must not silently become valid domain state.

---

# 7. Ownership and Relationship Rules

Relationships between domain objects must have explicit semantics.

## BR-REL-001 — Relationships Must Have a Meaning

Every persistent domain relationship must represent a meaningful domain concept.

A technical foreign-key-like relationship is not sufficient by itself to establish a domain relationship.

---

## BR-REL-002 — Relationship Creation Requires Valid Participants

A relationship may only be created when all participating domain objects satisfy the conditions required by the domain.

---

## BR-REL-003 — Relationship Removal Must Be Explicit

Removing a relationship must represent an intentional domain operation.

It must not occur accidentally as a side effect of unrelated metadata changes.

---

## BR-REL-004 — Relationship Direction Must Be Defined

Where the domain distinguishes between source and target concepts, the relationship direction must remain semantically consistent.

---

# 8. Lifecycle Invariants

Lifecycle states represent meaningful stages in the existence of a domain object.

## INV-LIFECYCLE-001 — One Current State

A lifecycle-managed entity must have exactly one current lifecycle state.

---

## INV-LIFECYCLE-002 — Valid Initial State

Every lifecycle-managed entity must begin in a valid initial state.

The initial state must be explicitly defined by the corresponding domain concept.

---

## INV-LIFECYCLE-003 — Valid Transition Only

A lifecycle transition is valid only when explicitly supported by the domain.

Conceptually:

```text
Current State
      |
      | valid transition
      v
Next State
```

An unsupported transition must not be treated as valid merely because it is technically possible.

---

## INV-LIFECYCLE-004 — No Silent State Changes

A lifecycle state must not change as an accidental side effect of unrelated operations.

---

## INV-LIFECYCLE-005 — Terminal States

If a lifecycle defines terminal states, an entity in a terminal state must not transition to another state unless the domain explicitly defines a reopening or recovery transition.

---

# 9. Command Preconditions

Commands represent requested domain actions.

A command must not be interpreted as proof that the requested operation is valid.

## BR-CMD-001 — Command Intent

A command represents an intention to perform a domain operation.

The domain must determine whether that intention can be fulfilled.

---

## BR-CMD-002 — Preconditions Must Be Satisfied

A command may succeed only when all required domain preconditions are satisfied.

---

## BR-CMD-003 — Failed Command Has No Successful Side Effect

If a command violates a domain invariant or business rule, the successful domain state change must not occur.

---

## BR-CMD-004 — Command Does Not Equal Event

The receipt or handling of a command does not itself constitute a domain fact.

Therefore:

```text
Command
    ↓
Decision
    ↓
State change
    ↓
Domain event
```

not:

```text
Command
    ↓
Event
```

---

# 10. Domain Event Rules

Domain events capture meaningful facts.

## BR-EVENT-001 — Events Follow Successful Domain Changes

A domain event representing a state-changing operation must only be produced after the domain operation has successfully occurred.

---

## BR-EVENT-002 — Event Semantics Are Historical

An event describes what happened, not what should happen.

For example:

```text
ItemAddedToCollection
```

means that the item was added.

It does not mean:

```text
AddItemToCollectionRequested
```

unless the domain explicitly models the latter as a separate concept.

---

## BR-EVENT-003 — Event Meaning Must Be Unambiguous

An event name must communicate a domain fact without requiring infrastructure knowledge to understand it.

---

## BR-EVENT-004 — Events Must Not Encode UI Semantics

Events must describe domain facts rather than presentation behavior.

Examples of domain concepts:

```text
ItemAddedToCollection
CollectionRenamed
ItemRemovedFromCollection
```

Examples of presentation concerns that do not belong in domain events:

```text
CollectionRowUpdated
RefreshCollectionScreen
ShowSuccessNotification
```

---

## BR-EVENT-005 — Historical Order

When event ordering has domain significance, events must preserve the order required to correctly interpret the domain history.

---

# 11. Collection Membership Business Rules

## BR-MEMBERSHIP-001 — Add Membership

An item may be added to a collection only when:

1. the collection exists;
2. the item exists;
3. the membership is valid;
4. the membership does not violate uniqueness rules;
5. the collection is in a state that permits modification.

---

## BR-MEMBERSHIP-002 — Duplicate Add

Attempting to add an item that already satisfies the collection's uniqueness constraint must not create another membership.

The domain must define whether the operation:

- is rejected;
- is treated as idempotent;
- or has another explicitly defined meaning.

The implementation must not invent this behavior independently of the domain model.

---

## BR-MEMBERSHIP-003 — Remove Membership

An item may be removed only when a corresponding membership exists, unless the domain explicitly defines removal as idempotent.

---

## BR-MEMBERSHIP-004 — Membership Changes Are Domain Facts

Successful membership changes must produce the corresponding domain facts defined in `07_DOMAIN_COMMANDS_AND_EVENTS.md`.

---

# 12. Metadata Rules

Metadata enriches domain objects but must not silently redefine their identity.

## BR-METADATA-001 — Metadata Must Respect Domain Semantics

Metadata may only contain values accepted by the corresponding domain concept.

---

## BR-METADATA-002 — Metadata Updates Preserve Identity

Updating metadata must not implicitly create a new entity.

---

## BR-METADATA-003 — Optional Metadata

Optional metadata may be absent without making the domain object invalid when the domain explicitly defines that information as optional.

---

## BR-METADATA-004 — Unknown Metadata

Unknown external metadata must not automatically become authoritative domain information.

External information must be interpreted according to the domain's trust and provenance rules.

---

# 13. Provenance Rules

CollectionHub may interact with information originating from external sources.

The domain must distinguish between information and its provenance.

## BR-PROVENANCE-001 — External Information Has Provenance

When provenance is relevant, externally sourced information must retain enough semantic context to determine where it originated.

---

## BR-PROVENANCE-002 — External Data Is Not Automatically Authoritative

Information originating from an external source must not automatically override existing domain truth.

---

## BR-PROVENANCE-003 — Conflicting External Information

When external information conflicts with existing domain state, the domain must apply an explicit reconciliation rule rather than silently replacing information.

---

## BR-PROVENANCE-004 — External Failure Does Not Redefine Domain Truth

Failure to retrieve external information must not automatically invalidate an otherwise valid domain object.

---

# 14. Temporal Rules

Time-dependent behavior must be explicitly modeled where time has domain significance.

## BR-TIME-001 — Temporal Values Must Be Semantically Valid

Dates and times must represent valid domain concepts.

---

## BR-TIME-002 — Chronological Consistency

When two temporal values have an ordered meaning, their relationship must remain valid.

For example:

```text
start <= end
```

when the domain defines such an interval.

---

## BR-TIME-003 — Past and Future Semantics

Rules involving past, present, or future must be evaluated using domain time semantics rather than presentation assumptions.

---

## BR-TIME-004 — Time Must Not Be Hidden

If a rule depends materially on time, that dependency must be explicit in the domain model.

---

# 15. Idempotency-Related Domain Rules

Idempotency is not universally required for every domain operation.

Where an operation is semantically idempotent, repeated execution must not create additional domain state.

## BR-IDEMPOTENCY-001 — Explicit Idempotency

An operation is idempotent only when the domain semantics explicitly support that interpretation.

---

## BR-IDEMPOTENCY-002 — No Accidental Idempotency

An implementation must not make every operation silently idempotent simply because doing so is technically convenient.

---

## BR-IDEMPOTENCY-003 — Repeated Domain Facts

Repeated requests must not generate misleading domain facts if no new domain fact actually occurred.

---

# 16. Aggregate-Level Invariants

Aggregates establish consistency boundaries.

## INV-AGG-001 — Aggregate Consistency

An aggregate must never expose a state that violates its own invariants.

---

## INV-AGG-002 — Aggregate-Owned Decisions

Business decisions affecting an aggregate's invariants must be made within the aggregate boundary or through a domain concept explicitly responsible for that decision.

---

## INV-AGG-003 — External Mutation Prohibited

External consumers must not be able to directly mutate aggregate state while bypassing domain rules.

---

## INV-AGG-004 — Aggregate Boundary Is Semantic

Aggregate boundaries must be defined according to transactional and consistency semantics, not merely according to database relationships.

---

## INV-AGG-005 — Aggregate Events Reflect Aggregate Facts

Events emitted from aggregate behavior must represent facts resulting from valid aggregate state transitions.

---

# 17. Cross-Aggregate Rules

Not every business rule should be enforced inside one aggregate.

Some rules involve multiple consistency boundaries.

## BR-CROSS-001 — Explicit Cross-Aggregate Coordination

A rule involving multiple aggregates must have an explicit coordination strategy.

---

## BR-CROSS-002 — No Artificial Aggregate Coupling

Aggregates must not be merged merely because they are related.

They should only share a consistency boundary when the domain requires immediate consistency.

---

## BR-CROSS-003 — Eventual Consistency Must Be Explicit

When a cross-aggregate rule permits eventual consistency, that behavior must be recognized as an explicit domain or application-level decision.

---

# 18. Deletion Rules

Deletion is a domain operation rather than merely a persistence operation.

## BR-DELETE-001 — Deletion Has Domain Meaning

An entity must only be deleted when the domain defines deletion as a valid operation.

---

## BR-DELETE-002 — Deletion Must Respect Dependencies

An entity must not be removed when doing so would violate an invariant or leave an invalid domain relationship.

---

## BR-DELETE-003 — Soft Deletion Is Not Assumed

The domain must not assume soft deletion unless the concept explicitly requires historical preservation through a retained entity state.

---

## BR-DELETE-004 — Deletion and Historical Facts

If historical facts must remain meaningful after deletion, the domain must preserve the necessary identity and historical semantics.

---

# 19. Uniqueness Rules

Uniqueness is a domain concept only when uniqueness has business meaning.

## BR-UNIQUE-001 — Domain-Level Uniqueness

A value must be unique only when the domain explicitly requires uniqueness.

---

## BR-UNIQUE-002 — Technical Uniqueness Is Not Business Uniqueness

A database uniqueness constraint does not, by itself, define a domain rule.

The domain rule must be identified independently.

---

## BR-UNIQUE-003 — Scope of Uniqueness

Every uniqueness rule must define its scope.

Examples:

```text
globally unique
unique within a collection
unique within an owner
unique within a relationship
```

---

# 20. Validation Boundaries

Validation must be divided according to responsibility.

## 20.1 Structural Validation

Examples:

- required field missing;
- malformed identifier;
- invalid primitive representation.

These may occur at application boundaries.

---

## 20.2 Domain Validation

Examples:

- invalid lifecycle transition;
- duplicate membership;
- invalid business relationship;
- prohibited operation.

These belong to domain behavior.

---

## 20.3 Infrastructure Validation

Examples:

- database constraint violations;
- network failures;
- serialization errors;
- unavailable external services.

These are not domain business rules unless the domain explicitly models them.

---

# 21. Rule Priority

When multiple rules apply, CollectionHub should resolve them according to the following conceptual priority:

```text
1. Domain invariants
2. Aggregate invariants
3. Business operation preconditions
4. State transition rules
5. Cross-aggregate policies
6. Application-level policies
7. Infrastructure constraints
```

A lower-level technical mechanism must not override a higher-level domain invariant.

---

# 22. Rule Classification

Every future business rule should be classified into one of the following categories:

| Category | Meaning |
|---|---|
| Invariant | Must always be true |
| Precondition | Must be true before an operation |
| Transition rule | Determines whether a state change is valid |
| Postcondition | Must be true after successful operation |
| Uniqueness rule | Defines uniqueness semantics |
| Relationship rule | Governs domain relationships |
| Temporal rule | Depends on time |
| Cross-aggregate rule | Involves multiple consistency boundaries |
| Policy | Selects behavior among valid alternatives |

This classification prevents the domain model from becoming an undifferentiated collection of validations.

---

# 23. Rule Identification Convention

Rules use the following identifiers:

```text
INV-*       Invariant
BR-*        Business Rule
```

More specific prefixes may be introduced later if the domain becomes sufficiently large.

Examples:

```text
INV-ITEM-001
INV-COLLECTION-001
INV-LIFECYCLE-001

BR-MEMBERSHIP-001
BR-PROVENANCE-001
BR-TIME-001
```

Identifiers are stable references for future analysis and traceability.

They should not be coupled to implementation classes or database structures.

---

# 24. Domain Rule Traceability

Every significant domain rule should eventually be traceable to one or more of:

```text
Domain Concept
      ↓
Aggregate
      ↓
Command
      ↓
Decision
      ↓
State Change
      ↓
Domain Event
```

This provides a mechanism for detecting incomplete domain behavior.

For example:

```text
Business Rule
    ↓
Requires a decision
    ↓
Requires a command or domain operation
    ↓
Changes domain state
    ↓
Produces a domain fact
```

If a rule has no identifiable owner or behavior, the domain model is probably incomplete.

---

# 25. Rules That Must Not Be Invented Yet

At this stage, certain rules must remain explicitly unresolved until the domain model provides sufficient evidence.

These include, unless already established elsewhere:

- exact authorization policies;
- exact persistence constraints;
- exact API validation rules;
- UI validation behavior;
- notification policies;
- retry policies;
- caching behavior;
- database cascade behavior;
- external service retry semantics;
- infrastructure failure handling;
- exact concurrency mechanisms;
- exact optimistic-locking strategy.

These concerns may eventually influence application or infrastructure behavior, but they must not be prematurely presented as domain invariants.

---

# 26. Open Domain Questions

The following questions should be resolved during subsequent domain analysis.

## Q-001 — Collection Ownership

Is ownership of a collection:

- mandatory;
- optional;
- single-owner;
- multi-owner;
- delegated?

---

## Q-002 — Item Uniqueness

What exactly makes two collectible items the same domain item?

Possible candidates may include:

- internal identity;
- canonical external identity;
- combination of attributes;
- source-specific identity.

This must be explicitly decided.

---

## Q-003 — Duplicate Membership

Should attempting to add an already-present item be:

- rejected;
- idempotent;
- interpreted as an update;
- or represented by another domain operation?

---

## Q-004 — Collection Lifecycle

Does a collection have a lifecycle?

If so:

- what are its states?
- which transitions are valid?
- are any states terminal?

---

## Q-005 — Item Lifecycle

Does a collectible item have an independent lifecycle?

If so, it must be modeled separately from collection membership.

---

## Q-006 — Deletion Semantics

Does CollectionHub distinguish between:

```text
removed
archived
deleted
deactivated
```

or are some of these concepts unnecessary?

---

## Q-007 — Provenance Authority

When multiple external sources provide conflicting metadata, which source, if any, is authoritative?

---

## Q-008 — Historical Preservation

Which domain facts must remain reconstructable after entities or relationships are removed?

---

# 27. Anti-Rules

The following behaviors should be actively avoided.

## ANTI-001 — Database-First Domain Rules

Do not define domain semantics solely because a relational schema makes a particular constraint convenient.

---

## ANTI-002 — UI-Driven Domain Rules

Do not define business invariants based on what happens to be convenient for a particular UI.

---

## ANTI-003 — Framework-Driven Domain Rules

Do not allow framework conventions to determine domain semantics.

---

## ANTI-004 — Implicit State Machines

Do not allow lifecycle transitions to emerge accidentally from property assignments.

---

## ANTI-005 — Event-as-Command Semantics

Do not treat an event as a request for an operation.

Events describe facts.

---

## ANTI-006 — External Data as Truth by Default

Do not assume external metadata is authoritative merely because it came from another system.

---

## ANTI-007 — Validation Everywhere

Do not duplicate the same business rule across controllers, services, repositories, and UI components without a clear domain owner.

---

## ANTI-008 — Accidental Invariants

Do not mistake implementation limitations for business rules.

---

# 28. Domain Consistency Principle

The central consistency principle for CollectionHub is:

> **A valid domain state is one that satisfies all applicable invariants, and every successful domain operation must preserve those invariants.**

Conceptually:

```text
Valid State
     |
     | Command
     v
Domain Decision
     |
     +---- invalid ----> no successful state change
     |
     +---- valid ------> new Valid State
                              |
                              v
                        Domain Event
```

This principle becomes the foundation for the subsequent aggregate and application-layer design.

---

# 29. Relationship With Previous Domain Documents

This document depends directly on the concepts established in previous modeling work.

Particularly:

```text
Domain Concepts
       ↓
Domain Entities / Value Objects
       ↓
Relationships
       ↓
Commands and Events
       ↓
Invariants and Business Rules
```

`07_DOMAIN_COMMANDS_AND_EVENTS.md` defines the language of requested actions and resulting domain facts.

This document defines the conditions under which those actions are valid and the constraints that must remain true after they execute.

---

# 30. Relationship With Future Documents

This document becomes an input to subsequent modeling phases.

The expected dependency is:

```text
08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md
                 |
                 v
        Aggregate Boundaries
                 |
                 v
       Domain Service Decisions
                 |
                 v
       Application Use Cases
                 |
                 v
      Ports / Interfaces / Contracts
                 |
                 v
       Technical Architecture
```

The important principle is that implementation should **derive from the domain rules**, not redefine them.

---

# 31. Completion Criteria

This document can be considered sufficiently mature for the next modeling phase when:

- all major domain concepts have identifiable invariants;
- important state transitions have explicit rules;
- collection membership semantics are defined;
- uniqueness semantics are explicit;
- identity semantics are explicit;
- domain events correspond to meaningful facts;
- aggregate consistency boundaries can be identified;
- cross-aggregate rules are distinguished from aggregate-local rules;
- unresolved business questions are explicitly recorded;
- infrastructure concerns have not been accidentally promoted to domain rules.

---

# 32. Final Principle

CollectionHub should not be modeled as a collection of CRUD operations.

Its domain should be modeled as a set of **valid states, meaningful transitions, business decisions, and domain facts**.

The purpose of invariants is therefore not merely to reject bad input.

Their deeper purpose is to protect the semantic integrity of the domain.

The fundamental rule is:

```text
No operation may leave the domain in a state
that contradicts its own meaning.
```

And consequently:

```text
Command
    ↓
Business Rules
    ↓
Domain Decision
    ↓
Invariant Preservation
    ↓
State Change
    ↓
Domain Event
```

This is the semantic foundation upon which the next modeling phase should build.