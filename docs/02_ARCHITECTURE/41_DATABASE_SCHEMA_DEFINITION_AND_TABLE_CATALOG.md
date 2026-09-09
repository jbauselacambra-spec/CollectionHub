# 41 — Database Schema Definition and Table Catalog

## 1. Purpose

This document defines the **logical database schema** for CollectionHub and establishes the authoritative catalog of relational tables required to support the domain, application, persistence, and infrastructure architecture defined in the preceding documents.

The purpose of this document is to translate the persistence model into a concrete relational structure while preserving the domain boundaries and invariants already established.

This document does **not** define EF Core implementation details in depth. Those concerns belong to:

- `39_TECHNICAL_PERSISTENCE_DESIGN_AND_EF_CORE_ARCHITECTURE.md`
- `40_DATABASE_SCHEMA_AND_DOMAIN_PERSISTENCE_MAPPING.md`

This document focuses specifically on:

- database schemas;
- tables;
- primary keys;
- foreign keys;
- uniqueness;
- nullability;
- indexes;
- concurrency metadata;
- audit metadata;
- persistence ownership;
- table responsibilities;
- relational constraints.

---

# 2. Database Design Principles

The CollectionHub database follows these principles.

## 2.1 Relational integrity is mandatory

Relationships represented by the domain model must be represented explicitly through foreign keys whenever the relationship crosses a persistence boundary.

Application-level validation must not be the only mechanism protecting relational integrity.

---

## 2.2 Database structure follows domain ownership

Tables are organized according to the ownership of the corresponding aggregate or persistence concept.

The database must not become an anemic technical data store where unrelated domains freely share tables.

---

## 2.3 Aggregate boundaries remain visible

A foreign key does not imply that two entities belong to the same aggregate.

The distinction between:

- aggregate ownership;
- relational association;
- application-level reference;

must remain explicit.

---

## 2.4 Business identifiers and technical identifiers are different concepts

Where the domain requires a business identifier, it must not be confused with the technical primary key.

Technical identifiers provide stable persistence identity.

Business identifiers provide domain identity or externally meaningful references.

---

## 2.5 Soft deletion is not assumed globally

CollectionHub must not introduce soft deletion merely because it is convenient for persistence.

Soft deletion is only applicable to entities for which the domain explicitly requires historical retention or reversible removal.

---

## 2.6 Auditability is explicit

Where operational traceability is required, creation and modification metadata must be persisted explicitly.

Audit fields are infrastructure concerns and must not leak infrastructure implementation details into the domain model.

---

## 2.7 Concurrency must be protected

Mutable aggregate roots must support optimistic concurrency.

The database representation must therefore provide a concurrency mechanism that can be mapped by EF Core.

---

# 3. Database Schema Organization

The logical database is organized into the following schemas.

| Schema | Responsibility |
|---|---|
| `dbo` | Core application/domain persistence |
| `integration` | External integration state and synchronization metadata |
| `audit` | Optional operational/audit persistence |
| `system` | Technical infrastructure metadata |

The initial implementation should avoid unnecessary schema fragmentation.

The `dbo` schema is the primary application schema.

Additional schemas should only be created when the architectural boundary provides a concrete operational or security benefit.

---

# 4. Core Table Catalog

The following tables constitute the initial logical database catalog.

| Table | Aggregate / Concept | Responsibility |
|---|---|---|
| `dbo.Collections` | Collection | Collection aggregate root |
| `dbo.CollectionItems` | Collection | Items owned by a collection |
| `dbo.ItemTypes` | Item Type | Controlled classification of collected items |
| `dbo.Categories` | Category | Hierarchical/domain classification |
| `dbo.Tags` | Tag | Reusable classification metadata |
| `dbo.ItemTags` | Collection Item | Many-to-many relationship between items and tags |
| `dbo.ItemMetadata` | Collection Item | Optional extended metadata |
| `dbo.ExternalSources` | External Integration | Registered external systems |
| `dbo.ExternalReferences` | External Integration | Mapping between internal entities and external identifiers |
| `integration.SyncStates` | Synchronization | Synchronization state with external systems |
| `integration.SyncOperations` | Synchronization | Individual synchronization executions |
| `system.SchemaVersions` | Infrastructure | Database schema/version tracking |

