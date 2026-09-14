# CollectionHub — Domain Model P0 Decision Resolution

> **Phase:** 2.2 — Domain Modeling
> **Artifact:** 20B — P0 Decision Resolution
> **Status:** Proposed Resolution / Pending Ratification
> **Depends on:** `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md`, `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`, `19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md`

---

# 1. Purpose

`20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` (Section 71, "Decision Priority") identifies four questions as **P0 — blocks coherent domain model**:

```text
Q-002   Membership boundary
Q-015   Item identity / duplicate semantics
Q-007   External Identity cardinality
Q-014   Item eligibility (dependent on Membership)
```

Downstream architecture documents (persistence, schema, EF Core mapping) already assume answers to these questions without those answers having been formally recorded. This document closes that gap by providing an explicit, ratifiable resolution for each P0 question, following the Decision Resolution Protocol defined in `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` Section 72.

This document does **not** invent new domain concepts. It selects among the alternatives already identified in the source documents and states the rationale, consequences, and residual open items for each.

---

# 2. Decision Format

Each decision follows the protocol already established:

```text
Business requirement
Domain concepts involved
Affected invariants
Affected aggregates
Alternatives considered
Selected option
Consequences
Status / Confidence
```

---

# 3. Decision P0-1 — Membership Boundary (resolves Q-002, D-011)

## Business requirement

A Collection must be able to determine, unambiguously and atomically, which Items it contains, and must prevent the same Item from being added twice.

## Alternatives considered (from `20_...md` and `09_...md`)

```text
Option A — Membership as internal Entity of Collection
Option B — Membership as simple identity reference (ItemId list)
Option C — Membership as independent Aggregate
```

## Evaluation

Membership currently has:

- **no independent identity** requirement demonstrated by any use case;
- **no independent lifecycle** beyond "present / absent" (`14_DOMAIN_STATE_MACHINES.md` Section 12–20);
- **candidate attributes** (position, acquisition info, notes, source) that are anticipated but not yet required by any approved use case (`09_DOMAIN_AGGREGATES...md` Section 6).

Per Aggregate Boundary Test (`19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md` Section 11): an object earns Aggregate status only when it has an invariant, behavior, independent lifecycle, and requires independent transactional consistency. Membership satisfies none of these independently of Collection.

## Selected Option

**Option A — Membership is an internal entity of the `Collection` Aggregate**, referencing the Item by identity (`ItemId`) only.

```text
Collection Aggregate
    |
    +-- Membership (internal entity)
              |
              +-- ItemId (reference)
              +-- MembershipMetadata (position, addedAt, notes — extensible)
```

## Consequences

- `Collection` remains the sole consistency boundary responsible for the "no duplicate membership" invariant (`INV-COLLECTION-005`).
- `Membership` does not receive its own repository (per `08_REPOSITORIES/README.md`, "one Repository per Aggregate").
- If a future use case requires Membership to be queried, modified, or reasoned about independently of its Collection (e.g. cross-collection membership analytics), this decision must be reopened — it is **not** treated as permanent.
- `INV-COLLECTION-004/005/006/008` (`08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`) are confirmed as Collection-owned without modification.

## Status

**RESOLVED (provisional-strong).** Confidence: **HIGH**. Supersedes D-011/D-012 `OPEN` status → `ACCEPTED`.

---

# 4. Decision P0-2 — Item Domain Identity / Duplicate Semantics (resolves Q-015, D-010)

## Business requirement

The domain must be able to say definitively whether two Item records represent "the same logical item," in order to enforce `INV-COLLECTION-005` (no duplicate membership) and `INV-ITEM-004` (external identifiers are not automatically domain identity).

## Alternatives considered

```text
Option A — Internal domain identity only (system-generated identifier)
Option B — Composite identity (attributes-based)
Option C — External-identity-derived identity
```

## Evaluation

`02_DOMAIN_IDENTITY_AND_INVARIANTS.md` (D-INV-002, D-INV-003) already establishes that Item identity is independent of mutable attributes and that external identifiers must not define internal identity. Option B and C are therefore already excluded by prior accepted decisions — this question is really about **confirming** that exclusion rather than reopening it.

## Selected Option

**Option A — Item domain identity is a system-assigned internal identifier (`ItemId`), generated at creation time, immutable for the life of the Item.**

"Same logical item" for the purpose of `INV-COLLECTION-005` means: **same `ItemId`**. Two Items with identical attributes but different `ItemId` are, by domain definition, different Items — this is a deliberate simplification that avoids requiring fuzzy/semantic duplicate detection inside the core domain (semantic duplicate *detection* remains available as an AI-assisted advisory capability per `03_AI` UC-AI-021, but its result is never authoritative for the Membership invariant).

## Consequences

- `BR-MEMBERSHIP-002` ("Duplicate Add") is resolved: duplicate detection is keyed on `ItemId`, not on attribute similarity.
- External identity conflicts (two Items claiming the same external reference) are a **separate** concern, addressed by Decision P0-3 below, and must not be conflated with domain-identity duplication.

## Status

**RESOLVED.** Confidence: **HIGH**.

---

# 5. Decision P0-3 — External Identity Cardinality and Ownership (resolves Q-007, Q-008, D-014)

## Business requirement

CollectionHub may need to associate an Item with identifiers from one or more external sources (catalogues, marketplaces) without those identifiers becoming or overriding the internal domain identity.

## Alternatives considered

