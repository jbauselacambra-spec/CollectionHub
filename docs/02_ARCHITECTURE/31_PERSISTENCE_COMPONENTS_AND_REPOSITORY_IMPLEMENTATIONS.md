# 31. Persistence Components and Repository Implementations

## 1. Purpose

This document defines the persistence components and repository implementations required by CollectionHub.

Its purpose is to translate the persistence architecture defined previously into a concrete component-level model while preserving the architectural boundaries established between:

- Domain
- Application
- Infrastructure
- Persistence
- External systems

This document does **not** define source code or framework-specific implementations.

It establishes the responsibilities, dependencies, contracts, mapping rules, transactional behavior and consistency rules that future implementations must respect.

---

## 2. Architectural Context

CollectionHub follows a layered architecture in which the Domain Model remains independent from persistence technology.

The fundamental dependency direction is:

```text
Presentation
    |
Application
    |
Domain
    ^
    |
Infrastructure / Persistence
```

The domain defines persistence-independent repository contracts where repository abstractions are required by domain/application use cases.

Infrastructure provides the concrete implementations of those contracts.

Therefore:

```text
Domain/Application
        |
        | repository abstraction
        v
Infrastructure
        |
        | persistence implementation
        v
Database / Persistence Provider
```

The database must never become a source of domain behavior.

---

## 3. Persistence Responsibilities

The persistence subsystem is responsible for:

1. Storing aggregate state.
2. Loading aggregates.
3. Reconstructing valid domain objects.
4. Persisting aggregate changes.
5. Managing transactional boundaries.
6. Maintaining persistence identity.
7. Mapping persistence representations to domain representations.
8. Executing application-specific read queries where appropriate.
9. Managing optimistic concurrency.
10. Translating persistence failures into application/infrastructure errors.
11. Maintaining database-specific concerns outside the domain model.

The persistence subsystem is **not** responsible for:

- Business invariants.
- Domain decisions.
- Aggregate behavior.
- Domain validation.
- Domain event semantics.
- Application workflow orchestration.
- Authorization decisions.
- User interface concerns.

---

# 4. Repository Model

## 4.1 Repository Definition

A repository represents the persistence boundary for an aggregate.

Its conceptual responsibility is:

```text
Repository
    = Aggregate persistence boundary
```

A repository must therefore be defined around aggregate roots rather than individual tables.

For example:

```text
CollectionRepository
    -> Collection aggregate

ItemRepository
    -> Item aggregate, if Item is an independent aggregate

UserRepository
    -> User aggregate
```

The exact repository set must follow the aggregate boundaries established in the domain model.

---

## 4.2 Repository Contract

Repository contracts belong to the architectural layer that requires them.

The contract should express domain/application concepts rather than persistence technology.

Conceptually:

```text
findById(id)
save(aggregate)
remove(aggregate)
exists(...)
```

Additional query methods may be introduced only when they represent meaningful application/domain requirements.

Avoid contracts such as:

```text
executeSql(...)
createQuery(...)
getEntityManager(...)
loadFromTable(...)
```

because these expose persistence implementation details.

---

# 5. Repository Implementation Structure

Concrete repository implementations belong to Infrastructure.

A conceptual structure is:

```text
Infrastructure
└── Persistence
    ├── Repositories
    │   ├── CollectionRepositoryImpl
    │   ├── ItemRepositoryImpl
    │   └── ...
    │
    ├── Mappers
    │   ├── CollectionMapper
    │   ├── ItemMapper
    │   └── ...
    │
    ├── Records / Entities
    │   ├── CollectionRecord
    │   ├── ItemRecord
    │   └── ...
    │
    ├── Queries
    │   ├── CollectionQuery
    │   ├── ItemQuery
    │   └── ...
    │
    └── Transactions
        └── UnitOfWork
```

The exact physical package/module structure may evolve with the selected technology.

The architectural responsibilities must remain stable.

---

# 6. Aggregate Persistence

Repositories must persist complete aggregate state according to the aggregate's consistency boundary.

A repository must not arbitrarily persist only individual pieces of an aggregate if doing so can violate aggregate invariants.

Conceptually:

```text
Aggregate Root
      |
      +-- Entity
      |
      +-- Value Object
      |
      +-- Entity
```

The repository is responsible for translating this structure into the persistence model.

The database representation may be structurally different from the domain representation.

That difference is acceptable and expected.

---

# 7. Persistence Entities / Records

Persistence entities are infrastructure representations of stored data.

They must not be treated as domain entities.

For example:

```text
Domain Collection
        |
        | mapper
        v
Persistence CollectionRecord
```

The persistence representation may contain:

- database identifiers
- foreign keys
- timestamps
- version columns
- normalized fields
- denormalized fields
- indexes
- technical metadata

These fields must not automatically become part of the domain model.

---

# 8. Domain-to-Persistence Mapping

Mapping must be explicit.

Conceptually:

```text
Domain Aggregate
       |
       | toPersistence()
       v
Persistence Representation
```

and:

```text
Persistence Representation
       |
       | toDomain()
       v
Domain Aggregate
```

The mapper is responsible for translating between models.

It must not introduce business decisions.

---

# 9. Persistence-to-Domain Reconstruction

Reconstruction is a critical architectural operation.

Loading an aggregate must result in a domain object that satisfies the assumptions required by the domain model.

The reconstruction process must therefore:

1. Load all required aggregate state.
2. Convert persistence primitives into domain value objects.
3. Restore relationships required by the aggregate.
4. Restore lifecycle state.
5. Restore version information where applicable.
6. Re-establish valid aggregate state.
7. Avoid triggering unrelated domain side effects.

A database row should never be considered equivalent to a valid domain object.

---

# 10. Value Object Mapping

Domain value objects must remain domain concepts.

Examples may include:

```text
CollectionId
ItemId
UserId
Name
Description
Quantity
Currency
Money
```

Persistence may store them as primitives:

```text
UUID
VARCHAR
INTEGER
DECIMAL
```

The mapping boundary is responsible for converting between these representations.

The domain must not be forced to use database primitives merely because they are convenient for persistence.

---

# 11. Identity Mapping

Persistence identity and domain identity must be explicitly aligned.

The preferred conceptual model is:

```text
Domain Identity
      |
      v
Persistence Identity
```

A persistence implementation must not generate a second independent identity unless there is a clear architectural reason.

When a domain aggregate already owns its identity, the persistence layer should normally preserve that identity.

---

# 12. Repository Loading Strategies

Repositories may use different loading strategies depending on aggregate requirements.

Possible strategies include:

### Eager loading

Used when the complete aggregate state is required for consistency.

```text
load aggregate
    -> all required state
```

### Lazy loading

Must be used carefully.

Lazy-loading infrastructure must not leak into the domain model.

The domain should never depend on an ORM proxy, session or persistence context being available.

### Explicit loading

Preferred when loading requirements are complex or use-case specific.

```text
Repository
    |
    +-- load aggregate
    +-- load required related state
    +-- reconstruct aggregate
```

The selected strategy must preserve aggregate consistency and avoid hidden database access from domain objects.

---

# 13. Query Components

Not every read operation should be implemented through aggregate repositories.

CollectionHub may distinguish between:

```text
Aggregate Repository
```

and:

```text
Read Query / Projection
```

Repositories are appropriate for operations requiring domain aggregates.

Read components are appropriate for:

- lists
- search results
- dashboards
- summaries
- reporting
- pagination
- filtering
- sorting
- projections

This avoids loading complete aggregates when only read-oriented data is required.

---

# 14. Command and Query Separation

The persistence architecture should support a practical separation between:

```text
Command side
    -> aggregate repositories
```

and:

```text
Query side
    -> optimized read components
```

This does not require a full CQRS architecture.

The principle is simply:

> Do not force every read requirement through the aggregate model.

---

# 15. Repository Save Semantics

`save()` represents persistence of an aggregate state transition.

The implementation must determine whether the aggregate is:

```text
new
```

or:

```text
existing
```

The distinction may be based on:

- aggregate identity
- persistence existence
- explicit lifecycle state
- repository semantics

The decision must remain transparent to the application layer.

---

# 16. Insert and Update Behavior

Persistence implementations should distinguish conceptually between:

```text
INSERT
```

and:

```text
UPDATE
```

although a persistence framework may abstract these operations.

Updates must preserve:

- identity
- aggregate relationships
- version information
- invariant-relevant state
- required historical metadata

Technical persistence metadata must not accidentally overwrite domain-owned values.

---

# 17. Delete Semantics

Deletion must be driven by domain/application requirements.

Possible strategies include:

```text
hard delete
soft delete
archive
state transition
```

The persistence implementation must not choose among these strategies implicitly.

If deletion has domain meaning, it should be represented as a domain/application operation rather than merely:

```text
repository.delete(id)
```

where that would bypass domain behavior.

---

# 18. Transaction Boundaries

Transactions must normally align with application use-case boundaries.

Conceptually:

```text
Application Use Case
        |
        | transaction
        v
Aggregate operations
        |
        v
Repository persistence
```

A transaction should encompass the state changes required to complete one consistent application operation.

Transactions should not be opened by domain entities.

---

# 19. Unit of Work

A Unit of Work abstraction may be used when required by the persistence technology.

Its responsibility may include:

- transaction lifecycle
- tracking changes
- coordinating repository operations
- committing changes
- rolling back changes

Conceptually:

```text
Application Service
        |
        v
UnitOfWork
        |
        +-- Repository A
        +-- Repository B
        |
        v
Commit
```

The Unit of Work must remain an infrastructure concern.

The domain must not depend on it.

---

# 20. Cross-Aggregate Persistence

Multiple aggregates may participate in one application transaction when the application use case requires atomic persistence.

However, the fact that multiple aggregates are persisted together does not make them one aggregate.

The distinction is:

```text
Aggregate boundary
    = domain consistency boundary
```

versus:

```text
Transaction boundary
    = application persistence boundary
```

These boundaries may coincide, but they do not have to.

---

# 21. Optimistic Concurrency

Optimistic concurrency should be considered for mutable aggregates where concurrent modification is possible.

A conceptual version field may be maintained:

```text
Aggregate
    version = 7
```

Persistence update:

```text
UPDATE ...
WHERE id = X
AND version = 7
```

Successful update:

```text
version = 8
```

If no row is updated, the repository must detect a concurrency conflict.

The resulting error must be translated into an appropriate application/infrastructure error.

---

# 22. Concurrency Conflict Handling

Concurrency conflicts must not be silently overwritten.

The preferred conceptual flow is:

```text
Load aggregate version N
        |
        v
Modify aggregate
        |
        v
Save expecting version N
        |
        +---- success -> N + 1
        |
        +---- conflict -> Concurrency Error
```

The application layer may then decide whether to:

- retry
- reject
- refresh
- inform the user

The persistence layer should not automatically retry business operations unless explicitly designed to do so.

---

# 23. Persistence Exceptions

Database-specific exceptions must not leak into the domain.

For example:

```text
SQL exception
ORM exception
driver exception
connection exception
constraint exception
```

should be translated at the infrastructure boundary.

Conceptually:

```text
Database Exception
        |
        v
Infrastructure Exception
        |
        v
Application Error
```

The exact error taxonomy must be defined consistently with the application error model.

---

# 24. Constraint Violations

Database constraints may protect structural integrity.

Examples:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
```

However, database constraints do not replace domain invariants.

The rule is:

```text
Domain invariants
    -> domain responsibility

Persistence structural constraints
    -> persistence responsibility
```

Both layers may enforce complementary guarantees.

---

# 25. Referential Integrity

Foreign keys may be used to guarantee structural relationships between persistence records.

They must not be interpreted as proof that the corresponding domain relationship is semantically valid.

For example:

```text
foreign key exists
```

does not necessarily imply:

```text
domain relationship is valid
```

Domain rules remain authoritative.

---

# 26. Persistence Lifecycle

A typical aggregate persistence lifecycle is:

```text
Application Use Case
        |
        v
Repository.load()
        |
        v
Persistence records
        |
        v
Mapper
        |
        v
Domain aggregate
        |
        v
Domain behavior
        |
        v
Repository.save()
        |
        v
Mapper
        |
        v
Persistence records
        |
        v
Transaction commit
```

This flow must remain independent from any specific ORM or database technology.

---

# 27. Domain Event Persistence

If domain events are persisted, event persistence must be treated separately from aggregate persistence.

Potential architecture:

```text
Aggregate
    |
    +-- state
    |
    +-- domain events
              |
              v
        Event Persistence
```

Domain event persistence must not be hidden inside domain objects.

If reliable publication is required, an Outbox mechanism may be introduced.

Conceptually:

```text
Aggregate state
      +
Outbox event
      |
      v
Same transaction
```

This guarantees that state and event publication intent are persisted atomically.

---

# 28. Outbox Consideration

An Outbox pattern should be considered when CollectionHub requires reliable integration with external systems.

Conceptually:

```text
Application Transaction
        |
        +-- Aggregate state
        |
        +-- Outbox message
        |
        v
      Commit
        |
        v
Outbox Processor
        |
        v
