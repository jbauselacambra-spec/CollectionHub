# Domain Behavior and Operations

> **Document:** `05_DOMAIN_BEHAVIOR_AND_OPERATIONS.md`  
> **Phase:** 2.1 — Domain Modeling  
> **Status:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`, `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`, `03_AGGREGATE_BOUNDARY_ANALYSIS.md`, `04_DOMAIN_MODEL.md`  
> **Purpose:** Define the meaningful domain behavior of CollectionHub and identify the operations that can change domain state.

---

# 1. Purpose

The previous documents established the static structure of the CollectionHub domain.

We now move from:

```text
What exists?
```

to:

```text
What can happen?
```

This document identifies:

- domain operations;
- valid state changes;
- invalid state changes;
- operation preconditions;
- invariants protected by each operation;
- Aggregate ownership of behavior;
- cross-Aggregate operations;
- candidate commands;
- candidate Domain Events;
- unresolved behavioral rules.

This document intentionally remains at the **domain level**.

It does not define:

- HTTP endpoints;
- application services;
- controllers;
- database transactions;
- message brokers;
- ORM methods;
- DTOs.

---

# 2. Behavioral Modeling Principles

The domain model should be behavior-oriented rather than data-oriented.

We should avoid thinking only in terms of:

```text
Collection
    fields...
    
Item
    fields...
```

and instead reason in terms of:

```text
Collection
    can perform domain behavior

Item
    can perform domain behavior
```

The domain should protect its own rules.

---

# 3. Operation Categories

Domain operations are divided into four categories.

## 3.1 Creation

Operations that bring a new domain object into existence.

Examples:

```text
Create Collection
Create Collection Item
Create Relationship
```

---

## 3.2 State Mutation

Operations that change an existing domain object.

Examples:

```text
Rename Collection
Change Item Status
Set Attribute Value
Add Tag
```

---

## 3.3 Structural Mutation

Operations that modify the configuration or structure of the domain.

Examples:

```text
Change Collection Configuration
Add Attribute Definition
Remove Attribute Definition
```

---

## 3.4 Lifecycle Operations

Operations that move an Entity through its lifecycle.

Examples:

```text
Archive Collection
Archive Item
Restore Item
```

The exact lifecycle operations remain subject to business-rule validation.

---

# 4. Aggregate Behavior Principle

An operation should belong to the Aggregate that owns the invariant it modifies.

For example:

```text
Change Item Status
        ↓
Collection Item Aggregate
```

not:

```text
Change Item Status
        ↓
Collection Aggregate
```

Likewise:

```text
Change Collection Configuration
        ↓
Collection Aggregate
```

---

# 5. Collection Aggregate Operations

The Collection Aggregate currently owns the following behavior candidates.

```text
Create Collection
Rename Collection
Change Description
Change Owner
Change Configuration
Archive Collection
```

Each operation is analyzed below.

---

# 6. Create Collection

## Operation

```text
CreateCollection
```

## Purpose

Create a new Collection with valid initial state.

## Inputs

Conceptually:

```text
Owner
Name
Description?
Initial Configuration?
```

## Preconditions

- Owner must be valid.
- Name must satisfy collection naming rules.
- Initial configuration must be valid if provided.

## Postconditions

```text
Collection exists
Collection has stable identity
Collection has valid owner
Collection has valid initial state
```

## Invariants Protected

- INV-COL-001
- INV-COL-002

## Aggregate

```text
Collection
```

## Candidate Domain Event

```text
CollectionCreated
```

---

# 7. Rename Collection

## Operation

```text
RenameCollection
```

## Purpose

Change the human-readable name of a Collection.

## Preconditions

- Collection must exist.
- Collection must permit renaming.
- New name must satisfy naming rules.

## Postconditions

```text
Collection.Name = NewName
Collection.Identity unchanged
```

## Invariants

- ID-001
- INV-COL-001

## Important Rule

Renaming must never create a new Collection identity.

---

# 8. Change Collection Description

## Operation

```text
ChangeCollectionDescription
```

## Preconditions

- Collection must be mutable.
- New description must satisfy domain constraints.

## Postconditions

Only descriptive state changes.

Collection identity remains unchanged.

---

# 9. Change Collection Owner

## Operation

```text
ChangeCollectionOwner
```

## Purpose

Transfer ownership.

## Preconditions

Potentially:

- current actor has permission;
- target owner is valid;
- Collection is transferable.

Authorization is not itself an Aggregate invariant and should be modeled separately.

## Postconditions

```text
Owner = NewOwner
CollectionId unchanged
```

## Invariants

- INV-COL-002
- INV-OWN-002

## Candidate Event

```text
CollectionOwnershipChanged
```

---

# 10. Change Collection Configuration

This is one of the most important operations in the domain.

## Operation

```text
ChangeCollectionConfiguration
```

## Purpose

Modify the rules defining the structure of a Collection.

Potential changes include:

```text
Add Item Type
Remove Item Type
Add Attribute
Remove Attribute
Modify Attribute Definition
Change applicability rules
```

---

# 11. Configuration Change Problem

Configuration may affect existing Items.

For example:

```text
Configuration
    Attribute "Year"
    required for Type "Book"
