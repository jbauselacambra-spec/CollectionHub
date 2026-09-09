# Architectural Boundaries and Layers

**CollectionHub — Phase 2.3: Domain-to-Architecture Translation**

**Status:** Draft / Architectural Baseline  
**Document:** `24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md`  
**Location:** `CollectionHub/docs/02_ARCHITECTURE/`  
**Purpose:** Define the architectural boundaries, layers, dependency rules, ownership responsibilities, and communication paths required to preserve the CollectionHub domain model.

---

# 1. Purpose

This document translates the architectural constraints established in:

`23_ARCHITECTURAL_CONSTRAINTS_FROM_DOMAIN.md`

into an explicit architectural boundary model.

The objective is to define:

- architectural layers;
- module boundaries;
- dependency directions;
- allowed interactions;
- forbidden interactions;
- entry points;
- exit points;
- domain isolation;
- application orchestration;
- infrastructure responsibilities;
- interface responsibilities;
- synchronous and asynchronous communication boundaries.

The fundamental architectural principle remains:

> **The architecture must protect the domain model rather than make the domain conform to the architecture.**

---

# 2. Architectural Model

The initial CollectionHub architecture will use a **layered architecture with explicit dependency inversion and modular boundaries**.

At the conceptual level:

```text
┌──────────────────────────────────────────────────────┐
│                    INTERFACES                        │
│                                                      │
│ API / UI / CLI / Import / External Adapters          │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                   APPLICATION                        │
│                                                      │
│ Use Cases / Commands / Workflows / Ports             │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                     DOMAIN                           │
│                                                      │
│ Aggregates / Entities / Value Objects                │
│ Domain Services / Specifications / Events            │
└──────────────────────────────────────────────────────┘
                           ▲
                           │
┌──────────────────────────┴───────────────────────────┐
│                  INFRASTRUCTURE                      │
│                                                      │
│ Persistence / Event Transport / External Services    │
│ Search / Files / Observability / Technical Adapters  │
└──────────────────────────────────────────────────────┘
```

This diagram represents **logical dependencies**, not necessarily physical deployment units.

---

# 3. Architectural Layers

CollectionHub will initially define four primary architectural areas:

1. **Interfaces**
2. **Application**
3. **Domain**
4. **Infrastructure**

These layers have different responsibilities.

---

# 4. Layer 1 — Interfaces

## 4.1 Purpose

The Interfaces layer represents the boundary between CollectionHub and actors or external systems.

It translates external representations into application-level requests.

Examples include:

- HTTP APIs;
- command-line interfaces;
- UI adapters;
- import adapters;
- external event consumers;
- scheduled jobs;
- integration endpoints.

---

## 4.2 Responsibilities

Interfaces are responsible for:

- protocol handling;
- request parsing;
- DTO mapping;
- authentication context extraction;
- transport-level validation;
- response mapping;
- protocol-specific error handling.

They are **not** responsible for:

- domain rules;
- aggregate mutation;
- persistence;
- business workflows;
- domain event creation.

---

## 4.3 Interface Flow

```text id="7z0k1n"
External Actor
      ↓
Interface Adapter
      ↓
Application Command / Query
```

The interface should remain thin.

---

# 5. Layer 2 — Application

## 5.1 Purpose

The Application layer orchestrates use cases.

It is the entry point for business operations from interfaces and technical adapters.

---

## 5.2 Responsibilities

The Application layer owns:

- use-case orchestration;
- command handling;
- query orchestration;
- transaction coordination;
- repository access through ports;
- domain-event collection;
- event publication requests;
- authorization coordination;
- application-level policies;
- mapping between external/application models and domain operations.

---

## 5.3 Application Does Not Own

The Application layer must not own:

- aggregate invariants;
- entity behavior;
- value-object validity;
- domain state transitions;
- domain policies that belong to the domain;
- persistence implementation;
- transport protocols.

---

# 6. Layer 3 — Domain

## 6.1 Purpose

The Domain layer is the business core of CollectionHub.

It represents what the system **means**.

---

## 6.2 Responsibilities

The Domain layer contains:

