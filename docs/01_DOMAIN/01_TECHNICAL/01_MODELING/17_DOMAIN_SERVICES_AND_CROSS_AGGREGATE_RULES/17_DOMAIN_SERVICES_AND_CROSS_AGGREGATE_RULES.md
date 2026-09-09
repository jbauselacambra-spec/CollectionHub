# CollectionHub — Domain Services and Cross-Aggregate Rules

> **Phase:** 2.2 — Domain Modeling  
> **Artifact:** 17 — Domain Services and Cross-Aggregate Rules  
> **Status:** Draft / Design Baseline  
> **Scope:** Domain Layer  
> **Depends on:** Domain Concept Inventory, Domain Glossary, Domain Invariants, Domain Aggregates, Domain Use Cases, Application Workflows, Domain Policies and Decision Tables, Domain Events and Side Effects

---

# 1. Purpose

This document defines the **Domain Services** and **cross-aggregate business rules** required by CollectionHub.

The purpose is to establish clear boundaries between:

- logic that belongs inside an Aggregate;
- logic that belongs inside a Value Object;
- logic that belongs inside a Domain Service;
- logic that belongs inside an Application Service;
- rules that require coordination between multiple Aggregates;
- and rules that should not be represented as domain behavior at all.

The central objective is to prevent business logic from being placed arbitrarily in application services, repositories, controllers or infrastructure.

---

# 2. Core Principle

A Domain Service exists when a domain operation:

1. represents meaningful domain behavior;
2. does not naturally belong to a single entity or aggregate;
3. requires one or more domain concepts to make a decision;
4. and should remain independent of application/infrastructure concerns.

A Domain Service must not become a generic utility class.

The guiding question is:

> **Does this operation express domain meaning that cannot naturally be owned by one Aggregate?**

If the answer is no, a Domain Service is probably unnecessary.

---

# 3. Placement Decision Framework

Before introducing a Domain Service, apply the following decision sequence.

```text
Is the rule about one Value Object?
        │
       YES → Value Object

Is the rule about one Entity/Aggregate?
        │
       YES → Aggregate

Does the rule require multiple Aggregates
and represent domain meaning?
        │
       YES → Domain Service

Does the operation coordinate a use case
without containing domain decisions?
        │
       YES → Application Service

Is the operation technical?
        │
       YES → Infrastructure
```

This decision tree should be applied before creating every new service.

---

# 4. Aggregate Boundary Principle

Aggregates are consistency boundaries.

A Domain Service must not be used as an excuse to bypass those boundaries.

Incorrect:

```text id="qv2v42"
Domain Service
    ↓
mutates Collection
    ↓
mutates Item
    ↓
mutates Classification
    ↓
mutates Synchronization
```

This creates an artificial super-aggregate.

Prefer:

```text id="04p3lk"
Domain Service
    ↓
evaluates cross-aggregate rule
    ↓
returns decision
    ↓
Application Service coordinates commands
    ↓
each Aggregate changes its own state
```

---

# 5. Cross-Aggregate Rule

A cross-aggregate rule is a business rule whose decision depends on state belonging to more than one Aggregate.

Examples include:

- whether an Item may be added to a Collection;
- whether an external identity may be linked to an existing Item;
- whether an ownership transfer is valid across related objects;
- whether a synchronization conflict can be automatically resolved;
- whether a collection operation is allowed based on another domain concept.

Cross-aggregate rules require particular care because they often introduce consistency concerns.

---

# 6. Strong vs Eventual Consistency

Not every cross-aggregate rule requires a single transaction.

CollectionHub should distinguish between:

### Strong consistency

The rule must be satisfied before the operation is committed.

Example:

```text
An Item cannot be added to a Collection
if the Collection is immutable.
```

### Eventual consistency

The rule can be evaluated or completed asynchronously.

Example:

```text
After ItemAddedToCollection,
update a search projection.
```

### Architectural principle

Do not introduce strong transactional coupling between Aggregates unless the business rule genuinely requires it.

---

# 7. Candidate Domain Services

The initial CollectionHub model suggests the following candidate services:

| Service | Purpose | Status |
|---|---|---|
| CollectionMembershipPolicy | Determine membership eligibility | Candidate |
| CollectionMembershipService | Coordinate membership-specific domain logic | Candidate |
| CollectionAccessPolicy | Evaluate domain-level access | Candidate |
| ItemIdentityResolutionService | Resolve external item identity | Candidate |
| ClassificationService | Apply classification rules | Candidate |
| SynchronizationConflictResolver | Resolve synchronization conflicts | Candidate |
| CollectionEligibilityService | Determine whether a collection operation is valid | Candidate |
| OwnershipTransferService | Validate ownership transfer across concepts | Candidate |
| CollectionOrderingService | Apply domain-level ordering rules | Candidate if ordering is domain-significant |

These are **candidates**, not automatic implementation requirements.

A service should only be created when the final domain model confirms that the behavior cannot be naturally owned by an Aggregate.

---

# 8. Domain Service DS-001 — Collection Membership Eligibility

## Purpose

Determine whether an Item can become a member of a Collection.

The service may need to consider:

- Collection state;
- Item state;
- ownership/authorization context;
- membership existence;
- collection capacity;
- membership-specific business rules.

## Conceptual API

```text id="9i5l7b"
evaluateMembershipEligibility(
    collection,
    item,
    actorContext
) → MembershipDecision
```

The result should be a domain decision, not an infrastructure response.

Possible results:

```text
ALLOWED
ALREADY_MEMBER
COLLECTION_NOT_MUTABLE
ITEM_NOT_ELIGIBLE
CAPACITY_EXCEEDED
NOT_AUTHORIZED
```

---

# 9. Important Boundary for Membership

If all membership rules can be evaluated entirely inside the `Collection` Aggregate, no Domain Service is required.

For example:

```text
collection.addItem(itemId)
```

is preferable when the Collection already owns all information required to enforce the rule.

A Domain Service becomes justified when the decision genuinely depends on independent Aggregates:

```text
Collection
    +
Item
    +
another domain concept
```

and the rule cannot be naturally assigned to one of them.

---

# 10. Domain Service DS-002 — Item Identity Resolution

## Purpose

Determine whether an externally identified item corresponds to an existing domain Item.

## Inputs

```text
externalSource
externalIdentity
candidateItemData
```

## Output

Conceptually:

```text
ExistingItem
NewItemCandidate
IdentityConflict
InvalidExternalIdentity
```

## Responsibilities

The service may:

- interpret domain-level identity rules;
- compare candidate identities;
- detect conflicting mappings;
- determine whether a match is sufficiently strong.

It must not:

- call HTTP APIs directly;
- execute SQL;
- know ORM entities;
- persist results.

Those responsibilities belong elsewhere.

---

# 11. Domain Service DS-003 — Synchronization Conflict Resolver

## Purpose

Determine how a synchronization conflict should be resolved according to domain rules.

## Inputs

```text
localState
externalState
conflictContext
```

## Output

```text
LOCAL_WINS
EXTERNAL_WINS
MERGED
MANUAL_REVIEW
REJECT
```

The service must be deterministic.

Given the same inputs:

```text
same inputs
    ↓
same resolution
```

unless an explicitly provided domain decision context changes.

---

# 12. Conflict Resolution Rule

The resolver must not silently introduce a technical rule such as:

```text
latest database update wins
```

unless the domain explicitly defines that concept.

Technical timestamps and database metadata are not automatically domain truth.

If "latest modification wins" is a business rule, the model must define:

- what modification time means;
- which clock is authoritative;
- whether timestamps are comparable;
- how concurrent updates are handled.

---

# 13. Domain Service DS-004 — Classification Service

If classification involves more than one Aggregate or a set of domain rules, a Domain Service may evaluate the classification.

Potential responsibilities:

```text
evaluateClassification(item, context)
    ↓
ClassificationDecision
```

The service may determine:

- applicable classification;
- conflicting classifications;
- whether a classification is compatible with the item;
- whether manual review is required.

The service must not directly persist the classification.

---

# 14. Domain Service DS-005 — Ownership Transfer

If ownership transfer involves several domain concepts, a dedicated service may evaluate the operation.

Conceptually:

```text
evaluateOwnershipTransfer(
    collection,
    currentOwner,
    proposedOwner,
    context
)
```

Possible outcomes:

```text
ALLOWED
NOT_AUTHORIZED
INVALID_TARGET
TRANSFER_NOT_SUPPORTED
REQUIRES_REVIEW
```

