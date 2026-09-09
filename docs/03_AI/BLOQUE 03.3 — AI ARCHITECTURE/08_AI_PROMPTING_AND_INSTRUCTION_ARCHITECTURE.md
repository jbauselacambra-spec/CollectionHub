# 08 — AI Prompting and Instruction Architecture

## 1. Purpose

This document defines the architectural strategy for prompts, instructions, system directives, capability instructions, output instructions and prompt composition within CollectionHub.

Its purpose is to ensure that AI behavior is:

- explicit;
- versioned;
- traceable;
- testable;
- composable;
- provider-independent;
- secure;
- maintainable;
- separated from application and domain logic.

This document establishes:

- the instruction hierarchy;
- prompt ownership;
- prompt composition;
- prompt versioning;
- instruction sources;
- dynamic context insertion;
- output-format instructions;
- prompt validation;
- prompt security;
- prompt observability;
- prompt testing;
- prompt evolution;
- provider-specific prompt adaptation.

This document does not define the complete context/knowledge architecture or model/provider selection strategy.

---

## 2. Architectural Context

Prompting is an internal AI runtime concern.

The application expresses capability intent:

```text id="2f6n1a"
Application
    |
    v
AI Capability
```

The AI subsystem transforms that intent into model instructions:

```text id="q8m4v2"
AI Capability
    |
    v
Instruction Architecture
    |
    v
Prompt Composition
    |
    v
Model Gateway
```

The application must not construct provider-specific prompts directly.

---

## 3. Core Principle

The primary rule is:

> **Prompts are executable AI configuration and must be treated as versioned architectural artifacts, not as arbitrary strings embedded in application code.**

This means prompts must be:

- identifiable;
- versioned;
- testable;
- reviewable;
- traceable to capabilities;
- independent from provider SDKs;
- separated from domain rules.

---

## 4. Prompt Architecture Objectives

The prompt architecture must provide:

1. consistency across AI capabilities;
2. explicit instruction hierarchy;
3. controlled context injection;
4. structured output expectations;
5. provider portability;
6. prompt versioning;
7. reproducibility;
8. security against instruction injection;
9. evaluation support;
10. operational traceability.

---

## 5. Instruction Hierarchy

Instructions should be layered.

The conceptual hierarchy is:

```text id="m7v9k2"
Global AI Safety / Policy
        |
        v
CollectionHub AI System Instructions
        |
        v
Capability Instructions
        |
        v
Execution-Specific Instructions
        |
        v
Context
        |
        v
User / External Input
```

Higher-level instructions have precedence over lower-level content.

External input must never override system-level architectural or security constraints.

---

## 6. Instruction Categories

Instructions are classified into:

### 6.1 Global AI Instructions

Apply across the AI subsystem.

Examples:

- safety principles;
- output discipline;
- privacy requirements;
- general behavioral constraints.

### 6.2 Capability Instructions

Define the purpose and expected behavior of a specific capability.

Examples:

```text id="c4h7p8"
ItemClassification
MetadataSuggestion
DuplicateDetection
```

### 6.3 Execution Instructions

Define requirements for a specific execution.

Examples:

- output schema;
- requested language;
- confidence requirements;
- execution mode.

### 6.4 Context

Context provides information.

It is not automatically an instruction.

This distinction is important for preventing external data from being interpreted as authoritative system directives.

---

## 7. Instruction vs Context

The architecture must explicitly distinguish:

```text id="v5q2n8"
Instruction
    -> "Classify the item."

Context
    -> "Item title: ..."
```

Context may contain text that looks like an instruction.

That text must not automatically acquire instruction authority.

This distinction is a fundamental defense against prompt injection.

---

## 8. Prompt Composition

Prompt composition should follow a deterministic process:

```text id="f9m2r4"
Capability
    |
    v
Load Instruction Set
    |
    v
Resolve Version
    |
    v
Insert Authorized Context
    |
    v
Insert Execution Parameters
    |
    v
Apply Output Contract
    |
    v
Validate Prompt
    |
    v
Final Model Request
```

Prompt composition should be reproducible for the same:

- capability version;
- instruction version;
- context version;
- execution parameters;
- model contract.

---

## 9. Prompt Components

