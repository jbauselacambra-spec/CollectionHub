CollectionHub — Phase 2.3: Domain-to-Architecture Translation

Status: Draft / Architectural Baseline
Location: CollectionHub/docs/02_ARCHITECTURE/
Previous: 24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md
Next: 26_APPLICATION_AND_DOMAIN_MODULE_STRUCTURE.md

1. Purpose

This document translates the architectural boundaries and layers defined in 24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md into a concrete set of architectural components and responsibilities.

The objective is to answer:

What components will exist inside the architectural boundaries, what does each component own, and how are they allowed to collaborate?

This document intentionally remains technology-independent.

It defines responsibility and dependency structure, not programming-language packages, frameworks, database technologies, or deployment units.

2. Architectural Component Model

The baseline component model is:

┌──────────────────────────────────────────────────────────────┐
│                         INTERFACES                           │
│                                                              │
│  API Adapters   UI Adapters   Import Adapters   CLI/Jobs     │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                        APPLICATION                           │
│                                                              │
│  Use Cases   Commands   Queries   Policies   Ports            │
│  Workflow Orchestration   Authorization Coordination         │
└──────────────────────────────┬───────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                           DOMAIN                             │
│                                                              │
│  Aggregates   Entities   Value Objects   Domain Services      │
│  Specifications   Policies   Domain Events   Domain Errors    │
└──────────────────────────────┬───────────────────────────────┘
                               ▲
                               │
┌──────────────────────────────────────────────────────────────┐
│                       INFRASTRUCTURE                         │
│                                                              │
│  Persistence   Eventing   Integrations   Search   Files       │
│  Observability   Scheduling   Technical Services             │
└──────────────────────────────────────────────────────────────┘

The important point is that these components are logical responsibilities.

They do not necessarily correspond one-to-one with source-code projects or deployment processes.

3. Component Classification

Every architectural component should belong to one of four categories:

Category	Main Question
Interface	How does something enter or leave the system?
Application	What operation is being orchestrated?
Domain	What business meaning and rules apply?
Infrastructure	How is a technical capability provided?

This classification is the primary mechanism for preventing responsibility leakage.

4. Interface Components

The Interfaces boundary contains components that translate external interactions into application operations.

Baseline components:

Interfaces
├── API
├── UI
├── CLI
├── Import
├── External Event Consumers
└── Scheduled Entry Points

Not every component necessarily needs to exist in the first implementation.

5. API Adapter
Responsibility

The API Adapter exposes application functionality through an external API.

It is responsible for:

receiving requests;
parsing transport data;
authentication context extraction;
request validation;
DTO mapping;
invoking application use cases;
mapping application results;
mapping errors to transport responses.
Must Not

The API adapter must not:

implement business rules;
access the database directly;
mutate aggregates directly;
publish domain events directly;
contain workflow logic.
6. UI Adapter
Responsibility

The UI Adapter translates user interactions into application commands and queries.

It owns:

presentation state;
user interaction;
presentation validation;
formatting;
navigation;
application invocation.

It does not own business rules.

7. CLI Adapter

A command-line interface, if required, is another application entry point.

CLI
 ↓
Command
 ↓
Application Use Case

It should use exactly the same application contracts as other interfaces.

This prevents CLI-specific business behavior.

8. Import Adapter

The Import Adapter receives external collection data.

Responsibilities:

parsing external formats;
validating external structure;
translating external records;
invoking import application workflows;
reporting import-specific technical failures.

It must not:

directly insert database records;
bypass aggregate rules;
define alternative business rules.
9. External Event Consumer

This component receives events from external systems.

Its responsibility is:

External Event
      ↓
Transport Translation
      ↓
Application Command / Event Handler

External event formats must not leak into the domain.

10. Scheduled Entry Point

Scheduled operations such as:

imports;
maintenance;
projection updates;
synchronization;

must enter through application use cases.

Scheduler
   ↓
Application Use Case

The scheduler itself contains no business logic.

11. Application Components

The Application layer is composed conceptually of:

Application
├── Use Cases
├── Commands
├── Queries
├── Workflows
├── Application Policies
├── Authorization Coordination
├── Ports
├── Application Results
└── Event Handling
12. Application Use Case

A Use Case represents an application-level operation meaningful to an actor or system.

Examples conceptually include:

CreateCollection
AddItem
MoveItem
RemoveItem
ArchiveCollection
SearchItems
ImportCollection
ExportCollection

