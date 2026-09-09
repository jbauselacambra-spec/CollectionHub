Project: CollectionHub
Phase: 2.2 — Domain Modeling
Artifact: 20 — Domain Model Decisions and Open Questions
Status: Decision Register / Living Document
Purpose: Consolidate, justify and govern the domain-model decisions emerging from the previous modeling artifacts.

1. Purpose

This document is the decision register for the CollectionHub domain model.

Its purpose is to distinguish clearly between:

decisions that are already sufficiently established;
decisions that require explicit architectural commitment;
questions that remain intentionally open;
alternatives that have been considered and rejected;
questions that should not yet be decided because the domain evidence is insufficient.

This document prevents an important modeling failure:

mistaking an assumption for a decision.

The existence of a concept in an earlier document does not automatically mean that its final structural representation has been decided.

2. Decision Philosophy

CollectionHub follows the principle:

Model first, decide explicitly, implement last.

A decision should not be made merely because:

a framework encourages it;
a database makes it convenient;
a common DDD pattern exists;
the implementation would be easier;
or the current code structure suggests it.

A domain decision must be justified by business meaning.

3. Decision Statuses

Each decision uses one of the following statuses:

Status	Meaning
ACCEPTED	Decision is sufficiently established
PROVISIONAL	Current preferred option, subject to evidence
OPEN	No option has been selected
DEFERRED	Intentionally postponed until a later phase
REJECTED	Option explicitly rejected
SUPERSEDED	Previous decision replaced by a newer one
4. Decision Confidence

Each decision also receives a confidence level:

Confidence	Meaning
HIGH	Strongly supported by domain evidence
MEDIUM	Reasonable interpretation, but some uncertainty remains
LOW	Working hypothesis requiring validation

A PROVISIONAL decision with HIGH confidence is still not an implementation freeze.

5. Decision Record Format

Each significant decision follows this structure:

Decision
Context
Options
Chosen Option
Rationale
Consequences
Related Artifacts
Status
Confidence

This provides traceability and prevents decisions from becoming undocumented assumptions.

6. Decision D-001 — Domain-Centric Modeling
Context

CollectionHub is being modeled before implementation.

Decision

The domain model is the primary source of business semantics.

Application, infrastructure and persistence concerns must not define domain concepts.

Rationale

This preserves:

business clarity;
testability;
replaceable infrastructure;
explicit business rules;
architectural independence.
Consequences

The future implementation must adapt technical concerns to the domain rather than the reverse.

Status: ACCEPTED
Confidence: HIGH

7. Decision D-002 — Aggregates Own Consistency
Decision

Aggregate boundaries define the primary boundaries for immediate domain consistency.

Aggregates are responsible for protecting their own invariants.

Consequence

Specifications and Policies may evaluate conditions, but they cannot replace Aggregate invariant enforcement.

Related
08_DOMAIN_INVARIANTS.md
09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md
18_DOMAIN_SPECIFICATIONS_AND_REUSABLE_RULES.md
19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md

Status: ACCEPTED
Confidence: HIGH

8. Decision D-003 — Specifications Are Predicates
Decision

Specifications represent reusable domain predicates.

They should remain:

deterministic;
side-effect free;
composable;
domain-oriented.
Consequence

Specifications must not become disguised:

Domain Services;
Application Services;
repositories;
workflows.

Status: ACCEPTED
Confidence: HIGH

9. Decision D-004 — Policies Own Business Decisions
Decision

Policies represent meaningful business decisions rather than low-level validation.

A Policy may compose multiple Specifications and domain facts.

Conceptually:

Specifications
      ↓
Policy
      ↓
Decision

Status: ACCEPTED
Confidence: HIGH

10. Decision D-005 — Domain Services Are Exceptional
Decision

Domain Services are introduced only when behavior:

is genuinely domain behavior;
does not naturally belong to one Aggregate;
spans multiple domain concepts;
and cannot be expressed more clearly as a Policy or Specification.
Consequence

The presence of multiple Aggregates alone is not sufficient justification for a Domain Service.

Status: ACCEPTED
Confidence: HIGH

11. Decision D-006 — Domain Events Represent Facts
Decision

Domain Events represent facts that have happened in the domain.

