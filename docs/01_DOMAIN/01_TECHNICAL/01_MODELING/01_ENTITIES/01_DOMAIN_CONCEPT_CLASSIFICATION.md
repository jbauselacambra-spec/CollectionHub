# Domain Concept Classification

> **Documento:** `01_DOMAIN_CONCEPT_CLASSIFICATION.md`  
> **Fase:** 2.1 — Domain Modeling  
> **Estado:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`  
> **Purpose:** Classify domain concepts according to their semantic role, identity, lifecycle, and behavioral significance.

---

# 1. Purpose

This document classifies the concepts identified in the Domain Concept Inventory.

The objective is not to translate concepts directly into implementation structures.

Instead, each concept is analyzed according to questions such as:

- Does it have its own identity?
- Does that identity matter over time?
- Does it have an independent lifecycle?
- Is it defined by its attributes rather than by identity?
- Can it exist independently?
- Does it participate in domain invariants?
- Is it merely a classification?
- Is it configuration rather than domain data?
- Could it represent a boundary of consistency?

The resulting classification will provide the foundation for:

- Entity identification;
- Value Object identification;
- Aggregate Candidate identification;
- Aggregate boundary analysis;
- domain invariant analysis;
- eventual domain service identification.

---

# 2. Classification Vocabulary

The following categories are used.

## 2.1 Entity

A domain concept is considered an **Entity** when:

- it has a meaningful identity;
- that identity persists independently of changes to its attributes;
- the domain cares about distinguishing one instance from another.

An Entity is therefore defined primarily by **identity and continuity**, not by its current values.

---

## 2.2 Value Object

A **Value Object** is defined by its value rather than by independent identity.

Two instances representing the same value are semantically interchangeable.

Typical characteristics:

- immutable or effectively immutable;
- no meaningful independent lifecycle;
- equality based on value;
- normally owned by another domain concept.

---

## 2.3 Aggregate Candidate

An **Aggregate Candidate** is a concept that may define a consistency boundary.

It is not automatically an Aggregate merely because it is an Entity.

A candidate becomes an Aggregate only after we establish:

- which invariants it protects;
- which objects must change together;
- what constitutes its boundary;
- what must be accessed through its root.

---

## 2.4 Classification Concept

A classification concept describes or categorizes other concepts.

Examples:

- Item Type;
- Tag;
- Item Status.

Classification concepts may themselves have identity, but their semantic role must be distinguished from the object being classified.

---

## 2.5 Supporting Concept

A Supporting Concept provides information or context to core domain concepts but does not necessarily possess independent domain behavior.

---

## 2.6 Actor / Role

An Actor represents something capable of participating in domain interactions.

A Role represents a responsibility assumed by an actor in a specific context.

---

## 2.7 Configuration Concept

A Configuration Concept describes rules or options that determine how a domain area behaves.

Configuration must be distinguished from the actual domain data governed by that configuration.

---

## 2.8 Relationship Concept

A Relationship Concept represents a meaningful association between domain concepts where the association itself carries semantic meaning.

---

# 3. Classification Summary

| ID | Concept | Primary Classification | Identity | Lifecycle | Aggregate Candidate |
|---|---|---|---|---|---|
| DC-001 | Collection | Entity | Yes | Yes | **Yes** |
| DC-002 | Collection Item | Entity | Yes | Yes | **Yes** |
| DC-003 | Item Type | Classification Concept | TBD | TBD | No |
| DC-004 | Attribute | Entity / Definition | Potentially | Potentially | No |
| DC-005 | Attribute Value | Value Object / Instance | No | No independent lifecycle | No |
| DC-006 | Collection Owner | Role | No independent identity | Contextual | No |
| DC-007 | User | Entity / Actor | Yes | Yes | No |
| DC-008 | Metadata | Supporting Concept | No independent identity | No | No |
| DC-009 | External Reference | Value Object candidate | Probably no | No independent lifecycle | No |
| DC-010 | Source | Supporting Entity candidate | TBD | TBD | No |
| DC-011 | Tag | Classification Concept | TBD | TBD | No |
| DC-012 | Item Status | Value / State Concept | No independent identity | No | No |
| DC-013 | Item Relationship | Relationship Concept | Potentially | Potentially | **Candidate** |
| DC-014 | Collection Configuration | Configuration Concept | Collection-scoped | Collection lifecycle | **Part of Collection** |
| DC-015 | Item Identifier | Value Object | No | No | No |

This table is a **working classification**, not a final domain model.

---

# 4. Detailed Classification

# 4.1 Collection

**Concept:** DC-001

**Classification:** Entity + Aggregate Candidate

## Identity

A Collection has an identity that remains meaningful independently of its current contents.

Two collections containing exactly the same items are still different collections.

Therefore:

> Collection identity is intrinsic to the domain.

## Lifecycle

A Collection has an independent lifecycle.

Potential lifecycle events include:

- creation;
- modification;
- ownership changes;
- archival;
- deletion.

The exact lifecycle will be defined later.

## Aggregate candidacy

Collection is a strong Aggregate Candidate.

The reason is not simply that it is an Entity.

The stronger argument is that the Collection may be responsible for invariants concerning:

- which items belong to it;
- collection-specific configuration;
- item-type applicability;
- attribute requirements;
- collection ownership.

However, this must be validated against the eventual requirements.

## Current classification

```text
Collection
    Entity
       +
