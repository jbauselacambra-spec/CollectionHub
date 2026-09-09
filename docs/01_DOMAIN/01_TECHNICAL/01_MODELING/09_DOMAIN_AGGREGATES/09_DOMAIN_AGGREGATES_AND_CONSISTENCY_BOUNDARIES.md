# CollectionHub — Domain Aggregates and Consistency Boundaries

## 1. Purpose

This document defines the **aggregate structure and consistency boundaries** of the CollectionHub domain.

It builds directly on:

- the domain concepts;
- entities and value objects;
- relationships;
- commands and domain events;
- invariants;
- business rules.

The purpose is to determine:

> **Which domain concepts must change together, which invariants must be protected atomically, and where the boundaries of consistency should exist.**

This document does not define:

- database tables;
- ORM mappings;
- repository implementations;
- HTTP endpoints;
- framework components;
- transaction implementation details;
- message broker configuration;
- deployment topology.

Those concerns belong to later architectural phases.

---

# 2. Aggregate Philosophy

An aggregate is not simply a group of related entities.

An aggregate is a **consistency boundary**.

Its purpose is to define:

1. which domain objects are modified together;
2. which invariants must hold immediately;
3. which object owns the decisions that protect those invariants;
4. which changes are exposed through the aggregate root;
5. which relationships should remain outside the boundary.

The central principle is:

> **An aggregate should be as small as possible while still protecting the invariants that require a single consistency boundary.**

This prevents two opposite problems:

### Oversized aggregates

```text
Too many concepts
       ↓
Large consistency boundary
       ↓
High coupling
       ↓
Low concurrency
       ↓
Complex behavior
```

### Undersized aggregates

```text
Too many boundaries
       ↓
Invariants span multiple aggregates
       ↓
Constant coordination
       ↓
Complex workflows
       ↓
Weak domain consistency
```

The objective is therefore not to minimize or maximize aggregate count.

The objective is to establish the **correct semantic boundaries**.

---

# 3. Aggregate Design Criteria

An aggregate boundary should be considered when one or more of the following are true:

- several objects must always change together;
- an invariant spans multiple objects;
- one object controls the lifecycle of another;
- a domain decision requires immediate consistency;
- a collection of objects behaves as a single conceptual unit;
- the domain naturally treats one object as the owner of another.

An aggregate boundary should be avoided when:

- objects merely reference one another;
- objects have independent lifecycles;
- consistency can be eventual;
- one object only needs another object's identity;
- coupling would provide no domain benefit.

---

# 4. Candidate Aggregate Map

At the current stage, CollectionHub identifies the following candidate aggregates:

```text id="q5h6j8"
Collection
    |
    +-- Collection membership semantics
    +-- Collection lifecycle
    +-- Collection metadata/value objects

Item
    |
    +-- Item identity
    +-- Item intrinsic metadata
    +-- Item lifecycle, if applicable

Potential future aggregates
    |
    +-- Source / Catalog
    +-- Import / Ingestion
    +-- Classification / Taxonomy
    +-- Ownership / Sharing
```

The exact final aggregate structure remains subject to validation against the complete domain model.

The important point is that **Collection and Item should not automatically become one aggregate merely because collections contain items**.

---

# 5. Aggregate A — Collection

## 5.1 Aggregate Root

The proposed aggregate root is:

```text id="a1g2r7"
Collection
```

The Collection root is responsible for protecting the invariants directly related to the collection itself and its membership semantics.

---

## 5.2 Aggregate Responsibility

The Collection aggregate is responsible for:

- collection identity;
- collection metadata;
- collection lifecycle, if applicable;
- membership semantics;
- membership uniqueness;
- rules governing addition/removal of members;
- collection-level invariants;
- collection-related domain events.

---

## 5.3 Candidate Internal Objects

The Collection aggregate may contain:

```text id="v0z7kd"
Collection
    |
    +-- CollectionName
    +-- CollectionMetadata
    +-- CollectionMembership*
```

