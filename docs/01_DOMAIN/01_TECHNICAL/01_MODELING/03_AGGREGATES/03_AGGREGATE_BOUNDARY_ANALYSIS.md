# Aggregate Boundary Analysis

> **Document:** `03_AGGREGATE_BOUNDARY_ANALYSIS.md`  
> **Phase:** 2.1 — Domain Modeling  
> **Status:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`, `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`  
> **Purpose:** Evaluate and define Aggregate boundaries based on domain invariants, ownership, lifecycle, consistency, and transactional requirements.

---

# 1. Purpose

This document evaluates the candidate Aggregate boundaries identified during the previous modeling steps.

The goal is not to determine database relationships.

The goal is to answer:

> **Which objects must form a consistency boundary so that the domain can protect its invariants?**

An Aggregate is therefore treated as:

- a consistency boundary;
- a transactional boundary;
- a domain ownership boundary;
- a mechanism for protecting invariants.

It is **not** treated as:

- a database table;
- an object graph;
- a container for every related concept;
- a synonym for Entity;
- a synonym for persistence.

---

# 2. Aggregate Principles

The CollectionHub model will follow these principles.

## A-001 — Aggregates protect invariants

An Aggregate exists because some rules must be protected together.

---

## A-002 — Aggregates should be as small as practical

If two concepts do not need to change together to preserve a domain invariant, they should not automatically belong to the same Aggregate.

---

## A-003 — External references between Aggregates use identity

An Aggregate should not directly depend on the complete object graph of another Aggregate.

Conceptually:

```text
Aggregate A
    │
    └── references
          Aggregate B identity
```

rather than:

```text
Aggregate A
    │
    └── contains
          entire Aggregate B
```

---

## A-004 — Transactional consistency should be intentional

A requirement for atomic consistency is evidence for an Aggregate boundary.

A mere conceptual relationship is not.

---

## A-005 — Aggregate roots own access to their internals

Objects inside an Aggregate should not become independently manipulated domain objects unless the domain explicitly requires such independence.

---

## A-006 — Persistence must follow the domain model

The database must not determine the Aggregate structure.

---

# 3. Candidate Aggregates

Based on the previous analysis, the main candidates are:

```text id="v3fq0w"
Candidate A
Collection Aggregate
    └── all Collection Items

Candidate B
Collection Aggregate
    └── Collection Configuration

Collection Item Aggregate
    └── Item data

Candidate C
Relationship Aggregate
    └── Item-to-item relationship
```

We will evaluate these independently.

---

# 4. Candidate A — Collection Contains All Items

The first possible model is:

```text id="3w3y1c"
Collection
  │
  ├── Configuration
  │
  ├── Item 1
  ├── Item 2
  ├── Item 3
  ├── ...
  └── Item N
```

Here the Collection is the Aggregate Root and every Item is part of the same Aggregate.

---

# 5. Advantages of Candidate A

## 5.1 Simple ownership model

The conceptual ownership is straightforward:

```text id="9kwm8m"
Collection
    owns
Items
```

---

## 5.2 Strong collection-wide consistency

Any rule involving Collection and Item could potentially be enforced atomically.

For example:

```text id="m74zpd"
Add Item
    ↓
Validate Collection Configuration
    ↓
Validate Item
    ↓
Commit
```

---

## 5.3 Easy reasoning for small collections

If collections are always small, the model may be perfectly reasonable.

---

# 6. Problems with Candidate A

The model becomes problematic if collections can grow significantly.

Imagine:

```text id="x1z7rx"
Collection
    100 items
    1,000 items
    10,000 items
    100,000+ items
```

A single Aggregate would imply that all those items conceptually belong to one consistency boundary.

This introduces several risks.

---

## 6.1 Excessive Aggregate size

Large Aggregates are expensive to load, reason about, and modify.

---

## 6.2 Unnecessary contention

Two unrelated operations:

```text id="k7ph4j"
User A
    edits Item 1

User B
    edits Item 2
