# Technical Persistence Design and EF Core Architecture

## 1. Purpose

This document defines the concrete persistence architecture for CollectionHub using PostgreSQL and Entity Framework Core, as established in:

`37_TECHNOLOGY_STACK_AND_PLATFORM_SELECTION.md`

and structurally integrated through:

`38_TECHNICAL_SOLUTION_STRUCTURE_AND_PROJECT_DEPENDENCIES.md`

The purpose is to define how domain aggregates and application persistence contracts will be technically persisted without allowing the persistence technology to redefine the domain model.

This document establishes:

- EF Core ownership,
- `DbContext` boundaries,
- persistence model strategy,
- entity mappings,
- repository implementation structure,
- transaction management,
- concurrency handling,
- database constraints,
- migrations,
- query strategy,
- persistence error handling,
- testing strategy,
- performance principles,
- persistence-specific architectural rules.

No concrete domain entity implementation is introduced here.

---

# 2. Persistence Architectural Principle

The persistence architecture follows:

```text
Domain Aggregate
       │
       ▼
Repository Contract
       │
       ▼
Repository Implementation
       │
       ▼
Persistence Model
       │
       ▼
EF Core
       │
       ▼
PostgreSQL
```

The fundamental rule is:

> Entity Framework Core is an infrastructure implementation detail and must not become part of the domain model.

---

# 3. Persistence Ownership

Persistence belongs exclusively to:

```text
CollectionHub.Infrastructure
```

The expected structure is:

```text id="6r4kpl"
CollectionHub.Infrastructure/
└── Persistence/
    ├── Context/
    ├── Configurations/
    ├── Models/
    ├── Mappings/
    ├── Repositories/
    ├── Transactions/
    ├── Migrations/
    └── Errors/
```

The exact subdirectory structure may evolve, but all persistence-specific implementation must remain within Infrastructure.

---

# 4. EF Core Dependency Boundary

Only `CollectionHub.Infrastructure` may directly reference:

- Entity Framework Core,
- EF Core relational packages,
- PostgreSQL EF Core provider,
- migration tooling.

The following are prohibited:

```text id="8pv2mx"
CollectionHub.Domain
    → Microsoft.EntityFrameworkCore

CollectionHub.Application
    → Microsoft.EntityFrameworkCore
```

This ensures that persistence technology cannot leak inward.

---

# 5. DbContext Responsibility

The primary EF Core `DbContext` represents the persistence boundary of the modular monolith.

Conceptually:

```text id="2j6u8a"
CollectionHubDbContext
        │
        ├── DbSet / mappings
        ├── Model configuration
        ├── Transaction participation
        └── Persistence lifecycle
```

The `DbContext` is responsible for:

- database session management,
- entity tracking where used,
- persistence mapping,
- transaction participation,
- query execution,
- persistence state management.

It is not responsible for:

- business rules,
- aggregate decisions,
- use-case orchestration,
- external API calls.

---

# 6. DbContext Scope

The initial architecture should use one logical application persistence context:

```text
CollectionHubDbContext
```

This context represents the transactional persistence boundary of the modular monolith.

Multiple contexts should not be introduced merely to mirror every domain concept.

A separate context should only be introduced if a concrete architectural requirement justifies it.

---

# 7. DbContext Lifetime

The default lifetime should align with the application request/use-case execution boundary.

For HTTP requests, the expected lifecycle is approximately:

```text id="n0e1go"
HTTP Request
     ↓
Application Use Case
     ↓
DbContext
     ↓
Commit / Rollback
     ↓
HTTP Response
```

The context must not become a singleton.

A singleton `DbContext` would introduce:

- shared state,
- concurrency problems,
- stale tracking,
- memory growth,
- transaction ambiguity.

---

# 8. Domain Model vs Persistence Model

The architecture distinguishes between:

```text id="2q7d4n"
Domain Model
     ↕
Persistence Mapping
     ↕
Persistence Model
```

The persistence model exists to represent database requirements.

It may contain:

- relational foreign keys,
- technical identifiers,
- database-specific columns,
- concurrency metadata,
- persistence-only navigation properties.

These must not automatically become domain concepts.

---

# 9. Persistence Model Strategy

Persistence models should be explicit classes owned by Infrastructure.

Conceptually:

```text id="h7tqj0"
CollectionPersistenceModel
    ├── Id
    ├── Database fields
    ├── Foreign keys
    ├── Persistence metadata
    └── Relationships
```

The domain aggregate remains separate.

This prevents:

- EF Core annotations in domain classes,
- lazy-loading proxies,
- persistence lifecycle semantics,
- database-specific concerns.

---

# 10. Mapping Strategy

Mappings should use EF Core's Fluent API.

Preferred:

```text id="m5uyq2"
Persistence Model
       │
       ▼
IEntityTypeConfiguration<T>
       │
       ▼
EF Core Model
```

This keeps database configuration outside the persistence model where practical.

Mapping classes should reside under:

```text
Persistence/Configurations/
```

---

# 11. Fluent Mapping Principles

Mappings should explicitly define:

- table names,
- column names where useful,
- primary keys,
- foreign keys,
- indexes,
- required properties,
- maximum lengths,
- relationships,
- delete behavior,
- concurrency configuration,
- value conversions where necessary.

Implicit database conventions should only be relied upon when they are clear and stable.

---

# 12. Aggregate Persistence

Persistence must respect aggregate boundaries established by the domain architecture.

An aggregate should be loaded and persisted as a consistency boundary.

The repository should not expose arbitrary internal persistence records as application-level objects.

Conceptually:

```text id="i4ksl0"
Aggregate Root
     │
     ├── Entity
     ├── Entity
     └── Value Object
```

should be reconstructed as one coherent domain model.

---

# 13. Aggregate Root Ownership

Repositories should normally correspond to aggregate roots rather than individual entities.

Preferred:

```text id="1k3r1f"
CollectionRepository
```

rather than:

```text id="m1ojj6"
CollectionItemRepository
CollectionMetadataRepository
CollectionTagRepository
```

when those objects belong to the same aggregate.

The repository boundary must follow the domain consistency boundary.

---

# 14. Repository Implementation

Repository implementations belong under:

```text
Persistence/Repositories/
```

They implement application/domain-facing contracts.

Conceptually:

```text id="g1gr7v"
ICollectionRepository
        ▲
        │ implements
        │
EfCollectionRepository
        │
        ▼
CollectionHubDbContext
```

The repository implementation may use EF Core directly.

The application layer must not.

---

# 15. Repository Responsibilities

Repositories are responsible for:

- loading aggregates,
- persisting aggregate changes,
- translating persistence models,
- executing aggregate-specific queries,
- participating in transactions,
- translating persistence failures.

Repositories are not responsible for:

- enforcing domain invariants,
- deciding business behavior,
- coordinating unrelated use cases,
- calling external APIs,
- managing HTTP requests.

---

# 16. Query Strategy

Queries should be designed around application use cases.

The architecture should avoid generic repositories such as:

```text id="22x7iv"
GetAll()
Find()
SaveAnything()
DeleteAnything()
```

unless a concrete domain requirement justifies them.

Repositories should expose meaningful persistence capabilities.

For example:

```text id="4gnd7g"
FindById(...)
FindByExternalReference(...)
Save(...)
```

The exact methods will be derived from application use cases.

---

# 17. Read and Write Separation

The initial architecture does not require a full CQRS infrastructure.

However, read operations and aggregate modification operations may use different persistence strategies when justified.

For example:

```text id="gh1kpa"
Write:
Application
   ↓
Aggregate Repository
   ↓
EF Core
```

while:

```text id="i9x8e5"
Read:
Application Query
   ↓
Read-oriented persistence query
   ↓
Projection / DTO
```

The distinction should only be introduced where it improves a concrete use case.

---

# 18. Tracking Strategy

EF Core tracking should be used intentionally.

For aggregate modification:

```text
Load
  ↓
Modify Domain Aggregate
  ↓
Persist
```

Tracking may be useful where it simplifies persistence.

For read-only queries, `AsNoTracking()` should generally be preferred when entity tracking is unnecessary.

The choice must be based on operation semantics rather than habit.

---

# 19. Lazy Loading

Lazy loading should **not be enabled by default**.

Reasons include:

- hidden database queries,
- unpredictable performance,
- accidental N+1 queries,
- persistence behavior leaking into domain behavior.

Required relationships should be loaded explicitly.

---

# 20. Eager Loading

Eager loading should only load data required by the use case.

The repository should avoid blindly loading entire aggregate graphs when unnecessary.

The aggregate consistency boundary determines what must be loaded.

---

# 21. N+1 Query Prevention

Persistence implementations must explicitly guard against N+1 query patterns.

Potential mitigations include:

- appropriate eager loading,
- projection queries,
- explicit joins,
- batch operations,
- query-specific repositories.

Performance-sensitive queries should be verified through integration tests or profiling rather than assumptions.

---

