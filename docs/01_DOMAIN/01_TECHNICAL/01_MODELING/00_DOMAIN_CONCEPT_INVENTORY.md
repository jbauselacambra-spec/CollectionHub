# Domain Concept Inventory

> **Documento:** `00_DOMAIN_CONCEPT_INVENTORY.md`  
> **Fase:** 2.1 — Domain Modeling  
> **Estado:** Draft  
> **Propósito:** Inventario canónico de conceptos del dominio de CollectionHub.

---

## 1. Purpose

This document establishes the vocabulary of the CollectionHub domain.

Its purpose is to identify and describe the concepts that exist in the business domain **before translating them into software structures** such as classes, entities, database tables, repositories, APIs, or framework-specific models.

The inventory acts as the foundation for subsequent domain-modeling decisions.

The guiding principle is:

> **Understand the domain first; model the software second.**

This document therefore focuses on:

- what concepts exist;
- what each concept means;
- why it exists;
- how concepts relate to one another;
- which concepts have independent identity;
- which concepts represent values rather than entities;
- which concepts are collections, classifications, or supporting concepts;
- which concepts are still uncertain and require further analysis.

It intentionally does **not** define implementation details.

---

# 2. Domain Vocabulary

The following concepts currently form the working vocabulary of CollectionHub.

| ID | Concept | Category | Status |
|---|---|---|---|
| DC-001 | Collection | Core concept | Confirmed |
| DC-002 | Collection Item | Core concept | Confirmed |
| DC-003 | Item Type | Classification | Confirmed |
| DC-004 | Attribute | Supporting concept | Confirmed |
| DC-005 | Attribute Value | Value concept | Confirmed |
| DC-006 | Collection Owner | Actor / Role | Confirmed |
| DC-007 | User | Actor | Confirmed |
| DC-008 | Metadata | Supporting concept | Confirmed |
| DC-009 | External Reference | Supporting concept | Confirmed |
| DC-010 | Source | Supporting concept | Confirmed |
| DC-011 | Tag | Classification | Confirmed |
| DC-012 | Item Status | Classification | Confirmed |
| DC-013 | Item Relationship | Domain relationship | Confirmed |
| DC-014 | Collection Configuration | Supporting concept | Candidate |
| DC-015 | Item Identifier | Value concept | Candidate |

The inventory is deliberately conservative. A concept should only be promoted to the model when its domain meaning is sufficiently understood.

---

# 3. Core Concepts

## DC-001 — Collection

### Definition

A **Collection** is a logical set of items intentionally grouped together and managed as a coherent unit.

A collection provides the organizational boundary within which items are catalogued, described, classified, and related.

### Responsibilities

A Collection conceptually:

- groups items;
- establishes an organizational context;
- may define the type of collection being managed;
- may have an owner;
- may provide configuration governing its items;
- provides the boundary for collection-specific organization and metadata.

### Identity

A Collection has an independent identity.

Two collections containing similar or even identical items are still different collections.

### Key relationships

```text
Collection
 ├── belongs to → Collection Owner
 ├── contains → Collection Item
 ├── may define → Collection Configuration
 └── may use → Item Type / Attribute definitions
```

### Open questions

- Can a collection contain items of multiple item types?
- Can a collection be shared by multiple users?
- Can an item belong to more than one collection?
- Is collection configuration immutable after creation?
- Can collections contain other collections?

These questions must be resolved before finalizing aggregate boundaries.

---

## DC-002 — Collection Item

### Definition

A **Collection Item** represents an individual object recorded inside a Collection.

It is the primary unit of cataloguing within the system.

An item represents the thing being collected rather than the collection itself.

### Responsibilities

A Collection Item conceptually:

- belongs to a collection;
- identifies the collected object;
- contains descriptive information;
- may have typed attributes;
- may have metadata;
- may reference external sources;
- may have tags;
- may have a lifecycle/status;
- may participate in relationships with other items.

### Identity

A Collection Item has an independent identity within the domain.

The identity of an item must not be confused with any external identifier belonging to the real-world object.

### Key relationships

```text
Collection Item
 ├── belongs to → Collection
 ├── has → Item Type
 ├── has → Attributes
 ├── has → Metadata
 ├── has → External References
 ├── has → Tags
 ├── has → Item Status
 └── relates to → other Collection Items
```

### Important distinction

The following concepts must remain separate:

```text
Collection Item Identity
        ≠
Real-world object identity
        ≠
External source identifier
```

This distinction is important because an item may reference an object using several external systems while still representing a single domain item inside CollectionHub.

---

# 4. Classification Concepts

## DC-003 — Item Type

### Definition

An **Item Type** classifies the kind of object represented by a Collection Item.

Examples may include different categories of collectible objects, depending on the collection domain.

### Purpose

Item Type provides semantic classification and allows the system to determine which characteristics are relevant to an item.

### Key relationships

```text
Item Type
    ↓
Collection Item
```

### Open questions

- Are item types global or collection-specific?
- Can users create custom item types?
- Can an item have more than one type?
- Can types form a hierarchy?

---

## DC-011 — Tag

### Definition

A **Tag** is a lightweight classification label that can be associated with one or more collection items.

Unlike Item Type, a tag does not define what an item fundamentally is.

It provides additional organization, filtering, or discovery semantics.

### Key distinction

```text
Item Type
    = semantic classification

Tag
    = flexible descriptive classification
```

---

## DC-012 — Item Status

### Definition

**Item Status** represents the relevant state of a Collection Item within its lifecycle.

Examples may include states such as:

- active;
- archived;
- unavailable;
- pending;
- removed.

The definitive state vocabulary remains subject to domain validation.

### Important consideration

Status is not merely presentation data.

If status influences permitted domain operations, it should be treated as a meaningful domain concept and potentially as part of the item's state machine.

---

# 5. Attribute Concepts

## DC-004 — Attribute

### Definition

An **Attribute** describes a characteristic that may be associated with a Collection Item.

Attributes allow CollectionHub to represent domain information without requiring every possible characteristic to become a hard-coded property of the item model.

Examples could include:

- manufacturer;
- edition;
- year;
- material;
- condition;
- dimensions;
- serial number.

The actual vocabulary will depend on the supported collection types.

### Responsibilities

An Attribute definition determines:

- the semantic meaning of the characteristic;
- its name;
- its applicable item types;
- potentially its value type;
- potentially whether it is required or optional.

---

## DC-005 — Attribute Value

### Definition

An **Attribute Value** represents the concrete value assigned to an Attribute for a particular Collection Item.

Conceptually:

```text
Attribute
    +
Collection Item
    ↓
Attribute Value
```

### Important distinction

An Attribute is a **definition**.

An Attribute Value is an **instance of information** assigned to an item.

This distinction is fundamental to the modeling of dynamic item characteristics.

---

# 6. Actors

## DC-007 — User

### Definition

A **User** represents an individual interacting with CollectionHub.

A user may create, own, manage, or otherwise interact with collections and their contents, subject to the authorization model.

### Important modeling principle

Authentication and authorization mechanisms are implementation concerns.

The domain concept of User should not be coupled to a specific identity provider, authentication framework, token format, or persistence mechanism.

---

## DC-006 — Collection Owner

### Definition

**Collection Owner** represents the domain role responsible for a Collection.

The owner may be a User, but the concepts should not automatically be treated as identical.

```text
User
  ↓ performs role
Collection Owner
  ↓
Collection
```

Keeping the role conceptually separate allows the model to evolve if ownership later supports teams, organizations, shared ownership, or delegated management.

---

# 7. Supporting Concepts

## DC-008 — Metadata

### Definition

**Metadata** represents descriptive information associated with a Collection Item that provides contextual information about the item.

Metadata should be distinguished from the item's core identity and from dynamically defined attributes.

Potential metadata may include:

- descriptive text;
- creation or acquisition information;
- timestamps;
- provenance information;
- notes;
- supplementary descriptions.

The exact boundary between Metadata and Attribute Value requires further domain clarification.

---

## DC-009 — External Reference

### Definition

An **External Reference** identifies or links a Collection Item to information maintained outside CollectionHub.

Examples may include:

- external catalogue identifiers;
- URLs;
- identifiers from third-party services;
- references to external databases.

### Key principle

An external reference does not define the identity of the Collection Item.

It is an association between the Collection Item and an external system.

---

## DC-010 — Source

### Definition

A **Source** identifies the origin from which information about an item or reference was obtained.

A Source may represent an external catalogue, database, website, provider, or other information origin.

### Relationship

```text
Source
   ↓
External Reference
   ↓
Collection Item
```

The exact relationship between Source and External Reference remains subject to refinement.

---

