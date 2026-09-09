# 05 — AI Application Integration

## 1. Purpose

This document defines how the AI subsystem integrates with the CollectionHub application layer.

Its purpose is to establish the architectural boundary between AI capabilities and the application runtime, defining:

- how application use cases invoke AI capabilities;
- how AI capabilities return results to the application layer;
- which responsibilities belong to the application layer and which belong to AI;
- how AI participates in application workflows;
- how application commands, queries, orchestration and domain interaction remain controlled;
- how AI failures affect application execution;
- how AI integration remains replaceable, testable and observable;
- how AI-specific concerns are prevented from leaking into the domain model.

This document does **not** define the internal implementation of AI providers, prompts, model selection, memory, orchestration engines or AI persistence. Those concerns are defined by subsequent AI architecture documents.

---

## 2. Architectural Context

CollectionHub follows a layered architecture in which the application layer coordinates business workflows while the domain layer owns business rules and invariants.

AI is treated as an application capability and integration concern.

The architectural relationship is therefore:

```text
User / External Trigger
        |
        v
Application Entry Point
        |
        v
Application Use Case
        |
        +----------------------+
        |                      |
        v                      v
Domain Operations         AI Capability
        |                      |
        v                      v
Domain Model              AI Runtime
        |                      |
        |                 Model / Provider
        |                      |
        +----------+-----------+
                   |
                   v
            Application Result
```

The application layer remains the owner of the business workflow.

AI does not become the owner of application orchestration merely because it participates in the workflow.

---

## 3. Core Architectural Principle

The primary rule is:

> **AI assists application workflows; AI does not own the application's business workflow.**

An application use case may invoke an AI capability when AI provides value, but the application layer remains responsible for:

- workflow sequencing;
- authorization;
- business operation boundaries;
- transaction boundaries;
- validation of application commands;
- domain interaction;
- persistence decisions;
- error propagation;
- application-level consistency;
- final acceptance of AI-generated information.

AI output is therefore considered an input to an application workflow rather than an authoritative application command.

---

## 4. AI Integration Boundary

The integration boundary is defined as:

```text
Application Layer
        |
        | AI Application Contract
        v
AI Integration Boundary
        |
        v
AI Subsystem
        |
        +--> Context
        +--> Instructions
        +--> Orchestration
        +--> Model
        +--> Provider
        +--> Validation
```

The application layer must not depend directly on:

- model SDKs;
- provider SDKs;
- provider-specific request objects;
- provider-specific response objects;
- prompt templates;
- model-specific configuration;
- provider-specific error types;
- AI infrastructure implementation details.

Instead, the application layer depends on application-facing AI contracts.

---

## 5. Application Responsibilities

The application layer owns the following responsibilities.

### 5.1 Workflow Coordination

Application use cases determine:

- when AI should be invoked;
- why AI is being invoked;
- what application operation is currently executing;
- what information can be provided to AI;
- what happens after AI returns;
- whether the AI result is optional or mandatory;
- whether the workflow continues after an AI failure.

### 5.2 Input Preparation

The application layer prepares an AI request using information that is valid for the current application context.

It must not expose unrestricted application state.

The request should contain only the information required by the AI capability.

### 5.3 Authorization

Authorization remains an application concern.

AI must never be considered an authorization mechanism.

An AI response cannot grant:

- access;
- permissions;
- ownership;
- role elevation;
- visibility of restricted data;
- execution rights.

### 5.4 Result Acceptance

The application layer determines whether an AI result is acceptable for the current workflow.

Where required, the result must pass:

- structural validation;
- application validation;
- domain validation;
- business rule validation.

### 5.5 Failure Handling

The application layer determines whether an AI failure:

- aborts the workflow;
- produces a degraded result;
- schedules a retry;
- creates an asynchronous job;
- is logged and ignored;
- requires user intervention.

---

## 6. AI Responsibilities

The AI subsystem owns AI-specific concerns.

These include:

- AI request execution;
- prompt construction;
- model interaction;
- provider abstraction;
- context assembly;
- AI-specific validation;
- AI-specific guardrails;
- token/resource management;
- AI execution telemetry;
- model and provider selection;
- AI orchestration where explicitly delegated.

The AI subsystem does not own application business rules.

---

## 7. Application-to-AI Interaction Contract

Application-to-AI communication must use explicit contracts.

A conceptual request should contain information such as:

