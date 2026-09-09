# Domain Model Traceability Matrix

**CollectionHub — Phase 2.2: Domain Modeling**

**Status:** Draft / Consolidated  
**Document:** `21_DOMAIN_MODEL_TRACEABILITY_MATRIX.md`  
**Purpose:** Establish explicit traceability between business concepts, domain rules, invariants, aggregates, use cases, workflows, events, domain services, specifications, consistency decisions, and unresolved questions.

---

## 1. Purpose

This document provides the traceability matrix for the CollectionHub domain model produced throughout Phase 2.2.

The objective is not merely to list relationships between documents, but to verify that every important business concept has a traceable path through the model:

```text
Business Concept
    ↓
Domain Invariant
    ↓
Aggregate / Consistency Boundary
    ↓
Use Case
    ↓
Application Workflow
    ↓
Domain Rule / Specification / Service
    ↓
State Change
    ↓
Domain Event
    ↓
Observable Side Effect
```

The matrix is intended to answer the following questions:

1. Where is each important business concept represented?
2. Which invariants protect it?
3. Which aggregate owns the corresponding state?
4. Which use cases can modify or query it?
5. Which domain services participate in its behavior?
6. Which specifications formalize reusable rules?
7. Which domain events communicate meaningful state changes?
8. Which decisions constrain its implementation?
9. Are there business concepts without an owner?
10. Are there invariants without an enforcing mechanism?
11. Are there use cases without corresponding domain behavior?
12. Are there domain events without a meaningful business cause?
13. Are there unresolved questions that could invalidate the current model?

---

# 2. Traceability Principles

The CollectionHub model follows these principles.

### 2.1 Every business-critical rule must have an owner

A rule must be associated with one of:

- an entity;
- a value object;
- an aggregate;
- a domain service;
- a specification;
- or an explicit architectural/application policy.

Rules must not exist only as undocumented assumptions.

### 2.2 Every invariant must have an enforcement point

For each invariant we must be able to identify where it is guaranteed.

Typical enforcement locations are:

```text
Value Object
    ↓
Entity
    ↓
Aggregate
    ↓
Domain Service
    ↓
Application Boundary
```

The higher the invariant's scope, the less appropriate it is to hide it inside a local entity.

### 2.3 Every state-changing use case must have domain behavior

An application service must orchestrate domain behavior rather than becoming the location where business rules are implemented.

```text
Application Service
    → loads aggregate
    → invokes domain behavior
    → persists resulting state
    → publishes resulting domain events
```

### 2.4 Events represent meaningful domain facts

A domain event must represent something that became true in the domain.

Events must not be introduced merely because an implementation needs an integration message.

### 2.5 Traceability must be bidirectional

It must be possible to navigate:

```text
Requirement → Model
```

and:

```text
Model → Business Need
```

This prevents both under-modeling and accidental complexity.

---

# 3. Document-Level Traceability