- aggregates;
- aggregate roots;
- entities;
- value objects;
- domain services;
- specifications;
- domain policies;
- domain invariants;
- domain errors;
- domain events.

---

## 6.3 Domain Independence

The domain must not depend on:

- HTTP;
- databases;
- ORM frameworks;
- message brokers;
- cloud providers;
- external services;
- UI frameworks;
- serialization formats.

---

# 7. Layer 4 — Infrastructure

## 7.1 Purpose

Infrastructure implements technical mechanisms required by the application and domain.

It answers:

> How is this architectural capability technically realized?

---

## 7.2 Responsibilities

Infrastructure may contain:

- database implementations;
- repository implementations;
- transaction mechanisms;
- event dispatchers;
- message brokers;
- external API clients;
- file storage;
- search infrastructure;
- caching;
- observability;
- scheduling;
- background processing.

---

## 7.3 Infrastructure Must Not Define Domain Behavior

Infrastructure can enforce technical constraints.

It must not invent business rules.

---

# 8. Dependency Direction

The fundamental dependency direction is:

```text id="8l2b5m"
Interfaces
     ↓
Application
     ↓
Domain
```

Infrastructure depends on contracts defined inward:

```text id="c7s4p9"
Infrastructure
     ↓
Application / Domain Contracts
```

The conceptual dependency graph therefore becomes:

```text id="5p9j1q"
                  ┌───────────────┐
                  │  Interfaces   │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │  Application  │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │    Domain     │
                  └───────────────┘
                          ▲
                          │
                  ┌───────┴───────┐
                  │ Infrastructure│
                  └───────────────┘
```

The apparent upward dependency from Infrastructure is deliberate.

Infrastructure implements ports defined by inner layers.

---

# 9. Dependency Inversion

When an inner layer requires an external capability, the contract must be defined toward the inside.

For example:

```text id="x5j8m2"
Application
    │
    │ requires CollectionRepository
    ▼
Application Port
    ▲
    │ implements
    │
Infrastructure Adapter
```

This prevents the domain/application layer from depending directly on a database technology.

---

# 10. Domain Boundary

The Domain layer is the most protected architectural boundary.

It should contain only concepts necessary to express business meaning.

A domain object should never need to know:

```text id="0j5k3n"
HTTP
JSON
SQL
ORM
Database Connection
Message Broker
Filesystem
Cloud Provider
```

---

# 11. Application Boundary

The Application boundary separates:

```text id="x6r1m0"
"What the system does"
```

from:

```text id="k8t2w4"
"How the business rules work"
```

For example:

```text id="7c3g8h"
CreateCollection
       ↓
Application Use Case
       ↓
Collection.create(...)
       ↓
Domain
```

The application decides **which operation to orchestrate**.

The domain decides **whether that operation is valid**.

---

# 12. Infrastructure Boundary

Infrastructure is the outer technical boundary.

It contains implementations of capabilities such as:

```text id="3n5w2f"
Persistence
Messaging
External APIs
Search
Files
Scheduling
Observability
```

Infrastructure may depend on application/domain contracts.

The reverse dependency is forbidden.

---

# 13. Interface Boundary

Interfaces represent translation boundaries.

For example:

```text id="0m8k2p"
HTTP Request
     ↓
HTTP Adapter
     ↓
Application Command
```

and:

```text id="v5c1n9"
Application Result
     ↓
HTTP DTO
     ↓
HTTP Response
```

The domain must never receive an HTTP request object.

---

# 14. Synchronous Communication

Synchronous application execution follows:

```text id="h5s2r7"
Interface
   ↓
Application
   ↓
Domain
   ↓
Repository Port
   ↓
Infrastructure
```

The infrastructure result flows back through the same boundary.

---

# 15. Asynchronous Communication

Asynchronous processing follows:

```text id="d2m7q1"
Domain State Change
       ↓
Domain Event
       ↓
Application Event Dispatcher
       ↓
Infrastructure Transport
       ↓
Consumer
```

The transport is not part of the domain.

---

# 16. Domain Event Boundary

Domain events originate inside the domain.

The event publication mechanism exists outside the domain.