```text
AI Request
├── Capability
├── Operation Context
├── Input
├── Context References
├── Constraints
├── Execution Options
└── Correlation Information
```

A conceptual response should contain:

```text
AI Response
├── Status
├── Result
├── Confidence / Quality Metadata
├── Validation State
├── Execution Metadata
└── Failure Information
```

The concrete contract will be defined in:

`15_AI_OUTPUT_CONTRACTS_AND_STRUCTURED_RESPONSES.md`

---

## 8. AI Capability Invocation

Application use cases should invoke named capabilities rather than models.

Preferred:

```text
Application Use Case
        |
        v
AI Capability
        |
        v
AI Runtime
```

Avoid:

```text
Application Use Case
        |
        v
GPT Provider SDK
```

The application should express intent at the capability level.

Examples include conceptual capabilities such as:

- classify collection item;
- suggest metadata;
- extract structured information;
- generate descriptive text;
- detect possible duplicates;
- enrich item information;
- assist with search;
- summarize collection information.

The exact capability catalogue is governed by the AI capability inventory.

---

## 9. Synchronous Integration

AI may participate synchronously when the result is required immediately by the application workflow.

Conceptual flow:

```text
Application Use Case
        |
        v
Build AI Request
        |
        v
Invoke AI Capability
        |
        v
Validate AI Response
        |
        v
Continue Application Workflow
```

Synchronous AI operations must have explicit:

- timeout policies;
- cancellation behavior;
- resource limits;
- failure handling;
- observability;
- retry rules.

A synchronous AI call must not implicitly create an unbounded application execution dependency.

---

## 10. Asynchronous Integration

AI should be executed asynchronously when:

- processing may be long-running;
- the operation does not require an immediate response;
- batch enrichment is required;
- external provider latency is unpredictable;
- retryability is important;
- processing may consume substantial resources;
- the operation is suitable for background execution.

Conceptual flow:

```text
Application Use Case
        |
        v
Create AI Processing Request
        |
        v
Persist / Publish Work
        |
        v
Background AI Processing
        |
        v
Validate Result
        |
        v
Apply Result Through Application Boundary
```

Asynchronous processing must not bypass application and domain boundaries.

---

## 11. AI Results Are Not Domain Commands

A critical architectural rule is:

> **An AI response must not be treated as an implicit domain command.**

For example:

```text
AI
 |
 | "This item appears to belong to category X"
 v
Application
 |
 | validates and decides
 v
Domain operation
```

Not:

```text
AI
 |
 | directly modifies domain state
 v
Database
```

AI can propose information.

The application decides whether that information becomes part of an application operation.

The domain decides whether the resulting operation is valid according to domain invariants.

---

## 12. Domain Boundary Preservation

The AI subsystem must not directly manipulate domain entities.

AI must not:

- mutate aggregates;
- bypass domain methods;
- modify domain state directly;
- bypass domain invariants;
- write directly to domain persistence;
- introduce AI-specific infrastructure into domain entities.

The application layer translates AI results into valid application operations.

Example:

```text
AI Result
    |
    v
Application Validation
    |
    v
Application Command
    |
    v
Domain Operation
    |
    v
Domain Invariants
```

---

## 13. AI Integration with Commands

An application command may use AI as an assisting capability.

Conceptual example:

```text
CreateCollectionItem
        |
        +--> Validate command
        |
        +--> Create domain object
        |
        +--> Optional AI enrichment
        |
        +--> Apply validated enrichment
        |
        +--> Persist
```

The exact ordering depends on the use case.

AI should not be inserted into a command merely because it is technically possible.

Its participation must be justified by an explicit application requirement.

---

## 14. AI Integration with Queries

AI may also support query-oriented application workflows.

Examples include:

- semantic search assistance;
- natural-language filtering;
- result summarization;
- query interpretation;
- relevance assistance.

Conceptually:

```text
User Query
    |
    v
Application Query
    |
    +--> Traditional Query Processing
    |
    +--> AI Assistance
    |
    v
Application Result
```

AI must not silently change the semantics of authoritative queries without an explicit contract.

---

## 15. AI-Assisted Application Operations

AI-assisted operations should distinguish between:

### 15.1 Suggestion

AI proposes information.

```text
AI -> Suggestion -> User/Application
```

No domain mutation is implied.

### 15.2 Recommendation

AI proposes an application action.

```text
AI -> Recommendation -> Application
```

The application decides whether to execute it.

### 15.3 Automated Processing