| Source Document | Primary Responsibility | Traceability Target |
|---|---|---|
| `00_DOMAIN_CONCEPT_INVENTORY.md` | Identify domain concepts | Concepts → model elements |
| `01_DOMAIN_GLOSSARY.md` | Establish canonical terminology | Terms → concepts |
| `02_DOMAIN_ACTORS_AND_ROLES.md` | Identify domain actors | Actors → capabilities/use cases |
| `03_DOMAIN_CAPABILITIES.md` | Identify business capabilities | Capabilities → use cases |
| `04_DOMAIN_RELATIONSHIPS.md` | Identify conceptual relationships | Concepts → relationships |
| `05_DOMAIN_LIFECYCLES.md` | Identify lifecycle behavior | Concepts → states/transitions |
| `06_DOMAIN_POLICIES.md` | Identify explicit policies | Policies → rules/invariants |
| `07_DOMAIN_RULES.md` | Identify business rules | Rules → enforcement |
| `08_DOMAIN_INVARIANTS.md` | Identify non-negotiable truths | Invariants → owners |
| `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md` | Define transactional ownership | Invariants → aggregates |
| `10_DOMAIN_ENTITIES_AND_VALUE_OBJECTS.md` | Define domain building blocks | Concepts → entities/value objects |
| `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md` | Define business operations | Capabilities → use cases |
| `12_APPLICATION_USE_CASES_AND_WORKFLOWS.md` | Define orchestration | Use cases → workflows |
| `13_DOMAIN_COMMANDS_AND_INPUT_MODELS.md` | Define domain/application inputs | Workflows → commands |
| `14_DOMAIN_STATE_TRANSITIONS.md` | Define valid state changes | Lifecycle → transitions |
| `15_DOMAIN_REPOSITORIES_AND_PERSISTENCE_CONTRACTS.md` | Define persistence boundaries | Aggregates → repositories |
| `16_DOMAIN_EVENTS_AND_SIDE_EFFECTS.md` | Define domain events | State changes → events |
| `17_DOMAIN_SERVICES_AND_CROSS_AGGREGATE_RULES.md` | Define behavior crossing boundaries | Rules → services |
| `18_DOMAIN_SPECIFICATIONS_AND_REUSABLE_RULES.md` | Formalize reusable predicates/rules | Rules → specifications |
| `19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md` | Validate model coherence | Model → consistency findings |
| `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` | Record decisions and uncertainty | Decisions/questions → model |
| `21_DOMAIN_MODEL_TRACEABILITY_MATRIX.md` | Consolidate traceability | Entire model → business intent |

---

# 4. Core Concept Traceability Matrix

The following matrix represents the canonical traceability structure for the principal domain concepts.

| Domain Concept | Lifecycle | Invariants | Aggregate Owner | Use Cases | Domain Behavior | Events | Specifications | Decision / Question |
|---|---|---|---|---|---|---|---|---|
| Collection | Collection lifecycle | Collection identity and ownership rules | Collection aggregate | Create, update, archive collection | Collection behavior | CollectionCreated, CollectionUpdated, CollectionArchived | Collection ownership/status specifications | Collection boundary decision |
| Collectible Item | Item lifecycle | Identity, ownership and valid state rules | Collection Item / Collectible aggregate | Add, update, remove item | Item state transitions | ItemAdded, ItemUpdated, ItemRemoved | Item validity specifications | Item identity decision |
| Item Classification | Classification lifecycle | Classification consistency | Classification-related aggregate/value model | Classify/reclassify | Classification behavior | ItemClassified, ItemReclassified | Classification specifications | Classification strategy |
| Item Metadata | Metadata lifecycle | Metadata validity | Item aggregate or metadata boundary | Add/update metadata | Metadata validation | MetadataUpdated | Metadata specifications | Metadata ownership |
| Acquisition | Acquisition lifecycle | Acquisition integrity | Item / Acquisition aggregate depending boundary | Record acquisition | Acquisition behavior | AcquisitionRecorded | Acquisition specifications | Acquisition ownership |
| Disposal | Disposal lifecycle | Item cannot be disposed inconsistently | Item / Disposal boundary | Dispose item | Disposal behavior | ItemDisposed | Disposal specifications | Disposal model |
| Location | Location lifecycle | Location identity and hierarchy rules | Location aggregate | Create/update/move location | Location behavior | LocationCreated, LocationUpdated, ItemRelocated | Location specifications | Location hierarchy |
| Item Location | Assignment lifecycle | An item has valid location assignment | Item or Location relationship boundary | Assign/move item | Relocation behavior | ItemRelocated | Location assignment specification | Ownership of assignment |
| Collection Membership | Membership lifecycle | Membership consistency | Collection aggregate | Add/remove item | Membership behavior | ItemAddedToCollection, ItemRemovedFromCollection | Membership specifications | Membership cardinality |
| Tag / Label | Tag lifecycle | Valid tag identity | Tag aggregate or value object | Create/apply/remove tag | Tag behavior | TagCreated, TagApplied, TagRemoved | Tag specifications | Tag ownership |
| Notes / Annotations | Annotation lifecycle | Valid association | Item aggregate | Add/update/remove note | Annotation behavior | NoteAdded, NoteUpdated, NoteRemoved | Annotation specifications | Annotation ownership |
| Search / Query Model | Query lifecycle | Query consistency | Read model | Search/filter | Query services | Optional read-side events | Query specifications | CQRS/read-model decision |
| Import | Import lifecycle | Imported data must satisfy domain rules | Import process boundary | Import collection/items | Import domain service | ImportCompleted, ImportFailed | Import validation specifications | Import architecture |
| Export | Export lifecycle | Export must represent valid domain state | Export process boundary | Export data | Export service | ExportCompleted | Export specifications | Export contract |
| Audit History | Audit lifecycle | Historical records are immutable | Audit boundary | View history | Audit recording behavior | Audit events | Audit specifications | Audit strategy |