A composed prompt may contain:

```text id="b6n8q3"
Prompt
├── System Instructions
├── Capability Instructions
├── Task Instructions
├── Context
├── Examples
├── Constraints
├── Output Contract
└── User Input
```

Not every capability requires every component.

Prompt complexity must remain proportional to capability requirements.

---

## 10. System Instructions

System instructions establish the highest-level behavioral contract.

They should define stable rules such as:

- role;
- safety boundaries;
- output discipline;
- general response constraints;
- non-disclosure requirements;
- instruction precedence.

System instructions must not contain volatile application data.

---

## 11. Capability Instructions

Capability instructions define what the AI capability is expected to accomplish.

Example conceptual definition:

```text id="m3w7p2"
Capability:
    ItemClassification

Instruction:
    Determine the most appropriate category
    using the supplied item information.
```

Capability instructions must remain independent from provider-specific syntax.

---

## 12. Task Instructions

Task instructions specify execution-specific requirements.

Examples:

```text id="g8r2v5"
Return one classification.
Return structured output.
Do not infer missing information as fact.
```

Task instructions may be dynamically generated from application-level execution requirements.

---

## 13. Output Instructions

Output instructions describe how the model should represent its result.

Where structured output is required, instructions should specify:

- expected fields;
- data types;
- allowed values;
- nullability;
- confidence representation;
- uncertainty representation.

Output instructions complement formal schema validation.

They do not replace it.

---

## 14. Examples and Few-Shot Guidance

Examples may be used where they improve capability performance.

Examples must be:

- versioned;
- representative;
- reviewed;
- safe;
- relevant to the capability;
- free from sensitive production data unless explicitly authorized.

Examples must not silently become business rules.

---

## 15. Prompt Templates

Prompt templates should represent reusable instruction structures.

Conceptually:

```text id="d2m6p9"
Prompt Template
    |
    +--> System Instructions
    +--> Capability Instructions
    +--> Context Slots
    +--> Output Contract
```

Templates should expose explicit placeholders rather than rely on uncontrolled string concatenation.

---

## 16. Prompt Variables

Prompt variables should be explicitly declared.

Example:

```text id="p8x4n2"
{{item_title}}
{{item_description}}
{{collection_context}}
{{requested_language}}
```

Each variable should have:

- source;
- type;
- optionality;
- maximum size;
- sensitivity classification;
- escaping/serialization rules.

---

## 17. Context Injection

Dynamic context must be inserted through a controlled mechanism.

Conceptual flow:

```text id="q5m7v1"
Application Context
        |
        v
Context Resolver
        |
        v
Authorized Context
        |
        v
Prompt Variable
```

The prompt engine must not query arbitrary databases while composing prompts.

---

## 18. Data Serialization

Structured data should be serialized deterministically.

For example:

```text id="x2r9k6"
Collection Item
    |
    v
Canonical Representation
    |
    v
Prompt Context
```

This reduces ambiguity and improves reproducibility.

---

## 19. Prompt Size Management

Prompt composition must respect model context limits.

The system should control:

- maximum context size;
- maximum examples;
- maximum conversation history;
- maximum retrieved documents;
- maximum generated instruction size.

Context truncation must be deliberate and observable.

Silent truncation is prohibited where it could materially affect capability behavior.

---

## 20. Prompt Prioritization

When context exceeds available capacity, the architecture should prioritize:

1. mandatory system instructions;
2. mandatory capability instructions;
3. required output contract;
4. critical context;
5. relevant supporting context;
6. optional examples;
7. historical or low-priority context.

Lower-priority information must be removed before mandatory instructions.

---

## 21. Prompt Versioning

Every production prompt or instruction set must have a version.

Conceptually:

```text id="f4m8q2"
Capability
    |
    v
Prompt Version
    |
    v
Model Execution
```

Prompt changes must be distinguishable from:

- model changes;
- context changes;
- application changes.

---

## 22. Prompt Identity

A prompt identity should allow an execution to be associated with:

```text id="a8q5m1"
Capability
Instruction Set
Prompt Template
Prompt Version
Execution Configuration
```

This enables later reconstruction of the execution configuration.

---

## 23. Prompt Change Management

Prompt changes must follow controlled change management.