```

could conceptually target the same Aggregate.

That is undesirable when the operations do not share invariants.

---

## 6.3 Artificial coupling

Changing one Item should not necessarily require coordination with every other Item.

---

## 6.4 Poor scalability

The domain should not assume that the entire Collection must participate in every Item operation.

---

# 7. Candidate A Evaluation

| Criterion | Result |
|---|---|
| Conceptual simplicity | Excellent |
| Strong consistency | Excellent |
| Aggregate size | Poor |
| Scalability | Poor |
| Independent item operations | Poor |
| Concurrency | Poor |
| Domain flexibility | Medium |
| Long-term suitability | Low |

### Conclusion

**Rejected as the default model.**

Candidate A may only become valid if future requirements explicitly establish strong collection-wide invariants requiring all Items to change atomically.

Current evidence does not support that requirement.

---

# 8. Candidate B — Separate Collection and Item Aggregates

The second model is:

```text id="3td2h6"
Collection Aggregate
    │
    └── Collection Configuration


Collection Item Aggregate
    │
    ├── Attributes
    ├── Metadata
    ├── Tags
    ├── Status
    └── External References
```

The Collection references Items by identity.

---

# 9. Collection Aggregate

The Collection Aggregate would contain:

```text id="xw2u3h"
Collection
 ├── identity
 ├── owner
 └── configuration
```

Potential Collection invariants:

- valid identity;
- valid ownership;
- valid collection configuration;
- valid configuration transitions.

---

# 10. Collection Item Aggregate

The Collection Item Aggregate would contain:

```text id="y6h6qz"
Collection Item
 ├── identity
 ├── collection identity
 ├── item type
 ├── attribute values
 ├── metadata
 ├── tags
 ├── status
 └── external references
```

Potential Item invariants:

- valid item identity;
- valid lifecycle;
- valid attributes;
- valid status;
- valid external references;
- valid internal item state.

---

# 11. Advantages of Candidate B

## 11.1 Smaller consistency boundaries

Item changes do not require loading the Collection Aggregate.

---

## 11.2 Independent lifecycle

An Item can evolve independently.

---

## 11.3 Better concurrency

Different Items can be modified independently.

---

## 11.4 Better scalability

Large Collections do not imply large Aggregates.

---

## 11.5 Clear domain ownership

The Collection owns collection-level configuration.

The Item owns item-level state.

---

# 12. Problems with Candidate B

The major difficulty is cross-Aggregate invariants.

For example:

```text id="6vhk5g"
Collection Configuration
        ↓
requires Attribute A
        ↓
Collection Item
        ↓
must contain A
```

The Collection and Item are separate Aggregates.

Therefore the rule cannot simply assume that both are changed atomically.

---

# 13. Configuration Change Scenario

Consider:

```text id="7n0qve"
Configuration:
    Attribute "Year" required
```

Existing Item:

```text id="s4v3we"
Item:
    Year = 1987
```

Now configuration changes:

```text id="a1o4kc"
"Year" no longer supported
```

What happens?

Possible policies:

### Policy A

Reject the configuration change.

### Policy B

Allow the change but keep historical values.

### Policy C

Migrate existing Items.

### Policy D

Mark affected Items as requiring attention.

The correct answer is a business rule.

It must not be solved accidentally through Aggregate design.

---

# 14. Candidate B Evaluation

| Criterion | Result |
|---|---|
| Conceptual simplicity | Very good |
| Strong local consistency | Excellent |
| Aggregate size | Excellent |
| Scalability | Excellent |
| Concurrency | Excellent |
| Collection-wide invariants | Medium |
| Domain flexibility | Excellent |
| Long-term suitability | High |

### Conclusion

**Strong candidate.**

---

# 15. Candidate C — Separate Relationship Aggregate

The third question concerns Item Relationships.

Possible model:

```text id="8k1o1f"
Item A
   │
   └── Relationship ──> Item B
```

The relationship itself could be a separate Aggregate.

---

# 16. Why Relationships Are Special

A relationship connects potentially independent Items.

For example:

```text id="zv0d8s"
Collection Item A
       │
       │ VARIANT_OF
       ▼
