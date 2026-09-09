# CollectionHub — Domain Specifications and Reusable Rules

> **Phase:** 2.2 — Domain Modeling  
> **Artifact:** 18 — Domain Specifications and Reusable Rules  
> **Status:** Draft / Design Baseline  
> **Scope:** Domain Layer  
> **Depends on:** Domain Concept Inventory, Domain Glossary, Domain Invariants, Domain Aggregates, Domain Use Cases, Domain Policies and Decision Tables, Domain Events and Side Effects, Domain Services and Cross-Aggregate Rules

---

# 1. Purpose

This document defines the **Domain Specifications** and other reusable domain rules that may be used throughout CollectionHub.

The purpose is to prevent business rules from being duplicated across:

- Aggregates;
- Domain Services;
- Domain Policies;
- Application Services;
- Use Cases;
- repositories;
- controllers;
- validation layers.

A Specification represents a reusable domain predicate or business condition.

The fundamental idea is:

> **A Specification answers a domain question without owning the state that it evaluates.**

Examples:

```text
Is this Collection active?
Is this Collection mutable?
Is this Item eligible?
Is this actor the owner?
Does this membership satisfy the required condition?
Is this external identity valid?
```

---

# 2. Core Principle

A Specification should represent a meaningful business rule.

It should not merely wrap an arbitrary boolean expression.

Prefer:

```text
ActiveCollectionSpecification
```

over:

```text
CollectionStatusEqualsActiveSpecification
```

The first expresses domain meaning.

The second exposes implementation detail.

---

# 3. Specification Concept

Conceptually:

```text
Specification<T>
        │
        └── isSatisfiedBy(T) → boolean
```

For more complex rules, the result may be richer than a boolean, but the conceptual responsibility remains:

> Evaluate whether a domain condition is satisfied.

Specifications should normally be:

- deterministic;
- side-effect free;
- composable;
- reusable;
- independently testable;
- expressed in domain language.

---

# 4. Specification vs Invariant

These concepts must remain distinct.

## Invariant

An invariant describes a condition that must always hold for valid domain state.

Example:

```text
A Collection cannot contain the same logical Item twice.
```

The Aggregate is responsible for protecting this invariant.

## Specification

A Specification evaluates whether a condition is satisfied.

Example:

```text
CollectionIsActive
```

It can be used by a policy or service to determine whether an operation is allowed.

Therefore:

```text
Invariant
    → protects domain correctness

Specification
    → evaluates domain conditions
```

A Specification must not replace an Aggregate invariant.

---

# 5. Specification vs Policy

A Policy answers:

> What should the domain decide?

A Specification answers:

> Is this condition satisfied?

Example:

```text
ActiveCollectionSpecification
```

may answer:

```text
true
```

A policy may then combine it with other rules:

```text
CollectionMutationPolicy
```

to determine:

```text
ALLOW
DENY
```

Conceptually:

```text
Specifications
      ↓
Policy
      ↓
Domain Decision
```

---

# 6. Specification vs Domain Service

A Specification should be preferred when the behavior is primarily a predicate.

Example:

```text
Is the Collection active?
```

A Domain Service is more appropriate when the operation:

- involves several aggregates;
- performs a meaningful calculation;
- resolves conflicts;
- or produces a richer domain result.

Example:

```text
ResolveSynchronizationConflict
```

Therefore:

```text
Predicate → Specification
Decision → Policy
Cross-concept behavior → Domain Service
```

---

# 7. Specification Characteristics

A valid CollectionHub Specification should normally satisfy the following properties.

## 7.1 Deterministic

Same input means same result.

```text
same domain state
+
same evaluation context
=
same result
```

---

## 7.2 Side-effect free

Evaluating a Specification must not mutate domain state.

Avoid:

```text
isSatisfiedBy()
    → update entity
```

Prefer:

```text
isSatisfiedBy()
    → evaluate
    → return result
```

---

## 7.3 Domain-oriented

Specifications should use domain concepts.

Avoid persistence terminology.

---

## 7.4 Composable

Where useful, Specifications should be combinable.

```text
A AND B
A OR B
NOT A
```

---

## 7.5 Testable

A Specification should be testable without:

- HTTP;
- database;
- ORM;
- message broker;
- framework.

---

# 8. Composite Specifications

Specifications can be composed.

