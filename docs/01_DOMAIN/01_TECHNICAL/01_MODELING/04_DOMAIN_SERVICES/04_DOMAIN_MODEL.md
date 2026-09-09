# CollectionHub Domain Model

> **Document:** `04_DOMAIN_MODEL.md`  
> **Phase:** 2.1 — Domain Modeling  
> **Status:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`, `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`, `03_AGGREGATE_BOUNDARY_ANALYSIS.md`  
> **Purpose:** Provide the canonical conceptual model of the CollectionHub domain.

---

# 1. Purpose

This document consolidates the domain-modeling work completed so far.

It defines the current canonical interpretation of the CollectionHub domain in terms of:

- Entities;
- Value Objects;
- Aggregate Roots;
- Aggregate boundaries;
- domain relationships;
- ownership;
- identity;
- lifecycle;
- invariants;
- configuration;
- unresolved domain questions.

This document is intentionally positioned between conceptual analysis and implementation.

It is **not**:

- a database schema;
- an ORM model;
- an API contract;
- a class diagram;
- a persistence specification;
- an implementation design.

The model described here represents the current understanding of the business domain.

---

# 2. Domain Model at a Glance

The current domain can be summarized as:

```text
User
  │
  │ owns / manages
  ▼
Collection Aggregate
  │
  ├── Collection
  ├── Collection Owner
  └── Collection Configuration
          │
          ├── Item Type definitions
          ├── Attribute definitions
          └── Collection-level rules
          
Collection
  │
  │ identified by CollectionId
  │
  ├──────────────┬──────────────┬──────────────┐
  ▼              ▼              ▼              ▼
Item Aggregate  Item Aggregate  Item Aggregate  ...
     │
     ├── Collection Item
     ├── Item Identifier
     ├── Item Type
     ├── Attribute Values
     ├── Metadata
     ├── Tags
     ├── Item Status
     └── External References

Item Aggregate
     │
     └── may participate in
             │
             ▼
      Item Relationships
```

The most important structural decision is:

> **A Collection organizes Items, but does not contain all Items inside its consistency boundary.**

---

# 3. Domain Building Blocks

The current model contains the following building blocks.

| Building Block | Classification | Aggregate |
|---|---|---|
| Collection | Entity / Aggregate Root | Collection |
| Collection Configuration | Entity-owned configuration | Collection |
| Collection Owner | Role | Collection |
| Collection Item | Entity / Aggregate Root | Collection Item |
| Item Identifier | Value Object | Collection Item |
| Item Type | Classification / Definition | TBD / Collection-scoped candidate |
| Attribute | Definition / Entity candidate | TBD / Collection-scoped candidate |
| Attribute Value | Value Object | Collection Item |
| Metadata | Supporting concept | Collection Item |
| Tag | Classification concept | Collection Item / TBD |
| Item Status | State / Value concept | Collection Item |
| External Reference | Value Object candidate | Collection Item |
| Source | Supporting Entity candidate | TBD |
| Item Relationship | Relationship concept | TBD |

---

# 4. Aggregate Model

The domain currently contains two confirmed Aggregate Roots.

```text
AR-001 Collection
AR-002 Collection Item
```

A third Aggregate for Item Relationships remains unresolved.

---

# 5. Aggregate Root — Collection

## 5.1 Definition

The Collection Aggregate represents the organizational context in which a set of Collection Items is managed.

It owns collection-level rules and configuration.

It does **not** own the consistency lifecycle of every Item.

---

## 5.2 Conceptual Structure

```text
Collection Aggregate
│
├── Collection
│   ├── CollectionId
│   ├── Name
│   ├── Description
│   └── Ownership
│
└── Collection Configuration
    ├── Item Type rules
    ├── Attribute definitions
    └── Collection-specific constraints
```

The exact decomposition of configuration is still subject to refinement.

---

## 5.3 Responsibilities

The Collection Aggregate is responsible for:

- maintaining Collection identity;
- maintaining collection ownership;
- maintaining collection-level configuration;
- enforcing collection-level invariants;
- controlling changes to configuration;
- establishing the domain context in which Items exist.

---

## 5.4 Collection does not own Item State

The following are deliberately outside the Collection Aggregate's consistency boundary:

- Item metadata;
- Item attributes;
- Item status;
- Item tags;
- Item external references;
- Item lifecycle.

This means:

```text id="q0xg9e"
Collection
   does not directly control
