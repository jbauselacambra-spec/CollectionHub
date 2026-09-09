30 — Persistence Architecture and Data Boundaries

1. Purpose

This document defines the persistence architecture of CollectionHub based on the domain model, application use cases, component interactions and infrastructure architecture established in documents 24 through 29.

The objective is to define:

authoritative data ownership;
persistence boundaries;
aggregate persistence;
transaction boundaries;
consistency guarantees;
persistence models;
repository responsibilities;
concurrency control;
read models;
projections;
search persistence;
caching;
audit/history;
schema evolution;
deletion policies;
data integrity;
persistence anti-corruption boundaries.

This document intentionally avoids selecting a concrete database technology.

The architectural question is:

How must CollectionHub persist and protect its domain state without allowing the persistence technology to redefine the domain model?

2. Architectural Context

The persistence architecture sits below the Application and Domain layers:

                    Delivery
                       │
                       ▼
                 Application
                       │
                 Use Cases / Ports
                       │
                       ▼
                    Domain
                       │
              Aggregates / Events
                       │
                       ▼
              Persistence Ports
                       │
                       ▼
               Infrastructure
                       │
          ┌────────────┼─────────────┐
          ▼            ▼             ▼
      Transactional  Read Models   Search
        Storage        / Projections

The transactional store is authoritative.

Read models, caches and search indexes are derived representations.

3. Persistence Principles

CollectionHub adopts the following principles.

P1 — Domain state is authoritative

Transactional persistence stores the authoritative state of the business model.

P2 — Aggregate boundaries define consistency

An aggregate determines the minimum unit whose invariants must be protected atomically.

P3 — Repositories persist aggregates

Repositories are not generic table-access utilities.

P4 — Read models are optimized representations

Read models may denormalize information without becoming authoritative.

P5 — Search is derived

Search infrastructure is eventually consistent unless an explicit use case requires otherwise.

P6 — Persistence is replaceable

The Domain must not depend on a concrete database technology.

P7 — Technical schema must not dictate business concepts

Database normalization, indexes, keys and storage optimizations remain infrastructure concerns.

P8 — Cross-aggregate transactions require justification

Frequent cross-aggregate writes are a signal to review boundaries rather than automatically expand transactions.

4. Authoritative Data Model

The authoritative persistence model represents domain state associated with:

Collection
Item
Classification
Acquisition
Valuation
Disposal
Provenance
Media

The exact physical schema remains an implementation concern.

Conceptually:

                   ┌──────────────┐
                   │  Collection  │
                   └──────┬───────┘
                          │
                          │ owns / contains
                          ▼
                   ┌──────────────┐
                   │     Item     │
                   └──┬─────┬─────┘
                      │     │
          ┌───────────┘     └─────────────┐
          ▼                               ▼
   Classification                    Acquisition
          │                               │
          ▼                               ▼
      Valuation                       Provenance
          │
          ▼
      Disposal


Item ───────────────► Media

This is a conceptual relationship map, not a final relational schema.

5. Data Ownership

Each business concept must have one authoritative owner.

Concept	Authoritative Owner
Collection identity	Collection
Collection lifecycle	Collection
Item identity	Item
Item lifecycle	Item
Classification definition	Classification
Item classification relationship	Domain-defined owner
Acquisition record	Acquisition boundary
Valuation record	Valuation boundary
Disposal record	Disposal boundary
Provenance history	Provenance boundary
Media association	Media boundary
Binary media content	Media Storage

The final aggregate model must remain the authority for resolving any ambiguity.

6. Aggregate Persistence Boundary

An aggregate is persisted as a consistency boundary.

Conceptually:

Aggregate
    │
    ├── identity
    ├── state
    ├── owned entities
    ├── value objects
    └── invariant state
          │
          ▼
       Repository
          │
          ▼
      Persistence

The repository should reconstruct an aggregate in a valid state.

7. Aggregate Reconstruction

Repository loading must not produce partially valid aggregates.

The expected flow is:

Database
   │
   ▼
Persistence Model
   │
   ▼
Mapper
   │
   ▼
Aggregate Factory / Reconstruction
   │
   ▼