Preferred:

CollectionArchived
ItemAddedToCollection
ItemRemovedFromCollection

Not:

ArchiveCollection
AddItemToCollection
RemoveItem
Consequence

Events should not be confused with commands.

Status: ACCEPTED
Confidence: HIGH

12. Decision D-007 — Application Layer Coordinates
Decision

Application Services and Workflows coordinate use cases.

They should not become the primary owners of domain rules.

The expected structure is:

Application
    ↓
Domain decision
    ↓
Domain behavior
    ↓
Persistence / publication

Status: ACCEPTED
Confidence: HIGH

13. Decision D-008 — Infrastructure Does Not Define the Domain
Decision

Infrastructure concerns must not determine domain semantics.

Examples:

HTTP
SQL
ORM
message broker
cache
cloud provider

must remain outside the domain model.

Status: ACCEPTED
Confidence: HIGH

14. Decision D-009 — Collection Is a Core Aggregate Candidate
Decision

Collection is currently modeled as a primary Aggregate.

It owns, at minimum:

Collection identity;
Collection lifecycle;
collection-level invariants;
membership state where membership belongs to the Collection boundary.

Status: PROVISIONAL
Confidence: HIGH

The remaining question is the exact representation of Membership.

15. Decision D-010 — Item Is Independently Modeled
Decision

Item is currently treated as an independently identifiable domain concept with its own lifecycle and invariants.

It should not automatically become part of the Collection Aggregate merely because Collections contain Items.

Status: PROVISIONAL
Confidence: MEDIUM

Further domain validation may modify the final boundary.

16. Decision D-011 — Membership Requires Explicit Modeling Decision
Context

Membership is central to Collection behavior.

Several models are possible.

Option A — Membership as Entity
Collection
    └── Membership
Option B — Membership as Value-like association
Collection
    └── ItemId
Option C — Membership as Aggregate
Membership
    ├── CollectionId
    └── ItemId
Current Decision

Do not freeze the representation yet.

The determining questions are:

Does Membership have independent identity?
Does it have lifecycle?
Does it have meaningful attributes?
Does it have behavior?
Does it have independent invariants?
Must it be modified independently?
Does it need independent concurrency?

Status: OPEN
Confidence: HIGH that the question is important.

17. Decision D-012 — Membership Invariants Remain Collection-Protected Until Proven Otherwise

Even though Membership's final representation is open, the following invariant remains conceptually associated with Collection consistency:

A Collection cannot contain the same logical Item more than once.

Therefore, until the boundary is changed explicitly, the Collection remains the provisional consistency owner.

Status: PROVISIONAL
Confidence: HIGH

18. Decision D-013 — External Identity Is a Domain Concept

External Identity is treated as domain-relevant when it participates in:

identity resolution;
synchronization;
matching;
uniqueness;
conflict resolution.

It must therefore not be modeled merely as an infrastructure DTO.

Status: ACCEPTED
Confidence: HIGH

19. Decision D-014 — External Identity Ownership Remains Open

Possible ownership models include:

Item
  └── ExternalIdentity

or:

ExternalIdentity
    ↓
Item

or a dedicated identity mapping model.

The correct decision depends on:

cardinality;
lifecycle;
uniqueness;
reassignment;
conflict behavior;
synchronization requirements.

Status: OPEN
Confidence: HIGH

20. Decision D-015 — Ownership Is a Domain Concept

Ownership should not be inferred solely from technical authentication.

The domain distinction is:

Actor
    ≠
Authenticated technical principal
    ≠
Owner
    ≠
Authorized collaborator

These concepts must not be collapsed without explicit justification.

Status: ACCEPTED
Confidence: HIGH

21. Decision D-016 — Authorization Is Partially Domain-Relevant

Some authorization rules are domain rules.

Examples:

ActorOwnsCollection
ActorCanModifyCollection
ActorCanArchiveCollection

Technical authentication remains outside the domain.

Status: ACCEPTED
Confidence: HIGH

22. Decision D-017 — Ownership Model Remains Open

Questions:

Can a Collection have exactly one owner?
Can ownership be shared?
Can ownership be transferred?
Is ownership persistent domain state?
Can ownership be delegated?
Are collaborators distinct from owners?
Are ownership changes audited?

