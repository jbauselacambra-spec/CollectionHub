# 10. AI Knowledge Sources, Grounding and Retrieval Architecture

## 1. Purpose

This document defines the architecture governing how CollectionHub AI obtains, selects, retrieves, validates, contextualizes, and grounds knowledge used during AI-assisted interactions.

The objective is to ensure that AI responses are based on the appropriate combination of:

- authoritative domain knowledge;
- application state;
- user-provided context;
- persisted CollectionHub data;
- explicitly retrieved information;
- deterministic domain rules;
- AI-generated interpretation.

This document establishes the architectural boundaries between knowledge acquisition, retrieval, grounding, reasoning, and response generation.

It does **not** define the implementation of a specific vector database, embedding provider, LLM provider, or external search engine.

Those decisions remain implementation concerns unless explicitly promoted to an architectural decision.

---

## 2. Architectural Position

The AI knowledge and retrieval architecture is positioned between the AI orchestration layer and the CollectionHub application/domain boundaries.

Conceptually:

```text
User
 |
 v
AI Interaction
 |
 v
AI Orchestration
 |
 +-----------------------------+
 | Context Assembly             |
 | Knowledge Requirement        |
 | Retrieval Planning           |
 +-----------------------------+
              |
              v
     Knowledge Access Layer
              |
       +------+------+
       |             |
       v             v
Domain Knowledge   Runtime Data
       |             |
       +------+------+
              |
              v
        Retrieval Layer
              |
              v
       Grounded Context
              |
              v
        AI Reasoning
              |
              v
        AI Response
```

The architecture explicitly separates:

1. **knowledge sources**;
2. **knowledge retrieval**;
3. **context assembly**;
4. **grounding**;
5. **reasoning**;
6. **response generation**.

AI reasoning must not directly access arbitrary persistence or infrastructure resources.

---

## 3. Core Principles

### 3.1 Grounding Before Reasoning

Whenever an AI response depends on CollectionHub-specific facts, the relevant facts must be grounded before reasoning over them.

The AI must not assume that knowledge about the application exists implicitly inside the model.

---

### 3.2 Source Authority Must Be Explicit

Not all knowledge sources have the same authority.

The architecture therefore requires each source to have an explicit authority classification.

Example:

```text
SYSTEM_RULE
    >
DOMAIN_RULE
    >
PERSISTED_APPLICATION_STATE
    >
USER_PROVIDED_CONTEXT
    >
RETRIEVED_REFERENCE
    >
AI_INFERENCE
```

This ordering is conceptual and must be refined according to the specific use case.

The AI must never silently replace an authoritative source with a lower-authority inference.

---

### 3.3 Retrieval Is Not Truth

Retrieval provides candidate information.

Retrieval does not establish that the information is:

- correct;
- current;
- applicable;
- authoritative;
- internally consistent.

The grounding stage is responsible for evaluating retrieved information before it becomes trusted AI context.

---

### 3.4 Domain Truth Remains Outside the AI

The AI must not become the owner of CollectionHub business truth.

Business invariants remain enforced by the domain/application architecture.

AI may:

- interpret;
- explain;
- suggest;
- classify;
- summarize;
- identify possible relationships;
- request domain operations.

AI must not independently redefine domain truth.

---

### 3.5 Context Must Be Purpose-Bound

Only knowledge relevant to the current AI interaction should be included in the reasoning context.

The architecture must avoid indiscriminate retrieval.

More context does not necessarily produce better reasoning.

---

### 3.6 Traceability Is Mandatory

Knowledge used to produce an AI response should be traceable whenever technically and legally possible.

The system should be able to identify:

- which source was used;
- which version was used;
- when it was retrieved;
- which retrieval strategy selected it;
- how it was transformed;
- which AI interaction consumed it.

---

## 4. Knowledge Source Taxonomy

CollectionHub knowledge is divided into several categories.

### 4.1 System Knowledge

System knowledge describes AI runtime constraints and operational rules.

Examples:

- AI capabilities;
- safety constraints;
- tool permissions;
- output contracts;
- context limits;
- execution policies.

