# CollectionHub — Domain Error Model

## 1. Purpose

This document defines the error model of CollectionHub.

It establishes:

- what constitutes a domain error;
- how business rule violations are represented conceptually;
- how domain errors differ from application errors;
- how technical failures differ from business failures;
- how errors relate to aggregates, policies, commands and use cases;
- which errors are retryable;
- which errors are deterministic;
- how error semantics should propagate across application boundaries.

This document builds on:

- `07_DOMAIN_COMMANDS_AND_EVENTS.md`
- `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`
- `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`
- `10_DOMAIN_SERVICES_AND_POLICIES.md`
- `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md`
- `12_APPLICATION_USE_CASES_AND_WORKFLOWS.md`

This document does not define:

- HTTP status codes;
- REST error payloads;
- database exception classes;
- framework exceptions;
- message broker errors;
- logging formats;
- infrastructure-specific exception hierarchies.

---

# 2. Error Model Philosophy

An error is meaningful only when its semantic origin is understood.

CollectionHub distinguishes between:

    Domain
       ↓
    Application
       ↓
    Infrastructure

Each layer has different failure semantics.

The fundamental rule is:

> A business rule violation is not a technical failure.

For example:

    Item already belongs to Collection

is fundamentally different from:

    Database connection unavailable

The first represents a valid domain decision.

The second represents an inability of the system to complete an otherwise valid operation.

---

# 3. Error Categories

The current model defines five conceptual categories:

1. Domain Rule Violation
2. Not Found
3. Conflict
4. Application Validation Error
5. Infrastructure Failure

Conceptually:

    Error
      |
      +-- Domain
      |     +-- Rule Violation
      |     +-- Invalid State
      |
      +-- Application
      |     +-- Invalid Input
      |     +-- Authorization
      |
      +-- Resource
      |     +-- Not Found
      |     +-- Conflict
      |
      +-- Infrastructure
            +-- Persistence
            +-- Messaging
            +-- External Dependency

The final technical hierarchy will be defined later.

---

# 4. Domain Error

A Domain Error represents a condition where:

> The requested business operation cannot legally be performed according to the domain model.

Examples:

    DuplicateMembership
    InvalidCollectionName
    InvalidItemClassification
    CollectionModificationNotAllowed

A Domain Error is deterministic with respect to the relevant domain state and input.

---

# 5. Domain Errors Are Expected Outcomes

A domain error is not necessarily an exceptional system failure.

For example:

    Add Item to Collection
          ↓
    membership already exists
          ↓
    DuplicateMembership

The system has behaved correctly.

The domain has rejected an invalid operation.

Therefore:

> Domain rejection is part of normal domain behavior.

---

# 6. Domain Error vs Technical Exception

Consider:

    Collection.addItem(ItemId)

Case A:

    Item already exists in Collection

Result:

    DuplicateMembership

Case B:

    Database is unavailable

Result:

    Infrastructure failure

The aggregate is responsible for Case A.

The infrastructure is responsible for Case B.

The two must not be semantically merged.

---

# 7. Domain Error Properties

A domain error should conceptually provide enough information to identify:

- error type;
- affected domain concept;
- relevant identifiers when appropriate;
- violated rule;
- semantic reason;
- whether the caller may change the request and retry.

It should not depend on:

- HTTP;
- SQL;
- ORM;
- message broker;
- framework exception classes.

---

# 8. Error Naming

Domain errors should use business terminology.

Preferred:

    DuplicateMembership
    InvalidCollectionName
    ItemNotCompatibleWithCollection
    CollectionCannotBeModified

Avoid:

    ValidationException
    BadRequestException
    SqlConstraintException
    EntityAlreadyExistsException

The preferred names describe **why the domain rejected the operation**.

---

# 9. Domain Error Inventory

Current candidate domain errors:

    DuplicateMembership
    MembershipNotFound
    InvalidCollectionName
    CollectionModificationNotAllowed
    InvalidItemMetadata
    InvalidItemClassification
    ItemClassificationChangeNotAllowed
    ItemNotCompatibleWithCollection
    InvalidCollectionState
    InvalidItemState

