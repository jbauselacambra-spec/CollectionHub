# AI Concept Inventory

## 1. Purpose

This document establishes the controlled vocabulary and conceptual inventory for Artificial Intelligence within CollectionHub.

Its purpose is to identify the concepts that will be used throughout the `03_AI` phase and to provide a stable semantic foundation for subsequent documents.

This document does not define the final technical architecture of AI.

It defines **what concepts exist**, how they relate to one another, and which concepts require explicit architectural treatment.

All subsequent AI documents must use these concepts consistently unless a later architectural decision explicitly introduces a refinement.

---

# 2. Scope

The inventory covers AI-related concepts across:

- AI capabilities;
- AI use cases;
- model interaction;
- prompts and instructions;
- context;
- knowledge;
- retrieval;
- memory;
- state;
- tools;
- agents;
- workflows;
- structured outputs;
- validation;
- guardrails;
- safety;
- observability;
- evaluation;
- providers;
- persistence;
- and AI governance.

The inventory also defines the relationship between AI concepts and the existing:

- Domain;
- Application;
- Infrastructure;
- Product;
- and external-system concepts.

---

# 3. Conceptual Principles

The following principles govern the inventory.

## 3.1 Domain concepts remain authoritative

AI concepts must not redefine established domain concepts.

For example:

```text
Domain Entity
Domain Aggregate
Domain Value Object
Domain Event
Domain Invariant
```

remain domain concepts even when AI operates on representations of them.

---

## 3.2 AI concepts represent capabilities, not business truth

An AI classification is not automatically a domain classification.

An AI recommendation is not automatically a domain relationship.

An AI-generated description is not automatically authoritative metadata.

The distinction between AI-derived information and authoritative information must remain explicit.

---

## 3.3 Technical concepts and conceptual concepts are separated

A concept such as:

```text
AI Capability
```

is different from:

```text
OpenAI API
Azure OpenAI
LLM SDK
Embedding Library
```

The first represents an architectural capability.

The second represents a possible implementation technology.

---

# 4. Concept Taxonomy

The CollectionHub AI vocabulary is organized into the following conceptual groups.

```text
AI
│
├── Capabilities
│   ├── AI Capability
│   ├── AI Use Case
│   └── AI Interaction
│
├── Intelligence
│   ├── Model
│   ├── Inference
│   ├── Generation
│   ├── Classification
│   ├── Extraction
│   └── Recommendation
│
├── Context & Knowledge
│   ├── Context
│   ├── Knowledge Source
│   ├── Retrieval
│   ├── Retrieval Result
│   └── Grounding
│
├── Instructions
│   ├── Prompt
│   ├── Instruction
│   ├── System Instruction
│   └── Prompt Template
│
├── State
│   ├── AI State
│   ├── Conversation
│   ├── Session
│   ├── Memory
│   └── Working Context
│
├── Tools & Agents
│   ├── Tool
│   ├── Tool Contract
│   ├── Agent
│   ├── Agent Action
│   └── Tool Result
│
├── Workflow
│   ├── AI Workflow
│   ├── AI Step
│   ├── Orchestration
│   └── Job
│
├── Output & Control
│   ├── AI Output
│   ├── Structured Output
│   ├── Validation
│   ├── Guardrail
│   └── Confidence
│
├── Governance
│   ├── Safety Policy
│   ├── AI Policy
│   ├── AI Boundary
│   └── Human Oversight
│
├── Runtime
│   ├── AI Provider
│   ├── Model Deployment
│   ├── AI Request
│   ├── AI Response
│   └── AI Execution
│
└── Quality
    ├── Evaluation
    ├── Evaluation Dataset
    ├── Evaluation Metric
    ├── AI Test
    └── AI Regression
```

---

# 5. Core AI Concepts

## 5.1 AI Capability

### Definition

An **AI Capability** is a reusable system capability in which AI provides a meaningful function to CollectionHub.

Examples:

- semantic search;
- metadata enrichment;
- natural-language interpretation;
- summarization;
- classification;
- recommendation.

### Characteristics

An AI capability:

- has a defined purpose;
- has defined inputs;
- has defined outputs;
- has defined boundaries;
- has defined quality expectations;
- may be reused by multiple use cases.