> The exact aggregate names and boundaries remain subordinate to the decisions recorded in the canonical aggregate and decision documents.

---

# 5. Invariant Traceability

Every invariant must be traceable to an enforcement mechanism.

| Invariant Category | Invariant | Owner | Enforcement | Relevant Use Cases | Failure Strategy |
|---|---|---|---|---|---|
| Identity | Domain identity must be valid and stable | Entity / Aggregate | Entity construction | Create/update operations | Reject invalid command |
| Ownership | Collection data belongs to the correct owner/context | Aggregate | Aggregate authorization boundary | Collection operations | Reject operation |
| Membership | An item cannot be simultaneously inconsistent with its collection membership state | Collection aggregate | Aggregate method | Add/remove item | Domain error |
| Lifecycle | State transitions must follow the declared lifecycle | Aggregate | Transition methods | Update/archive/dispose | Domain error |
| Referential Integrity | Referenced domain objects must exist when required | Aggregate / Service | Repository lookup + domain validation | Cross-object operations | Domain error |
| Uniqueness | Values declared unique cannot collide | Aggregate / Domain Service | Specification + repository constraint | Create/update | Conflict error |
| Validity | Invalid value combinations cannot enter the model | Value Object / Specification | Constructor/factory/specification | All write operations | Validation error |
| Consistency | Related state must remain coherent inside a consistency boundary | Aggregate | Aggregate transaction | State-changing use cases | Transaction failure |
| Immutability | Historical facts must not be silently rewritten | Event / Audit model | Append-only policy | Audit/history | Reject mutation |
| Transition Integrity | Events must correspond to actual valid transitions | Aggregate | Event emission after transition | State-changing operations | No event / rollback |

---

# 6. Aggregate Traceability Matrix

Aggregates are the primary consistency boundaries.

| Aggregate | Protects | Owns | Commands | Invariants | Emits |
|---|---|---|---|---|---|
| Collection Aggregate | Collection-level consistency | Collection identity, status and membership where applicable | CreateCollection, UpdateCollection, ArchiveCollection, AddItem, RemoveItem | Collection invariants, membership invariants | CollectionCreated, CollectionUpdated, CollectionArchived |
| Collectible / Item Aggregate | Item-level consistency | Item identity, metadata and lifecycle state | CreateItem, UpdateItem, DisposeItem, AssignLocation | Item invariants, lifecycle invariants | ItemCreated, ItemUpdated, ItemDisposed |
| Location Aggregate | Location consistency | Location identity and hierarchy | CreateLocation, UpdateLocation | Location invariants | LocationCreated, LocationUpdated |
| Tag Aggregate | Tag consistency | Tag identity and semantics | CreateTag, UpdateTag | Tag invariants | TagCreated, TagUpdated |
| Import Aggregate / Process | Import consistency | Import execution state | StartImport, ValidateImport, CompleteImport | Import validation rules | ImportStarted, ImportCompleted, ImportFailed |
| Audit Boundary | Historical consistency | Audit records | RecordAuditEntry | Append-only invariant | AuditRecorded |

### Aggregate boundary verification

For each aggregate, the following must remain true:

```text
Aggregate owns invariant
        ↓
Command enters aggregate
        ↓
Aggregate validates transition
        ↓
Aggregate changes state
        ↓
Aggregate records domain event
```

If a rule requires coordination between two aggregates, it must not be implemented by silently reaching into the internal state of the other aggregate.

---

# 7. Use Case Traceability

