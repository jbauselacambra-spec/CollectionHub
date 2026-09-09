# 04 — AI Domain Interaction and Boundaries

## 1. Purpose

This document defines how the AI subsystem interacts with the CollectionHub domain and, critically, where that interaction must stop.

The objective is to establish explicit architectural boundaries between:

- AI capabilities.
- Application use cases.
- Domain logic.
- Domain entities and aggregates.
- Domain services.
- Domain events.
- Persistence.
- External integrations.
- AI-specific infrastructure.

The AI subsystem is an **application capability that consumes and produces information around the domain**. It is not the owner of domain truth and must not become an alternative implementation of the domain model.

The domain remains authoritative for:

- business invariants;
- aggregate consistency;
- state transitions;
- business rules;
- authorization-relevant domain decisions;
- domain identity;
- domain relationships;
- domain events;
- persistence of authoritative business state.

AI may assist with interpretation, classification, extraction, recommendation, generation, search, summarization, enrichment, and other explicitly approved capabilities, but it must operate through defined application contracts and domain boundaries.

---

# 2. Architectural Position

The AI subsystem belongs above the domain model in terms of decision responsibility.

The conceptual dependency direction is:

```text
User / External Actor
        |
        v
AI Interaction / Application Layer
        |
        v
AI Application Capabilities
        |
        v
Application Use Cases
        |
        v
Domain Model
        |
        v
Persistence / Infrastructure
```

The AI subsystem may also consume information from:

```text
Domain
  |
  +--> Domain Queries / Read Models
  |
  +--> Application Services
  |
  +--> Domain Events
  |
  +--> Approved Context Providers
```

However, the AI subsystem must not bypass the application and domain boundaries to manipulate persistence directly.

The forbidden dependency direction is:

```text
AI
 |
 +--> Direct database manipulation
 |
 +--> Direct aggregate mutation
 |
 +--> Direct EF Core access
 |
 +--> Direct infrastructure ownership of domain rules
```

AI infrastructure may use persistence mechanisms to store **AI-owned technical state**, but this must remain separate from authoritative domain state.

---

# 3. Core Architectural Principle

## 3.1 AI is not the domain

CollectionHub must never treat a language model, agent, prompt, embedding model, vector store, or AI-generated output as the authoritative representation of the domain.

The following principle is mandatory:

> AI interprets, assists, proposes, transforms, and enriches domain information; the domain validates and owns business truth.

This means:

- AI cannot redefine an aggregate invariant.
- AI cannot decide that an invalid domain transition is valid.
- AI cannot create an authoritative business state merely because a model generated it.
- AI cannot replace domain services.
- AI cannot replace application authorization.
- AI cannot bypass application use cases.
- AI cannot directly persist business state without passing through the appropriate application/domain boundary.

---

# 4. AI Interaction Categories

AI-domain interaction is divided into six categories.

## 4.1 Domain Observation

AI consumes information originating from the domain.

Examples:

- collection data;
- item data;
- user-owned collection information;
- domain classifications;
- domain relationships;
- historical events;
- domain metadata;
- read models.

AI may interpret this information but does not own it.

---

## 4.2 Domain Context Construction

AI may require a contextual representation of domain information.

The AI subsystem may construct:

- prompts;
- context packages;
- retrieval results;
- summaries;
- semantic representations;
- embeddings;
- model-specific input structures.

These representations are derived artifacts.

They must not be considered replacements for the underlying domain model.

---

## 4.3 AI Interpretation

AI may interpret domain information.

Examples include:

- extracting structured information;
- classifying an item;
- identifying possible duplicates;
- interpreting natural-language descriptions;
- generating candidate metadata;
- suggesting relationships;
- summarizing collections;
- identifying possible anomalies.

Interpretation does not automatically become domain truth.

---

## 4.4 AI Recommendation

AI may propose actions or values.

Examples:

- suggested item classification;
- suggested metadata;
- suggested categorization;
- suggested duplicate matches;
- suggested collection organization;
- suggested enrichment;
- suggested next actions.

Recommendations must remain distinguishable from authoritative domain state until explicitly accepted through an appropriate application flow.

---

## 4.5 Domain Command Execution

AI may initiate or assist an application command where explicitly authorized.

The sequence must remain:

```text
AI Proposal
    |
    v
Application Command
    |
    v
Authorization
    |
    v
Domain Validation
    |
    v
Domain State Change
    |
    v
Persistence
```

AI does not replace the application command.

---

## 4.6 Domain Event Consumption

AI may consume domain events where there is a justified use case.

Examples:

```text
ItemCreated
ItemUpdated
CollectionCreated
CollectionUpdated
ItemAssociatedWithCollection
ItemRemovedFromCollection
```

The exact event catalogue is owned by the domain architecture.

AI consumption of events must be:

- asynchronous where appropriate;
- resilient to retries;
- idempotent;
- observable;
- tolerant of delayed processing;
- isolated from the originating domain transaction unless explicitly required.

AI processing must not cause the originating domain transaction to depend on successful model execution unless a specific synchronous business requirement has been formally established.

---

# 5. Domain Ownership

The following ownership model is mandatory.

| Concern | Owner |
|---|---|
| Entity identity | Domain |
| Aggregate boundaries | Domain |
| Invariants | Domain |
| Business rules | Domain |
| Domain state transitions | Domain |
| Domain events | Domain |
| Application authorization | Application |
| Use-case orchestration | Application |
| AI interpretation | AI |
| Prompt construction | AI |
| Model selection | AI infrastructure |
| AI context construction | AI |
| AI-generated recommendations | AI |
| AI execution orchestration | AI/Application |
| AI technical state | AI infrastructure |
| Business persistence | Persistence/Application |
| AI persistence | AI infrastructure |
| External AI provider integration | Infrastructure |

This separation prevents the AI layer from becoming a hidden business-rule engine.

---

# 6. AI Must Not Own Domain Truth

AI-generated information has a different epistemic status from domain-authoritative information.

The system must distinguish at least conceptually between:

```text
Authoritative Domain Data
        |
        +--> validated by domain rules

AI-Derived Data
        |
        +--> generated or inferred by AI

AI Recommendation
        |
        +--> proposed but not accepted

AI Evidence / Context
        |
        +--> supporting information used during reasoning
```

These categories must not be silently conflated.

For example:

```text
AI:
"The item appears to be a first-edition book."

Incorrect:
    Persist FirstEdition = true

Correct:
    Create recommendation:
        ProposedClassification = FirstEdition
        Confidence = ...
        Evidence = ...

    Application flow:
        User/system accepts recommendation

    Domain:
        Validates and persists authoritative state
```

---

# 7. Aggregate Boundaries

AI interaction must respect aggregate boundaries established by the domain model.

AI must never use model output as a reason to modify multiple aggregates directly.

If an AI capability determines that several aggregates should change, the interaction must be represented as an application-level workflow.

Example:

```text
AI identifies possible duplicate items
        |
        v
DuplicateCandidateDetected
        |
        v
Application workflow
        |
        +--> Validate source item
        +--> Validate target item
        +--> Validate relationship
        +--> Execute domain command(s)
        |
        v
Domain state
```

AI itself does not coordinate aggregate mutation.

---

# 8. Application Layer as AI Boundary

The application layer is the primary boundary between AI capabilities and the domain.

AI should interact with the domain through application-level contracts whenever an operation affects business behavior.

Examples:

```text
GetCollectionContext
GetItemContext
SearchCollection
SuggestItemMetadata
ClassifyItem
CreateItem
UpdateItem
AssociateItemWithCollection
RemoveItemFromCollection
```

The exact use cases are defined by the application architecture.

The important rule is:

> AI must consume and invoke capabilities rather than directly manipulate domain internals.

---

# 9. Read Interaction

AI may require broad access to domain information for contextual reasoning.

This should preferably occur through dedicated read-oriented contracts.

Conceptually:

```text
AI Context Request
        |
        v
Application Query
        |
        v
Read Model / Domain Query
        |
        v
Context DTO
        |
        v
AI
```

The AI layer should not receive domain entities as its primary public contract.

Instead, it should receive stable context representations designed for the intended AI capability.

Example:

```text
ItemContext
CollectionContext
CollectionSearchContext
UserCollectionContext
DomainEventContext
```

These are contextual representations rather than replacements for domain entities.

---

# 10. Write Interaction

Any AI-driven write must follow the normal application/domain path.

```text
AI
 |
 v
Proposed Action
 |
 v
Application Use Case
 |
 v
Authorization
 |
 v
Validation
 |
 v
Domain Command
 |
 v
Aggregate
 |
 v
Domain Events
 |
 v
Persistence
```

AI cannot execute:

```text
AI -> DbContext -> SaveChanges()
```

for authoritative domain state.

This rule is absolute unless an explicitly documented infrastructure operation concerns AI-owned technical data rather than business state.

---

# 11. AI-Generated Domain Commands

AI may generate a candidate command representation.

For example:

```text
AI output:

{
    "action": "UpdateItemMetadata",
    "itemId": "...",
    "metadata": {
        "author": "...",
        "publisher": "..."
    }
}
```

This output is not automatically a valid command.

The application must:

1. Parse the output.
2. Validate the structure.
3. Validate authorization.
4. Resolve domain identity.
5. Validate business rules.
6. Execute the appropriate application use case.
7. Allow the domain to enforce invariants.
8. Persist only if the operation is valid.

