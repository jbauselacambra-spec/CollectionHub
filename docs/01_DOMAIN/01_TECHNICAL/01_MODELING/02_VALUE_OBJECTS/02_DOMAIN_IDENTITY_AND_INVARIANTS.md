# Domain Identity and Invariants

> **Document:** `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`  
> **Phase:** 2.1 — Domain Modeling  
> **Status:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`  
> **Purpose:** Define domain identity, invariants, consistency responsibilities, and preliminary ownership of domain rules.

---

# 1. Purpose

This document identifies the properties and rules that must remain true within the CollectionHub domain.

The objective is to answer four fundamental questions:

1. **What makes a domain object the same object over time?**
2. **What must always be true?**
3. **Which concept is responsible for protecting each rule?**
4. **Which concepts need to change together to preserve consistency?**

These answers are prerequisites for determining Aggregate boundaries.

The guiding principle is:

> **Aggregate boundaries should emerge from invariants, not the other way around.**

---

# 2. Identity Model

Identity is a domain concept, not merely a persistence concern.

A database-generated identifier may eventually be used to implement identity, but the database mechanism must not define the semantic meaning of identity.

---

# 3. Collection Identity

## 3.1 Definition

A Collection is uniquely identified within the CollectionHub domain.

Its identity is independent of:

- its name;
- its description;
- its owner;
- its configuration;
- its items.

Therefore:

```text
Collection Identity
    ≠
Collection Name
```

Two collections may have the same name and still represent different domain objects.

---

## 3.2 Identity Stability

Changing the following must not change the identity of the Collection:

- name;
- description;
- configuration;
- contained items;
- owner, if ownership transfer is supported.

Therefore:

> Collection identity must be stable across the Collection lifecycle.

---

# 4. Collection Item Identity

## 4.1 Definition

A Collection Item has a stable identity within CollectionHub.

The identity is independent of:

- attributes;
- metadata;
- tags;
- status;
- external references.

Therefore:

```text
Item Identity
    ≠
Item Description
    ≠
External Identifier
```

---

## 4.2 External Identity

A Collection Item may refer to one or more external identifiers.

For example:

```text
CollectionHub Item
    │
    ├── External Reference A
    │       └── identifier = X123
    │
    └── External Reference B
            └── identifier = ABC-789
```

Neither external identifier is automatically the identity of the Collection Item.

This protects the domain from coupling its identity model to external systems.

---

# 5. User Identity

A User has an identity independent of mutable profile information.

Changing:

- display name;
- email address;
- preferences;

must not automatically create a different User.

Authentication credentials and identity-provider identifiers are separate concerns.

---

# 6. Value Identity

The following concepts currently favor value-based equality:

- Item Identifier;
- Attribute Value;
- External Reference;
- Item Status.

For a Value Object:

```text
same semantic value
        ↓
same domain value
```

No independent lifecycle is required.

---

# 7. Identity Rules

The following rules are currently established.

| ID | Rule |
|---|---|
| ID-001 | Collection identity is independent of collection name. |
| ID-002 | Collection Item identity is independent of mutable item data. |
| ID-003 | External identifiers do not automatically define Collection Item identity. |
| ID-004 | User identity is independent of mutable profile data. |
| ID-005 | Value Objects are identified by their semantic values. |
| ID-006 | Persistence identifiers must not leak into domain semantics. |

---

# 8. Domain Invariants

An invariant is a condition that must remain true whenever the domain is in a valid state.

Invariants are more important than data structures because they describe what the domain considers valid.

---

# 9. Collection Invariants

## INV-COL-001 — Collection identity is unique

A Collection must have a unique domain identity.

```text
∀ Collection C:
    identity(C) is unique
```

### Responsibility

Collection Aggregate Candidate.

---

## INV-COL-002 — Collection must have an owner

A valid Collection must have an owner or owning subject.

```text
Collection
    → must have
Collection Owner
```

### Current confidence

**High**, assuming ownership is part of the current business model.

### Open question

Whether ownership is mandatory for the entire lifecycle must be validated.

---

## INV-COL-003 — Collection Items belong to a Collection

A Collection Item must belong to a valid Collection.

```text
Collection Item
    → belongs to
Collection
```

### Important consequence

An item cannot exist as a valid domain object without an ownership/containment context unless future requirements explicitly introduce standalone items.

---

## INV-COL-004 — Collection membership must be valid

A Collection must not contain an invalid reference to an Item.

This means:

```text
Collection
    contains
