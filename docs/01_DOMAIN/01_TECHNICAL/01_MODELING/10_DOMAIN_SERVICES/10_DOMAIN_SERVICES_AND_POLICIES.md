# CollectionHub — Domain Services and Policies

## 1. Purpose

This document defines the role of **Domain Services and Domain Policies** within CollectionHub.

It establishes:

- which domain behavior belongs inside aggregates;
- which behavior does not naturally belong to a single aggregate;
- when a Domain Service is justified;
- when a Domain Policy is more appropriate;
- how cross-aggregate decisions should be modeled;
- how external information can participate in domain decisions;
- which responsibilities must remain outside the domain layer;
- which candidate services and policies are currently justified;
- which apparent services are deliberately rejected.

The objective is not to create a technical service layer.

The objective is to preserve domain semantics while keeping aggregate boundaries coherent.

The central question is:

> **When a domain decision cannot naturally be owned by one aggregate, where should that decision live?**

---

# 2. Core Principle

CollectionHub should follow this priority when locating domain behavior:

```text
1. Entity
   ↓
2. Value Object
   ↓
3. Aggregate
   ↓
4. Domain Policy
   ↓
5. Domain Service
```

This ordering is intentional.

A Domain Service should not become the default home for business logic.

The preferred design is always:

> **Keep behavior as close as possible to the domain concept that owns the invariant.**

Only when that ownership becomes unnatural should the behavior move outward.

---

# 3. What Is a Domain Service?

A Domain Service represents domain behavior that:

- is meaningful to the business;
- does not naturally belong to one entity;
- does not naturally belong to one aggregate;
- usually operates without its own persistent identity;
- may coordinate multiple domain concepts.

Conceptually:

```text id="x4m8qk"
Domain Service
      |
      +---- uses ----> Aggregate A
      |
      +---- uses ----> Aggregate B
      |
      v
Domain Decision
```

A Domain Service is therefore not:

- an application service;
- a controller;
- a repository;
- an API client;
- an infrastructure service;
- a generic utility class.

---

# 4. What Is a Domain Policy?

A Domain Policy represents a business rule that determines **how a decision should be made** when multiple valid alternatives or contextual conditions exist.

A policy answers questions such as:

> Which source should be considered authoritative?

> Is this item compatible with this collection?

> Which classification should be preferred?

> Which candidate should win when several candidates are valid?

A policy can therefore be understood as:

```text id="q6r2jw"
Context + Facts
       ↓
     Policy
       ↓
   Decision
```

---

# 5. Domain Service vs Domain Policy

The distinction is conceptual rather than merely technical.

## Domain Service

Focuses on:

> **Performing a domain operation that does not belong to one aggregate.**

Example:

```text
Compare two domain objects
Coordinate multiple domain concepts
Calculate a domain-specific result
```

## Domain Policy

Focuses on:

> **Choosing or determining the correct business outcome according to a rule.**

Example:

```text
Select authoritative metadata source
Determine compatibility
Choose preferred classification
Resolve conflicting values
```

---

# 6. First Principle — Aggregate Ownership Comes First

Before introducing a Domain Service or Policy, ask:

> **Can this behavior belong to the aggregate that owns the relevant invariant?**

If yes, it should remain there.

For example:

```text id="n7c3tp"
AddItemToCollection
```

should belong to Collection because Collection owns:

- membership;
- membership uniqueness;
- collection modification rules.

Creating:

```text id="p5x8qa"
CollectionMembershipService
```

would unnecessarily weaken the aggregate boundary.

---

# 7. Second Principle — Avoid Anemic Aggregates

A common anti-pattern is:

```text id="h3m9xw"
Collection
    |
    +-- data only

CollectionService
    |
    +-- all business logic
```

This effectively turns the aggregate into a data container.

CollectionHub should instead prefer:

```text id="v8q2jd"
Collection
    |
    +-- state
    +-- invariants
    +-- behavior
    +-- domain decisions
```