External System
```

The Outbox implementation belongs entirely to Infrastructure.

The domain should only produce domain events.

---

# 29. Repository Dependency Rules

Repository implementations must follow these rules.

### Rule 1

Infrastructure may depend on Domain.

### Rule 2

Infrastructure may implement application/domain repository contracts.

### Rule 3

Domain must never depend on repository implementation classes.

### Rule 4

Domain must never depend on ORM classes.

### Rule 5

Domain must never depend on database drivers.

### Rule 6

Persistence entities must not be passed into application/domain services.

### Rule 7

ORM sessions and persistence contexts must remain outside the domain.

### Rule 8

Mapping logic must remain outside aggregate behavior.

---

# 30. Repository Anti-Corruption Boundary

The repository implementation acts as an anti-corruption boundary between:

```text
Domain Model
```

and:

```text
Persistence Model
```

It prevents database concepts from contaminating the domain.

Therefore the repository layer must absorb differences such as:

- naming conventions
- normalization
- joins
- foreign keys
- surrogate keys
- database-specific types
- ORM lifecycle
- persistence proxies
- technical metadata

---

# 31. Avoiding Anemic Persistence Models

Persistence entities should not become accidental domain models.

The following pattern must be avoided:

```text
Database Entity
    |
    +-- business rules
    +-- domain decisions
    +-- workflow logic
```

Instead:

```text
Persistence Entity
    -> storage representation

Domain Entity
    -> behavior and invariants
```

The mapper connects the two.

---

# 32. Avoiding Active Record Leakage

If the selected technology provides Active Record semantics, those semantics must not automatically become the CollectionHub domain model.

Avoid:

```text
Domain Entity
    -> save()
    -> delete()
    -> query()
```

when those operations introduce persistence coupling.

Prefer:

```text
Domain Entity
        |
        v
Repository
        |
        v
Persistence
```

---

# 33. Repository Granularity

Repositories should normally exist at aggregate-root granularity.

Avoid repositories such as:

```text
CollectionItemRepository
CollectionTagRepository
CollectionMetadataRepository
```

when those objects are internal parts of a single aggregate.

Such repositories would bypass the aggregate consistency boundary.

A separate repository is justified when an entity represents an independent aggregate.

---

# 34. Specialized Queries

Some use cases may require specialized persistence queries.

Examples:

```text
findCollectionsByOwner(...)
searchCollections(...)
findItemsByCategory(...)
countItems(...)
getCollectionSummary(...)
```

These operations should be evaluated individually.

If they require only read data, a query component or projection may be preferable to extending the aggregate repository.

---

# 35. Pagination

Pagination belongs to the query/application boundary rather than the domain model.

Persistence query components may expose:

```text
page
pageSize
cursor
sort
filters
```

The domain should not need to understand database pagination mechanics.

Where possible, cursor-based pagination should be considered for large datasets.

---

# 36. Sorting and Filtering

Generic database filtering APIs must not leak into the domain.

Avoid exposing:

```text
SQL WHERE
ORM predicates
database expressions
```

to domain/application code.

Instead, application-level query criteria should represent meaningful use-case requirements.

The infrastructure translates them into persistence-specific queries.

---

# 37. Caching

Caching is an infrastructure concern.

A repository may internally use caching where justified:

```text
Application
    |
Repository
    |
    +-- Cache
    |
    +-- Database
```

The domain must remain unaware of whether data came from:

```text
cache
database
remote store
```

Caching must not compromise aggregate consistency.

---

# 38. Connection Management

Database connection management belongs entirely to Infrastructure.

Application services must not:

- open connections
- close connections
- select databases
- configure connection pools
- manage transactions directly through database APIs

These responsibilities belong to the infrastructure/runtime configuration layer.

---

# 39. Migration Responsibility

Database schema migrations are infrastructure artifacts.

They must be versioned independently from domain objects while remaining aligned with the persistence model.

Conceptually:

```text
Domain Model
     |
     v
Persistence Model
     |
     v
Schema Migration
```

Schema changes must be evaluated for their impact on:

- existing data
- repository mappings
- backward compatibility
- application deployment
- migration order

---

# 40. Persistence Testing Strategy

Persistence implementations require several test levels.

### Repository contract tests

Verify that implementations satisfy repository semantics.

### Mapping tests

Verify:

```text
Domain -> Persistence
Persistence -> Domain
```

### Integration tests

Verify real interaction with the selected database technology.

### Transaction tests

Verify:

- commit
- rollback
- atomicity
- concurrency behavior

### Migration tests

Verify schema creation and migration correctness.

---

# 41. Repository Contract Test Principle

The application/domain repository contract should be testable independently from the implementation details.

Conceptually:

```text
Repository Contract Tests
        |
        +-- SQL implementation
        +-- ORM implementation
        +-- Test implementation
