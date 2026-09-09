# 09 — AI Context and Knowledge Architecture

## 1. Purpose

This document defines the architectural strategy for providing context and knowledge to AI capabilities within CollectionHub.

Its purpose is to establish how AI obtains relevant information without acquiring uncontrolled access to application state, domain persistence or external data sources.

This document defines:

- context concepts;
- knowledge concepts;
- context sources;
- context ownership;
- context assembly;
- context boundaries;
- retrieval strategies;
- authorization;
- data minimization;
- contextual relevance;
- context prioritization;
- context versioning;
- knowledge access contracts;
- semantic retrieval;
- structured retrieval;
- context lifecycle;
- caching;
- invalidation;
- context observability;
- security against untrusted knowledge;
- implementation boundaries.

This document does not define prompt composition in detail, model/provider selection or AI persistence implementation.

---

## 2. Architectural Context

The AI subsystem requires information to produce useful results.

However, AI must not receive unrestricted access to CollectionHub data.

The architectural relationship is:

```text
Application
    |
    v
AI Capability
    |
    v
Context Requirements
    |
    v
Context Manager
    |
    +-------------------+
    |                   |
    v                   v
Application Queries   Knowledge Sources
    |                   |
    +---------+---------+
              |
              v
       Context Assembly
              |
              v
       Authorized Context
              |
              v
          AI Runtime
```

The Context Manager is therefore the controlled boundary between AI execution and contextual information.

---

## 3. Core Principle

The fundamental principle is:

> **AI receives explicitly authorized and purpose-specific context, never unrestricted application state.**

The AI subsystem should receive:

```text id="q4m8v2"
Only what the capability requires.
```

It should not receive:

```text id="p7n3x5"
Everything that happens to be available.
```

---

## 4. Context vs Knowledge

Context and knowledge are related but distinct concepts.

### Context

Context is information assembled specifically for the current AI execution.

Examples:

- current collection item;
- current user request;
- selected collection;
- current conversation;
- relevant application state.

### Knowledge

Knowledge is information that can be retrieved to support AI execution.

Examples:

- reference data;
- indexed collection information;
- documentation;
- semantic search results;
- historical information;
- external reference material.

Conceptually:

```text id="m5q8v1"
Knowledge
    |
    v
Retrieval
    |
    v
Context
    |
    v
AI Execution
```

---

## 5. Context Architecture Objectives

The context architecture must optimize for:

1. relevance;
2. correctness;
3. authorization;
4. data minimization;
5. freshness;
6. traceability;
7. performance;
8. cost;
9. security;
10. reproducibility.

Context should be sufficient for the capability without becoming unnecessarily large.

---

## 6. Context Ownership

Context ownership must remain explicit.

Potential ownership:

```text id="r8m3q5"
Application Context
    -> Application

Domain Information
    -> Domain / Application

AI Context
    -> AI Subsystem

External Knowledge
    -> External Source / Integration
```

AI should not become the authoritative owner of domain information.

---

## 7. Context Sources

Context may originate from:

```text id="n4v7m2"
Context Sources
├── User Input
├── Application State
├── Domain Read Data
├── Query Results
├── Collection Data
├── Conversation State
├── AI Memory
├── Knowledge Indexes
├── Reference Data
├── External Integrations
└── Tool Results
```

Each source must have explicit trust, authorization and freshness characteristics.

---

## 8. Context Source Classification

Every context source should be classified according to:

| Attribute | Meaning |
|---|---|
| Owner | Who owns the information |
| Trust | How authoritative it is |
| Sensitivity | Data classification |
| Freshness | How current it must be |
| Scope | Which users/operations may access it |
| Relevance | Which capabilities may consume it |
| Retention | How long it may remain available |

This classification supports controlled context assembly.

---

## 9. User Input Context

User input is a primary context source.

It may contain:

- questions;
- instructions;
- descriptions;
- search criteria;
- corrections;
- preferences.

User input must remain logically distinct from trusted system instructions.

---

## 10. Application Context

Application context represents information associated with the current application operation.

Examples:

```text id="j6m8q2"
Current User
Current Collection
Current Item
Current Operation
Current Locale
Current Preferences
```

Only required information should be exposed to AI.

---

## 11. Domain Context

AI may require domain information.

The preferred access pattern is:

```text id="x4n7q9"
AI Capability
      |
      v
Application Context Provider
      |
      v
Domain Information
```