AI produces information that is automatically consumed by an application workflow.

```text
AI -> Structured Result -> Validation -> Application Operation
```

This requires stronger validation and governance.

### 15.4 Autonomous Agentic Execution

Any capability allowing AI to initiate multiple application operations requires an explicit agent boundary.

Such behavior must never be introduced implicitly through ordinary AI integration.

Agent behavior is governed separately by:

`12_AI_TOOL_USE_AND_AGENT_BEHAVIOR.md`

---

## 16. Transaction Boundaries

AI execution must not automatically determine database transaction boundaries.

In general:

```text
Database Transaction
        |
        X
AI External Call
```

should be avoided when the external AI operation may be slow or unpredictable.

A database transaction should not remain open while waiting for an external model provider unless a specific architectural decision explicitly justifies it.

Preferred pattern:

```text
Application
    |
    v
Prepare State
    |
    v
Commit / Persist
    |
    v
AI Processing
    |
    v
Validate
    |
    v
Apply Follow-up Operation
```

The exact strategy depends on consistency requirements and will be formalized by the persistence and asynchronous processing architecture.

---

## 17. Idempotency

AI-integrated application operations must consider idempotency.

Repeated execution may occur because of:

- retries;
- provider failures;
- network failures;
- job redelivery;
- user repetition;
- background worker restarts.

Application workflows must therefore distinguish between:

- safe repeated AI inference;
- repeated application mutation;
- repeated persistence;
- repeated external side effects.

AI idempotency does not automatically imply application idempotency.

---

## 18. Retry Responsibility

Retries must be explicitly assigned.

AI infrastructure may retry transient provider failures.

The application layer may retry an application operation when appropriate.

Neither layer should blindly retry the same operation independently.

Conceptually:

```text
Application Retry Policy
        |
        v
AI Capability
        |
        v
AI Infrastructure Retry Policy
        |
        v
Provider
```

Retry multiplication must be prevented.

---

## 19. Timeout and Cancellation

Every AI invocation must have explicit execution constraints.

At minimum:

- timeout;
- cancellation;
- maximum execution duration;
- resource limits;
- retry policy.

Cancellation must propagate through the application-to-AI boundary where technically supported.

A cancelled application operation must not leave uncontrolled AI processing behind.

---

## 20. Error Translation

AI-specific failures must not leak directly into the application layer.

For example, the application should not depend on:

```text
OpenAIException
AnthropicException
ProviderSpecificTimeoutException
```

Instead, AI integration should expose application-neutral failure categories such as:

```text
AIUnavailable
AIExecutionFailed
AIValidationFailed
AIRequestRejected
AIResourceLimitExceeded
AIExecutionCancelled
```

The concrete error taxonomy is defined by the AI reliability and output-contract documents.

---

## 21. Graceful Degradation

AI should be considered an optional dependency whenever the business requirement allows it.

Possible degraded behavior includes:

```text
AI available
    -> enhanced application result

AI unavailable
    -> standard application result
```

Where AI is mandatory for a specific capability, failure must be explicit and observable.

The system must not silently fabricate an AI result when AI execution failed.

---

## 22. Security Boundary

The application layer is responsible for ensuring that AI receives only information that the current workflow is authorized to expose.

AI integration must therefore respect:

- authorization;
- tenant boundaries;
- ownership boundaries;
- sensitive-data policies;
- privacy restrictions;
- data minimization;
- contextual access rules.

The AI subsystem must not be used as a mechanism to bypass application authorization.

---

## 23. Data Minimization

Application-to-AI requests should contain only the minimum information required.

The integration should prefer:

```text
Required Context
```

over:

```text
Entire Aggregate / Entire Database Record
```

Context assembly should be deliberate and traceable.

This is especially important when AI providers are external services.

---

## 24. Observability

Every AI-integrated application operation should be traceable.

Where applicable, telemetry should correlate:

```text
Application Operation
        |
        +--> Correlation ID
        |
        +--> AI Invocation ID
        |
        +--> Provider Request ID
        |
        +--> Result / Failure
```

The application layer should be able to determine:

- which use case invoked AI;
- which capability was requested;
- whether execution succeeded;
- how long it took;
- whether the result was accepted;
- whether the workflow degraded;
- whether a retry occurred.

Detailed AI telemetry is defined in:

`18_AI_OBSERVABILITY_AND_TELEMETRY.md`

---

## 25. Configuration Boundary