Item internal state
```

unless a future domain rule explicitly requires such control.

---

# 6. Aggregate Root — Collection Item

## 6.1 Definition

The Collection Item Aggregate represents one individual object recorded in a Collection.

Each Item has its own identity and lifecycle.

---

## 6.2 Conceptual Structure

```text
Collection Item Aggregate
│
└── Collection Item
    │
    ├── Item Identifier
    ├── Collection Identity
    ├── Item Type
    ├── Attribute Values
    ├── Metadata
    ├── Tags
    ├── Item Status
    └── External References
```

---

## 6.3 Responsibilities

The Item Aggregate is responsible for:

- maintaining Item identity;
- maintaining Collection membership reference;
- maintaining Item lifecycle;
- maintaining Item state;
- validating item-level attribute values;
- managing item tags;
- managing external references;
- enforcing item-level invariants.

---

# 7. Collection Membership

An Item references its Collection by identity.

Conceptually:

```text
Collection Item
      │
      └── CollectionId
```

The Item does not contain the complete Collection Aggregate.

Likewise, the Collection does not contain complete Item Aggregates.

This creates a clean boundary:

```text
Collection Aggregate
        ↕
    CollectionId
        ↕
Collection Item Aggregate
```

---

# 8. Entity Model

The core Entity model is:

```text
User
 │
 └── manages
       │
       ▼
Collection
 │
 └── identifies collection context

Collection Item
 │
 └── identifies collected object
```

---

# 9. Entity Identity

## 9.1 Collection

```text
CollectionId
```

represents the stable identity of a Collection.

Changing:

- name;
- description;
- configuration;
- ownership;

does not change the Collection identity.

---

## 9.2 Collection Item

```text
ItemId
```

represents the stable identity of an Item.

Changing:

- attributes;
- metadata;
- tags;
- status;
- external references;

does not change the Item identity.

---

## 9.3 User

```text
UserId
```

represents the stable identity of a User.

Profile or authentication changes do not change the domain identity of the User.

---

# 10. Value Objects

The current model favors the following Value Objects.

## 10.1 Item Identifier

Represents the internal identity value of a Collection Item.

---

## 10.2 Attribute Value

Represents a concrete value assigned to an Attribute for an Item.

---

## 10.3 External Reference

Represents an external reference associated with an Item.

It does not define Item identity.

---

## 10.4 Item Status

Represents the current lifecycle state of an Item.

The status value itself has no independent identity.

---

# 11. Attribute Model

The Attribute model is intentionally divided into two conceptual levels.

```text
Attribute Definition
        │
        │ defines
        ▼
Attribute Value
```

For example:

```text
Attribute:
    Year

Item:
    X

Attribute Value:
    1987
```

The definition answers:

> What characteristic is being represented?

The value answers:

> What is the value of that characteristic for this Item?

---

# 12. Attribute Ownership

The current leading hypothesis is:

```text
Collection
   │
   └── Collection Configuration
          │
          └── Attribute Definitions
```

This would allow different Collections to define different attribute vocabularies.

However, this remains a **candidate decision**.

It must be confirmed against the actual requirements.

---

# 13. Item Type Model

Item Type is currently treated as a classification concept.

Conceptually:

```text
Collection Item
      │
      └── Item Type
```

The Item Type determines the semantic category of the Item.

Potential future relationship:

```text
Item Type
    │
    └── defines applicable Attributes
```

This relationship is currently a hypothesis.

---

# 14. Tags

Tags provide flexible classification.

They differ from Item Type.

```text
Item Type
    = what the item is

Tag
    = useful classification attached to the item
```

The current model treats assigned Tags as part of the Item Aggregate.

The ownership and identity of Tag definitions remain unresolved.

---

# 15. Metadata

Metadata provides contextual information about an Item.

It is currently considered part of the Item Aggregate because:

- it describes the Item;
- it has no independent identity;
- it has no independent lifecycle;
- it does not appear to require an independent consistency boundary.

The exact boundary between Metadata and Attribute Values remains open.

---

# 16. Item Status

Status represents Item lifecycle state.

Conceptually:

```text
Collection Item
      │
      └── Status
