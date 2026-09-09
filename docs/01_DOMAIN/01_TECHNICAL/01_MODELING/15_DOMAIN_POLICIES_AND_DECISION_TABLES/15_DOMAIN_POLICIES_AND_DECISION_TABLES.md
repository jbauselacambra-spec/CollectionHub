# CollectionHub — Domain Policies and Decision Tables

> **Phase:** 2.2 — Domain Modeling  
> **Artifact:** 15 — Domain Policies and Decision Tables  
> **Status:** Draft / Design Baseline  
> **Scope:** Domain Layer  
> **Depends on:** Domain Concept Inventory, Domain Glossary, Domain Invariants, Domain Aggregates, Domain Use Cases, Application Use Cases and Workflows

---

## 1. Purpose

This document defines the **domain policies** and **decision tables** that govern the behavior of CollectionHub.

The objective is to make explicit the rules that determine:

- whether a domain operation is allowed;
- which state transitions are valid;
- how conflicting conditions are resolved;
- which business rules have priority;
- which decisions belong to the domain;
- which decisions must remain outside the domain;
- and which rules must be deterministic and testable.

The policies defined here are intended to complement, not replace:

- domain invariants;
- aggregate consistency boundaries;
- domain entities and value objects;
- domain services;
- application workflows.

A domain invariant describes something that **must always be true**.

A domain policy describes **how the domain decides what should happen when certain conditions are met**.

A decision table makes that policy explicit.

---

# 2. Policy Classification

CollectionHub policies are classified according to their responsibility.

| Policy type | Responsibility | Typical implementation |
|---|---|---|
| Validation Policy | Determines whether an operation or state is valid | Aggregate / Value Object |
| Eligibility Policy | Determines whether something qualifies for an operation | Domain Service / Policy |
| Transition Policy | Determines whether a state transition is allowed | Aggregate |
| Ownership Policy | Determines ownership and authority | Aggregate / Domain Policy |
| Uniqueness Policy | Determines whether two concepts may coexist | Domain Service / Repository-backed policy |
| Collection Policy | Governs membership of items in collections | Aggregate / Domain Service |
| Lifecycle Policy | Governs creation, activation, archival or removal | Aggregate |
| Conflict Resolution Policy | Determines what happens when multiple rules apply | Domain Policy |
| Ordering / Ranking Policy | Determines domain-level ordering when applicable | Domain Policy |
| Classification Policy | Determines categorization or classification | Domain Service |
| Synchronization Policy | Governs reconciliation with external information | Domain Service / Application Layer |

Not every policy belongs inside an aggregate.

A policy belongs to the domain when its decision depends on **business meaning**, rather than technical infrastructure.

---

# 3. Core Principles

## 3.1 Policies must be deterministic

For the same domain state and the same decision inputs, a policy must produce the same result.

A policy must not depend implicitly on:

- current system time;
- random values;
- network availability;
- database implementation details;
- UI state;
- HTTP semantics;
- framework behavior.

When time is relevant, it must be provided explicitly as a domain input.

---

## 3.2 Policies must operate on domain concepts

Policies should be expressed using concepts already defined by the domain model.

Avoid rules such as:

```text
if databaseRecord.status == 3
```

Prefer:

```text
if collection.isArchived()
```

The policy must communicate **business intent**, not persistence representation.

---

## 3.3 Invariants have priority over policies

A policy can recommend or determine an action, but it cannot violate an invariant.

Conceptually:

```text
Policy Decision
      ↓
Invariant Validation
      ↓
State Change
```

A policy returning `ALLOW` does not guarantee that an operation can be committed if another invariant is violated.

---

## 3.4 Policies should be composable

Complex decisions should preferably be decomposed into smaller policies.

For example:

```text
CanAddItemToCollection
        ↓
    Ownership
        ↓
    Collection State
        ↓
    Item Eligibility
        ↓
    Duplication
        ↓
    Capacity
```

This is preferable to a single monolithic rule.

---

# 4. Policy Result Model

Domain policies should communicate decisions explicitly.

The conceptual result of a policy may be:

```text
ALLOW
DENY
REJECT
DEFER
REQUIRE_REVIEW
```

The exact implementation may differ, but the semantic distinction is important.

### ALLOW

The domain conditions permit the requested operation.

### DENY

The operation is understood but the current domain state does not permit it.