# 22. Value Objects

Domain value objects should not be forced into persistence-specific structures.

Where a value object maps naturally to one database column, EF Core value conversion or owned/value-object mapping may be used.

The mapping remains infrastructure-owned.

Conceptually:

```text id="zjxy0o"
Domain Value Object
       ↓
EF Core Conversion
       ↓
Database Column
```

---

# 23. Strongly Typed Identifiers

If CollectionHub uses strongly typed domain identifiers, EF Core mappings should explicitly define how they are persisted.

For example:

```text id="w1y2b8"
Domain Identifier
       ↓
Value Conversion
       ↓
Database Primitive
```

The database should use a representation appropriate to PostgreSQL and the domain requirements.

The domain should not be forced to use raw database primitives merely for persistence convenience.

---

# 24. Database Primary Keys

Primary key strategy must be consistent across aggregates.

The exact identifier type must follow the domain model.

The persistence layer should not introduce a separate technical identifier unless there is a clear reason.

Potential representations include:

- UUID,
- integer,
- strongly typed UUID wrapper.

The final selection must remain aligned with the domain identifier strategy.

---

# 25. Foreign Keys

Foreign keys should be used to enforce relational integrity where relationships are persistence-relevant.

However, foreign-key relationships must not be interpreted automatically as domain aggregate relationships.

Database relationships and domain boundaries are related but not identical concepts.

---

# 26. Delete Behavior

Delete behavior must be explicitly defined for relationships where accidental cascading could violate domain expectations.

Possible strategies include:

- cascade,
- restrict,
- no action,
- explicit application-controlled deletion.

Cascade deletes should not be enabled blindly.

If deletion has domain meaning, the application/domain model must determine whether deletion is permitted.

---

# 27. Database Constraints

The database should enforce technical integrity constraints where appropriate.

Examples include:

- primary keys,
- foreign keys,
- unique indexes,
- not-null constraints,
- length constraints,
- check constraints where technically appropriate.

Database constraints complement domain invariants.

They do not replace them.

---

# 28. Domain Invariants vs Database Constraints

The distinction is:

```text id="b4y5kq"
Domain:
"What is valid according to business rules?"

Database:
"What must remain technically consistent in storage?"
```

A database uniqueness constraint may protect a domain invariant, but the domain must still express the business meaning when the rule is relevant to behavior.

---

# 29. Unique Constraints

Unique database constraints should be used when uniqueness is a real persistence invariant.

Examples may include:

- external identifiers,
- provider references,
- normalized technical keys.

The application must still translate unique constraint violations into meaningful application behavior where required.

---

# 30. PostgreSQL-Specific Features

PostgreSQL-specific features may be used inside Infrastructure when they provide meaningful value.

Examples may include:

- UUID support,
- JSONB,
- partial indexes,
- specialized indexes,
- database functions,
- PostgreSQL-specific constraints.

Such features must remain isolated inside Infrastructure.

Their use must not force PostgreSQL-specific concepts into the domain model.

---

# 31. JSON / JSONB Usage

JSONB should not become the default persistence mechanism for domain objects.

Structured relational columns should be preferred when data participates in:

- relationships,
- querying,
- constraints,
- indexing,
- domain consistency.

JSONB may be appropriate for genuinely flexible or external metadata.

Its use must be justified per data structure.

---

# 32. Database Schema Ownership

The CollectionHub application owns its persistence schema.

External systems must not directly modify the CollectionHub database.

Schema evolution must occur through application-controlled migrations.

---

# 33. Migration Structure

Migrations should reside inside:

```text
CollectionHub.Infrastructure/
└── Persistence/
    └── Migrations/
```

Migration files are infrastructure artifacts.

They must be reviewed as part of source control.

---

# 34. Migration Rules

Migration rules:

1. Every schema change must be represented by a migration.
2. Migrations must be deterministic.
3. Migrations must be tested.
4. Production schema changes must not rely on manual SQL execution.
5. Destructive migrations require explicit review.
6. Data migrations must be treated separately from simple schema changes where necessary.

---

# 35. Destructive Migrations

Destructive operations such as:

- dropping columns,
- dropping tables,
- removing indexes required by existing queries,
- changing incompatible types,

must be treated as high-risk changes.

Where necessary, schema changes should use staged migrations:

```text id="j3q4po"
Expand
  ↓
Migrate
  ↓
Deploy
  ↓
Validate
  ↓
Contract
```

This reduces deployment risk.

---

# 36. Transaction Management

The application use case determines the transactional intent.