The model output is therefore an **input candidate**, not an authoritative instruction.

---

# 12. Domain Services and AI

AI must not duplicate domain services.

If a business rule exists in a domain service, the AI layer must invoke the relevant application/domain capability rather than implement an approximate version of the same rule in prompts.

Incorrect:

```text
Prompt:
"If collection has more than 100 items,
the user may perform X."
```

if that is an actual business rule.

Correct:

```text
AI requests operation X
        |
        v
Application
        |
        v
Domain Service
        |
        v
Business rule validation
```

Prompts may contain contextual explanations, but those explanations are not authoritative business logic.

---

# 13. Domain Specifications and AI

Domain specifications may be used by application services and potentially exposed indirectly to AI through approved contextual representations.

However, AI must not reconstruct specifications from natural language and assume that the reconstruction is equivalent to the actual domain specification.

For example:

```text
Actual:
DomainSpecification.CanAssociate(item, collection)
```

must remain authoritative.

AI may generate:

```text
"This association appears valid."
```

but this statement has no domain authority.

The application/domain validation remains authoritative.

---

# 14. Domain Events and AI

Domain events provide an appropriate mechanism for many asynchronous AI interactions.

Example:

```text
ItemCreated
    |
    +--> AI enrichment workflow
             |
             +--> retrieve context
             +--> invoke model
             +--> validate output
             +--> create recommendation
```

The original transaction does not need to wait for the AI workflow.

This improves:

- resilience;
- latency;
- provider independence;
- retryability;
- scalability;
- operational isolation.

---

# 15. AI Side Effects

AI side effects must be classified.

## 15.1 Non-Domain Side Effects

Examples:

- storing an embedding;
- updating AI processing status;
- recording model usage;
- storing inference metadata;
- storing evaluation information;
- updating an AI cache.

These may belong to AI infrastructure.

---

## 15.2 Domain Side Effects

Examples:

- changing item metadata;
- creating a collection;
- associating an item;
- deleting an entity;
- changing a business status.

These must pass through the application/domain architecture.

The distinction is:

```text
AI technical state
    -> AI infrastructure

Business state
    -> Application + Domain
```

---

# 16. AI Context Boundary

The AI subsystem must not receive unlimited access to the entire domain.

Context should be explicitly constructed.

Conceptually:

```text
Domain Data
    |
    v
Context Selection
    |
    v
Context Transformation
    |
    v
Context Filtering
    |
    v
AI Context
    |
    v
Model
```

Context construction should consider:

- relevance;
- authorization;
- ownership;
- privacy;
- sensitivity;
- token/resource constraints;
- data minimization;
- tenant/user boundaries;
- temporal validity.

---

# 17. Identity Boundary

Domain identity must remain domain-owned.

AI may reference:

```text
ItemId
CollectionId
UserId
```

or other domain identifiers when required.

AI must not invent authoritative identities.

For generated entities, the application/domain layer must establish the actual identity.

Example:

```text
AI proposes:
"Create item"

Application:
CreateItemCommand

Domain:
ItemId = generated by domain/application mechanism
```

AI-generated IDs must not be treated as authoritative unless explicitly defined as external correlation identifiers.

---

# 18. Authorization Boundary

AI must never be considered an authorization authority.

A user being authorized to ask AI a question does not automatically mean that the AI is authorized to access every piece of domain information.

The system must validate authorization before constructing AI context.

Conceptually:

```text
User
 |
 v
AI Request
 |
 v
Authorization
 |
 +--> Allowed context
 |
 v
AI
```

Not:

```text
User
 |
 v
AI
 |
 v
"Give me whatever data is available"
```

Authorization remains an application/security concern.

---

# 19. Privacy Boundary

AI context must follow the same privacy principles as other application processing.

The AI subsystem must not automatically expose:

- unrelated user information;
- private collection data;
- information belonging to another user;
- internal system data;
- secrets;
- credentials;
- infrastructure configuration;
- unrestricted persistence records.

Context construction must be explicit and purpose-driven.

---

# 20. External AI Provider Boundary

External model providers are infrastructure dependencies.

The domain must not depend directly on:

- OpenAI-specific APIs;
- Anthropic-specific APIs;
- Google-specific APIs;
- Azure-specific model APIs;
- provider-specific SDK types;
- provider-specific response objects.

The architectural boundary should resemble:

```text
AI Application Contract
        |
        v
AI Provider Abstraction
        |
        v
Provider Adapter
        |
        v
External Model Provider
```

This preserves provider independence and prevents external AI technology from leaking into the domain model.

---

# 21. AI Provider Responses

Provider responses must be transformed before being used by the application.

Example:

```text
External Provider Response
        |
        v
Provider Adapter
        |
        v
Normalized AI Result
        |
        v
AI Application Capability
        |
        v
Application Contract
```