### REJECT

The requested operation is invalid from a domain perspective.

### DEFER

The domain cannot complete the decision yet because a prerequisite decision or process must occur.

### REQUIRE_REVIEW

The domain recognizes a legitimate but exceptional condition requiring an explicit decision.

Not every policy needs every result.

---

# 5. Policy P-001 — Collection Ownership

## Intent

Only an actor with the appropriate ownership authority may perform ownership-sensitive operations on a collection.

## Inputs

- Collection
- Actor
- Requested operation

## Decision

The actor must satisfy the collection's ownership/authorization rules.

## Rules

| Condition | Decision |
|---|---|
| Actor owns collection | ALLOW |
| Actor does not own collection | DENY |
| Collection does not exist | REJECT / handled by application layer |
| Collection is inaccessible | DENY |

## Notes

Authentication is not a domain policy.

The domain should receive an actor identity or domain authorization context rather than knowing how authentication was performed.

---

# 6. Policy P-002 — Collection Mutability

## Intent

Determine whether a collection can currently be modified.

## Decision Table

| Collection state | Operation | Decision |
|---|---|---|
| Active | Add item | ALLOW |
| Active | Remove item | ALLOW |
| Active | Reorder item | ALLOW |
| Active | Update metadata | ALLOW |
| Archived | Add item | DENY |
| Archived | Remove item | DENY |
| Archived | Reorder item | DENY |
| Archived | Update metadata | DENY |
| Deleted | Any mutation | REJECT |

The exact state names must remain aligned with the lifecycle model defined in the domain state machine.

---

# 7. Policy P-003 — Item Eligibility for Collection Membership

## Intent

Determine whether an item may become a member of a collection.

## Decision Table

| Item condition | Collection condition | Decision |
|---|---|---|
| Valid item | Active collection | ALLOW |
| Invalid item | Active collection | DENY |
| Valid item | Archived collection | DENY |
| Invalid item | Archived collection | DENY |
| Already member | Active collection | DENY / idempotent according to command semantics |
| Valid item | Deleted collection | REJECT |

This policy must not determine persistence behavior.

---

# 8. Policy P-004 — Duplicate Membership

## Intent

Prevent a collection from containing the same logical item more than once when the collection semantics require uniqueness.

The definition of "same item" must be based on the domain identity established by the model.

It must not be based accidentally on:

- database row identity;
- object reference;
- serialized representation;
- external provider identifier unless explicitly defined as domain identity.

## Decision Table

| Item membership exists | Collection mutable | Decision |
|---|---|---|
| No | Yes | ALLOW |
| No | No | DENY |
| Yes | Yes | DENY / IDEMPOTENT depending on use case |
| Yes | No | DENY |

### Important distinction

A command such as:

```text
AddItem
```

may be intentionally idempotent.

In that case:

```text
AlreadyMember → no state change
```

rather than:

```text
AlreadyMember → domain error
```

This decision must be made at the use-case level and documented explicitly.

---

# 9. Policy P-005 — Collection Capacity

## Intent

Determine whether a collection can accept another member when a capacity constraint exists.

## Decision Table

| Capacity configured | Current size | Capacity reached | Decision |
|---|---:|---|---|
| No | Any | No | ALLOW |
| Yes | Below capacity | No | ALLOW |
| Yes | Equal to capacity | Yes | DENY |
| Yes | Above capacity | Yes | DENY / invariant violation |

The domain must never allow a state in which:

```text
currentSize > maximumCapacity
```

if the capacity invariant exists.

---

# 10. Policy P-006 — Collection Lifecycle Transition

## Intent

Control valid transitions between collection lifecycle states.

## Decision Table

| Current state | Requested state | Decision |
|---|---|---|
| Draft | Active | ALLOW |
| Draft | Archived | ALLOW only if explicitly supported |
| Active | Archived | ALLOW |
| Active | Draft | DENY |
| Archived | Active | ALLOW only if reactivation is supported |
| Archived | Draft | DENY |
| Deleted | Any | DENY |

The definitive lifecycle must be represented by the domain state machine.

This table must not be duplicated independently in multiple application services.

---

# 11. Policy P-007 — Removal Eligibility

## Intent

Determine whether a collection member may be removed.

## Decision Table