and only extract behavior when it genuinely has no natural owner.

---

# 8. Third Principle — Services Must Represent Domain Concepts

A Domain Service should have a meaningful domain name.

Prefer:

```text id="f7x4pm"
ItemCompatibilityPolicy
MetadataResolutionPolicy
CollectionEligibilityPolicy
```

over generic names such as:

```text id="m2r8vk"
DomainService
BusinessService
CommonService
HelperService
UtilityService
Manager
Processor
```

The name should communicate domain intent.

---

# 9. Candidate Domain Policies

Based on the current model, the following policies are potential candidates.

```text id="r8n4cy"
1. Item Compatibility Policy
2. Metadata Resolution Policy
3. External Information Reconciliation Policy
4. Classification Resolution Policy
```

These are deliberately treated as **candidates**, not final abstractions.

Each must earn its existence through an actual domain rule.

---

# 10. Item Compatibility Policy

## 10.1 Purpose

The Item Compatibility Policy determines whether an Item is compatible with a Collection under a rule that cannot naturally be owned by either aggregate.

Conceptually:

```text id="c7p5za"
Collection
     |
     +----+
          |
          v
 Item Compatibility Policy
          ^
          |
     Item
```

The policy evaluates facts from both aggregates.

---

## 10.2 Example Rule

A future rule might state:

> An item may only belong to a collection of a compatible type.

For example:

```text id="m4x7qn"
Collection Type
       +
Item Classification
       ↓
Compatibility Policy
       ↓
Allowed / Not Allowed
```

The important point is that neither aggregate necessarily owns the complete rule.

---

## 10.3 Ownership

The policy should not mutate either aggregate.

It should answer a domain question:

```text id="t5j9vc"
IsItemCompatibleWithCollection?
```

The eventual application workflow may then use that decision before invoking the Collection aggregate's membership behavior.

---

# 11. Compatibility Policy and Aggregate Integrity

The policy must not replace Collection's own invariants.

The correct sequence is:

```text id="y8k2sd"
Cross-aggregate compatibility
        ↓
Compatibility Policy
        ↓
allowed
        ↓
Collection.AddItem(...)
        ↓
Collection invariants
        ↓
state change
```

The Collection aggregate remains responsible for:

- membership uniqueness;
- membership validity;
- collection lifecycle;
- collection-level rules.

The policy is responsible only for the cross-aggregate decision.

---

# 12. Metadata Resolution Policy

CollectionHub may receive metadata from multiple sources.

A future domain rule may require determining which metadata value should become authoritative.

This is naturally modeled as a policy.

Conceptually:

```text id="q9w3ma"
Source A
Source B
Source C
   |
   v
Metadata Resolution Policy
   |
   v
Selected Domain Value
```

---

# 13. Metadata Resolution Is a Business Decision

The resolution policy should not simply mean:

```text first value wins
```

or:

```text latest HTTP response wins
```

Those are technical behaviors.

The domain policy should express semantic criteria such as:

- source authority;
- confidence;
- completeness;
- recency;
- explicit user override;
- provenance.

The exact weighting remains an open domain decision.

---

# 14. External Information Reconciliation Policy

When external information conflicts with existing domain state, CollectionHub needs a rule for reconciliation.

Example:

```text id="e5r7wk"
Existing Item
    |
    +-- Title = A

External Source
    |
    +-- Title = B

        ↓

Reconciliation Policy

        ↓

Decision
```

Possible outcomes:

```text id="k3v8ns"
Keep existing value
Accept external value
Create conflict
Require explicit review
Store both with provenance
```

The domain must eventually choose which outcomes are meaningful.

---

# 15. Classification Resolution Policy

If an Item receives multiple classifications from different sources, a policy may determine the canonical classification.

Conceptually:

```text id="z6p1cx"
Classification A
Classification B
Classification C
        |
        v
Classification Resolution Policy
        |
        v
Canonical Classification
```

Again, this should only become a real policy when the domain confirms that such a decision is required.

---