| Use Case | Business Capability | Command | Aggregate(s) | Domain Rules | Events | Side Effects |
|---|---|---|---|---|---|---|
| Create Collection | Collection Management | CreateCollection | Collection | Collection creation rules | CollectionCreated | Persistence |
| Update Collection | Collection Management | UpdateCollection | Collection | Update rules | CollectionUpdated | Persistence |
| Archive Collection | Collection Management | ArchiveCollection | Collection | Lifecycle rules | CollectionArchived | Persistence / projection |
| Add Item | Collection Management | AddItem | Collection + Item as applicable | Membership rules | ItemAdded / MembershipChanged | Persistence |
| Update Item | Item Management | UpdateItem | Item | Item validity rules | ItemUpdated | Persistence |
| Remove Item | Collection Management | RemoveItem | Collection / Item | Removal rules | ItemRemoved | Persistence |
| Assign Location | Location Management | AssignLocation | Item + Location | Assignment rules | ItemRelocated | Persistence / projection |
| Create Location | Location Management | CreateLocation | Location | Location rules | LocationCreated | Persistence |
| Apply Tag | Classification | ApplyTag | Item / Tag | Tag rules | TagApplied | Persistence |
| Remove Tag | Classification | RemoveTag | Item / Tag | Tag rules | TagRemoved | Persistence |
| Record Acquisition | Acquisition Management | RecordAcquisition | Acquisition / Item | Acquisition rules | AcquisitionRecorded | Persistence |
| Dispose Item | Lifecycle Management | DisposeItem | Item | Disposal rules | ItemDisposed | Persistence / audit |
| Import Data | Data Ingestion | ImportData | Import process + domain aggregates | Import specifications | ImportCompleted / Failed | Persistence / reporting |
| Export Data | Data Export | ExportData | Read model / export service | Export specifications | ExportCompleted | File generation |
| Search Collection | Discovery | SearchCollection | Read model | Search specifications | None normally | Query response |

---

# 8. Workflow Traceability

Application workflows should remain orchestration mechanisms.

| Workflow Stage | Responsibility | Domain Ownership |
|---|---|---|
| Receive command | Validate command shape | Application layer |
| Resolve actor/context | Establish execution context | Application layer |
| Load aggregate(s) | Retrieve required domain state | Repository |
| Invoke behavior | Execute business rule | Domain |
| Persist state | Store resulting state | Repository |
| Collect events | Obtain domain facts | Aggregate |
| Publish events | Dispatch domain events | Application/infrastructure |
| Execute side effects | Notifications, projections, integrations | Application/infrastructure |
| Return result | Map domain result | Application layer |

A workflow must not become a hidden second domain model.

---

# 9. Domain Service Traceability

Domain services exist only where behavior cannot naturally belong to one aggregate or value object.

| Domain Service | Reason for Existence | Inputs | Rules | Output |
|---|---|---|---|---|
| Collection Classification Service | Classification requiring multiple domain concepts | Item + classification context | Classification rules | Classification result |
| Cross-Aggregate Membership Service | Coordination between collection and item boundaries | Collection + Item | Membership rules | Validated membership operation |
| Import Validation Service | Validate imported structures against domain rules | Imported data | Import specifications | Validation result |
| Duplicate Detection Service | Detect semantic duplicates across boundaries | Candidate + existing domain state | Duplicate specification | Duplicate analysis |
| Location Assignment Service | Coordinate item/location consistency | Item + Location | Location assignment rules | Assignment result |

A service should be removed if its behavior can be expressed naturally inside a single aggregate.

---

# 10. Specification Traceability

Specifications provide reusable, composable business predicates.

| Specification | Purpose | Used By | Related Invariants |
|---|---|---|---|
| ValidCollectionSpecification | Determines whether a collection satisfies required conditions | Collection creation/update | Collection validity |
| ActiveCollectionSpecification | Determines whether a collection can receive operations | Membership workflows | Lifecycle |
| ValidItemSpecification | Validates item state | Item creation/update | Item validity |
| ItemBelongsToCollectionSpecification | Verifies membership | Collection operations | Membership |
| ValidLocationSpecification | Validates location | Location operations | Location validity |
| ItemCanBeRelocatedSpecification | Determines whether an item may change location | Relocation workflow | Location/lifecycle |
| ValidTagSpecification | Validates tag application | Tag operations | Tag validity |
| DuplicateItemSpecification | Detects domain-level duplication | Item creation/import | Uniqueness |
| ImportableItemSpecification | Determines whether imported data can become domain state | Import workflow | Import validity |