```

Status transitions may eventually become explicit domain behavior.

For example:

```text
ACTIVE
  │
  ├── archive
  ▼
ARCHIVED
```

The final state machine has not yet been defined.

---

# 17. External References

An Item may contain multiple external references.

Conceptually:

```text
Collection Item
      │
      ├── External Reference
      ├── External Reference
      └── External Reference
```

Each reference may contain:

- Source;
- external identifier;
- external location;
- reference metadata.

The precise shape remains subject to Source modeling.

---

# 18. Source

Source is currently a supporting Entity candidate.

The model intentionally does not force a decision.

Two possible interpretations remain:

### Interpretation A — Simple Value

```text
External Reference
    └── source = "ExternalCatalogue"
```

### Interpretation B — Managed Entity

```text
Source
 ├── identity
 ├── configuration
 ├── metadata
 └── lifecycle
```

The second model should only be adopted if the domain gives Source independent behavior.

---

# 19. Item Relationships

Item Relationships connect Collection Items.

Conceptually:

```text
Item A
   │
   └── relationship ──> Item B
```

The current model deliberately avoids deciding that relationships are separate Aggregates.

Possible future classifications:

```text
Value Object
Entity
Independent Aggregate
```

The correct choice depends on whether the relationship has independent identity and lifecycle.

---

# 20. Ownership Model

The current ownership model is:

```text
User
  │
  │ ownership role
  ▼
Collection
```

Collection Owner is treated as a role rather than an independent Entity.

This allows the model to evolve toward:

- shared ownership;
- delegated management;
- organizations;
- teams;

without prematurely introducing those concepts.

---

# 21. Domain Responsibilities

The model can be divided into responsibility areas.

## Collection

Responsible for:

- ownership;
- collection identity;
- collection configuration;
- collection-level rules.

## Collection Item

Responsible for:

- item identity;
- lifecycle;
- item state;
- item-level data;
- item-level invariants.

## Configuration

Responsible for:

- defining applicable item types;
- defining attributes;
- defining collection-level rules.

## Relationship

Responsible for:

- semantic association between Items, if eventually promoted.

---

# 22. Domain Invariant Ownership

| Invariant Category | Primary Owner |
|---|---|
| Collection identity | Collection |
| Collection ownership | Collection |
| Configuration validity | Collection |
| Item identity | Collection Item |
| Item lifecycle | Collection Item |
| Attribute Value validity | Collection Item |
| Item status | Collection Item |
| External reference validity | Collection Item |
| Attribute definition validity | Configuration |
| Item-Type applicability | Configuration + Item |
| Item-to-item relationship validity | Relationship / TBD |

The important principle is:

> The Aggregate Root is responsible for protecting the invariants that define its boundary.

---

# 23. Domain Reference Rules

Aggregates should communicate through identity rather than object graphs.

Preferred:

```text
Collection
    references ItemId

Collection Item
    references CollectionId
```

Avoid:

```text
Collection
    contains complete Item Aggregate

Item
    contains complete Collection Aggregate
```

This keeps boundaries explicit.

---

# 24. Domain Lifecycle

The currently known lifecycle model is intentionally minimal.

## Collection

```text
Created
   │
   ▼
Active
   │
   ▼
Archived
```

## Collection Item

```text
Created
   │
   ▼
Active
   │
   ▼
Archived
```

These states are illustrative and must not yet be treated as final domain state machines.

---

# 25. Domain Operations

The following operations are currently identified as domain behavior candidates.

## Collection

```text
Create
Rename
Change Owner
Change Configuration
Archive
```

## Collection Item

```text
Create
Change Type
Set Attribute
Remove Attribute
Add Tag
Remove Tag
Change Status
Add External Reference
Remove External Reference
Archive
```

These are domain-level operations, not API endpoints.

---

# 26. Cross-Aggregate Operations

Some use cases naturally span multiple Aggregates.

Examples include:

```text
Move Item Between Collections
Create Relationship Between Items
Apply Configuration Change
Transfer Collection Ownership
```

Such operations should not automatically enlarge Aggregate boundaries.

They may instead be coordinated by:

- domain services;
- application services;
- domain events;
- explicit workflows.

The exact mechanism will be decided later.

---

# 27. Preliminary Domain Services

No Domain Service has yet been formally established.

This is intentional.

A Domain Service should only be introduced when:

- behavior is genuinely domain behavior;
- it does not naturally belong to one Entity/Aggregate;
- moving it into an Entity would create inappropriate coupling.

Potential future candidates include:

```text
CollectionTransferService
ItemRelationshipService
ConfigurationCompatibilityPolicy
```

These are only hypotheses.

---

# 28. Domain Events

Domain Events have not yet been modeled formally.

Potential events are visible from the current lifecycle:

```text
CollectionCreated
CollectionArchived
CollectionOwnershipChanged