The exact table set remains subject to confirmation against the final domain model.

Tables must not be created solely because they appear in this catalog if the corresponding domain concept is later removed.

---

# 5. `dbo.Collections`

## 5.1 Purpose

`Collections` stores the persistence representation of the **Collection aggregate root**.

A collection represents a logical grouping owned and managed by the application.

## 5.2 Columns

| Column | Type | Nullable | Key | Description |
|---|---|---:|---|---|
| `Id` | `uniqueidentifier` | No | PK | Technical aggregate identifier |
| `Name` | `nvarchar(200)` | No | | Collection name |
| `Description` | `nvarchar(2000)` | Yes | | Optional description |
| `CreatedAtUtc` | `datetime2` | No | | Creation timestamp |
| `UpdatedAtUtc` | `datetime2` | No | | Last modification timestamp |
| `Version` | `rowversion` | No | Concurrency | Optimistic concurrency token |

## 5.3 Constraints

- `Id` is the primary key.
- `Name` is mandatory.
- Collection names must comply with domain-level uniqueness rules if such a rule is confirmed.
- `Version` is used for optimistic concurrency.

## 5.4 Indexes

Recommended:

```text
PK_Collections
IX_Collections_Name
```

`IX_Collections_Name` should only become unique if collection-name uniqueness is a confirmed domain invariant.

---

# 6. `dbo.CollectionItems`

## 6.1 Purpose

`CollectionItems` stores items owned by a collection.

An item must not exist without its owning collection when the domain defines the collection as the aggregate owner.

## 6.2 Columns

| Column | Type | Nullable | Key | Description |
|---|---|---:|---|---|
| `Id` | `uniqueidentifier` | No | PK | Technical item identifier |
| `CollectionId` | `uniqueidentifier` | No | FK | Owning collection |
| `Name` | `nvarchar(300)` | No | | Item display name |
| `Description` | `nvarchar(4000)` | Yes | | Optional description |
| `ItemTypeId` | `uniqueidentifier` | Yes | FK | Item classification |
| `CategoryId` | `uniqueidentifier` | Yes | FK | Optional category |
| `CreatedAtUtc` | `datetime2` | No | | Creation timestamp |
| `UpdatedAtUtc` | `datetime2` | No | | Last modification timestamp |
| `Version` | `rowversion` | No | Concurrency | Optimistic concurrency token |

## 6.3 Foreign keys

```text
FK_CollectionItems_Collections
CollectionItems.CollectionId -> Collections.Id

FK_CollectionItems_ItemTypes
CollectionItems.ItemTypeId -> ItemTypes.Id

FK_CollectionItems_Categories
CollectionItems.CategoryId -> Categories.Id
```

## 6.4 Delete behavior

Deletion of a collection must respect aggregate semantics.

If collection deletion removes all owned items, the relationship may use cascading delete.

If historical preservation is required, explicit application-level deletion must be used instead.

The chosen behavior must be finalized during EF Core implementation.

## 6.5 Indexes

```text
PK_CollectionItems
IX_CollectionItems_CollectionId
IX_CollectionItems_ItemTypeId
IX_CollectionItems_CategoryId
IX_CollectionItems_CollectionId_Name
```

---

# 7. `dbo.ItemTypes`

## 7.1 Purpose

Stores controlled item classifications.

## 7.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `Code` | `nvarchar(100)` | No | Unique |
| `Name` | `nvarchar(200)` | No | |
| `Description` | `nvarchar(1000)` | Yes | |
| `IsActive` | `bit` | No | |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |

## 7.3 Constraints

```text
UQ_ItemTypes_Code
```

`Code` is the stable business identifier used by application and integration layers.

## 7.4 Indexes

```text
PK_ItemTypes
UQ_ItemTypes_Code
IX_ItemTypes_IsActive
```

---

# 8. `dbo.Categories`

## 8.1 Purpose