# 16. Candidate Domain Services

Current modeling does not yet require many Domain Services.

This is intentional.

Potential candidates include:

```text id="j7q4bw"
ItemRelationshipService
CollectionCompatibilityService
MetadataComparisonService
```

However, most of these should remain **hypotheses** until an actual domain operation demonstrates that a service is necessary.

---

# 17. Why Few Services Are Better

A large number of Domain Services often indicates one of two problems:

1. aggregate behavior is being extracted unnecessarily;
2. the domain model has not identified the correct concepts.

For example:

```text id="d8s2mv"
CollectionService
ItemService
MembershipService
MetadataService
ClassificationService
UserService
SourceService
```

is not evidence of good domain modeling.

It may simply represent CRUD-oriented thinking.

CollectionHub should resist this pattern.

---

# 18. Domain Service Candidate — Metadata Comparison

A metadata comparison operation may become a genuine Domain Service if comparison itself has domain meaning.

For example:

```text id="p4m9tx"
CompareMetadata
      |
      +-- ItemMetadata A
      +-- ItemMetadata B
      |
      v
Domain Result
```

The result could represent:

- equality;
- conflict;
- completeness;
- superiority;
- compatibility.

However, if comparison is merely a technical utility, it should not be a Domain Service.

---

# 19. Domain Service Candidate — Item Relationship Analysis

If CollectionHub eventually supports domain-level analysis across multiple Items, a service could become appropriate.

For example:

```text id="w3k7pq"
Items
  |
  v
Relationship Analysis
  |
  v
Domain Result
```

This is only a candidate.

No such service should be introduced until the business requirement exists.

---

# 20. Statelessness

Domain Services and Policies should normally be stateless.

They should not own mutable business state between invocations.

Conceptually:

```text id="n2v8cx"
Input Facts
     |
     v
Service / Policy
     |
     v
Decision / Result
```

not:

```text id="r7k4mb"
Service
   |
   +-- hidden mutable state
   +-- previous request
   +-- cached business decision
   +-- implicit workflow state
```

Persistent business state belongs to domain entities, aggregates, or explicitly modeled processes.

---

# 21. Services Must Not Become Repositories

A Domain Service must not hide persistence concerns.

Avoid:

```text id="x5c9jk"
Domain Service
    |
    +-- SQL
    +-- ORM
    +-- HTTP
    +-- database queries
```

If a domain decision needs information that is not already available, the architecture should provide that information through appropriate boundaries.

The Domain Service itself should remain focused on the domain decision.

---

# 22. Services Must Not Become API Clients

External communication is not inherently domain logic.

This is infrastructure behavior:

```text id="m6q2rz"
External API Client
```

The domain may define:

```text id="k8w4vc"
What external information means
```

but it should not define:

```text id="s1x7pd"
How HTTP requests are performed
```

---

# 23. Services Must Not Become Application Services

An Application Service coordinates a use case.

A Domain Service performs domain behavior.

For example:

```text id="q3v9la"
Application Service

    load Collection
    load Item
    invoke policy
    invoke aggregate
    persist changes
    publish events
```

Whereas:

```text id="h5m8cx"
Domain Policy

    evaluate compatibility
    return domain decision
```

These are different responsibilities.

---

# 24. Pure Domain Functions

Not every piece of domain logic needs a service object.

Some logic may be naturally represented as:

- value object behavior;
- entity behavior;
- aggregate behavior;
- pure domain calculation.

For example:

```text id="u6k2bp"
CollectionName.normalize(...)
```

should not automatically become:

```text id="f8r3nm"
CollectionNameNormalizationService
```

The abstraction should match the domain concept.

---

# 25. Policy Input Rules

A policy should receive explicit domain facts.

For example:

```text id="y4c7nd"
CollectionType
ItemClassification
PolicyContext
```

rather than:

```text id="r3m9qx"
DatabaseConnection
HttpRequest
UserInterfaceState
```

The latter belong to technical layers.

---

# 26. Policy Output Rules