Specifications should remain declarative wherever possible.

---

# 11. Event Traceability

Domain events must be traceable backwards to the business transition that produced them.

| Event | Business Fact | Triggering Use Case | Aggregate | Invariant Protected | Possible Consumers |
|---|---|---|---|---|---|
| CollectionCreated | A collection was created | Create Collection | Collection | Collection identity | Read model, audit |
| CollectionUpdated | Collection data changed | Update Collection | Collection | Collection validity | Read model, audit |
| CollectionArchived | Collection became archived | Archive Collection | Collection | Lifecycle | Read model, audit |
| ItemCreated | Item entered domain | Create Item | Item | Item identity | Read model, audit |
| ItemUpdated | Item data changed | Update Item | Item | Item validity | Read model, audit |
| ItemRemoved | Item ceased membership / existence in context | Remove Item | Collection / Item | Membership | Read model, audit |
| ItemRelocated | Item location changed | Assign Location | Item | Location consistency | Read model, audit |
| TagApplied | Tag became associated with item | Apply Tag | Item / Tag | Tag consistency | Read model |
| TagRemoved | Tag association ceased | Remove Tag | Item / Tag | Tag consistency | Read model |
| AcquisitionRecorded | Acquisition fact recorded | Record Acquisition | Acquisition | Acquisition integrity | Audit |
| ItemDisposed | Item entered disposed state | Dispose Item | Item | Lifecycle | Audit, reporting |
| ImportCompleted | Import process completed | Import | Import process | Import integrity | Reporting |
| ImportFailed | Import process failed | Import | Import process | Import integrity | Monitoring |
| AuditRecorded | Historical fact recorded | Domain action | Audit | History integrity | Audit consumers |

---

# 12. Side-Effect Traceability

Side effects must be downstream from domain facts.

```text
Domain State Change
        ↓
Domain Event
        ↓
Event Handler
        ↓
Side Effect
```

| Domain Event | Side Effect | Boundary |
|---|---|---|
| CollectionCreated | Update read model | Infrastructure |
| CollectionUpdated | Refresh projection | Infrastructure |
| CollectionArchived | Update search/index state | Infrastructure |
| ItemCreated | Add item to read model | Infrastructure |
| ItemUpdated | Update projection | Infrastructure |
| ItemRelocated | Update location projection | Infrastructure |
| ItemDisposed | Record audit / update reporting | Infrastructure |
| TagApplied | Update search/index | Infrastructure |
| ImportCompleted | Refresh derived data | Infrastructure |
| ExportCompleted | Record export activity | Infrastructure |

The domain model must not depend directly on these side effects.

---

# 13. Lifecycle Traceability

Lifecycle transitions must be explicitly connected to commands and events.

| Concept | State / Transition | Command | Domain Operation | Event |
|---|---|---|---|---|
| Collection | New → Active | CreateCollection | Create | CollectionCreated |
| Collection | Active → Updated | UpdateCollection | Update | CollectionUpdated |
| Collection | Active → Archived | ArchiveCollection | Archive | CollectionArchived |
| Item | New → Active | CreateItem | Create | ItemCreated |
| Item | Active → Updated | UpdateItem | Update | ItemUpdated |
| Item | Active → Disposed | DisposeItem | Dispose | ItemDisposed |
| Item Location | Unassigned → Assigned | AssignLocation | Assign | ItemRelocated |
| Item Location | Assigned → Assigned elsewhere | AssignLocation | Relocate | ItemRelocated |
| Import | Pending → Processing | ImportData | Start | ImportStarted |
| Import | Processing → Completed | ImportData | Complete | ImportCompleted |
| Import | Processing → Failed | ImportData | Fail | ImportFailed |

Any lifecycle transition without a defined domain operation is a traceability gap.

---

# 14. Repository Traceability