Until these are resolved, ownership should remain conceptually modeled but structurally flexible.

Status: OPEN

23. Decision D-018 — Lifecycle Is Explicit Domain State

Lifecycle states must be represented explicitly rather than inferred from arbitrary flags.

Avoid models such as:

isActive
isArchived
isDeleted

if they can create contradictory combinations.

Prefer a coherent lifecycle model.

Status: ACCEPTED
Confidence: HIGH

24. Decision D-019 — Lifecycle Transitions Are Domain Behavior

A lifecycle transition should be expressed through domain behavior.

Conceptually:

Collection.archive()
Collection.reactivate()
Collection.delete()

rather than arbitrary state mutation.

Status: ACCEPTED

25. Decision D-020 — Lifecycle Completeness Remains Open

The final lifecycle requires confirmation of:

initial state;
active state;
archived state;
deleted state;
restoration;
irreversible transitions;
invalid transitions.

The current model contains candidates but should not assume completeness.

Status: OPEN

26. Decision D-021 — Strong Consistency Is Local to Aggregates

Immediate consistency should normally be guaranteed inside an Aggregate.

Cross-Aggregate consistency should not automatically become a distributed transaction.

Status: ACCEPTED

27. Decision D-022 — Cross-Aggregate Consistency Requires Explicit Classification

Each cross-Aggregate rule must be classified as:

Strong / immediate
Eventual
Compensating
Constraint-based

No cross-Aggregate dependency should remain implicitly transactional.

Status: ACCEPTED

28. Decision D-023 — Search / Projection Side Effects Are Eventual

Where read models, search indexes or similar projections are involved, eventual consistency is the default unless the business explicitly requires otherwise.

Conceptually:

Domain Event
      ↓
Projection Update

rather than:

Aggregate Transaction
      ↓
Search Index Transaction

Status: PROVISIONAL

29. Decision D-024 — Domain Events Are Not Integration Events

A Domain Event represents an internal domain fact.

An Integration Event represents a message intended for an external boundary.

They may eventually be derived from one another, but they should not be treated as identical concepts by default.

Status: ACCEPTED

30. Decision D-025 — Side Effects Must Have Explicit Triggers

Every meaningful side effect must be traceable to:

a Domain Event;
an Application Workflow;
or another explicitly modeled trigger.

Hidden side effects are prohibited.

Status: ACCEPTED

31. Decision D-026 — Aggregate Invariants Cannot Depend on Pre-Validation

The model explicitly rejects:

Specification
    ↓
"looks valid"
    ↓
Aggregate assumes it is valid

Instead:

Specification / Policy
    ↓
Decision
    ↓
Aggregate
    ↓
Invariant enforcement

The Aggregate remains authoritative.

Status: ACCEPTED

32. Decision D-027 — Idempotency Must Be Explicit

Each externally meaningful command/use case should eventually document whether repeated execution is:

idempotent;
rejected;
tolerated;
or semantically different.

This is particularly important for:

Add Item
Remove Item
Archive Collection
Synchronize Item
Import External Identity

Status: ACCEPTED

33. Decision D-028 — Domain Time Must Be Explicit

If a rule depends on time, the evaluation time should be explicit.

Conceptually:

Rule.evaluate(state, evaluationTime)

rather than silently reading system time.

Status: ACCEPTED

34. Decision D-029 — Technical IDs Are Not Automatically Domain Identities

The model distinguishes:

Domain Identity
External Identity
Persistence Identity

A database-generated identifier must not automatically become the business identity merely because it is convenient.

Status: ACCEPTED

35. Decision D-030 — Classification Ownership Remains Open

Classification appears to be domain-relevant, but its exact ownership remains to be confirmed.

Possible models:

Item
    └── Classification

or:

Classification
    └── independent concept

or a relationship between Item and Classification.

Questions include:

Can an Item have multiple classifications?
Are classifications reusable?
Can classifications evolve independently?
Are classifications user-defined?
Do classifications have lifecycle?
Can classification changes trigger events?

Status: OPEN

36. Decision D-031 — Synchronization Is a Domain Concern When It Changes Meaningful State

Synchronization is domain-relevant when it involves:

external identity;
matching;
conflict;
reconciliation;
domain state changes.

Pure technical scheduling is not domain behavior.

Status: ACCEPTED

37. Decision D-032 — Conflict Resolution Is Separate From Synchronization Transport

The domain should model:

Conflict
    ↓
Resolution decision

independently from:

HTTP
API client
scheduler
retry mechanism

Status: ACCEPTED

38. Decision D-033 — Automatic Conflict Resolution Requires Explicit Eligibility

A synchronization conflict should only be resolved automatically when a domain rule explicitly permits it.

Conceptually:

Conflict
   ↓
SynchronizationCanBeResolvedAutomatically
   ↓
Policy
   ↓
Resolution

Status: ACCEPTED

39. Decision D-034 — No Generic "God Policy"

The model rejects policies such as:

CollectionPolicy
ItemPolicy
DomainPolicy
BusinessRulesPolicy

when they become containers for unrelated decisions.

Policies should represent meaningful business decisions.

Status: ACCEPTED

40. Decision D-035 — No Generic Domain Service

The model rejects a catch-all service such as:

CollectionDomainService
DomainService
BusinessService

without a precise domain responsibility.

Status: ACCEPTED

41. Decision D-036 — No Generic Specification Base Without Semantic Value

A technical Specification abstraction may exist internally, but domain concepts should remain explicit.

For example:

CollectionIsMutable

is more important than:

BooleanSpecification

The abstraction must support domain expression, not replace it.

Status: ACCEPTED

42. Decision D-037 — No Premature Persistence Model

The domain model must not be shaped around:

relational tables;
ORM entities;
document structures;
indexes;
foreign keys.

Persistence design will follow domain decisions.

Status: ACCEPTED

43. Decision D-038 — Domain Model Must Be Framework-Agnostic

The domain model should not depend on:

web frameworks;
persistence frameworks;
messaging frameworks;
dependency injection containers.

Status: ACCEPTED

44. Open Question Q-001 — Definitive Aggregate Map

The final Aggregate inventory still requires confirmation.

Candidate model:

Collection
Item
Membership?
ExternalIdentity?
Classification?
Synchronization?

The final answer must be based on:

invariants;
lifecycle;
identity;
transactional consistency;
independent behavior.

Status: OPEN

45. Open Question Q-002 — Membership Semantics

Determine:

Is Membership an Entity?
Is Membership a Value Object?
Is Membership an Aggregate?

Required evidence:

identity;
attributes;
lifecycle;
behavior;
concurrency;
independent operations.

Status: OPEN

46. Open Question Q-003 — Membership Attributes

Determine whether Membership has meaningful attributes such as:

position
addedAt
metadata
source
status
notes
classification

If it does, Membership may require richer modeling.

Status: OPEN

47. Open Question Q-004 — Ownership Cardinality

Determine whether:

Collection → Owner

is:

1 : 1
1 : N
N : N

and whether ownership can change.

Status: OPEN

48. Open Question Q-005 — Collaborator Model

Determine whether CollectionHub requires:

Owner
Collaborator
Viewer
Administrator

or another permission model.

Do not introduce roles merely because they are technically convenient.

Status: OPEN

49. Open Question Q-006 — Classification Model

Determine:

cardinality;
ownership;
lifecycle;
mutability;
uniqueness;
compatibility rules.

Status: OPEN

50. Open Question Q-007 — External Identity Cardinality

Determine:

Item → ExternalIdentity

relationship.

Possible:

1 : 1
1 : N
N : 1
N : N

This directly affects Aggregate design.

Status: OPEN

51. Open Question Q-008 — External Identity Reassignment

Determine whether an External Identity can move from one Item to another.

If yes, this becomes a major invariant and conflict rule.

Status: OPEN

52. Open Question Q-009 — Synchronization Ownership

Determine which model owns synchronization state:

Item
ExternalIdentity
Dedicated Synchronization Aggregate
Application process

The answer depends on whether synchronization state has domain significance or is merely technical process state.

Status: OPEN

53. Open Question Q-010 — Collection Lifecycle

Finalize:

Initial state
Active state
Archived state
Deleted state
Restoration
Irreversible transitions

Status: OPEN