The exact internal structure must not be interpreted as a persistence schema.

These are domain concepts.

---

# 6. Collection Membership as an Internal Concept

Membership is particularly important.

A membership relationship may contain domain meaning beyond:

```text id="2u0w4q"
Collection → Item
```

It may eventually include concepts such as:

- position;
- acquisition information;
- notes;
- quantity;
- condition;
- date added;
- source;
- user-defined metadata.

Therefore the relationship should not prematurely be treated as a simple technical association.

The domain question is:

> **Is membership itself a meaningful domain concept?**

Current modeling suggests that it should be treated as one.

---

# 7. Collection Aggregate Invariants

The Collection aggregate must protect the collection-specific invariants identified in `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`.

At minimum:

```text id="x2d7pn"
INV-COLLECTION-001
INV-COLLECTION-002
INV-COLLECTION-003
INV-COLLECTION-004
INV-COLLECTION-005
INV-COLLECTION-006
INV-COLLECTION-007
INV-COLLECTION-008
```

Particularly important are:

- stable collection identity;
- valid collection metadata;
- valid membership;
- membership uniqueness;
- valid collection modification state.

---

# 8. Collection Aggregate Boundary

The proposed boundary is:

```text id="q4b8m1"
+--------------------------------------+
|              Collection              |
|                                      |
|  CollectionName                      |
|  CollectionMetadata                  |
|                                      |
|  Membership                          |
|      ├── ItemIdentity                |
|      └── MembershipMetadata           |
|                                      |
|  Collection invariants               |
+--------------------------------------+
```

The important design decision is that the aggregate may reference an Item by **identity**, rather than embedding the complete Item aggregate.

Conceptually:

```text id="4g5mca"
Collection
    |
    +---- references ----> ItemId
```

rather than:

```text id="5qv8x1"
Collection
    |
    +---- contains ----> complete Item aggregate
```

This distinction is fundamental.

---

# 9. Item Aggregate

## 9.1 Aggregate Root

The proposed root is:

```text id="k1t5vx"
Item
```

The Item aggregate represents the intrinsic identity and state of a collectible item.

---

## 9.2 Aggregate Responsibility

The Item aggregate is responsible for:

- item identity;
- intrinsic item metadata;
- item classification;
- item lifecycle, if applicable;
- item-level invariants;
- item-related domain events.

---

## 9.3 Candidate Internal Objects

The aggregate may contain concepts such as:

```text id="r7w1zc"
Item
    |
    +-- ItemIdentifier
    +-- ItemMetadata
    +-- Classification
    +-- Provenance
    +-- LifecycleState
```

These are conceptual members of the aggregate, not persistence instructions.

---

# 10. Item Aggregate Invariants

The Item aggregate protects:

```text id="e2r6fd"
INV-ITEM-001
INV-ITEM-002
INV-ITEM-003
INV-ITEM-004
INV-ITEM-005
```

In particular:

- stable identity;
- valid intrinsic information;
- separation between identity and metadata;
- correct handling of external identifiers;
- valid classification.

---

# 11. Why Item Should Be Separate From Collection

At first glance, it may appear attractive to model:

```text id="2a9k0f"
Collection
    └── Items
```

as one aggregate.

This would be appropriate only if the domain required Item and Collection to be modified under one consistency boundary.

Current analysis does not establish that requirement.

Items have their own:

- identity;
- metadata;
- provenance;
- potential lifecycle;
- potential relationships with other collections.

Therefore embedding an Item inside a Collection would create unnecessary coupling.

The preferred model is:

```text id="s8v1pk"
Collection Aggregate
        |
        | references
        v
      ItemId
        ^
        |
Item Aggregate
```

---

# 12. Multiple Collection Membership

An Item may potentially belong to multiple collections.

This is another strong argument against making Item an internal entity of Collection.

The domain should therefore allow:

```text id="z5e2nq"
             +----------------+
             |      Item      |
             +----------------+
                ^          ^
                |          |
                |          |
        +-------+--+   +---+--------+
        | Collection|   | Collection |
        |     A     |   |     B      |
        +-----------+   +------------+
```

Each collection owns its own membership semantics.

The Item aggregate remains independent.

---

# 13. Aggregate References

Cross-aggregate references should use domain identity.

Preferred:

```text id="4u2b8s"
Collection
    |
    +---- ItemId
```

Avoid:

```text id="r8x4qy"
Collection
    |
    +---- Item object graph
```

This keeps aggregate boundaries explicit.

---

# 14. Cross-Aggregate Consistency

The following principle applies:

> **A Collection operation should not require the entire Item aggregate to be loaded and modified merely because the collection references an item.**

For example, adding an item to a collection should primarily modify Collection state.

Conceptually:

```text id="f1g6za"
AddItemToCollection
        |
        v
Collection Aggregate
        |
        +-- validate membership
        +-- add ItemId
        +-- produce ItemAddedToCollection
```

The Item aggregate does not need to change simply because a membership was created.

---

# 15. Example — Adding an Item

A conceptual operation:

```text id="1s8r3x"
AddItemToCollection
```

may follow:

```text id="e5t2cw"
Command
  |
  v
Collection identified
  |
  v
Collection loaded
  |
  v
Membership rules evaluated
  |
  +---- invalid ----> reject
  |
  +---- valid
          |
          v
    Collection changed
          |
          v
ItemAddedToCollection
```

The Item aggregate remains untouched unless the domain explicitly requires an Item-side change.

---

# 16. Example — Updating Item Metadata

Consider:

```text id="0j6v8p"
UpdateItemMetadata
```

This operation belongs conceptually to Item.

The Collection aggregate does not need to be modified merely because the Item changed.

```text id="7j4d9p"
UpdateItemMetadata
       |
       v
Item Aggregate
       |
       +-- validate
       +-- change metadata
       |
       v
ItemMetadataUpdated
```

Existing collection memberships remain valid.

This is a strong indication that Item and Collection should remain separate aggregates.

---

# 17. Example — Removing an Item From a Collection

The operation:

```text id="3x9c2e"
RemoveItemFromCollection
```

belongs to Collection.

Conceptually:

```text id="x5m7qk"
Command
   |
   v
Collection
   |
   +-- membership exists?
   |
   +-- allowed?
   |
   v
Membership removed
   |
   v
ItemRemovedFromCollection
```

The Item itself is not deleted.

This distinction is essential:

```text id="j7q4bn"
Remove membership
        ≠
Delete item
```

---

# 18. Item Deletion

Item deletion requires additional domain clarification.

If Item is referenced by multiple collections, deleting Item could affect multiple aggregates.

Therefore:

```text id="f4x8d2"
Delete Item
    |
    +---- Collection A membership
    |
    +---- Collection B membership
    |
    +---- Collection C membership
```

would become a cross-aggregate operation.

The domain must therefore explicitly decide whether deletion:

- is allowed;
- is replaced by archival;
- leaves historical identity;
- automatically removes memberships;
- requires prior membership removal;
- or is not a domain operation at all.

This remains an open modeling question.

---

# 19. Aggregate Root Rules

Only aggregate roots should provide the public behavioral entry point to their aggregate.

Conceptually:

```text id="c2n8vk"
External operation
       |
       v
Aggregate Root
       |
       v
Internal state
```

not:

```text id="b7x1ms"
External operation
       |
       +---- directly mutates internal entity
```

This ensures that aggregate invariants remain protected.

---

# 20. Aggregate Internal Entities

An entity may exist inside an aggregate without being an independent aggregate root.

For example:

```text id="e9s4pa"
Collection
    |
    +-- Membership
```

Membership does not necessarily need its own repository or independent lifecycle.

Its independence should be determined by domain behavior, not by the fact that it has an identifier.

---

# 21. Aggregate Internal Value Objects

Value objects should generally remain inside the aggregate whose invariants they support.