```

This ensures that infrastructure implementations remain substitutable.

---

# 42. Test Doubles

Test doubles may be used at the application layer.

Examples:

```text
InMemoryCollectionRepository
FakeCollectionRepository
MockRepository
```

However, test doubles must preserve the semantic contract of the real repository.

An in-memory implementation must not become a second, inconsistent domain model.

---

# 43. Persistence Observability

Persistence components should expose technical observability where required.

Potential metrics include:

- query duration
- transaction duration
- connection pool usage
- repository operation counts
- failed transactions
- concurrency conflicts
- database errors

Observability must not alter domain behavior.

---

# 44. Logging

Persistence logs must contain enough information to diagnose technical failures without exposing sensitive data.

Logging should focus on:

```text
operation
aggregate type
operation outcome
duration
technical error
correlation identifier
```

Avoid logging complete domain objects or sensitive persisted data.

---

# 45. Performance Boundaries

Performance optimizations must remain inside infrastructure unless they change an explicit application/domain requirement.

Possible optimizations include:

- indexes
- batching
- query projections
- connection pooling
- caching
- prepared statements
- bulk persistence

These optimizations must not compromise domain correctness.

---

# 46. Bulk Operations

Bulk operations require special treatment.

A bulk database operation may bypass:

- aggregate behavior
- domain invariants
- domain events
- concurrency checks

Therefore bulk persistence must only be used when the operation semantics explicitly permit it.

A generic:

```text
UPDATE all records
```

must never silently replace aggregate-level domain behavior.

---

# 47. Persistence Consistency Rules

The following rules are mandatory:

1. Aggregates are persisted through their aggregate roots.
2. Domain invariants remain independent from persistence constraints.
3. Persistence entities are not domain entities.
4. ORM objects do not cross into the domain.
5. Mapping is explicit.
6. Transactions are controlled outside the domain.
7. Repository implementations remain infrastructure concerns.
8. Database exceptions are translated at the infrastructure boundary.
9. Concurrent updates must not be silently lost.
10. Read projections must not be forced through aggregate loading.
11. Bulk operations must not bypass domain semantics accidentally.
12. Domain events must remain independent from event transport mechanisms.

---

# 48. Architectural Decision Summary

The persistence architecture establishes the following decisions.

| Decision | Status |
|---|---|
| Aggregate-oriented repositories | Adopted |
| Repository abstractions separated from implementations | Adopted |
| Persistence entities separated from domain entities | Adopted |
| Explicit domain/persistence mapping | Adopted |
| ORM/database independence in Domain | Mandatory |
| Application-oriented transaction boundaries | Adopted |
| Optimistic concurrency | Supported |
| Specialized read queries | Supported |
| CQRS | Not mandatory |
| Outbox | Conditional |
| Caching | Infrastructure concern |
| Bulk operations | Restricted |
| Database constraints replacing domain invariants | Rejected |

---

# 49. Open Questions

The following questions remain implementation-level decisions for later phases:

1. Which database technology will be selected?
2. Which ORM or persistence framework, if any, will be used?
3. Will repositories use synchronous or asynchronous APIs?
4. Will optimistic locking be required for every mutable aggregate?
5. Will an Outbox be required?
6. Which read models require dedicated projections?
7. What migration framework will be used?
8. What level of database normalization is appropriate?
9. Which indexes are required by expected query patterns?
10. Is caching necessary for any high-frequency access path?
11. What transaction isolation level is required?
12. How will integration tests provision the persistence environment?
13. Which persistence errors should become explicit application errors?
14. Are soft deletion or archival semantics required for any aggregate?
15. Which data-retention requirements affect persistence design?

These questions should be resolved before concrete persistence implementation begins.

---

# 50. Final Architectural Position

CollectionHub persistence is defined as an **Infrastructure concern implementing domain/application persistence contracts while protecting the Domain Model from persistence technology**.

The essential boundary is:

```text
                 APPLICATION
                      |
                      v
               Repository Port
                      |
          +-----------+-----------+
          |                       |
          v                       v
   Repository Impl          Read Queries
          |
          v
       Mappers
          |
          v
 Persistence Records
          |
          v
       Database
```

The central architectural rule is:

> Persistence exists to store and reconstruct domain state; it must never become the owner of domain behavior.

This boundary allows CollectionHub to evolve its database, ORM, query strategy, caching model or persistence infrastructure without requiring the Domain Model to absorb those technical concerns.

The persistence implementation is therefore considered replaceable infrastructure, while the domain model remains the authoritative representation of business behavior and invariants.