Valid Aggregate

If persisted data violates a domain invariant, this represents a persistence integrity problem that must be surfaced rather than silently ignored.

8. Collection Persistence

Collection persistence is responsible for collection-specific authoritative state.

Conceptual state:

Collection
├── CollectionId
├── Name
├── Description
├── Status
├── Lifecycle information
└── Version

The physical schema may contain additional technical fields.

Collection persistence must not directly manage item state unless the aggregate boundary explicitly requires it.

9. Item Persistence

Item is expected to be one of the central persistence boundaries.

Conceptual state:

Item
├── ItemId
├── CollectionId / Collection Reference
├── Identity
├── Description
├── Classification Reference
├── Lifecycle State
├── Version
└── Domain Metadata

Historical information such as acquisition, valuation and provenance should not automatically be embedded into the Item persistence record.

Separate boundaries should be retained where their lifecycle and consistency requirements differ.

10. Classification Persistence

Classification definitions require their own persistence boundary.

Conceptual model:

Classification
├── ClassificationId
├── ParentReference
├── Name
├── Description
├── Status
└── Version

A hierarchical classification system should be persisted in a way that supports:

parent-child relationships;
hierarchy traversal;
uniqueness rules;
lifecycle management.

The physical representation is not prescribed here.

11. Acquisition Persistence

Acquisition records represent historical business facts.

Conceptually:

Acquisition
├── AcquisitionId
├── ItemReference
├── Date
├── Source
├── Cost
├── Metadata
└── Version

Acquisition history should normally be append-oriented.

Historical acquisition facts should not be overwritten merely to reflect current state.

12. Valuation Persistence

Valuation is inherently temporal.

Conceptually:

Valuation
├── ValuationId
├── ItemReference
├── Date
├── Amount
├── Currency
├── Source
└── Metadata

The persistence model should support:

current valuation;
historical valuations;
valuation chronology;
source identification.

The "current valuation" should preferably be derived from authoritative valuation history rather than becoming a separate competing source of truth.

13. Disposal Persistence

Disposal represents a lifecycle transition and potentially a historical fact.

Conceptually:

Disposal
├── DisposalId
├── ItemReference
├── Date
├── Reason
├── Method
├── Destination
└── Metadata

Disposal information should normally remain historically available.

Physical deletion of an item should not automatically erase its disposal history.

14. Provenance Persistence

Provenance requires preservation of historical relationships.

Conceptually:

ProvenanceRecord
├── ProvenanceId
├── ItemReference
├── PreviousOwner
├── NewOwner
├── Date
├── Source
└── Evidence

Provenance persistence should favor immutability or controlled correction rather than unrestricted updates.

The architecture must preserve historical meaning.

15. Media Persistence

Media has two distinct persistence concerns:

Media Metadata
       +
Binary Content

They should not be treated as the same storage problem.

Conceptually:

Media Metadata
     │
     ├── MediaId
     ├── ItemReference
     ├── MediaType
     ├── Role
     ├── StorageReference
     └── Metadata
            │
            ▼
      Media Storage
            │
            ▼
       Binary Content

The database need not contain the actual binary data.

16. Data Ownership Rule

A component must not directly modify data owned by another aggregate.

Bad:

ItemService
    │
    ▼
UPDATE classification_table

when Classification owns that state.

Preferred:

Application Use Case
       │
       ├── ClassificationRepository
       │
       └── ItemRepository

with domain behavior deciding whether the interaction is valid.

17. Foreign Keys and Domain Boundaries

Physical foreign keys do not automatically imply aggregate boundaries.

For example:

item.collection_id

may represent a technical relationship.

It does not mean that:

Collection + all Items

must become one aggregate.

The distinction is:

Database Relationship
        ≠
Aggregate Boundary

This is an important architectural constraint.

18. Referential Integrity

The persistence layer should enforce technical integrity where appropriate.

Examples:

required references;
uniqueness;
non-null constraints;
valid foreign keys;
version fields;
storage references.

However, business invariants that span aggregates should remain explicitly modeled in Application/Domain logic.

19. Unique Constraints