54. Open Question Q-011 — Deletion Semantics

Determine whether deletion means:

Soft deletion
Hard deletion
Archival
Tombstone
Lifecycle state

and whether deleted Collections remain addressable.

Status: OPEN

55. Open Question Q-012 — Ordering Semantics

If Collection membership has ordering, determine whether order is:

domain-significant;
user-controlled;
stable;
mutable;
concurrent;
externally synchronized.

If ordering is not domain-significant, it should not complicate the Aggregate.

Status: OPEN

56. Open Question Q-013 — Capacity Semantics

Determine whether Collection capacity exists.

If it does:

what defines capacity?
is it configurable?
can it change?
is it a hard invariant?
is capacity exceeded a domain decision?

If not, CollectionHasCapacity should be removed from the final model.

Status: OPEN

57. Open Question Q-014 — Item Eligibility

Determine the definitive conditions for:

ItemIsEligible

The rule must not remain a placeholder.

Status: OPEN

58. Open Question Q-015 — Duplicate Semantics

Determine what constitutes the same Item.

Possibilities include:

Domain ID
External Identity
Canonical identity
Composite identity

This directly determines the duplicate-membership invariant.

Status: OPEN

59. Open Question Q-016 — Synchronization Conflict Taxonomy

Determine the conflict categories supported by the domain.

For example:

Identity conflict
State conflict
Classification conflict
Deletion conflict
Version conflict
Mapping conflict

The actual taxonomy must be derived from business behavior.

Status: OPEN

60. Open Question Q-017 — Automatic Resolution Rules

For each conflict category determine:

Can resolve automatically?
Can resolve deterministically?
Requires manual review?
Requires external authority?

Status: OPEN

61. Open Question Q-018 — Event Catalog Completeness

The current event catalog must eventually be checked against every meaningful state transition.

Questions:

Which transitions emit events?
Which are intentionally silent?
Which events are internal only?
Which become integration messages?

Status: OPEN

62. Open Question Q-019 — Side Effect Guarantees

For each side effect determine:

At-most-once?
At-least-once?
Effectively-once?
Retryable?
Idempotent?

This will become especially important during application architecture.

Status: OPEN

63. Open Question Q-020 — Audit Requirements

Determine whether CollectionHub requires explicit domain auditing for:

Ownership changes
Membership changes
Lifecycle changes
Classification changes
Synchronization resolutions
External identity changes

If auditing is business-significant, it may influence events and state modeling.

Status: OPEN

64. Open Question Q-021 — Multi-Tenancy

Determine whether CollectionHub has a tenant boundary.

If yes:

Is Tenant a domain concept?
Is it an Aggregate?
Is it a consistency boundary?
Does every Collection belong to exactly one Tenant?
Can Actors belong to multiple Tenants?

Do not introduce Tenant into the domain model unless the business actually requires it.

Status: OPEN

65. Open Question Q-022 — Permission Model

Determine whether permissions are:

Ownership-based
Role-based
Capability-based
Relationship-based
Hybrid

This should be resolved from domain requirements, not framework conventions.

Status: OPEN

66. Open Question Q-023 — Temporal Rules

Determine whether any domain rule depends on:

deadlines;
expiration;
scheduled transitions;
time windows;
temporal ownership;
synchronization timestamps.

If yes, these rules require explicit temporal modeling.

Status: OPEN

67. Open Question Q-024 — Concurrency Requirements

Determine which operations are sensitive to concurrent modification.

At minimum review:

Add Item
Remove Item
Reorder Membership
Ownership Change
Classification Change
Synchronization
Conflict Resolution

Status: OPEN

68. Deferred Decisions

Some decisions should intentionally remain deferred.

Deferred D-001 — Database technology

Reason:

Persistence design should follow domain boundaries.

Deferred D-002 — ORM strategy

Reason:

ORM structure must not determine Aggregate structure.

Deferred D-003 — Messaging technology

Reason:

Event semantics should be finalized before transport.

Deferred D-004 — API contract

Reason:

External API should expose application capabilities, not define domain semantics.

Deferred D-005 — Deployment topology

Reason:

Deployment architecture should follow consistency and integration requirements.

69. Rejected Alternatives