### Architectural significance

High.

AI capabilities are product-facing architectural capabilities but are not necessarily equivalent to individual implementation components.

---

# 6. AI Use Case

### Definition

An **AI Use Case** describes a concrete interaction or workflow in which an AI capability is applied to achieve a user or system goal.

Example:

```text
User asks for similar collection items
        ↓
AI interprets request
        ↓
Application retrieves relevant data
        ↓
AI assists ranking
        ↓
Application returns result
```

### Relationship

```text
AI Capability
      ↓
AI Use Case
      ↓
Application Use Case
```

An AI use case may participate in an existing application use case or represent an AI-specific workflow.

---

# 7. AI Interaction

An **AI Interaction** is an individual runtime exchange between an application or user and an AI capability.

It may include:

- input;
- context;
- instructions;
- model invocation;
- output;
- validation;
- tool calls;
- telemetry.

An interaction is a runtime concept rather than a business entity.

---

# 8. Model

A **Model** is an AI inference capability used to process inputs and produce outputs.

Examples conceptually include:

- language model;
- embedding model;
- classification model;
- multimodal model.

The specific provider and model selection are intentionally outside this document.

---

# 9. Model Invocation

A **Model Invocation** is an execution request sent to a model.

It may contain:

```text
Model
Instructions
Input
Context
Parameters
Tools
Output Requirements
```

The invocation is a runtime operation.

---

# 10. Inference

**Inference** is the execution of a model against a defined input and context to produce an output.

Inference is not equivalent to business reasoning.

The system must distinguish:

```text
Model inference
```

from:

```text
Domain rule evaluation
```

---

# 11. Generation

**Generation** is the production of new content or structured information by an AI model.

Examples:

- text;
- summaries;
- descriptions;
- structured candidate data;
- explanations.

Generated content is non-authoritative by default.

---

# 12. Classification

**Classification** is the assignment of one or more candidate categories, labels, or classes to input data.

Examples:

```text
Item → Category
Text → Intent
Content → Tag
Entity → Classification
```

Classification results require validation appropriate to their business impact.

---

# 13. Extraction

**Extraction** is the identification of structured information from unstructured or semi-structured input.

Examples:

```text
Text
 ↓
Entity
Metadata
Date
Category
Relationship
```

Extraction output must be treated as AI-derived information until validated.

---

# 14. Recommendation

A **Recommendation** is an AI-generated proposal that identifies potentially relevant information, actions, relationships, or choices.

Recommendations are advisory unless explicitly accepted through an application workflow.

---

# 15. Context

**Context** is the information provided to an AI capability to enable an interaction to be interpreted correctly.

Context may include:

- user input;
- conversation history;
- retrieved information;
- relevant domain data;
- workflow state;
- system instructions;
- tool results.

Context must be intentionally constructed rather than assumed to be globally available.

---

# 16. Knowledge Source

A **Knowledge Source** is a source from which information may be retrieved or supplied as context to AI.

Possible sources include:

- CollectionHub domain data;
- application data;
- user-provided information;
- documentation;
- external systems;
- curated knowledge repositories.

Knowledge source authority must be explicit.

---

# 17. Retrieval

**Retrieval** is the process of obtaining information relevant to an AI operation.

Retrieval may be:

- deterministic;
- keyword-based;
- semantic;
- vector-based;
- hybrid;
- application-specific;
- external.

Retrieval is a mechanism for obtaining context and does not itself establish truth.

---

# 18. Retrieval Result

A **Retrieval Result** is information returned from a knowledge source as the result of a retrieval operation.

It may contain:

- content;
- identifier;
- relevance score;
- source;
- metadata;
- provenance.

Retrieval results may subsequently become AI context.

---

# 19. Grounding

**Grounding** is the process of constraining or supporting an AI response using explicit information sources.

Conceptually:

```text
Question
   ↓
Retrieval
   ↓
Relevant Sources
   ↓
Context
   ↓
AI Generation
```

Grounding is particularly important when AI output must reflect CollectionHub data rather than general model knowledge.

---

# 20. Prompt

A **Prompt** is the structured input provided to an AI model.