AI should not directly access domain repositories or aggregates.

---

## 12. Domain Context Read Model

For AI use cases, the application may provide specialized read representations.

Example:

```text id="m7q2v4"
Domain Aggregate
      |
      v
AI Read Representation
      |
      v
Context
```

This prevents accidental exposure of:

- internal domain structures;
- persistence concerns;
- irrelevant fields;
- mutable aggregate internals.

---

## 13. Knowledge Sources

Knowledge sources may include:

### 13.1 Structured Knowledge

Examples:

- collection metadata;
- item attributes;
- reference classifications;
- controlled vocabularies.

### 13.2 Unstructured Knowledge

Examples:

- descriptions;
- notes;
- documents;
- external textual sources.

### 13.3 Semantic Knowledge

Examples:

- embeddings;
- similarity indexes;
- vector search results.

### 13.4 External Knowledge

Examples:

- authorized external APIs;
- reference databases;
- external documents.

Each category requires different retrieval and validation strategies.

---

## 14. Knowledge Access Boundary

AI must access knowledge through explicit interfaces.

Preferred:

```text id="c6m8q1"
AI
 |
 v
Knowledge Access Contract
 |
 v
Authorized Knowledge Provider
```

Prohibited:

```text id="t3n7p5"
AI
 |
 v
Arbitrary Database Access
```

---

## 15. Context Requirements

Each capability should declare its context requirements.

Conceptual structure:

```text id="p8m4v7"
ContextRequirements
├── Required Sources
├── Optional Sources
├── Maximum Size
├── Freshness Requirement
├── Sensitivity Restrictions
├── Retrieval Strategy
└── Relevance Criteria
```

This allows context assembly to remain capability-driven.

---

## 16. Context Profile

A capability may define a context profile:

```text id="q7m2n5"
ItemClassification
    |
    +--> Current Item
    +--> Collection Context
    +--> Classification Vocabulary
    +--> Relevant Similar Items
```

The profile should specify which sources are necessary and which are optional.

---

## 17. Context Assembly

Context assembly is the process of transforming source information into an AI-ready context.

Canonical flow:

```text id="v5n8m2"
Capability Request
        |
        v
Resolve Context Requirements
        |
        v
Retrieve Sources
        |
        v
Authorize
        |
        v
Filter
        |
        v
Rank
        |
        v
Normalize
        |
        v
Limit
        |
        v
Assemble Context
```

---

## 18. Context Authorization

Authorization must happen before context reaches the model.

The architecture should assume:

```text id="k3m7q9"
Accessible by Application
```

does not automatically mean:

```text id="x8p2v4"
Appropriate for AI
```

AI-specific data policies may impose additional restrictions.

---

## 19. Context Data Minimization

Context assembly must follow the principle of minimum necessary information.

For example, if classification requires:

```text id="n4q7m2"
Item Name
Item Description
Relevant Category Vocabulary
```

the system should not automatically include:

```text id="r6v9p1"
User Email
Internal IDs
Billing Data
Audit Metadata
Unrelated Collection Items
```

---

## 20. Context Filtering

Filtering removes information that is:

- unauthorized;
- irrelevant;
- stale;
- redundant;
- too sensitive;
- outside the capability scope.

Filtering should happen before prompt composition.

---

## 21. Context Relevance

Relevant context should be selected according to the capability's requirements.

Possible relevance signals include:

- semantic similarity;
- lexical matching;
- entity relationships;
- collection membership;
- recency;
- user selection;
- explicit references.

Relevance ranking must remain observable where it materially affects AI behavior.

---

## 22. Context Ranking

When more context is available than can be supplied, context should be ranked.

Conceptual ranking:

```text id="m2v8q5"
Mandatory Context
      |
      v
Highly Relevant Context
      |
      v
Relevant Supporting Context
      |
      v
Optional Context
```

Lower-ranked context may be omitted.

---

## 23. Context Freshness

Context sources should declare freshness requirements.

Examples:

```text id="z5n7m2"
Current User State
    -> Very Fresh

Collection Metadata
    -> Fresh

Reference Documentation
    -> Versioned

Historical Knowledge
    -> Stable
```

A stale source must not be used when freshness is critical to correctness.

---

## 24. Context Consistency

Context should represent a coherent snapshot whenever the capability depends on multiple related sources.

Potential inconsistency:

```text id="p4m8q2"
Item State at T1
Collection State at T2
Reference State at T3
```