The application should not depend on provider-specific response structures.

---

# 22. Prompt Boundary

Prompts are part of the AI subsystem.

They must not become a hidden location for authoritative domain logic.

Prompts may contain:

- contextual information;
- behavioral instructions;
- output requirements;
- classification guidance;
- examples;
- interpretation guidance;
- safety constraints.

Prompts must not be considered authoritative implementations of:

- invariants;
- authorization rules;
- aggregate rules;
- persistence constraints;
- business permissions.

---

# 23. Tool Use Boundary

AI agents may use tools to obtain information or perform actions.

Tool definitions must expose application-level capabilities rather than infrastructure internals.

Preferred:

```text
search_collection()
get_item()
suggest_item_metadata()
create_item()
update_item()
```

Not:

```text
execute_sql()
query_table()
update_row()
delete_row()
```

The latter would bypass architectural boundaries.

---

# 24. Agent Boundary

An AI agent may orchestrate reasoning and tool invocation, but it does not become a replacement for the application's orchestration layer.

The distinction is:

```text
AI Agent
    = adaptive reasoning and tool selection

Application Workflow
    = deterministic business workflow

Domain
    = business truth and invariants
```

Where deterministic business behavior is required, the application workflow remains authoritative.

---

# 25. Human Approval Boundary

For operations with meaningful business consequences, the AI subsystem may produce a proposal requiring explicit approval.

Conceptually:

```text
AI
 |
 v
Proposal
 |
 v
Validation
 |
 v
Approval
 |
 v
Application Command
 |
 v
Domain
```

This is especially appropriate for:

- destructive operations;
- bulk changes;
- ambiguous classifications;
- high-impact metadata changes;
- cross-aggregate modifications;
- externally visible actions.

The exact approval policy belongs to the application and product architecture.

---

# 26. Confidence and Uncertainty

AI output may contain uncertainty.

The domain must not treat model confidence as a replacement for domain validation.

For example:

```text
AI:
confidence = 0.94
```

does not mean:

```text
domain validity = true
```

Confidence is an AI signal.

Domain validity is determined by domain rules.

---

# 27. Recommendations as First-Class AI Results

Where useful, AI recommendations should be represented explicitly rather than silently mutating domain state.

Conceptually:

```text
AIRecommendation
----------------
RecommendationId
TargetEntityId
RecommendationType
ProposedValue
ReasoningMetadata
Confidence
CreatedAt
Status
```

The exact model and persistence strategy are defined by later AI architecture and persistence documents.

The important architectural distinction is:

```text
Recommendation != Domain State
```

---

# 28. AI Persistence Boundary

AI may require persistence for:

- prompts;
- inference records;
- model metadata;
- execution state;
- job state;
- embeddings;
- retrieval indexes;
- recommendations;
- evaluation results;
- audit information.

This data is AI-owned unless explicitly classified otherwise.

AI persistence must not redefine the domain persistence model.

The preferred conceptual separation is:

```text
Domain Persistence
        |
        +--> authoritative business state

AI Persistence
        |
        +--> AI technical and derived state
```

---

# 29. Vector and Semantic Storage Boundary

Vector representations are derived artifacts.

An embedding does not become the authoritative representation of the entity from which it was generated.

For example:

```text
Item
 |
 +--> authoritative domain representation
 |
 +--> textual representation
 |
 +--> embedding
```

The embedding may be regenerated when required.

This means vector storage must be treated as a derived projection or AI-owned representation rather than as a replacement for domain persistence.

---

# 30. Search Boundary

AI-powered semantic search must complement rather than replace domain authorization and filtering.

Correct flow:

```text
Search Request
    |
    v
Authorization / Scope
    |
    v
Semantic Retrieval
    |
    v
Candidate Results
    |
    v
Domain/Application Filtering
    |
    v
Final Results
```

Semantic similarity alone must not determine whether a user may see an entity.

---

# 31. Consistency Model

AI-derived information may be eventually consistent.

The architecture therefore allows:

```text
Domain State
    |
    v
Domain Event
    |
    v
AI Processing
    |
    v
AI Derived State
```

Temporary divergence between domain state and AI-derived state is acceptable where the capability does not require synchronous consistency.

Examples:

```text
Item updated
    |
    +--> embedding temporarily stale
    +--> AI summary temporarily stale
    +--> recommendation temporarily based on previous state
```

The system must provide appropriate invalidation, regeneration, or retry mechanisms.

---

# 32. AI Staleness

AI-derived context must have a defined relationship to the source domain state.

Where stale information can materially affect behavior, the system should track enough information to detect it.

Conceptually:

```text
SourceVersion
AIRepresentationVersion
```

or equivalent metadata may be used.

The implementation strategy is defined in later AI data lifecycle and persistence documents.

---