valid Collection Item
```

rather than merely:

```text
Collection
    contains
identifier
```

The precise enforcement mechanism remains an Aggregate Boundary question.

---

# 10. Collection Item Invariants

## INV-ITEM-001 — Item identity is unique

Every Collection Item must have a unique domain identity.

---

## INV-ITEM-002 — Item must belong to a Collection

A Collection Item cannot be considered valid without a Collection association.

---

## INV-ITEM-003 — Item Type must be valid

If an Item Type is required for an item, the assigned type must be valid within the relevant domain context.

Potential future rule:

```text
Item.Type ∈ AllowedTypes(Collection)
```

Whether this is collection-specific remains unresolved.

---

## INV-ITEM-004 — Attribute Values must correspond to valid Attributes

An Attribute Value cannot exist semantically without an Attribute definition.

```text
Attribute Value
    → references
Attribute
```

---

## INV-ITEM-005 — Attribute applicability must be respected

If an Attribute is restricted to specific Item Types, an item must not receive an incompatible Attribute.

Conceptually:

```text
if Attribute A is applicable to Type T
then Item(Type T) may use A
```

Conversely:

```text
if Attribute A is not applicable to Type T
then Item(Type T) must not use A
```

This invariant may become central to the Collection Configuration model.

---

## INV-ITEM-006 — Status must be valid

An Item must always have a valid domain status if the domain requires explicit lifecycle state.

```text
Item.Status ∈ ValidStatuses
```

---

## INV-ITEM-007 — Status transitions must obey domain rules

An item must not transition arbitrarily between statuses if the lifecycle contains restrictions.

For example:

```text
ACTIVE → ARCHIVED
```

may be valid while:

```text
ARCHIVED → ACTIVE
```

may or may not be valid.

The actual state machine is intentionally deferred.

---

# 11. Attribute Invariants

## INV-ATTR-001 — Attribute identity must be stable

If Attributes become managed Entities, their identity must remain stable across renaming or configuration changes.

---

## INV-ATTR-002 — Attribute Values must respect their value definition

If an Attribute defines a value type or validation rule, assigned values must conform.

Conceptually:

```text
Attribute
    defines validation
        ↓
Attribute Value
    must satisfy validation
```

---

## INV-ATTR-003 — Required Attributes must be present

If a Collection Configuration defines an Attribute as required for a given Item Type, a valid Item of that type must satisfy the requirement.

Conceptually:

```text
Required(Attribute A, ItemType T)
        +
Item.Type = T
        ↓
Item must contain valid A
```

This is potentially one of the strongest reasons for having Collection Configuration as part of a consistency boundary.

---

# 12. External Reference Invariants

## INV-EXT-001 — External Reference must identify a source

An External Reference should contain sufficient information to identify its origin.

Conceptually:

```text
External Reference
    → Source
    → External Identifier / Reference
```

---

## INV-EXT-002 — External references must not replace internal identity

An external identifier must not become the implicit identity of the Collection Item.

---

## INV-EXT-003 — Invalid external references must be representable

If an external source becomes unavailable or changes its identifier scheme, the Collection Item should remain valid.

Therefore:

> External references are dependencies, not domain identity.

---

# 13. Tag Invariants

## INV-TAG-001 — Tags must be valid within their context

If Tags are managed concepts rather than simple values, an Item may only reference valid Tags.

---

## INV-TAG-002 — Tag duplication should be semantically controlled

If Tags are case-insensitive or normalized, the domain should prevent semantically duplicate Tags.

For example:

```text
"Favourite"
"favourite"
"FAVOURITE"
```

may or may not represent the same tag.

This is an unresolved business rule and must not yet be assumed.

---

# 14. Relationship Invariants

Item relationships require particular care because they connect two potentially independent Item aggregates.

---

## INV-REL-001 — Relationship endpoints must be valid Items

A relationship must connect valid Collection Items.

```text
Relationship
    Item A
    Item B
```

Both endpoints must satisfy domain validity.

---

## INV-REL-002 — Relationship type must be valid

A relationship must use a recognized relationship semantic.

For example:

```text
VARIANT_OF
PART_OF
RELATED_TO
```

The final vocabulary is not yet defined.

---

## INV-REL-003 — Self-reference must be explicitly allowed or prohibited

The domain must decide whether:

```text
Item A → related_to → Item A
```

is valid.

Default modeling assumption:

> Self-referential relationships should be prohibited unless explicitly justified by the domain.

This remains a decision to validate.

---

## INV-REL-004 — Relationship direction must be explicit

Some relationships are naturally directional.

For example:

```text
Item A
    ── PART_OF ──>