Repositories correspond to aggregate persistence boundaries.

| Aggregate | Repository Contract | Responsibilities | Must Not Contain |
|---|---|---|---|
| Collection | CollectionRepository | Load/save collections | Business decisions |
| Item | ItemRepository | Load/save items | Domain rules |
| Location | LocationRepository | Load/save locations | Location policy |
| Tag | TagRepository | Load/save tags | Tag semantics |
| Import | ImportRepository if persistent | Load/save import state | Import validation logic |
| Audit | AuditRepository | Append/read audit records | Mutation of historical records |

Repository abstractions must express domain needs rather than expose persistence technology.

---

# 15. Requirement-to-Model Traceability

The conceptual business requirements can be mapped to model elements as follows.

| Business Need | Domain Representation | Enforcement | Observable Result |
|---|---|---|---|
| Manage collections | Collection aggregate | Collection invariants | Collection lifecycle events |
| Manage individual items | Item aggregate | Item invariants | Item events |
| Organize items | Membership/location model | Membership/location rules | Membership/relocation events |
| Classify items | Classification/tag model | Classification specifications | Classification events |
| Preserve history | Audit model | Append-only rules | Audit records |
| Import existing data | Import workflow + validation service | Import specifications | Import result events |
| Export domain data | Export service | Export specifications | Export result |
| Search and discover | Read/query model | Query specifications | Search results |
| Maintain consistency | Aggregates | Invariants | Valid state |
| Support future integrations | Domain events | Event contracts | External projections/integrations |

---

# 16. Traceability Coverage Matrix

The following matrix is used as a completeness check.

Legend:

- **✓** = explicitly traceable
- **△** = partially defined / requires verification
- **—** = not applicable
- **?** = unresolved

| Model Element | Concept | Rule | Invariant | Aggregate | Use Case | Workflow | Event | Service | Specification | Decision |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Collection | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | ✓ | ✓ |
| Item | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | ✓ | ✓ |
| Classification | ✓ | ✓ | ✓ | △ | ✓ | ✓ | ✓ | △ | ✓ | ? |
| Location | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | △ | ✓ | ✓ |
| Membership | ✓ | ✓ | ✓ | △ | ✓ | ✓ | ✓ | △ | ✓ | ? |
| Tags | ✓ | ✓ | ✓ | △ | ✓ | ✓ | ✓ | — | ✓ | ? |
| Acquisition | ✓ | ✓ | ✓ | △ | ✓ | ✓ | ✓ | △ | ✓ | ? |
| Disposal | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — | ✓ | ✓ |
| Import | ✓ | ✓ | ✓ | △ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Export | ✓ | ✓ | △ | — | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Audit | ✓ | ✓ | ✓ | ✓ | △ | ✓ | ✓ | — | ✓ | ✓ |
| Search | ✓ | ✓ | △ | — | ✓ | ✓ | — | ✓ | ✓ | ✓ |

The `△` and `?` entries are deliberate indicators for areas that must be verified before implementation.

---

# 17. Bidirectional Traceability Rules

## 17.1 Concept → Implementation

Every important domain concept should follow this path:

```text
Concept
  ↓
Entity / Value Object / Aggregate
  ↓
Invariant
  ↓
Use Case
  ↓
Workflow
  ↓
Persistence
  ↓
Event
```

If a concept stops before reaching a model owner, it is under-modeled.

---

## 17.2 Implementation → Business Meaning

Every domain element must have a reverse explanation:

```text
Aggregate
  ↓
What business concept does it protect?

Domain Service
  ↓
What business rule requires it?

Specification
  ↓
What business decision does it formalize?

Domain Event
  ↓
What business fact does it communicate?
```

If an element cannot answer these questions, it is a candidate for removal or redesign.

---

# 18. Orphan Detection

The following categories must be treated as traceability defects.

### 18.1 Orphan concepts

A business concept exists in the glossary or inventory but has no:

- entity;
- value object;
- aggregate;
- service;
- or explicit architectural representation.

### 18.2 Orphan invariants

An invariant exists without:

- an owner;
- an enforcement point;
- or a corresponding testable specification.

