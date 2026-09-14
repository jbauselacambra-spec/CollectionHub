# CollectionHub — Domain Glossary

> **Phase:** 2.2 — Domain Modeling
> **Artifact:** 01 — Domain Glossary
> **Status:** Draft / Consolidation Baseline
> **Depends on:** `00_DOMAIN_CONCEPT_INVENTORY.md`, `01_DOMAIN_CONCEPT_CLASSIFICATION.md`, `02_DOMAIN_IDENTITY_AND_INVARIANTS.md`, `04_DOMAIN_MODEL.md`, `Palabras Prohibidas.md`

---

# 1. Purpose

This document consolidates the **canonical technical vocabulary** used across the CollectionHub domain model.

It is not a duplicate of the Ubiquitous Language defined in `00_FOUNDATION` / `00_CONCEPTUAL`. That layer defines the **business** vocabulary (Collector, Portfolio, Collection, Collectible, Strategic Capital, etc.).

This glossary defines the **technical domain modeling vocabulary** — the terms used across `01_TECHNICAL/01_MODELING` to describe entities, aggregates, policies, and their relationships within the Collection/Item bounded context established in `03_AGGREGATE_BOUNDARY_ANALYSIS.md` and `04_DOMAIN_MODEL.md`.

Every term listed here **must** be used consistently across all subsequent modeling documents. A term not listed here should not be introduced into technical modeling documents without first being added here.

---

# 2. Relationship to Conceptual Vocabulary

```text
00_CONCEPTUAL / Palabras Prohibidas
        ↓  (business language)
01_TECHNICAL / 01_DOMAIN_GLOSSARY   ← this document
        ↓  (technical modeling language)
Entities / Value Objects / Aggregates / Policies / Events
```

Where a conceptual business term and a technical modeling term refer to the same underlying concept, the mapping is made explicit in Section 5.

The technical glossary must never contradict `Palabras Prohibidas.md`. Where the conceptual layer forbids a word (e.g. "Item", "User"), the technical layer may still use domain-modeling-specific terms (e.g. "Collection Item" as an aggregate name) **only** where an explicit mapping has already been established in `04_DOMAIN_MODEL.md` Section 35 ("Canonical Domain Language"). No new synonym may be introduced without updating that table.

---

# 3. Canonical Term Table