Collection Item B
```

If A and B are separate Aggregates, forcing them into the same Aggregate would immediately create a large consistency boundary.

That is usually undesirable.

---

# 17. Relationship as Value Object

The simplest option is to treat a relationship as a value:

```text id="g7y8gq"
Relationship(
    sourceItemId,
    targetItemId,
    relationshipType
)
```

This works if the relationship:

- has no independent lifecycle;
- has no meaningful identity;
- has no significant metadata;
- is simply part of an Item's state.

---

# 18. Relationship as Entity

A relationship becomes an Entity if the domain cares about the relationship itself.

For example, if it can:

- be created;
- be approved;
- be invalidated;
- contain metadata;
- have provenance;
- have timestamps;
- be audited;
- have its own lifecycle.

Then:

```text id="1yd1vu"
Relationship
    Entity
```

becomes justified.

---

# 19. Relationship as Independent Aggregate

An independent Relationship Aggregate is justified if the relationship itself must protect invariants.

For example:

```text id="a9c5gc"
Relationship
    ├── source item
    ├── target item
    ├── relationship type
    ├── status
    └── provenance
```

This allows relationship lifecycle to remain independent from both Items.

---

# 20. Current Relationship Decision

There is currently insufficient evidence to make Item Relationship an independent Aggregate.

Therefore:

> **Do not create a Relationship Aggregate yet.**

Instead, treat the relationship as a **domain concept requiring further analysis**.

---

# 21. Cross-Aggregate References

If Collection and Collection Item become separate Aggregates, the model should conceptually use:

```text id="4x0j8d"
Collection
    │
    └── CollectionItemId references


Collection Item
    │
    └── CollectionId references
```

rather than embedding the complete object.

This gives us a useful rule:

> Aggregates reference other Aggregates by identity.

---

# 22. Important Exception — Aggregate Root Does Not Mean Database Root

The Collection Aggregate may reference many Items without containing their full object graphs.

Conceptually:

```text id="rx6f3j"
Collection
 ├── identity
 ├── owner
 └── configuration
       │
       └── references item rules

Collection Item
 ├── identity
 ├── collection identity
 └── item state
```

This is a domain relationship, not a persistence prescription.

---

# 23. Ownership Analysis

The proposed boundaries produce the following ownership model.

| Concept | Owned by |
|---|---|
| Collection | Collection Aggregate |
| Collection Configuration | Collection Aggregate |
| Collection Item | Item Aggregate |
| Attribute Value | Item Aggregate |
| Metadata | Item Aggregate |
| Tags assigned to Item | Item Aggregate |
| Item Status | Item Aggregate |
| External References | Item Aggregate |
| Item Relationship | TBD |
| Attribute Definition | TBD |
| Item Type | TBD |
| Tag Definition | TBD |
| Source | TBD |

This is deliberately conservative.

---

# 24. Aggregate Boundary Test

We can now apply the following test.

For two concepts A and B:

> **Must A and B change atomically to preserve a valid domain state?**

If the answer is **yes**, they are strong candidates for the same Aggregate.

If the answer is **no**, separate Aggregates should be preferred.

---

# 25. Boundary Test Results

## Collection + Collection Configuration

```text
Must change together?
Potentially yes.
```

### Result

Same Aggregate.

---

## Collection + Collection Item

```text
Must change together?
Current evidence: no.
```

### Result

Separate Aggregates.

---

## Collection Item + Attribute Values

```text
Must change together?
Yes, for item validity.
```

### Result

Same Aggregate.

---

## Collection Item + Metadata

```text
Must change together?
Potentially yes.
```

### Result

Same Aggregate for now.

---

## Collection Item + Status

```text
Must change together?
Yes.
```

### Result

Same Aggregate.

---

## Collection Item + External References

```text
Must change together?
Generally yes for item consistency.
```

### Result

Same Aggregate for now.

---

## Item A + Item B

```text
Must change together?
Current evidence: no.
```

### Result

Separate Aggregates.

---

## Item A + Relationship + Item B

```text
Must change together?
Unknown.
```

### Result

Relationship boundary remains unresolved.

---

# 26. Proposed Aggregate Model

Based on current evidence, the recommended model is:

```text id="g4o4lm"
┌──────────────────────────────────────┐
│          COLLECTION AGGREGATE        │
│                                      │
│  Collection                          │
│      │                               │
│      ├── Owner                       │
│      └── Collection Configuration    │
│                                      │
└──────────────────────────────────────┘