For example:

```text id="v8n5kc"
Collection
    |
    +-- CollectionName
```

The CollectionName does not need an independent consistency boundary.

Its validity is part of Collection validity.

---

# 22. Aggregate Lifecycle Independence

Aggregates should have independent lifecycles unless the domain explicitly requires otherwise.

For example:

```text id="m2f8xq"
Collection lifecycle
        ≠
Item lifecycle
```

Changing one must not automatically imply changing the other.

---

# 23. Aggregate Boundary and Ownership

Ownership of a concept is not determined by database foreign keys.

Instead, ask:

> **Who is responsible for deciding whether this change is valid?**

If the answer is Collection, the behavior belongs to Collection.

If the answer is Item, the behavior belongs to Item.

This provides a more reliable aggregate boundary than technical relationship analysis.

---

# 24. Aggregate Command Ownership

Based on the current command model, commands can provisionally be grouped as follows.

## Collection

```text id="k6z4tw"
CreateCollection
RenameCollection
AddItemToCollection
RemoveItemFromCollection
```

Potential future commands:

```text id="j3r9pv"
ArchiveCollection
RestoreCollection
ChangeCollectionMetadata
ReorderCollectionItem
```

These remain subject to final domain confirmation.

---

## Item

Potential commands:

```text id="h7c2nq"
CreateItem
UpdateItemMetadata
ChangeItemClassification
ArchiveItem
RestoreItem
```

Again, lifecycle commands remain provisional until the Item lifecycle is formally established.

---

# 25. Aggregate Event Ownership

Events should originate from the aggregate whose state actually changed.

Examples:

```text id="u4f8kq"
Collection
    |
    +-- CollectionCreated
    +-- CollectionRenamed
    +-- ItemAddedToCollection
    +-- ItemRemovedFromCollection
```

And:

```text id="s7p3jd"
Item
    |
    +-- ItemCreated
    +-- ItemMetadataUpdated
    +-- ItemClassificationChanged
```

The exact event names remain governed by `07_DOMAIN_COMMANDS_AND_EVENTS.md`.

---

# 26. Event Ownership Rule

An event should not be attributed to an aggregate merely because the aggregate is related to the event.

The question is:

> **Which aggregate's state changed to make this fact true?**

For example:

```text id="x1k8rm"
Item added to Collection
```

changes Collection membership.

Therefore the primary fact belongs to Collection.

The Item does not need to emit an event merely because its identity was referenced.

---

# 27. Cross-Aggregate Business Rules

Some future rules may involve both Item and Collection.

For example:

```text id="w6r3pv"
"Only items satisfying condition X may be added
to collections of type Y."
```

This creates a cross-aggregate decision.

The solution should not automatically be:

```text id="v4y7cs"
Merge Item + Collection
```

Instead, possible approaches include:

- domain service;
- domain policy;
- application coordination;
- read model;
- eventual consistency.

The appropriate choice depends on whether the rule requires immediate consistency.

---

# 28. Consistency Levels

CollectionHub should distinguish between three conceptual levels.

## 28.1 Strong Aggregate Consistency

Rules inside one aggregate must hold immediately after successful mutation.

```text id="n7d3za"
Aggregate mutation
      ↓
All aggregate invariants hold
```

---

## 28.2 Coordinated Cross-Aggregate Consistency

Some operations may require coordination between multiple aggregates.

These should be explicitly modeled rather than hidden inside one aggregate.

---

## 28.3 Eventual Consistency

Some derived information may become consistent asynchronously.

This is appropriate only where the domain permits temporary divergence.

It must not be used to weaken an invariant that actually requires immediate consistency.

---

# 29. Do Not Use Aggregates as Transaction Containers by Default

A transaction boundary and an aggregate boundary are related but not identical concepts.

The domain question is:

> **Which invariants require atomic consistency?**

The technical question is:

> **How will that consistency be implemented?**

The second question belongs to later architecture.

---

# 30. Candidate Aggregate Diagram