| Term | Category | Canonical Definition | Primary Source |
|---|---|---|---|
| Collection | Entity / Aggregate Root | Organizational unit containing collected Items; owns configuration and membership rules. | `04_DOMAIN_MODEL.md` |
| Collection Item | Entity / Aggregate Root | Individual object recorded within a Collection; independent identity and lifecycle. | `04_DOMAIN_MODEL.md` |
| Membership | Open (Entity / Value Object / Aggregate — unresolved) | The relationship by which a Collection Item is considered part of a Collection. | `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` D-011 |
| Item Type | Classification Concept | Classification of the kind of object an Item represents. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Attribute | Entity / Definition Candidate | Definition of a characteristic that may apply to an Item. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Attribute Value | Value Object | Concrete value assigned to an Attribute for a specific Item. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Tag | Classification Concept | Lightweight, reusable classification label. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Item Status | Value / State Concept | Current lifecycle state of an Item. | `14_DOMAIN_STATE_MACHINES.md` |
| Metadata | Supporting Concept | Descriptive information about an Item without independent identity. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| External Reference | Value Object Candidate | Association between a Collection Item and information maintained outside CollectionHub. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| External Identity | Domain Concept (ownership open) | The identity assigned to an Item by an external source; distinct from internal domain identity. | `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` D-013/D-014 |
| Source | Entity Candidate / Supporting Concept | Origin from which an External Reference was obtained. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Item Relationship | Relationship Concept (boundary open) | Meaningful domain association between two Collection Items. | `03_AGGREGATE_BOUNDARY_ANALYSIS.md` |
| Collection Owner | Role | Domain role responsible for a Collection; not automatically identical to a User. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| User | Entity / Actor | Individual interacting with CollectionHub; identity independent of authentication mechanism. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Collection Configuration | Configuration Concept | Rules governing which Item Types/Attributes are valid within a Collection. | `00_DOMAIN_CONCEPT_INVENTORY.md` |
| Aggregate | Modeling Category | A consistency boundary that groups the objects that must change together to preserve an invariant. | `00_STANDARDS/03_AGGREGATE_STANDARD.md` |
| Aggregate Root | Modeling Category | The single entry point through which an Aggregate may be mutated. | `00_STANDARDS/03_AGGREGATE_STANDARD.md` |
| Domain Invariant | Modeling Category | A condition that must always hold for the domain to be valid. | `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md` |
| Domain Policy | Modeling Category | A strategic business rule that determines how the domain decides what should happen under given conditions. | `00_STANDARDS/06_POLICY_STANDARD.md` |
| Specification | Modeling Category | A reusable domain predicate answering a single business question. | `00_STANDARDS/07_SPECIFICATION_STANDARD.md` |
| Domain Service | Modeling Category | Domain behavior that does not naturally belong to one Entity, Value Object or Aggregate. | `00_STANDARDS/04_DOMAIN_SERVICE_STANDARD.md` |
| Domain Event | Modeling Category | An immutable record of a fact that has already occurred in the domain. | `00_STANDARDS/05_DOMAIN_EVENT_STANDARD.md` |
| Repository | Modeling Category | A persistence contract for a single Aggregate Root, technology-independent. | `00_STANDARDS/08_REPOSITORY_STANDARD.md` |
| Use Case | Modeling Category | A meaningful business capability initiated by an actor or process; distinct from a command. | `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md` |
| Application Workflow | Modeling Category | The orchestration sequence an Application Service follows to fulfill a Use Case. | `12_APPLICATION_USE_CASES_AND_WORKFLOWS.md` |

---

# 4. Deliberately Undefined Terms

The following terms appear in candidate/provisional documents but are **intentionally not yet canonical**, pending resolution of the corresponding open question in `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md`:

| Term | Blocked By | Question ID |
|---|---|---|
| Membership Aggregate | Membership boundary decision | Q-002 |
| Classification (as independent concept) | Classification ownership | Q-006 |
| Synchronization Aggregate | Synchronization ownership | Q-009 |
| Collaborator | Collaborator model | Q-005 |
| Tenant | Multi-tenancy decision | Q-021 |

These terms **must not** be used as if settled in any document produced after this glossary. Where they must be referenced, they should be marked `(provisional)`.

---

# 5. Conceptual ↔ Technical Mapping

| Conceptual Term (Business) | Technical Term (Modeling) |
|---|---|
| Collectible | Collection Item |
| Collector | User (technical) / Collection Owner (role) |
| Portfolio | *(not yet mapped — Portfolio is a Phase-1 conceptual construct without a confirmed technical aggregate; see Open Question below)* |
| Collection (business, "LEGO Helmets") | Collection (technical aggregate) |

## Open Mapping Question

`Portfolio` is used extensively in `00_CONCEPTUAL` as the complete set of a Collector's Collections, but no technical aggregate for `Portfolio` has been defined in `01_TECHNICAL/01_MODELING`. This glossary flags it as an **unmapped conceptual term** requiring resolution before Strategic Capital / Strategic Reasoning features can be technically modeled.

---

# 6. Term Governance Rules

1. A term may only enter Section 3 once it has an approved source document.
2. A term must have exactly one canonical definition.
3. Renaming a canonical term requires updating every document that references it — this glossary is the index of that obligation, not a substitute for it.
4. Terms in Section 4 must be promoted to Section 3 only after their blocking open question is resolved in `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md`.

---

# 7. Status

**Current status:** Consolidation baseline — reflects the vocabulary already in use across existing modeling documents as of this writing.

This glossary does not introduce new domain concepts. It exists to prevent the divergence risk noted in the Domain Model Consistency Review (`19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md`, Section 6 "Vocabulary Consistency"), where the same concept was found to be referenced under slightly different names across documents.