The exact catalogue remains defined by the domain/application use-case documentation.

13. Use Case Responsibilities

A use case may:

receive an application command;
establish execution context;
validate application-level requirements;
authorize the operation;
load required aggregates;
invoke domain behavior;
coordinate multiple domain objects where required;
persist changes;
collect domain events;
request event publication;
return an application result.

It must not become the owner of domain invariants.

14. Command Component

Commands represent requested state-changing operations.

Command
   ↓
Use Case

A command should contain the information required to perform an operation.

It should not contain:

database behavior;
business algorithms;
infrastructure dependencies.
15. Query Component

Queries represent read intentions.

Query
   ↓
Query Handler
   ↓
Query Port / Read Model

Queries must not mutate domain state.

16. Workflow Component

Some application operations may require multiple steps.

A Workflow coordinates these steps.

For example:

Import Workflow
    ↓
Parse
    ↓
Validate
    ↓
Transform
    ↓
Execute Domain Operations
    ↓
Persist
    ↓
Report Result

The workflow owns orchestration.

The domain still owns business decisions.

17. Application Policy

An Application Policy represents an application concern rather than a domain invariant.

Examples may include:

authorization requirements;
rate limits;
execution policies;
retry decisions;
orchestration rules.

Application policies must not duplicate domain policies.

18. Authorization Coordinator

Authorization determines whether an actor may invoke an application operation.

Conceptually:

Actor
  ↓
Authorization
  ↓
Use Case

Authorization may use domain information, but the authorization mechanism itself remains an application/security concern unless the domain explicitly models permissions as part of its business semantics.

19. Application Ports

Ports define capabilities required by the application.

Examples:

CollectionRepository
ItemRepository
EventPublisher
Clock
IdentityProvider
ExternalCatalogGateway
FileStorage
SearchReader

The final port inventory will be determined from concrete use cases.

20. Port Ownership

A critical rule:

The component requiring a capability owns its abstraction.

Therefore:

Application
    ↓
Application Port
    ↑
Infrastructure Adapter

rather than:

Infrastructure
    ↓
Interface exposed to Application

This keeps dependency inversion explicit.

21. Application Result

Application results represent the outcome of a use case.

They are not necessarily domain objects.

For example:

Use Case
   ↓
Application Result
   ↓
Interface DTO

This prevents transport concerns from leaking into the application or domain.

22. Domain Components

The Domain layer contains the business model.

Baseline components:

Domain
├── Aggregates
├── Entities
├── Value Objects
├── Domain Services
├── Specifications
├── Domain Policies
├── Domain Events
├── Domain Errors
└── Domain Factories

Not every domain module needs every component.

23. Aggregate Component

Aggregates are the primary consistency components.

An aggregate:

protects invariants;
controls state transitions;
owns domain behavior;
emits domain events;
exposes controlled operations.

External components must interact through aggregate roots.

24. Aggregate Root

The Aggregate Root is the only externally accessible entry point for mutation inside the aggregate.

Conceptually:

Application
    ↓
Aggregate Root
    ↓
Internal Entity / Value Object

Not:

Application
    ↓
Internal Entity
25. Entity Component

Entities model domain objects with identity and lifecycle.

Responsibilities include:

identity;
state;
domain behavior;
lifecycle transitions.

Entities must not become persistence models simply because an ORM uses the same terminology.

26. Value Object Component

Value Objects encapsulate meaningful domain values.

Responsibilities:

semantic validity;
equality;
normalization where appropriate;
domain-specific operations.

A Value Object should prevent invalid states where practical.

27. Domain Service Component

A Domain Service contains domain behavior that cannot naturally belong to one aggregate or value object.

It must have:

explicit business responsibility;
meaningful domain terminology;
minimal dependencies;
no infrastructure responsibility.

It must not become a generic DomainService dumping ground.

28. Specification Component

Specifications represent reusable domain predicates.

They can support:

eligibility;
classification;
policy evaluation;
reusable conditions.

They must remain expressed in domain language.

29. Domain Policy Component

A Domain Policy represents a business decision or rule that may span concepts.

Policies may be implemented through:

aggregate behavior;
domain services;
specifications;
dedicated policy objects.

The exact mechanism depends on the rule's ownership.

30. Domain Event Component

Domain Events represent facts that occurred in the domain.

Examples conceptually:

CollectionCreated
ItemAdded
ItemMoved
CollectionArchived

The actual event inventory is defined by the domain event documentation.

Domain events must be:

meaningful;
immutable;
transport-independent;
domain-oriented.
31. Domain Error Component

Domain errors express business rejection.

Examples conceptually:

InvalidStateTransition
CollectionCannotAcceptItem
ItemAlreadyAssigned
OperationNotAllowedByDomainRule

They should describe why the business operation is invalid.

32. Domain Factory Component

Factories may be used when creation involves meaningful domain decisions that do not naturally belong to a constructor.

A factory must not become an infrastructure factory.

33. Infrastructure Components

Infrastructure provides technical implementations.

Baseline structure:

Infrastructure
├── Persistence
├── Transactions
├── Eventing
├── Integrations
├── Search
├── File Storage
├── Scheduling
├── Background Processing
└── Observability
34. Persistence Component

Persistence implements repository and data-storage contracts.

Responsibilities:

database connectivity;
persistence mapping;
repository implementation;
query execution;
persistence transactions;
concurrency mechanisms.

It must not define domain behavior.

35. Persistence Mapping

Persistence mapping translates between:

Domain Model
      ↕
Persistence Model

The persistence representation may differ significantly from the domain representation.

This is intentional.

36. Transaction Component

The Transaction component implements the technical transaction strategy required by application operations.

Responsibilities may include:

transaction creation;
commit;
rollback;
isolation configuration;
concurrency handling.

The business meaning of the transaction remains defined by the application/domain boundaries.

37. Eventing Component

The Eventing component provides technical event infrastructure.

Responsibilities:

event serialization;
event persistence;
event dispatch;
broker integration;
delivery guarantees;
retry mechanisms;
dead-letter handling where required.

It must not define domain event semantics.

38. Integration Component

Integrations communicate with external systems.

Application Port
      ↑
Integration Adapter
      ↓
External System

Examples may include:

external catalogues;
identity systems;
notification services;
external metadata sources.
39. Search Component

Search infrastructure implements search capabilities.

Possible responsibilities:

indexing;
projection;
querying;
filtering;
ranking;
search synchronization.

Search remains a derived representation unless the domain explicitly establishes otherwise.

40. File Storage Component

File storage provides technical access to:

imports;
exports;
attachments;
generated files.

The application should interact through an abstraction.

41. Scheduling Component

Scheduling infrastructure invokes application operations according to time or operational triggers.

It should not contain business rules.

42. Background Processing Component

Background processing handles asynchronous application operations.

Examples:

event consumers;
projection updates;
imports;
exports;
synchronization.

Workers invoke application-level handlers.

43. Observability Component

Observability provides:

logs;
metrics;
tracing;
operational diagnostics.

Observability must not alter domain behavior.

44. Component Communication

The preferred communication mechanism between components is explicit contracts.

Interfaces
    ↓
Commands / Queries
    ↓
Application
    ↓
Domain

Infrastructure communicates through ports and adapters.

45. Component Dependency Matrix
Component	Domain	Application	Interfaces	Infrastructure
API Adapter	No direct business dependency	Yes	—	No direct persistence
UI Adapter	No	Yes	—	No direct persistence
Import Adapter	Through application	Yes	—	May use technical parsers
Use Case	Yes	—	No	Through ports
Command	Domain-independent input	Yes	No	No
Query	Domain-independent input	Yes	No	Through ports
Aggregate	—	No	No	No
Domain Service	Yes	No	No	No
Repository Port	Domain-aware	Application-aware	No	Implemented by Infrastructure
Repository Adapter	No business ownership	Implements ports	No	Yes
Event Publisher Port	Domain/Application contract	Yes	No	Implemented by Infrastructure
Event Transport	No	Through port	No	Yes
External Integration Adapter	No	Implements port	No	Yes
46. Component Dependency Rules
Rule C-001

Interfaces must depend on Application contracts.

Rule C-002

Application may depend on Domain.

Rule C-003

Application must not depend directly on concrete Infrastructure.

Rule C-004

Domain must not depend on Infrastructure.

Rule C-005

Infrastructure implements inward-facing contracts.

Rule C-006

Interfaces must not access persistence directly.

Rule C-007

Infrastructure must not bypass Application workflows for business operations.

47. Command Component Flow

The canonical command flow is:

External Request
      ↓
Interface Adapter
      ↓
Command
      ↓
Application Use Case
      ↓
Aggregate
      ↓
Domain Behavior
      ↓
Invariant
      ↓
State Change
      ↓
Domain Event
48. Query Component Flow

The canonical query flow is:

External Request
      ↓
Interface Adapter
      ↓
Query
      ↓
Application Query Handler
      ↓
Query Port
      ↓
Read Infrastructure
      ↓
Result
49. Event Component Flow

The canonical event flow is:

Aggregate
    ↓
Domain Event
    ↓
Application Event Handling
    ↓
Event Publication Port
    ↓
Infrastructure Event Adapter
    ↓
Transport
50. External Integration Flow
Use Case
   ↓
Integration Port
   ↑
Integration Adapter
   ↓
External System

This structure preserves replaceability.

51. Persistence Flow
Use Case
   ↓
Repository Port
   ↑
Repository Adapter
   ↓
Persistence Model
   ↓
Database

The database remains an implementation detail.

52. Component Lifecycle

Components should be evaluated through the following lifecycle:

Domain Requirement
       ↓
Architectural Responsibility
       ↓
Component
       ↓
Contract
       ↓
Implementation
       ↓
Verification

A component should not be introduced merely because a framework encourages it.

53. Avoiding Generic Service Layers

The architecture explicitly rejects a generic:

Service

layer as a catch-all.

Instead:

CreateCollectionUseCase
MoveItemUseCase
CollectionRepository
ItemRepository
CollectionPolicy
ItemClassificationService

should express responsibility explicitly.

54. Avoiding Generic Utility Components

Generic utility modules must be treated cautiously.

A utility should exist only if its responsibility is:

stable;
reusable;
well-defined;
not hiding domain behavior.

Business logic should never be hidden inside generic utilities.

55. Shared Infrastructure

Cross-cutting infrastructure may provide:

logging;
metrics;
tracing;
serialization;
configuration;
clock implementation;
technical ID generation.

However, shared infrastructure must not become a dependency sink for unrelated business functionality.

56. Component Granularity

Components should be:

cohesive;
independently understandable;
responsibility-oriented;
explicitly named;
traceable to requirements.

Components should not be split simply to create more classes or packages.

57. Component Ownership

Every component must have one primary owner.

The ownership test is:

Which architectural decision becomes invalid if this component is removed?

The answer identifies the component's responsibility.

58. Component Naming

Component names should use domain/application terminology whenever possible.

Prefer:

CollectionRepository
CollectionArchiver
ItemSearch
ImportCollection
CollectionPolicy

Avoid generic names such as:

Manager
Helper
Processor
Handler
Service
Utility

unless the responsibility is genuinely represented by that term.

59. Component Traceability

Every important component should eventually be traceable:

Component
   ↓
Responsibility
   ↓
Use Case / Domain Rule
   ↓
Domain Concept
   ↓
Business Requirement

This traceability will be formalized later in the architecture traceability matrix.

60. Component Boundaries and Transactions

Components must not imply transaction boundaries automatically.

For example:

CollectionService
ItemService
LocationService

does not mean:

3 Services
↓
3 Transactions

Transactions derive from consistency requirements.

61. Component Boundaries and Events

Similarly, components do not automatically require events.

Events should exist when there is a meaningful:

fact of domain significance

not merely because two classes need to communicate.

62. Component Boundaries and APIs

An API endpoint is not automatically a domain operation.

For example:

POST /collections/{id}/items

is an interface representation.

The application/domain operation may be:

AddItemToCollection

The architecture preserves this distinction.

63. Component Boundaries and Persistence

A database table is not automatically a domain entity.

Likewise:

CollectionTable

does not necessarily imply:

CollectionAggregate

The domain model remains authoritative.

64. Component Boundaries and Read Models

Read models are specialized components.

They may be optimized for:

search;
filtering;
dashboards;
reporting;
list views.

They must remain downstream from authoritative state.

65. Component Boundaries and Caching

Caching is an infrastructure concern unless the domain explicitly models cache state.

Caches must never be treated as authoritative sources of business truth without an explicit architectural decision.

66. Component Boundaries and Configuration

Technical configuration belongs outside the domain.

Examples:

database connection strings;
broker addresses;
API endpoints;
cache settings;
operational limits.

Domain constants, however, may belong to the domain when they represent genuine business rules.

67. Component Boundaries and Time

Time is a cross-cutting concern requiring explicit treatment.

Where business behavior depends on time, the architecture should provide an abstraction such as a clock through an inward-facing contract.

This allows deterministic domain testing.

68. Component Boundaries and Identity

Identity has multiple meanings:

Business Identity
Technical Persistence Identity
External System Identity