┌──────────────────────────────────────┐
│       COLLECTION ITEM AGGREGATE      │
│                                      │
│  Collection Item                     │
│      │                               │
│      ├── Item Type                   │
│      ├── Attribute Values            │
│      ├── Metadata                    │
│      ├── Tags                        │
│      ├── Status                      │
│      └── External References         │
│                                      │
└──────────────────────────────────────┘


┌──────────────────────────────────────┐
│       RELATIONSHIP — TBD             │
│                                      │
│  Item A ── Relationship ──> Item B  │
│                                      │
└──────────────────────────────────────┘
```

---

# 27. Aggregate Root Candidates

The current Aggregate Roots are:

## AR-001 — Collection

Responsibilities:

- maintain Collection identity;
- maintain ownership;
- protect collection-level invariants;
- manage collection configuration.

---

## AR-002 — Collection Item

Responsibilities:

- maintain Item identity;
- maintain Item lifecycle;
- protect item-level invariants;
- maintain item attributes;
- maintain metadata;
- maintain item tags;
- maintain external references.

---

# 28. What Does Not Belong in Collection Aggregate

The following should **not** automatically be loaded into the Collection Aggregate:

- all Collection Items;
- all Item metadata;
- all Item attributes;
- all Item relationships.

The Collection knows enough to manage collection-level rules.

It does not need to become an object graph representing the entire collection.

---

# 29. What Does Not Belong in Item Aggregate

The Collection Item Aggregate should not contain:

- the entire Collection;
- the Collection Owner;
- other Collection Items;
- complete Item Relationships containing other Items;
- global configuration.

Instead, it should use identities or domain references.

---

# 30. Domain Operations Suggested by the Model

The Aggregate analysis also reveals likely domain operations.

For Collection:

```text id="a0k5yt"
Create Collection
Rename Collection
Change Owner
Change Configuration
Archive Collection
```

For Collection Item:

```text id="x5x9v8"
Create Item
Change Item Type
Set Attribute Value
Remove Attribute Value
Add Tag
Remove Tag
Change Status
Add External Reference
Remove External Reference
Archive Item
```

These are **domain-operation candidates**, not final APIs.

Their actual validity depends on the business rules still to be discovered.

---

# 31. Cross-Aggregate Operations

Some operations will require coordination across Aggregates.

For example:

```text id="5p0h2g"
Move Item
    Collection A
        ↓
    Collection B
```

This could involve:

```text id="cxw5ti"
Collection A
    +
Collection B
    +
Collection Item
```

The operation should not automatically force these concepts into a single Aggregate.

Instead, it may become a domain/application operation coordinating multiple Aggregates.

This is a critical distinction.

---

# 32. Transaction Boundary Implication

The current model suggests:

```text id="j4wzqh"
Transaction
    should normally target
one Aggregate
```

Cross-Aggregate transactions should be exceptional and justified by a genuine consistency requirement.

This is a design principle, not an absolute technical limitation.

---

# 33. Concurrency Analysis

The proposed model improves concurrency.

Example:

```text id="f65i4a"
User A
   updates Item 1
       │
       ▼
Item Aggregate 1


User B
   updates Item 2
       │
       ▼
Item Aggregate 2
```

Neither operation needs to coordinate through the entire Collection Aggregate.

This is particularly important if CollectionHub is expected to support large collections or collaborative use.

---

# 34. Scaling Analysis

The proposed boundaries allow the number of Items to grow independently of the Collection Aggregate.

Conceptually:

```text id="2sv5et"
Collection
   │
   ├── Item Aggregate 1
   ├── Item Aggregate 2
   ├── Item Aggregate 3
   ├── ...
   └── Item Aggregate N