# 33. Failure Boundary

AI failure must not corrupt domain state.

Examples of AI failures:

- provider unavailable;
- timeout;
- invalid model response;
- malformed structured output;
- content filtering;
- token limit;
- rate limit;
- network failure;
- provider configuration error.

Preferred behavior:

```text
Domain transaction
       |
       +--> succeeds independently

AI processing
       |
       +--> fails/retries/re-enters queue
```

unless a synchronous AI operation has been explicitly classified as mandatory for the application use case.

---

# 34. Transaction Boundary

AI provider calls should not normally participate in the same database transaction as domain state changes.

Avoid:

```text
BEGIN TRANSACTION
    |
    +--> Domain mutation
    |
    +--> External AI call
    |
    +--> Save
COMMIT
```

because external calls can be slow, unavailable, or non-transactional.

Prefer:

```text
BEGIN TRANSACTION
    |
    +--> Domain mutation
    +--> Domain event/outbox
COMMIT

Later:
    |
    v
AI processing
```

This boundary improves resilience and transactional integrity.

---

# 35. Outbox and Event-Driven AI Interaction

Where AI processing is triggered by domain changes, the architecture should support reliable event publication.

Conceptually:

```text
Domain Transaction
       |
       +--> Business State
       |
       +--> Outbox Event
       |
       v
Commit
       |
       v
Event Publication
       |
       v
AI Processing
```

The exact outbox implementation belongs to infrastructure architecture.

The architectural principle remains:

> AI processing must not create an unreliable dependency inside the authoritative domain transaction.

---

# 36. Bulk AI Operations

Bulk AI operations must be handled as controlled application workflows.

Example:

```text
AI:
"These 500 items should be reclassified."

Incorrect:
AI -> direct bulk database update

Correct:
AI -> 500 proposals
       |
       v
Application workflow
       |
       v
Validation
       |
       v
Domain commands
```

Bulk operations may require batching, rate limiting, approval, progress tracking, retries, and partial-failure handling.

---

# 37. Cross-Aggregate AI Decisions

AI may identify relationships between aggregates.

For example:

```text
Item A
   |
   +--> possible duplicate --> Item B
```

AI may generate the candidate relationship.

However, establishing the relationship must use the domain's approved application/domain operation.

AI cannot create an implicit cross-aggregate invariant.

---

# 38. AI and Domain Events: Direction of Dependency

The preferred dependency direction is:

```text
Domain
   |
   v
Domain Event
   |
   v
AI Consumer
```

The domain must not depend on an AI consumer.

Avoid:

```text
Domain
   |
   v
AI Service
   |
   v
Domain completion
```

unless the specific business operation has been explicitly designed as a synchronous AI-dependent use case.

The default architecture is domain-first and AI-consumer-oriented.

---

# 39. AI Context from Domain Events

When consuming an event, AI should not assume that the event contains everything required for reasoning.

The event may contain:

```text
EntityId
EventType
OccurredAt
Version
```

The AI workflow may then retrieve the required authorized context.

Conceptually:

```text
Domain Event
    |
    v
AI Workflow
    |
    v
Authorized Context Query
    |
    v
AI
```

This reduces excessive event payloads and allows context construction to remain purpose-specific.

---

# 40. Preventing AI Leakage into the Domain

The following concepts should not be introduced into core domain entities merely because they are useful to AI:

- Prompt;
- TokenCount;
- ModelName;
- Embedding;
- Vector;
- Temperature;
- Completion;
- ToolCall;
- AgentRun;
- ProviderResponse;
- ModelConfidence.

These are AI/infrastructure concerns.

If the business explicitly requires an AI-derived concept, it must be modeled deliberately as a business concept rather than leaking technical AI terminology into the domain.

---

# 41. Domain Model Protection

The domain model must remain usable without AI.

This is an important architectural test.

The following should remain conceptually possible:

```text
Application
    |
    v
Domain
    |
    v
Persistence
```

without:

```text
AI
```

being present.

Therefore:

> CollectionHub must not become structurally dependent on AI for basic domain correctness.

AI is an optional capability for the domain architecture unless a future product decision explicitly makes a specific AI capability mandatory.

---

# 42. AI Capability Isolation

AI capabilities should be independently replaceable.

Replacing:

```text
Provider A
```

with:

```text
Provider B
```

must not require modifications to:

- domain aggregates;
- domain entities;
- domain services;
- domain specifications;
- domain events;
- persistence model.

Similarly, disabling an AI capability should not invalidate the core domain model.

---

# 43. Boundary Rules

The following rules are normative.

### Rule 1 — Domain authority

The domain is authoritative for business truth.

### Rule 2 — Application mediation

AI-driven business operations pass through application use cases.

### Rule 3 — No direct persistence mutation

AI cannot directly mutate authoritative domain persistence.