Where consistency matters, the architecture should define how the snapshot is established.

---

## 25. Context Versioning

Context should be identifiable when reproducibility matters.

Conceptually:

```text id="q6n2v8"
Context
├── Source Versions
├── Retrieval Parameters
├── Selection Rules
├── Assembly Version
└── Timestamp
```

This allows later analysis of why a particular AI result was produced.

---

## 26. Structured Context

Structured data should be preferred where the capability requires exact values.

Example:

```text id="v3m7q1"
{
  "item_name": "...",
  "category": "...",
  "year": 2026
}
```

The exact serialization format is an implementation concern.

The architectural principle is that structured information should remain structured whenever semantic precision matters.

---

## 27. Unstructured Context

Unstructured context may be appropriate for:

- descriptive text;
- notes;
- documents;
- natural-language history.

Unstructured content must still be bounded and classified.

---

## 28. Context Normalization

Before entering AI execution, context may require normalization.

Examples:

- unit normalization;
- date normalization;
- field naming;
- language normalization;
- removal of irrelevant metadata;
- canonical serialization.

Normalization should not change authoritative business meaning.

---

## 29. Semantic Retrieval

Semantic retrieval may be used when exact matching is insufficient.

Conceptual flow:

```text id="h5m8q2"
Query
  |
  v
Embedding
  |
  v
Semantic Index
  |
  v
Candidate Results
  |
  v
Authorization
  |
  v
Relevance Ranking
  |
  v
Context
```

Semantic retrieval must not bypass access controls.

---

## 30. Hybrid Retrieval

Hybrid retrieval may combine:

```text id="n8q3m5"
Keyword Search
      +
Structured Filters
      +
Semantic Search
```

This may improve relevance while retaining deterministic constraints.

---

## 31. Retrieval Filters

Retrieval should support explicit filters such as:

- tenant;
- collection;
- item type;
- date;
- ownership;
- visibility;
- language;
- source type.

Security filters must be mandatory where applicable.

---

## 32. Retrieval and Authorization Ordering

Authorization must be enforced as early as practical.

The preferred sequence is:

```text id="p7m4v2"
Request
  |
  v
Authorization Constraints
  |
  v
Retrieval
  |
  v
Ranking
  |
  v
Context
```

A retrieval result that the user is not authorized to access must never enter AI context.

---

## 33. Knowledge Quality

Knowledge sources should have quality metadata where appropriate.

Potential attributes:

```text id="m5q7v3"
Source
├── Authority
├── Freshness
├── Confidence
├── Version
└── Provenance
```

AI should not treat all knowledge sources as equally authoritative.

---

## 34. Provenance

Where practical, retrieved knowledge should preserve provenance.

Conceptually:

```text id="x8m2q5"
Context Entry
├── Source
├── Source Identifier
├── Source Version
└── Retrieval Timestamp
```

Provenance supports:

- auditing;
- debugging;
- user explanation;
- evaluation;
- reproducibility.

---

## 35. Authoritative vs Advisory Knowledge

Knowledge should be classified as:

### Authoritative

Information that the application considers a trusted source of truth.

### Advisory

Information that can help AI but must be independently validated.

For example:

```text id="q4m7v8"
Domain Classification
    -> Authoritative

External Description
    -> Advisory
```

AI must not silently elevate advisory knowledge into authoritative domain truth.

---

## 36. Conflicting Knowledge

Knowledge sources may conflict.

The architecture should define precedence.

Example:

```text id="v8n2m5"
Domain Source
    >
Approved Reference Source
    >
External Source
    >
AI Inference
```

The exact precedence is capability-specific but must be explicit.

---

## 37. AI Inference vs Retrieved Knowledge

The system must distinguish:

```text id="r3m7q9"
Retrieved Fact
```

from:

```text id="p5n8v2"
AI Inference
```

This distinction is important for:

- confidence;
- provenance;
- validation;
- user presentation;
- persistence.

---

## 38. Context Confidence

Context quality may influence AI confidence.

The architecture should avoid treating AI confidence as a substitute for source quality.

For example:

```text id="n6q2m8"
High AI Confidence
+
Low Source Quality
=
Still Uncertain
```

Source quality and model confidence are separate dimensions.

---

## 39. Context Compression

Where context is large, compression or summarization may be used.

However, compression can remove important information.

Therefore context compression should be:

- capability-aware;
- versioned;
- testable;
- observable.

