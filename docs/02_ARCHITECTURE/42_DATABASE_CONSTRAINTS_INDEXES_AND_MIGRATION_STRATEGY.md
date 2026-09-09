# 42 — Database Constraints, Indexes and Migration Strategy

## 1. Purpose

This document defines the physical persistence rules required to implement the CollectionHub relational database consistently with the domain model, persistence architecture, EF Core architecture, and database schema previously defined.

It establishes the implementation contract for:

- primary keys;
- foreign keys;
- unique constraints;
- check constraints;
- indexes;
- delete behaviors;
- optimistic concurrency;
- generated values;
- value conversions;
- SQL Server-specific persistence decisions;
- EF Core migration strategy;
- seed/reference data;
- schema evolution;
- deployment;
- rollback;
- migration verification.

The objective is to ensure that the database is not merely structurally compatible with the domain model, but actively protects the relational invariants that can and should be enforced at persistence level.

---

# 2. Architectural Position

This document sits between the logical database definition and the future concrete implementation.

The persistence architecture is therefore:

```text
Domain Model
     |
     v
Persistence Mapping
     |
     v
Logical Database Schema
     |
     v
Database Constraints + Indexes + Migration Strategy
     |
     v
EF Core Model Configuration
     |
     v
SQL Server Database
```

The database must enforce structural integrity while the domain model remains responsible for business behavior.

---

# 3. Governing Principles

## 3.1 Database constraints protect persistence invariants

Constraints should be used whenever the database can reliably enforce a rule independently of application execution flow.

Examples:

- primary-key uniqueness;
- foreign-key integrity;
- mandatory values;
- unconditional uniqueness;
- basic value ranges.

---

## 3.2 Business behavior remains outside the database

The database must not become the primary execution environment for domain behavior.

The following remain application/domain responsibilities:

- aggregate behavior;
- business workflows;
- complex state transitions;
- cross-aggregate policies;
- authorization;
- domain services;
- external integration decisions.

---

## 3.3 Constraints are defense in depth

A domain invariant should ideally be enforced at the earliest appropriate boundary.

For example:

```text
Domain
  -> validates business rule

Application
  -> coordinates operation

Database
  -> guarantees relational integrity
```

Database constraints do not replace domain validation.

---

# 4. Primary Key Strategy

## 4.1 Default identifier

Persisted entities use technical GUID identifiers where the domain model requires generated technical identity.

Default representation:

```text
uniqueidentifier NOT NULL
```

Primary keys are immutable.

---

## 4.2 Aggregate roots

Aggregate roots must have independent primary keys.

Examples:

```text
Collections.Id
ItemTypes.Id
Categories.Id
Tags.Id
ExternalSources.Id
```

The primary key must not depend on mutable business attributes.

---

## 4.3 Composite keys

Composite keys are permitted for pure association entities.

The principal example is:

```text
ItemTags
---------
ItemId
TagId
```

with:

```text
PK_ItemTags (ItemId, TagId)
```

Composite keys must not be introduced merely to encode business logic that belongs in the domain.

---

# 5. Primary Key Naming

The naming convention is:

```text
PK_<TableName>
```

Examples:

```text
PK_Collections
PK_CollectionItems
PK_ItemTypes
PK_Categories
PK_Tags
PK_ItemTags
PK_ItemMetadata
PK_ExternalSources
PK_ExternalReferences
PK_SyncStates
PK_SyncOperations
```

For composite keys, the column order must follow the most common lookup direction.

---

# 6. Foreign Key Strategy

Foreign keys must exist for all relationships that require database-level referential integrity.

The standard naming convention is:

```text
FK_<DependentTable>_<PrincipalTable>
```

Examples:

```text
FK_CollectionItems_Collections
FK_CollectionItems_ItemTypes
FK_CollectionItems_Categories
FK_Categories_Categories
FK_ItemTags_CollectionItems
FK_ItemTags_Tags
FK_ItemMetadata_CollectionItems
FK_ExternalReferences_ExternalSources
FK_SyncStates_ExternalSources
FK_SyncOperations_ExternalSources
```

---

# 7. Foreign Key Matrix

