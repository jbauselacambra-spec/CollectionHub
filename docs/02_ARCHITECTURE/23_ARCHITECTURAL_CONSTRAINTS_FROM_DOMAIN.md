# Architectural Constraints From Domain

**CollectionHub — Phase 2.3: Domain-to-Architecture Translation**

**Status:** Draft / Architectural Baseline  
**Document:** `23_ARCHITECTURAL_CONSTRAINTS_FROM_DOMAIN.md`  
**Location:** `CollectionHub/docs/02_ARCHITECTURE/`  
**Purpose:** Translate the validated CollectionHub domain model into explicit architectural constraints that must guide all subsequent architectural and technical decisions.

---

# 1. Purpose

This document establishes the architectural constraints derived directly from the completed Phase 2.2 domain model.

Its purpose is to answer:

> **Given the domain model we have established, what must the architecture preserve?**

The architecture must not begin from framework conventions, database structures, deployment preferences, or generic software patterns.

Instead:

```text
Domain Model
     ↓
Domain Constraints
     ↓
Architectural Constraints
     ↓
Architectural Structure
     ↓
Technical Implementation
```

The constraints defined here therefore become the initial architectural contract for CollectionHub.

---

# 2. Architectural Context

The domain modeling phase established:

- canonical domain terminology;
- business concepts;
- business capabilities;
- relationships;
- lifecycles;
- policies;
- business rules;
- invariants;
- aggregates;
- consistency boundaries;
- entities;
- value objects;
- use cases;
- application workflows;
- commands;
- state transitions;
- repositories;
- domain events;
- domain services;
- specifications;
- traceability;
- domain decisions.

The architecture phase must preserve those conclusions.

The architecture is therefore constrained by the domain rather than defining the domain.

---

# 3. Architectural Principles Derived From the Domain

The following principles are mandatory architectural constraints.

## 3.1 Domain independence

The domain model must not depend on:

- web frameworks;
- database frameworks;
- HTTP;
- UI technologies;
- messaging infrastructure;
- cloud providers;
- serialization formats;
- external APIs.

The dependency direction must point toward the domain.

```text id="8t7m6q"
Infrastructure
      ↓
Application
      ↓
Domain
```

Never:

```text id="m3j2kn"
Domain
   ↓
Infrastructure
```

---

# 4. Constraint AC-001 — Domain Must Remain Framework Independent

**Category:** Dependency  
**Severity:** Critical

The core domain must not require a specific framework to execute its business behavior.

The following concepts belong outside the domain:

- HTTP controllers;
- ORM entities;
- database contexts;
- message brokers;
- HTTP clients;
- framework annotations;
- dependency injection containers.

### Architectural implication

The domain must be implemented as a framework-independent core.

```text id="qj9q6g"
┌──────────────────────────┐
│        DOMAIN            │
│                          │
│ Entities                 │
│ Value Objects            │
│ Aggregates              │
│ Domain Services          │
│ Specifications           │
│ Domain Events            │
│ Invariants               │
└──────────────────────────┘
```

---

# 5. Constraint AC-002 — Aggregate Boundaries Must Drive Consistency Boundaries

**Category:** Consistency  
**Severity:** Critical

Aggregate boundaries established in the domain model must determine where atomic consistency is required.

The architecture must not create broader transactions merely because multiple tables or objects are conveniently available.

```text id="6zj4cg"
Aggregate
   ↓
Consistency Boundary
   ↓
Transaction Boundary
```

This does not imply that every aggregate must have its own database transaction in every implementation, but it does require the architecture to preserve the semantic boundary.

---

# 6. Constraint AC-003 — Aggregate Roots Control Mutation

**Category:** Encapsulation  
**Severity:** Critical

External application components must not modify aggregate internals directly.

The architectural dependency should be:

```text id="0m0l6j"
Application
     ↓
Aggregate Root
     ↓
Domain Behavior
     ↓
State Change
```

Not:

```text id="xqj4gs"
Application
     ↓
Entity
     ↓
Property mutation
```

This protects domain invariants from bypass.

---

# 7. Constraint AC-004 — Domain Behavior Must Live in the Domain

**Category:** Behavioral integrity  
**Severity:** Critical

Business decisions must not migrate into:

- controllers;
- API handlers;
- application services;
- repositories;
- persistence adapters;
- UI components.

Application services may orchestrate.

They must not become a second domain model.