### Rule 4 — No invariant duplication

AI prompts and agents must not become alternate implementations of domain invariants.

### Rule 5 — Explicit context

AI receives only explicitly constructed and authorized context.

### Rule 6 — Provider isolation

External AI providers remain behind infrastructure adapters.

### Rule 7 — Derived data distinction

AI-derived information must remain distinguishable from authoritative domain state.

### Rule 8 — Event independence

Domain operations should not depend on asynchronous AI success unless explicitly designed otherwise.

### Rule 9 — Authorization remains authoritative

AI cannot grant itself access to information or operations.

### Rule 10 — Identity remains domain-owned

AI cannot create authoritative domain identities.

### Rule 11 — Agent containment

Agents may reason and orchestrate tools but cannot bypass application/domain boundaries.

### Rule 12 — Domain independence

The domain must remain structurally valid without AI.

---

# 44. Allowed Interaction Matrix

| AI Operation | Domain Read | Application Use Case | Domain Mutation | Direct Persistence |
|---|---:|---:|---:|---:|
| Summarize collection | Yes | Preferred | No | No |
| Classify item | Yes | Preferred | No | No |
| Suggest metadata | Yes | Preferred | No | No |
| Generate recommendation | Yes | Preferred | AI-owned state only | No |
| Create item from AI proposal | Yes | Required | Through application/domain | No |
| Update item from AI proposal | Yes | Required | Through application/domain | No |
| Associate items | Yes | Required | Through application/domain | No |
| Semantic search | Yes | Required for authorization/scope | No | AI index only |
| Generate embeddings | Yes | AI capability | No | AI-owned storage allowed |
| Consume domain event | Event input | AI workflow | Not directly | AI-owned storage allowed |
| Execute destructive action | Yes | Required + approval where applicable | Through domain | No |

---

# 45. Forbidden Interaction Matrix

| Forbidden Pattern | Reason |
|---|---|
| AI -> DbContext -> business update | Bypasses domain |
| AI -> SQL -> business mutation | Bypasses application/domain |
| Prompt contains authoritative invariant | Duplicates business logic |
| Model confidence used as authorization | AI cannot authorize |
| Model output persisted as truth without validation | Unvalidated business state |
| Provider SDK referenced by domain | Infrastructure leakage |
| Embeddings stored as authoritative identity | Derived data confusion |
| AI agent directly invokes repository | Boundary violation |
| AI modifies aggregate internals | Aggregate encapsulation violation |
| AI decides cross-aggregate consistency | Domain responsibility violation |

---

# 46. Synchronous vs Asynchronous Interaction

The default interaction mode should be selected according to business necessity.

## Synchronous

Appropriate when:

- the user is waiting for an AI response;
- no long-running processing is required;
- the operation is advisory;
- immediate feedback is valuable.

Example:

```text
User
 -> AI
 -> Context
 -> Model
 -> Response
```

## Asynchronous

Preferred when:

- processing is expensive;
- domain events trigger AI enrichment;
- bulk processing is involved;
- embeddings must be generated;
- retries are expected;
- the user does not need immediate completion.

Example:

```text
Domain Event
 -> Queue
 -> AI Job
 -> Model
 -> AI Persistence
```

---

# 47. Interaction with Application Workflows

AI may participate in application workflows as:

- initiator;
- advisor;
- classifier;
- enrichment provider;
- proposal generator;
- asynchronous processor.

It must not silently replace workflow semantics.

For example:

```text
User:
"Organize my collection."

AI:
Generate organization proposal.

Application:
Present proposal / execute approved workflow.

Domain:
Apply valid state changes.
```

The workflow remains an application concern.

---

# 48. Interaction with Infrastructure

AI infrastructure is responsible for technical concerns such as:

- model provider clients;
- retries;
- rate limits;
- provider authentication;
- serialization;
- token accounting;
- embeddings;
- vector storage;
- AI job execution;
- provider-specific configuration.

These concerns must remain outside the domain model.

---

# 49. Architectural Dependency Rules

The intended dependency structure is:

```text
Domain
  ^
  |
Application
  ^
  |
AI Application Capabilities
  ^
  |
AI Infrastructure / Providers
```

In dependency terms:

```text
Domain
  <- Application
  <- AI Application
  <- Infrastructure
```

The domain must not depend on:

```text
AI
Provider SDK
LLM
Prompt Framework
Vector Database
Agent Framework
```

---

# 50. Context Contracts

AI should consume explicit context contracts.

A conceptual contract may look like:

```text
AIItemContext
-------------------------
ItemId
Name
Description
RelevantMetadata
CollectionContext
DomainVersion
```

The exact contract is defined in later AI architecture documents.

Context contracts should be:

- purpose-specific;
- stable;
- minimal;
- authorization-aware;
- provider-independent;
- detached from persistence entities.