| Dependent | Principal | FK | Nullable | Default Delete Behavior |
|---|---|---|---:|---|
| `CollectionItems` | `Collections` | `CollectionId` | No | Cascade or explicit aggregate deletion |
| `CollectionItems` | `ItemTypes` | `ItemTypeId` | Yes | Restrict |
| `CollectionItems` | `Categories` | `CategoryId` | Yes | Restrict |
| `Categories` | `Categories` | `ParentCategoryId` | Yes | Restrict |
| `ItemTags` | `CollectionItems` | `ItemId` | No | Cascade |
| `ItemTags` | `Tags` | `TagId` | No | Restrict |
| `ItemMetadata` | `CollectionItems` | `ItemId` | No | Cascade |
| `ExternalReferences` | `ExternalSources` | `ExternalSourceId` | No | Restrict |
| `SyncStates` | `ExternalSources` | `ExternalSourceId` | No | Restrict |
| `SyncOperations` | `ExternalSources` | `ExternalSourceId` | No | Restrict |

The final EF Core delete behavior must be explicitly configured.

Provider defaults must never determine domain behavior accidentally.

---

# 8. Delete Behavior Strategy

## 8.1 Aggregate-owned entities

Where an entity is strictly owned by an aggregate root and cannot meaningfully exist independently, cascade deletion is acceptable.

Example:

```text
Collection
    |
    +--- CollectionItems
```

If the domain confirms that `CollectionItem` has no independent lifecycle, deleting a collection may cascade to its items.

---

## 8.2 Reference data

Reference concepts such as:

- `ItemTypes`;
- `Categories`;
- `Tags`;

should normally not cascade into collection data.

Deleting a classification value that is still referenced must be rejected or prevented by the application.

---

## 8.3 Integration data

External-source deletion must not automatically destroy synchronization history.

Therefore:

```text
ExternalSource
    |
    +--- ExternalReferences
    +--- SyncStates
    +--- SyncOperations
```

should use restrictive deletion semantics.

An external source must be deactivated rather than casually deleted when historical integration information must be retained.

---

# 9. Unique Constraints

Unique constraints enforce unconditional uniqueness requirements.

Initial unique constraints:

```text
UQ_ItemTypes_Code
UQ_Categories_Code
UQ_Tags_Name
UQ_ExternalSources_Code
UQ_SyncStates_Source_Scope
```

---

# 10. Collection Name Uniqueness

Collection name uniqueness remains dependent on the final domain decision.

If collections must have globally unique names:

```text
UQ_Collections_Name
```

must be created.

If duplicate names are allowed, only a non-unique search index should be created.

The database must not impose uniqueness that the domain does not require.

---

# 11. External Reference Uniqueness

The preferred uniqueness model is:

```text
UQ_ExternalReferences_Source_Type_ExternalId
(
    ExternalSourceId,
    EntityType,
    ExternalId
)
```

This allows an external system to use the same identifier in different entity namespaces.

If the external provider guarantees global identifier uniqueness, the simpler constraint may be used:

```text
UQ_ExternalReferences_Source_ExternalId
(
    ExternalSourceId,
    ExternalId
)
```

The implementation must select one strategy based on actual integration contracts.

---

# 12. Check Constraints

Check constraints should be used for simple relationally expressible invariants.

They must not attempt to reproduce complex domain behavior.

Recommended constraints include:

### Categories

```text
ParentCategoryId IS NULL
OR ParentCategoryId <> Id
```

### Synchronization counters

```text
ItemsProcessed >= 0
ItemsSucceeded >= 0
ItemsFailed >= 0
```

### Synchronization counters relationship

Where appropriate:

```text
ItemsSucceeded + ItemsFailed <= ItemsProcessed
```

The latter must only be enforced if partial processing semantics guarantee this invariant.

---

# 13. Status Constraints

Database-level status constraints should be introduced only when the permitted state values are stable and tightly controlled.

For example:

```text
Status IN ('Pending', 'Running', 'Completed', 'Failed')
```

should not be embedded prematurely if the application is expected to evolve the status model frequently.

The preferred approach is:

1. domain/application validates allowed states;
2. database stores stable representation;
3. database-level check is introduced only when the status set is sufficiently stable.

---

# 14. Nullability Strategy

Nullability must correspond to persistence semantics.

Required values:

```text
NOT NULL
```

Optional values:

```text
NULL
```

Examples:

```text
CollectionItems.CollectionId -> NOT NULL
CollectionItems.ItemTypeId -> NULL
CollectionItems.CategoryId -> NULL
SyncOperations.CompletedAtUtc -> NULL
SyncOperations.ErrorMessage -> NULL
```