Conceptually:

```text
Specification A
       AND
Specification B
       AND
Specification C
```

Example:

```text
CanModifyCollection
    =
CollectionIsActive
AND
ActorOwnsCollection
AND
CollectionIsNotLocked
```

The composition should remain readable in domain language.

---

# 9. AND Specification

The `AND` composition is satisfied only when all constituent Specifications are satisfied.

```text
A AND B
```

Decision table:

| A | B | Result |
|---|---|---|
| False | False | False |
| False | True | False |
| True | False | False |
| True | True | True |

Example:

```text
ActiveCollection
AND
AuthorizedActor
```

---

# 10. OR Specification

The `OR` composition is satisfied when at least one constituent Specification is satisfied.

```text
A OR B
```

Example:

```text
CollectionOwner
OR
AuthorizedCollaborator
```

This is useful when multiple legitimate domain paths exist.

---

# 11. NOT Specification

The `NOT` composition inverts a condition.

```text
NOT ArchivedCollection
```

However, negation should be used carefully.

A business rule expressed as:

```text
NOT X
```

may be less explicit than a domain-specific Specification.

Prefer:

```text
CollectionIsMutable
```

over:

```text
NOT CollectionIsArchived
```

when the two concepts are not semantically identical.

---

# 12. Specification Composition Example

A conceptual rule:

```text
CanAddItemToCollection
=
CollectionIsActive
AND
ActorIsAuthorized
AND
ItemIsEligible
AND
ItemIsNotAlreadyMember
AND
CapacityIsAvailable
```

The resulting Specification may be useful as a reusable predicate.

However, the final business decision should still be represented by the appropriate Policy.

---

# 13. Initial Specification Inventory

The initial candidate inventory is:

| Specification | Purpose | Status |
|---|---|---|
| CollectionIsActive | Determines active lifecycle state | Candidate |
| CollectionIsMutable | Determines whether mutation is allowed | Candidate |
| CollectionIsArchived | Determines archived state | Candidate |
| CollectionIsDeleted | Determines deleted state | Candidate |
| ActorOwnsCollection | Determines ownership | Candidate |
| ActorCanModifyCollection | Determines modification authority | Candidate |
| ItemIsEligible | Determines item eligibility | Candidate |
| ItemIsAlreadyMember | Determines membership existence | Candidate |
| ItemIsNotAlreadyMember | Determines absence of membership | Candidate |
| CollectionHasCapacity | Determines capacity availability | Candidate |
| MembershipExists | Determines membership existence | Candidate |
| ValidLifecycleTransition | Determines transition validity | Candidate |
| ExternalIdentityIsValid | Validates external identity semantics | Candidate |
| ClassificationIsCompatible | Determines classification compatibility | Candidate |
| SynchronizationCanBeResolvedAutomatically | Determines automatic conflict eligibility | Candidate |

These are candidates and must be reduced if the final domain model does not require them.

---

# 14. Specification S-001 — CollectionIsActive

## Purpose

Determine whether a Collection is currently active.

```text
CollectionIsActive(collection)
    → true / false
```

This specification may be reused by:

- mutation policies;
- lifecycle policies;
- application decision logic.

It should not itself perform a state transition.

---

# 15. Specification S-002 — CollectionIsMutable

## Purpose

Determine whether the current lifecycle state allows mutation.

This is intentionally different from simply checking whether a Collection is active if the domain later supports additional mutable states.

Conceptually:

```text
CollectionIsMutable(collection)
```

may evolve independently from:

```text
CollectionIsActive(collection)
```

This avoids encoding assumptions about lifecycle states in unrelated policies.

---

# 16. Specification S-003 — CollectionIsArchived

Determines whether the Collection is archived.

```text
CollectionIsArchived(collection)
```

This is useful for lifecycle decisions and read/access policies.

It must not be used as a substitute for every possible mutability rule.

---

# 17. Specification S-004 — CollectionIsDeleted

Determines whether a Collection is in the deleted state.

```text
CollectionIsDeleted(collection)
```

This can be used to prevent operations that are invalid after deletion.

---

# 18. Specification S-005 — ActorOwnsCollection

## Purpose

Determine whether an Actor is the owner of a Collection.

Conceptually:

```text
ActorOwnsCollection(
    collection,
    actor
)
```

The Specification evaluates domain identity.

