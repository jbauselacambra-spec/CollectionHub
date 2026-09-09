# Domain Policies and Rules

> **Document:** `06_DOMAIN_POLICIES_AND_RULES.md`  
> **Phase:** 2.1 — Domain Modeling  
> **Status:** Draft  
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`, `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`, `03_AGGREGATE_BOUNDARY_ANALYSIS.md`, `04_DOMAIN_MODEL.md`, `05_DOMAIN_BEHAVIOR_AND_OPERATIONS.md`  
> **Purpose:** Identify, classify, and formalize the business rules and policies governing CollectionHub domain behavior.

---

# 1. Purpose

The previous documents established:

```text
Concepts
    ↓
Identity
    ↓
Invariants
    ↓
Aggregates
    ↓
Domain Model
    ↓
Behavior
```

This document focuses on the next question:

> **Under what conditions is a domain operation allowed, forbidden, or conditionally valid?**

The purpose is to extract business rules from the previously identified behavior and classify them according to their nature.

This prevents business rules from becoming accidentally embedded in:

- controllers;
- repositories;
- database constraints;
- UI code;
- application services;
- infrastructure adapters.

---

# 2. Rule Taxonomy

CollectionHub rules are classified into the following categories.

## 2.1 Invariant

A condition that must always be true for a valid Aggregate state.

Example:

```text
An Item must always have a valid identity.
```

---

## 2.2 Validation Rule

A condition that determines whether an input is acceptable.

Example:

```text
A Collection name must satisfy naming constraints.
```

---

## 2.3 Domain Policy

A business decision that determines what the domain permits.

Example:

```text
An archived Item cannot be modified.
```

---

## 2.4 Lifecycle Rule

A rule controlling state transitions.

Example:

```text
ACTIVE → ARCHIVED
```

may be valid while:

```text
ARCHIVED → ACTIVE
```

may not be.

---

## 2.5 Compatibility Rule

A rule determining whether two concepts are mutually compatible.

Example:

```text
Item Type + Attribute
```

---

## 2.6 Authorization Rule

A rule determining whether an actor is permitted to perform an operation.

Example:

```text
Only the Collection Owner may change configuration.
```

Authorization is related to the domain but should not be confused with Aggregate invariants.

---

## 2.7 Cross-Aggregate Rule

A rule involving multiple Aggregates.

Example:

```text
An Item may only belong to an existing Collection.
```

---

## 2.8 Derived Rule

A fact that can be calculated from authoritative domain state rather than explicitly stored.

Example:

```text
Collection Item Count
```

may be derived from Items rather than maintained as mutable Aggregate state.

---

# 3. Rule Ownership Principle

Each rule should have an explicit owner.

The general preference is:

```text
Rule about Aggregate state
        ↓
Aggregate
```

```text
Rule about multiple Aggregates
        ↓
Domain/Application coordination
```

```text
Rule about permissions
        ↓
Authorization policy
```

```text
Rule about external systems
        ↓
Integration boundary
```

This avoids creating a generic "BusinessRules" component containing unrelated logic.

---

# 4. Collection Naming Rules

Collection names are user-visible domain data and therefore require explicit rules.

## RULE-COL-NAME-001

A Collection must have a non-empty name.

### Type

Validation Rule.

### Owner

Collection.

---

## RULE-COL-NAME-002

The Collection name must satisfy the domain's maximum length.

### Type

Validation Rule.

### Status

**TBD**

The actual limit must be defined from requirements rather than invented at this stage.

---

## RULE-COL-NAME-003

Whitespace normalization rules must be explicit.

Questions include:

```text
"Books"
" Books "
"Books  "
```

Should these be considered equivalent?

### Status

**Open**

---

## RULE-COL-NAME-004

Case sensitivity must be explicitly defined.

For example:

```text
"Books"
"books"
```

may or may not represent the same name.

### Status

**Open**

---

# 5. Collection Ownership Rules

## RULE-OWN-001

Every valid Collection must have an owner.

### Type

Invariant.

### Owner

Collection.

---

## RULE-OWN-002

A Collection Owner must reference a valid domain actor.

### Type

Invariant.

---

## RULE-OWN-003

Ownership transfer must not change Collection identity.

### Type

Invariant.

---

## RULE-OWN-004

The domain must define whether multiple owners are supported.

### Status

**Open**

---

## RULE-OWN-005

The domain must distinguish ownership from permission.

For example:

```text
Owner
    ≠
Editor
    ≠
Viewer
```

if collaboration is eventually supported.

---

# 6. Collection Lifecycle Rules

The current conceptual lifecycle is:

```text
ACTIVE
   │
   ▼