Current conceptual model:

```text id="q1h5wd"
                    +----------------------+
                    |   Collection         |
                    |   Aggregate          |
                    |                      |
                    |  CollectionName      |
                    |  Metadata            |
                    |                      |
                    |  Memberships         |
                    |     |                |
                    |     +-- ItemId       |
                    +----------+-----------+
                               |
                               | identity reference
                               v
                    +----------------------+
                    |       Item           |
                    |      Aggregate       |
                    |                      |
                    |  Identity            |
                    |  Metadata            |
                    |  Classification      |
                    |  Provenance          |
                    |  Lifecycle           |
                    +----------------------+
```

The two aggregates are related but independently consistent.

---

# 31. What Does Not Belong Inside Collection

The following should not be embedded inside Collection merely because they are related to a collection:

- complete Item state;
- external source configuration;
- global taxonomy;
- external catalog state;
- unrelated user profile state;
- infrastructure metadata;
- search indexes;
- presentation-specific data.

Collection should contain only concepts required to protect Collection's own semantics.

---

# 32. What Does Not Belong Inside Item

Similarly, Item should not contain:

- complete Collection state;
- collection membership state for every collection;
- UI-specific information;
- search result state;
- external integration workflow state;
- unrelated ownership workflows.

An Item should remain focused on the item's intrinsic domain meaning.

---

# 33. Potential Future Aggregate — Source

A future Source or Catalog aggregate may become necessary if CollectionHub models external sources as first-class domain concepts.

Potential responsibility:

```text id="n6p4xj"
Source
    |
    +-- Source identity
    +-- Source configuration
    +-- Source status
    +-- Source provenance rules
```

However, this should only become an aggregate if the domain gives Source its own:

- identity;
- lifecycle;
- invariants;
- commands;
- events;
- independent consistency boundary.

It must not be created merely because integrations exist.

---

# 34. Potential Future Aggregate — Import

Import or ingestion may eventually become a separate domain concept.

For example:

```text id="x8q2sm"
Import
    |
    +-- Import identity
    +-- Source
    +-- Status
    +-- Progress
    +-- Result
    +-- Errors
```

This would be appropriate if importing is itself a meaningful business process.

If import is merely an infrastructure mechanism, it should remain outside the domain model.

This distinction must be resolved later.

---

# 35. Potential Future Aggregate — Taxonomy

Classification may eventually justify an independent aggregate if taxonomy becomes dynamic and governed by domain rules.

Possible concept:

```text id="p5c7wn"
Taxonomy
    |
    +-- Classification
    +-- Hierarchy
    +-- Rules
```

For now, Classification is treated as part of Item semantics unless further analysis demonstrates independent lifecycle and consistency requirements.

---

# 36. Aggregate Boundary Decision Rules

When considering whether to create a new aggregate, use these questions:

### Question 1

Does the concept have an independent lifecycle?

### Question 2

Does it have its own invariants?

### Question 3

Does it need to be modified independently?

### Question 4

Must it always change atomically with another concept?

### Question 5

Does it have an identifiable aggregate root?

### Question 6

Would keeping it inside another aggregate create excessive coupling?

### Question 7

Would separating it make important invariants impossible to protect?

Only when these questions support a meaningful boundary should a separate aggregate be introduced.

---

# 37. Aggregate Size Principle

The preferred aggregate size is:

> **The smallest boundary capable of protecting the required invariants.**

This means that aggregate size should not be optimized for:

- database joins;
- object navigation;
- convenience;
- API response shape;
- UI screens.

It should be optimized for **domain consistency**.

---

# 38. Concurrency Implications

Aggregate boundaries also establish natural concurrency boundaries.

If two users modify different collections:

```text id="f2m8dq"
User A → Collection A
User B → Collection B
```

their operations should not need to conflict merely because both collections contain items.

Similarly, changing Item metadata should not inherently lock or mutate every collection containing that Item.

This is an architectural consequence of keeping the aggregates independent.