---

# 51. Output Contracts

AI output must also be treated as a contract.

Conceptually:

```text
AIResult
-------------------------
Capability
Status
StructuredOutput
Confidence
Evidence
Warnings
Metadata
```

The exact output contract is defined in:

`15_AI_OUTPUT_CONTRACTS_AND_STRUCTURED_RESPONSES.md`

The important boundary is that raw model responses must not cross directly into domain logic.

---

# 52. Validation Boundary

AI outputs must be validated at multiple levels.

```text
Raw Model Output
       |
       v
Structural Validation
       |
       v
Semantic AI Validation
       |
       v
Application Validation
       |
       v
Domain Validation
```

Each layer has a different responsibility.

AI validation checks whether the response conforms to the expected AI contract.

Application validation checks whether the requested operation is allowed within the use case.

Domain validation checks business invariants.

---

# 53. Error Propagation

AI errors should be translated into application-level errors or statuses.

Provider-specific errors must not leak into the domain.

For example:

```text
OpenAI RateLimitError
```

must not become a domain concept.

Instead:

```text
AIExecutionUnavailable
AIExecutionThrottled
AIExecutionFailed
AIOutputInvalid
```

may be represented at the AI/application boundary as appropriate.

---

# 54. Observability Boundary

AI-specific telemetry belongs to AI/application infrastructure.

Examples:

- model used;
- latency;
- token usage;
- provider;
- retry count;
- execution status;
- tool calls;
- prompt version;
- evaluation score.

These values must not be added to domain entities merely for observability.

Correlation identifiers may cross boundaries where required for traceability.

---

# 55. Audit Boundary

Business audit information belongs to the application/domain architecture.

AI execution audit information belongs to AI infrastructure.

Where both are required, they should be correlated rather than merged conceptually.

Example:

```text
Domain Action
    |
    +--> Business Audit Record

AI Execution
    |
    +--> AI Execution Record

CorrelationId
    |
    +--> connects both when appropriate
```

---

# 56. Security Boundary

The AI subsystem must be treated as an untrusted processing component with respect to authoritative business state.

This does not imply that AI is inherently malicious. It means model output must be treated as untrusted input.

Therefore:

```text
AI Output
    |
    v
Validate
    |
    v
Authorize
    |
    v
Apply domain rules
```

must be the default architecture.

---

# 57. Prompt Injection Boundary

Domain information retrieved for AI processing may contain user-generated content.

Such content must not automatically be interpreted as system instructions.

Conceptually:

```text
System Instructions
        |
        +--> trusted

Domain/User Content
        |
        +--> untrusted context
```

The AI architecture must maintain this distinction.

The detailed implementation belongs to the AI validation, guardrails, security, and privacy documentation.

---

# 58. External Content Boundary

If AI processes external information, that information must not automatically become trusted domain information.

Example:

```text
External Source
    |
    v
AI extraction
    |
    v
Candidate data
    |
    v
Application validation
    |
    v
Domain acceptance
```

External content remains untrusted until validated according to the relevant business process.

---

# 59. AI as a Domain Consumer

The preferred mental model is:

```text
                    +----------------+
                    |     DOMAIN     |
                    | authoritative  |
                    +-------+--------+
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
        Application     Events       Read Models
             |              |              |
             +--------------+--------------+
                            |
                            v
                    +---------------+
                    |      AI       |
                    | interpretation|
                    | recommendation|
                    +---------------+
```

AI consumes domain capabilities and representations.

It does not become another domain.

---

# 60. AI as a Domain Initiator

AI can initiate application behavior, but only through controlled boundaries.

```text
AI
 |
 v
Candidate Action
 |
 v
Application
 |
 +--> Authorization
 |
 +--> Validation
 |
 v
Domain
 |
 v
State Change
```

This allows AI to be an active participant without granting it domain authority.

---

# 61. Boundary Decision Table

| Question | Decision |
|---|---|
| Can AI read domain data? | Yes, through authorized application/context mechanisms |
| Can AI directly read database tables? | No |
| Can AI propose domain changes? | Yes |
| Can AI directly apply domain changes? | No |
| Can AI invoke application use cases? | Yes, through approved contracts/tools |
| Can AI bypass authorization? | No |
| Can AI define domain rules? | No |
| Can AI consume domain events? | Yes |
| Can AI own AI-specific persistence? | Yes |
| Can AI own business persistence? | No |
| Can domain depend on AI? | Not by default |
| Can infrastructure depend on AI providers? | Yes |
| Can domain entities contain AI technical concepts? | No, unless explicitly justified as business concepts |
| Can AI-generated values become domain state? | Only after normal application/domain validation |

---

# 62. Implementation Consequences

The architecture requires the implementation to provide clear interfaces between:

```text
Application
AI Application
Domain
AI Infrastructure
Infrastructure
```

The implementation should avoid direct references from AI components to:

- EF Core;
- repositories intended for domain internals;
- database contexts;
- provider SDKs from application/domain code;
- aggregate internals.

Instead, explicit interfaces/contracts should be introduced where required.

---

# 63. Testing Consequences

The boundary architecture implies separate test categories.

## Domain Tests

Must verify domain behavior without AI.

## Application Tests

Must verify AI-triggered use cases without depending on an actual model provider.

## AI Capability Tests

Must verify:

- context construction;
- model interaction;
- structured output;
- validation;
- failure handling.

## Integration Tests

Must verify:

```text
AI -> Application -> Domain
```

without treating provider-specific behavior as domain behavior.

## Contract Tests

Must verify provider adapters against normalized AI contracts.

---

# 64. Evolution Rules

Future AI capabilities must follow the same boundaries.

Adding:

```text
New Model
New Provider
New Agent
New Embedding Strategy
New AI Tool
New Retrieval System
```

must not require redefining domain invariants.

Adding a new domain capability may require a new AI context or tool, but the inverse should not normally occur.

---

# 65. Architectural Invariants

The following invariants are established for the AI/domain boundary.

### AI-BOUNDARY-001

AI is not the owner of authoritative domain truth.

### AI-BOUNDARY-002

Authoritative business state can only be changed through approved application/domain paths.

### AI-BOUNDARY-003

AI output is untrusted input until validated.

### AI-BOUNDARY-004

AI cannot bypass application authorization.

### AI-BOUNDARY-005

AI cannot bypass aggregate invariants.

### AI-BOUNDARY-006

AI technical concepts do not leak into the domain model.

### AI-BOUNDARY-007

External AI providers are isolated behind infrastructure boundaries.

### AI-BOUNDARY-008

AI-derived data is distinguishable from authoritative domain data.

### AI-BOUNDARY-009

The core domain remains structurally independent from AI.

### AI-BOUNDARY-010

Asynchronous AI processing must not compromise domain transactional integrity.

---

# 66. Traceability

This document establishes the boundary foundation for the remaining AI architecture.

| Concern | Defined Here | Detailed Later |
|---|---|---|
| AI/domain relationship | Yes | — |
| Domain ownership | Yes | — |
| Application boundary | Yes | 05 |
| AI architecture | Boundary only | 06 |
| Provider strategy | Boundary only | 07 |
| Prompt architecture | Boundary only | 08 |
| Context architecture | Boundary only | 09 |
| Data lifecycle | Boundary only | 10 |
| Memory/state | Boundary only | 11 |
| Tool use | Boundary only | 12 |
| Orchestration | Boundary only | 13 |
| Async processing | Boundary only | 14 |
| Output contracts | Boundary only | 15 |
| Validation/guardrails | Boundary only | 16 |
| Security/privacy | Boundary only | 17 |
| Observability | Boundary only | 18 |
| Cost/latency | Boundary only | 19 |
| Reliability | Boundary only | 20 |
| Evaluation | — | 21 |
| Testing | Boundary implications | 22 |
| Runtime integration | Boundary implications | 23 |
| External integrations | Boundary implications | 24 |
| Persistence | Boundary implications | 25 |

---

# 67. Open Decisions Deferred

The following decisions are intentionally deferred to subsequent AI architecture documents:

1. Exact AI application component structure.
2. Exact model/provider abstraction.
3. Prompt versioning strategy.
4. Context construction implementation.
5. AI memory architecture.
6. Tool contract definitions.
7. Agent orchestration model.
8. Asynchronous job infrastructure.
9. AI output schemas.
10. Guardrail implementation.
11. AI-specific security controls.
12. AI observability model.
13. Cost and token governance.
14. AI persistence model.
15. Evaluation methodology.

These decisions must remain consistent with the boundaries established here.

---

# 68. Final Boundary Statement

The CollectionHub AI architecture is based on a strict separation of responsibilities:

```text
DOMAIN
    Owns business truth.

APPLICATION
    Owns use-case execution and authorization boundaries.

AI
    Owns interpretation, generation, recommendation,
    semantic processing and AI-specific behavior.

AI INFRASTRUCTURE
    Owns models, providers, prompts, embeddings,
    execution mechanisms and AI technical state.

PERSISTENCE / INFRASTRUCTURE
    Owns technical persistence and external infrastructure.
```

The central architectural rule is:

> **AI may reason about the domain and propose actions on behalf of the user, but only the application and domain architecture may establish authoritative business state.**

This boundary is mandatory for all subsequent AI architecture decisions in CollectionHub.

Any future AI capability that cannot respect these boundaries must be treated as an explicit architectural exception and documented as such before implementation.