Stores hierarchical classification categories.

## 8.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `ParentCategoryId` | `uniqueidentifier` | Yes | FK |
| `Code` | `nvarchar(100)` | No | Unique |
| `Name` | `nvarchar(200)` | No | |
| `Description` | `nvarchar(1000)` | Yes | |
| `IsActive` | `bit` | No | |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |

## 8.3 Hierarchical relationship

```text
Categories.ParentCategoryId -> Categories.Id
```

The root category has `ParentCategoryId = NULL`.

## 8.4 Constraints

The database must prevent invalid direct self-reference:

```text
ParentCategoryId <> Id
```

Additional cycle detection remains an application/domain responsibility unless database-specific recursive constraints are introduced.

## 8.5 Indexes

```text
PK_Categories
UQ_Categories_Code
IX_Categories_ParentCategoryId
IX_Categories_IsActive
```

---

# 9. `dbo.Tags`

## 9.1 Purpose

Stores reusable tags applicable to collection items.

## 9.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `Name` | `nvarchar(100)` | No | Unique |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |

## 9.3 Constraints

```text
UQ_Tags_Name
```

Case sensitivity and normalization rules must be defined consistently with the application/database collation strategy.

---

# 10. `dbo.ItemTags`

## 10.1 Purpose

Represents the many-to-many association between collection items and tags.

## 10.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `ItemId` | `uniqueidentifier` | No | PK/FK |
| `TagId` | `uniqueidentifier` | No | PK/FK |
| `CreatedAtUtc` | `datetime2` | No | |

## 10.3 Primary key

Composite key:

```text
PK_ItemTags (ItemId, TagId)
```

## 10.4 Foreign keys

```text
FK_ItemTags_CollectionItems
ItemTags.ItemId -> CollectionItems.Id

FK_ItemTags_Tags
ItemTags.TagId -> Tags.Id
```

## 10.5 Indexes

```text
PK_ItemTags
IX_ItemTags_TagId
```

---

# 11. `dbo.ItemMetadata`

## 11.1 Purpose

Stores optional extended metadata associated with a collection item.

This table exists only where metadata cannot be represented safely as strongly typed domain properties.

The design must avoid turning `ItemMetadata` into an unrestricted replacement for a domain model.

## 11.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `ItemId` | `uniqueidentifier` | No | PK/FK |
| `MetadataJson` | `nvarchar(max)` | No | |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |
| `Version` | `rowversion` | No | Concurrency |

## 11.3 Relationship

```text
ItemMetadata.ItemId -> CollectionItems.Id
```

The one-to-one relationship uses `ItemId` as both primary and foreign key.

---

# 12. `dbo.ExternalSources`

## 12.1 Purpose

Represents external systems known by CollectionHub.

Examples may include:

- external collection systems;
- import sources;
- synchronization providers;
- third-party catalogues.

## 12.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `Code` | `nvarchar(100)` | No | Unique |
| `Name` | `nvarchar(200)` | No | |
| `IsActive` | `bit` | No | |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |

## 12.3 Constraints

```text
UQ_ExternalSources_Code
```

---

# 13. `dbo.ExternalReferences`

## 13.1 Purpose

Stores mappings between internal CollectionHub entities and identifiers owned by external systems.

External identifiers must not become primary identifiers for internal aggregates.

## 13.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `ExternalSourceId` | `uniqueidentifier` | No | FK |
| `EntityType` | `nvarchar(100)` | No | |
| `EntityId` | `uniqueidentifier` | No | |
| `ExternalId` | `nvarchar(300)` | No | |
| `CreatedAtUtc` | `datetime2` | No | |
| `UpdatedAtUtc` | `datetime2` | No | |

## 13.3 Uniqueness

The external identity should normally be unique within its source:

```text
UQ_ExternalReferences_Source_ExternalId
(
    ExternalSourceId,
    ExternalId
)
```

If different entity types can legitimately reuse an external identifier, the uniqueness constraint becomes:

```text
UQ_ExternalReferences_Source_Type_ExternalId
(
    ExternalSourceId,
    EntityType,
    ExternalId
)
```