Policies should return domain meaningful results.

Possible examples:

```text id="g7p2ka"
Compatible
NotCompatible
Preferred
Rejected
ConflictDetected
RequiresReview
```

These outcomes should be modeled according to actual domain language.

Avoid returning technical concepts such as:

```text id="z2w8mx"
HTTP 400
DatabaseException
Boolean flag because it's convenient
```

when richer domain semantics are required.

---

# 27. Policy Composition

Policies may eventually be composed.

For example:

```text id="v5r1cp"
Source Authority Policy
        +
Completeness Policy
        +
User Override Policy
        |
        v
Metadata Resolution Policy
```

However, composition should not create unnecessary abstraction layers.

The domain model should remain understandable in business terms.

---

# 28. Policy Determinism

Where a policy evaluates the same facts under the same contextual rules, it should produce the same result.

For example:

```text id="n8c3wy"
same facts
   +
same policy version
   +
same context
   =
same decision
```

If the decision depends on time or external state, that dependency must be explicit.

---

# 29. Temporal Policies

Some policies may depend on time.

For example:

```text id="c4x9qm"
Item availability at date X
Collection eligibility at date X
Metadata validity at date X
```

In these cases, time should be an explicit input.

Avoid hidden dependence on system clock behavior.

---

# 30. User Preferences vs Domain Policy

A user preference is not automatically a domain policy.

For example:

```text id="m7p4sz"
Preferred display order
Preferred image size
UI sorting
```

may belong to presentation or application concerns.

By contrast:

```text id="t8q1vn"
Preferred metadata source
```

could be domain-relevant if source precedence affects canonical domain truth.

The distinction depends on business meaning.

---

# 31. Configuration vs Policy

A configuration value becomes domain policy only when it affects business decisions.

For example:

```text id="b4r7xd"
Maximum image resolution
```

may be technical.

But:

```text id="w6m2qp"
Maximum number of items allowed in a collection
```

could be a business rule.

The domain meaning must be established before deciding where the value belongs.

---

# 32. Cross-Aggregate Decision Pattern

A cross-aggregate rule should follow this conceptual pattern:

```text id="s3v8ka"
             Aggregate A
                  |
                  |
                  v
             Domain Policy
                  ^
                  |
                  |
             Aggregate B
                  |
                  v
               Decision
```

The policy does not own either aggregate.

It evaluates facts and returns a domain decision.

---

# 33. Cross-Aggregate Mutation Pattern

A policy should generally **not directly mutate aggregates**.

Preferred:

```text id="x7q4mn"
Application Use Case
        |
        +---- load A
        +---- load B
        |
        +---- evaluate Policy
        |
        +---- invoke A
        |
        +---- invoke B if required
```

This keeps mutation responsibility explicit.

---

# 34. Avoid Hidden Aggregate Mutation

Avoid:

```text id="k5n9pw"
Policy
   |
   +-- modifies Collection
   +-- modifies Item
   +-- emits events
   +-- persists state
```

This turns a policy into an orchestration mechanism.

The policy should primarily answer a domain question.

---

# 35. Domain Decision vs Workflow

A domain decision:

```text id="d3m8qa"
"Is this item compatible?"
```

is different from a workflow:

```text id="p7x2vc"
Load item
Load collection
Check compatibility
Add membership
Save collection
Publish event
Notify user
```

The first is domain logic.

The second is application orchestration.

This distinction becomes critical in the next phase.

---

# 36. Error Semantics

Domain Services and Policies should communicate domain failure meaningfully.

Examples:

```text id="n6r4tp"
NotCompatible
ConflictDetected
InsufficientInformation
UnsupportedClassification
```

Avoid exposing infrastructure failures as domain decisions.

For example:

```text id="q8m2vx"
DatabaseTimeout
HTTP500
NetworkUnavailable
```

are not domain outcomes unless the domain explicitly models them.

---

# 37. Provenance and Policies

Provenance is particularly relevant to CollectionHub because external information may influence domain state.