```

The Collection Aggregate remains small even as the collection grows.

This is a strong architectural advantage.

---

# 35. Decision Matrix

| Boundary | Invariants | Size | Concurrency | Scalability | Decision |
|---|---|---|---|---|---|
| Collection + all Items | Strong | Very large | Poor | Poor | **Reject** |
| Collection + Configuration | Strong | Small | Good | Excellent | **Accept** |
| Item + Item data | Strong | Small | Excellent | Excellent | **Accept** |
| Item A + Item B | Weak | Large | Poor | Poor | **Reject** |
| Relationship + Items | Unclear | Potentially large | Poor | Poor | **Reject for now** |
| Independent Relationship | Unclear | Small | Good | Good | **TBD** |

---

# 36. Final Aggregate Decision

Based on the current domain evidence:

## Decision AGG-001

**Collection is an Aggregate Root.**

Its boundary includes:

- Collection;
- Collection ownership;
- Collection Configuration;
- collection-level invariants.

---

## Decision AGG-002

**Collection Item is an independent Aggregate Root.**

Its boundary includes:

- Collection Item;
- Item lifecycle;
- Attribute Values;
- Metadata;
- Item Tags;
- Item Status;
- External References.

---

## Decision AGG-003

**Collection does not contain all Collection Items inside its Aggregate boundary.**

Items are referenced conceptually by identity.

---

## Decision AGG-004

**Item-to-item relationships are not yet independent Aggregates.**

Their final classification requires further domain evidence.

---

# 37. Important Consequence

The following conceptual model is now rejected:

```text id="e9y7td"
Collection
   └── Items[]
```

as an Aggregate representation.

Instead, the domain model is:

```text id="i3v9c4"
Collection
   │
   └── CollectionId
          │
          ├── Item Aggregate
          ├── Item Aggregate
          ├── Item Aggregate
          └── ...
```

The Collection conceptually **contains** Items.

But the Collection Aggregate does not **own their consistency boundary**.

This distinction is fundamental.

---

# 38. Remaining Uncertainties

The Aggregate boundaries are now substantially clearer, but several domain questions remain.

### U-001

Whether Item Type definitions belong to Collection Configuration.

### U-002

Whether Attribute definitions belong to Collection Configuration.

### U-003

Whether Tags are Collection-scoped or global.

### U-004

Whether Source is a managed Entity.

### U-005

Whether Item Relationships require independent identity.

### U-006

Whether an Item can move between Collections.

### U-007

Whether Collection Configuration changes can invalidate existing Items.

### U-008

Whether Collections can be shared between Users.

These questions should be resolved before the final Domain Model is frozen.

---

# 39. Architectural Principle Emerging

The current modeling work leads to an important architectural principle for CollectionHub:

> **A Collection is an organizational boundary, but not necessarily a consistency boundary for every object it contains.**

This allows CollectionHub to model large collections without turning every operation into a Collection-wide transaction.

---

# 40. Current Aggregate Map

```text id="n0r2n5"
                    ┌─────────────────────────┐
                    │   COLLECTION AGGREGATE  │
                    │                         │
                    │   Collection             │
                    │      │                  │
                    │      ├── Owner          │
                    │      └── Configuration  │
                    │                         │
                    └────────────┬────────────┘
                                 │
                           CollectionId
                                 │
                 ┌───────────────┼───────────────┐
                 │               │               │
                 ▼               ▼               ▼
          ┌────────────┐  ┌────────────┐  ┌────────────┐
          │ ITEM       │  │ ITEM       │  │ ITEM       │
          │ AGGREGATE  │  │ AGGREGATE  │  │ AGGREGATE  │
          │            │  │            │  │            │
          │ Item       │  │ Item       │  │ Item       │
          │ Attributes │  │ Attributes │  │ Attributes │
          │ Metadata   │  │ Metadata   │  │ Metadata   │
          │ Tags       │  │ Tags       │  │ Tags       │
          │ Status     │  │ Status     │  │ Status     │
          │ References │  │ References │  │ References │
          └────────────┘  └────────────┘  └────────────┘

                         │
                         │
                         ▼
                 Relationship — TBD
```

---

# 41. Next Step

The next artifact should be:

```text id="m2c0xv"
04_DOMAIN_MODEL.md
```

At that point we can consolidate the work from:

```text id="qv8x5h"
Concept Inventory
       ↓
Concept Classification
       ↓
Identity
       ↓
Invariants
       ↓
Aggregate Boundaries
       ↓
Canonical Domain Model
```

`04_DOMAIN_MODEL.md` should become the first document that presents the **coherent domain model as a whole**, including:

- Entities;
- Value Objects;
- Aggregate Roots;
- internal Aggregate objects;
- domain relationships;
- domain rules;
- ownership;
- identity;
- lifecycle;
- unresolved decisions.

Only after that consolidated model is stable will it make sense to move toward domain behavior, commands, policies, and eventually implementation.