ARCHIVED
```

## RULE-COL-LIFE-001

A Collection must begin in a valid initial lifecycle state.

---

## RULE-COL-LIFE-002

Only valid lifecycle transitions may occur.

---

## RULE-COL-LIFE-003

The domain must explicitly define whether archived Collections can be modified.

### Current recommendation

Prefer:

```text
Archived Collection
    → immutable except for lifecycle operations
```

unless requirements indicate otherwise.

This remains a provisional policy.

---

## RULE-COL-LIFE-004

The domain must define whether an archived Collection can be restored.

### Status

Open.

---

# 7. Collection Configuration Rules

Configuration is one of the most significant policy areas.

## RULE-CONF-001

Collection Configuration must always be internally valid.

---

## RULE-CONF-002

A Collection may only use valid Item Type definitions.

---

## RULE-CONF-003

An Attribute Definition must have a valid semantic definition.

---

## RULE-CONF-004

An Attribute may only be applicable to Item Types for which it is valid.

---

## RULE-CONF-005

Required Attributes must define what "required" means.

Possible interpretations:

```text
Required at creation
Required while active
Required permanently
Required only for specific Item Types
```

This must be explicitly decided.

---

# 8. Configuration Evolution Policy

This is currently the highest-risk policy area.

Suppose:

```text
Item Type = Book
Attribute = ISBN
```

and ISBN is required.

An existing Item contains:

```text
ISBN = 978...
```

Now the configuration removes ISBN.

The system must decide what this means.

---

# 9. Configuration Change Policies

## Policy CONF-A — Reject

Configuration cannot be changed if existing Items would become invalid.

Advantages:

- strong consistency;
- simple mental model;
- no migration state.

Disadvantages:

- configuration becomes difficult to evolve.

---

## Policy CONF-B — Migrate

Configuration changes initiate an explicit migration.

Advantages:

- flexible;
- explicit data evolution.

Disadvantages:

- introduces cross-Aggregate workflow;
- potentially expensive.

---

## Policy CONF-C — Historical Configuration

Existing Items retain compatibility with the configuration version under which they were created.

Conceptually:

```text
Configuration v1
    ↓
Item A created

Configuration v2
    ↓
Item B created
```

Advantages:

- historical consistency;
- flexible evolution.

Disadvantages:

- significantly more complex domain model.

---

# 10. Current Configuration Decision

No final policy is selected.

However:

> **The domain must not silently invalidate existing Items as a side effect of a Configuration change.**

At minimum, the operation must either:

- reject the change;
- explicitly migrate affected Items;
- or introduce a historical/versioned configuration model.

---

# 11. Item Type Rules

## RULE-TYPE-001

Every Item must have a valid Item Type if Item Type is mandatory.

---

## RULE-TYPE-002

An Item Type must be valid in the Item's Collection context.

---

## RULE-TYPE-003

An Item Type change must not silently create invalid Item state.

---

## RULE-TYPE-004

The domain must define whether an Item Type can change after creation.

### Status

Open.

---

# 12. Attribute Rules

## RULE-ATTR-001

An Attribute Value must correspond to a valid Attribute Definition.

---

## RULE-ATTR-002

An Attribute Value must satisfy the Attribute's value constraints.

---

## RULE-ATTR-003

An Item may only contain Attributes applicable to its Item Type/configuration.

---

## RULE-ATTR-004

Required Attributes must be present when their requirement applies.

---

## RULE-ATTR-005

Removing an Attribute Value must not violate a required-attribute rule.

---

# 13. Attribute Value Validation

Attribute validation should be considered domain behavior.

Possible validation dimensions include:

```text
Data Type
Format
Range
Length
Allowed Values
Cardinality
Requiredness
Uniqueness
```

For example:

```text
Attribute:
    Year

Value:
    1987

Validation:
    integer
    range >= minimum
    range <= maximum
```

The exact validation vocabulary remains to be defined.

---

# 14. Attribute Cardinality

The domain must determine whether an Attribute supports:

```text
single value
```

or:

```text
multiple values
```

For example:

```text
Genre
    Fantasy
    Science Fiction
