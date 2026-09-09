# Database Schema and Domain Persistence Mapping


## 1. Purpose


This document defines the concrete database schema strategy for CollectionHub and establishes how the domain model is represented in PostgreSQL through the persistence layer.


It builds directly on:


- `22_DOMAIN_MODEL_FINAL.md`
- `24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md`
- `30_PERSISTENCE_ARCHITECTURE_AND_DATA_BOUNDARIES.md`
- `31_PERSISTENCE_COMPONENTS_AND_REPOSITORY_IMPLEMENTATIONS.md`
- `36_TECHNICAL_ARCHITECTURE_AND_TECHNOLOGY_DECISIONS.md`
- `37_TECHNOLOGY_STACK_AND_PLATFORM_SELECTION.md`
- `38_TECHNICAL_SOLUTION_STRUCTURE_AND_PROJECT_DEPENDENCIES.md`
- `39_TECHNICAL_PERSISTENCE_DESIGN_AND_EF_CORE_ARCHITECTURE.md`


The objective is to define:


- database ownership,
- schema organization,
- aggregate-to-table mapping,
- identifiers,
- relationships,
- value-object persistence,
- foreign keys,
- indexes,
- uniqueness constraints,
- concurrency metadata,
- temporal data,
- deletion behavior,
- database invariants,
- EF Core mapping responsibilities,
- migration strategy.


This document defines the persistence model without allowing PostgreSQL or EF Core to redefine the domain model.


---


# 2. Core Persistence Principle


The database is a representation of the domain's persistent state.


The direction remains:


Domain Model
    ↓
Persistence Mapping
    ↓
Relational Model
    ↓
PostgreSQL


The database must not become the source of truth for domain behavior.


The source of business meaning remains the domain model.


---


# 3. Persistence Boundary


The complete persistence boundary is:


```text
CollectionHub.Application
        │
        │ persistence contracts
        ▼
CollectionHub.Infrastructure
        │
        ├── Repository implementations
        ├── Persistence models
        ├── EF Core mappings
        └── DbContext
                │
                ▼
           PostgreSQL

No API component accesses PostgreSQL directly.

No domain component accesses PostgreSQL directly.

4. Database Ownership

The CollectionHub application owns its database schema.

External systems must never depend directly on CollectionHub tables.

External integrations communicate through application/infrastructure adapters.

Therefore:

External System
      │
      │ API / Integration
      ▼
CollectionHub
      │
      ▼
PostgreSQL

and never:

External System
      │
      └──────────► CollectionHub PostgreSQL
5. Initial Database Strategy

The initial deployment uses:

Database Engine:
PostgreSQL


Application Database:
CollectionHub


Primary Schema:
public

The use of PostgreSQL's public schema is acceptable initially because the application is a modular monolith with a single owned database.

A dedicated application schema may be introduced later if operational or modularization requirements justify it.

No premature multi-schema strategy is required.

6. Logical Database Organization

Although the database initially uses a single PostgreSQL schema, logical ownership remains aligned with the domain.

Conceptually:

CollectionHub Database
│
├── Collection persistence
├── Catalog/reference persistence
├── User/ownership persistence
├── Integration persistence
└── Technical persistence

The exact tables are derived from the finalized domain model and application use cases.

Technical tables must not become business aggregates merely because they exist in the database.

7. Aggregate-to-Table Principle

The primary mapping rule is:

One aggregate root represents one persistence ownership boundary, but not necessarily one database table.

An aggregate may require:

Aggregate Root
    │
    ├── Root Table
    ├── Child Entity Table
    ├── Relationship Table
    └── Supporting Tables

The important invariant is:

Tables belonging to the same aggregate must be persisted consistently as one aggregate operation.

8. Aggregate Boundaries

The aggregate boundaries defined by the domain model remain authoritative.

Persistence must not introduce new aggregate relationships merely because two records have a foreign key relationship.

For example:

Aggregate A
   │
   └── Entity A1


Aggregate B
   │
   └── Entity B1

may be represented relationally by:

A
A1
B
B1

with a foreign key between A and B.

The foreign key does not imply that A and B become one domain aggregate.

9. Persistence Model vs Domain Entity

A persistence model may differ from the corresponding domain object.

For example:

Domain:
Collection


Persistence:
CollectionRecord

The persistence representation may contain:

database identifier,
foreign keys,
normalized fields,
concurrency token,
persistence metadata.

The domain object contains:

business identity,
invariants,
behavior,
value objects,
domain semantics.

These concerns must remain separate.

10. Table Naming Convention

Tables should use singular or plural naming consistently across the entire schema.

The preferred convention is:

lower_snake_case

Examples:

collections
collection_items
external_references

The final convention must be applied consistently through EF Core configuration.

11. Column Naming Convention

Database columns use:

lower_snake_case

Examples:

id
created_at
updated_at
external_reference

C# properties remain idiomatic PascalCase.

Example:

CreatedAt

maps to:

created_at
12. Primary Key Strategy

Every aggregate root persistence record requires a stable primary key.

The preferred default is UUID-based identity where consistent with the domain identifier model.

Conceptually:

Domain Identifier
        ↓
Strongly Typed Identifier
        ↓
Guid / UUID
        ↓
PostgreSQL uuid

The domain identifier must not be replaced by an unrelated database-generated identifier without explicit architectural justification.

13. Identifier Generation

Identifiers should preferably be generated at the application/domain boundary when the domain requires identity before persistence.

This allows:

Create Aggregate
      ↓
Aggregate receives identity
      ↓
Domain Event / Application logic
      ↓
Persistence

rather than:

Database generates ID
      ↓
Application discovers identity

The exact generation strategy must remain consistent across aggregate roots.

14. Strongly Typed Identifiers

If strongly typed identifiers are used in the domain, Infrastructure must provide explicit EF Core conversions.

Conceptually:

CollectionId
     ↓
Value Converter
     ↓
uuid

The database must not dictate the domain identifier abstraction.

15. Aggregate Root Table

Each persisted aggregate root should have a dedicated primary table.

Conceptually:

collections
------------
id
...

The aggregate root table contains the persistent state required to reconstruct the root.

It must not contain arbitrary fields belonging to unrelated aggregates.

16. Child Entity Tables

Child entities inside an aggregate may be represented by separate tables.

Example:

collections
collection_items

with:

collection_items.collection_id
        ↓
collections.id

The foreign key enforces relational ownership.

The application/domain layer determines whether the child can exist independently.

17. Owned Value Objects

Simple value objects should preferably be persisted as columns of their owning record where relationally appropriate.

Example:

Domain:
Name


Persistence:
collections.name

For a composite value object:

Domain:
ExternalReference
    Provider
    Value


Persistence:
provider
external_value

The mapping remains infrastructure-owned.

18. Value Object Table Strategy

A value object should receive its own table only when there is a clear persistence reason.

Possible reasons include:

independent cardinality,
large collection size,
independent querying,
database normalization requirements,
lifecycle requirements.

A value object should not become an entity merely because EF Core makes separate mapping convenient.

19. Enumerations

Domain enumerations should normally be persisted using a stable representation.

The preferred initial strategy is usually:

Domain enum
    ↓
integer or string
    ↓
PostgreSQL

The choice must depend on whether enum values are internal implementation details or stable persisted business codes.

Where persisted values may evolve independently, explicit string/code representations are preferable.

20. Date and Time

Persisted instants should use UTC.

Preferred conceptual mapping:

DateTimeOffset / UTC instant
        ↓
PostgreSQL timestamptz

The application must avoid persisting ambiguous local timestamps for system events.

Examples include:

created timestamps,
updated timestamps,
synchronization timestamps,
external event timestamps.
21. Created and Updated Timestamps

Persisted aggregate records should support temporal auditing where required.

Typical fields:

created_at
updated_at

These fields are technical metadata unless the domain explicitly gives them business meaning.

The application must distinguish:

technical persistence timestamp

from:

business event timestamp
22. Concurrency Column

Where optimistic concurrency is required, the persistence model should contain a concurrency token.

Conceptually:

version

or an equivalent PostgreSQL-compatible representation.

The concurrency token belongs to Infrastructure.

It must not become a domain concept unless version semantics are genuinely part of the business model.

23. Concurrency Mapping

The mapping is:

Domain Aggregate
        │
        ▼
Persistence Record
        │
        └── version
               │
               ▼
          PostgreSQL

On update:

Expected Version
      ↓
UPDATE
      ↓
0 rows affected
      ↓
Concurrency conflict

The Infrastructure layer translates the conflict into an application-level outcome.

24. Foreign Key Strategy

Foreign keys are mandatory for relationships where database referential integrity is required.

They protect against:

orphan records,
invalid references,
accidental deletion of referenced records.

However:

A foreign key is a database integrity mechanism, not a substitute for an aggregate boundary.

25. Foreign Keys Across Aggregates

Cross-aggregate references should normally be represented using identifiers rather than persistence navigation graphs.

Example:

Aggregate A
    external_id → Aggregate B identifier

rather than forcing EF Core to construct a complete object graph.

This keeps aggregate boundaries explicit.

26. Cross-Aggregate Navigation Properties

Cross-aggregate EF Core navigation properties should be avoided unless there is a strong query-specific reason.

The preferred pattern is:

Aggregate A
    BId

rather than:

Aggregate A
    B

This reduces accidental coupling and aggregate over-fetching.

27. Delete Behavior

Delete behavior must be explicit.

Default cascade behavior should not be accepted blindly.

Possible strategies:

Cascade
Restrict
NoAction
SetNull
Explicit application deletion

The correct choice depends on aggregate ownership.

28. Aggregate Child Deletion

If a child belongs completely to an aggregate and cannot exist independently, cascade deletion may be appropriate.

Conceptually:

Collection
    │
    └── CollectionItem

Deleting the collection may delete its items.

This must be explicitly configured rather than relying on EF Core conventions.

29. Cross-Aggregate Deletion

Cross-aggregate relationships should normally use restrictive deletion semantics.

Example:

Aggregate A
    │
    └── references Aggregate B

Deleting B should not automatically delete A unless this is an explicit domain rule.

30. Unique Constraints

Database uniqueness must be enforced where the domain requires globally unique persistence state.

Examples may include:

external_provider + external_identifier

or:

normalized_name

when uniqueness is a true business or technical invariant.

31. Composite Unique Constraints

Composite uniqueness should be represented explicitly.

Example:

UNIQUE (
    provider,
    external_identifier
)

This is useful for external integration identities.

The domain/application layer must still translate uniqueness violations into meaningful behavior.

32. Indexing Strategy

Indexes should be derived from actual application query patterns.

Indexes should generally exist for:

primary keys,
foreign keys where useful,
unique constraints,
frequent lookup fields,
external identifiers,
synchronization queries,
pagination keys.

Indexes must not be created speculatively.

33. Query-Driven Index Design

The workflow is:

Application Use Case
        ↓
Query Pattern
        ↓
Persistence Query
        ↓
Index Requirement
        ↓
Database Index

The database schema must therefore remain traceable to application behavior.

34. External Identifier Indexes

External integration identifiers are likely to be frequently queried.

Where applicable:

(provider, external_identifier)

should be indexed and potentially declared unique.

This supports:

synchronization,
deduplication,
provider lookups,
external reference resolution.
35. Pagination Indexes

If queries use:

ORDER BY created_at

and pagination, an index supporting that query should be considered.

For large datasets, keyset pagination may require a compound index aligned with:

created_at
id

The final index should be derived from measured query behavior.

36. Soft Delete

Soft deletion is not enabled globally by default.

A table should only contain:

deleted_at

if the domain/application actually requires recoverable deletion or historical retention.

Soft delete should not be introduced merely to avoid physical deletion.

37. Hard Delete

Hard deletion is acceptable when:

the aggregate has no retention requirement,
historical preservation is unnecessary,
deletion is allowed by the domain,
dependent records are handled correctly.

The deletion policy must be explicit per aggregate.

38. Audit History

Audit requirements must be distinguished from aggregate state.

If CollectionHub later requires:

who changed what
when it changed
previous value
new value

this should be designed as an explicit audit capability.

It must not be approximated by adding arbitrary columns to every table.

39. Integration Persistence

External integration state may require dedicated persistence.

Examples:

external_references
synchronization_state
integration_errors

These tables belong to Infrastructure.

They must not be mistaken for domain aggregates unless the domain explicitly models them.

40. External Reference Model

An external reference should conceptually contain:

id
aggregate_id
provider
external_identifier
created_at
updated_at

where the exact fields depend on the integration requirements.

A uniqueness rule may be:

(provider, external_identifier)

when the external provider guarantees uniqueness.

41. Synchronization State

If synchronization with external systems is required, synchronization metadata should remain separate from core aggregate state where possible.

Potential fields include:

last_synchronized_at
synchronization_status
last_error
external_version

These are technical integration concerns unless explicitly modeled by the domain.

42. JSONB Persistence

JSONB may be used for flexible provider-specific metadata.

Example:

provider_metadata jsonb

However:

business-critical fields should remain relational,
frequently queried fields should be indexed appropriately,
JSONB must not become a dumping ground for the domain model.
43. Nullability

Database nullability must represent whether data is actually optional.

The mapping should distinguish:

Required business property

from:

Optional property

A nullable database column must not be introduced merely because the value is difficult to initialize.

44. Maximum Lengths

String fields should have explicit maximum lengths where meaningful.

Examples:

name
external_identifier
provider
description

Unlimited strings should only be used when genuinely required.

This protects storage and prevents invalid values from reaching the database.

45. Text Fields

Large textual content may use PostgreSQL text.

Short bounded values should use explicit length constraints.

Example conceptual policy:

Short identifier → varchar(n)
Large description → text

The exact limits must derive from domain requirements.

46. Database Check Constraints

Database check constraints may be used for simple technical invariants.

Examples:

quantity >= 0

or:

status in (...)

However, complex business logic must remain in the domain.

A database check constraint is a final safety boundary, not the primary business-rule implementation.

47. Normalization

The schema should use normal relational modeling by default.

Normalization is particularly important for:

identity,
relationships,
external references,
searchable attributes,
constrained data.

Denormalization should only be introduced when performance evidence justifies it.

48. Aggregate Snapshot Strategy

The database should persist the current aggregate state.

Event sourcing is not part of the initial persistence architecture.

Therefore:

Aggregate State
      ↓
Relational Persistence

rather than:

Domain Event Stream
      ↓
Aggregate Reconstruction

Event sourcing may be reconsidered only if explicit requirements emerge.

49. Domain Event Persistence

Domain events are not automatically stored in the database.

If durable asynchronous publication becomes necessary, the architecture may introduce an Outbox pattern.

Until then, domain events remain domain/application mechanisms.

50. Transactional Consistency

A single aggregate modification should be atomic.

Where a use case modifies multiple aggregates, transaction requirements must be explicitly evaluated.

The persistence layer must not assume:

one HTTP request = always one transaction

Instead:

Use-case consistency requirement
        ↓
Transaction boundary
51. Multiple Aggregate Transactions

Transactions involving multiple aggregates are allowed when a use case genuinely requires atomic persistence.

However, frequent multi-aggregate transactions should trigger architectural review.

They may indicate that:

aggregate boundaries are wrong,
orchestration is too tightly coupled,
eventual consistency should be considered.
52. Transaction Boundary

The preferred sequence is:

Application Use Case
        ↓
Load required state
        ↓
Execute domain behavior
        ↓
Persist changes
        ↓
Commit transaction
        ↓
Publish/execute side effects according to architecture

External network calls should not normally be held open inside database transactions.

53. External Integrations and Transactions

Never assume that a PostgreSQL transaction can atomically include an external HTTP call.

This is not supported:

BEGIN
  ↓
Database update
  ↓
HTTP call
  ↓
COMMIT

because the external system cannot participate in the same database transaction.

Where atomicity between persistence and external publication is required, use an Outbox-style architecture.

54. Migration Versioning

Database migrations are version-controlled artifacts.

Each migration must represent a deliberate schema evolution.

Example:

Migration 001
Migration 002
Migration 003
...

Migrations must not be edited after they have been applied to shared environments except under an explicit controlled procedure.

55. Migration Review

Every migration should be reviewed for:

locking behavior,
data loss,
index creation cost,
compatibility,
rollback implications,
production execution time.

Schema changes are production changes.

They must receive the same architectural discipline as application code.

56. Expand/Contract Strategy

For incompatible schema changes:

Phase 1
Add new structure


Phase 2
Application supports old + new


Phase 3
Migrate existing data


Phase 4
Application uses new structure


Phase 5
Remove old structure

This minimizes deployment coupling.

57. Seed Data

Seed data must be classified.

Reference Data

Stable technical/business reference data may be seeded through migrations or controlled initialization.

Development Data

Development-only data must not be embedded into production migrations.

Test Data

Test data belongs to test fixtures.

This prevents environment-specific records from leaking into production.

58. Environment Separation

The following databases must be logically separated:

Development
Test
Staging
Production

No environment should accidentally point to another environment's database.

59. Database Credentials

Connection strings must be provided through configuration/secrets management.

They must not be committed to source control.

The persistence layer receives configuration through the Infrastructure configuration boundary.

60. Schema Mapping Responsibility

EF Core mappings are responsible for:

table mapping,
column mapping,
relationship mapping,
constraints,
indexes,
conversions,
concurrency configuration.

Domain classes are not responsible for database configuration.

61. Configuration Class Organization

Mappings should follow:

Persistence/
└── Configurations/
    ├── CollectionConfiguration.cs
    ├── CollectionItemConfiguration.cs
    └── ...

Each mapping class should have one clear persistence responsibility.

Large global mapping classes should be avoided.

62. DbContext Configuration

CollectionHubDbContext should:

expose persistence sets where necessary,
apply entity configurations,
configure database-specific behavior,
manage persistence state.

It should not contain large amounts of business-specific mapping logic.

Mappings belong in dedicated configuration classes.

63. Automatic Configuration Discovery

EF Core configuration discovery may be used to apply all IEntityTypeConfiguration<T> implementations from the Infrastructure assembly.

This reduces manual registration and keeps mappings modular.

The mechanism must remain Infrastructure-owned.

64. Repository Mapping Boundary

The repository performs:

Database Record
      ↓
Persistence Mapping
      ↓
Domain Aggregate

and:

Domain Aggregate
      ↓
Persistence Mapping
      ↓
Database Record

The mapping code must not embed business decisions.

65. Mapping Validation

Persistence integration tests should verify that:

all required fields persist,
value objects round-trip correctly,
relationships reconstruct correctly,
identifiers remain stable,
concurrency metadata behaves correctly,
database constraints are enforced.
66. Database Schema Traceability

Every important table should be traceable to:

Domain Concept
      ↓
Aggregate
      ↓
Application Use Case
      ↓
Persistence Model
      ↓
Database Table

If a table cannot be explained through this chain, its architectural purpose must be reviewed.

67. Technical Tables

Technical tables may exist without direct domain equivalents.

Examples:

outbox_messages
integration_sync_state
audit_entries

Such tables must explicitly document:

why they exist,
who owns them,
which process consumes them,
retention policy,
cleanup strategy.
68. Database Boundary Rules
DBR-01

The database is owned by CollectionHub.

DBR-02

External systems cannot directly modify CollectionHub tables.

DBR-03

Domain concepts determine aggregate boundaries.

DBR-04

Foreign keys do not define aggregate boundaries.

DBR-05

EF Core mappings remain in Infrastructure.

DBR-06

Persistence models remain in Infrastructure.

DBR-07

Business invariants remain in Domain.

DBR-08

Technical constraints may be reinforced by PostgreSQL.

DBR-09

Migrations are version-controlled.

DBR-10

Destructive schema changes require explicit review.

69. Initial Schema Baseline

At this architectural stage, the exact final table catalogue must remain traceable to the finalized domain concepts and application use cases.

The schema baseline therefore consists of the following categories:

Aggregate Root Tables
Child Entity Tables
External Reference Tables
Integration State Tables
Technical Tables

Concrete table creation should occur only after each persistent domain concept has been explicitly classified.

This avoids prematurely inventing tables that have no architectural justification.

70. Persistence Mapping Matrix

The following matrix is the governing mapping model:

Domain Concept	Persistence Representation	Database Representation
Aggregate Root	Persistence Root Model	Root Table
Entity inside Aggregate	Persistence Entity Model	Child Table or owned structure
Value Object	EF conversion / owned mapping	Column(s)
Strong Identifier	Value Converter	uuid or selected primitive
Domain Enum	Converter	Stable primitive/code
Aggregate Reference	Identifier	Foreign Key where required
Domain Event	Not persisted by default	No table
External Reference	Infrastructure model	Dedicated table where required
Sync Metadata	Infrastructure model	Technical table/columns
Concurrency	Technical metadata	Version/concurrency column
71. Anti-Patterns Explicitly Rejected

The following persistence designs are rejected:

71.1 Domain Entities as EF Models

Rejected because ORM concerns leak into the domain.

71.2 Generic Repository for Every Table

Rejected because it hides aggregate semantics.

71.3 One Repository per Database Table

Rejected because repositories should align with aggregate boundaries.

71.4 Database-First Domain Design

Rejected because the domain model is the architectural source of business meaning.

71.5 Universal JSONB Storage

Rejected because it sacrifices relational integrity and queryability.

71.6 Automatic Cascade Everywhere

Rejected because deletion semantics are domain-dependent.

71.7 Global Soft Delete

Rejected because retention requirements are aggregate-specific.

71.8 Event Sourcing by Default

Rejected because current requirements do not justify its complexity.

71.9 Premature Read Database

Rejected because current architecture does not require CQRS infrastructure.

72. Performance Principles

The initial schema must prioritize:

Correct aggregate persistence.
Referential integrity.
Appropriate indexing.
Efficient query patterns.
Predictable transaction behavior.
Operational simplicity.

Performance optimizations must be based on actual workload evidence.

73. Security Principles

The database layer must enforce:

least privilege,
secure credentials,
encrypted connections where required,
environment separation,
restricted administrative access.

Sensitive application data must not be written to logs merely because EF Core debugging is enabled.

74. Backup and Recovery Requirements

The operational database architecture must eventually define:

backup frequency,
retention,
point-in-time recovery,
recovery point objective,
recovery time objective,
restoration testing.

These are operational requirements rather than domain persistence mappings.

75. Acceptance Criteria

The database schema design is considered established when:

 Database ownership is defined.
 PostgreSQL schema strategy is defined.
 Aggregate-to-table strategy is defined.
 Identifier strategy is defined.
 Value-object strategy is defined.
 Foreign-key strategy is defined.
 Delete behavior principles are defined.
 Unique constraints are defined conceptually.
 Indexing strategy is defined.
 Concurrency mapping is defined.
 Temporal persistence is defined.
 External-reference persistence is defined.
 Migration strategy is defined.
 Schema traceability is defined.
 Database boundary rules are defined.
 Concrete SQL schema has been generated.
 Concrete EF Core configurations have been implemented.

The final two items belong to the implementation phase and are intentionally outside this architectural document.

76. Final Architecture Decision

CollectionHub will use a relational PostgreSQL schema derived from the domain aggregate model rather than from database-first modeling.

The persistence architecture is:

                    DOMAIN
                      │
                      │ aggregate model
                      ▼
                APPLICATION
                      │
                      │ persistence contract
                      ▼
                INFRASTRUCTURE
                      │
             ┌────────┴─────────┐
             │                  │
       Repository          Persistence
       Implementation        Mapping
             │                  │
             └────────┬─────────┘
                      ▼
                   EF Core
                      │
                      ▼
                 PostgreSQL

The governing principle is:

The database must faithfully persist the domain model without becoming the owner of the domain model.

The schema will therefore be:

relational,
explicit,
constraint-aware,
aggregate-oriented,
migration-controlled,
PostgreSQL-native where justified,
isolated behind Infrastructure.

Database architecture: ESTABLISHED.

Domain-to-persistence mapping strategy: ESTABLISHED.

PostgreSQL schema strategy: ESTABLISHED.

Next step: 41_DATABASE_SCHEMA_DEFINITION_AND_TABLE_CATALOG.md