Application code must not embed provider-specific AI configuration.

Configuration should be supplied through the application's configuration and dependency-injection mechanisms.

Examples include:

- enabled AI capabilities;
- timeout policies;
- feature flags;
- provider selection;
- model configuration;
- execution limits.

The application should consume capability-level configuration rather than provider SDK configuration.

---

## 26. Dependency Injection

AI capabilities should be injected through application-facing abstractions.

Conceptual structure:

```text
Application Use Case
        |
        v
IAICapability
        |
        v
AI Implementation
        |
        v
Provider Adapter
```

This allows:

- testing;
- provider replacement;
- mock implementations;
- deterministic integration tests;
- controlled configuration.

Application use cases should not instantiate AI providers directly.

---

## 27. Testing Boundary

Application tests must be able to test application behavior without requiring a real external AI provider.

Therefore the architecture should support:

```text
Application Test
       |
       v
Fake / Mock AI Capability
```

Integration tests may additionally verify:

```text
Application
    |
    v
Real AI Integration
    |
    v
Provider
```

These are separate test concerns.

---

## 28. Determinism and Reproducibility

AI outputs may vary.

Application behavior must therefore not rely on uncontrolled natural-language variation.

Where an AI result affects application logic, the result should be converted into a structured representation with explicit validation.

Preferred:

```text
AI
 |
 v
Structured Result
 |
 v
Validation
 |
 v
Application Logic
```

Avoid:

```text
AI
 |
 v
Free-form Text
 |
 v
Implicit Application Decision
```

---

## 29. Human-in-the-Loop Boundary

Where AI output requires human confirmation, the application layer owns the human interaction workflow.

Example:

```text
AI Suggestion
      |
      v
Application Review State
      |
      v
User Confirmation
      |
      v
Application Command
      |
      v
Domain Operation
```

AI does not bypass user confirmation when the application requires explicit approval.

---

## 30. Feature Enablement

AI capabilities should be independently enableable where practical.

The application should support capability-level control such as:

```text
AI Capability
    |
    +--> Enabled
    +--> Disabled
    +--> Degraded
    +--> Restricted
```

Feature enablement must not require recompiling application business logic.

---

## 31. Application Workflow Ownership Matrix

| Responsibility | Application | AI |
|---|---:|---:|
| Use-case orchestration | YES | NO |
| Domain rule enforcement | YES / Domain | NO |
| Authorization | YES | NO |
| AI execution | NO | YES |
| Prompt construction | NO | YES |
| Model selection | NO | YES |
| Provider selection | NO | YES |
| AI context assembly | Shared by contract | YES |
| AI result validation | Shared | YES |
| Application result acceptance | YES | NO |
| Persistence decision | YES | NO |
| Domain mutation | Through domain | NO |
| Retry policy | Shared by boundary | Shared by boundary |
| AI telemetry | Shared | YES |
| Application telemetry | YES | Shared |
| AI provider SDK access | NO | YES |

---

## 32. Dependency Direction

The intended dependency direction is:

```text
Application
    |
    v
AI Application Abstraction
    |
    v
AI Implementation
    |
    v
AI Provider Adapter
```

The inverse dependency is prohibited:

```text
AI Provider
    |
    X
Application Business Logic
```

The provider must never dictate application architecture.

---

## 33. Forbidden Integration Patterns

The following patterns are architecturally prohibited.

### 33.1 Direct Provider Usage from Use Cases

```text
Use Case -> Provider SDK
```

### 33.2 Direct AI Persistence

```text
AI -> Database
```

### 33.3 AI-Owned Domain Mutation

```text
AI -> Aggregate
```

### 33.4 AI-Owned Authorization

```text
AI -> Permission Decision
```

### 33.5 Free-Form Output Driving Business Logic

```text
AI Text -> Business Rule
```

### 33.6 Hidden AI Invocation

An application operation must not invoke AI implicitly without its architectural behavior being documented and observable.

### 33.7 Unbounded AI Calls

No application workflow may create uncontrolled recursive or repeated AI execution.

---

## 34. Integration Decision Rules

When introducing AI into an application use case, the following questions must be answered:

1. What business or user problem does AI solve?
2. Is AI mandatory or optional?
3. Is the operation synchronous or asynchronous?
4. What data is supplied to AI?
5. What information is explicitly excluded?
6. What contract does AI return?
7. How is the response validated?
8. Can the operation continue if AI fails?
9. Does the result modify domain state?
10. Which application operation applies that modification?
11. What authorization applies?
12. How is the operation observed?
13. What is the retry strategy?
14. What is the timeout strategy?
15. How is the operation tested without a real AI provider?