These remain subject to refinement as the domain model evolves.

---

# 10. Membership Errors

## 10.1 DuplicateMembership

Meaning:

> The requested Item is already a member of the Collection.

Example:

    Add Item X
          ↓
    Collection already contains X
          ↓
    DuplicateMembership

This protects the membership uniqueness invariant.

---

## 10.2 MembershipNotFound

Meaning:

> The requested membership does not exist.

Example:

    Remove Item X
          ↓
    X is not a member
          ↓
    MembershipNotFound

Whether this error remains necessary depends on the final idempotency decision for removal.

---

# 11. Collection Errors

## 11.1 InvalidCollectionName

Meaning:

> The supplied Collection name violates domain rules.

Examples may include:

- empty semantic value;
- prohibited format;
- invalid length;
- invalid normalized representation.

The exact rules remain defined by the domain invariant model.

---

## 11.2 CollectionModificationNotAllowed

Meaning:

> The Collection currently cannot accept the requested modification.

This should only exist if Collection lifecycle semantics require such a restriction.

---

## 11.3 InvalidCollectionState

Meaning:

> The requested operation cannot be performed because the Collection is in a domain state incompatible with the operation.

This error should not be used as a generic replacement for more precise errors.

Prefer a specific semantic error when one exists.

---

# 12. Item Errors

## 12.1 InvalidItemMetadata

Meaning:

> The supplied metadata violates Item metadata rules.

---

## 12.2 InvalidItemClassification

Meaning:

> The supplied classification is not valid for the Item or domain.

---

## 12.3 ItemClassificationChangeNotAllowed

Meaning:

> The classification itself may be valid, but the transition from the current classification is not permitted.

This distinction is important.

    InvalidClassification
        ≠
    InvalidTransition

---

## 12.4 InvalidItemState

Meaning:

> The Item is currently in a domain state incompatible with the requested operation.

Again, this should only be used where a lifecycle model actually requires it.

---

# 13. Cross-Aggregate Errors

## 13.1 ItemNotCompatibleWithCollection

Meaning:

> The requested Item cannot validly belong to the specified Collection.

This may originate from a compatibility policy.

Conceptually:

    Collection
         +
       Item
         ↓
    Compatibility Policy
         ↓
    reject
         ↓
    ItemNotCompatibleWithCollection

The exact policy semantics belong to the domain model.

---

# 14. Not Found

Not Found is distinct from a domain rule violation.

Example:

    Rename Collection
          ↓
    CollectionId does not exist
          ↓
    CollectionNotFound

The requested business operation may be perfectly valid.

The required domain object simply does not exist.

Candidate errors:

    CollectionNotFound
    ItemNotFound

---

# 15. Not Found Semantics

A Not Found result normally means:

> The requested domain identity cannot be resolved in the current consistency context.

It does not necessarily mean:

> The request is invalid.

This distinction becomes important when mapping domain/application outcomes to external interfaces.

---

# 16. CollectionNotFound

Meaning:

> No Collection corresponding to the supplied identity can be found.

Potential workflows:

    Rename Collection
    Add Item
    Remove Item

---

# 17. ItemNotFound

Meaning:

> No Item corresponding to the supplied identity can be found.

Potential workflows:

    Add Item to Collection
    Update Item
    Change Item Classification

---

# 18. Application Validation Errors

Application validation concerns structural correctness before domain execution.

Examples:

    MissingCollectionId
    MissingItemId
    MissingCollectionName
    InvalidIdentifierFormat
    MissingRequiredField

These are not domain errors.

They answer:

> Can the application interpret this request?

rather than:

> Is this business operation valid?

---

# 19. Structural vs Semantic Validation

The distinction is:

    Structural Validation
            ↓
    "Is this input correctly formed?"

    Domain Validation
            ↓
    "Is this operation semantically valid?"

Example:

    CollectionId = null

is structural invalidity.