| Collection state | Membership exists | Actor authorized | Decision |
|---|---|---|---|
| Active | Yes | Yes | ALLOW |
| Active | Yes | No | DENY |
| Active | No | Yes | DENY / IDEMPOTENT depending on command |
| Archived | Yes | Yes | DENY |
| Deleted | Any | Any | REJECT |

---

# 12. Policy P-008 — Reordering Eligibility

## Intent

Determine whether an item's position inside a collection can change.

## Decision Table

| Collection state | Membership exists | Ordering enabled | Decision |
|---|---|---|---|
| Active | Yes | Yes | ALLOW |
| Active | Yes | No | DENY |
| Active | No | Any | DENY |
| Archived | Any | Any | DENY |
| Deleted | Any | Any | REJECT |

The ordering mechanism itself is an implementation concern unless ordering has explicit business semantics.

---

# 13. Policy P-009 — Metadata Update Eligibility

## Intent

Determine whether collection metadata can be changed.

Metadata may include concepts such as:

- name;
- description;
- classification;
- visibility;
- presentation metadata;
- other attributes explicitly defined by the domain.

## Decision Table

| Collection state | Actor authorized | Decision |
|---|---|---|
| Active | Yes | ALLOW |
| Active | No | DENY |
| Archived | Yes | DENY unless specific metadata is explicitly mutable |
| Archived | No | DENY |
| Deleted | Any | REJECT |

The policy should avoid assuming that all metadata shares the same lifecycle.

---

# 14. Policy P-010 — Visibility / Access Policy

If CollectionHub supports collection visibility, access must be evaluated according to the collection's visibility policy.

Possible conceptual states:

```text
PRIVATE
SHARED
PUBLIC
```

## Decision Table

| Visibility | Actor relationship | Read access |
|---|---|---|
| PRIVATE | Owner | ALLOW |
| PRIVATE | Authorized collaborator | ALLOW if collaboration is supported |
| PRIVATE | Other actor | DENY |
| SHARED | Authorized actor | ALLOW |
| SHARED | Unauthorized actor | DENY |
| PUBLIC | Any actor | ALLOW |

Write access remains governed by ownership/authorization policies and must not be inferred from read access.

---

# 15. Policy P-011 — Collection Deletion

## Intent

Determine whether a collection can transition to a deleted state.

## Decision Table

| Collection state | Actor authorized | Decision |
|---|---|---|
| Draft | Yes | ALLOW |
| Active | Yes | ALLOW |
| Archived | Yes | ALLOW |
| Any | No | DENY |
| Deleted | Any | IDEMPOTENT / NO-OP |

Deletion semantics must distinguish between:

```text
logical deletion
```

and:

```text
physical persistence deletion
```

The former is domain behavior.

The latter is an infrastructure concern unless explicitly modeled otherwise.

---

# 16. Policy P-012 — Item Classification

If CollectionHub supports classification, an item may be assigned to one or more domain classifications according to the classification rules.

## Decision Table

| Item valid | Classification valid | Classification conflict | Decision |
|---|---|---|---|
| Yes | Yes | No | ALLOW |
| Yes | No | N/A | DENY |
| Yes | Yes | Yes | REQUIRE_REVIEW / resolution policy |
| No | Any | Any | DENY |

Classification must not silently mutate unrelated aggregates.

---

# 17. Policy P-013 — External Identity Resolution

When an item originates from an external source, CollectionHub may need to determine whether an existing domain item corresponds to the external identity.

## Decision Table

| External identity | Existing mapping | Decision |
|---|---|---|
| Valid | Exists | Resolve existing item |
| Valid | Does not exist | Create/import candidate |
| Invalid | Any | Reject |
| Conflicting mapping | Exists | REQUIRE_REVIEW / reject |

External identifiers should not automatically become domain identity unless explicitly defined by the model.

---

# 18. Policy P-014 — Synchronization Conflict Resolution

If CollectionHub synchronizes domain information with external sources, conflicting values must be resolved through an explicit policy.

## Decision Table

| Local state | External state | Local modification | External modification | Decision |
|---|---|---|---|---|
| Same | Same | No | No | NO-OP |
| Different | Different | No | Yes | Accept external |
| Different | Different | Yes | No | Preserve local |
| Different | Different | Yes | Yes | CONFLICT |
| Local missing | External exists | No | Yes | Import candidate |
| Local exists | External missing | Yes/No | Yes | Apply disappearance policy |

