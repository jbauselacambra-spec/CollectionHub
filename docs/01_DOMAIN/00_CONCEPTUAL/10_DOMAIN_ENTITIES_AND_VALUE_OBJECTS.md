# CollectionHub — Domain Entities and Value Objects

> **Phase:** 2.2 — Domain Modeling
> **Artifact:** 10 — Domain Entities and Value Objects
> **Status:** Draft / Consolidation Baseline
> **Depends on:** `01_DOMAIN_GLOSSARY.md`, `04_DOMAIN_MODEL.md`, `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`, `20B_DOMAIN_MODEL_P0_DECISION_RESOLUTION.md`, `00_STANDARDS/01_ENTITY_STANDARD.md`, `00_STANDARDS/02_VALUE_OBJECT_STANDARD.md`

---

# 1. Purpose

This document consolidates the definitive list of **Entities** and **Value Objects** within the Collection/Item bounded context, incorporating the P0 decisions resolved in `20B_DOMAIN_MODEL_P0_DECISION_RESOLUTION.md`.

It does not redefine the classification methodology (see `01_DOMAIN_CONCEPT_CLASSIFICATION.md`) — it applies it, now that the previously open boundary questions (Membership, Item identity, External Identity) have provisional resolutions.

---

# 2. Entities

An Entity is included here only if it has stable identity independent of its mutable attributes, per `00_STANDARDS/01_ENTITY_STANDARD.md`.

## 2.1 Collection (Aggregate Root)

- **Identity:** `CollectionId` (system-assigned, stable).
- **Owns:** Name, Description, Configuration, Membership (internal entities), lifecycle state.
- **Invariants protected:** `INV-COLLECTION-001..008`.
- **Source:** `04_DOMAIN_MODEL.md` §5; confirmed by `20B_...md` P0-1.

## 2.2 Collection Item (Aggregate Root)

- **Identity:** `ItemId` (system-assigned, stable — confirmed by `20B_...md` P0-2).
- **Owns:** intrinsic metadata, classification, provenance, lifecycle state, `ExternalReference` value objects.
- **Invariants protected:** `INV-ITEM-001..005`.
- **Source:** `04_DOMAIN_MODEL.md` §9.

## 2.3 Membership (Internal Entity of Collection)

- **Identity:** none independent of its owning Collection; conceptually identified by `(CollectionId, ItemId)`.
- **Owns:** reference to `ItemId`, membership metadata (position, addedAt — extensible per future requirement).
- **Not independently persisted; not repository-addressable.**
- **Source:** `20B_...md` P0-1 (resolves prior OPEN status in `09_DOMAIN_AGGREGATES...md` §44 OPEN-AGG-001).

## 2.4 User (Actor, outside Collection/Item Aggregate boundary)

- **Identity:** `UserId`, independent of mutable profile data.
- **Note:** Not part of the Collection or Item consistency boundary. Included here only because it is referenced by `Collection Owner` (role). Its own Aggregate boundary is out of scope for this bounded context.
- **Source:** `00_DOMAIN_CONCEPT_INVENTORY.md` DC-007.

## 2.5 Attribute (Entity Candidate — status unchanged)

- **Identity:** provisional; depends on whether Attributes are collection-scoped or global (`20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` — not a P0 item, remains open).
- Included here as a **candidate** only. Must not be treated as ratified.

---

# 3. Value Objects

A Value Object is included here only if equality is fully determined by its value, per `00_STANDARDS/02_VALUE_OBJECT_STANDARD.md`.

## 3.1 CollectionName

- **Value definition:** normalized string satisfying `RULE-COL-NAME-001..004` (`06_DOMAIN_POLICIES_AND_RULES.md`).
- **Invariant:** non-empty; length and normalization rules remain open (`RULE-COL-NAME-002/003/004`).

## 3.2 ItemIdentifier

- Wraps `ItemId` as an explicit domain type rather than a raw primitive.
- **Equality:** value-based on the underlying identifier.

## 3.3 ExternalReference

- **Value definition:** `(Source, ExternalIdentifier)`.
- **Invariant:** uniqueness scoped per owning Item (`20B_...md` P0-3) — **not** globally unique.
- **Owned by:** `Collection Item` (zero or more).
- **Must never become domain identity** (`INV-ITEM-004`).

## 3.4 ItemStatus

- **Value definition:** one of the states defined by the (currently minimal) Item lifecycle in `14_DOMAIN_STATE_MACHINES.md` §9–10.
- **Note:** the full Item lifecycle state machine remains intentionally unresolved beyond `exists / valid / mutable`; this Value Object must not be assumed to contain `ACTIVE`/`ARCHIVED` until that state machine is formally ratified.

## 3.5 AttributeValue

- **Value definition:** `(AttributeId, Value)` pair, meaningful only in the context of `(Item, Attribute)`.
- **No independent lifecycle** (`04_DOMAIN_MODEL.md` §10.3).

## 3.6 MembershipMetadata

- **Value definition:** extensible structure attached to a Membership internal entity (position, addedAt, notes).
- **Owned by:** Membership, which is itself owned by Collection (P0-1).

---

# 4. What Remains Explicitly Unclassified

The following concepts from `00_DOMAIN_CONCEPT_INVENTORY.md` are **not** promoted to Entity or Value Object status by this document, consistent with the "deliberately undefined terms" listed in `01_DOMAIN_GLOSSARY.md` §4:

```text
Item Type        — classification ownership open (Q-006)
Tag               — scope open (global vs. collection-specific)
Source            — Entity vs. simple value undecided (04_DOMAIN_MODEL.md §18)
Item Relationship — Aggregate boundary undecided (03_AGGREGATE_BOUNDARY_ANALYSIS.md §20)
```

Any document that treats these as settled Entities or Value Objects without an explicit new decision record should be considered inconsistent with the current model.

---

# 5. Traceability

```text
Entity/VO
    ↓
Aggregate boundary (09_DOMAIN_AGGREGATES...md, confirmed by 20B_...md)
    ↓
Invariant (08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md)
    ↓
Standard (00_STANDARDS/01_ENTITY_STANDARD.md or 02_VALUE_OBJECT_STANDARD.md)
```

---

# 6. Status

**Current status:** Consolidation baseline, reflecting the domain model **after** P0 resolution. This document should be revisited immediately if Q-006 (Classification), Q-009 (Synchronization) or the Item Relationship boundary question are resolved, since those resolutions would each add or reclassify entries in Section 2/3.