```

Existing Item:

```text
Book
Year = 1987
```

Now:

```text
Remove Attribute "Year"
```

The domain must decide whether this is valid.

---

# 12. Configuration Compatibility Policies

Three broad policies are possible.

## Policy A — Strict

Reject configuration changes that invalidate existing Items.

```text
Configuration Change
        ↓
Compatibility Check
        ↓
Invalid
        ↓
Reject
```

---

## Policy B — Migrating

Accept the configuration change and explicitly migrate affected Items.

This would be a cross-Aggregate workflow.

---

## Policy C — Historical

Allow the configuration to evolve while preserving historical Item data.

This creates temporal semantics around configuration.

---

# 13. Current Decision

No policy is selected yet.

Therefore:

> `ChangeCollectionConfiguration` must remain a domain operation candidate with an unresolved compatibility policy.

This is intentionally left open rather than hidden inside implementation.

---

# 14. Archive Collection

## Operation

```text
ArchiveCollection
```

## Purpose

Move a Collection into an inactive lifecycle state.

## Preconditions

Potentially:

- Collection is currently active.

## Postconditions

```text
Collection.Status = Archived
```

## Critical Question

What happens to its Items?

Possible policies:

### A

Items remain active.

### B

Items are automatically archived.

### C

Items become inaccessible but retain their own state.

### D

Archiving is forbidden while active Items exist.

This must be decided before the operation becomes final.

---

# 15. Collection Item Operations

The Collection Item Aggregate owns Item-level behavior.

Candidate operations:

```text
Create Item
Change Item Type
Set Attribute Value
Remove Attribute Value
Set Metadata
Add Tag
Remove Tag
Add External Reference
Remove External Reference
Change Status
Archive Item
```

---

# 16. Create Collection Item

## Operation

```text
CreateCollectionItem
```

## Purpose

Create a valid Item inside a Collection context.

## Inputs

Conceptually:

```text
CollectionId
ItemType
Initial Attribute Values
Initial Metadata
Initial Tags
Initial External References
```

## Preconditions

- Collection must exist.
- Item Type must be valid.
- Initial attributes must comply with configuration.
- Required values must be present.

## Postconditions

```text
Item exists
Item has unique identity
Item references Collection
Item is internally valid
```

## Important Boundary Principle

The Item Aggregate does not load or contain the complete Collection Aggregate.

The operation may require an external validation step to establish that the Collection and configuration are valid.

---

# 17. Change Item Type

## Operation

```text
ChangeItemType
```

## Purpose

Change the classification of an Item.

## Preconditions

- New Item Type must be allowed.
- Existing Attribute Values must remain compatible.
- Required attributes for the new type must be satisfied.

## Example

```text
Old Type
    Book

New Type
    Magazine
```

Potential incompatibility:

```text
Book-specific Attribute
    ISBN
```

may no longer be valid.

---

# 18. Type Change Policy

If changing Item Type invalidates current state, possible policies include:

### Policy A

Reject the change.

### Policy B

Require migration before changing type.

### Policy C

Automatically remove incompatible values.

### Policy D

Allow transitional invalid state.

Current default:

> **Reject invalid state transitions unless the domain explicitly introduces migration semantics.**

---

# 19. Set Attribute Value

## Operation

```text
SetAttributeValue
```

## Inputs

```text
Attribute
Value
```

## Preconditions

- Attribute must be applicable to the Item.
- Value must satisfy Attribute validation.
- Item must permit modification.

## Postconditions

```text
Attribute Value
    added or replaced