No AI capability should be introduced into the application layer without clear answers to these questions.

---

## 35. Reference Interaction Pattern

The canonical integration pattern is:

```text
                 APPLICATION
                     |
                     v
              Application Use Case
                     |
                     v
              AI Capability Contract
                     |
                     v
              AI Application Boundary
                     |
          +----------+----------+
          |                     |
          v                     v
       Context              AI Runtime
                                |
                                v
                         Model / Provider
                                |
                                v
                         Structured Result
                                |
          +---------------------+
          |
          v
       Validation
          |
          v
   Application Decision
          |
          v
     Domain Operation
          |
          v
       Persistence
```

This pattern establishes a controlled path from application intent to AI execution and from AI output back to application and domain processing.

---

## 36. Architectural Invariants

The following invariants are mandatory.

### INV-AI-APP-001

Application use cases must depend on AI abstractions, not provider SDKs.

### INV-AI-APP-002

AI must not bypass application authorization.

### INV-AI-APP-003

AI must not directly mutate domain state.

### INV-AI-APP-004

AI output must be explicitly validated before affecting application state.

### INV-AI-APP-005

AI provider failures must be translated into provider-neutral failure semantics.

### INV-AI-APP-006

AI execution must have explicit timeout and cancellation behavior.

### INV-AI-APP-007

AI invocation must be observable.

### INV-AI-APP-008

Application workflows must define whether AI failure is fatal or degradable.

### INV-AI-APP-009

External AI calls must not implicitly define database transaction boundaries.

### INV-AI-APP-010

AI integration must remain replaceable without changing domain behavior.

---

## 37. Relationship with Other AI Documents

This document depends conceptually on:

- `00_AI_FOUNDATION_AND_SCOPE.md`
- `01_AI_CONCEPT_INVENTORY.md`
- `02_AI_CAPABILITIES_AND_RESPONSIBILITIES.md`
- `03_AI_USE_CASES_AND_USER_INTERACTIONS.md`
- `04_AI_DOMAIN_INTERACTION_AND_BOUNDARIES.md`

It provides the application integration foundation for:

- `06_AI_ARCHITECTURE_AND_COMPONENTS.md`
- `07_AI_MODEL_AND_PROVIDER_STRATEGY.md`
- `08_AI_PROMPTING_AND_INSTRUCTION_ARCHITECTURE.md`
- `09_AI_CONTEXT_AND_KNOWLEDGE_ARCHITECTURE.md`
- `10_AI_DATA_FLOWS_AND_INFORMATION_LIFECYCLE.md`
- `12_AI_TOOL_USE_AND_AGENT_BEHAVIOR.md`
- `13_AI_ORCHESTRATION_AND_WORKFLOW_ARCHITECTURE.md`
- `14_AI_ASYNCHRONOUS_PROCESSING_AND_JOB_ARCHITECTURE.md`

---

## 38. Implementation Boundary

This document establishes architectural rules, not implementation details.

The implementation must provide:

- application-facing AI abstractions;
- dependency injection;
- provider-independent contracts;
- structured AI results;
- explicit validation;
- failure translation;
- timeout and cancellation handling;
- observability;
- test doubles;
- synchronous and asynchronous integration mechanisms where required.

The implementation must not introduce:

- provider dependencies into the domain;
- provider dependencies into ordinary application use cases;
- direct AI-to-database access;
- implicit domain mutation from AI;
- hidden AI execution;
- uncontrolled autonomous behavior.

---

## 39. Final Architectural Position

CollectionHub treats AI as a controlled application capability rather than as a replacement for application architecture.

The application layer remains responsible for workflow ownership, authorization, domain interaction, persistence coordination and final acceptance of AI-generated information.

AI provides specialized capabilities through explicit contracts.

The resulting architecture is:

```text
Application owns the workflow.
        |
        v
AI assists the workflow.
        |
        v
Application validates the result.
        |
        v
Domain enforces business invariants.
        |
        v
Persistence records the accepted state.
```

This boundary preserves architectural integrity while allowing AI capabilities to evolve independently from the application's core business model.

**Status:** Architectural baseline candidate.

**Next document:** `06_AI_ARCHITECTURE_AND_COMPONENTS.md`