```text id="3b1v0w"
Application Service
        ↓
Orchestration
        ↓
Domain Behavior
```

---

# 8. Constraint AC-005 — Application Layer Must Orchestrate Use Cases

**Category:** Application architecture  
**Severity:** High

The application layer must provide use-case orchestration.

Responsibilities include:

- receiving commands;
- establishing execution context;
- loading aggregates;
- invoking domain behavior;
- coordinating repositories;
- collecting domain events;
- initiating event publication;
- returning application results.

It must not own business invariants.

---

# 9. Constraint AC-006 — Domain Rules Must Have Explicit Owners

**Category:** Domain integrity  
**Severity:** Critical

Every important rule must have a clearly identifiable owner.

Possible owners:

```text id="0f0b2n"
Value Object
Entity
Aggregate
Domain Service
Specification
Application Policy
```

A rule must never exist only because a particular controller or repository happens to enforce it.

---

# 10. Constraint AC-007 — Invariants Must Be Enforced at the Correct Boundary

**Category:** Consistency  
**Severity:** Critical

An invariant must be enforced where all required state is available.

If the invariant belongs to one aggregate:

```text id="qf7i6j"
Invariant
   ↓
Aggregate
```

If it genuinely spans independent aggregates:

```text id="8u5k0h"
Cross-Aggregate Rule
       ↓
Domain Service / Policy
       ↓
Explicit Coordination
```

The architecture must not hide cross-aggregate rules inside persistence implementations.

---

# 11. Constraint AC-008 — Cross-Aggregate Access Must Be Explicit

**Category:** Coupling  
**Severity:** High

Aggregates must not expose internal object graphs for arbitrary traversal.

Cross-aggregate collaboration should use explicit mechanisms such as:

- aggregate identifiers;
- domain services;
- specifications;
- repositories;
- domain events;
- application orchestration.

The architecture must make cross-boundary dependencies visible.

---

# 12. Constraint AC-009 — Domain Events Represent Domain Facts

**Category:** Events  
**Severity:** High

Domain events must originate from meaningful domain transitions.

```text id="3y2z6r"
Command
   ↓
Aggregate
   ↓
Valid Transition
   ↓
Domain Event
```

The architecture must not make controllers the authoritative source of domain events.

---

# 13. Constraint AC-010 — Domain Events Must Be Decoupled From Transport

**Category:** Integration  
**Severity:** High

A domain event must not be defined in terms of:

- Kafka;
- RabbitMQ;
- HTTP;
- Azure Service Bus;
- AWS SNS/SQS;
- Redis Streams;
- or another specific transport.

The domain should express:

```text id="9q5jvw"
ItemRelocated
```

rather than:

```text id="g9y0u8"
RabbitMQItemRelocatedMessage
```

Transport belongs to infrastructure.

---

# 14. Constraint AC-011 — Side Effects Must Be Downstream From Domain Facts

**Category:** Side effects  
**Severity:** High

External side effects must not determine whether a domain operation is valid.

The desired direction is:

```text id="8j2x3n"
Domain State Change
       ↓
Domain Event
       ↓
Side Effect
```

Examples:

- update read model;
- write audit information;
- notify external systems;
- update search indexes.

The side effect must not redefine the source domain state.

---

# 15. Constraint AC-012 — Strong and Eventual Consistency Must Be Explicit

**Category:** Consistency  
**Severity:** Critical

The architecture must explicitly distinguish:

### Strong consistency

Used for invariants that must hold immediately.

```text id="g8h0g3"
Command
   ↓
Aggregate
   ↓
Atomic state change
```

### Eventual consistency

Used for derived or secondary representations.

```text id="9t1w8f"
Aggregate
   ↓
Domain Event
   ↓
Projection
```

The architecture must never introduce eventual consistency accidentally into a business invariant.

---

# 16. Constraint AC-013 — Persistence Must Follow Domain Boundaries

**Category:** Persistence  
**Severity:** Critical

Persistence structures must support the domain model rather than redefine it.

The architecture must avoid:

```text id="x1v4f7"
Database Tables
      ↓
ORM Models
      ↓
Domain Model
```

The preferred direction is:

```text id="7k5b3p"
Domain Model
      ↓
Persistence Requirements
      ↓
Persistence Model
```

Database design will therefore be performed after the domain and architectural boundaries are established.

---

# 17. Constraint AC-014 — Repository Contracts Must Be Domain-Oriented

**Category:** Persistence  
**Severity:** High