System knowledge has the highest architectural authority.

---

### 4.2 Domain Knowledge

Domain knowledge describes CollectionHub concepts and business semantics.

Examples:

- collection;
- item;
- ownership;
- valuation;
- acquisition;
- provenance;
- cataloguing;
- condition;
- classification;
- relationships between domain concepts.

Domain knowledge must remain aligned with the canonical domain model.

---

### 4.3 Application Knowledge

Application knowledge represents the current state of CollectionHub.

Examples:

- user's collections;
- stored items;
- acquisitions;
- valuations;
- metadata;
- classifications;
- relationships;
- historical records.

Application knowledge is dynamic.

It must therefore be retrieved from authoritative application sources rather than treated as static AI knowledge.

---

### 4.4 User Context

User context includes information explicitly provided during an interaction.

Examples:

- current task;
- requested objective;
- supplied item information;
- constraints;
- preferences relevant to the current interaction.

User context must not automatically become persistent knowledge.

---

### 4.5 External Reference Knowledge

External knowledge may be retrieved from approved external sources.

Examples:

- reference catalogues;
- public documentation;
- market information;
- manufacturer information;
- historical references;
- external databases.

External information must be treated as potentially incomplete or stale unless its authority and freshness are established.

---

### 4.6 Derived Knowledge

Derived knowledge is information produced through deterministic processing or AI reasoning.

Examples:

- inferred classification;
- estimated similarity;
- generated summaries;
- derived relationships;
- candidate valuations;
- recommendations.

Derived knowledge must never be confused with authoritative source data.

---

## 5. Knowledge Source Authority

Each knowledge source should expose metadata describing its authority.

At minimum:

```text
SourceId
SourceType
AuthorityLevel
Origin
Version
EffectiveFrom
EffectiveTo
RetrievedAt
TrustStatus
```

Possible authority levels include:

```text
AUTHORITATIVE
VERIFIED
TRUSTED
REFERENCE
USER_ASSERTED
DERIVED
UNVERIFIED
```

The exact enumeration belongs to the contracts and domain-modeling phases.

---

## 6. Knowledge Freshness

Knowledge must be classified according to its expected volatility.

### Static Knowledge

Examples:

- architectural documentation;
- domain vocabulary;
- stable conceptual definitions.

Static knowledge can be cached aggressively.

### Slowly Changing Knowledge

Examples:

- reference catalogues;
- classification systems;
- manufacturer information.

Caching may be used, but invalidation policies are required.

### Dynamic Knowledge

Examples:

- collection contents;
- current valuations;
- user-specific records;
- current application state.

Dynamic knowledge should normally be retrieved from the authoritative application source.

### Ephemeral Knowledge

Examples:

- current user message;
- temporary interaction context;
- current tool results.

Ephemeral knowledge should not automatically be persisted.

---

## 7. Retrieval Architecture

The retrieval architecture should be organized into explicit stages.

```text
AI Request
    |
    v
Knowledge Requirement Analysis
    |
    v
Source Selection
    |
    v
Retrieval Planning
    |
    v
Candidate Retrieval
    |
    v
Filtering
    |
    v
Ranking
    |
    v
Validation
    |
    v
Grounding
    |
    v
Context Assembly
```

Each stage has a different responsibility.

---

## 8. Knowledge Requirement Analysis

Before retrieving information, the AI orchestration layer should determine what knowledge is actually required.

A retrieval requirement may include:

```text
KnowledgeType
RequiredFacts
EntityScope
TemporalScope
AuthorityRequirement
FreshnessRequirement
MaximumResults
ConfidenceRequirement
```

Example:

```text
KnowledgeType:
    CollectionItem

EntityScope:
    CurrentUser

TemporalScope:
    Current

AuthorityRequirement:
    Authoritative

FreshnessRequirement:
    Current
```

This prevents generic retrieval from becoming the default strategy.

---

## 9. Source Selection

The retrieval architecture should select sources based on the knowledge requirement.

Example decision:

```text
Question about user's collection
        |
        +--> Application state required
        |
        +--> Query domain/application source
        |
        +--> External search unnecessary
```