---

## 40. Context Summarization

Summaries may be generated for long historical context.

The architecture must distinguish:

```text id="m7v3q5"
Original Source
    |
    v
Summary
```

and preserve provenance where the summary is used for consequential decisions.

---

## 41. Conversation Context

Conversational AI capabilities may use:

- current conversation;
- recent turns;
- summarized history;
- user preferences.

Conversation context must obey:

- retention policies;
- authorization;
- privacy rules;
- context limits.

Conversation history is not automatically domain state.

---

## 42. Memory Context

AI memory may provide persistent contextual information.

Memory must be treated separately from authoritative domain data.

Conceptually:

```text id="q8m4v2"
Domain State
    !=
AI Memory
```

AI memory must not silently override authoritative application data.

---

## 43. External Knowledge

External sources may be used where authorized.

External knowledge introduces:

- availability dependencies;
- freshness uncertainty;
- provenance requirements;
- provider-specific failures;
- security risks.

External information should therefore be clearly identified within the context model.

---

## 44. External Knowledge Trust

External knowledge should be classified according to trust.

Conceptually:

```text id="x5m7q2"
Trusted Internal Source
        >
Approved External Source
        >
Unverified External Source
```

Unverified external content must be treated as advisory.

---

## 45. Context Expiration

Context may become invalid after:

- source updates;
- permission changes;
- collection changes;
- model changes;
- capability changes;
- time expiration.

The architecture should support context expiration where required.

---

## 46. Context Caching

Context caching may improve performance.

Cache keys must account for:

- user/authorization scope;
- source version;
- capability;
- context requirements;
- freshness requirements.

Cross-user context leakage through caching is prohibited.

---

## 47. Context Invalidation

Invalidation should occur when relevant authoritative data changes.

Potential triggers:

```text id="n7m2q5"
Item Updated
Collection Updated
Permissions Changed
Reference Data Updated
Knowledge Source Updated
Context Policy Changed
```

Not every source requires immediate invalidation.

The invalidation strategy should reflect freshness requirements.

---

## 48. Context Cost Management

Large context increases:

- token usage;
- latency;
- cost;
- noise.

Context selection should therefore optimize for useful information rather than maximum information.

The architecture favors:

```text id="m4q8v2"
Relevant Context
```

over:

```text id="x7n3p5"
Maximum Context
```

---

## 49. Context Security

Security controls include:

- authorization;
- tenant isolation;
- sensitivity filtering;
- data minimization;
- source trust classification;
- provenance;
- injection resistance;
- secure caching.

Security must be applied before context enters the model boundary.

---

## 50. Prompt Injection Through Knowledge

Knowledge sources may contain malicious instructions.

The architecture must assume:

```text id="p8m4q7"
Retrieved Content
=
Untrusted Data
```

unless explicitly classified otherwise.

Prompt architecture must preserve instruction hierarchy.

---

## 51. Context and Prompt Boundary

The context architecture produces data.

The prompt architecture decides how that data is represented to the model.

```text id="v3q7m1"
Context Architecture
        |
        v
Authorized Context
        |
        v
Prompt Architecture
        |
        v
Model Request
```

Neither subsystem should absorb the other's responsibilities.

---

## 52. Context Assembly Components

Logical components may include:

```text id="q5m8v2"
Context Requirement Resolver
Context Source Registry
Context Provider
Authorization Filter
Context Retriever
Relevance Ranker
Context Normalizer
Context Assembler
Context Validator
Context Cache
Context Provenance Tracker
```

These are logical components and may be physically combined.

---

## 53. Context Provider

A Context Provider exposes application or domain information through an AI-safe representation.

It must:

- enforce scope;
- expose only required fields;
- preserve semantic meaning;
- provide provenance where appropriate.

---

## 54. Context Retriever

The Context Retriever obtains information from configured knowledge sources.

It must support:

- deterministic filtering;
- relevance criteria;
- authorization constraints;
- bounded result sets.

---

## 55. Context Ranker

The Context Ranker orders candidate context according to capability-specific relevance.

Ranking must be bounded.

The system must define a maximum number or size of context elements.

---

## 56. Context Validator

The Context Validator verifies:

- authorization;
- schema;
- completeness;
- size;
- sensitivity;
- source validity;
- freshness.

Invalid context must not be passed to the model.

---

## 57. Context Provenance Tracker

The Provenance Tracker associates context elements with their source.