```

## Invariants

- INV-ATTR-002
- INV-ATTR-003
- INV-ITEM-005

---

# 20. Remove Attribute Value

## Operation

```text
RemoveAttributeValue
```

## Preconditions

- Attribute exists on Item.
- Removal does not violate a required-attribute rule.

## Postconditions

Attribute value no longer exists.

---

# 21. Add Tag

## Operation

```text
AddTag
```

## Preconditions

- Tag must be valid in the current domain context.
- Duplicate semantics must be respected.

## Postconditions

Tag is associated with the Item.

---

# 22. Remove Tag

## Operation

```text
RemoveTag
```

## Preconditions

- Tag association exists.

## Postconditions

Tag is no longer associated with Item.

---

# 23. Set Metadata

## Operation

```text
SetMetadata
```

## Preconditions

- Metadata must satisfy domain constraints.

## Postconditions

Item metadata is updated.

The exact metadata model remains unresolved.

---

# 24. Add External Reference

## Operation

```text
AddExternalReference
```

## Preconditions

- Source information must be valid.
- External identifier must satisfy reference rules.
- Duplicate semantics must be respected.

## Postconditions

Item contains a valid External Reference.

## Invariants

- INV-EXT-001
- INV-EXT-002

---

# 25. Remove External Reference

## Operation

```text
RemoveExternalReference
```

## Preconditions

- Reference exists.

## Postconditions

Reference is removed.

The Item's internal identity remains unchanged.

---

# 26. Change Item Status

## Operation

```text
ChangeItemStatus
```

## Purpose

Move an Item between valid lifecycle states.

## Preconditions

Transition must be allowed by the Item lifecycle.

Conceptually:

```text
Current State
      │
      ▼
Transition Policy
      │
      ├── allowed → New State
      └── forbidden → reject
```

---

# 27. Status Transition Model

The final state machine is not yet established.

The current conceptual model is:

```text
          ┌─────────────┐
          │   ACTIVE    │
          └──────┬──────┘
                 │
              archive
                 │
                 ▼
          ┌─────────────┐
          │  ARCHIVED   │
          └─────────────┘
```

Potential future states may include:

```text
Draft
Active
Archived
Deleted
Invalid
Pending
```

These should only be added when supported by actual domain behavior.

---

# 28. Archive Item

## Operation

```text
ArchiveItem
```

## Preconditions

- Item must be in a state from which archival is valid.

## Postconditions

Item becomes archived.

Its identity remains unchanged.

---

# 29. Restore Item

This operation is currently a candidate.

```text
RestoreItem
```

It should only exist if the domain explicitly supports restoring archived Items.

Otherwise, the state machine should remain one-way.

---

# 30. Item Relationships

Relationship behavior requires independent analysis.

Potential operations include:

```text
CreateRelationship
RemoveRelationship
ChangeRelationshipType
AddRelationshipMetadata
```

But these operations cannot yet be finalized because the relationship model itself is unresolved.

---

# 31. Cross-Aggregate Operations

Some operations cannot naturally belong entirely to one Aggregate.

The current candidates are:

```text
Move Item Between Collections
Change Configuration Affecting Existing Items
Create Relationship Between Items
Transfer Collection Ownership
Archive Collection With Item Effects
```

These should be modeled as **domain-level use cases**, not automatically as Aggregate methods.

---

# 32. Move Item Between Collections

This is a particularly important scenario.

Conceptually:

```text
Item
  CollectionId = A

        move

Item
  CollectionId = B
```

The operation involves:

```text
Source Collection
Target Collection
Item
```

However, the need to coordinate them does not mean they belong in one Aggregate.

---

# 33. Move Item — Candidate Semantics

Potential process:

```text
1. Validate source Collection
2. Validate target Collection
3. Validate Item ownership
4. Validate target configuration
5. Change Item CollectionId
6. Persist resulting state
```

Whether this is one atomic transaction or a workflow is an implementation-level decision that depends on business requirements.

The domain decision that matters first is:

> **Is moving an Item between Collections a valid domain operation?**

Currently unresolved.

---

# 34. Domain Commands

The behavioral model naturally produces candidate Commands.

## Collection Commands

```text
CreateCollection
RenameCollection
ChangeCollectionDescription
ChangeCollectionOwner
ChangeCollectionConfiguration
ArchiveCollection
```

## Item Commands

```text
CreateCollectionItem
ChangeItemType
SetAttributeValue
RemoveAttributeValue
SetMetadata
AddTag
RemoveTag
AddExternalReference
RemoveExternalReference
ChangeItemStatus
ArchiveItem
```

These Commands represent requested domain actions.

They are not yet application-layer message contracts.

---

# 35. Command vs Domain Operation

A useful distinction:

```text
Command
    = request to perform an action