The persistence schema should support technical enforcement of invariants that are naturally representable as uniqueness constraints.

Examples may include:

Collection name within required scope
Classification name within parent scope
External identifier within provider scope
Media identifier

The exact constraints must be derived from finalized domain rules.

Database uniqueness is a useful safety net but not a replacement for Domain validation.

20. Transaction Boundary

The default transaction boundary is the Application use case.

Example:

RecordValuation
     │
     ├── Load Item
     ├── Execute domain behavior
     ├── Persist valuation
     └── Commit

The transaction should contain exactly the state that must become consistent atomically.

21. Cross-Aggregate Transactions

Cross-aggregate transactions are allowed when a business invariant genuinely requires atomicity.

However:

Frequent cross-aggregate transaction
        ↓
Potential boundary problem

The architecture therefore requires explicit justification for transactions involving multiple aggregates.

22. Transaction Example — Item Creation
[TX]
CreateItem
    │
    ├── validate required collection reference
    ├── create Item
    ├── persist Item
    └── commit

Secondary effects should normally occur after commit.

23. Transaction Example — Disposal
[TX]
DisposeItem
    │
    ├── load Item
    ├── validate disposal
    ├── change lifecycle
    ├── persist Item
    ├── persist Disposal Record
    └── commit

If Disposal is a separate aggregate, the need for a single transaction must be explicitly justified by the domain consistency requirement.

24. Transaction Example — Search

Search does not participate in the domain transaction.

[TX]
ItemUpdated
    │
    ▼
Commit
    │
    ▼
ItemUpdated Event
    │
    ▼
Search Projection [ASYNC]

This protects transactional state from search infrastructure failures.

25. Optimistic Concurrency

Aggregate persistence should support optimistic concurrency where concurrent modification is possible.

Conceptual model:

Item
├── id
├── state
└── version

Update:

Expected version = 12


Database version = 13


        ↓


Concurrency Conflict

The Application layer can then decide whether to:

reject;
inform the user;
retry safely.

Automatic retries must not blindly replay business operations.

26. Concurrency Rule

The infrastructure must never silently overwrite a newer aggregate state.

Bad:

last write wins

unless that behavior is explicitly required by the domain.

Preferred default:

detect conflict
    ↓
fail explicitly
27. Soft Delete vs. Hard Delete

Deletion must be modeled according to business meaning.

If an item is "disposed", that is not necessarily deletion.

Therefore:

Disposed Item
    ≠
Deleted Item

Disposal is a domain lifecycle state.

Deletion is a persistence operation.

These must never be conflated automatically.

28. Deletion Policy

The architecture should distinguish:

Business lifecycle removal
Active → Disposed
Technical deletion
Persisted → Removed

Technical deletion should be used only where retention rules allow it.

29. Historical Data

Historical facts such as:

acquisition;
valuation;
disposal;
provenance;

should normally remain queryable after state transitions.

This provides:

Current State
+
Historical State

rather than only:

Current State
30. Immutability of Historical Facts

Historical records should generally be treated as immutable facts.

Corrections should preferably be represented through:

Correction
+
Audit Trail

rather than silent mutation.

For example:

Original Valuation
       │
       ▼
Correction
       │
       ▼
New Valuation

This preserves traceability.

31. Audit Persistence

Audit information is a separate concern from domain history.

Potential audit model:

AuditRecord
├── AuditId
├── Timestamp
├── Actor
├── Operation
├── Entity
├── EntityId
├── Outcome
└── Metadata

Audit records should not be embedded into every aggregate merely to support technical auditing.

32. Domain History vs. Audit History

These are different concepts.

Domain history

Answers:

What happened to the business object?

Example:

Item acquired
Item valued
Item transferred
Item disposed
Audit history

Answers:

Who or what executed the operation and when?

Example:

User X executed DisposeItem at timestamp Y.

The architecture keeps these concerns separate.

33. Read Models

Read models are optimized for query requirements.

Example:

ItemDetailsReadModel
├── Item
├── Collection Name
├── Classification Name
├── Current Valuation
├── Lifecycle State
└── Media Summary

This avoids requiring the query layer to reconstruct multiple aggregates for every read.