This supports:

- traceability;
- evaluation;
- auditing;
- debugging.

---

## 58. Context Execution Contract

An AI capability should receive a conceptual context contract:

```text id="m7q3v8"
AIContext
├── Capability
├── Context Version
├── Entries
├── Source Metadata
├── Authorization Scope
├── Freshness Metadata
└── Provenance
```

The exact implementation is defined later.

---

## 59. Context Lifecycle

The context lifecycle is:

```text id="x2m8q4"
Requested
    |
    v
Resolved
    |
    v
Authorized
    |
    v
Retrieved
    |
    v
Filtered
    |
    v
Ranked
    |
    v
Normalized
    |
    v
Validated
    |
    v
Assembled
    |
    v
Consumed
    |
    v
Expired
```

Each stage may emit telemetry where appropriate.

---

## 60. Context Observability

Context-related telemetry should include metadata such as:

- capability;
- context version;
- source types;
- number of entries;
- retrieval latency;
- filtering counts;
- ranking strategy;
- context size;
- cache hit/miss;
- validation result.

Sensitive content should not be logged by default.

---

## 61. Context Debugging

When AI output is unexpected, the system should make it possible to determine:

```text id="q7m4v2"
What context was selected?
Which sources were used?
Which entries were excluded?
Which version was used?
What authorization scope applied?
```

This does not necessarily require storing raw context indefinitely.

Metadata and controlled trace reconstruction may be sufficient.

---

## 62. Context Quality Metrics

Relevant metrics may include:

- retrieval precision;
- retrieval recall;
- context relevance;
- context size;
- context truncation rate;
- cache hit rate;
- stale context rate;
- authorization rejection rate;
- context assembly latency.

These metrics support continuous improvement.

---

## 63. Context Testing

Testing should include:

### Authorization Tests

Verify restricted information never enters context.

### Relevance Tests

Verify relevant sources are selected.

### Freshness Tests

Verify stale information is rejected where required.

### Security Tests

Verify malicious retrieved content does not gain instruction authority.

### Boundary Tests

Verify context limits are respected.

### Regression Tests

Verify context changes do not unexpectedly degrade capability quality.

---

## 64. Context Fixtures

Context test fixtures should represent:

- minimal context;
- complete context;
- missing context;
- conflicting context;
- stale context;
- unauthorized context;
- malicious context;
- oversized context.

Fixtures must remain deterministic and sanitized.

---

## 65. Context Fallback

If an optional context source is unavailable:

```text id="m5q8v1"
Optional Source Failure
        |
        v
Continue with Reduced Context
```

If a mandatory source is unavailable:

```text id="x7n3q5"
Mandatory Source Failure
        |
        v
Capability Failure / Degraded Mode
```

The capability contract must define which behavior applies.

---

## 66. Context Availability

Context sources should have explicit availability requirements.

Examples:

```text id="p8m2v7"
Mandatory
Required for correctness.

Preferred
Used when available.

Optional
Improves result but is not required.

Forbidden
Must never be supplied.
```

This classification should be capability-specific.

---

## 67. Context and User Privacy

User-specific context must respect:

- user permissions;
- data minimization;
- retention policies;
- privacy requirements;
- tenant isolation.

AI capabilities must not use historical or personalized context simply because it is available.

---

## 68. Context and Multi-Tenancy

For multi-tenant scenarios, context retrieval must preserve tenant boundaries.

Conceptually:

```text id="q4m7n2"
Tenant A
   |
   v
Context Scope A

Tenant B
   |
   v
Context Scope B
```

Cross-tenant context leakage is a critical architectural violation.

---

## 69. Context and Authorization Changes

If authorization changes between retrieval and execution, the architecture must define whether the context remains valid.

For sensitive operations, authorization should be checked as close as possible to context consumption.

---

## 70. Context and Domain Consistency

AI context must not become an alternative representation of authoritative domain state without synchronization rules.

Where current domain state is required, context should be built from an authoritative source.

---

## 71. Context and AI Memory

AI memory may enrich context but must be subordinate to authoritative application information.

Precedence should generally be:

```text id="n5m7q3"
Authoritative Application State
        >
Approved Reference Knowledge
        >
AI Memory
        >
AI Inference
```

Capability-specific exceptions must be explicitly documented.

---

## 72. Context and External Tools

Tool results may become context.

However:

```text id="x8q2m4"
Tool Result
    |
    v
Validation / Trust Classification
    |
    v
Context
```