A prompt may contain:

- instructions;
- user input;
- contextual information;
- retrieved information;
- output requirements.

A prompt is an implementation/runtime concept and should not automatically become a persisted domain concept.

---

# 21. Instruction

An **Instruction** defines behavior expected from an AI capability.

Instructions may establish:

- role;
- behavioral constraints;
- response format;
- safety constraints;
- tool-use rules;
- domain context.

---

# 22. System Instruction

A **System Instruction** is a high-priority instruction that defines the intended behavior of an AI capability independently of a particular user request.

It should contain stable behavioral constraints rather than volatile user-specific information.

---

# 23. Prompt Template

A **Prompt Template** is a reusable structure used to construct prompts for a particular AI capability.

Templates should separate:

```text
Stable Instructions
+
Dynamic Context
+
User Input
+
Output Contract
```

Prompt architecture is defined later in:

`08_AI_PROMPTING_AND_INSTRUCTION_ARCHITECTURE.md`

---

# 24. AI State

**AI State** represents information required to continue or coordinate an AI workflow.

Examples include:

- current interaction state;
- workflow progress;
- pending tool operation;
- validation status;
- intermediate result.

AI state must not automatically be confused with domain state.

---

# 25. Conversation

A **Conversation** is a sequence of related user and AI interactions that share contextual continuity.

A conversation may contain:

```text
User Message
AI Response
Tool Call
Tool Result
User Follow-up
AI Response
```

The exact persistence model is defined later.

---

# 26. Session

A **Session** represents the runtime scope in which an AI interaction or conversation occurs.

A session may provide:

- authentication context;
- request context;
- conversation association;
- runtime configuration.

A session is primarily a runtime concept.

---

# 27. Memory

**Memory** represents information intentionally retained to support future AI interactions.

Memory may be:

- short-term;
- conversational;
- task-specific;
- long-term;
- user-specific;
- system-managed.

Memory must have explicit lifecycle, retention, privacy, and authority rules.

---

# 28. Working Context

**Working Context** is temporary information maintained during a specific AI operation or workflow.

It differs from persistent memory because it is generally scoped to the current operation.

---

# 29. Tool

A **Tool** is an explicitly exposed operation that an AI capability may request.

Examples:

```text
SearchCollection
RetrieveItem
FindRelatedItems
GetMetadata
CreateSuggestion
```

A tool is an application-controlled capability.

AI does not gain arbitrary system access merely because a tool exists.

---

# 30. Tool Contract

A **Tool Contract** defines:

- tool identity;
- input schema;
- output schema;
- allowed use;
- authorization requirements;
- failure behavior;
- side effects.

Tool contracts are critical safety boundaries.

---

# 31. Tool Call

A **Tool Call** is a request by an AI workflow to execute a defined tool.

The lifecycle is:

```text
AI
 ↓
Tool Call
 ↓
Validation
 ↓
Authorization
 ↓
Tool Execution
 ↓
Tool Result
 ↓
AI Context
```

The exact validation and authorization architecture is defined later.

---

# 32. Tool Result

A **Tool Result** is the structured result returned by a tool.

It should be distinguishable from model-generated content.

---

# 33. Agent

An **Agent** is an AI-driven runtime capability that can perform a bounded sequence of reasoning and tool-use operations toward a defined objective.

An agent is not equivalent to unrestricted autonomy.

An agent must have:

- an explicit objective;
- allowed tools;
- execution boundaries;
- termination conditions;
- error handling;
- resource limits;
- observability.

---

# 34. Agent Action

An **Agent Action** is an individual operation selected by an agent during execution.

Examples:

```text
Retrieve data
Search
Classify
Generate
Request confirmation
Execute approved tool
```

Actions must remain within the agent's declared capabilities.

---

# 35. AI Workflow

An **AI Workflow** is a defined sequence of AI and non-AI operations that achieves a particular objective.

Example:

```text
Input
 ↓
Interpret
 ↓
Retrieve
 ↓
Generate
 ↓
Validate
 ↓
Persist / Present
```

A workflow may contain deterministic application steps and AI steps.

---

# 36. AI Step

An **AI Step** is an individual AI-related operation inside a workflow.