```text id="q9m3v5"
DOMAIN
  │
  │ Domain Event
  ▼
APPLICATION
  │
  │ Publication
  ▼
INFRASTRUCTURE
  │
  │ Transport
  ▼
External Systems
```

---

# 17. Repository Boundary

Repositories are ports.

Their contracts should be defined inward.

A conceptual structure is:

```text id="k7v2p3"
Application / Domain
        │
        │ Repository Port
        ▼
Infrastructure Adapter
        │
        ▼
Database
```

Whether the repository contract belongs in the Domain or Application layer will be decided according to the exact responsibility of each repository.

The important constraint is that the implementation remains outside.

---

# 18. Query Boundary

Queries may follow a different path from commands.

Command:

```text id="n2j7q6"
Interface
   ↓
Application
   ↓
Domain
   ↓
Repository
```

Query:

```text id="c4x8v1"
Interface
   ↓
Application
   ↓
Query Port
   ↓
Read Model / Query Infrastructure
```

This avoids forcing read-oriented operations through aggregate loading when unnecessary.

---

# 19. Command Boundary

Commands enter through the Application layer.

```text id="w8p1r4"
Interface
    ↓
Command
    ↓
Application Use Case
    ↓
Aggregate
```

Commands must not contain domain behavior.

---

# 20. Query Boundary

Queries are read intentions.

They should not mutate domain state.

```text id="t3k7m2"
Query
  ↓
Read Model
  ↓
Result
```

A query must not bypass domain invariants because it does not change state.

---

# 21. Transaction Boundary

Transactions must follow consistency requirements.

The preferred conceptual rule is:

```text id="e7s1k4"
One domain consistency operation
              ↓
       Appropriate transaction
```

Not:

```text id="r4m8q2"
One request
      ↓
Everything in one transaction
```

---

# 22. Error Boundary

Errors must be translated at architectural boundaries.

Domain:

```text id="c8k2x4"
ItemCannotBeDisposed
```

Application:

```text id="f3q7m1"
Use case failure
```

Interface:

```text id="z6n1p8"
HTTP 409 / 422 / etc.
```

Infrastructure:

```text id="v4t9r2"
DatabaseUnavailable
```

These are different semantic layers.

---

# 23. Authorization Boundary

Authorization should occur at the application/interface boundary.

The application determines whether the current actor is allowed to invoke a use case.

The domain determines whether the operation is valid according to business rules.

```text id="b6p2w9"
Actor
 ↓
Authorization
 ↓
Use Case
 ↓
Domain Rule
```

The two concerns must not be conflated.

---

# 24. Validation Boundary

Validation is divided into three categories.

### Transport validation

```text id="q1w5z8"
Malformed HTTP request
```

Owned by Interfaces.

### Application validation

```text id="s3k7n4"
Required command data missing
```

Owned by Application.

### Domain validation

```text id="m8r2p6"
Operation violates business invariant
```

Owned by Domain.

---

# 25. Mapping Boundary

External DTOs must not automatically become domain objects.

The architecture uses explicit translation:

```text id="y6k4s2"
External DTO
    ↓
Command
    ↓
Domain Operation
```

and:

```text id="p3m7x1"
Domain Result
    ↓
Application Result
    ↓
External DTO
```

---

# 26. Import Boundary

Imports are external data ingestion processes.

The architecture should isolate them:

```text id="k4n8q2"
Import Source
      ↓
Import Adapter
      ↓
Import Application Workflow
      ↓
Domain Validation
      ↓
Domain State
```

Import data must never directly populate domain persistence structures.

---

# 27. Export Boundary

Exports are read-oriented transformations.

```text id="w2c7m5"
Domain / Read Model
       ↓
Export Application Service
       ↓
Export Adapter
       ↓
External Format
```

Export formats remain external concerns.

---

# 28. Search Boundary

Search should remain a query concern.

A possible architecture is:

```text id="n6p3x9"
Domain Events
      ↓
Search Projection
      ↓
Search Infrastructure
      ↓
Search Query
```

The search engine must not become the source of domain truth.

---

# 29. Audit Boundary

Audit recording is downstream from domain behavior.