ItemCreated
ItemArchived
ItemStatusChanged
ItemAttributeChanged
ItemRelationshipCreated
```

However, an event should only be introduced when there is a meaningful domain reason to represent the occurrence.

The event model should therefore follow domain behavior rather than infrastructure requirements.

---

# 29. Canonical Aggregate Map

The current canonical Aggregate model is:

```text
                         ┌──────────────────────────┐
                         │      USER ENTITY         │
                         └────────────┬─────────────┘
                                      │
                                      │ owns / manages
                                      ▼
                  ┌────────────────────────────────────┐
                  │        COLLECTION AGGREGATE         │
                  │                                    │
                  │  Collection                         │
                  │      │                             │
                  │      ├── CollectionId              │
                  │      ├── Owner                      │
                  │      └── Configuration              │
                  │              │                     │
                  │              ├── Item Types         │
                  │              └── Attributes         │
                  │                                    │
                  └────────────────┬───────────────────┘
                                   │
                              CollectionId
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
                 ▼                 ▼                 ▼
       ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
       │ ITEM AGGREGATE │ │ ITEM AGGREGATE │ │ ITEM AGGREGATE │
       │                │ │                │ │                │
       │ CollectionItem │ │ CollectionItem │ │ CollectionItem │
       │ ItemId         │ │ ItemId         │ │ ItemId         │
       │ Type           │ │ Type           │ │ Type           │
       │ Attributes     │ │ Attributes     │ │ Attributes     │
       │ Metadata       │ │ Metadata       │ │ Metadata       │
       │ Tags           │ │ Tags           │ │ Tags           │
       │ Status         │ │ Status         │ │ Status         │
       │ References     │ │ References     │ │ References     │
       └────────────────┘ └────────────────┘ └────────────────┘

                           │
                           ▼
                 ┌──────────────────────┐
                 │ ITEM RELATIONSHIP    │
                 │                      │
                 │ Classification: TBD  │
                 └──────────────────────┘
```

---

# 30. Domain Model Rules

The following rules are now considered foundational.

## RULE-001

A Collection is an Aggregate Root.

## RULE-002

A Collection Item is an independent Aggregate Root.

## RULE-003

A Collection does not contain all Items inside its consistency boundary.

## RULE-004

A Collection Item references its Collection by identity.

## RULE-005

Collection Configuration belongs to the Collection boundary unless future analysis proves otherwise.

## RULE-006

Item-level state belongs to the Collection Item boundary.

## RULE-007

External references do not define Item identity.

## RULE-008

Value Objects do not receive independent identity without explicit domain justification.

## RULE-009

Item-to-item relationships must not automatically enlarge Item Aggregates.

## RULE-010

Cross-Aggregate operations must not be used as justification for merging Aggregates.

---

# 31. Model Strength

The current model has several strong characteristics.

### Small consistency boundaries

Collection and Item are independently manageable.

### Stable identity

Entity identity is explicitly separated from mutable data.

### Explicit ownership

Collection owns collection-level rules.

Item owns item-level state.

### Controlled coupling

Aggregates communicate through identity.

### Extensibility

Dynamic attributes and classification can evolve without redefining the core Item identity.

### Scalability potential

A Collection can grow without growing the Collection Aggregate proportionally.

---

# 32. Remaining Domain Ambiguities

The model is coherent but not complete.

The following decisions remain important.

## AMB-001 — Item Type Scope

Are Item Types:

- global;
- collection-specific;
- user-defined;
- system-defined?

---

## AMB-002 — Attribute Scope

Are Attributes:

- global;
- collection-specific;
- item-type-specific;
- user-defined?

---

## AMB-003 — Tag Scope

Are Tags:

- global;
- collection-specific;
- user-specific;
- simple values?

---

## AMB-004 — Source

Does Source have independent identity and behavior?

---

## AMB-005 — Item Relationships

Does a relationship have:

- identity;
- lifecycle;
- metadata;
- provenance;
- independent rules?

---

## AMB-006 — Item Movement

Can an Item move between Collections?

---

## AMB-007 — Configuration Evolution

What happens when configuration changes make existing Item data incompatible?

---

## AMB-008 — Collaboration

Can multiple Users manage a Collection?

---

# 33. Model Stability Assessment

The model can now be considered:

### Stable

- Collection as Aggregate Root;
- Collection Item as Aggregate Root;
- Collection/Item separation;
- Entity identity principles;
- Value Object principles;
- Collection ownership concept;
- Item lifecycle ownership.

### Probable

- Collection-scoped configuration;
- Attribute Values inside Item Aggregate;
- Metadata inside Item Aggregate;
- External References inside Item Aggregate.

### Unresolved

- Item Type scope;
- Attribute scope;
- Tag scope;
- Source classification;
- Relationship classification;
- collaboration model;
- item movement semantics.

---

# 34. What This Model Explicitly Rejects

The current model rejects several tempting shortcuts.

## Rejected: One giant Collection object

```text
Collection
 └── Items[]