The database must not use nullable columns to represent invalid domain states.

---

# 15. Indexing Principles

Indexes must support actual access patterns.

Every index must have a justification based on at least one of:

- primary lookup;
- foreign-key traversal;
- filtering;
- sorting;
- uniqueness;
- synchronization processing;
- operational diagnostics.

Indexes must be reviewed after representative query workloads become available.

---

# 16. Mandatory Indexes

Primary keys automatically provide indexes.

Additional indexes should initially include:

```text
IX_CollectionItems_CollectionId
IX_CollectionItems_ItemTypeId
IX_CollectionItems_CategoryId
IX_Categories_ParentCategoryId
IX_ItemTags_TagId
IX_ExternalReferences_Entity
IX_SyncStates_ExternalSourceId
IX_SyncOperations_ExternalSourceId
IX_SyncOperations_Status
IX_SyncOperations_StartedAtUtc
```

---

# 17. Composite Indexes

Composite indexes must reflect actual query predicates.

Initial candidates include:

```text
IX_CollectionItems_CollectionId_Name
```

This supports common collection browsing scenarios where items are retrieved for a collection and ordered or filtered by name.

Composite indexes must not be created simply because two columns are commonly used together in the schema.

---

# 18. Index Column Ordering

Index column ordering must follow expected query selectivity and predicate usage.

For example:

```text
(CollectionId, Name)
```

is appropriate for queries shaped like:

```text
WHERE CollectionId = @collectionId
ORDER BY Name
```

The opposite ordering would not provide the same benefit.

Query plans must be validated against representative data.

---

# 19. Foreign Key Indexes

Foreign-key columns should generally have indexes when they are:

- frequently traversed;
- used for filtering;
- involved in joins;
- used in synchronization processing.

This is especially important for:

```text
CollectionItems.CollectionId
CollectionItems.ItemTypeId
CollectionItems.CategoryId
ItemTags.TagId
ExternalReferences.ExternalSourceId
SyncStates.ExternalSourceId
SyncOperations.ExternalSourceId
```

---

# 20. Unique Indexes Versus Unique Constraints

Where a unique requirement exists, EF Core may represent it through a unique index or a unique constraint depending on the mapping and provider.

The architectural requirement is uniqueness, not the exact SQL Server construct.

The generated migration must be reviewed to ensure the resulting database semantics are correct.

---

# 21. Concurrency Strategy

CollectionHub uses optimistic concurrency.

The preferred SQL Server mechanism is:

```text
rowversion
```

mapped to:

```text
Version
```

in the persistence model.

Affected tables include:

```text
Collections
CollectionItems
ItemMetadata
SyncStates
```

---

# 22. Concurrency Semantics

A successful update must verify that the row version read by the application still matches the persisted value.

If another transaction modifies the same row:

```text
UPDATE ... WHERE Id = @id AND Version = @originalVersion
```

must affect zero rows.

EF Core must surface this as a concurrency conflict.

The application layer must translate that conflict into a meaningful application result.

---

# 23. Generated Values

Database-generated values must be explicit.

Typical generated values include:

- `rowversion`;
- database identity where integer identifiers are used;
- default timestamps where appropriate.

The preferred timestamp strategy is application-generated UTC timestamps unless a database-generated timestamp is specifically required.

This keeps timestamp semantics consistent across application and database operations.

---

# 24. Timestamp Strategy

All timestamps use UTC.

Standard names:

```text
CreatedAtUtc
UpdatedAtUtc
StartedAtUtc
CompletedAtUtc
LastSuccessfulSyncAtUtc
LastAttemptedSyncAtUtc
```

No persisted domain timestamp should use local server time.

---

# 25. EF Core Configuration Strategy

Persistence configuration must be separated from domain classes.

The preferred structure is:

```text
Infrastructure
    Persistence
        Configurations
            CollectionConfiguration
            CollectionItemConfiguration
            ItemTypeConfiguration
            CategoryConfiguration
            TagConfiguration
            ItemTagConfiguration
            ItemMetadataConfiguration
            ExternalSourceConfiguration
            ExternalReferenceConfiguration
            SyncStateConfiguration
            SyncOperationConfiguration
```

Each configuration should implement:

```text
IEntityTypeConfiguration<T>
```

where appropriate.

---

# 26. Fluent API as Source of Persistence Truth

EF Core Fluent API should define:

- table names;
- schema;
- primary keys;
- composite keys;
- foreign keys;
- delete behavior;
- required/optional properties;
- maximum lengths;
- indexes;
- unique constraints;
- concurrency;
- value conversions;
- database-generated values.

Data annotations should not become the primary persistence configuration mechanism.

---

# 27. Maximum String Lengths

String lengths must match the logical schema.

Examples:

```text
Code            nvarchar(100)
Name            nvarchar(200)
Description     nvarchar(2000)
ExternalId      nvarchar(300)
ErrorCode       nvarchar(100)
ErrorMessage    nvarchar(2000)
```

Unbounded strings must be used only when the domain genuinely permits large content.

---

# 28. Value Conversion Strategy

Value objects and domain-specific types must be converted explicitly.

Potential conversions include:

```text
StronglyTypedId -> uniqueidentifier
Enumeration     -> stable database representation
ValueObject     -> scalar columns
```

Conversions must preserve:

- nullability;
- equality semantics;
- serialization stability;
- migration compatibility.

A conversion must never silently discard domain information.

---

# 29. Enumeration Conversion

Where enums are persisted as strings, the stored values must be stable.

Example:

```text
Pending
Running
Completed
Failed
```

Renaming an enum member therefore becomes a database migration concern.

Where numeric persistence is selected, explicit compatibility rules must prevent accidental value reassignment.

---

# 30. JSON Conversion

JSON metadata must use explicit serialization configuration.

The following must remain stable:

- property naming;
- null handling;
- enum representation;
- version compatibility.

JSON schema evolution must be handled deliberately.

---

# 31. Database Schema Configuration

The default schema is:

```text
dbo
```

Integration-specific tables use:

```text
integration
```

Technical metadata uses:

```text
system
```

Schema creation must be migration-controlled.

---

# 32. EF Core Migration Strategy

EF Core migrations are the authoritative mechanism for database evolution.

The migration lifecycle is:

```text
Domain/Persistence Model Change
            |
            v
EF Core Model Change
            |
            v
Migration Generation
            |
            v
Migration Review
            |
            v
Automated Validation
            |
            v
Deployment
```

No manual production schema changes should bypass this process.

---

# 33. Migration Naming

Migration names must describe intent.

Examples:

```text
InitialCollectionSchema
AddExternalIntegrationPersistence
AddCollectionItemMetadata
AddSynchronizationState
AddConcurrencyTokens
```

Names must not be generic:

```text
Update
Fix
Change
Migration1
```

---

# 34. Migration Review

Every migration must be reviewed before deployment.

The review must inspect:

- generated SQL;
- table creation;
- column changes;
- index creation;
- foreign keys;
- delete behavior;
- data movement;
- default values;
- destructive operations;
- locking implications.

Generating a migration successfully is not sufficient evidence that it is safe.

---

# 35. Destructive Migrations

Potentially destructive operations require explicit review.

Examples:

- dropping columns;
- dropping tables;
- removing constraints;
- changing column types;
- truncating data;
- renaming columns without data preservation.

The preferred strategy is:

```text
Expand
  ->
Migrate
  ->
Contract
```

rather than immediate destructive replacement.

---

# 36. Expand-and-Contract Strategy

For breaking schema changes:

### Phase 1 — Expand

Introduce new structures without removing the old ones.

### Phase 2 — Migrate

Move or synchronize data.

### Phase 3 — Switch

Update application code to use the new representation.

### Phase 4 — Contract

Remove obsolete structures only after compatibility has been verified.

This strategy supports safer deployments and reduces downtime.

---

# 37. Migration Transaction Strategy

Migrations should execute transactionally whenever SQL Server and the migration operation support transactional execution safely.

Large data migrations may require specialized strategies where a single transaction would create unacceptable:

- locking;
- transaction log growth;
- execution time;
- blocking.

Such migrations require explicit operational planning.

---

# 38. Seed Data Strategy

Reference data must be separated into two categories.

## 38.1 System-owned reference data

Examples:

- predefined item types;
- system-defined categories;
- system-defined integration sources.

These may be managed through controlled EF Core seed mechanisms.

## 38.2 User-managed data