---

# 39. Invariant Ownership Matrix

| Invariant / Rule | Primary Owner |
|---|---|
| Collection identity | Collection |
| Collection name validity | Collection |
| Collection lifecycle | Collection |
| Membership validity | Collection |
| Membership uniqueness | Collection |
| Item identity | Item |
| Item intrinsic metadata | Item |
| Item classification | Item |
| Item provenance | Item |
| Item lifecycle | Item |
| Cross-aggregate compatibility | Domain Policy / Service |
| External source reconciliation | Source / Policy, if modeled |

This table is provisional and must evolve as the domain becomes more precise.

---

# 40. Command Ownership Matrix

| Command | Aggregate / Owner |
|---|---|
| CreateCollection | Collection |
| RenameCollection | Collection |
| AddItemToCollection | Collection |
| RemoveItemFromCollection | Collection |
| CreateItem | Item |
| UpdateItemMetadata | Item |
| ChangeItemClassification | Item |
| ArchiveCollection | Collection |
| RestoreCollection | Collection |
| ArchiveItem | Item |
| RestoreItem | Item |

Commands marked as provisional must be confirmed against the final lifecycle model.

---

# 41. Event Ownership Matrix

| Domain Event | Primary Aggregate |
|---|---|
| CollectionCreated | Collection |
| CollectionRenamed | Collection |
| ItemAddedToCollection | Collection |
| ItemRemovedFromCollection | Collection |
| ItemCreated | Item |
| ItemMetadataUpdated | Item |
| ItemClassificationChanged | Item |
| CollectionArchived | Collection |
| CollectionRestored | Collection |
| ItemArchived | Item |
| ItemRestored | Item |

These mappings must remain aligned with `07_DOMAIN_COMMANDS_AND_EVENTS.md`.

---

# 42. Aggregate Anti-Patterns

## AGG-ANTI-001 — Everything Is One Aggregate

```text id="f9j3xk"
Collection
 └── Item
      └── Source
           └── User
                └── ...
```

This creates an enormous consistency boundary.

---

## AGG-ANTI-002 — Every Entity Is an Aggregate

Creating an aggregate for every entity produces excessive coordination.

An entity deserves an independent aggregate only when its domain semantics justify the boundary.

---

## AGG-ANTI-003 — Database-Driven Aggregates

Do not derive aggregate boundaries directly from tables.

---

## AGG-ANTI-004 — Navigation-Driven Aggregates

The fact that one object needs to navigate to another does not mean they belong to the same aggregate.

---

## AGG-ANTI-005 — Transaction-Driven Modeling

Do not create an aggregate solely because a technical transaction happens to touch several objects.

First establish the domain invariant.

---

## AGG-ANTI-006 — Bidirectional Aggregate Graphs

Avoid designs where aggregates hold complete references to one another.

Prefer identity references.

---

## AGG-ANTI-007 — Aggregate as Service Locator

An aggregate must not become a container for unrelated domain operations merely because it is convenient.

---

# 43. Current Aggregate Decisions

The current preferred model is:

```text id="p2h8za"
Collection Aggregate
        |
        | ItemId
        v
Item Aggregate
```

With:

```text id="m7c3qe"
Collection
    owns membership semantics

Item
    owns intrinsic item semantics
```

This is the strongest current interpretation of the domain based on the established invariants.

---

# 44. Decisions That Remain Open

The following decisions should remain explicitly unresolved until further domain analysis.

## OPEN-AGG-001

Whether CollectionMembership deserves richer internal behavior or should remain a simpler internal entity/value concept.

---

## OPEN-AGG-002

Whether Item actually requires an independent lifecycle.

---

## OPEN-AGG-003

Whether Collection has a meaningful lifecycle beyond existence and deletion.

---

## OPEN-AGG-004

Whether Source becomes a first-class domain aggregate.

---

## OPEN-AGG-005

Whether Import/Ingestion is a business process or merely infrastructure.

---

## OPEN-AGG-006