Whereas:

    Collection name violates business naming rule

is domain invalidity.

---

# 20. Authorization Errors

Authorization is conceptually separate.

Example:

    User is authenticated
          ↓
    User is not authorized
          ↓
    operation rejected

This is not necessarily a domain invariant.

It belongs primarily to the application/security boundary.

However, if ownership is itself a domain concept, some ownership rules may belong to the domain.

---

# 21. Conflict

Conflict represents a condition where an operation cannot safely complete because the current state conflicts with the requested operation or concurrent state.

Examples:

    OptimisticConcurrencyConflict
    DuplicateMembership
    ConflictingUpdate

However, semantic domain conflicts and technical concurrency conflicts must remain distinguishable.

---

# 22. Domain Conflict vs Concurrency Conflict

These are different:

### Domain Conflict

    Add Item
       ↓
    Item already belongs
       ↓
    DuplicateMembership

### Concurrency Conflict

    Request A loads Collection version 5
    Request B updates Collection to version 6
    Request A tries to save version 5

Result:

    OptimisticConcurrencyConflict

The first is a business rule.

The second is a consistency mechanism.

---

# 23. Infrastructure Errors

Infrastructure failures occur outside the domain.

Examples:

    DatabaseUnavailable
    PersistenceTimeout
    MessageBrokerUnavailable
    EventPublicationFailure
    ExternalServiceUnavailable

The domain should not know these names.

---

# 24. Persistence Failure

Example:

    Collection.rename(...)
          ↓
    valid domain state
          ↓
    persistence
          ↓
    database unavailable

The domain operation was valid.

The application could not complete persistence.

Therefore:

    PersistenceFailure

must remain separate from:

    InvalidCollectionName

---

# 25. Event Publication Failure

Consider:

    Collection.addItem(...)
          ↓
    ItemAddedToCollection
          ↓
    state persisted
          ↓
    event publication fails

This is not:

    ItemAddedToCollectionFailed

The domain fact still occurred.

The architecture must provide a reliable mechanism for publication.

The technical solution is outside this document.

---

# 26. External Dependency Failure

Future workflows may depend on external systems.

Examples:

    metadata provider unavailable
    external catalog unavailable
    identity provider unavailable

These are technical/application failures, not domain rule violations.

---

# 27. Retryability

Errors should conceptually be classified by retry behavior.

## Non-Retryable

The request itself must change.

Examples:

    InvalidCollectionName
    DuplicateMembership
    InvalidItemClassification

Retrying the same request without changing state/input is unlikely to succeed.

---

## Potentially Retryable

The request is valid but the environment may recover.

Examples:

    DatabaseUnavailable
    MessageBrokerUnavailable
    ExternalServiceUnavailable

---

## Conditionally Retryable

Retry may succeed depending on state.

Examples:

    OptimisticConcurrencyConflict
    MembershipNotFound

The caller/application may need to refresh state before retrying.

---

# 28. Retry Matrix

| Error | Retry Same Request? | Typical Action |
|---|---:|---|
| InvalidCollectionName | No | Change input |
| DuplicateMembership | No | Treat according to idempotency semantics |
| MembershipNotFound | Usually No | Refresh state |
| CollectionNotFound | No | Verify identity |
| ItemNotFound | No | Verify identity |
| InvalidItemClassification | No | Change input |
| ItemNotCompatibleWithCollection | No | Change relationship |
| PersistenceFailure | Maybe | Retry according to infrastructure policy |
| MessagePublicationFailure | Maybe | Retry publication |
| OptimisticConcurrencyConflict | Not blindly | Reload and retry |
| ExternalServiceUnavailable | Maybe | Retry with backoff |

---

# 29. Deterministic vs Transient Errors

A useful conceptual classification is:

    Error
      |
      +-- Deterministic
      |      |
      |      +-- same input/state → same result
      |
      +-- Transient
             |
             +-- environment may change result

Domain rule violations are generally deterministic.

Infrastructure failures may be transient.

This distinction is valuable for future retry architecture.

---