User-created collections, tags, categories, and items must not be overwritten by migrations.

Migration logic must never assume that existing application data matches the development database state.

---

# 39. Seed Identifier Stability

Seeded reference entities must use stable identifiers where their identity matters.

Changing a seeded identifier can create:

- foreign-key failures;
- duplicate reference data;
- orphaned records;
- migration complexity.

Seed identity must therefore be treated as part of the persistence contract.

---

# 40. Migration Environment Strategy

Migrations must be tested in multiple environments:

```text
Development
     |
     v
CI/Test
     |
     v
Staging
     |
     v
Production
```

The same migration artifact must be applicable across environments.

Environment-specific configuration must not alter schema semantics.

---

# 41. Production Migration Strategy

Production migration execution must be controlled.

Preferred process:

1. Build application artifact.
2. Generate migration SQL.
3. Review migration SQL.
4. Validate against staging.
5. Create backup/recovery point where appropriate.
6. Apply migration.
7. Verify schema.
8. Execute smoke tests.
9. Monitor application behavior.

Automatic startup migrations in the production application process are discouraged.

---

# 42. Migration Locking and Availability

Before deploying a migration, evaluate:

- table size;
- affected indexes;
- lock duration;
- concurrent traffic;
- transaction log growth;
- expected downtime;
- SQL Server execution characteristics.

Schema changes affecting large production tables may require online or staged strategies where supported.

---

# 43. Rollback Strategy

Rollback must be planned per migration.

There are two primary strategies.

## 43.1 Reverse migration

Applicable when the migration can be safely reversed without data loss.

## 43.2 Forward corrective migration

Preferred when reversing would destroy data or create inconsistent state.

Example:

```text
Migration A
    |
    v
Production
    |
    v
Problem detected
    |
    v
Migration B
    |
    v
Corrected schema
```

Production rollback must never blindly execute `Down()` if doing so would cause irreversible data loss.

---

# 44. Backup and Recovery

Database migrations must operate within the established backup and recovery strategy.

Before destructive migrations, the team must know:

- last verified backup;
- recovery point;
- restoration procedure;
- expected recovery time;
- responsible operator.

A backup that has never been tested through restoration is not considered sufficient recovery assurance.

---

# 45. Migration Failure Handling

If a migration fails:

1. stop subsequent deployment stages;
2. preserve logs;
3. determine whether the migration transaction rolled back;
4. inspect database state;
5. verify migration history;
6. determine whether corrective migration is required;
7. restore only when necessary;
8. document the incident.

The application must not automatically attempt repeated destructive migrations without operator control.

---

# 46. CI/CD Validation

Database migrations must be validated automatically.

The CI pipeline should verify:

- model builds successfully;
- migrations can be generated;
- migrations apply to an empty database;
- migrations apply to representative existing data;
- migration history is consistent;
- schema matches expected EF Core model;
- integration tests pass against the resulting database.

---

# 47. Migration Drift Detection

The project must detect schema drift between:

```text
EF Core Model
       |
       v
Expected Database Schema
       |
       v
Actual Database Schema
```

Manual database modifications must be considered drift.

Production drift must be investigated and reconciled before additional migrations are applied when necessary.

---

# 48. Integration Test Database

Persistence integration tests should execute against a database provider that behaves sufficiently like production.

The preferred strategy is a real SQL Server-compatible database for persistence tests rather than relying exclusively on:

- EF Core InMemory provider;
- mocks;
- SQLite substitutions where SQL Server semantics matter.

This is particularly important for:

- foreign keys;
- concurrency;
- indexes;
- transactions;
- SQL Server-specific behavior;
- migrations.

---

# 49. Migration Testing Matrix

| Scenario | Required |
|---|---:|
| Empty database → latest schema | Yes |
| Previous release → latest schema | Yes |
| Representative production dataset → latest schema | Yes |
| Migration with existing foreign-key data | Yes |
| Concurrency configuration | Yes |
| Unique constraint violation | Yes |
| Foreign-key violation | Yes |
| Seed/reference data preservation | Yes |
| Migration failure handling | Yes |
| Rollback/recovery procedure | Yes |

---

# 50. Database Constraint Testing

The test suite must explicitly verify that important database constraints behave as expected.

Examples:

### Duplicate ItemType code

Expected:

```text
Database rejects duplicate code.
```

### Duplicate Tag name