Repository contracts must express domain needs.

They must not expose:

- SQL;
- ORM sessions;
- database transactions;
- connection details;
- persistence-specific entities.

A repository should represent a persistence abstraction around an aggregate or domain-relevant query.

---

# 18. Constraint AC-015 — Read Operations Must Not Necessarily Load Aggregates

**Category:** Query architecture  
**Severity:** Medium

Not every query should force aggregate reconstruction.

For read-oriented operations such as:

- search;
- filtering;
- listing;
- reporting;
- projections;

the architecture may use dedicated read models.

```text id="m7b9qf"
Command Side
    ↓
Aggregates
    ↓
Events
    ↓
Read Models
    ↓
Queries
```

This remains an architectural option and must be evaluated according to actual requirements.

---

# 19. Constraint AC-016 — Query Models Must Not Become Sources of Truth

**Category:** Data integrity  
**Severity:** Critical

Read models and projections are derived representations.

They must never become authoritative sources for domain invariants unless the domain model explicitly establishes them as such.

The authoritative state remains within the appropriate domain consistency boundary.

---

# 20. Constraint AC-017 — Value Objects Must Protect Semantic Validity

**Category:** Domain modeling  
**Severity:** High

Where the domain model identifies meaningful value objects, architecture must preserve their semantics.

Examples of conceptual value objects may include:

- identifiers;
- names;
- classifications;
- location values;
- metadata values;
- domain-specific quantities.

The architecture must avoid flattening all domain concepts into primitive values merely because the persistence technology makes that convenient.

---

# 21. Constraint AC-018 — Domain Services Must Remain Minimal

**Category:** Domain structure  
**Severity:** Medium

Domain services are permitted only when behavior does not naturally belong to an aggregate or value object.

They must not become generic service containers.

A domain service must have a clear business responsibility.

---

# 22. Constraint AC-019 — Specifications Must Remain Domain Expressions

**Category:** Domain structure  
**Severity:** Medium

Specifications should express reusable domain predicates.

They must not become:

- generic query builders;
- database filters disguised as business rules;
- application validation frameworks.

The architecture must preserve their domain meaning.

---

# 23. Constraint AC-020 — Application Validation and Domain Validation Must Remain Distinct

**Category:** Validation  
**Severity:** High

Two different validation concerns must remain distinguishable.

### Application validation

Examples:

- required command fields;
- malformed input;
- invalid request structure.

### Domain validation

Examples:

- invalid state transition;
- forbidden business operation;
- invariant violation;
- domain-specific semantic inconsistency.

```text id="h1t4n0"
Input Validation
      ↓
Application

Business Validation
      ↓
Domain
```

---

# 24. Constraint AC-021 — Authorization Must Not Leak Into Domain Rules

**Category:** Security / architecture  
**Severity:** High

Authorization and domain rules must remain conceptually distinct.

For example:

```text id="y4s7ph"
"User cannot access collection"
```

is an access-control concern.

Whereas:

```text id="u7j9qm"
"Archived collection cannot receive new items"
```

is a domain invariant.

The architecture must allow these concerns to coexist without conflating them.

---

# 25. Constraint AC-022 — Import Must Reuse Domain Rules

**Category:** Integration  
**Severity:** Critical

Imported data must not bypass the domain model.

The import pipeline should ultimately pass through the same domain validation mechanisms used by normal operations.

```text id="7y5b1s"
External Data
     ↓
Import Validation
     ↓
Domain Commands / Domain Behavior
     ↓
Valid Domain State
```

Import-specific optimizations must not create a second definition of domain validity.

---

# 26. Constraint AC-023 — Export Must Reflect Domain Meaning

**Category:** Integration  
**Severity:** High

Export mechanisms must derive their semantics from the domain model.

Exports must not become accidental representations of:

- database tables;
- internal ORM objects;
- infrastructure DTOs.

The export contract may differ from the internal domain representation, but its meaning must remain traceable to domain concepts.

---

# 27. Constraint AC-024 — Audit Must Preserve Historical Facts

**Category:** Audit  
**Severity:** High

Audit information must preserve historical facts rather than representing current mutable state.

Where audit history is required:

```text id="q8g1g3"
Domain Event / Domain Action
        ↓
Audit Record
        ↓
Immutable History
```

The architecture must prevent accidental rewriting of historical records.

---

# 28. Constraint AC-025 — Infrastructure Must Be Replaceable

**Category:** Architecture  
**Severity:** High