Tool output must not automatically become trusted knowledge.

---

## 73. Context and Structured Output

Context should be structured according to the output requirements where practical.

For example, if AI must classify an item against a fixed vocabulary, the relevant vocabulary should be represented in a controlled structure rather than as an uncontrolled text dump.

---

## 74. Context Optimization

Context optimization should aim to maximize:

```text id="r6m2v8"
Useful Information
------------------
Context Size
```

Optimization techniques may include:

- filtering;
- ranking;
- summarization;
- deduplication;
- compression;
- source prioritization;
- caching.

Any optimization that can change semantics must be evaluated.

---

## 75. Context Deduplication

Duplicate information should be removed where practical.

Duplication can:

- increase cost;
- increase latency;
- confuse the model;
- distort relevance.

Deduplication must preserve provenance where provenance matters.

---

## 76. Context Ordering

Ordering may affect model interpretation.

The architecture should therefore define deterministic ordering rules.

Possible ordering:

```text id="q5m8v2"
Critical Context
    |
    v
Current State
    |
    v
Relevant Supporting Information
    |
    v
Historical / Optional Information
```

---

## 77. Context Contracts and Evolution

Context contracts must be versioned when changes may affect AI behavior.

Changes include:

- field removal;
- field meaning change;
- source change;
- serialization change;
- ranking change;
- authorization policy change.

---

## 78. Knowledge Index Lifecycle

Knowledge indexes should follow a controlled lifecycle:

```text id="m8q3v5"
Source
  |
  v
Ingestion
  |
  v
Normalization
  |
  v
Indexing
  |
  v
Validation
  |
  v
Available
  |
  v
Updated / Rebuilt
  |
  v
Retired
```

Index lifecycle is distinct from AI model lifecycle.

---

## 79. Embedding Knowledge Lifecycle

For semantic knowledge:

```text id="p4m7x2"
Source Data
    |
    v
Embedding Model
    |
    v
Embedding Generation
    |
    v
Vector Index
```

Changing the embedding model may require regeneration of existing embeddings.

---

## 80. Knowledge Ingestion

Knowledge ingestion should define:

- source;
- ownership;
- authorization;
- transformation;
- validation;
- version;
- indexing strategy;
- update frequency;
- retention.

Uncontrolled ingestion is prohibited.

---

## 81. Knowledge Deletion

When source information must be deleted, the architecture must define how deletion propagates to:

- indexes;
- embeddings;
- caches;
- AI memory;
- derived knowledge.

Deletion requirements must respect privacy and retention policies.

---

## 82. Knowledge Provenance

Every indexed knowledge item should retain sufficient provenance to determine:

- source;
- source version;
- ingestion timestamp;
- transformation pipeline;
- indexing version.

This supports trustworthy retrieval.

---

## 83. Knowledge Quality Gates

Knowledge should pass quality checks before becoming available to AI.

Possible checks:

- schema validation;
- completeness;
- duplication;
- encoding;
- authorization;
- source integrity;
- freshness.

---

## 84. Knowledge Conflict Resolution

If multiple sources provide conflicting information, the knowledge layer should expose source precedence rather than silently merging contradictory values.

The AI should be able to distinguish:

```text id="q7m3v5"
Source A says X.
Source B says Y.
```

when the conflict cannot be deterministically resolved.

---

## 85. Context Failure Semantics

The context architecture should classify failures:

```text id="n5m8q2"
ContextUnavailable
ContextUnauthorized
ContextInvalid
ContextStale
ContextTooLarge
ContextSourceTimeout
ContextRetrievalFailure
```

The AI capability determines whether each failure is:

- fatal;
- degradable;
- retryable;
- ignorable.

---

## 86. Architectural Invariants

### INV-AI-CONTEXT-001

AI must never receive unrestricted application state.

### INV-AI-CONTEXT-002

Context must be assembled according to explicit capability requirements.

### INV-AI-CONTEXT-003

Authorization must be enforced before unauthorized information reaches the model.

### INV-AI-CONTEXT-004

Domain repositories must not be directly exposed to AI.

### INV-AI-CONTEXT-005

Context must be minimized to the information required by the capability.

### INV-AI-CONTEXT-006

Untrusted knowledge must not gain instruction authority.

### INV-AI-CONTEXT-007

Context sources must have explicit trust and freshness characteristics.

### INV-AI-CONTEXT-008