Examples:

- classify;
- extract;
- generate;
- summarize;
- rank;
- interpret.

---

# 37. Orchestration

**Orchestration** coordinates multiple AI and application operations.

It determines:

- execution order;
- branching;
- retries;
- validation;
- tool invocation;
- state transitions;
- termination.

Orchestration must not replace domain rules.

---

# 38. Job

An **AI Job** is an asynchronous unit of work executed independently from the originating synchronous request.

Examples:

- batch enrichment;
- classification;
- embedding generation;
- re-indexing;
- evaluation.

Jobs must have explicit lifecycle and failure semantics.

---

# 39. AI Output

An **AI Output** is the result produced by an AI capability.

It may contain:

- generated content;
- structured data;
- classification;
- recommendation;
- extracted information;
- tool selection;
- reasoning-related metadata where appropriate.

AI output is untrusted by default.

---

# 40. Structured Output

A **Structured Output** is an AI response conforming to a defined machine-readable schema.

Examples:

```text
JSON object
Command candidate
Classification result
Extraction result
Recommendation structure
```

Structured output improves validation but does not guarantee semantic correctness.

---

# 41. Validation

**Validation** determines whether AI output is acceptable for its intended use.

Validation may include:

- schema validation;
- business validation;
- domain validation;
- safety validation;
- authorization;
- confidence thresholds;
- consistency checks.

Validation is mandatory for consequential AI operations.

---

# 42. Guardrail

A **Guardrail** is a control that prevents or limits undesired AI behavior.

Guardrails may operate:

```text
Before model invocation
During tool selection
After model output
Before state mutation
```

Examples:

- allowed tools;
- output schemas;
- content restrictions;
- token limits;
- permission checks;
- domain constraints.

---

# 43. Confidence

**Confidence** represents the estimated reliability of an AI result.

Confidence may be:

- model-provided;
- system-derived;
- heuristic;
- evaluation-based.

Confidence must not automatically be treated as proof of correctness.

---

# 44. AI Provider

An **AI Provider** is an external service or infrastructure implementation that provides access to AI models or related AI capabilities.

Provider examples are intentionally implementation-specific and are not fixed by this inventory.

Provider abstraction is addressed in:

`07_AI_MODEL_AND_PROVIDER_STRATEGY.md`

---

# 45. Model Deployment

A **Model Deployment** represents a configured runtime endpoint or deployment through which a model can be invoked.

It may define:

- provider;
- model;
- version;
- region;
- capacity;
- configuration.

This is an infrastructure/runtime concept.

---

# 46. AI Request

An **AI Request** is a runtime request from an application or AI workflow to execute an AI capability.

It may include:

- capability identifier;
- model selection;
- input;
- context;
- instructions;
- output requirements;
- correlation metadata.

---

# 47. AI Response

An **AI Response** is the runtime result returned by an AI provider or AI execution layer.

It may include:

- generated output;
- structured output;
- usage information;
- provider metadata;
- finish status;
- errors.

An AI response is not automatically an accepted application result.

---

# 48. AI Execution

**AI Execution** represents the complete runtime lifecycle of an AI operation.

Conceptually:

```text
Request
 ↓
Context Construction
 ↓
Instruction Construction
 ↓
Model Invocation
 ↓
Provider Response
 ↓
Output Parsing
 ↓
Validation
 ↓
Application Result
```

It may also contain:

```text
Tool Calls
Retries
Fallbacks
Telemetry
Cost Tracking
```

---

# 49. Safety Policy

A **Safety Policy** defines restrictions on AI behavior intended to prevent harmful, unauthorized, or unsafe outcomes.

Safety policies may apply to:

- inputs;
- outputs;
- tool usage;
- data access;
- autonomous actions.

---

# 50. AI Policy

An **AI Policy** is a formal architectural or operational rule governing AI behavior.

Examples:

- provider restrictions;
- retention requirements;
- model usage restrictions;
- tool permissions;
- human approval requirements.

---

# 51. AI Boundary

An **AI Boundary** defines what an AI component may and may not do.

Examples:

```text
May:
    classify data

May not:
    modify authoritative state directly
```

Boundaries must be explicit.

---

# 52. Human Oversight