Infrastructure technologies should be replaceable without rewriting domain behavior.

Potentially replaceable infrastructure includes:

- database;
- message broker;
- search engine;
- object storage;
- email provider;
- external APIs;
- logging infrastructure.

This does not mean every infrastructure component must have excessive abstraction.

Abstraction must be justified by architectural boundaries.

---

# 29. Constraint AC-026 — External Integrations Must Be Isolated

**Category:** Integration  
**Severity:** Critical

External systems must not become direct dependencies of the domain.

The architecture should use ports/adapters or equivalent boundary mechanisms.

```text id="g6w1c8"
Domain
   ↑
Application Port
   ↑
Adapter
   ↑
External System
```

The external system must remain replaceable.

---

# 30. Constraint AC-027 — Framework Dependency Must Be Inward-Controlled

**Category:** Dependency management  
**Severity:** Critical

Framework dependencies should enter through infrastructure/application boundaries.

The architecture should prevent framework APIs from becoming part of the domain model.

This includes avoiding domain entities that require framework-specific:

- decorators;
- annotations;
- lifecycle hooks;
- base classes;
- persistence interfaces.

---

# 31. Constraint AC-028 — Transactions Must Follow Business Consistency

**Category:** Transaction management  
**Severity:** Critical

Transaction boundaries must be derived from domain consistency requirements.

The architecture must not use:

> "One HTTP request = one giant transaction"

as a universal rule.

Instead:

```text id="6l6c0x"
Business Consistency Requirement
          ↓
Aggregate Boundary
          ↓
Transaction Strategy
```

---

# 32. Constraint AC-029 — Concurrency Must Protect Invariants

**Category:** Reliability  
**Severity:** Critical

Concurrent operations must not permit invalid aggregate state.

The architecture must therefore eventually define appropriate mechanisms for:

- optimistic concurrency;
- versioning;
- locking where necessary;
- conflict detection;
- retry behavior.

The exact mechanism remains an architecture decision.

The invariant requirement does not.

---

# 33. Constraint AC-030 — Failures Must Preserve Domain Integrity

**Category:** Reliability  
**Severity:** Critical

Technical failures must not leave the domain in an invalid state.

The architecture must consider failure boundaries around:

- persistence;
- event publication;
- asynchronous processing;
- external integrations;
- imports;
- projections.

A domain transition must not be considered complete merely because an external side effect succeeded.

---

# 34. Constraint AC-031 — Domain Event Publication Must Be Reliable

**Category:** Integration  
**Severity:** High

Where domain events drive important downstream behavior, the architecture must guarantee that successful domain state changes are not silently disconnected from their corresponding events.

Potential strategies may include:

- transactional outbox;
- event log;
- reliable domain-event dispatch;
- equivalent transactional mechanisms.

The specific mechanism will be decided later.

The architectural requirement is:

```text id="6x7n9m"
Successful State Change
        ↕
Reliable Event Availability
```

---

# 35. Constraint AC-032 — External Event Consumers Must Not Control Aggregate State

**Category:** Integration  
**Severity:** High

External consumers may react to domain events but must not directly manipulate the source aggregate.

External systems should interact through explicit application/domain entry points.

---

# 36. Constraint AC-033 — API Contracts Must Not Define Domain Concepts

**Category:** Interface architecture  
**Severity:** High

External API DTOs are transport contracts.

They may resemble domain objects but must not automatically become domain objects.

The architecture should preserve:

```text id="4o8g7u"
API DTO
   ↓
Command / Query Model
   ↓
Domain Model
```

rather than:

```text id="h2k5m1"
HTTP JSON
   ↓
ORM Entity
```

---

# 37. Constraint AC-034 — UI State Must Not Become Domain State

**Category:** Presentation  
**Severity:** High

The user interface may have:

- forms;
- drafts;
- filters;
- temporary selections;
- pagination;
- sorting;
- client-side state.

These must not be confused with domain state.

---

# 38. Constraint AC-035 — Architectural Modules Must Reflect Responsibilities

**Category:** Modularity  
**Severity:** High

The future module structure should reflect domain and application responsibilities rather than technical layers alone.

Avoid creating a structure where all domain behavior becomes scattered across:

```text id="8h8p6m"
controllers/
services/
repositories/
models/
```

without clear business ownership.

The architecture should make business boundaries discoverable.

---

# 39. Constraint AC-036 — Dependencies Must Respect Layer Direction