Whether Taxonomy requires an independent consistency boundary.

---

## OPEN-AGG-007

What happens when an Item is removed or archived while memberships still exist.

---

## OPEN-AGG-008

Whether historical membership must survive Item lifecycle changes.

---

# 45. Traceability to Invariants

Every aggregate must be traceable to the invariants it protects.

Current mapping:

```text id="d8m2vw"
Collection Aggregate
    |
    +-- INV-COLLECTION-*
    +-- BR-MEMBERSHIP-*
    +-- Collection lifecycle rules

Item Aggregate
    |
    +-- INV-ITEM-*
    +-- Item lifecycle rules
    +-- Metadata rules
    +-- Provenance rules
```

Cross-aggregate rules remain outside these boundaries unless future modeling demonstrates that they belong inside one of them.

---

# 46. Traceability to Commands and Events

The aggregate model provides ownership for the commands and events defined previously.

Conceptually:

```text id="a9r5zk"
Collection
    |
    +-- Commands
    |     |
    |     +-- CreateCollection
    |     +-- RenameCollection
    |     +-- AddItemToCollection
    |     +-- RemoveItemFromCollection
    |
    +-- Events
          |
          +-- CollectionCreated
          +-- CollectionRenamed
          +-- ItemAddedToCollection
          +-- ItemRemovedFromCollection
```

And:

```text id="v6x1cp"
Item
    |
    +-- Commands
    |     |
    |     +-- CreateItem
    |     +-- UpdateItemMetadata
    |     +-- ChangeItemClassification
    |
    +-- Events
          |
          +-- ItemCreated
          +-- ItemMetadataUpdated
          +-- ItemClassificationChanged
```

This establishes a coherent chain:

```text id="n4q7tm"
Invariant
    ↓
Aggregate
    ↓
Command
    ↓
Domain Decision
    ↓
State Change
    ↓
Event
```

---

# 47. Consequences for Future Architecture

The aggregate model will later influence:

- repository boundaries;
- transaction boundaries;
- application service responsibilities;
- command handling;
- concurrency control;
- event publication;
- persistence models;
- integration boundaries.

However, these technical decisions must be derived from the aggregate semantics.

The architecture must not redefine the domain boundaries for convenience.

---

# 48. Completion Criteria

This document is considered sufficiently mature for the next modeling phase when:

- each major invariant has an owner;
- aggregate roots are identifiable;
- aggregate boundaries are explicit;
- cross-aggregate relationships are explicit;
- commands have provisional ownership;
- events have provisional ownership;
- identity references are distinguished from object containment;
- cross-aggregate rules are identified;
- open questions are recorded;
- aggregate boundaries are justified by consistency rather than persistence.

---

# 49. Final Principle

The CollectionHub aggregate model should be governed by one central rule:

> **An aggregate is the smallest domain boundary that can protect the invariants that must remain consistent together.**

Therefore:

```text id="u8k3zf"
Related
    ≠
Same Aggregate
```

and:

```text id="h6r2qp"
Same Database
    ≠
Same Aggregate
```

and:

```text id="j4m9xs"
Same Transaction
    ≠
Same Aggregate
```

The correct question is always:

> **Which concepts must change together for the domain to remain valid?**

At the current level of analysis, the strongest answer is:

```text id="c7w5na"
+-----------------------------+
| Collection Aggregate        |
|                             |
| Collection                  |
| Membership                  |
| Collection invariants       |
+-------------+---------------+
              |
              | ItemId
              |
              v
+-----------------------------+
| Item Aggregate              |
|                             |
| Item                        |
| Intrinsic metadata          |
| Classification              |
| Provenance                  |
| Item invariants             |
+-----------------------------+
```

This gives CollectionHub two clearly separated consistency boundaries while preserving the semantic relationship between collections and collectible items.

The next modeling step should therefore focus on **Domain Services and Policies**: identifying business decisions that do not naturally belong to a single aggregate without weakening aggregate boundaries.