34. Read Model Ownership

Read models are derived.

Their lifecycle is controlled by the projection mechanism, not by the Domain.

Conceptually:

Domain State
    │
    ▼
Domain Event
    │
    ▼
Projection Handler
    │
    ▼
Read Model
35. Read Model Consistency

Read models are generally eventually consistent.

This means:

Transaction committed
        │
        ▼
Read model updated shortly afterwards

The Application API should not promise stronger consistency than the underlying model provides.

36. Search Projection

Search persistence is a specialized read model.

Conceptually:

ItemUpdated
ItemClassified
ItemDisposed
ValuationRecorded
      │
      ▼
Search Projection Handler
      │
      ▼
Search Index

The index may contain fields from multiple authoritative aggregates.

That denormalization is intentional.

37. Search Index Rebuild

Because the search index is derived, the architecture must allow reconstruction.

Required capability:

Authoritative State
        │
        ▼
Projection/Reindex Process
        │
        ▼
Rebuilt Search Index

This is an important architectural property.

The application must never depend on the search index being the only representation of an item.

38. Projection Versioning

Read and search projections may evolve independently.

Potential strategy:

Projection V1
Projection V2

with controlled migration/rebuild.

Projection schemas should not constrain Domain evolution unnecessarily.

39. Cache Boundaries

Caching may be introduced for:

frequently accessed collections;
classification trees;
item summaries;
read-heavy queries.

Cache contents are always derived.

Authoritative Store
        │
        ▼
Cache

Never:

Cache
  ↓
Authoritative State
40. Cache Invalidation

Cache invalidation should be associated with relevant changes.

Possible mechanisms:

Domain Event
    │
    ▼
Cache Invalidation

or:

TTL

The choice depends on consistency requirements.

41. Data Duplication

Controlled duplication is permitted in:

read models;
search indexes;
caches;
reporting projections.

Duplication is not permitted as an excuse for multiple competing sources of truth.

The architecture must always be able to answer:

Which store owns this value?

42. Data Ownership Matrix
Data	Source of Truth	Derived Copies
Collection	Collection persistence	Search / Read Models
Item	Item persistence	Search / Read Models
Classification	Classification persistence	Search / Read Models
Acquisition	Acquisition persistence	Reports / Read Models
Valuation	Valuation persistence	Search / Reports
Disposal	Disposal persistence + Item lifecycle	Search / Reports
Provenance	Provenance persistence	Historical Read Models
Media Metadata	Media persistence	Item Read Models
Media Binary	Media Storage	Optional derivatives
43. Persistence Model vs Domain Model

The persistence model may be optimized differently from the Domain.

For example:

Domain:
Item
  └── Value Objects


Persistence:
item_table
  ├── item_id
  ├── title
  ├── amount
  ├── currency
  └── version

This does not mean the Domain should expose primitive database fields.

Mapping protects the conceptual model.

44. Value Object Persistence

Value Objects should be persisted according to their semantics.

Possible strategies include:

Embedded columns
Separate table
Serialized representation
Native database type

The choice depends on:

query requirements;
lifecycle;
normalization;
technology;
performance.

The Domain remains independent from the choice.

45. Enumerations and States

Domain states may be persisted using stable technical representations.

For example:

ACTIVE
DISPOSED
ARCHIVED

The persistence representation must remain stable enough to support schema evolution.

Avoid coupling database values directly to programming-language enum implementation details where possible.

46. Database Transactions and Domain Events

Domain events must be correlated with transaction boundaries.

Preferred model:

Domain Operation
      │
      ▼
Aggregate raises Event
      │
      ▼
Transaction records state + event
      │
      ▼
Commit
      │
      ▼
Asynchronous publication

This strongly supports reliable event-driven processing.

47. Outbox Persistence Boundary

If the Outbox pattern is adopted, the outbox belongs to Infrastructure.

Conceptual model:

Transactional Store
├── Domain State
└── Outbox
      │
      ▼
Outbox Dispatcher
      │
      ▼
Message Broker

The Outbox is technical infrastructure, not a Domain aggregate.

48. Event Ordering

Some projections may require event ordering.