# 30. Error Ownership

Errors should have an identifiable owner.

| Error | Primary Owner |
|---|---|
| InvalidCollectionName | Collection |
| DuplicateMembership | Collection |
| MembershipNotFound | Collection |
| ItemNotCompatibleWithCollection | Compatibility Policy |
| InvalidItemMetadata | Item |
| InvalidItemClassification | Item |
| CollectionNotFound | Application/Repository boundary |
| ItemNotFound | Application/Repository boundary |
| AuthorizationDenied | Security/Application |
| OptimisticConcurrencyConflict | Infrastructure/Application |
| DatabaseUnavailable | Infrastructure |
| EventPublicationFailure | Infrastructure |

This table is intentionally conceptual.

---

# 31. Aggregate Error Responsibility

An aggregate should reject operations that would violate its invariants.

Example:

    Collection.addItem(ItemId)

must be capable of rejecting:

    DuplicateMembership

The application must not have to know the internal representation of membership to enforce this rule.

---

# 32. Policy Error Responsibility

A domain policy may reject a relationship or decision.

Example:

    CompatibilityPolicy
          ↓
    incompatible
          ↓
    ItemNotCompatibleWithCollection

The policy should express the business reason.

---

# 33. Application Error Responsibility

The application layer is responsible for translating or coordinating errors from different sources.

For example:

    Use Case
       |
       +-- CollectionNotFound
       +-- DuplicateMembership
       +-- PersistenceFailure

The application layer may normalize these into an application contract later.

It must not erase their semantic distinction.

---

# 34. Error Propagation

Conceptually:

    Domain
       ↓
    Domain Error
       ↓
    Application Workflow
       ↓
    Application Result
       ↓
    External Contract

Technical infrastructure failures follow a different path:

    Infrastructure
       ↓
    Technical Failure
       ↓
    Application Boundary
       ↓
    Application Result
       ↓
    External Contract

---

# 35. External Error Mapping

The domain error model does not prescribe HTTP semantics.

For example, later an API may map:

    CollectionNotFound
          ↓
    HTTP 404

and:

    DuplicateMembership
          ↓
    HTTP 409

But those mappings belong to the API/interface model.

The domain must remain transport-independent.

---

# 36. Error Codes

Stable machine-readable error codes may eventually be useful.

Conceptual examples:

    COLLECTION_NAME_INVALID
    COLLECTION_NOT_FOUND
    MEMBERSHIP_ALREADY_EXISTS
    MEMBERSHIP_NOT_FOUND
    ITEM_NOT_FOUND
    ITEM_CLASSIFICATION_INVALID
    ITEM_NOT_COMPATIBLE

However, error codes are application contracts rather than domain concepts.

Their final definition should be made later.

---

# 37. Error Messages

Human-readable messages should not be the primary semantic identity of an error.

Bad:

    "Something went wrong"

Better conceptual identity:

    DuplicateMembership

A localized message can later be derived from the semantic error.

---

# 38. Sensitive Information

Error information must not unnecessarily expose:

- database internals;
- SQL statements;
- stack traces;
- infrastructure topology;
- credentials;
- internal identifiers not intended for external callers.

The domain itself should not contain infrastructure details.

---

# 39. Error Context

Where useful, errors may carry contextual information.

For example:

    DuplicateMembership
        CollectionId
        ItemId

or:

    InvalidItemClassification
        ItemId
        RequestedClassification

However, contextual data should be limited to what is semantically meaningful and safe to expose.

---

# 40. Error Causality

Technical failures may preserve underlying causes internally.

Example:

    PersistenceFailure
          caused by
    DatabaseTimeout

The domain should never depend on the database exception itself.

The causal chain belongs to the application/infrastructure diagnostic model.

---

# 41. Error Aggregation

Some workflows may encounter multiple validation problems.

The architecture must distinguish:

    single domain rejection

from:

    multiple input validation failures

For example:

    Create Item
       ↓
    invalid metadata
    invalid classification
    missing identity

This is primarily an application input-validation concern.