Changes should be evaluated for:

- quality impact;
- safety impact;
- output contract compatibility;
- cost;
- latency;
- regression risk.

Prompt changes that materially affect behavior must be evaluated before production rollout.

---

## 24. Prompt and Model Independence

Prompt architecture must not assume a single model.

However, models may have different instruction characteristics.

Therefore:

```text id="v7n3m5"
Canonical Capability Instructions
          |
          v
Model Adaptation Layer
          |
          v
Provider / Model Specific Representation
```

Provider-specific adaptations should remain isolated.

---

## 25. Model-Specific Prompt Adaptation

Model-specific adaptation may include:

- instruction formatting;
- role mapping;
- structured-output syntax;
- tool-call formatting;
- modality handling.

These adaptations belong below the capability-level instruction architecture.

They must not leak into application code.

---

## 26. Prompt Security

Prompt security protects the instruction hierarchy from unauthorized manipulation.

Threats include:

- prompt injection;
- indirect prompt injection;
- malicious context;
- malicious retrieved documents;
- user attempts to override system instructions;
- tool output containing instructions.

The architecture must treat external content as untrusted unless explicitly classified otherwise.

---

## 27. Prompt Injection Boundary

The conceptual trust boundary is:

```text id="r4m6x2"
Trusted Instructions
        |
        +--> System
        +--> Capability
        +--> Execution Policy
        |
        v
Instruction Boundary
        |
        v
Untrusted Content
        |
        +--> User Input
        +--> Retrieved Content
        +--> External Documents
        +--> Tool Output
```

Untrusted content must not be promoted to trusted instruction authority.

---

## 28. Instruction Precedence

The system must define deterministic precedence.

Conceptually:

```text id="z8q2m4"
Security / System Policy
        >
Capability Instructions
        >
Execution Constraints
        >
User Request
        >
Retrieved / External Content
```

Where a conflict exists, the higher-level instruction wins.

---

## 29. User Input

User input is an input to the capability, not a system-level instruction.

The model may be asked to interpret the user's intent, but user content must not override:

- security policies;
- capability boundaries;
- output contracts;
- tool restrictions;
- privacy policies.

---

## 30. Retrieved Content

Retrieved information may contain adversarial or misleading instructions.

Therefore retrieved content should be explicitly represented as data.

Conceptual pattern:

```text id="k7p3v9"
Retrieved Document
    |
    v
<DATA>
...
</DATA>
```

The exact representation may vary by model, but the architectural distinction must remain.

---

## 31. Tool Output

Tool output is also untrusted data unless explicitly designated otherwise.

A tool returning:

```text id="w3m8q1"
"Ignore previous instructions..."
```

must not automatically change AI execution policy.

Tool results should be treated as information.

---

## 32. Prompt Guardrails

Guardrails should operate both before and after prompt composition.

### Before Composition

Validate:

- capability;
- context authorization;
- variable values;
- input size;
- policy constraints.

### After Composition

Validate:

- required instructions present;
- prohibited content absent;
- context within limits;
- output contract included;
- model constraints satisfied.

---

## 33. Prompt Validation

Prompt validation should verify structural correctness.

Example:

```text id="f6n2r8"
Prompt Validator
├── Required Sections
├── Required Variables
├── Maximum Size
├── Instruction Integrity
├── Context Classification
├── Output Contract
└── Policy Constraints
```

A prompt that fails mandatory validation must not be sent to the model.

---

## 34. Prompt Compilation Concept

The prompt engine may be conceptually treated as a compiler:

```text id="c5v7m2"
Capability Definition
        |
        v
Instruction Set
        |
        v
Context
        |
        v
Execution Parameters
        |
        v
Prompt Compiler
        |
        v
Model Request
```

This analogy reinforces that prompt generation is a controlled transformation, not arbitrary text assembly.

---

## 35. Prompt Determinism

Given equivalent inputs and configuration, prompt composition should be deterministic.

The same:

```text id="q2m8v5"
Capability Version
Instruction Version
Context Version
Execution Configuration
```

should produce the same logical prompt representation.

Provider-specific serialization may differ.

---

## 36. Prompt Observability

AI execution telemetry should record prompt metadata without necessarily recording the entire prompt.