Infrastructure should preserve ordering where the relevant consistency boundary requires it.

Possible ordering key:

AggregateId
+
AggregateVersion

This allows a consumer to identify stale or out-of-order events.

49. Event Replay

Derived projections should ideally support replay.

Example:

Historical Events
       │
       ▼
Projection Handler
       │
       ▼
New Read Model

This is especially useful for:

search;
reporting;
analytics;
future projections.

Replay capability should not be confused with requiring full event sourcing.

50. Event Sourcing Decision

CollectionHub does not require event sourcing by default.

The current architecture assumes:

Current Aggregate State
+
Historical Domain Records
+
Domain Events

rather than:

Event Store
     ↓
Reconstruct every aggregate state

Full event sourcing should only be introduced if explicit domain requirements justify its complexity.

51. Temporal Data

Valuation, acquisition, disposal and provenance contain temporal information.

Persistence should preserve:

business effective date;
creation timestamp;
update timestamp where relevant.

These concepts must not be conflated.

Example:

valuationDate
≠
recordedAt
52. Time Zone Policy

All technical timestamps should follow one explicit storage policy.

A likely architectural default is:

UTC

Business-local dates should remain modeled as dates where time-of-day has no business meaning.

The final policy must be documented before implementation.

53. Monetary Data

Monetary values require explicit persistence semantics.

A valuation should conceptually contain:

Amount
Currency

Avoid storing money as an ambiguous floating-point number.

The exact database representation will be chosen during technology-specific design.

54. External Identifiers

External identifiers should not replace internal domain identity.

Conceptually:

Internal ItemId
       +
ExternalReference

This allows integration with multiple external systems without coupling domain identity to an external provider.

55. Data Import Persistence

Large imports should not necessarily execute one giant transaction.

Potential architecture:

Import
  │
  ▼
Validation
  │
  ▼
Batch Processing
  │
  ├── Batch 1
  ├── Batch 2
  ├── Batch 3
  └── ...

The consistency model must be explicitly defined.

A failed batch must not leave partially invalid domain state.

56. Bulk Persistence Rule

Bulk operations may optimize persistence only when domain invariants remain preserved.

Unsafe:

Bulk SQL UPDATE
    ↓
Bypass Domain

Preferred:

Application Bulk Workflow
       ↓
Domain Rules
       ↓
Controlled Persistence Optimization

If a bulk operation requires bypassing domain invariants, it is not an ordinary application operation and must be treated as a special administrative capability.

57. Reporting Data

Reporting should preferably use derived projections rather than querying the transactional schema for every complex report.

Conceptual architecture:

Domain State
    │
    ▼
Domain Events
    │
    ▼
Reporting Projection
    │
    ▼
Reporting Store

This protects transactional workloads from expensive analytical queries.

58. Data Retention

Retention rules must be defined separately for:

transactional data;
domain history;
audit records;
search indexes;
media;
logs;
integration messages.

No generic "delete old data" policy should be applied across all categories.

59. Backup and Recovery

Infrastructure persistence must support:

backups;
restore;
disaster recovery;
consistency verification.

The authoritative transactional store receives the highest recovery priority.

Derived stores should be reconstructable where practical.

60. Disaster Recovery Principle

A derived store should ideally be recoverable from authoritative state.

For example:

Search Store Lost
       │
       ▼
Rebuild from authoritative data

This substantially reduces operational coupling.

61. Schema Evolution

Persistence schemas must evolve independently from Domain source structure where possible.

Expected migration sequence:

Domain Change
      │
      ▼
Persistence Mapping Change
      │
      ▼
Schema Migration
      │
      ▼
Projection Update

Backward compatibility may be required during rolling deployments.

62. Expand-and-Contract Migration Strategy

For potentially disruptive schema changes:

1. Add new structure
2. Deploy compatible code
3. Backfill data
4. Switch reads/writes
5. Remove old structure

This should be preferred over destructive one-step migrations in production environments.

63. Persistence Security

Persistence infrastructure must enforce:

authentication;
authorization;
encryption in transit;
encryption at rest where required;
secret management;
least privilege;
backup protection.