The exact conflict strategy must be domain-specific.

Possible strategies:

```text
LOCAL_WINS
EXTERNAL_WINS
LATEST_WINS
MERGE
MANUAL_REVIEW
REJECT
```

`LATEST_WINS` must only be used when the domain explicitly defines reliable timestamps and their semantics.

---

# 19. Policy P-015 — Conflict Resolution Priority

When multiple policies apply simultaneously, CollectionHub must resolve them in a deterministic order.

Recommended priority:

```text
1. Domain invariants
2. Aggregate lifecycle constraints
3. Ownership / authorization
4. Membership eligibility
5. Uniqueness constraints
6. Capacity constraints
7. Classification / enrichment rules
8. Presentation / ordering rules
```

This ordering prevents secondary concerns from overriding fundamental domain rules.

For example:

```text
"Item should appear first"
```

must never override:

```text
"Archived collections cannot be modified"
```

---

# 20. Decision Table — Add Item to Collection

This is one of the principal composite domain decisions.

| Condition | Result |
|---|---|
| Collection does not exist | REJECT |
| Collection is deleted | REJECT |
| Actor not authorized | DENY |
| Collection not mutable | DENY |
| Item invalid | DENY |
| Item already member | DENY / IDEMPOTENT |
| Capacity exceeded | DENY |
| All invariants satisfied | ALLOW |

Conceptual evaluation:

```text
CanAddItemToCollection
        │
        ├── Collection exists?
        │      └── NO → REJECT
        │
        ├── Collection mutable?
        │      └── NO → DENY
        │
        ├── Actor authorized?
        │      └── NO → DENY
        │
        ├── Item eligible?
        │      └── NO → DENY
        │
        ├── Duplicate?
        │      └── YES → DENY / IDEMPOTENT
        │
        ├── Capacity available?
        │      └── NO → DENY
        │
        └── ALLOW
```

---

# 21. Decision Table — Remove Item from Collection

| Condition | Result |
|---|---|
| Collection does not exist | REJECT |
| Collection deleted | REJECT |
| Actor unauthorized | DENY |
| Collection immutable | DENY |
| Membership does not exist | DENY / IDEMPOTENT |
| Membership exists | ALLOW |

---

# 22. Decision Table — Update Collection

| Condition | Result |
|---|---|
| Collection does not exist | REJECT |
| Collection deleted | REJECT |
| Actor unauthorized | DENY |
| Collection immutable | DENY |
| Update violates invariant | DENY |
| Update valid | ALLOW |

The policy must not decide which persistence mechanism is used.

---

# 23. Decision Table — Archive Collection

| Condition | Result |
|---|---|
| Collection does not exist | REJECT |
| Already archived | IDEMPOTENT |
| Deleted | REJECT |
| Actor unauthorized | DENY |
| Active | ALLOW |
| Draft | ALLOW only if lifecycle permits |
| Other unsupported state | DENY |

---

# 24. Decision Table — Restore / Reactivate Collection

| Condition | Result |
|---|---|
| Collection does not exist | REJECT |
| Active | IDEMPOTENT |
| Archived | ALLOW if reactivation supported |
| Deleted | DENY |
| Actor unauthorized | DENY |
| Reactivation violates invariant | DENY |

---

# 25. Decision Table — Delete Collection

| Condition | Result |
|---|---|
| Collection does not exist | IDEMPOTENT / REJECT according to use case |
| Already deleted | IDEMPOTENT |
| Actor unauthorized | DENY |
| Deletion blocked by invariant | DENY |
| Valid deletion | ALLOW |

The exact semantics must be consistent across the corresponding application use case.

---

# 26. Policy Composition

Composite policies should be represented conceptually as a decision pipeline.

Example:

```text
Request
  ↓
Authorization Policy
  ↓
Lifecycle Policy
  ↓
Eligibility Policy
  ↓
Uniqueness Policy
  ↓
Capacity Policy
  ↓
Domain Invariants
  ↓
Decision
```

The order is significant.

A later policy must never invalidate the assumptions required by an earlier policy without producing an explicit conflict.

---

# 27. Policy vs Invariant

The following distinction must remain explicit.

### Invariant

```text
A collection cannot contain the same logical item twice.
```

This is always true.

### Policy