If ownership is fully contained by the Collection Aggregate, the transfer should remain an Aggregate operation instead.

---

# 15. Domain Service DS-006 — Collection Eligibility

A generic `CollectionEligibilityService` should only exist if there are meaningful rules involving multiple independent domain concepts.

It may evaluate:

```text
Collection
+
Actor
+
Item
+
Subscription/Quota/Entitlement
+
Other domain concept
```

The service must not become a dumping ground for unrelated validation logic.

---

# 16. Domain Service DS-007 — Collection Ordering

Ordering deserves special treatment.

If ordering is simply:

```text
display position
```

it may belong to the application/read-model layer.

If ordering has domain semantics such as:

```text
priority
ranking
curated sequence
manual ordering with business meaning
```

then a Domain Service may be justified.

Potential operation:

```text
calculateNewPosition(
    collection,
    item,
    requestedPosition
)
```

The service should enforce domain ordering rules while the Collection Aggregate remains responsible for maintaining its own invariant.

---

# 17. Service vs Aggregate Responsibility

The following rule should be applied consistently.

### Aggregate owns:

- state;
- invariants;
- state transitions;
- behavior over its own state;
- domain events resulting from its state changes.

### Domain Service owns:

- domain decisions involving multiple concepts;
- calculations that have business meaning;
- cross-aggregate policies;
- domain algorithms that do not naturally belong to one Aggregate.

### Application Service owns:

- workflow orchestration;
- loading Aggregates;
- transaction coordination;
- invoking Domain Services;
- invoking Aggregate methods;
- handling application-level errors.

---

# 18. Cross-Aggregate Operation Pattern

A typical operation should look conceptually like:

```text id="wyf19r"
Application Use Case
        ↓
Load Aggregate A
        ↓
Load Aggregate B
        ↓
Domain Service
        ↓
Cross-Aggregate Decision
        ↓
Aggregate A mutation
        ↓
Aggregate B mutation if independently required
        ↓
Persist changes
        ↓
Publish resulting events
```

The Domain Service should not own the entire workflow.

---

# 19. Example — Add Item to Collection

Suppose adding an Item requires information from both:

```text
Collection
Item
```

A possible design is:

```text
Application Service
    ↓
Load Collection
Load Item
    ↓
MembershipEligibilityService
    ↓
ALLOW
    ↓
Collection.addItem(itemId)
    ↓
ItemAddedToCollection
```

The service decides.

The Collection Aggregate mutates its own state.

The Application Service coordinates.

---

# 20. Avoiding the God Domain Service

A dangerous design is:

```text
CollectionDomainService
```

containing:

```text
createCollection()
updateCollection()
deleteCollection()
addItem()
removeItem()
archiveCollection()
classifyItem()
syncItem()
resolveConflict()
transferOwnership()
```

This service becomes an artificial domain façade and destroys Aggregate boundaries.

Prefer small services with explicit domain responsibilities.

---

# 21. Domain Services Should Be Stateless

A Domain Service should normally be stateless.

Conceptually:

```text
Inputs
  ↓
Domain Service
  ↓
Decision / Result
```

Avoid services containing mutable domain state between invocations.

If the service needs persistent state, that state probably belongs to an Aggregate or another domain concept.

---

# 22. Domain Service Dependencies

Domain Services may depend on domain abstractions when genuinely required.

Possible dependency:

```text
Domain Service
    ↓
Domain Repository Interface
```

However, repository dependencies should be minimized.

A Domain Service should not depend on:

```text
SQL connection
ORM session
HTTP client
message broker
filesystem
cache implementation
framework service container
```

These are infrastructure concerns.

---

# 23. Repository Access From Domain Services

Repository access requires particular care.

A Domain Service may need repository information to answer a domain question such as:

```text
Does another active Collection already own this unique domain relationship?
```

In that case the service may depend on an abstraction such as:

```text
CollectionRepository
```

but not on its implementation.

The repository abstraction must expose domain-oriented queries.

Prefer:

```text
findActiveCollectionForOwner(...)
```

over:

```text
executeSQL(...)
```

---

# 24. Query Responsibility

Domain Services should only query what is required for a domain decision.

Avoid loading entire graphs of objects merely because the repository makes it convenient.

Cross-aggregate rules should minimize unnecessary coupling.