Expected:

```text
Database rejects duplicate tag.
```

### Invalid collection reference

Expected:

```text
Database rejects orphan CollectionItem.
```

### Duplicate ItemTag

Expected:

```text
Database rejects duplicate association.
```

### Concurrent update

Expected:

```text
One update succeeds.
Concurrent stale update fails with concurrency conflict.
```

---

# 51. Performance Validation

Index decisions must be validated against representative data volumes.

The validation process should examine:

- query execution plans;
- logical reads;
- CPU cost;
- duration;
- index usage;
- write overhead.

Indexes should be removed or redesigned when their operational cost exceeds their benefit.

---

# 52. Database Statistics

SQL Server statistics are operational infrastructure.

The deployment and maintenance strategy must ensure that statistics remain sufficiently current.

Application code must not attempt to manage statistics directly.

Database maintenance policies belong to the operational environment.

---

# 53. Large Table Considerations

Before introducing high-volume operational tables such as `SyncOperations`, evaluate:

- retention;
- indexing;
- archival;
- partitioning;
- cleanup jobs;
- storage growth.

Partitioning is not part of the initial design unless scale requirements justify it.

---

# 54. Database Triggers

Triggers are discouraged.

They may only be introduced when:

1. the requirement cannot reasonably be implemented through constraints;
2. application/domain enforcement is insufficient;
3. the behavior is clearly infrastructure-oriented;
4. the operational consequences are documented.

Business workflows must not be implemented through triggers.

---

# 55. Stored Procedures

Stored procedures are not part of the default persistence architecture.

They may be introduced for:

- performance-critical bulk operations;
- specialized reporting;
- administrative operations;
- provider-specific operations that cannot reasonably be expressed through EF Core.

Their use requires architectural justification.

---

# 56. Views

Database views may be introduced for:

- reporting;
- read-only projections;
- complex operational queries.

Views must not redefine aggregate ownership or become the primary persistence model for domain entities.

---

# 57. Transaction Boundaries

Transaction boundaries must follow application use cases and aggregate consistency requirements.

The database transaction must not be used to artificially combine unrelated business workflows.

A typical transaction boundary is:

```text
Application Use Case
       |
       v
Aggregate mutation
       |
       v
Repository / DbContext
       |
       v
Database transaction
```

External system calls must generally not remain inside long-running database transactions.

---

# 58. External Integration and Transactions

Integration workflows must avoid distributed transaction assumptions.

The preferred model is:

```text
Local transaction
      |
      v
Persist integration state
      |
      v
Commit
      |
      v
External operation
      |
      v
Update synchronization state
```

Where reliable asynchronous processing is required, the architecture may introduce an outbox/inbox mechanism as a later persistence capability.

---

# 59. Migration and Integration Compatibility

Schema migrations affecting:

- external references;
- synchronization state;
- integration operations;

must be coordinated with the external integration contracts.

A database migration must not silently invalidate external identifiers or synchronization cursors.

---

# 60. Schema Versioning

The database schema version is represented by EF Core migration history.

If additional application-level schema metadata is required, it must be kept separate from domain tables.

The preferred mechanism is the framework migration history rather than introducing redundant custom version tracking.

---

# 61. Migration History Table

For SQL Server and EF Core, the framework-managed migrations history table is the default source of migration state.

If a custom name/schema is required, it must be explicitly configured.

The application must never modify migration history manually as a normal operational procedure.

---

# 62. Security of Migration Execution

Migration execution credentials must have only the privileges required for deployment.

Runtime application credentials should not automatically have unrestricted schema modification permissions in production.

The preferred separation is:

```text
Application identity
    -> runtime data access

Deployment identity
    -> schema modification
```

---

# 63. Environment Configuration

Connection strings and migration configuration must be externalized.

No environment-specific credentials may be embedded in:

- source code;
- migration files;
- seed data;
- EF Core configuration classes.

Secrets belong to the approved secret-management infrastructure.

---

# 64. Migration Observability

Migration execution must produce operationally useful information:

- migration identifier;
- start time;
- completion time;
- success/failure;
- database target;
- duration;
- error information.

Sensitive connection information must never appear in logs.

---

# 65. Operational Runbook Requirements

Before production deployment, the project must have documented procedures for:

- applying migrations;
- verifying migrations;
- handling migration failures;
- restoring backups;
- executing corrective migrations;
- identifying schema drift;
- validating application/database compatibility.

These procedures belong to the operational documentation set rather than the domain model.

---

# 66. Implementation Checklist

Before implementing the database layer:

- [ ] Configure all primary keys.
- [ ] Configure all foreign keys.
- [ ] Configure all delete behaviors explicitly.
- [ ] Configure all unique constraints.
- [ ] Configure all required check constraints.
- [ ] Configure all required indexes.
- [ ] Configure composite keys.
- [ ] Configure concurrency tokens.
- [ ] Configure UTC timestamps.
- [ ] Configure string lengths.
- [ ] Configure value conversions.
- [ ] Configure database schemas.
- [ ] Configure migration history.
- [ ] Define seed/reference data.
- [ ] Define migration naming conventions.
- [ ] Define migration review process.
- [ ] Define production deployment process.
- [ ] Define rollback/recovery strategy.
- [ ] Add database integration tests.
- [ ] Validate generated SQL.
- [ ] Validate representative query plans.

---

# 67. Architectural Decisions

The following decisions are established by this document.

| Decision | Status |
|---|---|
| EF Core migrations are authoritative | Accepted |
| SQL Server is the initial relational target | Accepted |
| GUID technical identifiers are the default | Accepted |
| Composite keys are limited to association entities | Accepted |
| Explicit FK delete behavior is mandatory | Accepted |
| Optimistic concurrency uses `rowversion` | Accepted |
| UTC timestamps are mandatory | Accepted |
| Fluent API is the primary persistence configuration mechanism | Accepted |
| Production auto-migration at application startup is discouraged | Accepted |
| Destructive migrations require explicit review | Accepted |
| Expand/contract is preferred for breaking schema changes | Accepted |
| Runtime credentials should not normally modify schema | Accepted |
| Database triggers are discouraged | Accepted |
| Business logic must remain outside the database | Accepted |

---

# 68. Open Decisions

The following decisions remain dependent on final domain/integration confirmation:

1. Global uniqueness of `Collections.Name`.
2. Exact external-reference uniqueness semantics.
3. Final delete behavior for `Collections -> CollectionItems`.
4. Final status enumeration persistence strategy.
5. Exact seed/reference-data catalog.
6. Need for custom migration history schema.
7. Whether synchronization history requires partitioning or archival.
8. Whether an outbox/inbox mechanism is required.
9. Final SQL Server deployment topology.
10. Production database maintenance policy.

These decisions must be resolved before the corresponding implementation areas are considered final.

---

# 69. Consistency Verification

The physical persistence model must remain consistent with the previous architectural artifacts.

Verification must establish:

```text
Domain invariant
      |
      v
Aggregate boundary
      |
      v
Persistence mapping
      |
      v
Table structure
      |
      v
Constraint/index
      |
      v
EF Core configuration
      |
      v
Migration
```

Any contradiction in this chain must be treated as an architectural defect rather than patched locally in the database implementation.

---

# 70. Definition of Done

The database persistence design is considered implementation-ready when:

- all domain-owned persistence entities are mapped;
- all relational relationships are explicit;
- all unconditional uniqueness rules are represented;
- all required indexes are justified;
- all concurrency requirements are mapped;
- all delete behaviors are explicit;
- migration conventions are established;
- migration testing is automated;
- production migration execution is controlled;
- recovery procedures exist;
- schema drift can be detected;
- EF Core generated SQL has been reviewed;
- no unresolved decision blocks the initial implementation.

---

# 71. Architectural Status

**Status:** Defined — implementation contract established.

This document completes the detailed definition of the **database constraint, indexing, concurrency, and migration strategy**.

The database layer can now be implemented without making fundamental architectural decisions during coding.

The remaining work is primarily implementation-level and validation-level.

---

# 72. Next Architectural Step

The next step should be to consolidate the complete persistence architecture into an **implementation readiness review**.

The review should verify the consistency of:

```text
39 — EF Core Architecture
40 — Domain/Persistence Mapping
41 — Database Schema and Table Catalog
42 — Constraints, Indexes and Migration Strategy
```

against:

```text
Domain Model
Application Use Cases
Infrastructure Boundaries
External Integrations
```

Once that review is complete, the project will have a stable persistence foundation suitable for moving from architectural definition toward concrete implementation planning.