```text
When AddItem is requested for an item already present,
the command is treated as idempotent.
```

This determines behavior in a particular situation.

Therefore:

```text
Invariant ≠ Policy
```

Both are required.

---

# 28. Policy vs Application Workflow

A policy answers:

```text
"Is this allowed?"
```

or:

```text
"What should the domain decide?"
```

An application workflow answers:

```text
"What sequence of operations should the system execute?"
```

Example:

```text
Application Workflow
    ↓
Load Collection
    ↓
Load Item
    ↓
Invoke AddItem Policy
    ↓
Invoke Aggregate
    ↓
Persist Aggregate
    ↓
Publish Event
```

The workflow coordinates.

The domain decides.

---

# 29. Policy vs Infrastructure

Infrastructure must not contain domain decisions.

Avoid:

```text
Repository decides whether an item may be added.
```

Prefer:

```text
Domain policy decides whether an item may be added.
Repository persists the resulting state.
```

Infrastructure may enforce technical constraints that support domain rules, but it must not become the source of business meaning.

---

# 30. Error Semantics

Policy failures should be represented using domain-level meanings.

Examples:

```text
CollectionNotMutable
ActorNotAuthorized
ItemNotEligible
DuplicateMembership
CollectionCapacityExceeded
InvalidLifecycleTransition
CollectionNotFound
ItemNotFound
ClassificationConflict
SynchronizationConflict
```

The domain should not expose transport-specific concepts such as:

```text
HTTP 403
HTTP 404
HTTP 409
HTTP 422
```

Those mappings belong to the application/interface layers.

---

# 31. Policy Traceability

Every significant domain policy should be traceable to one or more business concepts.

Recommended traceability:

| Policy | Origin |
|---|---|
| Collection Ownership | Authorization / ownership concepts |
| Collection Mutability | Lifecycle invariants |
| Item Eligibility | Item + Collection membership rules |
| Duplicate Membership | Collection membership invariant |
| Capacity | Collection capacity invariant |
| Lifecycle Transition | Collection lifecycle |
| Removal Eligibility | Membership + ownership |
| Reordering Eligibility | Collection ordering semantics |
| Metadata Update | Collection lifecycle |
| Visibility | Access / visibility semantics |
| Deletion | Collection lifecycle |
| Classification | Classification model |
| External Identity Resolution | External identity model |
| Synchronization Conflict | Synchronization semantics |

This mapping should be maintained whenever the domain model evolves.

---

# 32. Policy Testing Strategy

Every policy should be testable independently from:

- HTTP;
- database;
- UI;
- message broker;
- framework;
- external provider.

Tests should primarily verify:

```text
Given
When
Then
```

Example:

```text
Given an active collection
And an authorized owner
And a valid item
And the item is not already a member

When the item is added

Then the policy returns ALLOW
```

Negative cases are equally important.

```text
Given an archived collection

When an item is added

Then the policy returns DENY
```

---

# 33. Decision Table Testing

Decision tables should produce a minimum test set covering:

- every positive decision;
- every negative decision;
- every boundary condition;
- every conflict condition;
- every lifecycle transition;
- every idempotent operation;
- every invariant violation.

The objective is not merely high code coverage.

The objective is **decision coverage**.

---

# 34. Ambiguities Requiring Explicit Resolution

The following questions must be resolved before implementation if they have not already been settled by previous domain artifacts:

1. Is adding an already-present item an error or an idempotent no-op?
2. Can archived collections be reactivated?
3. Can metadata be modified while a collection is archived?
4. Is collection capacity bounded?
5. Is collection ordering meaningful to the domain or only to presentation?
6. Are collections private, shared, public, or some combination?
7. Can multiple actors collaborate on a collection?
8. What constitutes logical identity for an item?
9. How are external identifiers mapped to domain identities?
10. What is the authoritative source during synchronization conflicts?
11. Are synchronization conflicts automatically resolved or manually reviewed?
12. Is deletion reversible?
13. Which operations are idempotent?
14. Which state transitions are irreversible?
15. Which domain decisions require explicit user confirmation?

These questions should not be silently answered by implementation convenience.

---

# 35. Decision Policy Matrix

The following matrix summarizes the principal domain decisions.