Infrastructure provides the mechanism.

Preferred conceptual flow:

```text id="p0j8q1"
Use Case
   ↓
Transaction Begin
   ↓
Repository Operations
   ↓
Domain Changes
   ↓
Repository Persistence
   ↓
Commit
```

On failure:

```text id="n9y7q3"
Failure
   ↓
Rollback
   ↓
Infrastructure Error Translation
   ↓
Application Error
```

---

# 37. Unit of Work

The `DbContext` naturally provides unit-of-work semantics for EF Core.

A separate generic `IUnitOfWork` abstraction should not be introduced automatically.

It should only be added if application-level transaction requirements justify an explicit abstraction.

Avoid creating abstractions that merely duplicate EF Core concepts.

---

# 38. Transaction Abstraction

If application use cases require transaction control independently of EF Core, the application layer may define an abstraction such as:

```text id="s6i9me"
ITransactionBoundary
```

Infrastructure would implement it.

However, this should only be introduced when actual use cases demonstrate the need.

The initial default is to keep transaction mechanics inside infrastructure while exposing only the minimum required application contract.

---

# 39. Concurrency

The persistence layer should support optimistic concurrency where required.

Potential implementation:

```text id="h4g3b2"
Aggregate
    ↓
Version
    ↓
Database concurrency column
```

A stale update should result in a technical concurrency failure.

Infrastructure translates that failure into an application-level concurrency outcome.

---

# 40. Concurrency Error Handling

A concurrency conflict should not be returned as a raw EF Core exception.

Conceptually:

```text id="l5o7w2"
DbUpdateConcurrencyException
          ↓
Infrastructure Translation
          ↓
Application Concurrency Conflict
```

The application can then determine the appropriate user-facing behavior.

---

# 41. Persistence Error Translation

Infrastructure must translate provider-specific exceptions.

Examples:

```text id="o8u1i9"
PostgresException
DbUpdateException
DbUpdateConcurrencyException
TimeoutException
```

must not leak directly into domain logic.

The exact application error taxonomy will be aligned with the application error model.

---

# 42. Connection Management

Database connections should be managed by EF Core and the underlying PostgreSQL provider.

Application code must not manually manage raw database connections unless a specific infrastructure operation requires it.

Connection configuration belongs to Infrastructure/Runtime configuration.

---

# 43. Connection Resilience

Transient database failures may justify retry strategies.

Retries must be carefully evaluated because database operations may interact with transactions and side effects.

Automatic retries should only be enabled where:

- operations are safe to retry,
- provider behavior is understood,
- transaction semantics remain correct.

---

# 44. Database Initialization

The application must distinguish between:

```text id="lrxm7k"
Schema migration
```

and:

```text id="x5a7m2"
Test/development database initialization
```

Production environments should not rely on uncontrolled automatic database creation.

Migrations should be executed through an explicit deployment process.

---

# 45. Development Database

Local development should use PostgreSQL through Docker.

Conceptually:

```text id="31h0l2"
Docker Compose
      ↓
PostgreSQL
      ↓
CollectionHub Development
```

The local database must use the same database technology as production.

---

# 46. Integration Test Database

Integration tests should use an isolated PostgreSQL database.

Preferred approach:

```text id="f4r9z1"
Integration Test
      ↓
Ephemeral PostgreSQL
      ↓
Migrations
      ↓
Test
      ↓
Dispose
```

This ensures tests validate actual PostgreSQL behavior.

---

# 47. Test Data Strategy

Test data should be explicit and deterministic.

Avoid:

- shared mutable test databases,
- order-dependent tests,
- random data without controlled seeds,
- production database snapshots containing sensitive information.

Test fixtures should represent domain scenarios rather than raw database records where possible.

---

# 48. Persistence Testing Levels

Persistence testing should include:

### Mapping tests

Verify:

- keys,
- constraints,
- relationships,
- conversions.

### Repository integration tests

Verify:

- save,
- load,
- update,
- query,
- delete where applicable.

### Transaction tests

Verify:

- commit,
- rollback,
- atomicity.

### Concurrency tests

Verify:

- stale update detection,
- conflict translation.

### Migration tests

Verify:

- schema creation,
- upgrade paths,
- compatibility.

---

# 49. Repository Test Principle

Repository tests must validate behavior rather than implementation.

For example:

```text id="91r4ne"
Given an existing aggregate
When the repository loads it
Then the domain aggregate is reconstructed correctly
```

rather than:

```text id="z4q2x6"
Then EF Core called Include(...)
```

The latter couples tests to implementation details.