Context retrieval must respect tenant and ownership boundaries.

### INV-AI-CONTEXT-009

AI memory must not override authoritative application state.

### INV-AI-CONTEXT-010

Context used for consequential operations must be traceable.

### INV-AI-CONTEXT-011

Context limits must be explicit and enforced.

### INV-AI-CONTEXT-012

Changes to context architecture that materially affect AI behavior must be evaluable and reversible.

---

## 87. Decision Matrix

| Decision | Position |
|---|---|
| AI receives entire domain aggregate | Rejected |
| AI receives unrestricted database access | Rejected |
| Explicit context requirements | Required |
| Context Manager | Required |
| Application-controlled domain context | Required |
| Knowledge access contracts | Required |
| Authorization before model exposure | Required |
| Data minimization | Required |
| Semantic retrieval | Supported |
| Hybrid retrieval | Supported |
| AI memory as authoritative data | Rejected |
| External knowledge | Supported with governance |
| Context versioning | Required where consequential |
| Provenance | Required where relevant |
| Context caching | Supported |
| Cross-tenant context | Prohibited |
| Untrusted knowledge as instruction | Rejected |

---

## 88. Relationship with Other AI Documents

This document builds upon:

- `04_AI_DOMAIN_INTERACTION_AND_BOUNDARIES.md`
- `05_AI_APPLICATION_INTEGRATION.md`
- `06_AI_ARCHITECTURE_AND_COMPONENTS.md`
- `07_AI_MODEL_AND_PROVIDER_STRATEGY.md`
- `08_AI_PROMPTING_AND_INSTRUCTION_ARCHITECTURE.md`

It provides architectural input to:

- `10_AI_DATA_FLOWS_AND_INFORMATION_LIFECYCLE.md`
- `11_AI_MEMORY_AND_STATE_MANAGEMENT.md`
- `12_AI_TOOL_USE_AND_AGENT_BEHAVIOR.md`
- `13_AI_ORCHESTRATION_AND_WORKFLOW_ARCHITECTURE.md`
- `15_AI_OUTPUT_CONTRACTS_AND_STRUCTURED_RESPONSES.md`
- `16_AI_VALIDATION_AND_GUARDRAILS.md`
- `17_AI_SAFETY_SECURITY_AND_PRIVACY_BOUNDARIES.md`
- `18_AI_OBSERVABILITY_AND_TELEMETRY.md`
- `21_AI_EVALUATION_AND_QUALITY_STRATEGY.md`
- `25_AI_PERSISTENCE_AND_AI_DATA_STORAGE.md`

---

## 89. Implementation Boundary

The implementation should provide conceptual boundaries for:

```text id="v8m3q5"
Context Requirement Resolver
Context Source Registry
Context Provider
Knowledge Access Contract
Context Retriever
Authorization Filter
Relevance Ranker
Context Normalizer
Context Assembler
Context Validator
Context Cache
Context Provenance
Knowledge Index
Knowledge Ingestion
Knowledge Lifecycle
```

The physical project structure may consolidate components according to implementation constraints.

The implementation must not:

- expose repositories directly to AI;
- bypass authorization;
- mix trusted instructions with untrusted knowledge;
- use unlimited context;
- silently discard critical context;
- make AI memory authoritative over domain state.

---

## 90. Final Architectural Position

CollectionHub treats context as a controlled, capability-specific information boundary.

The canonical flow is:

```text id="m4q8v2"
Capability
    |
    v
Context Requirements
    |
    v
Authorized Sources
    |
    v
Retrieval
    |
    v
Filtering
    |
    v
Ranking
    |
    v
Normalization
    |
    v
Validation
    |
    v
Context Assembly
    |
    v
Prompt Architecture
    |
    v
AI Runtime
```

The architecture establishes a clear separation between:

```text id="q7n3m5"
Authoritative Application Data
        |
        v
Controlled Context
        |
        v
AI Interpretation
        |
        v
Validated Application Result
```

AI receives only the information required to perform its capability, within explicit authorization, freshness, relevance and security boundaries.

This preserves domain integrity, protects sensitive information, improves reproducibility and provides a foundation for semantic retrieval, AI memory, external knowledge and advanced AI capabilities without granting AI uncontrolled access to CollectionHub.

**Status:** Architectural baseline candidate.

**Next document:** `10_AI_DATA_FLOWS_AND_INFORMATION_LIFECYCLE.md`