Recommended metadata:

- capability;
- prompt identifier;
- prompt version;
- template identifier;
- model;
- provider;
- context version;
- execution identifier.

Full prompt capture should only occur where permitted by privacy and security policy.

---

## 37. Sensitive Prompt Data

Prompts may contain sensitive information.

The architecture must therefore distinguish:

```text id="r7v2n5"
Prompt Metadata
```

from:

```text id="m4q8x1"
Prompt Content
```

Telemetry should prefer metadata.

Sensitive prompt content must not be logged by default.

---

## 38. Prompt Testing

Prompt testing should include:

### Structural Tests

Verify:

- required sections;
- variables;
- output instructions.

### Behavioral Tests

Verify:

- expected capability behavior;
- refusal behavior;
- edge cases.

### Security Tests

Verify resistance to:

- instruction override;
- prompt injection;
- malicious context.

### Regression Tests

Compare results against approved evaluation datasets.

---

## 39. Prompt Fixtures

Prompt tests should use controlled fixtures.

Fixtures should be:

- versioned;
- deterministic;
- representative;
- sanitized;
- reusable.

Production data should not become a test fixture automatically.

---

## 40. Prompt Evaluation

Prompt changes should be evaluated against capability-specific datasets.

Conceptual flow:

```text id="x9m4q7"
Prompt V1
    |
    v
Evaluation Dataset
    |
    v
Metrics

Prompt V2
    |
    v
Evaluation Dataset
    |
    v
Metrics
```

Changes should be approved based on measured impact rather than subjective inspection alone.

---

## 41. Prompt Rollback

Prompt versions must be reversible.

A production capability should be able to return to a previously approved instruction version.

Rollback should be possible independently from unrelated application deployment where practical.

---

## 42. Prompt Deployment

Prompt deployment should support controlled environments.

Conceptually:

```text id="h5v8q2"
Draft
  |
  v
Evaluated
  |
  v
Approved
  |
  v
Staged
  |
  v
Production
```

Production prompt versions must be identifiable.

---

## 43. Prompt Ownership

Each prompt or instruction set must have an identified owner.

Ownership responsibilities include:

- semantic correctness;
- change review;
- evaluation;
- versioning;
- documentation;
- rollback readiness.

Prompt ownership must not become an excuse for embedding business rules into prompts.

---

## 44. Business Rules vs Prompt Instructions

Business rules belong in the domain or application architecture.

For example:

```text id="j8r4p2"
Domain Rule:
    A collection item must have a valid owner.
```

should not become:

```text id="n6w2q9"
Prompt:
    Please remember that every item must have an owner.
```

The prompt may instruct AI to respect known application constraints, but it must not become the authoritative source of those constraints.

---

## 45. Prompt-Based Reasoning Boundaries

Prompts may guide reasoning.

They must not be used as a substitute for deterministic validation.

Preferred:

```text id="s5m9v3"
Prompt Guidance
      |
      v
AI Output
      |
      v
Deterministic Validation
```

Not:

```text id="q7x2m4"
Prompt Guidance
      |
      v
Assumed Business Truth
```

---

## 46. Output Schema Integration

Prompting and structured output are related but distinct.

The architecture should combine:

```text id="b9m4q7"
Prompt Output Instructions
        |
        +
        v
Formal Output Schema
        |
        v
Output Validator
```

Prompt instructions improve compliance.

Formal schema validation provides enforcement.

---

## 47. Language Instructions

Where capabilities support multiple languages, language requirements should be explicit.

The language may be determined by:

- user preference;
- application context;
- capability configuration;
- requested output language.

Language selection must not alter safety or structural requirements.

---

## 48. Localization

Localized instructions should preserve semantic equivalence.

The architecture should avoid maintaining independently diverging versions of capability logic for each language.

Preferred:

```text id="p4m7x2"
Capability Definition
        |
        +--> Language Representation
        |
        +--> Shared Output Contract
```

---

## 49. Prompt Composition Pipeline

The canonical pipeline is:

```text id="f2n7m4"
Capability Request
        |
        v
Resolve Prompt Version
        |
        v
Load Instruction Set
        |
        v
Validate Execution Parameters
        |
        v
Resolve Authorized Context
        |
        v
Insert Context
        |
        v
Insert Task Parameters
        |
        v
Insert Output Contract
        |
        v
Apply Model Adaptation
        |
        v
Validate Final Prompt
        |
        v
Model Gateway
```

---

## 50. Prompt Runtime Components

The logical prompting subsystem may contain:

```text id="m8q4v2"
Prompt Registry
Prompt Resolver
Instruction Composer
Template Engine
Context Inserter
Prompt Validator
Model Adaptation Layer
Prompt Version Store
Prompt Telemetry
```

These are logical components.

They do not necessarily require separate physical projects.

---

## 51. Prompt Registry

The Prompt Registry identifies available instruction sets.

It should provide:

- capability mapping;
- version information;
- status;
- ownership;
- compatibility metadata.

It must not expose provider credentials or sensitive runtime state.

---

## 52. Prompt Resolver

The Prompt Resolver determines which instruction version should be used.

Inputs may include:

- capability;
- environment;
- feature flag;
- model compatibility;
- rollout state.

The resolver must return an approved instruction set.

---

## 53. Instruction Composer

The Instruction Composer assembles the logical instruction hierarchy.

Responsibilities:

- ordering;
- precedence;
- composition;
- variable binding;
- instruction integrity.

It should remain independent from provider SDKs.

---

## 54. Template Engine

The Template Engine renders prompt templates using validated variables.

It must prevent:

- malformed interpolation;
- uncontrolled object serialization;
- accidental instruction promotion;
- variable omission.

---

## 55. Prompt Validator

The Prompt Validator is responsible for final prompt integrity.

Validation failures should stop execution.

The validator should produce provider-neutral errors.

---

## 56. Model Adaptation Layer

The Model Adaptation Layer translates canonical prompt structures into model-compatible representations.

It may handle:

- message roles;
- structured output configuration;
- tool-call formatting;
- modality-specific content;
- model-specific limitations.

It belongs below the canonical prompt architecture.

---

## 57. Prompt Registry Lifecycle

Prompt artifacts should follow:

```text id="n4m7q2"
Draft
  |
  v
Reviewed
  |
  v
Evaluated
  |
  v
Approved
  |
  v
Active
  |
  v
Deprecated
```

Only approved prompts may become active production versions.

---

## 58. Prompt Metadata

Each prompt artifact should have conceptual metadata:

```text id="q8v3m5"
PromptId
Capability
Version
Status
Owner
CreatedAt
ApprovedAt
ModelCompatibility
EvaluationReference
```

This metadata enables traceability.

---

## 59. Prompt Compatibility

A prompt version may have compatibility constraints.

Examples:

- minimum model capability;
- structured-output support;
- tool support;
- context requirements;
- modality requirements.

Compatibility must be checked before execution.

---

## 60. Prompt and Context Versioning

For reproducibility, prompt and context versions should be independently identifiable.

Example:

```text id="x4m8q2"
Execution
├── Capability: ItemClassification
├── Prompt: v3
├── Context: v7
├── Model: X
└── Provider: Y
```

This distinction is essential when investigating behavioral changes.

---

## 61. Prompt Cache

Prompt compilation may be cached where safe.

Cache keys should account for:

- prompt version;
- template version;
- relevant execution parameters;
- model compatibility.

Dynamic context should not be accidentally reused across users or executions.

---

## 62. Prompt Reuse

Reusable instruction fragments may be shared.

However, excessive reuse can create hidden coupling.

Shared fragments should therefore have:

- explicit ownership;
- versioning;
- compatibility rules;
- impact analysis.

---

## 63. Prompt Fragment Architecture

A reusable instruction system may contain:

```text id="c8m2q6"
Global Fragment
     |
     +--> Capability Fragment
             |
             +--> Task Fragment
                     |
                     +--> Output Fragment
```

The architecture must preserve clear precedence.

---

## 64. Prompt Fragment Risks

Shared fragments can affect many capabilities.

Therefore a fragment change should trigger impact analysis.

A globally reused fragment should not be changed casually.

---

## 65. Prompt Security Invariants

The following rules are mandatory:

### INV-AI-PROMPT-001

Untrusted content must not override trusted instructions.