The architecture must establish explicit dependency rules.

A candidate baseline is:

```text id="m5x6t2"
┌──────────────────────────────┐
│       Presentation           │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│        Application           │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│           Domain             │
└──────────────────────────────┘
               ↑
┌──────────────────────────────┐
│       Infrastructure         │
└──────────────────────────────┘
```

The exact packaging and dependency mechanism will be decided in subsequent architecture documents.

---

# 40. Constraint AC-037 — Technical Concerns Must Not Leak Into Domain Contracts

The following concepts must remain outside domain contracts unless a future domain decision explicitly establishes them as domain concepts:

- HTTP status codes;
- SQL;
- database IDs generated by a specific engine;
- JSON serialization;
- ORM tracking;
- message broker metadata;
- HTTP headers;
- framework request objects;
- cloud resource identifiers.

---

# 41. Constraint AC-038 — Domain Errors Must Be Semantic

Domain failures should communicate domain meaning.

Instead of exposing:

```text id="2s1q9c"
DatabaseConstraintViolation
```

the application should receive a meaningful domain-level failure such as:

```text id="7l9k3p"
ItemCannotBeAddedToArchivedCollection
```

The exact error taxonomy will be designed later.

The principle is already established.

---

# 42. Constraint AC-039 — Technical Failures Must Remain Distinguishable

Domain errors and infrastructure failures must not be collapsed into one generic error model.

```text id="n4n8e0"
Domain Failure
    ↓
Business decision rejected

Infrastructure Failure
    ↓
Technical execution failed
```

This distinction will be important for:

- retries;
- observability;
- API mapping;
- user feedback;
- operational recovery.

---

# 43. Constraint AC-040 — Observability Must Not Modify Domain Semantics

Logging, metrics and tracing must observe domain behavior.

They must not become implicit sources of domain state.

Observability should therefore be attached at architectural boundaries such as:

- application use cases;
- domain-event publication;
- repository operations;
- integration adapters.

---

# 44. Constraint AC-041 — Architecture Must Preserve Testability

The domain model must be testable without requiring:

- a database;
- HTTP server;
- message broker;
- UI;
- cloud infrastructure.

The architecture should therefore allow tests such as:

```text id="y7h0x3"
Aggregate
    ↓
Command / Domain Operation
    ↓
Invariant
    ↓
Expected State / Event
```

to execute in isolation.

---

# 45. Constraint AC-042 — Domain Tests Must Express Business Semantics

Tests around domain behavior should use business terminology.

Preferred:

```text id="t9k4c7"
Given an archived collection
When an item is added
Then the operation is rejected
```

rather than:

```text id="b3p6d2"
Given database row status = 4
When repository method X executes
Then SQL exception Y occurs
```

The latter is an implementation test, not a domain test.

---

# 46. Constraint AC-043 — Architecture Must Support Evolution

CollectionHub is expected to evolve.

Therefore, the architecture must avoid unnecessary coupling between:

- domain concepts;
- persistence schemas;
- external contracts;
- UI models;
- integration mechanisms.

The goal is not maximum abstraction.

The goal is **controlled change**.

---

# 47. Constraint AC-044 — Architectural Complexity Must Be Justified

The domain model does not justify introducing every available architectural pattern.

Patterns such as:

- CQRS;
- event sourcing;
- microservices;
- distributed transactions;
- message brokers;
- elaborate orchestration frameworks;

must only be introduced where domain or operational requirements justify them.

The architecture must not become more complex than the problem requires.

---

# 48. Constraint AC-045 — Start With a Modular Architecture

The initial architecture should favor strong internal boundaries before introducing physical distribution.

The preferred initial assumption is:

```text id="1p7c5j"
One deployable system
        ↓
Strong logical modules
        ↓
Explicit boundaries
        ↓
Independent evolution where justified
```

Physical distribution should be a consequence of actual requirements, not an architectural default.

---

# 49. Constraint AC-046 — Domain Boundaries Must Remain Visible in the Codebase

The eventual source tree should allow a developer to identify:

- where domain behavior lives;
- where application orchestration lives;
- where persistence lives;
- where integrations live;
- where API adapters live.

If the architecture makes these boundaries invisible, it is failing one of the primary goals of this phase.

---

# 50. Constraint AC-047 — Architecture Must Preserve Traceability

Every significant architectural component should be traceable to the domain model.

The desired chain is:

```text id="4v8f2j"
Architecture Component
        ↓
Architectural Responsibility
        ↓
Application / Domain Responsibility
        ↓
Use Case / Aggregate / Rule
        ↓
Business Concept
```

This relationship will be formalized later in:

`32_ARCHITECTURE_TRACEABILITY_MATRIX.md`

---

# 51. Constraint AC-048 — Architecture Decisions Must Record Their Rationale

When architecture chooses between alternatives, the decision must record:

- context;
- alternatives;
- decision;
- rationale;
- consequences;
- rejected alternatives.

This prevents architectural knowledge from existing only in individual developers' heads.

---

# 52. Constraint AC-049 — Domain Changes Must Trigger Architectural Review

If implementation discovers that:

- an aggregate boundary is wrong;
- an invariant cannot be enforced;
- a lifecycle is incomplete;
- a business rule is missing;
- a domain concept is incorrect;

the architecture must not silently compensate.

Instead:

```text id="f0n4g8"
Implementation Discovery
        ↓
Domain Model Review
        ↓
Domain Change
        ↓
Architecture Review
        ↓
Implementation Update
```

---

# 53. Constraint AC-050 — Architecture Must Not Prematurely Freeze Technical Choices

The current phase establishes constraints, not technology selections.

The following decisions are intentionally deferred:

- programming language;
- framework;
- database;
- ORM;
- message broker;
- search engine;
- cloud provider;
- deployment model.

They will be evaluated against the constraints established here.

---

# 54. Architectural Constraint Summary

The most important constraints can be summarized as follows:

```text id="5v4v6a"
                    DOMAIN
                      │
                      ▼
             ┌─────────────────┐
             │ Business Rules  │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │   Invariants    │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │   Aggregates    │
             └────────┬────────┘
                      │
             ┌────────┴────────┐
             ▼                 ▼
       Consistency         Behavior
       Boundaries             │
             │                ▼
             │          ┌─────────────┐
             │          │  Use Cases  │
             │          └──────┬──────┘
             │                 │
             └────────┬────────┘
                      ▼
              APPLICATION LAYER
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
     Persistence   Events     Integrations
          │           │           │
          └───────────┼───────────┘
                      ▼
                INFRASTRUCTURE
```

---

# 55. Architectural Constraint Priorities

Not all constraints have equal weight.

## Tier 1 — Non-negotiable

These protect the domain itself:

- aggregate boundaries;
- invariants;
- domain independence;
- mutation encapsulation;
- consistency requirements;
- domain behavior ownership;
- domain event semantics;
- cross-aggregate boundaries.

## Tier 2 — Strong architectural requirements

These protect maintainability:

- repository abstraction;
- integration isolation;
- application/domain separation;
- read-model separation;
- domain error semantics;
- testability;
- dependency direction.

## Tier 3 — Architectural preferences

These should be evaluated rather than blindly enforced:

- specific modularization strategies;
- CQRS;
- outbox;
- asynchronous processing;
- caching;
- physical service separation.

---

# 56. Architecture Decision Test

Every significant architectural proposal should pass these questions:

1. Does it preserve the domain invariants?
2. Does it preserve aggregate boundaries?
3. Does it preserve domain behavior ownership?
4. Does it maintain dependency direction?
5. Does it keep infrastructure replaceable where appropriate?
6. Does it preserve event semantics?
7. Does it introduce unnecessary coupling?
8. Does it introduce unnecessary complexity?
9. Can its responsibilities be traced to the domain model?
10. Does it make future domain evolution easier or harder?

If the answer to the first three is **no**, the architectural proposal must be rejected regardless of its technical advantages.

---

# 57. Architectural Baseline

The baseline architecture implied by the domain model is therefore:

```text id="6g0n0h"
┌─────────────────────────────────────────────┐
│                 INTERFACES                  │
│                                             │
│   API / UI / CLI / External Adapters        │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│                APPLICATION                  │
│                                             │
│   Use Cases                                 │
│   Commands                                  │
│   Workflows                                 │
│   Ports                                     │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│                   DOMAIN                   │
│                                             │
│   Aggregates                               │
│   Entities                                 │
│   Value Objects                            │
│   Domain Services                          │
│   Specifications                           │
│   Domain Events                             │
│   Invariants                                │
└──────────────────────┬──────────────────────┘
                       ▲
                       │
┌──────────────────────┴──────────────────────┐
│               INFRASTRUCTURE               │
│                                             │
│   Persistence                              │
│   Event Transport                          │
│   External Integrations                    │
│   Search                                   │
│   File Storage                              │
│   Observability                             │
└─────────────────────────────────────────────┘
```