A domain aggregate should not necessarily return a large collection of unrelated errors unless the domain semantics require it.

---

# 42. Fail-Fast vs Collect-All

Application validation may use:

    collect-all

when useful for client feedback.

Domain behavior generally follows:

    fail-fast

when an operation violates an invariant.

The exact strategy may vary by operation.

---

# 43. Error and Events

A failed domain operation should not produce a success event.

Example:

    Add Item
       ↓
    DuplicateMembership
       ↓
    no ItemAddedToCollection

This distinction is essential.

---

# 44. Error and Partial State

A failed aggregate operation must not leave the aggregate in an invalid partial state.

The domain operation should preserve:

    valid state
       ↓
    attempted operation
       ↓
    either
       |
       +-- valid new state
       |
       +-- original valid state + domain error

The aggregate must not expose intermediate invalid states as successful domain state.

---

# 45. Workflow Failure Example

For:

    Add Item to Collection

failure path:

    Trigger
      ↓
    Load Collection
      ↓
    Load Item
      ↓
    Compatibility Policy
      ↓
    incompatible
      ↓
    ItemNotCompatibleWithCollection
      ↓
    no Collection mutation
      ↓
    no ItemAddedToCollection event

---

# 46. Persistence Failure Example

For:

    Rename Collection

    Load Collection
       ↓
    Collection.rename(...)
       ↓
    valid new state
       ↓
    persist
       ↓
    persistence failure

The application must treat this as:

    PersistenceFailure

not:

    InvalidCollectionName

The distinction preserves domain truth.

---

# 47. Concurrency Failure Example

    Request A
       ↓
    load Collection version 5

    Request B
       ↓
    update Collection
       ↓
    version 6

    Request A
       ↓
    save version 5
       ↓
    OptimisticConcurrencyConflict

The domain operation may have been valid according to the state observed by A.

The failure arises from consistency enforcement.

---

# 48. Error Taxonomy

Current conceptual taxonomy:

    CollectionHubError
    |
    +-- DomainError
    |     |
    |     +-- CollectionError
    |     |     +-- InvalidCollectionName
    |     |     +-- CollectionModificationNotAllowed
    |     |
    |     +-- MembershipError
    |     |     +-- DuplicateMembership
    |     |     +-- MembershipNotFound
    |     |
    |     +-- ItemError
    |           +-- InvalidItemMetadata
    |           +-- InvalidItemClassification
    |           +-- ItemClassificationChangeNotAllowed
    |           +-- InvalidItemState
    |
    +-- ResourceError
    |     +-- CollectionNotFound
    |     +-- ItemNotFound
    |
    +-- ApplicationError
    |     +-- InvalidInput
    |     +-- AuthorizationDenied
    |
    +-- InfrastructureError
          +-- PersistenceFailure
          +-- EventPublicationFailure
          +-- ExternalServiceFailure
          +-- ConcurrencyConflict

This hierarchy is conceptual rather than a final programming-language class hierarchy.

---

# 49. Domain Error Invariants

The following principles are mandatory:

1. Domain errors must not depend on infrastructure.
2. Domain errors must express business meaning.
3. Aggregate invariants must be protected by aggregate behavior.
4. Policies must express policy-related rejection.
5. Application validation must remain distinct from domain validation.
6. Technical failures must remain distinct from business failures.
7. Failed domain operations must not emit successful domain events.
8. Error semantics must survive application-layer translation.

---

# 50. Use Case Error Matrix

| Use Case | Possible Domain/Application Errors |
|---|---|
| Create Collection | InvalidCollectionName, InvalidInput |
| Rename Collection | CollectionNotFound, InvalidCollectionName, CollectionModificationNotAllowed |
| Add Item | CollectionNotFound, ItemNotFound, DuplicateMembership, ItemNotCompatibleWithCollection |
| Remove Item | CollectionNotFound, MembershipNotFound, CollectionModificationNotAllowed |
| Create Item | InvalidInput, InvalidItemMetadata, InvalidItemClassification |
| Update Item Metadata | ItemNotFound, InvalidItemMetadata, InvalidItemState |
| Change Item Classification | ItemNotFound, InvalidItemClassification, ItemClassificationChangeNotAllowed |