Item B
```

Others may be symmetric.

For example:

```text
Item A
    ── RELATED_TO ──
Item B
```

The relationship type must determine the semantics.

---

# 15. Ownership Invariants

Ownership introduces an important distinction between **identity** and **authorization**.

## INV-OWN-001 — A Collection must have a valid owner

The owner must be a valid domain actor.

---

## INV-OWN-002 — Ownership transfer must preserve Collection identity

If ownership can change:

```text
Owner A
    ↓
Collection
    ↓
Owner B
```

the Collection itself remains the same domain Entity.

---

## INV-OWN-003 — Ownership is not equivalent to authentication

The fact that a User authenticated successfully does not by itself imply ownership of a Collection.

Authorization rules must establish whether the actor has the required role.

---

# 16. Configuration Invariants

Collection Configuration potentially determines the schema-like behavior of a Collection.

## INV-CONF-001 — Configuration belongs to a Collection context

Configuration must not accidentally affect unrelated Collections.

---

## INV-CONF-002 — Configuration changes must preserve validity

If configuration changes make existing items invalid, the domain must define the permitted behavior.

This is a critical unresolved question.

For example:

```text
Attribute "Year"
    required
       ↓
Items comply

Configuration changes
    "Year" no longer available
       ↓
What happens to existing values?
```

Possible domain policies include:

- reject the configuration change;
- migrate affected items;
- preserve historical values;
- allow temporarily inconsistent state;
- mark items as requiring attention.

No policy is selected yet.

---

# 17. Cross-Concept Invariants

Some rules involve multiple concepts.

These are particularly important because they often determine Aggregate boundaries.

---

## INV-X-001 — Item Type and Attribute compatibility

```text
Collection
    Configuration
       ↓
Item Type
       ↓
Allowed Attributes
       ↓
Collection Item
```

An Item must not contain attributes forbidden by its applicable type/configuration.

---

## INV-X-002 — Collection and Item consistency

A Collection must not claim membership of an Item that does not belong to that Collection.

Conceptually:

```text
Collection A
    contains
Item X

therefore

Item X.collection = Collection A
```

Whether both sides must be atomically consistent is an Aggregate Boundary decision.

---

## INV-X-003 — Relationship endpoint consistency

A relationship must reference valid item identities.

However, because the Items may belong to different aggregates, the relationship may require a weaker consistency model than the Item itself.

This is a strong candidate for later Aggregate Boundary analysis.

---

# 18. Invariant Strength

Not all invariants have the same consistency requirements.

We classify them as:

### Strong invariant

Must be true immediately after a domain operation.

```text
Operation
    ↓
valid state
```

### Eventual invariant

May temporarily be inconsistent across boundaries but must converge.

```text
Operation
    ↓
temporary state
    ↓
eventual processing
    ↓
valid state
```

### Derived invariant

Can be recalculated from authoritative domain data.

---

# 19. Preliminary Invariant Classification

| Invariant | Type | Likely Owner |
|---|---|---|
| INV-COL-001 | Strong | Collection |
| INV-COL-002 | Strong | Collection |
| INV-COL-003 | Strong | Collection / Item |
| INV-ITEM-001 | Strong | Collection Item |
| INV-ITEM-002 | Strong | Collection Item |
| INV-ITEM-003 | Strong | Collection / Configuration |
| INV-ITEM-004 | Strong | Collection Item |
| INV-ITEM-005 | Strong | Collection Item / Configuration |
| INV-ITEM-006 | Strong | Collection Item |
| INV-ITEM-007 | Strong | Collection Item |
| INV-ATTR-001 | Strong | Attribute |
| INV-ATTR-002 | Strong | Attribute / Item |
| INV-ATTR-003 | Strong | Configuration / Item |
| INV-EXT-001 | Strong | External Reference |
| INV-EXT-002 | Strong | Item |
| INV-REL-001 | Strong or Eventual | Relationship |
| INV-REL-002 | Strong | Relationship |
| INV-REL-003 | Strong | Relationship |
| INV-OWN-001 | Strong | Collection |
| INV-OWN-002 | Strong | Collection |
| INV-CONF-001 | Strong | Collection |
| INV-CONF-002 | TBD | Collection / Configuration |
| INV-X-001 | Strong | Configuration / Item |
| INV-X-002 | Strong | Collection / Item |
| INV-X-003 | Potentially Eventual | Relationship |

---

# 20. Preliminary Rule Ownership

A critical principle emerges from the analysis:

> **The concept that owns an invariant should be the primary guardian of that invariant.**

For example:

```text
Item lifecycle
      ↓