```text id="s7q2m4"
Domain Action
     ↓
Domain Event
     ↓
Audit Adapter
     ↓
Audit Store
```

Audit storage remains outside the core domain.

---

# 30. Scheduling Boundary

Scheduled operations such as:

- imports;
- exports;
- maintenance;
- projection rebuilding;

must enter through application use cases.

```text id="u5m8k1"
Scheduler
    ↓
Application Use Case
    ↓
Domain / Query Model
```

A scheduler must not manipulate persistence directly.

---

# 31. Background Processing Boundary

Background workers are infrastructure/application adapters.

They may invoke application use cases.

They must not contain domain rules.

```text id="j4p9v6"
Background Worker
       ↓
Application
       ↓
Domain
```

---

# 32. External Integration Boundary

External systems must be isolated behind adapters.

```text id="c9w2r7"
Domain / Application
       │
       ▼
Integration Port
       ▲
       │
Integration Adapter
       │
       ▼
External System
```

This allows external technologies to evolve independently.

---

# 33. Layer Dependency Matrix

| From / To | Interfaces | Application | Domain | Infrastructure |
|---|---:|---:|---:|---:|
| Interfaces | ✓ | ✓ | — | — |
| Application | — | ✓ | ✓ | Via ports |
| Domain | — | — | ✓ | — |
| Infrastructure | — | Via ports | Via ports | ✓ |

### Interpretation

- Interfaces may depend on Application.
- Application may depend on Domain.
- Domain must not depend on outer layers.
- Infrastructure implements contracts from inner layers.
- Infrastructure must not become a dependency of Domain.

---

# 34. Forbidden Dependencies

The following dependencies are forbidden.

### Domain → Infrastructure

```text id="q8w4m1"
Domain → ORM
Domain → Database
Domain → HTTP
Domain → Message Broker
```

### Domain → Interface

```text id="p6n3v7"
Domain → Controller
Domain → HTTP Request
Domain → UI
```

### Domain → Framework

```text id="h5k9c2"
Domain → Framework Base Class
Domain → Framework Annotation
```

### Application → Concrete Infrastructure

```text id="m3x8q1"
Application → PostgreSQLClient
Application → KafkaProducer
```

Instead:

```text id="z7r2w5"
Application → Port
Infrastructure → Port implementation
```

---

# 35. Allowed Dependencies

Examples of allowed relationships:

```text id="f8k4m2"
API Adapter
    → Application Command

Application
    → Domain Aggregate

Application
    → Repository Port

Infrastructure
    → Repository Port

Infrastructure
    → External API

Infrastructure
    → Database

Infrastructure
    → Message Broker
```

---

# 36. Architectural Boundary Tests

Every component should be evaluated with:

### Question 1

Does this component contain business rules?

If yes:

> It probably belongs in Domain.

### Question 2

Does it orchestrate a business operation?

If yes:

> It probably belongs in Application.

### Question 3

Does it translate a protocol?

If yes:

> It probably belongs in Interfaces.

### Question 4

Does it implement a technical mechanism?

If yes:

> It probably belongs in Infrastructure.

---

# 37. Module Boundary Principle

Layers alone are insufficient.

Within each layer, responsibilities should remain modular.

For example:

```text id="y4m8p2"
Domain
├── Collection
├── Item
├── Location
├── Classification
├── Acquisition
├── Audit
└── Shared Domain Concepts
```

The exact module decomposition will be defined in:

`26_APPLICATION_AND_DOMAIN_MODULE_STRUCTURE.md`

This document establishes only the boundary principle.

---

# 38. Boundary Ownership

| Concern | Primary Owner |
|---|---|
| Business concept | Domain |
| Business invariant | Domain |
| Aggregate behavior | Domain |
| Use-case orchestration | Application |
| Command handling | Application |
| Authorization coordination | Application |
| Protocol translation | Interfaces |
| Persistence implementation | Infrastructure |
| Event transport | Infrastructure |
| External API integration | Infrastructure |
| Search implementation | Infrastructure |
| File storage | Infrastructure |
| Observability infrastructure | Infrastructure |

---

# 39. Boundary Stability

The following boundaries are considered stable architectural concepts:

```text id="s6k3v8"
Domain
Application
Infrastructure
Interfaces
```

Their internal packaging may change.

Their responsibilities should not.

---

# 40. Logical vs Physical Boundaries

The architecture deliberately distinguishes logical boundaries from physical deployment.

Initially:

```text id="x7m2q4"
Logical Modules
      ↓
Potentially One Application
      ↓
Potentially One Deployment
```

Later, if justified:

```text id="p8v3k1"
Logical Module
      ↓
Independent Process
      ↓
Independent Deployment
```

Physical separation must never precede a demonstrated need.

---

# 41. Modular Monolith Baseline

The initial architectural hypothesis is a **modular monolith** unless future requirements demonstrate that another topology is necessary.

This provides:

- strong logical boundaries;
- simple deployment;
- simple transactions;
- low operational complexity;
- clear domain ownership;
- future extraction opportunities.

The architectural decision will be formally recorded later.

---

# 42. Communication Rules

Communication between boundaries should use explicit contracts.

Preferred:

```text id="d4k7p2"
Command
Query
Domain Event
Application Result
Port
```

Avoid:

```text id="n9m3x6"
Shared Mutable Object Graph
```

as a general communication mechanism.

---

# 43. Shared Kernel Considerations

A small shared kernel may exist for genuinely cross-cutting domain primitives.

Examples might include:

- domain identifiers;
- common result abstractions;
- domain error abstractions;
- common event infrastructure contracts.

However, the shared kernel must remain deliberately small.

A large shared kernel becomes a hidden coupling mechanism.

---

# 44. Dependency Rules for Domain Modules

Domain modules should communicate through:

- explicit domain interfaces;
- value objects;
- identifiers;
- domain services;
- domain events.

They should avoid direct dependencies on internal implementation details of unrelated aggregates.

---

# 45. Dependency Rules for Application Modules

Application modules may:

- invoke domain modules;
- coordinate repositories;
- invoke domain services;
- publish events;
- execute queries.

They should not:

- modify domain state directly;
- access infrastructure implementation details;
- bypass aggregate behavior.

---

# 46. Dependency Rules for Infrastructure Modules

Infrastructure modules may:

- implement application ports;
- implement repository contracts;
- implement integration contracts;
- publish events;
- access databases;
- access external systems.

They should not:

- define business invariants;
- modify aggregate state directly;
- become the canonical source of business rules.

---

# 47. Dependency Rules for Interface Modules

Interface modules may:

- parse external requests;
- authenticate;
- authorize;
- construct commands;
- invoke use cases;
- map results;
- map errors.

They should not:

- implement business rules;
- directly access repositories;
- directly access databases;
- construct aggregate state bypassing application workflows.

---

# 48. Request Lifecycle

A typical synchronous command should follow:

```text id="w8x3m6"
Client
  ↓
Interface Adapter
  ↓
Command
  ↓
Application Use Case
  ↓
Authorization
  ↓
Repository Port
  ↓
Aggregate
  ↓
Domain Behavior
  ↓
Invariant Validation
  ↓
State Change
  ↓
Domain Event
  ↓
Repository Persistence
  ↓
Event Publication
  ↓
Application Result
  ↓
Interface Response
```

This is the canonical application flow.

---

# 49. Query Lifecycle

A typical query should follow:

```text id="j3n8v5"
Client
  ↓
Interface
  ↓
Query
  ↓
Application Query Handler
  ↓
Query Port
  ↓
Read Model / Query Infrastructure
  ↓
Result
  ↓
Interface DTO
```

No aggregate mutation should occur.

---

# 50. Asynchronous Event Lifecycle

A typical asynchronous flow should follow:

```text id="p5k2w7"
Aggregate
   ↓
Domain Event
   ↓
Reliable Event Boundary
   ↓
Event Dispatcher
   ↓
Infrastructure Transport
   ↓
Consumer
   ↓
Application Handler
   ↓
Side Effect
```

The event transport remains replaceable.

---

# 51. Failure Boundaries

Each architectural boundary must define failure semantics.