---

# 51. Error Traceability

Every domain error should eventually be traceable to:

    Domain Concept
          ↓
    Invariant / Rule
          ↓
    Domain Operation
          ↓
    Use Case
          ↓
    Workflow
          ↓
    Error

Example:

    Membership uniqueness invariant
          ↓
    Collection.addItem(...)
          ↓
    Add Item to Collection
          ↓
    WF-COL-003
          ↓
    DuplicateMembership

This traceability prevents arbitrary error creation.

---

# 52. Errors Without Rules

If an error cannot be associated with:

- a domain rule;
- an application constraint;
- a resource lookup;
- or a technical failure,

it should be questioned.

For example:

    CollectionOperationFailed

is too generic.

The model should identify the actual reason.

---

# 53. Generic Errors

Generic errors may exist at technical boundaries for safety.

For example:

    InternalApplicationError

may be useful externally.

However, internally the original semantic cause should remain available.

Generic external representation must not replace internal semantic classification.

---

# 54. Error Translation Principle

Errors may be translated between layers, but their meaning must remain stable.

Example:

    Domain:
        DuplicateMembership

    Application:
        MembershipAlreadyExists

    API:
        MEMBERSHIP_ALREADY_EXISTS

These may be different representations of the same semantic outcome.

The translation must not turn it into:

    INTERNAL_SERVER_ERROR

unless the semantic information is intentionally hidden at an external security boundary.

---

# 55. Error Logging

Domain errors should generally not be logged as infrastructure incidents merely because they are failures.

For example:

    DuplicateMembership

is expected business behavior.

It may be recorded for diagnostics or analytics, but it should not automatically be treated as a system outage.

Infrastructure failures may require operational alerting.

---

# 56. Monitoring Implications

The taxonomy enables future monitoring distinctions:

    Business Rejections
        ↓
    Domain Rule Violations

versus:

    Technical Failures
        ↓
    Infrastructure Errors

This prevents business-level rejected operations from artificially inflating infrastructure error rates.

---

# 57. Open Questions

## OPEN-ERR-001

Should domain errors use a structured error-code system?

## OPEN-ERR-002

Which errors should be exposed directly through public application contracts?

## OPEN-ERR-003

Should `MembershipNotFound` be retained if removal becomes idempotent?

## OPEN-ERR-004

Should `InvalidCollectionState` and `InvalidItemState` be replaced by more precise lifecycle errors?

## OPEN-ERR-005

Where exactly should Not Found semantics live: application boundary or domain repository abstraction?

## OPEN-ERR-006

Which authorization failures are application concerns versus domain ownership rules?

## OPEN-ERR-007

Which external dependency failures require explicit domain-facing semantics?

## OPEN-ERR-008

Which errors are safe to retry automatically?

---

# 58. Completion Criteria

This document is considered sufficiently mature when:

- domain errors are explicitly identified;
- application errors are separated from domain errors;
- infrastructure failures are separated from business failures;
- aggregate-owned errors are identified;
- policy-owned errors are identified;
- not-found semantics are defined;
- concurrency conflicts are distinguished;
- retryability is considered;
- error/event semantics are defined;
- error traceability exists;
- transport-specific error mapping remains deferred.

---

# 59. Final Principle

The CollectionHub error model follows one central rule:

> **An error must describe why an operation could not produce its intended outcome without confusing business rejection with technical failure.**

Therefore:

    Invalid Domain Operation
             ↓
        Domain Error

    Missing Domain Resource
             ↓
        Resource Error

    Invalid Application Request
             ↓
       Application Error

    Technical Execution Failure
             ↓
     Infrastructure Error

The domain remains authoritative for business rejection.

The application layer coordinates and translates.

Infrastructure reports technical failure.

This separation is essential for preserving the semantic integrity of the system as CollectionHub grows.