## DC-013 — Item Relationship

### Definition

An **Item Relationship** represents a meaningful domain relationship between two Collection Items.

Examples might include relationships such as:

```text
Item A
   ├── is part of → Item B
   ├── variant of → Item B
   ├── related to → Item B
   └── replaces → Item B
```

The relationship itself may carry semantic meaning and therefore should not necessarily be reduced to a simple foreign-key association.

### Open questions

- Are relationships directional?
- Can relationships have types?
- Can relationships contain metadata?
- Are inverse relationships automatically derived?
- Can an item relate to itself?

---

# 8. Configuration Concepts

## DC-014 — Collection Configuration

### Definition

**Collection Configuration** represents the rules and preferences that determine how a particular Collection is organized and managed.

Potential configuration may include:

- available item types;
- available attributes;
- required attributes;
- permitted statuses;
- collection-specific classification rules.

### Modeling note

Configuration must be distinguished from domain data.

For example:

```text
"Year is a required attribute"
```

is configuration.

```text
"This item was produced in 1987"
```

is domain data.

This distinction will become important when defining aggregate boundaries and persistence responsibilities.

---

# 9. Identity Concepts

## DC-015 — Item Identifier

### Definition

An **Item Identifier** represents an identifier used to uniquely refer to a Collection Item within CollectionHub or within a relevant domain scope.

The exact identifier strategy remains an implementation-independent modeling concern at this stage.

### Important distinction

Several identifiers may coexist:

```text
CollectionHub Item Identifier
External Identifier
Real-world Identifier
```

They must not be implicitly treated as interchangeable.

---

# 10. Conceptual Relationships

The current conceptual model can be summarized as follows:

```text
                         ┌──────────┐
                         │   User   │
                         └────┬─────┘
                              │
                              │ owns/manages
                              ▼
                       ┌──────────────┐
                       │  Collection  │
                       └──────┬───────┘
                              │
                              │ contains
                              ▼
                    ┌──────────────────┐
                    │ Collection Item  │
                    └───────┬──────────┘
                            │
            ┌───────────────┼────────────────┐
            │               │                │
            ▼               ▼                ▼
       Item Type        Attributes         Tags
                            │
                            ▼
                      Attribute Value

                    Collection Item
                         │
             ┌───────────┼────────────┐
             │           │            │
             ▼           ▼            ▼
         Metadata    External      Item
                     Reference   Relationship
                         │
                         ▼
                       Source
```

This diagram is intentionally conceptual.

It does not imply:

- database tables;
- ORM relationships;
- class inheritance;
- aggregate boundaries;
- API resources;
- persistence strategy.

Those decisions belong to later modeling stages.

---

# 11. Important Domain Distinctions

Several distinctions have already emerged as particularly important.

## 11.1 Collection vs Collection Item

A Collection is the organizational container.

A Collection Item is an individual collected object.

```text
Collection
    contains
Collection Items
```

They must not be collapsed into a single concept.

---

## 11.2 Item Type vs Tag

An Item Type answers:

> "What kind of thing is this?"

A Tag answers:

> "What additional classification or characteristic is useful for organizing this thing?"

These concepts have different semantic responsibilities.

---

## 11.3 Attribute vs Attribute Value

An Attribute defines a characteristic.

An Attribute Value supplies the characteristic's value for an item.

```text
Attribute definition
        +
Item
        =
Attribute Value
```

---

## 11.4 Internal Identity vs External Identity

CollectionHub's identity for an item must remain independent from identifiers supplied by external systems.

```text
Internal identity
        ≠
External identity
```

This prevents external systems from defining the identity model of the CollectionHub domain.

---

## 11.5 Domain Data vs Configuration

Configuration determines what information or behavior is permitted.

Domain data represents actual collected information.

```text
Configuration
    → defines the model

Domain Data
    → represents the instance
```

---

# 12. Candidate Concepts Not Yet Promoted

The following concepts may eventually become part of the domain model but should not yet be promoted without additional evidence.

| Candidate | Reason for caution |
|---|---|
| Acquisition | May be relevant, but requires explicit acquisition workflows |
| Location | Physical/digital location semantics need clarification |
| Condition | Could be an Attribute rather than a first-class concept |
| Valuation | Requires financial/business rules |
| Media | May become a dedicated concept if images/files have domain significance |
| Note | May remain part of metadata |
| History | Could emerge from lifecycle/event modeling |
| Permission | Primarily authorization-related unless domain rules require otherwise |
| Organization | Only necessary if collections can belong to organizations |
| Collection Membership | May become relevant if sharing/collaboration is introduced |