```text id="n4q8s1"
Interface failure
      ↓
Request-level failure

Application failure
      ↓
Use-case failure

Domain failure
      ↓
Business rejection

Infrastructure failure
      ↓
Technical failure
```

These failures must remain distinguishable.

---

# 52. Transactional Boundary Rule

The architecture should prefer:

```text id="k7m3p8"
One aggregate consistency boundary
       ↓
One atomic state transition
```

Cross-aggregate operations require explicit analysis.

Possible strategies include:

- orchestration;
- domain service;
- event-driven coordination;
- eventual consistency;
- compensation.

No strategy should be selected automatically.

---

# 53. Boundary Enforcement Mechanisms

Architectural boundaries should eventually be enforced through mechanisms such as:

- module/package visibility;
- dependency rules;
- architecture tests;
- code review;
- static analysis;
- automated build validation;
- explicit interfaces.

Documentation alone is insufficient.

---

# 54. Architecture Tests

The future implementation should contain architecture-level tests that verify rules such as:

```text id="s9x2c4"
Domain must not depend on Infrastructure.
Domain must not depend on Interfaces.
Application must not depend on concrete Infrastructure.
Interface must not access persistence directly.
Infrastructure must implement inward contracts.
```

These tests should become part of the build pipeline.

---

# 55. Boundary Violation Example

Incorrect:

```text id="m4v7x2"
Controller
   ↓
Repository
   ↓
Entity
   ↓
Property mutation
```

Why it is incorrect:

- bypasses application use case;
- bypasses domain behavior;
- risks violating invariants;
- couples interface to persistence.

Correct:

```text id="r8k2n5"
Controller
   ↓
Application Use Case
   ↓
Aggregate
   ↓
Domain Behavior
   ↓
Repository
```

---

# 56. Architecture Boundary Heuristic

When uncertain where something belongs, ask:

> **Which decision does this component own?**

If the answer is:

- business validity → Domain;
- orchestration → Application;
- protocol translation → Interface;
- technical execution → Infrastructure.

This heuristic should be used throughout implementation.

---

# 57. Relationship With Previous Architecture Document

`23_ARCHITECTURAL_CONSTRAINTS_FROM_DOMAIN.md` defines **what architecture must protect**.

This document defines **where those protections live**.

The relationship is:

```text id="u4m8q1"
23
Architectural Constraints
        ↓
24
Architectural Boundaries & Layers
```

The next document will define the concrete components inside these boundaries.

---

# 58. Next Architectural Step

The next document is:

`25_ARCHITECTURAL_COMPONENTS_AND_RESPONSIBILITIES.md`

It will define:

- concrete architectural components;
- responsibilities;
- ownership;
- dependencies;
- communication contracts;
- ports and adapters;
- application services;
- domain modules;
- infrastructure adapters;
- interface adapters.

The sequence becomes:

```text id="x8k5m2"
23 Constraints
       ↓
24 Boundaries & Layers
       ↓
25 Components & Responsibilities
       ↓
26 Module Structure
```

---

# 59. Final Architectural Boundary Principle

The architecture must make the following distinction visible:

```text id="q2v7n4"
DOMAIN
"What is true?"

APPLICATION
"What operation are we performing?"

INTERFACES
"How did the request arrive?"

INFRASTRUCTURE
"How is the technical capability provided?"
```

These questions are related but must not be collapsed into one layer.

The architecture is successful when a developer can answer each question without accidentally crossing the wrong boundary.

---

# 60. Final Decision

The CollectionHub architectural baseline therefore adopts:

```text id="c5m9w3"
                 INTERFACES
                      │
                      ▼
                 APPLICATION
                      │
                      ▼
                   DOMAIN
                      ▲
                      │
                INFRASTRUCTURE
```

with:

- explicit dependency inversion;
- protected domain boundaries;
- aggregate-driven consistency;
- application-level orchestration;
- infrastructure adapters;
- interface translation;
- explicit synchronous/asynchronous boundaries;
- modular internal structure;
- logical boundaries preceding physical distribution.

**Document status: ACCEPTED AS ARCHITECTURAL BASELINE.**

**Next: `25_ARCHITECTURAL_COMPONENTS_AND_RESPONSIBILITIES.md`.**