```

could be represented as multiple values.

This is an important domain decision because it affects the shape of Attribute Values.

### Status

Open.

---

# 15. Tag Rules

## RULE-TAG-001

A Tag associated with an Item must be valid within the applicable Tag context.

---

## RULE-TAG-002

Duplicate Tag semantics must be explicitly defined.

Potentially:

```text
"Rare"
"rare"
"RARE"
```

could represent one semantic Tag.

---

## RULE-TAG-003

Adding a Tag that is already semantically associated with an Item should be idempotent or rejected according to domain policy.

### Recommendation

Prefer idempotent behavior where the domain treats Tags as sets.

---

# 16. Metadata Rules

Metadata must not become an uncontrolled escape hatch for domain data.

## RULE-META-001

Metadata must have a defined semantic boundary.

---

## RULE-META-002

Information that participates in domain invariants should not be hidden inside opaque Metadata.

For example:

```text
Bad:
metadata["status"] = "archived"
```

if Status is a real domain concept.

Instead:

```text
Item.Status
```

should remain explicit.

---

# 17. External Reference Rules

## RULE-EXT-001

An External Reference must identify its source.

---

## RULE-EXT-002

An External Reference must contain a valid external identifier or reference.

---

## RULE-EXT-003

External Reference identity must not replace Item identity.

---

## RULE-EXT-004

An Item should remain a valid domain object even if an external source becomes unavailable.

---

## RULE-EXT-005

Duplicate external references must have explicitly defined semantics.

Possible policy:

```text
same source + same external identifier
    → duplicate
```

This is a strong candidate for a uniqueness policy.

---

# 18. Item Lifecycle Rules

Current conceptual lifecycle:

```text
ACTIVE
   │
   ▼
ARCHIVED
```

## RULE-ITEM-LIFE-001

An Item must have a valid lifecycle state.

---

## RULE-ITEM-LIFE-002

Only valid lifecycle transitions may occur.

---

## RULE-ITEM-LIFE-003

An archived Item must not silently behave as an active Item.

---

## RULE-ITEM-LIFE-004

The domain must define whether archived Items can be restored.

---

## RULE-ITEM-LIFE-005

The domain must define which modifications remain possible after archival.

---

# 19. Item Modification Policy

A possible policy is:

```text
ACTIVE
    → normal modifications

ARCHIVED
    → read-only
```

This is a useful default because it gives archival semantic meaning.

However:

> This is a **provisional policy**, not yet a final business decision.

---

# 20. Relationship Rules

Item Relationships remain under active modeling.

Potential rules include:

## RULE-REL-001

Both relationship endpoints must reference valid Items.

---

## RULE-REL-002

The relationship type must be valid.

---

## RULE-REL-003

Self-references should be prohibited unless explicitly allowed.

---

## RULE-REL-004

Relationship direction must be defined per relationship type.

---

## RULE-REL-005

Duplicate relationships must have explicit semantics.

---

## RULE-REL-006

The domain must define whether multiple relationship types may exist between the same pair of Items.

---

# 21. Relationship Symmetry

Relationship types may be:

### Directed

```text
A ── PART_OF ──> B
```

### Symmetric

```text
A ── RELATED_TO ── B
```

### Inverse

```text
A ── PART_OF ──> B

B ── CONTAINS ──> A
```

The model must distinguish these semantics rather than assuming every relationship is simply an undirected link.

---

# 22. Collection Membership Rules

## RULE-MEM-001

Every Item must reference a valid Collection.

---

## RULE-MEM-002

An Item must belong to exactly one Collection unless the domain explicitly introduces multi-collection membership.

---

## RULE-MEM-003

If an Item may move between Collections, the move must be an explicit domain operation.

---

## RULE-MEM-004

Changing Collection membership must validate the target Collection context.

---

# 23. Multi-Collection Membership

Two possible models exist.

## Model A — Single Membership

```text
Item
  └── CollectionId
```

An Item belongs to exactly one Collection.

---

## Model B — Multiple Membership

```text
Item
  ├── Collection A
  ├── Collection B
  └── Collection C
```

This would fundamentally change the domain model.

Current assumption:

> **An Item belongs to exactly one Collection.**

This remains a requirement to validate rather than a permanently frozen decision.

---

# 24. Move Item Policy

If Item movement is supported:

```text
Collection A
     │
     │
     ▼
   Item
     │
     │
     ▼
Collection B
```

The operation must validate:

1. source membership;
2. target Collection existence;
3. target configuration compatibility;
4. target Item Type compatibility;
5. resulting Item validity.

The move must never leave the Item in an invalid Collection context.

---

# 25. Authorization Rules

Authorization is deliberately separated from domain invariants.

Example:

```text
User
   └── wants to change configuration