It must not know how authentication occurred.

---

# 19. Specification S-006 — ActorCanModifyCollection

This Specification represents domain-level modification authority.

It may compose:

```text
ActorOwnsCollection
OR
AuthorizedCollaborator
```

if collaboration exists in the final model.

The important distinction is:

```text
authenticated actor
```

does not automatically mean:

```text
authorized actor
```

---

# 20. Specification S-007 — ItemIsEligible

Determines whether an Item satisfies the conditions required for collection membership.

Possible conditions may include:

- valid domain state;
- required identity;
- lifecycle eligibility;
- absence of invalid classification;
- other business constraints.

The exact conditions must be derived from the final Item model.

---

# 21. Specification S-008 — ItemIsAlreadyMember

Determines whether an Item is already a member of a Collection.

Conceptually:

```text
ItemIsAlreadyMember(
    collection,
    itemId
)
```

The identity used must be the domain identity established by the model.

---

# 22. Specification S-009 — ItemIsNotAlreadyMember

This is the inverse membership predicate.

```text
NOT ItemIsAlreadyMember
```

However, a named Specification may still be preferable if:

```text
ItemIsNotAlreadyMember
```

has independent semantic importance.

The choice should prioritize readability over mechanical reuse.

---

# 23. Specification S-010 — CollectionHasCapacity

Determines whether a Collection can accept another member.

Conceptually:

```text
CollectionHasCapacity(collection)
```

The Specification must use the domain's capacity semantics.

If CollectionHub has no capacity constraint, this Specification should not exist.

---

# 24. Specification S-011 — MembershipExists

Determines whether a specific membership exists.

```text
MembershipExists(
    collection,
    itemId
)
```

This may be used by:

- remove policies;
- reorder policies;
- update membership policies.

---

# 25. Specification S-012 — ValidLifecycleTransition

Determines whether a requested lifecycle transition is supported.

Conceptually:

```text
ValidLifecycleTransition(
    currentState,
    requestedState
)
```

Example:

```text
Active → Archived
```

may be valid.

```text
Archived → Draft
```

may not be.

The definitive transitions must remain aligned with the Collection lifecycle model.

---

# 26. Specification S-013 — ExternalIdentityIsValid

Determines whether an external identity satisfies domain identity requirements.

Possible inputs:

```text
source
externalIdentity
```

The Specification must not validate HTTP-specific details such as response codes.

It validates the domain meaning of the external identity.

---

# 27. Specification S-014 — ClassificationIsCompatible

Determines whether a classification can be applied to a domain concept.

Conceptually:

```text
ClassificationIsCompatible(
    item,
    classification
)
```

This is useful when compatibility is a reusable business predicate.

---

# 28. Specification S-015 — SynchronizationCanBeResolvedAutomatically

Determines whether a synchronization conflict can be resolved without manual intervention.

Conceptually:

```text
SynchronizationCanBeResolvedAutomatically(
    localState,
    externalState,
    conflictContext
)
```

Possible result:

```text
true
false
```

The actual resolution should remain the responsibility of the conflict-resolution domain behavior.

---

# 29. Specifications Should Not Hide Complex Decisions

Avoid creating a Specification such as:

```text
EverythingIsValidForAddingAnItemToCollection
```

if it internally performs dozens of unrelated business decisions.

A Specification should remain understandable.

Prefer:

```text
CollectionIsMutable
ActorCanModifyCollection
ItemIsEligible
CollectionHasCapacity
```

and compose them explicitly.

---

# 30. Reusable Rules Must Have a Single Meaning

A Specification should have one stable semantic meaning.

For example:

```text
CollectionIsActive
```

must always mean:

> The Collection is in the active lifecycle state.

It should not later acquire hidden conditions such as:

```text
active
AND
not locked
AND
owner has quota
```

Those are different rules.

This principle prevents semantic drift.

---

# 31. Avoid Semantic Overloading

Do not reuse one Specification for two different business concepts simply because both currently return `true` or `false`.

Example:

```text
CollectionIsActive
```

should not become synonymous with:

```text
CollectionCanBeModified
```

The concepts may coincide today but diverge later.

Explicit semantic names make the model resilient to change.

---

# 32. Specification Inputs

Specifications should receive the minimum domain information required for evaluation.

Avoid:

```text
EntireApplicationContext
```