**Human Oversight** represents explicit human involvement required before or after an AI operation.

Possible modes include:

```text
No human review
Human review optional
Human confirmation required
Human approval required
Human-only decision
```

The appropriate mode depends on impact and risk.

---

# 53. Evaluation

**Evaluation** is the systematic measurement of AI capability quality.

Evaluation differs from ordinary unit testing because model behavior may be probabilistic.

Evaluation may measure:

- correctness;
- relevance;
- groundedness;
- safety;
- consistency;
- latency;
- cost;
- structured-output validity.

---

# 54. Evaluation Dataset

An **Evaluation Dataset** is a controlled collection of representative inputs and expected or acceptable outcomes used to assess an AI capability.

Datasets must be versioned and managed appropriately.

---

# 55. Evaluation Metric

An **Evaluation Metric** is a defined measurement used to determine whether an AI capability satisfies a quality requirement.

Examples:

- accuracy;
- precision;
- recall;
- groundedness;
- relevance;
- schema validity;
- task success;
- latency.

---

# 56. AI Test

An **AI Test** verifies a defined technical or behavioral property of an AI capability.

AI testing may include:

- unit tests;
- integration tests;
- contract tests;
- prompt tests;
- evaluation tests;
- safety tests;
- regression tests.

---

# 57. AI Regression

An **AI Regression** occurs when a change to:

- model;
- prompt;
- context;
- provider;
- tool;
- orchestration;
- configuration;

causes previously acceptable AI behavior to degrade.

AI regressions require explicit detection and management.

---

# 58. Provenance

**Provenance** identifies where information originated.

Potential provenance categories include:

```text
Domain
Application
User
External Source
AI Generated
AI Inferred
Retrieved
Tool Result
```

Provenance is important for:

- trust;
- auditing;
- validation;
- privacy;
- debugging;
- user transparency.

---

# 59. AI-Derived Data

**AI-Derived Data** is information produced or inferred through an AI operation.

It includes:

- classifications;
- summaries;
- extracted metadata;
- recommendations;
- inferred relationships;
- generated descriptions.

AI-derived data must not automatically become authoritative domain data.

---

# 60. AI Artifact

An **AI Artifact** is a persisted or externally visible result generated during AI processing.

Examples:

- generated description;
- classification result;
- embedding;
- evaluation result;
- AI interaction record.

Whether an artifact is persisted depends on the capability and lifecycle requirements.

---

# 61. AI Feature

An **AI Feature** is a user-visible product capability that uses one or more AI capabilities.

The distinction is:

```text
AI Capability
      ↓
Technical / architectural capability

AI Feature
      ↓
Product-facing functionality
```

Product determines which capabilities become features.

---

# 62. AI Capability vs AI Feature

These concepts must not be conflated.

Example:

```text
AI Capability:
Semantic Retrieval

AI Feature:
"Find similar items in my collection"
```

One capability may support multiple features.

---

# 63. AI Use Case vs AI Workflow

The distinction is:

```text
AI Use Case
    = desired outcome / interaction

AI Workflow
    = execution sequence used to achieve it
```

For example:

```text
Use Case:
Generate collection enrichment

Workflow:
Retrieve → Extract → Validate → Suggest → Confirm
```

---

# 64. AI State vs Domain State

The distinction is mandatory.

```text
AI State
    = information required to coordinate AI processing

Domain State
    = authoritative business state
```

AI state must not become a hidden substitute for domain state.

---

# 65. AI Memory vs Domain Persistence

Memory exists to improve future AI interactions.

Domain persistence exists to maintain authoritative business state.

Therefore:

```text
AI Memory
    ≠
Domain Persistence
```

They may coexist but must remain architecturally distinct.

---

# 66. Tool vs External Integration

A **Tool** is an AI-facing operation contract.

An **External Integration** is a technical connection to an external system.

The relationship may be:

```text
AI
 ↓
Tool Contract
 ↓
Application / Infrastructure
 ↓
External Integration
```

The AI should not directly own external integration mechanics.

---

# 67. Agent vs Workflow

An **Agent** may dynamically select actions within bounded constraints.

A **Workflow** generally defines a controlled execution sequence.