This is an **architectural baseline**, not yet the final component structure.

---

# 58. Constraints That Require Further Architectural Decisions

The following areas require dedicated architecture documents:

| Area | Required Decision |
|---|---|
| Layering | Define exact architectural boundaries |
| Modularity | Define module ownership and dependencies |
| Persistence | Define aggregate persistence strategy |
| Queries | Define read/query architecture |
| Events | Define event lifecycle and publication |
| Integrations | Define ports/adapters |
| APIs | Define external interface boundaries |
| Transactions | Define transaction strategy |
| Concurrency | Define consistency/conflict strategy |
| Reliability | Define failure and retry mechanisms |
| Observability | Define architectural observability boundaries |
| Security | Define authorization architecture |
| Deployment | Define physical topology if required |

---

# 59. Constraints vs Decisions

This document intentionally does **not** decide:

```text id="j2o7f4"
PostgreSQL
MongoDB
MySQL
Redis
Kafka
RabbitMQ
REST
GraphQL
gRPC
Docker
Kubernetes
.NET
Java
TypeScript
Python
```

Those are implementation and architecture choices.

What this document establishes is the evaluation criterion:

> **Any selected technology must serve the domain constraints rather than force the domain to conform to the technology.**

---

# 60. Traceability to Domain Model

The primary source documents for these constraints are:

| Architectural Constraint Area | Domain Source |
|---|---|
| Concepts | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Terminology | `01_DOMAIN_GLOSSARY.md` |
| Capabilities | `03_DOMAIN_CAPABILITIES.md` |
| Relationships | `04_DOMAIN_RELATIONSHIPS.md` |
| Lifecycles | `05_DOMAIN_LIFECYCLES.md` |
| Policies | `06_DOMAIN_POLICIES.md` |
| Rules | `07_DOMAIN_RULES.md` |
| Invariants | `08_DOMAIN_INVARIANTS.md` |
| Aggregates | `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md` |
| Entities / Value Objects | `10_DOMAIN_ENTITIES_AND_VALUE_OBJECTS.md` |
| Use Cases | `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md` |
| Workflows | `12_APPLICATION_USE_CASES_AND_WORKFLOWS.md` |
| Commands | `13_DOMAIN_COMMANDS_AND_INPUT_MODELS.md` |
| State Transitions | `14_DOMAIN_STATE_TRANSITIONS.md` |
| Repositories | `15_DOMAIN_REPOSITORIES_AND_PERSISTENCE_CONTRACTS.md` |
| Events | `16_DOMAIN_EVENTS_AND_SIDE_EFFECTS.md` |
| Domain Services | `17_DOMAIN_SERVICES_AND_CROSS_AGGREGATE_RULES.md` |
| Specifications | `18_DOMAIN_SPECIFICATIONS_AND_REUSABLE_RULES.md` |
| Consistency | `19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md` |
| Decisions | `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` |
| Traceability | `21_DOMAIN_MODEL_TRACEABILITY_MATRIX.md` |
| Readiness | `22_DOMAIN_MODEL_FINAL_REVIEW_AND_IMPLEMENTATION_READINESS.md` |

---

# 61. Phase 2.3 Starting Point

This document establishes the first architectural artifact.

The next document must transform these constraints into actual architectural boundaries:

```text id="w1m5h8"
23_ARCHITECTURAL_CONSTRAINTS_FROM_DOMAIN
                  ↓
24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS
```

The next step must answer:

- What are the architectural layers?
- What responsibilities belong to each?
- What dependencies are allowed?
- What dependencies are forbidden?
- Where does the domain end?
- Where does the application layer begin?
- Where does infrastructure attach?
- Where do external interfaces enter?
- Where do domain events leave the domain?
- Which boundaries are logical?
- Which boundaries may later become physical?

---

# 62. Final Principle

The most important architectural constraint established by this document is:

> **The architecture of CollectionHub must preserve the semantic integrity of the domain model, even when doing so is less convenient for a particular technical implementation.**

The architectural design therefore starts from:

```text id="p8q6k1"
Business Meaning
      ↓
Domain Constraints
      ↓
Architectural Boundaries
      ↓
Technical Decisions
      ↓
Implementation
```

and never the reverse.

**Phase 2.3 is now formally initiated.**

**Next document: `24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md`.**