Another example:

```text
Question about historical item information
        |
        +--> Domain knowledge
        +--> External reference knowledge
        |
        +--> Retrieve approved references
        |
        +--> Ground and compare sources
```

Source selection should therefore precede retrieval.

---

## 10. Retrieval Strategies

CollectionHub may support multiple retrieval strategies.

### 10.1 Deterministic Retrieval

Used when the required information can be obtained through structured queries.

Examples:

- collection lookup;
- item lookup;
- acquisition lookup;
- relationship lookup.

Deterministic retrieval should be preferred when authoritative structured data exists.

---

### 10.2 Semantic Retrieval

Used when the user request is conceptually related to stored knowledge but exact matching is insufficient.

Examples:

- similar item descriptions;
- related catalogue entries;
- semantic classification.

Semantic retrieval must not override deterministic domain relationships.

---

### 10.3 Keyword Retrieval

Useful for:

- exact terminology;
- identifiers;
- names;
- catalogue references;
- historical labels.

---

### 10.4 Hybrid Retrieval

Hybrid retrieval combines multiple strategies.

Example:

```text
Exact identifier
        +
Keyword match
        +
Semantic similarity
        +
Authority ranking
```

Hybrid retrieval should be preferred for complex reference scenarios.

---

## 11. Retrieval Filtering

Retrieved candidates must be filtered before entering the AI context.

Filtering criteria may include:

- source authority;
- entity ownership;
- tenant boundaries;
- access permissions;
- temporal validity;
- content type;
- language;
- relevance;
- duplication;
- source status.

Security filtering must occur before AI context construction.

The AI must never receive information merely because the retrieval mechanism found it.

---

## 12. Retrieval Ranking

Candidates may be ranked according to:

```text
Authority
+
Relevance
+
Freshness
+
Entity specificity
+
Source quality
+
Confidence
```

A conceptual ranking model is:

```text
FinalScore =
    AuthorityWeight
    +
    RelevanceWeight
    +
    FreshnessWeight
    +
    SpecificityWeight
    +
    ConfidenceWeight
```

The exact scoring algorithm is an implementation concern.

The architecture requires only that ranking criteria remain explicit and testable.

---

## 13. Grounding

Grounding transforms retrieved information into trusted AI context.

The grounding process should:

1. identify the source;
2. verify source eligibility;
3. validate scope;
4. validate temporal applicability;
5. preserve source metadata;
6. detect conflicts;
7. assign confidence;
8. mark the resulting context appropriately.

Conceptually:

```text
Retrieved Candidate
        |
        v
Source Validation
        |
        v
Scope Validation
        |
        v
Conflict Detection
        |
        v
Confidence Assignment
        |
        v
Grounded Knowledge
```

---

## 14. Conflict Resolution

Different sources may provide conflicting information.

The AI architecture must not resolve every conflict through probabilistic model reasoning.

Conflict resolution should first apply deterministic authority rules.

Example:

```text
Current persisted domain state
        >
obsolete external reference
```

Another example:

```text
verified catalogue source
        >
unverified user-generated reference
```

If a conflict cannot be deterministically resolved, the AI should preserve the uncertainty rather than manufacture certainty.

---

## 15. Uncertainty Representation

Grounded knowledge should be capable of representing uncertainty.

Possible states:

```text
CONFIRMED
PROBABLE
POSSIBLE
CONFLICTED
UNKNOWN
UNVERIFIED
```

The AI response layer may convert these states into natural language.

The underlying grounded context should retain the structured state.

---

## 16. Context Packaging

Grounded knowledge should be packaged into a structured context representation before being supplied to the reasoning model.

Conceptually:

```text
GroundedContext
 ├── InteractionContext
 ├── DomainContext
 ├── ApplicationContext
 ├── ExternalReferences
 ├── Constraints
 ├── Uncertainties
 └── Provenance
```

The context package must distinguish facts from interpretations.

---

## 17. Provenance

Every grounded knowledge element should preserve provenance where available.

Example:

```text
KnowledgeElement
 ├── SourceId
 ├── SourceType
 ├── RetrievedAt
 ├── Version
 ├── Authority
 ├── Confidence
 └── TransformationHistory
```

This enables:

- debugging;
- auditing;
- evaluation;
- explanation;
- reproducibility;
- incident analysis.

---

## 18. Context Compression

Retrieved knowledge may exceed the context budget.

Compression may therefore be required.

Allowed operations include:

- deduplication;
- summarization;
- prioritization;
- truncation;
- structured reduction.

However, compression must preserve:

- authoritative facts;
- uncertainty;
- source provenance;
- relevant identifiers;
- important constraints.

The architecture must never optimize context size by silently removing critical qualifiers.

---

## 19. Retrieval and Persistence Boundary

Retrieval must not imply persistence.

The following are distinct operations:

```text
Retrieve
Read
Interpret
Generate
Persist
```

AI may retrieve information without acquiring authority to persist it.

Likewise, AI-generated information must not automatically become persistent domain data.

Persistence requires an explicit application/domain operation.

---

## 20. Retrieval and Security Boundary

Retrieval must respect the same security model as the rest of CollectionHub.

Security boundaries include:

- user identity;
- authorization;
- tenant boundaries where applicable;
- ownership;
- visibility;
- role permissions;
- data classification.

Security must be enforced before context reaches the model.

The AI model is not a security boundary.

---

## 21. Retrieval and Privacy

Knowledge retrieval must minimize unnecessary exposure of personal or sensitive information.

The retrieval layer should follow:

```text
Need to know
    >
retrieve only what is necessary
    >
ground only what is necessary
    >
send only what is necessary to the model
```

Sensitive information must not be retrieved merely because it is available.

---

## 22. External Knowledge Retrieval

External retrieval must be treated as a controlled capability.

The architecture should define:

- approved source categories;
- source trust policies;
- retrieval limits;
- timeout policies;
- freshness policies;
- failure behavior;
- provenance requirements;
- content validation.

External knowledge should not silently override authoritative CollectionHub state.

---

## 23. Failure Modes

Retrieval may fail.

Possible failure conditions include:

```text
No results
Timeout
Unavailable source
Unauthorized source
Stale data
Conflicting sources
Low confidence
Malformed result
Insufficient context
Retrieval budget exceeded
```

The AI runtime must distinguish between:

```text
No knowledge found
```

and:

```text
Knowledge exists but retrieval failed
```

These conditions have different operational meanings.

---

## 24. Retrieval Fallback Policy

Fallback behavior should be explicit.

Example:

```text
Primary authoritative source
        |
        +--> available --> use
        |
        +--> unavailable
                |
                v
        approved secondary source
                |
                +--> available --> use with lower authority
                |
                +--> unavailable --> report uncertainty
```

The system must not silently substitute arbitrary external knowledge.

---

## 25. Knowledge Caching

Caching may be used to improve performance.

Caching must preserve:

- authorization boundaries;
- source identity;
- freshness requirements;
- version information;
- invalidation semantics.

Dynamic user-specific application state should have stricter cache policies than static domain knowledge.

---

## 26. Knowledge Versioning

Knowledge that affects AI behavior should be versionable where practical.

Versioning may apply to:

- domain documentation;
- retrieval indexes;
- classification taxonomies;
- external reference snapshots;
- grounding rules;
- ranking policies.

The version consumed by an AI interaction should be observable for diagnostics.

---

## 27. AI Interaction Trace

A traceable AI interaction should conceptually contain:

```text
InteractionId
    |
    +--> UserRequest
    |
    +--> KnowledgeRequirements
    |
    +--> RetrievalOperations
    |
    +--> RetrievedSources
    |
    +--> GroundingResults
    |
    +--> ContextPackage
    |
    +--> ModelExecution
    |
    +--> GeneratedResponse
```

This trace does not require storing sensitive content indiscriminately.

Observability must remain subject to privacy and retention policies.

---

## 28. Retrieval Governance

The retrieval architecture must be governed by explicit policies.

Policies should define:

- allowed sources;
- prohibited sources;
- authority levels;
- freshness requirements;
- confidence thresholds;
- maximum retrieval scope;
- security requirements;
- persistence boundaries;
- audit requirements.

These policies belong to the AI runtime and governance architecture rather than to individual prompts.

---

## 29. AI Prompt Boundary

Prompts should consume grounded context.

They should not be responsible for discovering authoritative application knowledge.

Incorrect:

```text
LLM
 |
 +--> "Find out what the user's collection contains."
```

Correct:

```text
AI Orchestrator
 |
 +--> Retrieval
 |
 +--> Grounding
 |
 +--> Context Assembly
 |
 +--> LLM
```

This separation makes the system deterministic, testable, and auditable.

---

## 30. Recommended Context Contract

A conceptual contract is:

```text
AIKnowledgeContext
{
    InteractionContext
    DomainFacts
    ApplicationFacts
    ExternalReferences
    Constraints
    Uncertainties
    Provenance
}
```

The concrete contract must be defined later in the AI contracts and runtime phase.

---

## 31. Architectural Responsibilities

### AI Orchestrator

Responsible for:

- identifying knowledge requirements;
- coordinating retrieval;
- assembling grounded context;
- invoking AI reasoning.

### Knowledge Access Layer

Responsible for:

- abstracting knowledge sources;
- enforcing access boundaries;
- providing consistent retrieval interfaces.

### Retrieval Layer

Responsible for:

- candidate discovery;
- filtering;
- ranking;
- retrieval policies.

### Grounding Layer

Responsible for:

- source validation;
- authority evaluation;
- conflict handling;
- confidence assignment;
- provenance.

### Domain/Application Layer

Responsible for:

- authoritative application truth;
- business rules;
- authorization;
- domain invariants.

### AI Model

Responsible for:

- reasoning over supplied context;
- language generation;
- interpretation.

The AI model does not own CollectionHub knowledge.

---

## 32. Anti-Patterns

### 32.1 Treating the LLM as the Database

The model must not be considered the authoritative source of user data.

### 32.2 Unrestricted Retrieval

Retrieving everything and allowing the model to decide what matters creates unnecessary security, cost, and quality risks.

### 32.3 Prompt-Based Authorization

Authorization must not be implemented by instructions such as:

```text
"Do not reveal private data."
```

The retrieval boundary must enforce access.

### 32.4 Mixing Facts and Inferences

A generated inference must not be presented internally as if it were source truth.

### 32.5 Silent External Substitution

External information must not silently replace unavailable authoritative application state.

### 32.6 Persisting AI Output Automatically

Generated information must pass through explicit application/domain workflows before persistence.

---

## 33. Quality Requirements

The retrieval architecture should be evaluated against:

### Relevance

Does retrieval return information relevant to the request?

### Authority

Does the system prioritize authoritative information?

### Freshness

Is dynamic information current enough for the use case?

### Precision

How much retrieved information is actually useful?

### Recall

Does retrieval find the required information?

### Grounding Accuracy

Does the generated response remain consistent with grounded facts?

### Traceability

Can the system identify the information used to generate a response?

### Security

Can unauthorized information reach the model?

The answer to the last question must be **no**.

---

## 34. Testing Strategy

Testing should cover at least:

### Unit Tests

- source selection;
- ranking;
- filtering;
- authority evaluation;
- freshness evaluation;
- conflict resolution.

### Integration Tests

- application data retrieval;
- external reference retrieval;
- authorization boundaries;
- context assembly.

### AI Evaluation Tests

- grounded response accuracy;
- unsupported claim detection;
- source adherence;
- uncertainty preservation.

### Security Tests

- cross-user retrieval;
- unauthorized source access;
- prompt injection through retrieved content;
- sensitive data leakage.

---

## 35. Prompt Injection Through Retrieved Knowledge

External or user-controlled knowledge must be treated as untrusted content.

Retrieved content may contain instructions intended to manipulate the AI.

Therefore:

```text
Retrieved Content
        !=
AI Instructions
```

The runtime must maintain a strict distinction between:

- system instructions;
- runtime policies;
- developer constraints;
- grounded facts;
- untrusted retrieved content.