This decision must be confirmed during integration implementation.

## 13.4 Indexes

```text
PK_ExternalReferences
IX_ExternalReferences_Entity
UQ_ExternalReferences_Source_ExternalId
```

---

# 14. `integration.SyncStates`

## 14.1 Purpose

Stores the durable synchronization state between CollectionHub and an external source.

## 14.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `ExternalSourceId` | `uniqueidentifier` | No | FK |
| `Scope` | `nvarchar(200)` | No | |
| `Status` | `nvarchar(50)` | No | |
| `LastSuccessfulSyncAtUtc` | `datetime2` | Yes | |
| `LastAttemptedSyncAtUtc` | `datetime2` | Yes | |
| `Cursor` | `nvarchar(1000)` | Yes | |
| `ErrorCode` | `nvarchar(100)` | Yes | |
| `ErrorMessage` | `nvarchar(2000)` | Yes | |
| `UpdatedAtUtc` | `datetime2` | No | |
| `Version` | `rowversion` | No | Concurrency |

## 14.3 Constraints

A source/scope combination must be unique:

```text
UQ_SyncStates_Source_Scope
(
    ExternalSourceId,
    Scope
)
```

---

# 15. `integration.SyncOperations`

## 15.1 Purpose

Records individual synchronization executions.

Unlike `SyncStates`, which represents current state, `SyncOperations` represents execution history.

## 15.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Id` | `uniqueidentifier` | No | PK |
| `ExternalSourceId` | `uniqueidentifier` | No | FK |
| `OperationType` | `nvarchar(50)` | No | |
| `Status` | `nvarchar(50)` | No | |
| `StartedAtUtc` | `datetime2` | No | |
| `CompletedAtUtc` | `datetime2` | Yes | |
| `ItemsProcessed` | `int` | No | |
| `ItemsSucceeded` | `int` | No | |
| `ItemsFailed` | `int` | No | |
| `ErrorCode` | `nvarchar(100)` | Yes | |
| `ErrorMessage` | `nvarchar(2000)` | Yes | |

## 15.3 Indexes

```text
PK_SyncOperations
IX_SyncOperations_ExternalSourceId
IX_SyncOperations_Status
IX_SyncOperations_StartedAtUtc
```

Operational history may grow significantly and therefore requires an explicit retention strategy.

---

# 16. `system.SchemaVersions`

## 16.1 Purpose

Tracks database schema evolution.

The preferred implementation mechanism remains the EF Core migration infrastructure.

This table is therefore a technical representation rather than a domain concept.

## 16.2 Columns

| Column | Type | Nullable | Key |
|---|---|---:|---|
| `Version` | `nvarchar(200)` | No | PK |
| `AppliedAtUtc` | `datetime2` | No | |
| `Checksum` | `nvarchar(128)` | Yes | |
| `Description` | `nvarchar(500)` | Yes | |

If EF Core migrations are used directly, this table may be replaced by the framework-managed migrations history table.

---

# 17. Primary Key Strategy

CollectionHub uses technical GUID-based identifiers for aggregate roots and persistence entities where appropriate.

The default pattern is:

```text
Id uniqueidentifier NOT NULL
```

Primary keys must:

- remain immutable;
- uniquely identify a persisted entity;
- never encode business semantics;
- not be exposed unnecessarily outside application boundaries.

Composite keys are reserved for true association tables such as `ItemTags`.

---

# 18. Foreign Key Strategy

Foreign keys must be defined for all relationships requiring database referential integrity.

Naming convention:

```text
FK_<DependentTable>_<PrincipalTable>
```

Example:

```text
FK_CollectionItems_Collections
```

Foreign keys must explicitly define deletion behavior.

Implicit provider defaults must not determine business behavior accidentally.

---

# 19. Nullability Rules

Nullability must reflect domain optionality.

The following principles apply:

- required domain properties → `NOT NULL`;
- optional domain properties → `NULL`;
- optional relationships → nullable foreign keys;
- technical timestamps → normally `NOT NULL`;
- concurrency tokens → `NOT NULL`;
- external errors → nullable;
- completion timestamps → nullable until operation completion.