---

# 25. Domain Service Return Types

A Domain Service should return explicit domain results.

Prefer:

```text
MembershipDecision
IdentityResolution
ConflictResolution
ClassificationDecision
OwnershipTransferDecision
```

over:

```text
boolean
null
generic object
exception for every expected business alternative
```

Boolean results often hide important domain semantics.

Instead of:

```text
true / false
```

prefer:

```text
ALLOWED
CAPACITY_EXCEEDED
ALREADY_MEMBER
NOT_AUTHORIZED
```

where those distinctions matter.

---

# 26. Expected Business Outcomes vs Errors

Not every negative decision is an exception.

For example:

```text
ALREADY_MEMBER
```

may be a normal domain outcome.

Likewise:

```text
MANUAL_REVIEW_REQUIRED
```

may represent a legitimate domain decision.

Unexpected technical failures should remain separate from expected business outcomes.

---

# 27. Cross-Aggregate Invariants

A cross-aggregate invariant should be introduced carefully.

Example:

```text
An Item can belong to at most one Collection.
```

If this rule truly exists, it affects multiple aggregates.

The architecture must decide where consistency is guaranteed.

Possible strategies:

### Strategy A — Single Aggregate

Redesign the aggregate boundary if the relationship requires strong transactional consistency.

### Strategy B — Domain Service + Repository Constraint

Use a domain decision plus an infrastructure uniqueness constraint.

### Strategy C — Eventual Consistency

Allow temporary divergence and reconcile asynchronously.

The correct choice depends on business semantics.

---

# 28. Aggregate Boundary Smell

If a Domain Service frequently performs:

```text
read Aggregate A
read Aggregate B
read Aggregate C
mutate A
mutate B
mutate C
```

this is a signal to reconsider the aggregate boundaries.

The service may be compensating for an incorrectly modeled consistency boundary.

---

# 29. Cross-Aggregate Transaction Smell

Similarly, if every important use case requires:

```text
transaction
    Aggregate A
    Aggregate B
    Aggregate C
    Aggregate D
```

then the model may have excessive transactional coupling.

The preferred architecture is:

```text
Aggregate
    ↓
local consistency
```

plus:

```text
Domain Event
    ↓
eventual consistency
```

where the business allows it.

---

# 30. Domain Service and Domain Events

A Domain Service may make a decision that causes one or more Aggregates to change.

However, the service itself should not generally fabricate events representing state it does not own.

Prefer:

```text
Domain Service
    ↓
Decision
    ↓
Aggregate mutation
    ↓
Aggregate records event
```

rather than:

```text
Domain Service
    ↓
directly publishes domain event
```

The aggregate remains the owner of its state transitions.

---

# 31. Domain Service and Application Events

A Domain Service should not publish integration messages.

For example, avoid:

```text
SynchronizationConflictResolver
    ↓
Kafka.publish(...)
```

Instead:

```text
SynchronizationConflictResolver
    ↓
ConflictResolution
    ↓
Aggregate / Domain State Change
    ↓
Domain Event
    ↓
Application / Infrastructure
```

---

# 32. Cross-Aggregate Rule Matrix

| Rule | Aggregates involved | Strong consistency? | Candidate owner |
|---|---|---:|---|
| Collection membership eligibility | Collection + Item | Usually yes for decision | Domain Policy / Service |
| External identity resolution | Item + External Identity | Depends | Domain Service |
| Ownership transfer | Collection + Actor/Ownership | Usually yes | Aggregate or Domain Service |
| Classification compatibility | Item + Classification | Depends | Aggregate / Domain Service |
| Synchronization conflict | Item + External representation | Usually process-level | Domain Service |
| Search projection update | Collection + Search Model | No | Application / Infrastructure |
| Notification | Collection + Actor | No | Application / Infrastructure |
| Cache invalidation | Any | No | Infrastructure |

---

# 33. Cross-Aggregate Decision Flow

A typical decision should follow:

```text id="ckz6z3"
Inputs
  ↓
Validate required domain state
  ↓
Evaluate cross-aggregate rule
  ↓
Return explicit decision
  ↓
Application coordinates resulting commands
  ↓
Aggregates enforce their own invariants
  ↓
Domain events are recorded
```

The Domain Service does not replace the Aggregates.

---

# 34. Service Naming