These must not automatically be treated as the same concept.

The domain model determines business identity.

Infrastructure handles technical identity.

69. Component Boundaries and Serialization

Serialization belongs at system boundaries.

The domain should not depend on:

JSON;
XML;
YAML;
protocol buffers;
database serialization formats.
70. Component Boundaries and Concurrency

Concurrency handling is implemented by Infrastructure/Application mechanisms.

The domain expresses the invariant.

For example:

Domain:
"Two operations must not produce an invalid collection state."


Infrastructure:
"Optimistic version check failed."

The two responsibilities remain separate.

71. Component Boundaries and Retry

Retry policy belongs to the Application/Infrastructure boundary.

Domain operations should remain deterministic with respect to the same input and state.

Technical retry should not cause accidental duplicate business effects.

72. Component Boundaries and Idempotency

Idempotency may require cooperation between:

Interface;
Application;
Infrastructure.

The business meaning of repeated operations remains a domain concern.

The technical mechanism for detecting duplicates may remain infrastructural.

73. Component Boundaries and Security

Security infrastructure may provide:

authentication;
credentials;
tokens;
identity resolution.

Application components perform authorization coordination.

Domain components enforce business rules.

74. Component Boundaries and Audit

Audit is downstream from application/domain activity.

The audit component must not become an alternative source of domain state.

75. Component Boundaries and Notifications

Notifications are side effects.

The architecture should use:

Domain Event
    ↓
Application Event Handling
    ↓
Notification Port
    ↓
Infrastructure Adapter

rather than:

Aggregate
    ↓
EmailClient
76. Component Boundaries and Search Indexing

Search indexing should be downstream:

Domain Change
      ↓
Domain Event
      ↓
Indexing Handler
      ↓
Search Adapter

This keeps search concerns out of aggregates.

77. Component Boundaries and Import/Export

Import:

External Data
     ↓
Interface/Import Adapter
     ↓
Application Workflow
     ↓
Domain

Export:

Application Query
     ↓
Read Model
     ↓
Export Adapter

The two operations are intentionally asymmetrical.

78. Architectural Component Baseline

The current baseline is:

INTERFACES
│
├── API Adapter
├── UI Adapter
├── CLI Adapter
├── Import Adapter
├── External Event Consumer
└── Scheduler Entry Point
│
APPLICATION
│
├── Use Cases
├── Commands
├── Queries
├── Workflows
├── Application Policies
├── Authorization
├── Ports
└── Application Results
│
DOMAIN
│
├── Aggregates
├── Entities
├── Value Objects
├── Domain Services
├── Specifications
├── Domain Policies
├── Domain Events
├── Domain Errors
└── Factories
│
INFRASTRUCTURE
│
├── Persistence
├── Transactions
├── Eventing
├── Integrations
├── Search
├── File Storage
├── Scheduling
├── Background Processing
└── Observability
79. What This Document Does Not Decide

This document intentionally does not define:

exact source-code folders;
exact namespaces/packages;
programming language;
framework;
database;
ORM;
message broker;
API framework;
deployment topology;
cloud provider.

Those decisions belong to subsequent architectural work.

80. Relationship With Document 24

The architectural progression is now:

23
Architectural Constraints
        ↓
24
Architectural Boundaries & Layers
        ↓
25
Architectural Components & Responsibilities

Each document answers a different question:

Document	Question
23	What must architecture protect?
24	Where are the architectural boundaries?
25	What components live inside those boundaries?
81. Next Step

The next document is:

26_APPLICATION_AND_DOMAIN_MODULE_STRUCTURE.md

It will translate the current component model into an initial module structure.

That document will answer:

how domain modules are organized;
how application modules correspond to use cases;
which components belong together;
how domain modules depend on each other;
where shared concepts live;
how module boundaries prevent accidental coupling;
how the future source tree should reflect the architecture.
82. Final Architectural Principle

The architecture must not be organized around technical nouns such as:

Controllers
Services
Repositories
Models
Utils

as the primary organizing principle.

Instead, the system should progressively expose:

Business Concepts
      ↓
Domain Modules
      ↓
Application Capabilities
      ↓
Architectural Components
      ↓
Technical Adapters

The final implementation should make it possible to navigate from a business capability to the code responsible for implementing it without crossing unnecessary architectural boundaries.

Document status: ACCEPTED AS ARCHITECTURAL BASELINE.

Next: 26_APPLICATION_AND_DOMAIN_MODULE_STRUCTURE.md.