| Operation | Authorization | Lifecycle | Membership | Uniqueness | Capacity | Result |
|---|---|---|---|---|---|---|
| Add item | Required | Mutable | Eligible | Must be unique | Must be available | ALLOW / DENY |
| Remove item | Required | Mutable | Must exist | N/A | N/A | ALLOW / DENY |
| Reorder item | Required | Mutable | Must exist | N/A | N/A | ALLOW / DENY |
| Update metadata | Required | Mutable | N/A | Attribute-specific | N/A | ALLOW / DENY |
| Archive | Required | Valid transition | N/A | N/A | N/A | ALLOW / DENY |
| Reactivate | Required | Valid transition | N/A | N/A | N/A | ALLOW / DENY |
| Delete | Required | Valid transition | N/A | N/A | N/A | ALLOW / DENY |
| Read | Access policy | Readable | N/A | N/A | N/A | ALLOW / DENY |

---

# 36. Architectural Constraints

The implementation of these policies must preserve the following architectural properties.

## 36.1 No business rules in controllers

Controllers / handlers should orchestrate input/output only.

---

## 36.2 No business rules in repositories

Repositories retrieve and persist domain state.

They may enforce technical uniqueness constraints required to preserve consistency, but the domain remains the source of business meaning.

---

## 36.3 No business rules in DTOs

DTOs transport data.

They do not decide whether an operation is valid.

---

## 36.4 No business rules in ORM mappings

Persistence mappings describe storage representation.

They must not become the domain model.

---

## 36.5 Policies must remain framework-independent

The domain policy model should be usable without:

```text
HTTP
REST
GraphQL
SQL
ORM
Message broker
Web framework
Cloud provider
External API
```

---

# 37. Evolution Rules

When a domain rule changes:

1. Update the corresponding policy.
2. Update the relevant decision table.
3. Review affected invariants.
4. Review affected aggregates.
5. Review affected use cases.
6. Review application workflows.
7. Update policy tests.
8. Review emitted domain events.
9. Review persistence constraints if applicable.
10. Record the change in the domain decision history.

A policy change is therefore treated as a **domain model change**, not merely a code change.

---

# 38. Final Domain Policy Principles

CollectionHub domain policies must satisfy the following principles:

```text
1. Explicit
2. Deterministic
3. Domain-oriented
4. Testable
5. Traceable
6. Composable
7. Independent of infrastructure
8. Consistent with invariants
9. Explicit about conflicts
10. Explicit about lifecycle
11. Explicit about idempotency
12. Explicit about authorization
```

The central principle is:

> **If a business decision matters enough to change domain state, the decision must be represented explicitly in the domain model.**

---

# 39. Relationship with Previous and Next Artifacts

This document sits between the structural domain model and the implementation-oriented application model.

```text
00 DOMAIN CONCEPT INVENTORY
        ↓
01–07 DOMAIN MODEL FOUNDATION
        ↓
08 DOMAIN INVARIANTS
        ↓
09 DOMAIN AGGREGATES & CONSISTENCY BOUNDARIES
        ↓
11 DOMAIN USE CASES & APPLICATION SERVICES
        ↓
12 APPLICATION USE CASES & WORKFLOWS
        ↓
15 DOMAIN POLICIES & DECISION TABLES
        ↓
[Next domain/application modeling artifacts]
```

The purpose of this artifact is to ensure that the model does not merely describe **what exists**, but also explicitly describes **how the domain decides**.

---

# 40. Definition of Done

This artifact is considered sufficiently mature when:

- [ ] Every significant domain decision has an identified policy.
- [ ] Every important policy has explicit inputs and outputs.
- [ ] Lifecycle decisions are represented as decision tables.
- [ ] Authorization-sensitive decisions are explicit.
- [ ] Duplicate behavior is explicit.
- [ ] Idempotency semantics are explicit.
- [ ] Capacity rules are explicit where applicable.
- [ ] Conflict resolution is explicit.
- [ ] Invariants and policies are clearly distinguished.
- [ ] Domain policies are separated from application workflows.
- [ ] Infrastructure does not own business decisions.
- [ ] Ambiguous business rules are explicitly identified.
- [ ] Policies are independently testable.
- [ ] Decision coverage can be derived from the tables.
- [ ] Policy changes are traceable to domain concepts.

---

**Status:** Domain policy baseline established.  
**Next step:** continue refining the remaining domain-model artifacts and use the accumulated model to identify unresolved business decisions before any implementation begins.