Database nullability must not be used as a substitute for domain validation.

---

# 20. Indexing Strategy

Indexes exist to support:

1. primary-key lookups;
2. foreign-key traversal;
3. business-identifier lookup;
4. common application queries;
5. synchronization processing;
6. uniqueness constraints.

Indexes must not be created indiscriminately.

Every additional index introduces:

- storage cost;
- write amplification;
- maintenance overhead;
- migration complexity.

Query patterns defined during application implementation must validate the initial index catalog.

---

# 21. Unique Constraints

Business uniqueness must be enforced at the database level whenever the rule is unconditional.

Initial unique constraints include:

```text
UQ_ItemTypes_Code
UQ_Categories_Code
UQ_Tags_Name
UQ_ExternalSources_Code
UQ_SyncStates_Source_Scope
```

External-reference uniqueness requires confirmation of the final integration model.

---

# 22. Concurrency Strategy

Mutable aggregate roots use optimistic concurrency.

The preferred relational representation for SQL Server is:

```text
rowversion
```

The following tables therefore expose concurrency tokens where required:

- `Collections`
- `CollectionItems`
- `ItemMetadata`
- `integration.SyncStates`

EF Core must map these columns as concurrency tokens.

A concurrency conflict must be translated by the application layer into an appropriate application-level result rather than exposing provider-specific exceptions directly.

---

# 23. Audit Metadata

Persistent mutable entities should use:

```text
CreatedAtUtc
UpdatedAtUtc
```

where auditability is required.

Timestamps must always be stored in UTC.

The application is responsible for establishing the semantic meaning of timestamps.

The database must not contain local-time values.

---

# 24. Enumeration Persistence

Domain enumerations should normally be persisted using stable textual or numeric representations rather than provider-specific database enums.

Examples include:

```text
Status
OperationType
EntityType
```

The persisted representation must remain backward compatible when new values are introduced.

Where textual persistence is selected:

```text
nvarchar(...)
```

should be constrained to an appropriate maximum length.

---

# 25. JSON Persistence

JSON is permitted only for genuinely extensible metadata.

It must not be used to bypass relational modeling of:

- aggregate identity;
- relationships;
- business invariants;
- frequently queried properties;
- unique business attributes.

`ItemMetadata.MetadataJson` is therefore an extension mechanism, not the primary persistence model for `CollectionItem`.

---

# 26. Referential Integrity Matrix

| Relationship | Cardinality | FK | Delete strategy |
|---|---:|---|---|
| Collection → CollectionItems | 1:N | `CollectionId` | Aggregate-dependent |
| ItemType → CollectionItems | 1:N | `ItemTypeId` | Restrict |
| Category → CollectionItems | 1:N | `CategoryId` | Restrict |
| Category → Category | 1:N | `ParentCategoryId` | Restrict |
| CollectionItem ↔ Tag | N:M | `ItemTags` | Association deletion |
| CollectionItem → ItemMetadata | 1:0..1 | `ItemId` | Dependent deletion |
| ExternalSource → ExternalReferences | 1:N | `ExternalSourceId` | Restrict |
| ExternalSource → SyncStates | 1:N | `ExternalSourceId` | Restrict |
| ExternalSource → SyncOperations | 1:N | `ExternalSourceId` | Restrict |

`Restrict` is the preferred default for relationships where automatic cascading could violate domain or integration semantics.

---

# 27. Logical Dependency Graph

```text
Collections
    |
    +---- CollectionItems
              |
              +---- ItemTypes
              |
              +---- Categories
              |
              +---- ItemMetadata
              |
              +---- ItemTags ---- Tags
              |
              +---- ExternalReferences ---- ExternalSources
                                                 |
                                                 +---- SyncStates
                                                 |
                                                 +---- SyncOperations

Categories
    |
    +---- Categories
           (ParentCategoryId)
```

The graph represents persistence dependencies and does not imply that every relationship belongs to the same domain aggregate.

---

# 28. Table Ownership Matrix