Domain Services should have names representing domain behavior.

Prefer:

```text
SynchronizationConflictResolver
ItemIdentityResolver
CollectionMembershipEligibility
OwnershipTransferPolicy
ClassificationResolver
```

Avoid:

```text
DomainHelper
CommonService
CollectionUtils
BusinessManager
GenericDomainService
Processor
Handler
```

Names should make the business responsibility obvious.

---

# 35. Policy vs Domain Service

These concepts overlap but should remain distinguishable.

### Domain Policy

Answers:

> What decision should the domain make?

Example:

```text
CollectionMembershipPolicy
```

### Domain Service

Provides:

> A domain operation that may evaluate or coordinate a business rule involving multiple concepts.

Example:

```text
ItemIdentityResolutionService
```

A policy can therefore be implemented internally by a Domain Service when appropriate.

---

# 36. Domain Service vs Application Service

| Concern | Domain Service | Application Service |
|---|---|---|
| Business rule | Yes | Coordinates |
| Domain calculation | Yes | No |
| Cross-aggregate decision | Yes | May invoke |
| Transaction orchestration | No | Yes |
| Repository loading | Minimal / abstracted | Yes |
| DTO mapping | No | Yes |
| HTTP | No | No |
| Message broker | No | No |
| Workflow coordination | No | Yes |
| Domain event handling | No | Yes |

---

# 37. Domain Service vs Repository

A Repository answers:

> How do I retrieve or persist an Aggregate?

A Domain Service answers:

> What does the domain decide or calculate?

Do not move business decisions into repositories.

---

# 38. Domain Service vs Factory

A Factory creates domain objects.

A Domain Service performs domain behavior or decisions.

Example:

```text
CollectionFactory
    → creates Collection
```

versus:

```text
CollectionMembershipEligibility
    → determines whether membership is allowed
```

If object construction contains complex domain rules, a Factory may be appropriate.

---

# 39. Domain Service vs Specification

A Specification expresses a reusable domain predicate.

For example:

```text
ActiveCollectionSpecification
```

may answer:

```text
isSatisfiedBy(collection)
```

A Domain Service is more appropriate when the operation:

- involves several concepts;
- performs a meaningful calculation;
- resolves a conflict;
- or produces a richer decision.

Specifications and Domain Services can coexist.

---

# 40. Cross-Aggregate Consistency Strategies

When a rule spans Aggregates, choose deliberately among:

```text
1. Same Aggregate
2. Synchronous Domain Service decision
3. Repository-backed domain check
4. Application-level coordination
5. Domain Event + eventual consistency
6. Process Manager / Saga
7. Infrastructure constraint
```

The decision must be based on business consistency requirements, not implementation convenience.

---

# 41. Process Manager Boundary

If a cross-aggregate workflow spans time and multiple asynchronous events, it should not be forced into a Domain Service.

For example:

```text
CollectionCreated
    ↓
External synchronization
    ↓
SynchronizationCompleted
    ↓
Classification
    ↓
ClassificationCompleted
```

This may eventually require a process manager / saga.

The Domain Service should remain focused on individual domain decisions.

---

# 42. Temporal Rules

Domain Services may evaluate temporal rules if time is a domain concept.

The current time should be supplied explicitly:

```text
evaluateEligibility(domainState, evaluationTime)
```

Avoid hidden dependencies on:

```text
system clock
database NOW()
framework time service
```

This makes the rule deterministic and testable.

---

# 43. External Systems

Domain Services must not directly depend on external systems.

Incorrect:

```text
Domain Service
    ↓
Spotify API
```

Correct:

```text
Application Service
    ↓
External Gateway
    ↓
Domain representation
    ↓
Domain Service
```

The domain receives information; it does not own the communication mechanism.

---

# 44. Example — External Identity Resolution

Correct architecture:

```text
External API
    ↓
Infrastructure Adapter
    ↓
ExternalItemData
    ↓
Application Service
    ↓
ItemIdentityResolutionService
    ↓
IdentityResolution
    ↓
Aggregate mutation
```

The Domain Service never knows whether the external information came from:

- HTTP;
- file;
- cache;
- database;
- message;
- API.

---

# 45. Example — Synchronization Conflict