Retrieved content must never acquire instruction authority merely because it appears inside the context.

---

## 36. Retrieval Budget

Each AI interaction should operate within explicit retrieval budgets.

Potential limits include:

```text
Maximum sources
Maximum records
Maximum tokens
Maximum external requests
Maximum retrieval latency
Maximum tool calls
```

Budgets protect:

- performance;
- cost;
- predictability;
- context quality.

---

## 37. Deterministic vs Probabilistic Boundaries

The architecture distinguishes deterministic responsibilities from probabilistic responsibilities.

### Deterministic

- authorization;
- source selection policies;
- domain queries;
- filtering;
- authority classification;
- persistence;
- business invariants.

### Probabilistic

- semantic similarity;
- natural-language interpretation;
- classification suggestions;
- summarization;
- reasoning.

Where deterministic mechanisms can provide authoritative information, they should be preferred.

---

## 38. Architectural Decision Rules

The following rules are established:

1. Authoritative application data must come from application/domain sources.
2. Retrieval must occur before grounded AI reasoning when external or application knowledge is required.
3. Retrieval must respect authorization boundaries.
4. Retrieved content is not automatically trusted.
5. Grounding must preserve provenance and uncertainty.
6. AI inference must remain distinguishable from source facts.
7. AI-generated information must not automatically become persistent domain truth.
8. External knowledge must not silently override authoritative application state.
9. The AI model must not be treated as a security boundary.
10. Retrieval behavior must be observable and testable.

---

## 39. Relationship With Previous AI Architecture Documents

This document extends:

- `00_AI_FOUNDATION_AND_SCOPE.md`;
- `01_AI_USE_CASES_AND_INTERACTION_MODEL.md`;
- `02_AI_CAPABILITY_MODEL.md`;
- `03_AI_DOMAIN_INTERACTION_AND_BOUNDARIES.md`;
- `04_AI_DOMAIN_INTERACTION_AND_BOUNDARIES.md`;
- `05_AI_ORCHESTRATION_AND_EXECUTION_MODEL.md`;
- `06_AI_AGENT_AND_TOOL_BOUNDARIES.md`;
- `07_AI_CONTEXT_ENGINEERING_AND_CONVERSATION_MODEL.md`;
- `08_AI_RUNTIME_CONTEXT_AND_STATE_MODEL.md`;
- `09_AI_CONTEXT_AND_KNOWLEDGE_ARCHITECTURE.md`.

The exact numbering and filenames must remain aligned with the canonical project baseline.

This document specifically deepens the knowledge retrieval and grounding concerns identified by the context architecture.

---

## 40. Relationship With Subsequent AI Documents

This document provides the foundation for subsequent work covering:

- AI knowledge contracts;
- retrieval interfaces;
- grounding contracts;
- AI safety boundaries;
- runtime policies;
- evaluation;
- observability;
- governance;
- implementation readiness.

Those concerns must not be implemented prematurely inside this architectural document.

---

## 41. Final Architectural Baseline

CollectionHub AI knowledge architecture is based on the following model:

```text
                 +----------------------+
                 |      AI Request      |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 | Knowledge Requirement|
                 |      Analysis        |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |    Source Selection  |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |      Retrieval       |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 | Filtering / Ranking  |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |      Grounding       |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |   Context Assembly   |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |     AI Reasoning     |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |    AI Response       |
                 +----------------------+
```

The architectural invariant is:

> **CollectionHub AI reasons over explicitly selected and grounded knowledge; it does not own or invent authoritative application truth.**

This invariant must remain valid throughout the subsequent AI contract, runtime, safety, testing, governance, and implementation phases.

---

## 42. Status

**Document status:** Architectural Baseline

**Phase:** AI Architecture

**Block:** 03.3 — AI Architecture

**Sequence:** 10

**Implementation status:** Not yet implemented

**Primary purpose:** Define knowledge retrieval, grounding, authority, provenance, and context boundaries.

**Next architectural concern:** Define the formal contracts governing AI knowledge retrieval and grounded context exchange.