```

The question:

> Is the configuration valid?

is a domain rule.

The question:

> Is this User allowed to change it?

is an authorization rule.

These must not be conflated.

---

# 26. Candidate Authorization Policies

Potential roles:

```text
Owner
Editor
Viewer
```

Potential permissions:

```text
Manage Collection
Manage Configuration
Create Item
Edit Item
Archive Item
Manage Relationships
```

The current model does not yet establish these roles.

---

# 27. Authorization Decision

The only current domain-level decision is:

> Collection ownership represents an ownership relationship, but ownership must not automatically be treated as the complete authorization model.

This leaves room for collaboration and delegated permissions.

---

# 28. Derived Rules

Some information should preferably be derived.

Potential examples:

```text
Collection Item Count
Number of Active Items
Number of Archived Items
```

These should not automatically become mutable Collection state.

For example:

```text
Collection.ItemCount++
```

should not be considered a domain invariant unless there is a compelling reason to make the count authoritative.

---

# 29. Derived State Principle

If information can be calculated reliably from authoritative state:

> Prefer deriving it over introducing a second mutable source of truth.

This reduces consistency problems.

---

# 30. Rule Priority

Not all rules are equally authoritative.

We classify them as:

### Mandatory

Explicitly required by the domain.

### Provisional

Strong modeling recommendation awaiting confirmation.

### Open

Requires a business decision.

The project must not silently convert provisional rules into mandatory ones.

---

# 31. Mandatory Rules

Current mandatory rules include:

```text
RULE-OWN-001
Every Collection has an owner.

RULE-ATTR-001
Attribute Values correspond to valid Attribute definitions.

RULE-ATTR-002
Attribute Values satisfy their validation rules.

RULE-EXT-001
External References identify a source.

RULE-EXT-003
External references do not define Item identity.

RULE-ITEM-LIFE-001
Items have valid lifecycle state.

RULE-MEM-001
Items reference a valid Collection.
```

---

# 32. Provisional Rules

Current provisional rules include:

```text
RULE-COL-LIFE-003
Archived Collections are effectively immutable.

RULE-ITEM-LIFE-003
Archived Items are not treated as active.

RULE-MEM-002
An Item belongs to exactly one Collection.

RULE-ATTR-005
Required Attribute Values cannot be removed.

RULE-TAG-003
Adding an existing Tag is idempotent.

RULE-REL-003
Self-references are prohibited.
```

These should be validated before implementation.

---

# 33. Open Rules

Important unresolved rules include:

```text
Collection naming normalization
Collection name uniqueness
Ownership multiplicity
Collection restoration
Item restoration
Item Type mutability
Attribute cardinality
Attribute scope
Tag scope
Source identity
Relationship identity
Relationship direction
Relationship duplication
Configuration evolution
Item movement
Multi-collection membership
Authorization roles
```

---

# 34. Policy Matrix

| Domain Area | Rule | Classification | Status |
|---|---|---|---|
| Collection | Identity stability | Invariant | Confirmed |
| Collection | Must have owner | Invariant | Confirmed |
| Collection | Name validation | Validation | Partial |
| Collection | Name uniqueness | Policy | Open |
| Collection | Archive behavior | Lifecycle Policy | Provisional |
| Ownership | Transfer preserves identity | Invariant | Confirmed |
| Configuration | Must remain valid | Invariant | Confirmed |
| Configuration | Evolution compatibility | Policy | Open |
| Item | Identity stability | Invariant | Confirmed |
| Item | Valid Collection | Cross-Aggregate Rule | Confirmed |
| Item | Item Type validity | Compatibility | Confirmed |
| Attribute | Value validation | Validation | Confirmed |
| Attribute | Requiredness | Policy | Partial |
| Tag | Duplicate semantics | Policy | Open |
| Metadata | Explicit domain data | Policy | Confirmed |
| External Reference | Valid source | Invariant | Confirmed |
| External Reference | Duplicate semantics | Policy | Open |
| Item Lifecycle | Valid transitions | Lifecycle | Confirmed |
| Item Lifecycle | Restore | Lifecycle | Open |
| Relationship | Valid endpoints | Invariant | Confirmed |
| Relationship | Self-reference | Policy | Provisional |
| Authorization | Owner permissions | Authorization | Open |

---

# 35. Rule Traceability

Every significant rule should eventually trace back to:

```text
Business Requirement
        ↓
Domain Rule
        ↓
Invariant / Policy
        ↓
Domain Behavior
        ↓
Aggregate
        ↓
Implementation
```

For example:

```text
Requirement
"An Item cannot have an invalid Year"

        ↓

Rule
Attribute value must satisfy definition

        ↓

Invariant
INV-ATTR-002

        ↓