A policy may need to consider:

```text id="u4p7kc"
Source
   +
Value
   +
Confidence
   +
Timestamp
   +
Existing Domain Value
```

to determine whether external information should be accepted.

The important principle is:

> **External data can inform a domain decision without becoming the domain model itself.**

---

# 38. External Source Trust

A future Source Trust Policy may determine:

```text id="r3w8nm"
Source A > Source B
```

for a particular type of information.

However, source trust should not be generalized blindly.

A source may be authoritative for:

```text id="x7q1pd"
Title
```

but not necessarily for:

```text id="m4c9vk"
Classification
```

Therefore trust may be contextual.

---

# 39. Conflict Resolution

When conflicting domain facts exist, the policy should make the conflict explicit.

Avoid:

```text id="g5n8qa"
if A != B:
    use B
```

without domain justification.

Prefer a semantic model such as:

```text id="z6p3rm"
Conflict
    |
    +-- ExistingValue
    +-- CandidateValue
    +-- Provenance
    +-- ResolutionReason
```

if the domain requires conflict visibility.

---

# 40. Policy Versioning

If a policy's rules change over time and historical decisions must remain reproducible, policy version may become domain-significant.

For example:

```text id="h2v7cx"
Policy v1
Policy v2
Policy v3
```

This is not required by default.

It becomes relevant only if historical reproducibility is a business requirement.

---

# 41. Candidate Policy Registry

Current candidates:

| Candidate | Status | Reason |
|---|---|---|
| ItemCompatibilityPolicy | Candidate | Potential cross-aggregate rule |
| MetadataResolutionPolicy | Candidate | Multiple metadata sources |
| ExternalInformationReconciliationPolicy | Candidate | Conflicting external data |
| ClassificationResolutionPolicy | Candidate | Multiple classifications |
| SourceTrustPolicy | Candidate | Source-specific authority |
| CollectionEligibilityPolicy | Candidate | Potential collection-specific eligibility |

These are not implementation requirements.

They are domain hypotheses requiring validation.

---

# 42. Candidate Domain Service Registry

Current candidates:

| Candidate | Status | Reason |
|---|---|---|
| MetadataComparisonService | Candidate | Potential domain-level comparison |
| ItemRelationshipAnalysisService | Candidate | Potential future analysis |
| CrossCollectionAnalysisService | Candidate | Only if analysis becomes domain behavior |

The intentionally small number of candidates is a positive result.

It indicates that most currently identified behavior has a natural owner in the aggregates.

---

# 43. Services Explicitly Rejected

The following should **not** be introduced as Domain Services merely for architectural symmetry:

```text id="q8c5xm"
CollectionService
ItemService
MembershipService
UserService
RepositoryService
PersistenceService
NotificationService
SearchService
ImportService
ApiService
ValidationService
```

Each of these may represent an application or infrastructure concern instead.

The existence of a noun does not automatically justify a Domain Service.

---

# 44. Domain Service Anti-Pattern — CRUD Wrappers

Avoid:

```text id="w7m3pq"
CollectionService.create(...)
CollectionService.update(...)
CollectionService.delete(...)
```

if those methods merely delegate to persistence.

This is application or infrastructure behavior, not Domain Service behavior.

---

# 45. Domain Service Anti-Pattern — Aggregate Proxy

Avoid:

```text id="c6r9vx"
ItemService.updateItem(...)
```

when the method simply forwards the request to:

```text id="p2x8mn"
item.update(...)
```

The aggregate should expose its own domain behavior.

---

# 46. Domain Service Anti-Pattern — Hidden Workflow

Avoid:

```text id="n4k7sp"
MetadataService
    |
    +-- load item
    +-- call API
    +-- modify item
    +-- save item
    +-- send event
```

This is an application workflow.

It should not be disguised as domain logic.

---

# 47. Domain Service Anti-Pattern — Infrastructure Leakage

Avoid dependencies on:

- ORM;
- HTTP clients;
- message brokers;
- filesystem;
- environment variables;
- database connections;
- framework request objects.

Domain logic should remain independent from those mechanisms.

---

# 48. Aggregate vs Policy Decision Matrix

| Question | Aggregate | Policy |
|---|---|---|
| Owns state? | Yes | No |
| Owns identity? | Root does | No |
| Protects local invariants? | Yes | No |
| Makes cross-concept decision? | Sometimes | Yes |
| Persists state? | Conceptually yes | No |
| Represents business rule? | Yes | Yes |
| Owns lifecycle? | Yes | No |
| Coordinates multiple aggregates? | No | Can evaluate them |
| Produces domain state change? | Yes | Normally no |

---

# 49. Aggregate vs Domain Service Decision Matrix

| Question | Aggregate | Domain Service |
|---|---|---|
| Has identity? | Yes | No |
| Owns persistent state? | Yes | No |
| Protects invariants? | Yes | Sometimes indirectly |
| Represents domain noun? | Yes | Usually domain operation |
| Has independent lifecycle? | Yes | No |
| Can coordinate multiple aggregates? | No | Yes |
| Should mutate aggregate directly? | Itself | Normally no |
| Owns domain state? | Yes | No |

---

# 50. Decision Tree

When discovering new domain behavior, use:

```text id="v3m8rq"
Does one aggregate naturally own it?
        |
       Yes
        |
        v
Put behavior in aggregate
        |
       No
        |
        v
Is it primarily a business decision?
        |
       Yes
        |
        v
Consider Domain Policy
        |
       No
        |
        v
Does it represent meaningful domain behavior
across multiple concepts?
        |
       Yes
        |
        v
Consider Domain Service
        |
       No
        |
        v
Reconsider whether it belongs in the domain
```

---

# 51. Domain Service Dependencies

A Domain Service may depend conceptually on domain abstractions, but those dependencies must not force infrastructure concerns into the domain.

For example:

```text id="x4n7cp"
MetadataResolutionPolicy
       |
       +-- SourceMetadata
       +-- ItemMetadata
       +-- Provenance
```

is acceptable.

Whereas:

```text id="j8q2mz"
MetadataResolutionPolicy
       |
       +-- HttpClient
       +-- SqlConnection
       +-- ORMContext
```

is not.

---

# 52. Domain Policies and Pure Functions

Many policies may be naturally modeled as deterministic domain logic.

For example:

```text id="c5r9vk"
CompatibilityPolicy.evaluate(
    collectionType,
    itemClassification
)
```

Conceptually:

```text id="n7m2qa"
Facts
  ↓
Policy
  ↓
Decision
```

This simplicity should be preserved whenever possible.

---

# 53. Policy Inputs Should Be Explicit

Do not let policies retrieve hidden context.

Avoid:

```text id="p4x8mw"
policy.evaluate()
```

when the decision actually depends on hidden state.

Prefer conceptually:

```text id="r6q3zn"
policy.evaluate(
    collection,
    item,
    context
)
```

or, preferably, the minimal domain facts required:

```text id="s2k7vc"
policy.evaluate(
    collectionType,
    itemClassification
)
```

Explicit inputs improve reasoning and testability.

---

# 54. Minimize Policy Knowledge

A policy should know only what it needs to make its decision.

For example, an Item Compatibility Policy should not need to know:

- database identifiers;
- HTTP requests;
- UI state;
- persistence metadata.

This reduces coupling.

---

# 55. Domain Policy and Authorization

Authorization should not automatically be modeled as a Domain Policy.

Questions such as:

```text id="q8w4xm"
"Is this user allowed to access this collection?"
```

may involve application security rather than pure domain semantics.

However, if the domain explicitly models ownership or rights as business concepts, some authorization-like rules may become domain rules.

The distinction must be established during the ownership and access-control modeling phase.

---

# 56. Domain Policy and User Preferences

Similarly, user preferences should remain outside the domain unless they materially change a business decision.

This prevents accidental coupling between:

```text id="t6m9pk"
Business semantics
```

and:

```text id="x3r7vc"
Presentation preferences
```

---

# 57. Policy and External Data Freshness

If freshness affects domain decisions, it should be explicit.

For example:

```text id="f8q2nw"
SourceValue
    +
ObservedAt
    +
CurrentDomainValue
```

may allow a policy to determine whether the source information is still relevant.

This is preferable to hidden reliance on network timing.

---

# 58. Policy and Confidence

If external sources provide confidence scores, the domain should not blindly expose technical scores.

The domain should define what confidence means.

For example:

```text id="m5r8cx"
HighConfidence
MediumConfidence
LowConfidence
```

may be more meaningful than:

```text id="k7v2qa"
0.82
0.54
0.12
```

unless numerical confidence itself is part of the domain language.

---

# 59. Policy and Human Review

If some decisions cannot be automated confidently, the domain may model:

```text id="n4x7ps"
RequiresReview
```

rather than forcing a false binary decision.

This is particularly relevant to metadata reconciliation and classification.

---

# 60. Domain Service and Event Generation

A Domain Service does not automatically generate domain events.

Events should normally be generated by the aggregate whose state changed.

For example:

```text id="r5k8vc"
Compatibility Policy
       |
       v
Allowed
       |
       v
Collection.addItem(...)
       |
       v
ItemAddedToCollection
```

The policy decides.

The aggregate records the resulting domain fact.

---

# 61. Policy and Side Effects

Domain Policies should not perform external side effects.

Avoid:

```text id="x7m3qn"
Policy
   |
   +-- send email
   +-- call API
   +-- save database
   +-- publish message
```

A policy should remain a domain decision mechanism.

Side effects belong to later layers.

---

# 62. Domain Services and Determinism

Whenever possible:

```text id="g2p9wx"
same domain facts
      +
same policy
      =
same result
```

This makes the domain easier to reason about and test.

Non-determinism should be introduced only when the domain genuinely requires it.

---

# 63. Domain Service Testability

A well-designed Domain Service should be testable using domain concepts alone.

A conceptual test should be able to express:

```text id="v8q4mc"
Given:
    CollectionType = X
    ItemClassification = Y

When:
    compatibility is evaluated

Then:
    result = Compatible
```

without requiring:

- database;
- HTTP;
- framework;
- filesystem;
- message broker.

---

# 64. Policy Testability

Policies should similarly be testable as domain decisions.

Examples:

```text id="n6k3rp"
Given source A is authoritative
And source B is less authoritative

When values conflict

Then A is preferred
```

This directly expresses business language.

---

# 65. Domain Rule Traceability

Every Domain Service or Policy should eventually trace back to an identified rule.

For example:

```text id="m5x8qa"
BR-PROVENANCE-003
        ↓
External information conflict
        ↓
Reconciliation requirement
        ↓
ExternalInformationReconciliationPolicy
```

If no business rule requires the service, its existence should be questioned.

---

# 66. Current Recommended Domain Model

At the current stage, the preferred model is:

```text id="j4n7vc"
+-----------------------+
| Collection Aggregate  |
|                       |
| Collection behavior   |
| Membership rules      |
+-----------+-----------+
            |
            | ItemId
            v
+-----------------------+
| Item Aggregate        |
|                       |
| Item behavior         |
| Metadata              |
| Classification        |
+-----------+-----------+
            |
            |
            v
+-------------------------------+
| Candidate Domain Policies     |
|                               |
| Compatibility                 |
| Metadata Resolution           |
| Reconciliation                |
| Classification Resolution     |
+-------------------------------+
```

The policies remain outside aggregate state.

---

# 67. Current Domain Service Position

At present, CollectionHub does **not require a large Domain Service layer**.

This is a deliberate modeling conclusion.

The majority of currently identified behavior belongs naturally to:

- Collection;
- Item;
- Value Objects.

Domain Services should be introduced only when additional domain analysis demonstrates behavior that cannot reasonably belong there.