### 18.3 Orphan use cases

A use case exists without:

- domain behavior;
- aggregate interaction;
- or an explicit reason why it is purely application-level.

### 18.4 Orphan events

An event exists without:

- a triggering transition;
- an aggregate/source;
- or a business fact.

### 18.5 Orphan services

A domain service exists without a rule that genuinely crosses aggregate or object boundaries.

### 18.6 Orphan specifications

A specification exists without a concrete business rule or reusable decision.

### 18.7 Orphan decisions

A decision affects the model but is not reflected in:

- aggregates;
- rules;
- workflows;
- or implementation constraints.

---

# 19. Completeness Criteria

The domain model should not progress to implementation until the following conditions are satisfied.

## 19.1 Concept coverage

- [ ] Every business-critical concept has an explicit representation.
- [ ] Synonyms and ambiguous terms have been resolved.
- [ ] No critical concept exists only in application code or UI terminology.

## 19.2 Rule coverage

- [ ] Every important business rule has an identified owner.
- [ ] Rules affecting consistency are represented as invariants.
- [ ] Reusable predicates are represented as specifications where appropriate.

## 19.3 Aggregate coverage

- [ ] Every invariant has an aggregate or object capable of enforcing it.
- [ ] Aggregate boundaries are justified by consistency requirements.
- [ ] Cross-aggregate rules are explicitly identified.

## 19.4 Use-case coverage

- [ ] Every major capability has corresponding use cases.
- [ ] Every state-changing use case invokes domain behavior.
- [ ] Application services do not contain hidden domain rules.

## 19.5 Event coverage

- [ ] Every meaningful state transition has an identifiable domain fact.
- [ ] Events are emitted only after successful domain transitions.
- [ ] Event consumers do not alter the source aggregate directly.

## 19.6 Decision coverage

- [ ] Every unresolved architectural/domain question is documented.
- [ ] Important decisions are reflected consistently throughout the model.
- [ ] No document contradicts an accepted decision.

---

# 20. Traceability Risk Classification

| Risk | Meaning | Action |
|---|---|---|
| Critical | Business rule has no enforcement point | Resolve before implementation |
| High | Aggregate boundary is unclear | Resolve before persistence design |
| Medium | Event/service/specification relationship is incomplete | Resolve before infrastructure implementation |
| Low | Documentation relationship is incomplete | Resolve during model consolidation |
| Informational | Relationship is documented but intentionally indirect | No action |

---

# 21. Current Traceability Assessment

Based on the Phase 2.2 model sequence, the domain model has reached a level where the main conceptual chain can be expressed coherently:

```text
Concept Inventory
      ↓
Glossary
      ↓
Capabilities / Relationships
      ↓
Lifecycle
      ↓
Policies
      ↓
Rules
      ↓
Invariants
      ↓
Aggregates
      ↓
Entities / Value Objects
      ↓
Use Cases
      ↓
Application Workflows
      ↓
Commands
      ↓
State Transitions
      ↓
Repositories
      ↓
Domain Events
      ↓
Domain Services
      ↓
Specifications
      ↓
Consistency Review
      ↓
Decisions & Open Questions
      ↓
Traceability Matrix
```

This is an important milestone because the model is no longer a collection of independent domain documents. It forms a connected system of reasoning.

---

# 22. Critical Traceability Questions

Before considering the domain model implementation-ready, the following questions must have explicit answers.

### Aggregates

1. Does every invariant have a clear aggregate owner?
2. Are any aggregates enforcing rules that belong elsewhere?
3. Are there aggregates whose boundaries are motivated only by database tables?

### Cross-Aggregate Rules

4. Which rules genuinely require more than one aggregate?
5. Are those rules implemented through domain services or application orchestration appropriately?
6. Could any cross-aggregate rule actually indicate an incorrect aggregate boundary?

### Events

7. Does every event represent a meaningful domain fact?
8. Are events emitted by the aggregate that owns the state transition?
9. Are any events really integration messages rather than domain events?

### Specifications

10. Which rules are sufficiently reusable to justify specifications?
11. Are specifications being used to express business semantics rather than simple technical validation?

### Use Cases