---

# 50. Performance Strategy

Performance optimization must be evidence-driven.

Initial priorities:

1. Correct aggregate loading.
2. Appropriate indexes.
3. Avoiding N+1 queries.
4. Appropriate tracking strategy.
5. Efficient pagination.
6. Query projection where useful.

Premature caching should be avoided.

---

# 51. Pagination

Large result sets must not be loaded indiscriminately.

Application queries returning collections should define explicit pagination requirements.

Persistence implementations should support efficient database-side pagination.

Offset-based pagination may be sufficient initially.

Keyset pagination can be introduced later where workload characteristics justify it.

---

# 52. Search Strategy

Search requirements must be evaluated independently from persistence storage.

If basic relational querying satisfies initial requirements, PostgreSQL querying should remain the default.

A dedicated search engine should not be introduced until:

- search complexity,
- ranking requirements,
- scale,
- indexing needs,

justify it.

---

# 53. Caching

No mandatory persistence cache is introduced initially.

Caching may be introduced later for:

- expensive reads,
- external provider results,
- reference data,
- high-frequency queries.

Any cache must preserve application consistency requirements.

---

# 54. Backup and Recovery

Production PostgreSQL must eventually provide:

- automated backups,
- point-in-time recovery where required,
- restoration testing,
- retention policies,
- recovery objectives.

These concerns belong to the deployment/operations architecture and are not implemented by repository code.

---

# 55. Database Security

Database access must use:

- credentials provided through secure configuration,
- encrypted transport where required,
- least-privilege database users,
- separate credentials per environment.

The application must not run with unnecessary administrative database privileges.

---

# 56. Database Naming Strategy

Database naming should be consistent and explicit.

Names should follow a predictable convention for:

- tables,
- columns,
- indexes,
- foreign keys,
- constraints.

The naming convention should be established before the first migration is committed.

---

# 57. Schema Naming

The default approach is to use a dedicated application schema where beneficial.

If PostgreSQL's default `public` schema is used initially, the decision must still be explicit.

The choice should consider:

- operational simplicity,
- future modularization,
- migration management,
- database administration.

---

# 58. Temporal Data

Date/time persistence must use an explicit strategy.

The application should standardize on UTC for persisted instants.

Conceptually:

```text id="5f9v8q"
Application Time
      ↓
UTC
      ↓
PostgreSQL
```

Local timezone interpretation belongs at application/API presentation boundaries.

---

# 59. External Identifiers

External provider identifiers should be stored separately from internal domain identifiers.

Conceptually:

```text id="k8u2q0"
Internal CollectionHub ID
        ≠
External Provider ID
```

External identifiers should be modeled as integration/persistence concerns unless domain semantics explicitly require otherwise.

Unique constraints may be applied where provider uniqueness is guaranteed.

---

# 60. Persistence and Domain Events

Persistence of domain events must not automatically imply an event broker.

If domain events require persistence for reliability, an outbox pattern may be introduced later.

Initial persistence should not introduce an outbox table unless a concrete integration requirement requires durable event publication.

---

# 61. Outbox Strategy

If asynchronous external side effects become necessary, the preferred future approach is:

```text id="x0q7n2"
Application Transaction
       │
       ├── Aggregate State
       │
       └── Outbox Message
               │
               ▼
          Commit
               │
               ▼
       Background Publisher
               │
               ▼
       External System
```

This ensures transactional consistency between domain state and external publication.

The outbox is therefore a future architectural capability, not an initial mandatory component.

---

# 62. Persistence Observability

Persistence instrumentation should capture:

- operation duration,
- database dependency,
- query category,
- failures,
- transaction failures,
- connection failures.

Actual SQL statement logging should be controlled carefully because it can:

- expose sensitive data,
- create excessive log volume,
- degrade performance.

Detailed SQL logging should normally be development-only.

---

# 63. EF Core Design-Time Configuration

EF Core design-time tooling may require creation of the `DbContext` outside normal runtime execution.

The design-time mechanism must remain inside Infrastructure and must not require domain/application changes merely to satisfy migration tooling.

The exact implementation will be established during project setup.

---

# 64. Migration Assembly

Migrations should belong to the Infrastructure assembly.

The runtime must explicitly identify the migration assembly where necessary.

This prevents migration artifacts from entering the application or domain projects.

---

# 65. Multiple Database Providers

The initial architecture targets PostgreSQL only.

Supporting multiple database providers should not be treated as a requirement.

Provider abstraction should only be introduced if a real business or operational requirement exists.

