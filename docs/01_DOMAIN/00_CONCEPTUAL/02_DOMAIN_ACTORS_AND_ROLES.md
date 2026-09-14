# CollectionHub — Domain Actors and Roles

> **Phase:** 2.2 — Domain Modeling
> **Artifact:** 02 — Domain Actors and Roles
> **Status:** Draft / Consolidation Baseline
> **Depends on:** `01_DOMAIN_GLOSSARY.md`, `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` (D-015..D-017)

---

# 1. Purpose

Consolidates the actors that may initiate domain operations and the roles they may hold, per the distinction established in `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` D-015:

```text
Actor
    ≠
Authenticated technical principal
    ≠
Owner
    ≠
Authorized collaborator
```

---

# 2. Actors

| Actor | Definition | Domain Identity |
|---|---|---|
| User | Individual interacting with CollectionHub. | `UserId` — independent of authentication mechanism. |
| Application Process | Automated/system-initiated actor (import, scheduled job). | No domain identity of its own; acts on behalf of a User or as System. |
| External System | Source of external data via integration. | Represented as `Source`, never as an Actor with domain rights. |

---

# 3. Roles

| Role | Definition | Status |
|---|---|---|
| Collection Owner | Role responsible for a Collection; not automatically identical to the creating User. | ACCEPTED (`00_DOMAIN_CONCEPT_INVENTORY.md` DC-006) |
| Collaborator | Hypothetical shared-access role. | OPEN (Q-005) — not modeled |
| Administrator | Hypothetical elevated-privilege role. | OPEN — not modeled |

---

# 4. Authorization vs. Domain Rule

Per `20_DOMAIN_MODEL_DECISIONS_AND_OPEN_QUESTIONS.md` D-016, only rules such as `ActorOwnsCollection` / `ActorCanModifyCollection` (`18_DOMAIN_SPECIFICATIONS...md` S-005/S-006) are domain-relevant. Technical authentication (JWT, session, OAuth) remains outside the domain model entirely — see `06_DOMAIN_POLICIES_AND_RULES.md` §25–27.

---

# 5. Open Questions Carried Forward

This document does **not** resolve Q-004 (ownership cardinality), Q-005 (collaborator model), or Q-022 (permission model). It only records the actors/roles already in use so subsequent documents reference them consistently while those questions remain open.

---

# 6. Status

Consolidation baseline. To be expanded once Q-004/Q-005/Q-022 are resolved.