12. Does every use case have a clear business purpose?
13. Are there use cases that exist only because of a UI interaction?
14. Are application workflows free from domain decision-making?

### Persistence

15. Does repository structure follow aggregate boundaries?
16. Are persistence constraints reinforcing rather than defining domain invariants?

### Open Questions

17. Are any unresolved questions capable of changing aggregate boundaries?
18. Are any unresolved questions capable of changing lifecycle semantics?
19. Are any unresolved questions capable of changing event contracts?

---

# 23. Traceability Acceptance Criteria

The model may be considered **traceability-complete** when:

```text
For every critical business concept:
    concept → model owner exists

For every critical business rule:
    rule → invariant/specification exists

For every invariant:
    invariant → enforcement point exists

For every aggregate:
    aggregate → business responsibility exists

For every state-changing use case:
    use case → domain behavior exists

For every meaningful transition:
    transition → domain event exists where appropriate

For every domain service:
    service → cross-boundary business reason exists

For every specification:
    specification → reusable business rule exists

For every decision:
    decision → affected model elements are identifiable

For every open question:
    impact → identifiable
```

---

# 24. Final Traceability Model

The consolidated CollectionHub domain model should ultimately satisfy the following graph:

```text
                         ┌─────────────────────┐
                         │ Business Concepts   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Business Rules      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Domain Invariants   │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Aggregates          │
                         │ & Boundaries        │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
          ┌─────────────┐   ┌──────────────┐   ┌──────────────┐
          │ Entities    │   │ Value        │   │ Specifications│
          │             │   │ Objects      │   │              │
          └──────┬──────┘   └──────────────┘   └──────────────┘
                 │
                 ▼
          ┌─────────────────┐
          │ Domain Behavior │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ Use Cases       │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ Application     │
          │ Workflows       │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ State Changes   │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ Domain Events   │
          └────────┬────────┘
                   │
          ┌────────┴──────────────┐
          ▼                       ▼
   ┌──────────────┐       ┌─────────────────┐
   │ Projections  │       │ Side Effects    │
   │ / Read Model │       │ / Integrations  │
   └──────────────┘       └─────────────────┘
```

The crucial property of this graph is that **business intent flows downward into implementation structure, while domain facts flow upward through events and observable behavior**.

---

# 25. Relationship with Previous Documents

This document is deliberately dependent on the complete Phase 2.2 model.

It does not replace any of the previous documents.

Instead:

```text
00–18
   ↓
19 — Consistency Review
   ↓
20 — Decisions & Open Questions
   ↓
21 — Traceability Matrix
```

Therefore, if a contradiction is discovered in this matrix, the matrix must not silently override the source model. The contradiction should be recorded and resolved in the appropriate canonical document.

---

# 26. Role of This Document in the Next Phase

`21_DOMAIN_MODEL_TRACEABILITY_MATRIX.md` closes the analytical chain of the current domain-modeling iteration.

The next phase should use this matrix as a verification instrument when translating the domain model into implementation architecture.

The implementation process should be able to answer:

> **"Where does this piece of code come from in the domain model?"**

and trace it backwards:

```text
Code
 ↓
Application Service / Domain Service
 ↓
Use Case
 ↓
Domain Behavior
 ↓
Aggregate / Entity / Value Object
 ↓
Invariant / Rule
 ↓
Business Concept
```

Likewise, every critical business concept should be able to trace forward toward implementation.

This establishes a foundation for architecture and coding without allowing implementation details to redefine the domain accidentally.

---

# 27. Final Principle

The purpose of traceability is not documentation for its own sake.

The purpose is to preserve **causal integrity** between business meaning and software structure.

CollectionHub should therefore maintain the following invariant at the architectural level:

> **Every significant implementation decision must be explainable as a consequence of an identified domain concept, rule, invariant, use case, lifecycle decision, or explicitly recorded architectural decision.**

If a piece of implementation cannot be traced back to the domain model, it is either:

1. a legitimate technical concern that belongs outside the domain model;
2. an undocumented requirement;
3. accidental complexity;
4. or a signal that the domain model is incomplete.

That distinction should be maintained throughout the remainder of the project.