Security mechanisms remain infrastructure concerns.

64. Sensitive Data

The architecture should classify data before implementation.

Potential categories:

Public
Internal
Sensitive
Restricted

The exact classification depends on CollectionHub requirements.

Persistence, logs, exports and backups must respect the classification.

65. Database Access Rule

Only Infrastructure persistence adapters should communicate directly with the database.

Forbidden:

Controller
    ↓
SQL


Application Use Case
    ↓
ORM Entity Manager


Domain
    ↓
Database

Preferred:

Application
    ↓
Repository Port
    ↓
Infrastructure Adapter
    ↓
Database
66. Query Responsibility

Read-heavy queries may use dedicated read ports rather than forcing repositories to return aggregates.

Example:

GetItemDetails
    ↓
ItemReadPort
    ↓
ItemDetailsReadModel

This avoids unnecessary aggregate reconstruction for queries.

67. Repository Responsibility

A repository is responsible for:

loading aggregate state;
persisting aggregate state;
querying by aggregate identity;
enforcing persistence-level concurrency;
mapping between domain and persistence.

A repository is not responsible for:

HTTP;
search ranking;
authorization;
business workflows;
external API orchestration;
event business semantics.
68. Persistence Error Translation

Technical persistence failures should be translated at the infrastructure boundary.

Example:

DatabaseUniqueConstraintViolation
          │
          ▼
Infrastructure Error
          │
          ▼
Application-level failure

The Domain should not receive raw database exceptions.

69. Infrastructure Exception Categories

Potential categories:

PersistenceFailure
ConcurrencyConflict
UniqueConstraintViolation
ConnectionFailure
TransactionFailure
SerializationFailure
ExternalServiceFailure
StorageFailure
MessageDeliveryFailure

The exact hierarchy remains implementation-specific.

70. Persistence Performance

Performance optimization must follow evidence.

Potential optimization mechanisms include:

indexes;
projections;
caching;
batching;
pagination;
connection pooling;
query specialization;
asynchronous processing.

Premature denormalization should be avoided.

71. Indexing Strategy

Indexes should support identified access patterns.

Likely candidates:

CollectionId
ClassificationId
LifecycleState
ExternalReference
ValuationDate
AcquisitionDate
DisposalDate

The final index set must derive from real query patterns.

Search-specific text indexes belong to Search Infrastructure rather than necessarily to the transactional database.

72. Pagination

Large result sets must not be loaded indiscriminately.

Application queries should define explicit pagination semantics.

Possible strategies:

Offset Pagination
Cursor Pagination
Keyset Pagination

The choice depends on expected data volume and consistency requirements.

73. Aggregate Loading Strategy

Repositories should avoid unnecessary loading of unrelated state.

Example:

Load Item

should not automatically load:

All Provenance
All Media
All Valuations
All Acquisitions

unless the aggregate boundary requires it.

This protects performance and preserves conceptual boundaries.

74. Persistence Anti-Corruption Layer

Where database schemas are legacy, shared or externally controlled, an explicit anti-corruption layer should isolate them.

Domain
   ▲
   │
Domain Mapper
   ▲
   │
Persistence Adapter
   ▲
   │
Legacy Schema

The Domain should not be reshaped merely to match a legacy database.

75. Shared Database Considerations

A shared database is not automatically a shared domain boundary.

Modules may share physical infrastructure while maintaining logical ownership.

For example:

Database
├── Collection-owned tables
├── Item-owned tables
├── Valuation-owned tables
└── Projection tables

Logical ownership must remain explicit.

76. Shared Table Warning

Multiple modules directly writing the same table should be treated as an architectural smell.

Preferred:

One Owner
    ↓
One Write Boundary
    ↓
Other modules read through contracts/projections

This reduces hidden coupling.