Domain Operation
    = behavior that changes domain state
```

For example:

```text
Command:
    ChangeItemStatus(ItemId, ARCHIVED)

        ↓

Domain Operation:
    Item.archive()
```

The exact software representation will be decided later.

---

# 36. Domain Events

Domain Events represent facts that have already happened.

Potential Events include:

### Collection

```text
CollectionCreated
CollectionRenamed
CollectionOwnershipChanged
CollectionConfigurationChanged
CollectionArchived
```

### Item

```text
CollectionItemCreated
ItemTypeChanged
ItemAttributeChanged
ItemTagAdded
ItemTagRemoved
ItemExternalReferenceAdded
ItemExternalReferenceRemoved
ItemStatusChanged
CollectionItemArchived
```

These are candidates, not yet a final event catalog.

---

# 37. Event Design Principle

An event should represent a meaningful domain fact.

Bad:

```text
DatabaseRowUpdated
```

Good:

```text
ItemStatusChanged
```

The first describes infrastructure.

The second describes domain behavior.

---

# 38. Event Ownership

Events should originate from the Aggregate whose state changed.

For example:

```text
Collection
    changes configuration
        ↓
CollectionConfigurationChanged
```

and:

```text
Collection Item
    changes status
        ↓
ItemStatusChanged
```

---

# 39. Invariant-to-Operation Mapping

| Operation | Main Invariants |
|---|---|
| Create Collection | ID-001, INV-COL-001, INV-COL-002 |
| Rename Collection | ID-001 |
| Change Owner | INV-COL-002, INV-OWN-002 |
| Change Configuration | INV-CONF-001, INV-CONF-002 |
| Create Item | ID-002, INV-ITEM-001, INV-ITEM-002 |
| Change Item Type | INV-ITEM-003, INV-ITEM-005 |
| Set Attribute | INV-ATTR-002, INV-ATTR-003 |
| Remove Attribute | INV-ATTR-003 |
| Add Tag | INV-TAG-001, INV-TAG-002 |
| Add External Reference | INV-EXT-001, INV-EXT-002 |
| Change Status | INV-ITEM-006, INV-ITEM-007 |
| Archive Item | INV-ITEM-007 |

This mapping provides traceability between static rules and dynamic behavior.

---

# 40. Invalid Operations

The domain must explicitly reject operations that would produce invalid state.

Examples:

```text
Rename Collection
    with invalid name
        → reject

Set Attribute Value
    with incompatible value
        → reject

Change Item Type
    creating incompatible attributes
        → reject

Change Status
    through forbidden transition
        → reject

Add External Reference
    without valid source/reference
        → reject