```

as a single consistency boundary.

---

## Rejected: Database-first identity

```text
Primary Key
    ↓
therefore Entity
```

Identity must come from domain semantics.

---

## Rejected: External identifier as domain identity

```text
External ID
    =
CollectionHub Item ID
```

These are separate concepts.

---

## Rejected: Every concept becomes an Entity

A concept becomes an Entity only when independent identity matters.

---

## Rejected: Every relationship becomes a database association

A relationship may have domain semantics and must be analyzed accordingly.

---

# 35. Canonical Domain Language

The following vocabulary should now be used consistently throughout the project:

| Term | Meaning |
|---|---|
| Collection | Organizational unit containing collected Items |
| Collection Item | Individual object recorded within a Collection |
| Item Type | Classification of an Item |
| Attribute | Definition of an Item characteristic |
| Attribute Value | Concrete value assigned to an Attribute |
| Tag | Flexible classification label |
| Item Status | Lifecycle state of an Item |
| Metadata | Supporting descriptive information |
| External Reference | Reference to information outside CollectionHub |
| Source | Origin of an External Reference |
| Collection Owner | Ownership role over a Collection |
| User | Domain actor |
| Collection Configuration | Rules defining collection-specific structure |
| Item Relationship | Semantic association between Items |

This vocabulary should be treated as the canonical language unless later domain analysis changes it.

---

# 36. Current Domain Model Statement

The CollectionHub domain can currently be stated as follows:

> **CollectionHub manages Collections owned or managed by Users. A Collection establishes an organizational and configuration context for its Collection Items. Each Collection Item has independent identity and lifecycle and maintains its own descriptive state, attributes, classification, status, and external references. Collection-level and Item-level consistency are separated into distinct Aggregate boundaries. Relationships between Items are modeled explicitly as domain concepts but their final consistency boundary remains unresolved.**

This statement represents the current canonical understanding of the domain.

---

# 37. Next Step

The next modeling stage should move from **static structure** toward **domain behavior**.

The recommended next artifact is:

```text
05_DOMAIN_BEHAVIOR_AND_OPERATIONS.md
```

That document should answer:

- What can Users actually do?
- What operations are meaningful in the domain?
- What commands change Aggregates?
- Which operations are valid or invalid?
- Which invariants are checked by each operation?
- Which operations cross Aggregate boundaries?
- Which operations may generate Domain Events?
- Which behavior belongs inside Aggregates versus outside them?

The progression is therefore:

```text
Concepts
   ↓
Classification
   ↓
Identity
   ↓
Invariants
   ↓
Aggregate Boundaries
   ↓
Canonical Domain Model
   ↓
Domain Behavior
   ↓
Commands / Policies / Events
```

Only after this behavioral model is sufficiently mature should we begin translating the domain into software architecture and implementation structures.