77. Data Boundary Matrix
Boundary	Owns Data	Others May
Collection	Collection state	Reference
Item	Item state	Reference
Classification	Classification definitions	Reference
Acquisition	Acquisition history	Read
Valuation	Valuation history	Read
Disposal	Disposal history	Read
Provenance	Provenance history	Read
Media	Media metadata	Reference
Search	Derived index	Query
Reporting	Derived reporting state	Query
Cache	Derived temporary state	Read
78. Persistence Interaction Map
                  Application Use Case
                          │
                          ▼
                   Repository Port
                          │
                          ▼
                Persistence Adapter
                          │
                    ┌─────┴─────┐
                    ▼           ▼
               Persistence    Unit of
                 Mapper       Work
                    │           │
                    └─────┬─────┘
                          ▼
                    Transaction
                          │
                          ▼
                    Authoritative
                       Store

After successful commit:

Authoritative Store
        │
        ▼
Domain Event / Outbox
        │
        ├──────────► Search Projection
        ├──────────► Read Model
        ├──────────► Reporting
        └──────────► Integration
79. Persistence Consistency Matrix
Concern	Consistency
Aggregate state	Strong
Aggregate invariants	Strong
Collection membership	Defined by aggregate boundary
Acquisition history	Strong within acquisition boundary
Valuation history	Strong within valuation boundary
Disposal state	Strong
Provenance history	Strong
Search	Eventual
Reporting	Eventual
Cache	Eventual
External integrations	Eventual / provider-dependent
80. Persistence Failure Strategy

The architecture distinguishes:

Transactional failure
Database unavailable
      ↓
Use case fails
      ↓
No partial commit
Projection failure
Search unavailable
      ↓
Domain transaction remains committed
      ↓
Projection retries
External integration failure
External API unavailable
      ↓
Retry / queue / failure state
      ↓
Core domain state remains protected
81. Persistence Testing Strategy

Persistence should be tested through:

Repository integration tests

Verify:

save;
load;
update;
delete where applicable;
concurrency;
mapping;
transactions.
Migration tests

Verify:

schema creation;
upgrade;
rollback strategy where supported;
compatibility.
Projection tests

Verify:

event → projection;
rebuild;
replay;
idempotency.
Performance tests

Verify:

realistic query volumes;
indexing;
pagination;
aggregate loading.
82. Persistence Contract Tests

Repository implementations must satisfy the same behavioral contract.

Conceptually:

Repository Contract
      │
      ├── SQL Implementation
      ├── Test Implementation
      └── Future Implementation

This ensures that changing the persistence technology does not alter Application semantics.

83. Data Migration Testing

Migration testing should verify:

Existing Data
     │
     ▼
Migration
     │
     ▼
New Schema
     │
     ▼
Valid Domain State

A migration that produces technically valid rows but invalid domain state is unacceptable.

84. Persistence Architecture Invariants

The following invariants are established.

PI1

Every authoritative business concept has one explicit owner.

PI2

Aggregate invariants are protected transactionally.

PI3

Repositories do not become generic database access layers.

PI4

Read models cannot become authoritative accidentally.

PI5

Search can be rebuilt from authoritative state.

PI6

Historical domain facts are preserved where required.

PI7

Technical deletion is distinct from business lifecycle transitions.

PI8

Concurrent updates are detected rather than silently overwritten.

PI9

Database structure does not dictate the Domain model.

PI10

Infrastructure errors do not leak raw technical exceptions inward.

PI11

Derived data can be discarded and reconstructed when practical.

PI12

Only the owning boundary writes authoritative data.

85. Persistence Risks
R1 — Aggregate/table confusion

Risk:

One table = One aggregate

being treated as an architectural rule.

Mitigation:

Define aggregate boundaries from domain invariants.

R2 — Shared-table coupling

Risk:

Multiple modules directly update the same persistence structure.

Mitigation:

Establish explicit ownership.

R3 — Search becoming authoritative

Risk:

Application reads critical state from search.

Mitigation:

Transactional store remains source of truth.

R4 — Excessive cross-aggregate transactions

Risk:

Application becomes tightly coupled to many aggregates.

Mitigation:

Review aggregate boundaries and workflows.

R5 — Persistence model leaking inward

Risk:

Domain begins using ORM/database types.

Mitigation:

Explicit mapping boundary.

R6 — Historical information loss

Risk:

Updates overwrite facts required for provenance/audit.

Mitigation:

Append-oriented historical records and explicit correction semantics.

R7 — Premature denormalization