```

The domain should not silently accept invalid state merely because persistence permits it.

---

# 41. Behavioral Invariants

Several important principles emerge.

## BEHAV-001

An operation must leave its Aggregate in a valid state.

---

## BEHAV-002

Invalid state transitions must be rejected.

---

## BEHAV-003

Aggregate invariants should be enforced inside the Aggregate whenever possible.

---

## BEHAV-004

Cross-Aggregate coordination must not weaken local Aggregate invariants.

---

## BEHAV-005

Commands describe intent; Aggregates enforce domain rules.

---

## BEHAV-006

Domain Events describe facts, not requests.

---

# 42. Aggregate Method Candidates

The current behavioral model suggests the following conceptual methods.

## Collection

```text
rename()
changeDescription()
changeOwner()
changeConfiguration()
archive()
```

## Collection Item

```text
changeType()
setAttributeValue()
removeAttributeValue()
setMetadata()
addTag()
removeTag()
addExternalReference()
removeExternalReference()
changeStatus()
archive()
```

These are **conceptual behaviors**.

No programming language or method signature should be derived from this document yet.

---

# 43. Where Behavior Should Live

The following rule should guide future implementation.

If behavior primarily protects the state of an Aggregate:

```text
→ Aggregate
```

If behavior coordinates several Aggregates:

```text
→ Domain/Application orchestration
```

If behavior is a reusable domain policy without natural ownership:

```text
→ Domain Policy / Domain Service
```

If behavior is purely technical:

```text
→ Infrastructure
```

This prevents business logic from leaking into controllers, repositories, or persistence models.

---

# 44. Candidate Domain Policies

Several rules may eventually become explicit Policies.

Potential examples:

```text
CollectionNamePolicy
ItemStatusTransitionPolicy
AttributeCompatibilityPolicy
ItemTypeCompatibilityPolicy
ConfigurationCompatibilityPolicy
ExternalReferenceUniquenessPolicy
```

However, policies should only be introduced when the rules become sufficiently complex to justify them.

---

# 45. Behavioral Decision Log

## DEC-BEH-001

Item lifecycle behavior belongs to the Collection Item Aggregate.

## DEC-BEH-002

Collection configuration behavior belongs to the Collection Aggregate.

## DEC-BEH-003

Commands represent intent and are distinct from domain behavior.

## DEC-BEH-004

Domain Events represent completed domain facts.

## DEC-BEH-005

Cross-Aggregate operations do not automatically imply a larger Aggregate.

## DEC-BEH-006

Invalid domain state must be rejected rather than delegated to persistence constraints alone.

---

# 46. Open Behavioral Questions

The following questions now become the primary blockers for completing the behavioral model.

## Q-BEH-001

Can Collections be renamed after creation?

---

## Q-BEH-002

Can ownership be transferred?

---

## Q-BEH-003

Can archived Collections be restored?

---

## Q-BEH-004

Can Items move between Collections?

---

## Q-BEH-005

Can Item Types change after Item creation?

---

## Q-BEH-006

Can required Attributes be removed from Configuration?

---

## Q-BEH-007

What happens to existing Items when Configuration changes?

---

## Q-BEH-008

Can archived Items be restored?

---

## Q-BEH-009

Can Items have multiple Tags with equivalent semantic values?

---

## Q-BEH-010

Can two Items have multiple relationships of the same type?

---

## Q-BEH-011

Can relationships be deleted?

---

## Q-BEH-012

Are relationships directional?

---

# 47. Behavioral Model Summary

The current behavioral model can be summarized as:

```text
                    ┌─────────────────────────┐
                    │       COLLECTION        │
                    │                         │
                    │ create                  │
                    │ rename                  │
                    │ change owner            │
                    │ change configuration    │
                    │ archive                 │
                    └────────────┬────────────┘
                                 │
                                 │ CollectionId
                                 ▼
                    ┌─────────────────────────┐
                    │    COLLECTION ITEM       │
                    │                         │
                    │ create                  │
                    │ change type             │
                    │ set attributes          │
                    │ manage metadata         │
                    │ manage tags              │
                    │ manage references       │
                    │ change status            │
                    │ archive                 │
                    └────────────┬────────────┘
                                 │
                                 ▼
                         Relationships
                              TBD
```

---

# 48. Domain Behavior Statement

The current domain behavior can be summarized as:

> **Collections manage collection-level identity, ownership, configuration, and lifecycle. Collection Items independently manage their own identity, lifecycle, classification, attributes, metadata, tags, status, and external references. Domain operations must preserve the invariants of their owning Aggregate. Operations spanning multiple Aggregates are modeled as coordinated domain use cases rather than as justification for merging consistency boundaries.**

---

# 49. Next Step

The next artifact should be:

```text
06_DOMAIN_POLICIES_AND_RULES.md
```

This is the natural next step because we now know:

```text
WHAT exists
    ↓
WHO owns it
    ↓
WHAT must always be true
    ↓
WHAT can happen
```

Now we need to formalize **WHY an operation is allowed or forbidden**.

`06_DOMAIN_POLICIES_AND_RULES.md` will extract the business rules hidden inside the operations and classify them as:

- invariant;
- validation rule;
- policy;
- lifecycle rule;
- compatibility rule;
- authorization rule;
- cross-Aggregate rule;
- derived rule.

That separation will be important before moving toward Commands, Domain Events and Application Use Cases.