Prefer:

```text
Collection
```

or:

```text
Collection + Actor
```

or:

```text
LocalState + ExternalState
```

depending on the rule.

---

# 33. Specification Dependencies

Pure Specifications should generally have no infrastructure dependencies.

A Specification that requires repository access is possible, but should be treated carefully.

For example:

```text
UniqueCollectionName
```

might require knowing whether another Collection already uses the same name.

This creates a query dependency.

Before introducing such a Specification, evaluate whether the rule belongs in:

- a Domain Service;
- a Policy;
- an Aggregate;
- a repository-backed domain query;
- an infrastructure uniqueness constraint.

---

# 34. Repository-Backed Specifications

A repository-backed Specification should be exceptional.

If required, the abstraction must remain domain-oriented.

Prefer:

```text
CollectionNameUniquenessChecker
```

over:

```text
SqlCollectionLookupSpecification
```

The domain must not know how the information is retrieved.

---

# 35. Specifications and Aggregate Invariants

A Specification may predict whether an operation is likely to be valid.

The Aggregate remains the final authority over its invariants.

Example:

```text
CollectionHasCapacity
```

may evaluate capacity.

But:

```text
Collection.addItem()
```

must still enforce the capacity invariant at mutation time.

This protects against stale decisions and concurrent modifications.

---

# 36. Specification Evaluation vs Mutation

Specifications evaluate.

Aggregates mutate.

Correct:

```text
Specification
    ↓
isSatisfiedBy
    ↓
boolean / result
```

Then:

```text
Aggregate
    ↓
perform mutation
```

Incorrect:

```text
Specification
    ↓
evaluate
    ↓
mutate Aggregate
```

---

# 37. Specification Composition in Policies

A Policy may use Specifications to produce a richer domain decision.

Example:

```text
CanAddItemToCollection
```

may conceptually evaluate:

```text
CollectionIsMutable
AND ActorCanModifyCollection
AND ItemIsEligible
AND ItemIsNotAlreadyMember
AND CollectionHasCapacity
```

The Policy then translates the failing condition into a domain-level decision.

Example:

```text
CollectionHasCapacity = false
```

becomes:

```text
CAPACITY_EXCEEDED
```

This is why Specification and Policy should remain distinct.

---

# 38. Composite Rule Example

Conceptual model:

```text
CanModifyCollection
        │
        ├── CollectionIsMutable
        │
        └── ActorCanModifyCollection
```

And:

```text
CanAddItem
        │
        ├── CanModifyCollection
        ├── ItemIsEligible
        ├── ItemIsNotAlreadyMember
        └── CollectionHasCapacity
```

This provides a readable domain decision tree.

---

# 39. Specification Evaluation Context

Some Specifications may require contextual information.

Example:

```text
ActorCanModifyCollection(
    collection,
    actor
)
```

The context should be explicit.

Avoid hidden dependencies such as:

```text
CurrentUser
CurrentTenant
CurrentTime
CurrentRequest
```

unless these are passed explicitly as domain concepts.

---

# 40. Temporal Specifications

If time is part of the domain, the evaluation time must be explicit.

Example:

```text
CollectionIsEditableAt(
    collection,
    evaluationTime
)
```

This makes the Specification:

- deterministic;
- testable;
- independent from system clocks.

Avoid:

```text
CollectionIsEditableNow()
```

inside the domain.

---

# 41. Tenant / Ownership Context

If CollectionHub eventually supports multiple tenants or ownership scopes, Specifications should use explicit domain concepts.

Example:

```text
ActorBelongsToCollectionScope(
    actor,
    collection
)
```

The domain should not inspect:

```text
HTTP headers
JWT claims
request context
```

---

# 42. Authorization Specifications

Authorization-related Specifications can exist when authorization is part of domain meaning.

Examples:

```text
ActorOwnsCollection
ActorCanModifyCollection
ActorCanArchiveCollection
ActorCanDeleteCollection
```

However, technical access-control mechanisms remain outside the domain.

The distinction is:

```text
Domain authorization rule
```

versus:

```text
Authentication / technical authorization mechanism
```

---

# 43. Lifecycle Specifications

Lifecycle Specifications should use domain lifecycle concepts.

Candidate rules:

```text
CollectionIsActive
CollectionIsArchived
CollectionIsDeleted
CollectionIsMutable
CollectionCanBeArchived
CollectionCanBeReactivated
CollectionCanBeDeleted
```

Only introduce a dedicated Specification when it improves semantic clarity or reuse.

---

# 44. Membership Specifications

Candidate membership rules:

```text
MembershipExists
ItemIsAlreadyMember
ItemIsNotAlreadyMember
MembershipCanBeAdded
MembershipCanBeRemoved
MembershipCanBeReordered
```

`MembershipCanBeAdded` should only be introduced if it represents a stable reusable domain concept rather than a large hidden policy.

---

# 45. Identity Specifications

Potential identity rules include:

```text
ExternalIdentityIsValid
ExternalIdentityMatchesItem
IdentityIsUnique
IdentityCanBeLinked
```

Identity Specifications should use domain identity concepts rather than database IDs.

---

# 46. Classification Specifications

Potential rules:

```text
ClassificationIsValid
ClassificationIsCompatible
ClassificationCanBeChanged
ClassificationConflictExists
```

These should remain separate if they represent different business meanings.

---

# 47. Synchronization Specifications

Potential rules:

```text
SynchronizationIsRequired
SynchronizationCanBeAutomatic
SynchronizationConflictExists
SynchronizationCanBeResolvedAutomatically
```

These should not be confused with technical scheduling conditions.

For example:

```text
"Synchronization job is scheduled"
```

is not a domain Specification.

---

# 48. Specifications and External Data

A Specification may evaluate external state if that state has already been translated into domain concepts.

Correct:

```text
ExternalItemRepresentation
```

Incorrect:

```text
HttpResponse
```

The domain should evaluate semantic information, not transport artifacts.

---

# 49. Specification Result Semantics

A simple Specification normally returns:

```text
true / false
```

However, some business rules may require richer outcomes.

For example:

```text
IdentityResolution
```

may produce:

```text
MATCH
NO_MATCH
CONFLICT
INVALID
```

In such cases, a Domain Service or dedicated domain resolver may be more appropriate than forcing everything into a boolean Specification.

This prevents Specifications from becoming disguised services.

---

# 50. When Not to Use a Specification

Do not create a Specification when:

1. the rule is used only once;
2. the rule belongs naturally inside an Aggregate;
3. the rule is actually a complex decision;
4. the rule performs side effects;
5. the rule requires infrastructure;
6. the name adds no semantic value;
7. the rule merely wraps a trivial implementation detail.

For example:

```text
CollectionIdIsNotNull
```

may not deserve a Domain Specification if the Value Object already guarantees valid identity.

---

# 51. Avoid Specification Explosion

A model containing hundreds of tiny Specifications can become harder to understand than the original conditional logic.

The goal is not maximum abstraction.

The goal is:

> **Explicit and reusable business meaning.**

A Specification should justify its existence through semantic value.

---

# 52. Specification Naming Rules

Use domain language.

Preferred:

```text
CollectionIsMutable
ActorOwnsCollection
ItemIsEligible
CollectionHasCapacity
ClassificationIsCompatible
```

Avoid:

```text
CollectionStatusPredicate
CheckCollectionState
ValidateCollectionBoolean
CollectionCondition1
```

Names should read naturally when composed.

---

# 53. Testing Specifications

Each Specification should have focused tests.

Example:

```text
Given an active Collection

When CollectionIsActive is evaluated

Then result is true
```

And:

```text
Given an archived Collection

When CollectionIsActive is evaluated

Then result is false
```

Boundary cases should be explicit.

---

# 54. Composite Specification Testing

Composite Specifications require truth-table testing.

For:

```text
A AND B
```

test all combinations.

For:

```text
A OR B
```

test all combinations.

For:

```text
NOT A
```

test both states.

This prevents subtle logical errors from being hidden inside business policies.

---

# 55. Specification Test Matrix