Aggregate Candidate
```

---

# 4.2 Collection Item

**Concept:** DC-002

**Classification:** Entity + Aggregate Candidate

## Identity

A Collection Item has identity that distinguishes it from other items.

The item identity must remain stable even if:

- metadata changes;
- attributes change;
- tags change;
- external references change.

Therefore identity is not derived from the item's mutable descriptive data.

## Lifecycle

An item can potentially experience meaningful state transitions.

For example:

```text
Created
   ↓
Active
   ↓
Archived
```

The final lifecycle is not yet defined.

## Aggregate candidacy

Collection Item is a strong Aggregate Candidate.

The main question is whether it should be:

```text
Collection
   └── Collection Item
```

inside a single aggregate,

or whether:

```text
Collection Aggregate
Collection Item Aggregate
```

should exist as separate consistency boundaries.

This is deliberately unresolved.

The answer must come from invariant analysis rather than from database convenience.

## Current classification

```text
Collection Item
       Entity
          +
Aggregate Candidate
```

---

# 4.3 Item Type

**Concept:** DC-003

**Classification:** Classification Concept

Item Type primarily answers:

> What kind of object does this item represent?

It does not represent the item itself.

## Identity

Identity is currently unresolved.

If Item Types are predefined system concepts, they may behave like reference data.

If users can create and manage them, they may become proper Entities.

## Current decision

Do not promote Item Type to Aggregate or Core Entity yet.

## Current classification

```text
Item Type
    Classification Concept
```

---

# 4.4 Attribute

**Concept:** DC-004

**Classification:** Entity / Definition Candidate

An Attribute is more interesting than it initially appears.

It represents a definition of a characteristic rather than the characteristic's concrete value.

For example:

```text
Attribute:
    "Year"

Attribute Value:
    "1987"
```

## Identity

An Attribute may require identity if:

- it is reused by many items;
- it has its own definition;
- it has configuration;
- it can be renamed;
- it has applicable item types;
- it has validation rules.

Therefore an Attribute is currently treated as an **Entity Candidate**.

## Aggregate candidacy

No evidence currently suggests that Attribute should form its own Aggregate.

It is more likely to be:

```text
Collection Configuration
       │
       └── Attribute Definition
```

The final decision depends on whether attributes are globally reusable or collection-scoped.

---

# 4.5 Attribute Value

**Concept:** DC-005

**Classification:** Value Object Candidate

An Attribute Value represents a concrete value assigned to an Attribute for an Item.

Its meaning comes from:

```text
Item + Attribute + Value
```

It does not currently appear to have an independent lifecycle.

Therefore it should not receive independent identity without strong domain evidence.

## Current classification

```text
Attribute Value
      Value Object candidate
```

---

# 4.6 Collection Owner

**Concept:** DC-006

**Classification:** Role

Collection Owner is not currently treated as an Entity.

The concept represents a responsibility assumed by an actor.

For example:

```text
User
  ↓
acts as
  ↓
Collection Owner
```

The ownership role may eventually become part of an authorization or collaboration model.

## Current classification

```text
Collection Owner
       Role