The principle is:

> **Do not model a concept as first-class merely because it can exist technically. Model it when the domain gives it independent meaning or behavior.**

---

# 13. Conceptual Invariants Identified So Far

The following invariants are candidates for later formalization:

1. A Collection Item belongs to a Collection.
2. A Collection has an independent identity.
3. A Collection Item has an independent identity.
4. External identifiers do not define CollectionHub item identity.
5. An Attribute definition is different from an Attribute Value.
6. An Item Type is different from a Tag.
7. Collection configuration is different from collection data.
8. An Item Relationship represents a domain relationship rather than merely a persistence association.
9. A User and the Collection Owner role are conceptually distinct.
10. Supporting concepts should not become entities solely because they require persistence.

These invariants will be revisited when aggregate boundaries and domain rules are defined.

---

# 14. Unresolved Domain Questions

The inventory is intentionally not considered final.

The following questions require resolution during subsequent modeling work:

### Collections

- Can one user own multiple collections?
- Can collections be shared?
- Can an item belong to multiple collections?
- Can collections have different schemas?
- Can collections be nested?

### Items

- Is an item always owned by exactly one collection?
- Can an item be moved between collections?
- What defines item uniqueness?
- Which item properties are universally applicable?

### Types and Attributes

- Are Item Types global or collection-specific?
- Who defines Item Types?
- Who defines Attributes?
- Can users create custom Attributes?
- Can Attributes be inherited from Item Types?
- Which attributes are required?

### Relationships

- Which item relationships are valid?
- Are relationships directional?
- Are relationship types extensible?
- Does a relationship have its own lifecycle?

### Ownership and Access

- Is ownership singular or shared?
- Can ownership be transferred?
- Are collaboration and permissions part of the domain or infrastructure?

### External Sources

- Can one item have multiple external references?
- Is Source an independent domain concept?
- Can external references become invalid?
- Is provenance part of the domain model?

---

# 15. Modeling Rules for Subsequent Phases

The following rules should guide the next modeling steps.

### Rule 1 — Do not translate concepts directly into classes

A domain concept does not automatically imply a class.

```text
Domain Concept
      ↓
Domain Analysis
      ↓
Entity / Value Object / Aggregate / Service / Policy
```

---

### Rule 2 — Identity determines modeling direction

Before treating a concept as an Entity, determine whether it has an identity that matters independently over time.

---

### Rule 3 — Behavior matters as much as data

A concept with meaningful domain behavior may deserve a stronger model than a concept that merely stores information.

---

### Rule 4 — Avoid premature normalization

The domain model must not be shaped by presumed database tables.

---

### Rule 5 — Avoid infrastructure leakage

Concepts such as:

- ORM models;
- repositories;
- API DTOs;
- database records;
- authentication tokens;

must not be introduced into this inventory as domain concepts unless they have genuine business meaning.

---

# 16. Current Domain Map

At the current stage, CollectionHub can be understood through the following conceptual chain:

```text
User
  │
  │ owns/manages
  ▼
Collection
  │
  │ contains
  ▼
Collection Item
  │
  ├── classified by → Item Type
  ├── described by → Attributes / Attribute Values
  ├── organized by → Tags
  ├── governed by → Item Status
  ├── enriched by → Metadata
  ├── linked externally through → External References
  └── related to → Other Collection Items
```

This is the current working vocabulary of the domain.

It should evolve as domain rules become clearer.

---

# 17. Status

**Current status:** Working Domain Inventory

This document is considered sufficiently mature to support the next modeling activity, but **not frozen**.

New concepts may be added.

Existing concepts may be:

- renamed;
- merged;
- split;
- downgraded;
- promoted to first-class domain concepts;
- removed when domain analysis disproves their necessity.

The inventory should therefore be treated as a **living domain artifact**, not as an implementation specification.

---

## Next Step

The next modeling activity should use this inventory to determine the nature of each important concept:

```text
Domain Concept Inventory
          ↓
Entity / Value Object analysis
          ↓
Identity analysis
          ↓
Aggregate candidate analysis
          ↓
Domain invariants
          ↓
Domain model
```

No implementation should be derived directly from this document until those decisions have been explicitly analyzed.