The following approaches are explicitly rejected unless future evidence overturns them.

R-001 — Anemic Domain Model

Rejected because it moves business behavior into application services.

R-002 — Database-First Domain Modeling

Rejected because database structure would become the accidental domain model.

R-003 — One Aggregate Per Database Table

Rejected because persistence boundaries and domain consistency boundaries are different concepts.

R-004 — Generic Domain Service

Rejected because it obscures business responsibility.

R-005 — Generic Policy Container

Rejected because unrelated business decisions become coupled.

R-006 — Specification as Validation Framework

Rejected because Specifications are domain predicates, not generic validation infrastructure.

R-007 — Domain Event for Every Method Call

Rejected because not every technical action represents a meaningful business fact.

R-008 — Distributed Transaction Across All Aggregates

Rejected as the default consistency strategy.

R-009 — Application Layer as Rule Owner

Rejected because business invariants would become scattered.

70. Decision Dependency Graph

Several decisions depend on others.

Membership Semantics
        ↓
Collection Aggregate Boundary
        ↓
Consistency Strategy
        ↓
Domain Services
        ↓
Use Case Workflows

Similarly:

External Identity Model
        ↓
Item Identity
        ↓
Duplicate Semantics
        ↓
Membership Invariants
        ↓
Synchronization Rules

And:

Ownership Model
        ↓
Authorization Model
        ↓
Modification Policies
        ↓
Use Case Permissions

Therefore, not every open question should be resolved independently.

71. Decision Priority

Open questions are classified by impact.

Priority	Meaning
P0	Blocks coherent domain model
P1	Strongly affects architecture
P2	Important but can remain provisional
P3	Implementation/detail-level

Current priorities:

Question	Priority
Membership boundary	P0
Item identity	P0
External Identity ownership	P0
Ownership semantics	P1
Lifecycle completeness	P1
Classification ownership	P1
Synchronization ownership	P1
Cross-Aggregate consistency	P1
Capacity	P2
Ordering	P2
Audit requirements	P2
Temporal rules	P2
Messaging guarantees	P2
72. Decision Resolution Protocol

When resolving an open question:

Step 1

Describe the business requirement.

Step 2

Identify the domain concepts involved.

Step 3

Identify affected invariants.

Step 4

Identify affected Aggregates.

Step 5

Evaluate alternatives.

Step 6

Record the selected option.

Step 7

Record consequences.

Step 8

Update affected artifacts.

Step 9

Run the consistency review again.

This prevents local decisions from silently breaking the global model.

73. Decision Traceability

Every accepted decision should eventually reference:

Concept
↓
Requirement
↓
Rule
↓
Aggregate
↓
Use Case
↓
Event

This provides a defensible reasoning chain.

74. What Must Not Happen

The following process is explicitly discouraged:

Open Question
    ↓
Developer preference
    ↓
Implementation
    ↓
"Now the model is fixed"

Instead:

Open Question
    ↓
Domain evidence
    ↓
Alternatives
    ↓
Decision
    ↓
Consistency review
    ↓
Implementation
75. Decision Freeze Criteria

A domain decision may be considered sufficiently mature when:

its business meaning is understood;
alternatives have been considered;
affected invariants are known;
Aggregate impact is understood;
use-case impact is understood;
event impact is understood;
consistency implications are known;
unresolved dependencies are documented.

Only then should the decision be treated as implementation guidance.