```text
External synchronization
        ↓
Application Service
        ↓
Local Item + External Item State
        ↓
SynchronizationConflictResolver
        ↓
Resolution
        ↓
Item Aggregate
        ↓
Domain Event
```

This keeps the conflict rule inside the domain while keeping communication infrastructure outside it.

---

# 46. Domain Service Testing

Domain Services should be testable without:

- HTTP;
- database;
- message broker;
- ORM;
- framework;
- UI.

Example:

```text
Given:
    active Collection
    valid Item
    authorized Actor

When:
    membership eligibility is evaluated

Then:
    decision = ALLOWED
```

For conflict resolution:

```text
Given:
    local state
    external state
    explicit conflict context

When:
    conflict is resolved

Then:
    resolution = LOCAL_WINS
```

---

# 47. Cross-Aggregate Test Cases

Testing should cover:

- valid combinations;
- invalid combinations;
- missing state;
- conflicting state;
- lifecycle restrictions;
- ownership restrictions;
- uniqueness;
- capacity;
- temporal boundaries;
- synchronization conflicts;
- idempotency;
- ambiguous identity.

The goal is **decision coverage**, not merely line coverage.

---

# 48. Candidate Service Inventory

The current candidate inventory is:

```text
CollectionMembershipEligibility
ItemIdentityResolver
SynchronizationConflictResolver
ClassificationResolver
OwnershipTransferPolicy
CollectionOrderingService
```

These candidates must be challenged during implementation design.

A candidate should be removed if the relevant behavior can be expressed naturally by an Aggregate or Value Object.

---

# 49. Rules for Introducing a New Domain Service

A new Domain Service should require justification.

Before introducing one, answer:

1. What domain rule does it represent?
2. Why does the rule not belong to an Aggregate?
3. Which Aggregates/concepts participate?
4. Is the rule synchronous or eventual?
5. Does it require repository access?
6. Is that access domain-oriented?
7. What does the service return?
8. What invariants remain owned by the Aggregates?
9. Can the service be deterministic?
10. Can it be tested without infrastructure?

If these questions cannot be answered clearly, the service probably does not belong in the domain.

---

# 50. Anti-Patterns

CollectionHub must explicitly avoid the following.

## 50.1 Anemic Aggregates

Aggregates containing only getters/setters while all business logic lives in services.

---

## 50.2 God Domain Service

One service containing most domain behavior.

---

## 50.3 Transaction Script Disguised as Domain Service

A service that merely performs:

```text
load
modify
save
send
```

without meaningful domain decisions.

That belongs in the Application layer.

---

## 50.4 Infrastructure Leakage

Domain Services depending on:

```text
ORM
SQL
HTTP
message broker
framework
cache
```

---

## 50.5 Hidden Aggregate Mutation

A service mutating several Aggregates without explicit application coordination.

---

## 50.6 Generic Utility Services

Services with names such as:

```text
DomainUtils
CollectionHelper
BusinessRules
CommonDomainService
```

These obscure domain meaning.

---

# 51. Aggregate Boundary Review Questions

The presence of a Domain Service should trigger a boundary review.

Ask:

### Question 1

Could these concepts actually belong to one Aggregate?

### Question 2

Does the business require atomic consistency between them?

### Question 3

Would moving them into one Aggregate create excessive size or contention?

### Question 4

Can the rule tolerate eventual consistency?

### Question 5

Is the service compensating for an incorrect model?

### Question 6

Is the relationship actually a domain relationship or merely a query/read-model concern?

---

# 52. Decision Matrix

| Scenario | Preferred Model |
|---|---|
| Rule concerns one Aggregate | Aggregate |
| Rule concerns one Value Object | Value Object |
| Rule is a reusable predicate | Specification / Policy |
| Rule spans Aggregates | Domain Service / Policy |
| Workflow coordinates several Aggregates | Application Service |
| Workflow spans asynchronous events | Process Manager / Saga |
| Technical operation | Infrastructure |
| Read-only projection | Query / Read Model |

---

# 53. Domain Service Contract

A conceptual contract should look like:

```text
Domain Service
    Input
      ↓
Domain Concepts
      ↓
Business Decision
      ↓
Explicit Domain Result
```

Not:

```text
Domain Service
    Input
      ↓
Database
      ↓
HTTP
      ↓
Mutation
      ↓
Message Broker
```

The latter is application/infrastructure orchestration.