Operation
SetAttributeValue

        ↓

Aggregate
Collection Item

        ↓

Implementation
Domain validation
```

This traceability will be valuable when implementation begins.

---

# 36. Anti-Patterns to Avoid

## Anti-Pattern 1 — Validation only at API level

```text
Controller
    validates everything
        ↓
Domain
    trusts input
```

This makes the domain fragile.

---

## Anti-Pattern 2 — Database as business-rule engine

```text
Domain
    accepts invalid state

Database
    rejects it
```

Persistence constraints are useful but should not replace domain rules.

---

## Anti-Pattern 3 — Generic Rules Service

```text
BusinessRulesService
    150 methods
```

This usually indicates poor rule ownership.

---

## Anti-Pattern 4 — Authorization mixed into Entity invariants

The Entity should know whether a state transition is valid.

It should not necessarily know the entire application's permission model.

---

# 37. Policy Ownership Matrix

| Policy | Candidate Owner |
|---|---|
| Collection Name Policy | Collection |
| Collection Lifecycle Policy | Collection |
| Configuration Compatibility Policy | Collection / Domain Policy |
| Item Type Compatibility Policy | Item / Configuration |
| Attribute Validation Policy | Attribute Definition |
| Item Lifecycle Policy | Collection Item |
| Tag Normalization Policy | Tag model |
| External Reference Policy | Collection Item / External Reference |
| Relationship Policy | Relationship model |
| Authorization Policy | Authorization layer |

---

# 38. Domain Rule Design Principle

A useful rule for future implementation:

> **Put a rule where the information required to decide it naturally lives.**

If an Item can decide:

```text
"Can I transition from ACTIVE to ARCHIVED?"
```

the rule belongs with the Item.

If the decision requires:

```text
Item
+
Collection Configuration
+
other Aggregates
```

the rule may require coordination outside the Item Aggregate.

---

# 39. Current Policy Architecture

The emerging policy architecture is:

```text
                 DOMAIN RULES
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   Aggregate      Policies      Cross-Aggregate
     Rules                        Rules
        │             │             │
        ▼             ▼             ▼
 Collection        Compatibility   Workflows
 Item              Lifecycle       Coordination
```

This is intentionally conceptual.

No technical framework is implied.

---

# 40. Final Decisions

## DEC-POL-001

Domain invariants must be protected independently of external interfaces.

---

## DEC-POL-002

Validation rules with domain meaning belong to the domain model.

---

## DEC-POL-003

Authorization rules are distinct from domain invariants.

---

## DEC-POL-004

Configuration changes must not silently invalidate existing Items.

---

## DEC-POL-005

External references never replace internal Item identity.

---

## DEC-POL-006

Derived information should not become mutable authoritative state without justification.

---

## DEC-POL-007

Provisional rules must remain explicitly marked until requirements confirm them.

---

# 41. Current Domain Rule Statement

The current rule model can be summarized as:

> **CollectionHub must preserve valid Collection and Collection Item state at all times. Collection-level policies govern ownership, configuration, and lifecycle, while Item-level policies govern classification, attributes, metadata, tags, references, and lifecycle. Compatibility rules connect Collection configuration with Item validity without automatically merging their Aggregate boundaries. Authorization is treated separately from domain validity.**

---

# 42. What We Have Achieved

At this point, the domain model has moved significantly beyond a list of entities.

We now have:

```text
Concepts
    ✓

Classification
    ✓

Identity
    ✓

Invariants
    ✓

Aggregate Boundaries
    ✓

Canonical Domain Model
    ✓

Domain Behavior
    ✓

Policies and Rules
    ✓
```

This means the model now has both:

```text
STRUCTURE
```

and:

```text
BEHAVIOR
```

---

# 43. Next Step

The next artifact should be:

```text
07_DOMAIN_COMMANDS_AND_EVENTS.md
```

The reason to do this **now** is that Commands and Events should emerge from the domain behavior we have just modeled.

We will establish:

```text
User Intent
     ↓
Command
     ↓
Domain Operation
     ↓
Aggregate
     ↓
State Change
     ↓
Domain Event
```

For each operation we will define:

- command intent;
- command inputs;
- target Aggregate;
- preconditions;
- resulting state change;
- emitted Domain Event;
- event meaning;
- whether the operation is synchronous or potentially workflow-based;
- whether it crosses Aggregate boundaries.

Only after that will we have a sufficiently mature behavioral model to move into **Application Use Cases and Application Services**.

The important discipline remains the same:

> **We are still modeling the domain. We are not designing the API or writing code yet.**