| Specification | Positive | Negative | Boundary | Conflict |
|---|---:|---:|---:|---:|
| CollectionIsActive | ✓ | ✓ | ✓ | |
| CollectionIsMutable | ✓ | ✓ | ✓ | |
| ActorOwnsCollection | ✓ | ✓ | | |
| ActorCanModifyCollection | ✓ | ✓ | ✓ | ✓ |
| ItemIsEligible | ✓ | ✓ | ✓ | ✓ |
| ItemIsAlreadyMember | ✓ | ✓ | | |
| CollectionHasCapacity | ✓ | ✓ | ✓ | |
| ValidLifecycleTransition | ✓ | ✓ | ✓ | ✓ |
| ExternalIdentityIsValid | ✓ | ✓ | ✓ | ✓ |
| ClassificationIsCompatible | ✓ | ✓ | ✓ | ✓ |
| SynchronizationCanBeResolvedAutomatically | ✓ | ✓ | ✓ | ✓ |

---

# 56. Specification Traceability

Each reusable rule should be traceable to one or more domain concepts.

| Specification | Related Domain Concept |
|---|---|
| CollectionIsActive | Collection Lifecycle |
| CollectionIsMutable | Collection Lifecycle |
| ActorOwnsCollection | Ownership |
| ActorCanModifyCollection | Authorization |
| ItemIsEligible | Item Lifecycle / Membership |
| ItemIsAlreadyMember | Collection Membership |
| CollectionHasCapacity | Collection Capacity |
| ValidLifecycleTransition | Collection Lifecycle |
| ExternalIdentityIsValid | External Identity |
| ClassificationIsCompatible | Classification |
| SynchronizationCanBeResolvedAutomatically | Synchronization |

This mapping must evolve with the domain model.

---

# 57. Specification Dependency Rules

Specifications may depend on:

```text
Value Objects
Entities
Aggregates
Domain concepts
Explicit evaluation context
Other Specifications
```

They should not depend directly on:

```text
Controllers
DTOs
HTTP requests
ORM entities
Database connections
Message brokers
External APIs
Framework services
```

---

# 58. Reusable Rule Hierarchy

The domain rules can be organized conceptually as:

```text
                    Domain Rules
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    Invariants       Specifications     Policies
        │                │                │
        │                │                └── Decisions
        │                │
        │                └── Predicates
        │
        └── State correctness
```

Domain Services sit alongside these when behavior spans concepts.

---

# 59. Full Behavioral Composition

The complete behavioral flow is now:

```text
Command
   ↓
Specification(s)
   ↓
Policy
   ↓
Domain Service if required
   ↓
Aggregate
   ↓
Invariant
   ↓
State Change
   ↓
Domain Event
   ↓
Application Workflow
   ↓
Side Effect
```

Not every use case requires every layer.

The model should remain as simple as the domain allows.

---

# 60. Example — Add Item

A conceptual implementation-independent decision model:

```text
CanAddItem
    │
    ├── CollectionIsMutable
    │
    ├── ActorCanModifyCollection
    │
    ├── ItemIsEligible
    │
    ├── ItemIsNotAlreadyMember
    │
    └── CollectionHasCapacity
```

If all conditions hold:

```text
ALLOW
```

Then:

```text
Collection.addItem(itemId)
```

Finally:

```text
ItemAddedToCollection
```

This illustrates the relationship between:

```text
Specification
Policy
Aggregate
Event
```

---

# 61. Example — Remove Item

```text
CanRemoveItem
    │
    ├── CollectionIsMutable
    │
    ├── ActorCanModifyCollection
    │
    └── MembershipExists
```

If all conditions are satisfied:

```text
Collection.removeItem(itemId)
```

Resulting event:

```text
ItemRemovedFromCollection
```

---

# 62. Example — Archive Collection

```text
CanArchiveCollection
    │
    ├── ActorCanModifyCollection
    │
    └── ValidLifecycleTransition
```

Then:

```text
Collection.archive()
```

Resulting event:

```text
CollectionArchived
```

---

# 63. Example — Reactivate Collection

```text
CanReactivateCollection
    │
    ├── ActorCanModifyCollection
    │
    └── ValidLifecycleTransition
```

Then:

```text
Collection.reactivate()
```

Resulting event:

```text
CollectionReactivated
```

---

# 64. Reusable Rule Governance

When a business rule changes, the impact analysis should include:

1. affected Specification;
2. policies composing it;
3. Aggregates relying on it;
4. Domain Services using it;
5. application use cases;
6. decision tables;
7. tests;
8. domain events if behavior changes.

A Specification must not be changed casually because it may affect multiple domain behaviors.

---

# 65. Open Questions

The following questions remain intentionally open:

1. Which candidate Specifications are actually reused enough to justify extraction?
2. Which predicates belong directly to Aggregates?
3. Which Specifications require cross-aggregate state?
4. Which rules are true domain predicates versus application validation?
5. Which authorization rules are domain-specific?
6. Which identity rules require repository-backed queries?
7. Which temporal rules exist?
8. Which classification rules require reusable predicates?
9. Which synchronization rules are deterministic?
10. Which composite Specifications improve readability?
11. Which Specifications should instead become Policies?
12. Which Specifications should instead become Domain Services?

These questions should be resolved before implementation abstractions are frozen.

---

# 66. Architectural Constraints

The following constraints become part of the CollectionHub modeling baseline.

### SPEC-001

Specifications express reusable domain predicates.

### SPEC-002

Specifications must be deterministic.

### SPEC-003

Specifications must be side-effect free.

### SPEC-004

Specifications must use domain language.

### SPEC-005

Specifications must not replace Aggregate invariants.

### SPEC-006

Specifications must not hide complex domain decisions.

### SPEC-007

Specifications should not depend directly on infrastructure.

### SPEC-008

Specifications may be composed when composition improves domain readability.

### SPEC-009

A Specification must have a stable semantic meaning.

### SPEC-010

Avoid creating Specifications solely for trivial technical validation.

### SPEC-011

Repository-backed Specifications require explicit architectural justification.

### SPEC-012

Aggregate mutation remains responsible for enforcing invariants even when Specifications are evaluated beforehand.

---

# 67. Definition of Done

This artifact is considered sufficiently mature when:

- [ ] Reusable domain predicates have been identified.
- [ ] Specifications are clearly distinguished from invariants.
- [ ] Specifications are clearly distinguished from policies.
- [ ] Specifications are clearly distinguished from Domain Services.
- [ ] Candidate Specifications have explicit domain meanings.
- [ ] Composite rules are documented where useful.
- [ ] Lifecycle Specifications are identified.
- [ ] Membership Specifications are identified.
- [ ] Authorization Specifications are identified.
- [ ] Identity Specifications are identified.
- [ ] Classification Specifications are identified.
- [ ] Synchronization Specifications are identified.
- [ ] Repository-backed rules are explicitly identified.
- [ ] Specification testing strategy exists.
- [ ] Specification dependencies are constrained.
- [ ] Semantic duplication is avoided.
- [ ] Open modeling questions are recorded.

---

# 68. Relationship With Previous Artifacts

The domain model now follows this behavioral progression:

```text
08_DOMAIN_INVARIANTS
        ↓
09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES
        ↓
11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES
        ↓
12_APPLICATION_USE_CASES_AND_WORKFLOWS
        ↓
15_DOMAIN_POLICIES_AND_DECISION_TABLES
        ↓
16_DOMAIN_EVENTS_AND_SIDE_EFFECTS
        ↓
17_DOMAIN_SERVICES_AND_CROSS_AGGREGATE_RULES
        ↓
18_DOMAIN_SPECIFICATIONS_AND_REUSABLE_RULES
```

The accumulated model now distinguishes:

```text
Invariant
    → What must always be true

Specification
    → Whether a domain condition is satisfied

Policy
    → What decision should be made

Domain Service
    → How meaningful cross-concept domain behavior is evaluated

Aggregate
    → Where state and consistency are owned

Domain Event
    → What happened

Application Service
    → How the use case is coordinated

Infrastructure
    → How technical side effects are executed
```

---

# 69. Final Principle

The purpose of Specifications is not to maximize abstraction.

It is to make reusable domain knowledge explicit.

Therefore:

> **Extract a Specification when a business condition has independent semantic meaning, is useful across domain behaviors, and benefits from being evaluated consistently.**

A good CollectionHub model should make it possible to read a rule such as:

```text
CanAddItemToCollection
    =
CollectionIsMutable
AND
ActorCanModifyCollection
AND
ItemIsEligible
AND
ItemIsNotAlreadyMember
AND
CollectionHasCapacity
```

and immediately understand the business decision without needing to inspect persistence, framework or infrastructure code.

---

**Status:** Domain Specifications and Reusable Rules baseline established.

**Next step:** continue the domain-modeling sequence by consolidating the complete domain vocabulary, boundaries, decisions, services and rules into a coherent domain contract before moving toward implementation architecture.