---

# 54. Relationship With Domain Events

The final conceptual relationship is:

```text
Cross-Aggregate Rule
        ↓
Domain Service / Policy
        ↓
Decision
        ↓
Aggregate State Change
        ↓
Aggregate Domain Event
        ↓
Application Reaction
```

This maintains a clean separation between:

- deciding;
- changing state;
- recording facts;
- reacting to facts.

---

# 55. Architectural Constraints

The following become explicit CollectionHub architectural constraints.

### DS-001

Aggregates remain the primary owners of state and invariants.

### DS-002

Domain Services exist only for behavior that cannot naturally belong to an Aggregate or Value Object.

### DS-003

Domain Services must express domain meaning.

### DS-004

Domain Services must not orchestrate infrastructure.

### DS-005

Domain Services must not publish integration messages directly.

### DS-006

Cross-aggregate mutations must be coordinated explicitly.

### DS-007

Cross-aggregate consistency must be chosen deliberately.

### DS-008

Repository dependencies inside Domain Services must use domain abstractions.

### DS-009

Domain Services should remain stateless and deterministic.

### DS-010

The existence of a Domain Service should trigger periodic review of Aggregate boundaries.

---

# 56. Open Questions

The following questions remain intentionally open until the domain is further refined:

1. Which CollectionHub concepts are confirmed Aggregates?
2. Which membership rules actually require Item state?
3. Can all membership decisions be owned by Collection?
4. Is Item identity independent from external identity?
5. Does classification belong to Item or to a separate Aggregate?
6. Does ownership belong entirely to Collection?
7. Does ordering have business meaning?
8. Which cross-aggregate rules require strong consistency?
9. Which can be eventually consistent?
10. Are any relationships currently suggesting an incorrect Aggregate boundary?
11. Which processes are long-running enough to require a Process Manager?
12. Which domain queries require repository abstractions?
13. Which candidate Domain Services can ultimately be eliminated?

These questions should be resolved before freezing the implementation architecture.

---

# 57. Definition of Done

This artifact is considered sufficiently mature when:

- [ ] Aggregate responsibilities are clearly distinguished from Domain Services.
- [ ] Cross-aggregate rules are explicitly identified.
- [ ] Each candidate Domain Service has a documented purpose.
- [ ] Domain Services are justified by domain semantics.
- [ ] Aggregate boundaries have been reviewed against cross-aggregate rules.
- [ ] Strong vs eventual consistency decisions are identified.
- [ ] Repository dependencies are explicitly constrained.
- [ ] Application orchestration is separated from domain decisions.
- [ ] Domain Services are independent of infrastructure.
- [ ] Domain Service outputs are explicit domain results.
- [ ] Cross-aggregate mutation patterns are documented.
- [ ] Potential God Services have been identified and rejected.
- [ ] Testing strategies are defined.
- [ ] Long-running workflows are distinguished from Domain Services.
- [ ] Open modeling questions are recorded.

---

# 58. Relationship With Previous Artifacts

The current behavioral model now forms the following chain:

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
```

The resulting model can now express:

```text
Business Rule
      ↓
Invariant / Policy
      ↓
Aggregate Boundary
      ↓
Domain Decision
      ↓
Cross-Aggregate Rule if required
      ↓
Aggregate State Change
      ↓
Domain Event
      ↓
Application Workflow
      ↓
Infrastructure Side Effect
```

This provides a substantially more complete behavioral model before implementation begins.

---

# 59. Final Principle

The most important rule established by this artifact is:

> **A Domain Service should exist because the domain requires a meaningful operation that does not naturally belong to one Aggregate — never merely because it is convenient to put business logic somewhere else.**

Therefore:

```text
Aggregate
    → owns state and invariants

Domain Policy
    → defines domain decisions

Domain Service
    → handles meaningful behavior spanning domain concepts

Application Service
    → coordinates the use case

Domain Event
    → records what happened

Infrastructure
    → executes technical consequences
```

This separation preserves the integrity of the CollectionHub domain model and gives us a clear basis for reviewing the Aggregate boundaries before any production implementation begins.

---

**Status:** Domain Services and Cross-Aggregate Rules baseline established.

**Next step:** continue with the next domain-model artifact, focusing on the remaining structural and behavioral contracts required to make the domain model implementation-ready.