```

---

# 4.7 User

**Concept:** DC-007

**Classification:** Entity / Actor

A User has independent identity and lifecycle.

The domain may need to distinguish one user from another independently of mutable profile information.

## Important boundary

The domain concept User should remain independent from:

- authentication providers;
- login credentials;
- OAuth identities;
- access tokens;
- password hashes.

Those belong to other concerns unless explicit domain requirements state otherwise.

## Aggregate candidacy

User is currently **not** considered an Aggregate Candidate for CollectionHub's core domain.

This does not prevent User from being an Aggregate in a broader identity/bounded context.

---

# 4.8 Metadata

**Concept:** DC-008

**Classification:** Supporting Concept

Metadata currently lacks evidence of independent identity or lifecycle.

It appears to describe an item rather than represent an independently managed object.

## Current classification

```text
Metadata
   Supporting Concept
```

The exact boundary between Metadata and Attribute Values remains an open modeling question.

---

# 4.9 External Reference

**Concept:** DC-009

**Classification:** Value Object Candidate

An External Reference is primarily defined by its value:

```text
Source
+
External Identifier
+
Reference Information
```

The domain currently does not indicate that an external reference requires independent identity.

Therefore the preferred direction is:

```text
External Reference
       Value Object
```

unless later requirements introduce independent behavior or lifecycle.

---

# 4.10 Source

**Concept:** DC-010

**Classification:** Entity Candidate / Supporting Concept

Source is currently ambiguous.

If Source is simply:

```text
"IMDb"
"Wikipedia"
"External Catalogue X"
```

it may behave as reference data.

If the domain needs to manage:

- source configuration;
- source reliability;
- source lifecycle;
- source-specific policies;
- source synchronization;

then Source becomes a meaningful Entity.

## Current decision

Keep Source as an **Entity Candidate**, but do not promote it yet.

---

# 4.11 Tag

**Concept:** DC-011

**Classification:** Classification Concept

A Tag is primarily a flexible classification mechanism.

Its semantic importance comes from its association with items.

Whether a Tag has independent identity depends on whether the domain cares about managing the Tag itself.

For example:

```text
Tag = "Favourite"
```

could be:

- a simple value;
- a reusable classification;
- a managed domain object.

The current model does not yet require the strongest interpretation.

## Current classification

```text
Tag
   Classification Concept
```

---

# 4.12 Item Status

**Concept:** DC-012

**Classification:** Value / State Concept

Status represents the current state of an Item.

For example:

```text
ACTIVE
ARCHIVED
```

The status itself does not require independent identity.

However, status transitions may contain domain behavior.

Therefore we must distinguish:

```text
Status
    = state representation

Status Transition
    = potentially meaningful domain behavior
```

The latter may become important during invariant analysis.

---

# 4.13 Item Relationship

**Concept:** DC-013

**Classification:** Relationship Concept + Aggregate Candidate

This is one of the most interesting concepts in the current model.

A relationship may be more than a technical association.

For example:

```text
Item A
    ── variant-of ──>
Item B
```

The relationship itself can have semantic meaning.

## Questions determining its classification

We need to establish whether relationships:

- have their own identity;
- have metadata;
- have lifecycle;
- can be created independently;
- can be invalidated;
- have domain rules.

If yes, Item Relationship may become an Entity.

If not, it may remain a Value Object or part of an Item Aggregate.

## Current classification

```text
Item Relationship
       Relationship Concept
              +
       Aggregate Candidate
```

But this is deliberately provisional.

---

# 4.14 Collection Configuration

**Concept:** DC-014

**Classification:** Configuration Concept

Collection Configuration is not treated as an independent Entity.

Its identity derives from the Collection to which it belongs.

Conceptually:

```text
Collection
    │
    └── Configuration
```

Configuration may therefore belong inside the Collection consistency boundary.

## Current classification

```text
Collection Configuration
       Configuration Concept
              +
       Collection-owned object
```

---

# 4.15 Item Identifier

**Concept:** DC-015

**Classification:** Value Object

An identifier is defined by its value.

For example:

```text
ItemIdentifier("abc-123")
```

Two identifiers with the same valid value represent the same identifier value.

It does not need an independent lifecycle.

## Current classification

```text
Item Identifier
    Value Object