### INV-AI-PROMPT-002

Prompt versions must be identifiable.

### INV-AI-PROMPT-003

Production prompts must be approved.

### INV-AI-PROMPT-004

Provider-specific prompt formatting must remain below the canonical prompt boundary.

### INV-AI-PROMPT-005

Business rules must not be implemented exclusively through prompts.

### INV-AI-PROMPT-006

Prompt variables must be explicitly defined.

### INV-AI-PROMPT-007

Prompt composition must respect context limits.

### INV-AI-PROMPT-008

Sensitive prompt content must not be logged by default.

### INV-AI-PROMPT-009

Prompt changes must be evaluated when they materially affect capability behavior.

### INV-AI-PROMPT-010

Prompt rollback must be possible for production capabilities.

---

## 66. Decision Matrix

| Decision | Position |
|---|---|
| Prompts embedded throughout application code | Rejected |
| Centralized prompt architecture | Required |
| Versioned prompts | Required |
| Provider-specific prompts | Isolated |
| Capability-specific instructions | Required |
| Untrusted content as instructions | Rejected |
| Prompt as source of business truth | Rejected |
| Formal output validation | Required |
| Prompt evaluation | Required |
| Prompt rollback | Required |
| Prompt metadata | Required |
| Sensitive prompt logging | Restricted |
| Dynamic context insertion | Supported |
| Uncontrolled string concatenation | Rejected |

---

## 67. Relationship with Other AI Documents

This document builds upon:

- `00_AI_FOUNDATION_AND_SCOPE.md`
- `02_AI_CAPABILITIES_AND_RESPONSIBILITIES.md`
- `06_AI_ARCHITECTURE_AND_COMPONENTS.md`
- `07_AI_MODEL_AND_PROVIDER_STRATEGY.md`

It provides architectural input to:

- `09_AI_CONTEXT_AND_KNOWLEDGE_ARCHITECTURE.md`
- `10_AI_DATA_FLOWS_AND_INFORMATION_LIFECYCLE.md`
- `12_AI_TOOL_USE_AND_AGENT_BEHAVIOR.md`
- `15_AI_OUTPUT_CONTRACTS_AND_STRUCTURED_RESPONSES.md`
- `16_AI_VALIDATION_AND_GUARDRAILS.md`
- `17_AI_SAFETY_SECURITY_AND_PRIVACY_BOUNDARIES.md`
- `18_AI_OBSERVABILITY_AND_TELEMETRY.md`
- `21_AI_EVALUATION_AND_QUALITY_STRATEGY.md`
- `22_AI_TESTING_AND_VERIFICATION_STRATEGY.md`

---

## 68. Implementation Boundary

The implementation should provide conceptual components for:

```text id="v7m3q9"
Prompt Registry
Prompt Resolver
Prompt Template Store
Instruction Composer
Prompt Variable Resolver
Context Inserter
Prompt Validator
Model Adaptation
Prompt Version Management
Prompt Evaluation Metadata
```

The physical implementation may consolidate components where appropriate.

The implementation must not:

- embed provider-specific prompts in application use cases;
- use prompts as a substitute for domain validation;
- allow arbitrary context injection;
- silently override instruction precedence;
- log sensitive prompt contents indiscriminately.

---

## 69. Final Architectural Position

CollectionHub treats prompting as a governed runtime architecture rather than ad-hoc text generation.

The canonical flow is:

```text id="j3m7q2"
Capability
    |
    v
Prompt Version
    |
    v
Instruction Hierarchy
    |
    v
Authorized Context
    |
    v
Execution Parameters
    |
    v
Output Contract
    |
    v
Model Adaptation
    |
    v
Model Gateway
```

The architecture establishes a strict separation between:

```text id="f8q4m1"
Trusted Instructions
        |
        v
AI Behavior Definition

and

Untrusted Content
        |
        v
Information Provided to AI
```

This provides CollectionHub with a prompt architecture that is versioned, testable, secure, provider-independent and operationally traceable.

Prompts guide AI behavior, but deterministic application and domain validation remain authoritative.

**Status:** Architectural baseline candidate.

**Next document:** `09_AI_CONTEXT_AND_KNOWLEDGE_ARCHITECTURE.md`