76. Current Decision Register
ID	Decision	Status	Confidence
D-001	Domain-centric modeling	ACCEPTED	HIGH
D-002	Aggregates own consistency	ACCEPTED	HIGH
D-003	Specifications are predicates	ACCEPTED	HIGH
D-004	Policies own decisions	ACCEPTED	HIGH
D-005	Domain Services are exceptional	ACCEPTED	HIGH
D-006	Events represent facts	ACCEPTED	HIGH
D-007	Application coordinates	ACCEPTED	HIGH
D-008	Infrastructure does not define domain	ACCEPTED	HIGH
D-009	Collection as core Aggregate	PROVISIONAL	HIGH
D-010	Item independently modeled	PROVISIONAL	MEDIUM
D-011	Membership representation	OPEN	HIGH
D-013	External Identity is domain concept	ACCEPTED	HIGH
D-014	External Identity ownership	OPEN	HIGH
D-015	Ownership is domain concept	ACCEPTED	HIGH
D-017	Ownership model	OPEN	HIGH
D-018	Lifecycle as explicit state	ACCEPTED	HIGH
D-020	Lifecycle completeness	OPEN	HIGH
D-021	Aggregate-local strong consistency	ACCEPTED	HIGH
D-023	Projection consistency	PROVISIONAL	MEDIUM
D-024	Domain vs Integration Events	ACCEPTED	HIGH
D-027	Explicit idempotency	ACCEPTED	HIGH
D-030	Classification ownership	OPEN	MEDIUM
D-031	Synchronization can be domain concern	ACCEPTED	HIGH
D-032	Conflict resolution separated from transport	ACCEPTED	HIGH
77. Current Open Question Register
ID	Question	Priority	Status
Q-001	Definitive Aggregate map	P0	OPEN
Q-002	Membership semantics	P0	OPEN
Q-003	Membership attributes	P0	OPEN
Q-004	Ownership cardinality	P1	OPEN
Q-005	Collaborator model	P1	OPEN
Q-006	Classification model	P1	OPEN
Q-007	External Identity cardinality	P0	OPEN
Q-008	External Identity reassignment	P1	OPEN
Q-009	Synchronization ownership	P1	OPEN
Q-010	Collection lifecycle	P1	OPEN
Q-011	Deletion semantics	P1	OPEN
Q-012	Ordering semantics	P2	OPEN
Q-013	Capacity semantics	P2	OPEN
Q-014	Item eligibility	P1	OPEN
Q-015	Duplicate semantics	P0	OPEN
Q-016	Conflict taxonomy	P1	OPEN
Q-017	Automatic resolution	P1	OPEN
Q-018	Event catalog completeness	P1	OPEN
Q-019	Side-effect guarantees	P2	OPEN
Q-020	Audit requirements	P2	OPEN
Q-021	Multi-tenancy	P1	OPEN
Q-022	Permission model	P1	OPEN
Q-023	Temporal rules	P2	OPEN
Q-024	Concurrency requirements	P1	OPEN
78. Current Domain Model Maturity

The model can now be described as:

                    DOMAIN MODEL
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       Concepts        Rules         Behaviors
          │              │              │
          └──────────────┼──────────────┘
                         │
                 Core structure
                         │
                         ▼
              Several decisions closed
                         │
                         ▼
              Critical questions remain
                         │
                         ▼
               Decision closure phase

This is an intentional state.

The model is sufficiently mature to identify its uncertainties precisely.

That is progress, not incompleteness.

79. Architectural Principle

An explicit open question is healthier than an undocumented assumption.

Therefore:

Unknown is an acceptable modeling state. Hidden uncertainty is not.

The purpose of this document is precisely to make uncertainty visible.

80. Definition of Done

This artifact is complete when:

 Major architectural/domain decisions are explicitly recorded.
 Each decision has a rationale.
 Consequences are documented.
 Open questions are explicit.
 Questions have priorities.
 Deferred decisions are separated from unresolved decisions.
 Rejected alternatives are recorded.
 Decision dependencies are known.
 Critical P0 questions are identified.
 A future decision can be traced back to affected domain artifacts.
81. Final Position

CollectionHub has now progressed from domain discovery into domain decision management.

The next objective should not be to produce abstractions indefinitely.

The next objective is to resolve the highest-impact questions:

P0
│
├── Membership boundary
├── Item identity
├── External Identity cardinality/ownership
└── Duplicate semantics

followed by:

P1
│
├── Ownership
├── Authorization
├── Lifecycle
├── Classification
├── Synchronization
├── Cross-Aggregate consistency
├── Event completeness
└── Concurrency

Once those decisions are resolved, the resulting model can be subjected to another consistency pass and transformed into a complete traceability model.

82. Status

Artifact: 20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md

Status: Decision Register Established

Current conclusion:

Domain concepts
      ↓
Domain rules
      ↓
Domain behavior
      ↓
Consistency review
      ↓
Explicit decisions
      ↓
Explicit uncertainty
      ↓
Ready for decision closure