```

---

# 5. Entity Candidates

The current Entity landscape is:

```text id="5v0z7r"
Core Entities
─────────────

Collection
Collection Item
User


Potential Entities
──────────────────

Attribute
Source
Item Relationship


Context-dependent
─────────────────

Item Type
Tag
```

The distinction between **Entity** and **Entity Candidate** is intentional.

We do not want to promote concepts simply because they can be represented as objects in software.

---

# 6. Value Object Candidates

The current Value Object landscape is:

```text id="2i1d4g"
Attribute Value
External Reference
Item Identifier
Item Status
```

Potential future Value Objects may include concepts such as:

- names;
- descriptions;
- URLs;
- dates;
- quantities;
- dimensions;
- identifiers;
- monetary values;
- coordinates.

These should be identified only when their domain semantics become explicit.

---

# 7. Aggregate Candidates

The strongest current Aggregate Candidates are:

```text id="l7q9tq"
Collection
Collection Item
Item Relationship
```

The key question is not:

> "Which objects have a database table?"

The key question is:

> "Which objects define a consistency boundary within which domain invariants must be protected?"

This distinction will drive the next stage.

---

# 8. Preliminary Aggregate Hypotheses

At this point there are several plausible models.

## Hypothesis A — Collection-centered

```text
Collection
 ├── Configuration
 ├── Item definitions
 └── Collection Items
```

This model provides strong consistency around the entire collection.

### Advantages

- straightforward conceptual ownership;
- collection-level rules are easy to enforce;
- configuration and items have an obvious relationship.

### Risks

- large aggregates;
- potentially expensive consistency boundaries;
- difficult scaling if collections become large.

---

## Hypothesis B — Item-centered

```text
Collection
    │
    └── references
          │
          ▼
     Collection Item
          ├── Attributes
          ├── Metadata
          ├── Tags
          └── Status
```

Collection and Collection Item become separate aggregates.

### Advantages

- smaller consistency boundaries;
- item operations remain independent;
- potentially better scalability.

### Risks

- collection-level invariants become harder to enforce atomically;
- configuration changes may require coordination.

---

## Hypothesis C — Hybrid

```text
Collection Aggregate
    ├── Collection Configuration
    └── collection-level rules

Collection Item Aggregate
    ├── Attribute Values
    ├── Metadata
    ├── Tags
    └── Status

Relationship Aggregate / Value
    └── Item-to-item relationships