Conceptually:

```text
Workflow:
Defined sequence

Agent:
Bounded dynamic decision-making
```

An agent may operate inside a workflow.

A workflow does not automatically require an agent.

---

# 68. AI Provider vs Model

A provider supplies access to models or AI capabilities.

A model performs inference.

Conceptually:

```text
Provider
   ↓
Model
   ↓
Inference
```

Provider and model must remain separate architectural concepts.

---

# 69. AI Concept Relationships

The core relationship graph is:

```text
AI Capability
      │
      ▼
AI Use Case
      │
      ▼
AI Workflow
      │
      ├───────────────┐
      ▼               ▼
AI Interaction      AI Job
      │
      ▼
Context + Instructions
      │
      ▼
Model Invocation
      │
      ▼
AI Response
      │
      ▼
AI Output
      │
      ▼
Validation
      │
      ├── Reject
      ├── Suggest
      ├── Confirm
      └── Apply
```

Supporting concepts:

```text
Knowledge Source
      ↓
Retrieval
      ↓
Retrieval Result
      ↓
Context
```

and:

```text
AI
 ↓
Tool
 ↓
Tool Result
 ↓
Context
```

---

# 70. Conceptual Relationship with CollectionHub

The broader system relationship is:

```text
User
 │
 ▼
Product Feature
 │
 ▼
Application Use Case
 │
 ├───────────────┐
 ▼               ▼
AI Capability   Domain
 │               │
 ▼               ▼
AI Workflow    Business Rules
 │
 ▼
AI Infrastructure
 │
 ├── Provider
 ├── Model
 ├── Retrieval
 ├── Memory
 └── Tools
```

This relationship must remain compatible with the architecture defined in `02_ARCHITECTURE`.

---

# 71. Authority Classification

AI concepts should be classified by authority.

| Concept | Default Authority |
|---|---|
| Domain Entity | Authoritative |
| Domain Aggregate | Authoritative |
| Domain Invariant | Authoritative |
| Application Contract | Authoritative within application boundary |
| AI Capability | Capability definition |
| AI Output | Non-authoritative |
| AI Recommendation | Non-authoritative |
| AI Classification | Candidate information |
| AI Memory | Non-authoritative |
| Retrieval Result | Source-dependent |
| Tool Result | Depends on source |
| External Source | Source-dependent |
| AI-Generated Data | Non-authoritative by default |
| Validated Application Result | Authoritative within its application context |

---

# 72. Concept Lifecycle Categories

AI concepts can also be classified by lifecycle.

## Ephemeral

Usually exists only during an execution.

Examples:

- AI Request;
- AI Response;
- Working Context;
- Tool Call.

## Session-Bound

Exists during a user/session interaction.

Examples:

- Session;
- Conversation;
- short-term state.

## Persistable

May require durable storage.

Examples:

- AI Memory;
- AI Artifact;
- Evaluation Result;
- AI Job;
- AI-derived data.

## Authoritative

Represents business truth.

These are primarily domain concepts rather than AI concepts.

---

# 73. Conceptual Anti-Patterns

The following interpretations are prohibited unless explicitly justified.

## 73.1 AI Entity

Treating an AI-generated concept as a domain entity merely because the model produced it.

---

## 73.2 AI-Owned Business Rule

Encoding business invariants inside prompts or model behavior.

---

## 73.3 AI Database

Creating a separate AI persistence model that becomes an alternative source of business truth without explicit architectural justification.

---

## 73.4 Prompt as Business Logic

Using prompt instructions as a replacement for deterministic domain validation.

---

## 73.5 Agent as Application

Allowing an agent to become an uncontrolled substitute for application architecture.

---

## 73.6 Memory as Domain State

Using AI memory to store authoritative business state.

---

## 73.7 Confidence as Validation

Treating model confidence as equivalent to deterministic correctness.

---

## 73.8 Retrieval as Truth

Treating retrieved information as authoritative without considering source authority and provenance.

---

# 74. Concept Naming Rules

AI documentation should follow these naming rules.

### Use "AI Capability"

When referring to a reusable AI-enabled capability.

### Use "AI Use Case"

When describing a concrete goal or interaction.