---

# 68. Open Questions

## OPEN-DS-001

What concrete cross-aggregate business rules actually exist between Collection and Item?

---

## OPEN-DS-002

Is compatibility between Item and Collection genuinely a business rule or merely a future possibility?

---

## OPEN-DS-003

Does external metadata reconciliation belong in the core domain or in an integration/application boundary?

---

## OPEN-DS-004

Are external sources themselves domain concepts?

---

## OPEN-DS-005

Is classification resolution a business decision or merely an import concern?

---

## OPEN-DS-006

Does CollectionHub require human review of conflicting metadata?

---

## OPEN-DS-007

Which user preferences, if any, materially affect business decisions?

---

## OPEN-DS-008

Does ownership/access control belong to the domain model or primarily to application/security layers?

---

# 69. Anti-Patterns Summary

CollectionHub should explicitly avoid:

```text id="r8m2vc"
Generic Services
       ↓
CRUD wrappers
       ↓
Anemic Aggregates
       ↓
Infrastructure leakage
       ↓
Hidden workflows
       ↓
Unclear domain ownership
```

Instead:

```text id="x6p9qa"
Domain Concept
       ↓
Correct Owner
       ↓
Explicit Behavior
       ↓
Explicit Policy where necessary
       ↓
Explicit Application Coordination
```

---

# 70. Relationship With Previous Documents

The dependency chain is now:

```text id="n5q8kc"
Domain Concepts
      ↓
Entities / Value Objects
      ↓
Commands / Events
      ↓
Invariants / Business Rules
      ↓
Aggregates / Consistency Boundaries
      ↓
Domain Services / Policies
```

Each layer answers a different question.

### Commands

> What does someone want to happen?

### Events

> What happened?

### Invariants

> What must always remain true?

### Aggregates

> Who protects those truths?

### Policies

> How should a domain decision be made when it does not belong naturally to one aggregate?

### Services

> What meaningful domain behavior requires coordination beyond one aggregate?

---

# 71. Relationship With the Next Phase

The next phase should move from **domain behavior** to **application orchestration**.

The application layer will answer:

> **How does a complete use case coordinate commands, aggregates, policies, repositories, and domain events?**

This is fundamentally different from the questions answered here.

The application layer should not reinvent domain rules.

Instead:

```text id="f3k7xp"
Application Use Case
        |
        +---- load aggregate
        |
        +---- invoke policy
        |
        +---- invoke aggregate
        |
        +---- persist result
        |
        +---- publish domain facts
```

The domain remains the source of business meaning.

---

# 72. Completion Criteria

This document is considered sufficiently mature when:

- aggregate-owned behavior is clearly distinguished from external behavior;
- candidate policies are identified;
- candidate Domain Services are identified;
- generic service abstractions are rejected;
- cross-aggregate decisions are explicitly recognized;
- infrastructure responsibilities are excluded;
- policy inputs and outputs are domain-oriented;
- domain rules have traceability to services or policies;
- unresolved questions are recorded;
- the domain layer remains intentionally small.

---

# 73. Final Principle

The central principle of CollectionHub is:

> **Do not create a Domain Service because logic exists. Create one only when the domain behavior has no natural owner within the existing model.**

Likewise:

> **Do not create a Policy because a conditional exists. Create one when the conditional represents a meaningful business decision that should be explicit and reusable.**

The resulting hierarchy should remain:

```text id="p8x4nm"
Aggregate owns its state
        ↓
Aggregate protects its invariants
        ↓
Policy decides cross-concept rules
        ↓
Domain Service performs domain behavior
        ↓
Application layer orchestrates the use case
        ↓
Infrastructure performs technical work
```

This keeps CollectionHub's domain model cohesive, explicit, and resistant to the common failure mode where all business logic eventually collapses into generic service classes.

The next step is therefore to define the **Application Use Cases and their orchestration boundaries**, building directly on the commands, aggregates, invariants, policies, and services established in the previous documents.