```

This is currently the **leading hypothesis**, but it is not yet a decision.

The final choice must emerge from invariant analysis.

---

# 9. Entity vs Value Object Decision Rules

The following rules will be used throughout the remainder of the modeling process.

## Rule 1

If the domain cares about **which instance** it is, favor Entity.

## Rule 2

If the domain cares only about **what value it represents**, favor Value Object.

## Rule 3

Do not introduce identity merely because persistence requires a primary key.

## Rule 4

Do not introduce identity merely because a concept has multiple properties.

## Rule 5

If a concept has meaningful lifecycle transitions, investigate Entity classification.

## Rule 6

If a concept is immutable and completely defined by its values, investigate Value Object classification.

---

# 10. Classification Matrix

| Concept | Identity | Mutable | Independent Lifecycle | Semantic Behavior | Current Classification |
|---|---:|---:|---:|---:|---|
| Collection | Yes | Yes | Yes | High | Entity / Aggregate Candidate |
| Collection Item | Yes | Yes | Yes | High | Entity / Aggregate Candidate |
| Item Type | TBD | TBD | TBD | Low–Medium | Classification |
| Attribute | Potentially | Yes | Potentially | Medium | Entity Candidate |
| Attribute Value | No | Preferably No | No | Low | Value Object |
| Collection Owner | No | Contextual | No | Low | Role |
| User | Yes | Yes | Yes | Medium | Entity / Actor |
| Metadata | No | Yes | No | Low | Supporting Concept |
| External Reference | No | Preferably No | No | Low | Value Object Candidate |
| Source | TBD | TBD | TBD | TBD | Entity Candidate |
| Tag | TBD | TBD | TBD | Low | Classification |
| Item Status | No | Yes as state | No | Medium | State / Value |
| Item Relationship | Potentially | Potentially | Potentially | Medium–High | Relationship / Aggregate Candidate |
| Collection Configuration | Collection-scoped | Yes | Collection-scoped | Medium | Configuration |
| Item Identifier | No | No | No | Low | Value Object |

---

# 11. Decisions Made

The following decisions can currently be considered reasonably strong:

### D-001 — Collection is an Entity

Its identity and lifecycle are independent from its contents.

### D-002 — Collection Item is an Entity

Its identity must remain stable independently of mutable descriptive information.

### D-003 — Item Identifier is a Value Object

The domain does not require independent identity for an identifier value.

### D-004 — Attribute Value should not have independent identity

Its meaning derives from the Item and Attribute context.

### D-005 — Collection Owner is a role

It should not automatically become a separate Entity.

### D-006 — External Reference should not define Item identity

External identifiers belong to external systems.

### D-007 — Collection and Collection Item are both Aggregate Candidates

The actual boundaries remain unresolved.

---

# 12. Decisions Deliberately Deferred

The following decisions must **not** be made yet:

- final Aggregate boundaries;
- exact Entity hierarchy;
- Attribute ownership;
- global vs collection-specific Item Types;
- global vs collection-specific Tags;
- Source identity;
- Item Relationship identity;
- collection/item cardinality;
- persistence representation;
- database schema;
- API representation.

Deferring these decisions is intentional.

---

# 13. Modeling Risks

Several risks have been identified.

## Risk R-001 — Database-driven modeling

There is a risk of turning every persistent record into an Entity.

**Mitigation:** identity and domain meaning must justify Entity classification.

---

## Risk R-002 — Generic "Item" model

There is a risk of creating an overly generic Item structure capable of representing everything but expressing little domain meaning.

**Mitigation:** Item Type and Attribute semantics must be analyzed before defining the final model.

---

## Risk R-003 — Overly large Collection Aggregate

Putting all items inside a Collection Aggregate could create a consistency boundary that is too large.

**Mitigation:** determine invariants before selecting the boundary.

---

## Risk R-004 — Anemic domain model

There is a risk of modeling only data structures and leaving all meaningful behavior outside the domain model.

**Mitigation:** subsequent phases must explicitly identify invariants and domain behaviors.

---

## Risk R-005 — Premature abstraction

There is a risk of creating generic abstractions before understanding the actual domain rules.

**Mitigation:** unresolved concepts remain candidates rather than finalized abstractions.

---

# 14. Current Conceptual Classification

The model currently looks like this:

```text
                         ┌────────────┐
                         │    User    │
                         │  Entity    │
                         └─────┬──────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │   Collection    │
                     │ Entity /       │
                     │ Aggregate      │
                     │ Candidate      │
                     └────────┬────────┘
                              │
                              │ contains
                              ▼
                    ┌──────────────────┐
                    │ Collection Item  │
                    │ Entity /         │
                    │ Aggregate        │
                    │ Candidate       │
                    └───────┬──────────┘
                            │
          ┌─────────────────┼──────────────────┐
          │                 │                  │
          ▼                 ▼                  ▼
    Item Type           Attributes           Tags
 Classification       Definitions        Classification
                            │
                            ▼
                     Attribute Value
                       Value Object

                    Collection Item
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
         Metadata    External Ref.   Status
         Supporting  Value Object     State

                    Collection Item
                          │
                          ▼
                 Item Relationship
                 Relationship /
                 Aggregate Candidate
```

---

# 15. Next Modeling Step

The classification exercise gives us enough information to move to the next level.

The next artifact should focus on **identity and invariants**.

Recommended sequence:

```text
00_DOMAIN_CONCEPT_INVENTORY.md
             ↓
01_DOMAIN_CONCEPT_CLASSIFICATION.md
             ↓
02_DOMAIN_IDENTITY_AND_INVARIANTS.md
             ↓
03_AGGREGATE_BOUNDARY_ANALYSIS.md
             ↓
04_DOMAIN_MODEL.md
```

The most important next step is therefore:

> **Determine what must always be true in the CollectionHub domain, who is responsible for enforcing each invariant, and which concepts must change together to preserve consistency.**

That analysis will allow us to stop talking about Aggregate Candidates and start making evidence-based decisions about actual Aggregate boundaries.