Avoid designing the persistence layer around hypothetical database replacement.

---

# 66. Persistence Architecture Diagram

The final persistence architecture is:

```text id="l0g4k8"
                 APPLICATION
                     │
                     │ Repository Contracts
                     ▼
             ┌─────────────────┐
             │  INFRASTRUCTURE │
             │                 │
             │  Repositories   │
             │       │         │
             │       ▼         │
             │  Persistence    │
             │  Models         │
             │       │         │
             │       ▼         │
             │  EF Core        │
             │       │         │
             │       ▼         │
             │  DbContext      │
             └───────┬─────────┘
                     │
                     ▼
                PostgreSQL
```

The domain remains outside the persistence implementation.

---

# 67. Persistence Dependency Rules

The following rules are mandatory.

### PDR-01

EF Core exists only in Infrastructure.

### PDR-02

PostgreSQL provider exists only in Infrastructure.

### PDR-03

Persistence models exist only in Infrastructure.

### PDR-04

Repository implementations exist only in Infrastructure.

### PDR-05

Repository contracts remain technology-independent.

### PDR-06

Domain entities do not require EF Core.

### PDR-07

API DTOs do not depend on persistence models.

### PDR-08

External integration models do not share persistence models unless explicitly justified.

### PDR-09

Transactions align with application consistency boundaries.

### PDR-10

Database migrations belong to Infrastructure.

---

# 68. Persistence Risk Register

| Risk | Impact | Mitigation |
|---|---:|---|
| EF Core leaks into domain | High | Project/package restrictions |
| Persistence model becomes domain model | High | Explicit mappings |
| Generic repositories hide domain behavior | Medium | Aggregate-oriented repositories |
| N+1 queries | Medium | Query design and integration tests |
| Hidden transactions | High | Explicit transaction policy |
| Database constraints replace domain rules | High | Maintain domain invariants |
| PostgreSQL-specific features leak inward | Medium | Infrastructure isolation |
| Migration drift | High | Version-controlled migrations |
| Production schema changes manually | High | Migration-based deployment |
| Database becomes integration contract | High | Database ownership boundary |

---

# 69. Acceptance Criteria

The persistence architecture is considered complete when:

- [x] EF Core ownership is defined.
- [x] PostgreSQL is defined as the persistence technology.
- [x] `DbContext` responsibility is defined.
- [x] Persistence model strategy is defined.
- [x] Mapping strategy is defined.
- [x] Repository implementation boundary is defined.
- [x] Aggregate persistence strategy is defined.
- [x] Transaction strategy is defined conceptually.
- [x] Concurrency strategy is defined conceptually.
- [x] Migration strategy is defined.
- [x] Persistence testing strategy is defined.
- [x] Error translation strategy is defined.
- [x] Performance principles are defined.
- [x] Persistence dependency rules are defined.
- [ ] Concrete domain-to-table mappings are defined.

The final unchecked item intentionally depends on the domain model and will be addressed when the concrete persistence model is designed.

---

# 70. Final Decision

CollectionHub will use:

```text id="qk0w4j"
Persistence:
    PostgreSQL

Data Access:
    Entity Framework Core

DbContext:
    CollectionHubDbContext

Repository:
    Aggregate-oriented implementations

Mapping:
    Explicit Fluent API configuration

Migrations:
    EF Core Migrations

Transactions:
    Application use-case boundary

Concurrency:
    Optimistic where required

Local Database:
    PostgreSQL via Docker

Integration Tests:
    Real PostgreSQL where practical
```

The persistence architecture deliberately avoids:

- generic CRUD repositories,
- ORM-driven domain models,
- lazy loading by default,
- multiple database providers,
- premature caching,
- premature CQRS,
- mandatory outbox infrastructure,
- database-specific concepts leaking into the domain.

The resulting technical boundary is:

```text id="1q2q2k"
Domain
   │
   ▼
Application Repository Contract
   │
   ▼
Infrastructure Repository
   │
   ▼
Persistence Mapping
   │
   ▼
EF Core
   │
   ▼
PostgreSQL
```

This provides the concrete persistence foundation required for the next stage of technical design.

**Persistence architecture: ESTABLISHED.**

**EF Core boundary: ESTABLISHED.**

**PostgreSQL boundary: ESTABLISHED.**

**Next step:** define the concrete database schema strategy, aggregate-to-table mappings, identifiers, relationships, indexes, constraints, and migration model in `40_DATABASE_SCHEMA_AND_DOMAIN_PERSISTENCE_MAPPING.md`.