Collection Item
```

not:

```text
Item lifecycle
      ↓
Application Service
```

Similarly:

```text
Collection ownership
      ↓
Collection
```

rather than:

```text
Database constraint
```

wherever the rule has genuine domain meaning.

---

# 21. Aggregate Boundary Signals

The invariant analysis gives us the first evidence for Aggregate boundaries.

### Strong signals for Collection

- ownership;
- collection identity;
- collection configuration;
- collection-level rules.

### Strong signals for Collection Item

- item identity;
- lifecycle;
- attributes;
- metadata;
- tags;
- item status.

### Weak signal for Collection containing all Items

The fact that a Collection contains Items does **not** automatically mean all Items belong to the same Aggregate.

Containment is not equivalent to consistency boundary.

---

# 22. Preliminary Boundary Hypothesis

The current evidence favors:

```text
Collection Aggregate
    │
    ├── Collection
    └── Collection Configuration


Collection Item Aggregate
    │
    └── Collection Item
         ├── Attribute Values
         ├── Metadata
         ├── Tags
         ├── External References
         └── Status


Relationship
    │
    └── references Item identities
```

This remains a hypothesis.

The next document will test it systematically.

---

# 23. Open Questions That Block Final Invariant Decisions

The following questions must be answered before Aggregate boundaries can be finalized.

## Q-001

Can a Collection contain thousands or millions of Items?

**Why it matters:** aggregate size and consistency cost.

---

## Q-002

Can an Item exist independently of a Collection?

**Why it matters:** determines ownership and lifecycle boundaries.

---

## Q-003

Can an Item move between Collections?

**Why it matters:** may require coordinated changes across aggregates.

---

## Q-004

Are Item Types global or Collection-specific?

**Why it matters:** determines configuration ownership.

---

## Q-005

Are Attributes global or Collection-specific?

**Why it matters:** determines whether Attribute definitions belong to Collection Configuration.

---

## Q-006

Can configuration changes invalidate existing Items?

**Why it matters:** determines strong consistency requirements.

---

## Q-007

Can relationships connect Items from different Collections?

**Why it matters:** strongly affects relationship boundaries.

---

## Q-008

Can users collaborate on the same Collection?

**Why it matters:** ownership and authorization model.

---

## Q-009

Are Tags global or Collection-specific?

**Why it matters:** identity and ownership.

---

## Q-010

Are Item Relationships first-class domain objects?

**Why it matters:** determines whether they require independent identity and lifecycle.

---

# 24. Decisions

The following decisions are now sufficiently supported:

### D-INV-001

Collection has stable independent identity.

### D-INV-002

Collection Item has stable independent identity.

### D-INV-003

External identifiers do not define internal item identity.

### D-INV-004

Attribute Values do not currently require independent identity.

### D-INV-005

Item lifecycle rules belong conceptually to Collection Item.

### D-INV-006

Collection ownership belongs conceptually to Collection.

### D-INV-007

Collection configuration is collection-scoped.

### D-INV-008

Item-to-item relationships require explicit semantic modeling and must not be reduced automatically to technical associations.

---

# 25. What We Have Learned

The most important result of this document is not the list of invariants.

It is the discovery that the domain contains **different kinds of consistency**.

We currently see at least three:

```text
1. Collection consistency
   └── ownership + configuration

2. Item consistency
   └── identity + lifecycle + item data

3. Relationship consistency
   └── semantic links between items
```

This is a strong indication that the domain should not be modeled as one giant object graph.

---

# 26. Next Step

The next modeling artifact is:

```text
03_AGGREGATE_BOUNDARY_ANALYSIS.md
```

Its purpose will be to take the evidence gathered here and evaluate candidate Aggregate boundaries systematically.

The analysis will compare:

```text
Option A
Collection contains Items

Option B
Collection and Item are separate Aggregates

Option C
Hybrid model
```

For each option we will evaluate:

- invariants;
- transaction boundaries;
- ownership;
- references;
- lifecycle;
- scalability;
- consistency;
- coupling;
- domain behavior;
- failure scenarios.

Only after that analysis should we commit to the Aggregate model.

No database schema, ORM mapping, API contract, or code should be created before this decision is made.