Risk:

Duplicated state becomes inconsistent.

Mitigation:

Introduce projections only for identified read requirements.

86. Persistence Decision Record

CollectionHub adopts the following architectural position:

Persistence is organized around domain ownership and aggregate consistency boundaries, while physical storage remains an infrastructure concern.

This means:

Domain
  ↓
defines meaning


Application
  ↓
defines use-case transaction


Infrastructure
  ↓
defines physical persistence
87. Technology Selection Constraints

When a concrete persistence technology is eventually selected, it must satisfy at least:

transactional consistency;
aggregate-level persistence;
optimistic concurrency or equivalent;
reliable migrations;
indexing;
pagination;
transaction support;
operational observability;
backup/recovery;
integration with the chosen deployment model.

The technology must adapt to these constraints.

The architecture must not be redesigned around a database's accidental limitations without explicit review.

88. Recommended Persistence Evolution

The expected evolution path is:

Phase 1
Transactional persistence
        │
        ▼
Phase 2
Read models where justified
        │
        ▼
Phase 3
Search projection
        │
        ▼
Phase 4
Caching where measurable
        │
        ▼
Phase 5
Reporting projections
        │
        ▼
Phase 6
Advanced scaling only when required

This avoids premature infrastructure complexity.

89. Complete Persistence Architecture

The resulting architecture is:

                         APPLICATION
                              │
                       Repository Ports
                              │
                              ▼
                    ┌──────────────────┐
                    │ Persistence      │
                    │ Adapters         │
                    └────────┬─────────┘
                             │
                       Unit of Work
                             │
                             ▼
                    ┌──────────────────┐
                    │ Authoritative    │
                    │ Transactional    │
                    │ Store            │
                    └────────┬─────────┘
                             │
                       Domain Events
                             │
                             ▼
                    ┌──────────────────┐
                    │ Outbox / Event   │
                    │ Publication      │
                    └────────┬─────────┘
                             │
          ┌──────────────────┼───────────────────┐
          ▼                  ▼                   ▼
     Search Index       Read Models        Reporting Store
          │                  │                   │
          └──────────────────┴───────────────────┘
                             │
                       Derived State

The important distinction is:

Authoritative State
        ≠
Derived State
90. Final Architectural Principles

The persistence architecture of CollectionHub is governed by the following principles:

Domain ownership determines authoritative data ownership.
Aggregate boundaries determine transactional consistency.
Repositories persist aggregates rather than arbitrary tables.
Persistence models are separate from domain models.
Read models optimize retrieval without becoming sources of truth.
Search is a derived projection.
Historical business facts must remain distinguishable from current state.
Business disposal is not technical deletion.
Concurrency conflicts must be explicit.
Cross-aggregate transactions require justification.
Derived data should be reconstructable where practical.
Infrastructure technology must remain replaceable.
Schema evolution must be deliberate and reversible where operationally possible.
Only owning boundaries modify authoritative state.
Persistence failures and projection failures must remain separate failure domains.
91. Relationship with Previous Documents

The architectural chain is now:

24 Architectural Boundaries
        ↓
25 Architectural Components
        ↓
26 Application / Domain Modules
        ↓
27 Component Interactions
        ↓
28 Application Use Case Interaction Map
        ↓
29 Infrastructure Components and Adapters
        ↓
30 Persistence Architecture and Data Boundaries

Each step progressively reduces architectural ambiguity without prematurely committing to implementation technology.

92. Next Architectural Step

The next document should define:

31_DATA_MODEL_AND_PERSISTENCE_MAPPING.md

That document will move one level deeper and establish the conceptual mapping between the Domain Model and the Persistence Model, including:

aggregate-to-persistence mapping;
entity mapping;
value-object mapping;
relationships;
identifiers;
versioning;
temporal fields;
historical records;
persistence-specific representations;
read-model mapping;
projection fields;
database constraints;
indexing candidates;
mapping decisions;
unresolved data-model questions.

The progression is therefore:

Domain
  ↓
Application
  ↓
Architecture
  ↓
Infrastructure
  ↓
Persistence Boundaries
  ↓
Persistence Mapping