| Table | Owner | Aggregate Boundary |
|---|---|---|
| `Collections` | Collection domain | Aggregate root |
| `CollectionItems` | Collection domain | Collection aggregate |
| `ItemTypes` | Classification domain | Independent reference concept |
| `Categories` | Classification domain | Independent hierarchical concept |
| `Tags` | Classification domain | Independent concept |
| `ItemTags` | Collection domain | Association |
| `ItemMetadata` | Collection domain | Item-dependent persistence |
| `ExternalSources` | Integration | Independent integration concept |
| `ExternalReferences` | Integration | External identity mapping |
| `integration.SyncStates` | Integration | Synchronization state |
| `integration.SyncOperations` | Integration | Operational history |
| `system.SchemaVersions` | Infrastructure | Technical metadata |

---

# 29. Database Naming Conventions

The following conventions are mandatory.

## Tables

PascalCase plural nouns:

```text
Collections
CollectionItems
ItemTypes
```

## Columns

PascalCase:

```text
CollectionId
CreatedAtUtc
UpdatedAtUtc
```

## Primary keys

```text
PK_<Table>
```

## Foreign keys

```text
FK_<DependentTable>_<PrincipalTable>
```

## Unique constraints

```text
UQ_<Table>_<Columns>
```

## Indexes

```text
IX_<Table>_<Columns>
```

## Concurrency columns

```text
Version
```

## Timestamps

```text
CreatedAtUtc
UpdatedAtUtc
```

---

# 30. Physical Database Considerations

The logical schema must remain provider-aware but not provider-dependent at the domain level.

The initial target database is expected to be relational and SQL Server-compatible.

Provider-specific details such as:

- clustered index strategy;
- fill factor;
- partitioning;
- temporal tables;
- compression;
- filtered indexes;
- computed columns;

must only be introduced when justified by measured operational requirements.

---

# 31. Migration Strategy

Database evolution must be performed through controlled migrations.

Every schema modification must be represented by a migration.

Migrations must:

1. preserve existing data;
2. maintain backward compatibility where required;
3. introduce constraints safely;
4. avoid destructive operations without explicit approval;
5. be reproducible;
6. be testable in CI/CD;
7. support rollback planning.

Production migrations must never rely on manually editing the database schema.

---

# 32. Seed Data Strategy

Reference data may be seeded for concepts such as:

- item types;
- categories;
- system-defined tags;
- integration providers.

Seed data must distinguish between:

### Immutable reference data

Stable values required by the application.

### User-managed data

Data that must never be overwritten by automatic migrations.

EF Core seeding must not accidentally replace user-maintained records.

---

# 33. Retention Strategy

Different persistence categories require different retention policies.

| Data | Retention |
|---|---|
| Collections | Domain-defined |
| CollectionItems | Domain-defined |
| Classification data | Long-lived |
| External references | Long-lived while integration exists |
| Sync states | Current state |
| Sync operations | Operational retention policy |
| Schema metadata | Permanent |

Operational synchronization history should be subject to a configurable retention policy to prevent unlimited growth.

---

# 34. Security Considerations

The database must not be treated as a security boundary by itself.

Application authorization remains responsible for deciding whether an operation is permitted.

Nevertheless:

- database credentials must use least privilege;
- integration credentials must not be stored in ordinary domain tables;
- secrets must be stored through the approved secret-management mechanism;
- production connection strings must not be committed to source control;
- sensitive metadata must not be persisted unnecessarily.

---

# 35. Performance Considerations

The initial schema is intentionally conservative.

Performance optimization must follow observed query behavior rather than speculative indexing.

Priority query paths are expected to include:

1. loading a collection;
2. retrieving items belonging to a collection;
3. filtering items by type;
4. filtering items by category;
5. retrieving tags;
6. resolving external identifiers;
7. retrieving synchronization state;
8. retrieving synchronization history.

These paths must be covered by integration tests and validated with representative datasets.

---

# 36. Schema Evolution Rules

Changes to the database schema must preserve the architectural boundaries.

The following changes require architectural review:

- introducing a new aggregate table;
- merging tables belonging to different aggregates;
- introducing cross-domain foreign keys;
- replacing relational properties with JSON;
- removing aggregate boundaries;
- introducing database triggers containing business rules;
- introducing database-specific logic that changes domain behavior.

Simple persistence optimizations do not necessarily require architectural review.

---

# 37. Explicitly Forbidden Persistence Patterns

The following patterns are prohibited unless explicitly justified by an architectural decision.

### 37.1 Generic Entity table

```text
Entities
    Id
    Type
    JsonData
```

This would hide the domain model and undermine relational integrity.

### 37.2 Generic attribute/value table as primary model

```text
Attributes
EntityId
Name
Value
```

This must not replace strongly typed domain properties.

### 37.3 Business logic in triggers

Domain invariants must remain implemented in the domain/application layers unless a database constraint is the appropriate enforcement mechanism.

### 37.4 Shared mutable tables between unrelated aggregates

Aggregate boundaries must remain explicit.

### 37.5 External IDs as internal primary keys

External systems must never dictate CollectionHub aggregate identity.

---

# 38. Verification Checklist

Before implementation, the following must be verified:

- [ ] Every persisted aggregate root has an explicit table.
- [ ] Every owned entity has a clear persistence owner.
- [ ] Every mandatory relationship has a foreign key.
- [ ] Every unconditional uniqueness rule has a unique constraint.
- [ ] Every mutable aggregate has a concurrency strategy.
- [ ] Every required timestamp is stored in UTC.
- [ ] Delete behavior has been explicitly defined.
- [ ] JSON is limited to genuinely extensible metadata.
- [ ] External identifiers are separated from internal identifiers.
- [ ] Synchronization state is separated from synchronization history.
- [ ] Indexes correspond to real query paths.
- [ ] Migration strategy is compatible with CI/CD.
- [ ] Seed data is separated from user-managed data.
- [ ] Retention policies exist for operational history.
- [ ] No infrastructure concern leaks into the domain model.
- [ ] No table has been introduced without a corresponding architectural responsibility.

---

# 39. Relationship With Previous Architecture Documents

This document operationalizes the persistence decisions defined in:

- `30_PERSISTENCE_ARCHITECTURE_AND_DATA_BOUNDARIES.md`
- `31_PERSISTENCE_COMPONENTS_AND_REPOSITORY_IMPLEMENTATIONS.md`
- `32_CONFIGURATION_AND_RUNTIME_INFRASTRUCTURE.md`
- `33_EXTERNAL_INTEGRATION_INFRASTRUCTURE.md`
- `34_INFRASTRUCTURE_ARCHITECTURE_CONSISTENCY_REVIEW.md`
- `36_TECHNOLOGY_STACK_AND_PLATFORM_SELECTION.md`
- `38_TECHNICAL_SOLUTION_STRUCTURE_AND_PROJECT_DEPENDENCIES.md`
- `39_TECHNICAL_PERSISTENCE_DESIGN_AND_EF_CORE_ARCHITECTURE.md`
- `40_DATABASE_SCHEMA_AND_DOMAIN_PERSISTENCE_MAPPING.md`

It therefore represents the **logical database contract** that the EF Core implementation must realize.

---

# 40. Architectural Status

**Status:** Defined — implementation pending.

The database schema is sufficiently defined to proceed to the next architectural step.

However, the schema remains subject to reconciliation against:

1. the final domain model;
2. final aggregate boundaries;
3. confirmed application use cases;
4. external integration requirements;
5. EF Core mapping decisions;
6. performance validation.

No production database implementation should begin until those dependencies have been reconciled.

---

# 41. Next Step

The next logical step is to define the **database constraints, indexes, keys, concurrency mappings, and migration strategy at implementation level**, establishing the bridge between this logical table catalog and the concrete EF Core model.

That work should consolidate:

- relational constraints;
- index definitions;
- delete behaviors;
- concurrency configuration;
- value conversions;
- database-generated values;
- migration conventions;
- seed-data strategy;
- provider-specific persistence decisions.

The resulting document should become the direct implementation contract for the database layer.