```text
Option A — Item owns a collection of ExternalReference value objects (1:N per source, N total)
Option B — Dedicated ExternalIdentityMapping aggregate, independent of Item
Option C — Single external identity per Item (1:1)
```

## Evaluation

- `INV-ITEM-004` already forbids treating an external identifier as domain identity.
- `00_DOMAIN_CONCEPT_INVENTORY.md` DC-009 defines External Reference as a Value Object candidate, not an Entity.
- Multiple external sources are explicitly anticipated (`08_DOMAIN_INVARIANTS...md` BR-PROVENANCE-001), which excludes Option C.
- No use case currently requires managing external references independently of their owning Item (e.g. bulk-reassigning a reference between Items is not an approved use case), which does not yet justify the added complexity of Option B.

## Selected Option

**Option A — an Item owns zero or more `ExternalReference` value objects.** Each reference is `(Source, ExternalIdentifier)`. Uniqueness of `(Source, ExternalIdentifier)` **within a single Item** is required; uniqueness *across* the whole domain (i.e., can two different Items claim the same external reference) is treated as a **synchronization conflict**, not an Item invariant, and is resolved through the Synchronization Conflict Resolution policy (`15_DOMAIN_POLICIES_AND_DECISION_TABLES.md` P-014).

## Consequences

- `ExternalReference` remains a Value Object inside the `Item` Aggregate boundary (confirms `09_DOMAIN_AGGREGATES...md` Section 9.3).
- Reassignment of an external reference from one Item to another (Q-008) is **not supported** as a direct operation. It must be modeled as: remove reference from Item A → resolve conflict → add reference to Item B, going through the Synchronization Conflict Resolution policy. This is a deliberate constraint to avoid silent identity drift.
- The database uniqueness question left open in `43_PERSISTENCE_ARCHITECTURE_IMPLEMENTATION_READINESS_REVIEW.md` §37.2 is now answered: uniqueness constraint should be `(ExternalSourceId, ExternalId)` scoped **per Item**, not globally — the persistence architecture document should be updated accordingly in a follow-up pass.

## Status

**RESOLVED.** Confidence: **MEDIUM-HIGH** (cardinality is settled; the conflict-resolution workflow itself remains subject to `SynchronizationConflictResolver` design maturity).

---

# 6. Decision P0-4 — Item Eligibility for Membership (resolves Q-014, dependent on P0-1/P0-2)

## Business requirement

`P-003` (`15_DOMAIN_POLICIES_AND_DECISION_TABLES.md`) requires an `ItemIsEligible` determination before an Item can be added to a Collection, but the concrete eligibility conditions were left as a placeholder.

## Selected Option

Given P0-1 (Membership is Collection-internal) and P0-2 (identity is `ItemId`-based), Item eligibility is now fully specified as:

```text
ItemIsEligible(item) :=
    item exists
    AND item.lifecycleState != Deleted   (see INV-LIFECYCLE-005)
    AND item satisfies Collection.Configuration
        (valid ItemType, required Attributes present — see RULE-CONF / RULE-ATTR series)
```

No additional cross-aggregate compatibility policy is required beyond `ItemIsEligible` + `ItemIsNotAlreadyMember` (already defined in `18_DOMAIN_SPECIFICATIONS_AND_REUSABLE_RULES.md` S-007/S-009). `CollectionMembershipEligibility` domain service candidate (`17_DOMAIN_SERVICES...md` DS-001) is confirmed as **not required** as a separate service — this logic is fully expressible as composed Specifications evaluated inside `Collection.addItem()`.

## Status

**RESOLVED.** Confidence: **HIGH**.

---

# 7. Consolidated Decision Register Update

The following rows in `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` Section 76/77 should be updated upon ratification of this document:

| ID | Previous Status | New Status |
|---|---|---|
| D-011 (Membership representation) | OPEN | ACCEPTED — internal entity of Collection |
| D-014 (External Identity ownership) | OPEN | ACCEPTED — Value Object owned by Item, per-Item uniqueness |
| Q-002 (Membership semantics) | OPEN / P0 | CLOSED |
| Q-007 (External Identity cardinality) | OPEN / P0 | CLOSED |
| Q-008 (External Identity reassignment) | OPEN / P1 | CLOSED — not supported directly |
| Q-014 (Item eligibility) | OPEN / P1 | CLOSED |
| Q-015 (Duplicate semantics) | OPEN / P0 | CLOSED |

Remaining P0-adjacent items (Q-006 Classification, Q-009 Synchronization ownership, Q-021 Multi-tenancy) are **not** resolved by this document and remain open — they were classified P1, not P0, and do not block the aggregate/persistence boundary in the way the four decisions above did.

---

# 8. Impact on Downstream Documents

This resolution directly unblocks the contradiction identified in the earlier review: the persistence schema (`41_DATABASE_SCHEMA_DEFINITION_AND_TABLE_CATALOG.md`) modeled `CollectionItems` as a table owned by `Collections`, which is now confirmed consistent with P0-1. No schema rework is required as a result of this document. The one required follow-up is the external-reference uniqueness scope correction noted in Decision P0-3.

---

# 9. Ratification

This document constitutes a **proposed resolution**. Per `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` Section 72 (Decision Resolution Protocol), it requires:

- [ ] Review against affected Aggregates (Section 8 of this document — done)
- [ ] Review against affected Invariants (done inline per decision)
- [ ] Update of `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` register (pending)
- [ ] Re-run of `19_DOMAIN_MODEL_CONSISTENCY_REVIEW.md` (pending)

Until the register is formally updated, this document should be treated as **authoritative but not yet merged**.