### Use "AI Workflow"

When describing execution steps.

### Use "AI Interaction"

When describing a runtime exchange.

### Use "AI Output"

When describing generated or inferred results.

### Use "AI-Derived Data"

When emphasizing provenance.

### Use "AI Memory"

When referring to intentionally retained AI-related context.

### Use "Domain State"

When referring to authoritative business state.

### Use "Tool"

When describing an explicitly exposed AI-callable operation.

### Use "Agent"

Only when bounded dynamic AI decision-making is actually required.

---

# 75. Conceptual Decision Rules

When a new AI concept is discovered during the `03_AI` phase, it should first be classified as one of:

```text
Existing Concept
     ↓
Specialization of Existing Concept
     ↓
New AI Concept
     ↓
Domain Concept
     ↓
Application Concept
     ↓
Infrastructure Concept
```

A new concept should not automatically become a new architectural component.

The semantic concept must be established before implementation structure is introduced.

---

# 76. Inventory Usage Rules

This inventory serves as the vocabulary baseline for:

- AI architecture documents;
- AI use cases;
- AI contracts;
- AI evaluation;
- AI testing;
- product documentation;
- development documentation.

Subsequent documents should avoid introducing synonyms that create ambiguity.

For example, the following should not be used interchangeably without explicit definition:

```text
Memory
Context
State
Knowledge
Data
Persistence
```

They represent different concepts.

---

# 77. Conceptual Consistency Requirements

The following requirements apply to the entire AI phase.

- [ ] AI concepts must not silently redefine domain concepts.
- [ ] AI-generated information must be distinguishable from authoritative information.
- [ ] AI state must remain distinct from domain state.
- [ ] AI memory must remain distinct from domain persistence.
- [ ] Tools must have explicit contracts.
- [ ] Agents must have explicit boundaries.
- [ ] AI workflows must have defined ownership.
- [ ] AI outputs must have explicit validation expectations.
- [ ] Provider and model concepts must remain distinct.
- [ ] Product features must remain distinct from AI capabilities.
- [ ] AI use cases must remain distinct from workflows.
- [ ] Provenance must be available where authority matters.
- [ ] Technical implementation details must not redefine conceptual terminology.

---

# 78. Inventory Completion Criteria

This document is considered complete when:

- [ ] Core AI concepts are identified.
- [ ] AI concepts are grouped by conceptual responsibility.
- [ ] AI capabilities are distinguished from AI features.
- [ ] AI use cases are distinguished from workflows.
- [ ] Models and providers are distinguished.
- [ ] Context and knowledge are distinguished.
- [ ] Memory and domain state are distinguished.
- [ ] Tools and integrations are distinguished.
- [ ] Agents and workflows are distinguished.
- [ ] AI output and authoritative data are distinguished.
- [ ] Validation and confidence are distinguished.
- [ ] AI-specific governance concepts are identified.
- [ ] AI evaluation concepts are identified.
- [ ] Provenance is recognized as an architectural concern.
- [ ] Conceptual anti-patterns are documented.
- [ ] Naming rules are established.
- [ ] The vocabulary is sufficient to support the remaining AI architecture phase.

---

# 79. Final Concept Inventory Statement

The CollectionHub AI vocabulary is based on a fundamental distinction:

```text
Business Truth
    ↓
Domain

AI Capability
    ↓
Interpretation / Generation / Retrieval / Assistance

AI Output
    ↓
Candidate Information

Validation
    ↓
Controlled Application Result

Persistence
    ↓
Authoritative State where applicable
```

AI introduces probabilistic capabilities into a deterministic domain-oriented architecture.

The purpose of this inventory is therefore not merely to define terminology.

It establishes the semantic boundaries required to prevent:

- AI concepts from leaking into the domain without justification;
- model outputs from being mistaken for business truth;
- prompts from becoming hidden business logic;
- memory from becoming hidden persistence;
- agents from becoming uncontrolled application logic;
- and technical AI provider concepts from contaminating stable architectural concepts.

This inventory provides the vocabulary baseline for the remainder of the `03_AI` phase.

The next document must build upon this vocabulary by defining the actual AI capabilities and their responsibilities within CollectionHub.