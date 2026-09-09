# CollectionHub — Domain State Machines

## 1. Purpose

This document defines the state machines that exist within the CollectionHub domain.

Its purpose is to establish:

- which domain concepts have meaningful lifecycle states;
- which states are valid;
- which transitions are allowed;
- which transitions are forbidden;
- which commands cause transitions;
- which domain events represent transitions;
- which invariants apply to each state;
- which transitions require domain policies;
- which transitions are terminal;
- which apparent states are merely application concerns and must not become domain states.

This document builds on:

- `07_DOMAIN_COMMANDS_AND_EVENTS.md`
- `08_DOMAIN_INVARIANTS_AND_BUSINESS_RULES.md`
- `09_DOMAIN_AGGREGATES_AND_CONSISTENCY_BOUNDARIES.md`
- `10_DOMAIN_SERVICES_AND_POLICIES.md`
- `11_DOMAIN_USE_CASES_AND_APPLICATION_SERVICES.md`
- `12_APPLICATION_USE_CASES_AND_WORKFLOWS.md`
- `13_DOMAIN_ERROR_MODEL.md`

This document does not define:

- database enum implementations;
- ORM mappings;
- HTTP representations;
- UI state;
- workflow execution states;
- infrastructure lifecycle states.

---

# 2. State Machine Philosophy

A state machine should exist only when the lifecycle of a domain concept has meaningful business semantics.

Not every mutable property represents a state.

For example:

    Collection.name

is a property.

It does not imply:

    NAMED
    RENAMED
    UPDATED

as domain states.

Likewise:

    Item.metadata

is data, not necessarily a state machine.

A state machine is justified when:

> The meaning of an operation depends on the current lifecycle state of the domain concept.

---

# 3. Domain State vs Application State

The system contains multiple kinds of state.

## Domain State

Represents business meaning.

Example:

    Collection
        ACTIVE
        ARCHIVED

## Application Workflow State

Represents execution progress.

Example:

    RECEIVED
    VALIDATING
    EXECUTING
    COMPLETED

These must not be confused.

Application workflow states were described in:

    12_APPLICATION_USE_CASES_AND_WORKFLOWS.md

They are not domain lifecycle states.

---

# 4. Current State Machine Inventory

At the current maturity of the model, the following potential state machines have been identified:

| Concept | State Machine Status |
|---|---|
| Collection | Candidate |
| Item | Candidate |
| Collection Membership | Candidate |
| Import Process | Future |
| Synchronization Process | Future |

The important conclusion is:

> Not every concept necessarily requires an explicit state machine.

The model should remain minimal until lifecycle semantics justify additional states.

---

# 5. Collection State Machine

## 5.1 Current Assessment

Collection currently has no fully established lifecycle beyond:

    exists
    does not exist

Therefore, a lifecycle state machine should not yet be invented without a business requirement.

The following remains provisional:

    ACTIVE
    ARCHIVED

These states should only be introduced if CollectionHub requires archived Collections.

---

# 6. Provisional Collection Lifecycle

If archival semantics are confirmed, the Collection lifecycle would become:

    ACTIVE
       |
       | ArchiveCollection
       v
    ARCHIVED
       |
       | RestoreCollection
       v
    ACTIVE

However, these transitions are currently **provisional**.

No implementation should be based on them until the domain requirement is explicitly accepted.

---

# 7. Why ACTIVE/ARCHIVED Must Not Be Assumed

Adding:

    ACTIVE
    ARCHIVED

creates consequences.

For example:

- Can Items be added to an archived Collection?
- Can a Collection be renamed after archival?
- Can archived Collections be deleted?
- Can an archived Collection be restored?
- Does archival affect visibility?
- Does archival affect ownership?
- Does archival emit events?
- Is archival permanent?

Without answers, the states should not be treated as established domain truth.

---

# 8. Current Collection State Model

For the currently established domain:

    Collection
       |
       +-- exists
       |
       +-- valid
       |
       +-- mutable

These are conceptual characteristics rather than formal enum states.

The current model therefore intentionally avoids an artificial Collection state machine.

---

# 9. Item State Machine

The same principle applies to Item.

At present, Item has:

- identity;
- metadata;
- classification;
- collection memberships.

However, no complete lifecycle has yet been established.

Therefore:

    ACTIVE
    ARCHIVED
    DELETED

must not automatically be introduced.

---

# 10. Item Lifecycle Questions

Before defining an Item state machine, the following questions must be answered:

- Can Items be archived?
- Can Items be restored?
- Can Items be deleted?
- Is deletion physical or logical?
- Can archived Items belong to Collections?
- Can archived Items be edited?
- Does classification affect lifecycle?
- Can an Item become invalid after creation?
- Can an Item be retired permanently?

Until these questions are answered, Item remains without a formal lifecycle state machine.

---

# 11. Classification Is Not Necessarily a State Machine

A classification value such as:

    BOOK
    MOVIE
    GAME
    ALBUM

does not automatically constitute a lifecycle state.

Classification describes:

> What the Item is.

Lifecycle state describes:

> Where the Item is in its business lifecycle.

These concepts must remain separate.

---

# 12. Collection Membership State

Membership is different.

A membership relationship may have meaningful lifecycle semantics.

The minimal current model is:

    ABSENT
       |
       | Add Item
       v
    PRESENT
       |
       | Remove Item
       v
    ABSENT

This is effectively a two-state relationship.

However, `ABSENT` is not necessarily persisted state.

It may simply represent:

> No membership exists.

---

# 13. Membership Transition

The fundamental transition is:

    AddItemToCollection

    ABSENT
       |
       v
    PRESENT

Resulting event:

    ItemAddedToCollection

---

# 14. Membership Removal

The inverse transition:

    RemoveItemFromCollection

    PRESENT
       |
       v
    ABSENT

Resulting event:

    ItemRemovedFromCollection

---

# 15. Duplicate Membership

The state machine makes duplicate membership semantics explicit.

Current state:

    PRESENT

Command:

    AddItemToCollection

Expected result:

    DuplicateMembership

There is no valid transition:

    PRESENT
       |
       | AddItem
       v
    PRESENT

unless the domain explicitly defines the command as idempotent.

This is an important open design decision.

---

# 16. Remove Missing Membership

Similarly:

    ABSENT
       |
       | RemoveItem
       v
    ?

Possible semantics:

### Option A — Reject

    MembershipNotFound

### Option B — Idempotent Success

    ABSENT
       |
       | RemoveItem
       v
    ABSENT

The final choice must be made explicitly.

---

# 17. Membership State Machine

Current conceptual model:

```text
        AddItemToCollection
                |
                v
            +-------+
            |       |
            | ABSENT|
            |       |
            +---+---+
                |
                |
                v
            +-------+
            |       |
            |PRESENT|
            |       |
            +---+---+
                |
                |
        RemoveItemFromCollection
                |
                v
            +-------+
            | ABSENT|
            +-------+

The important invariant is:

A Collection contains an Item at most once.

18. Membership Is Not an Aggregate

The membership state machine does not imply that Membership is an aggregate.

The current aggregate boundary remains:

Collection
    |
    +-- membership

Membership lifecycle is therefore owned by Collection.

19. State Transition Ownership

The rule is:

The aggregate owning the state controls the transition.

Therefore:

Collection.addItem(...)
Collection.removeItem(...)

rather than:

MembershipService.changeState(...)

unless a future domain model demonstrates that Membership itself requires independent identity and lifecycle.

20. Transition Anatomy

Every valid domain transition should be understood as:

Current State
      ↓
   Command
      ↓
Preconditions
      ↓
Domain Rules
      ↓
State Change
      ↓
Domain Event
      ↓
New State

Example:

ABSENT
   ↓
AddItemToCollection
   ↓
membership uniqueness
   ↓
add membership
   ↓
ItemAddedToCollection
   ↓
PRESENT
21. Invalid Transitions

Invalid transitions are important because they reveal domain errors.

Example:

PRESENT
   ↓
AddItemToCollection
   ↓
DuplicateMembership

Another:

ABSENT
   ↓
RemoveItemFromCollection
   ↓
MembershipNotFound

The state machine therefore provides a direct explanation for several errors defined in:

13_DOMAIN_ERROR_MODEL.md
22. Transition Guards

A transition may have guards.

For example:

ABSENT
   |
   | AddItem
   |
   +-- Item exists?
   |
   +-- compatible?
   |
   +-- Collection modifiable?
   |
   v
PRESENT

The guard conditions may be implemented by:

aggregate invariants;
domain policies;
application preconditions.

The authoritative business guard must remain in the domain.

23. State Invariants

A state machine is incomplete without defining state invariants.

For PRESENT membership:

Item identity is valid;
Collection identity is valid;
membership uniqueness holds;
Collection membership invariants hold.

For ABSENT:

no membership exists for the Item within the Collection.
24. Collection Membership State Invariants

When membership is PRESENT:

Collection.contains(ItemId) = true

When membership is ABSENT:

Collection.contains(ItemId) = false

The persistence representation is irrelevant to this domain definition.

25. Terminal States

A terminal state is a state from which no further valid domain transition exists.

The current CollectionHub model does not yet establish any terminal state.

Potential future examples:

DELETED
PERMANENTLY_ARCHIVED
RETIRED

These must not be introduced without explicit domain semantics.

26. Soft Delete Is Not Automatically a State

A technical field such as:

deleted_at

does not automatically imply a domain state:

DELETED

A domain state exists only if the business meaning of deletion affects behavior.

If deletion is introduced, it must answer:

what operations remain allowed;
whether restoration exists;
whether references remain valid;
whether events are emitted;
whether the state is terminal.
27. Archived State

Similarly, an ARCHIVED state is meaningful only if archival changes business behavior.

For example:

ARCHIVED Collection
      ↓
cannot accept new Items

would justify a state.

If archival only hides the Collection from a UI, it may instead be a presentation concern.

28. State Machines and Commands

Commands initiate transitions.

Current relationships:

Command	Current State	New State
AddItemToCollection	ABSENT	PRESENT
RemoveItemFromCollection	PRESENT	ABSENT

Potential future commands:

Command	Current State	New State
ArchiveCollection	ACTIVE	ARCHIVED
RestoreCollection	ARCHIVED	ACTIVE
ArchiveItem	ACTIVE	ARCHIVED
RestoreItem	ARCHIVED	ACTIVE

These future commands remain provisional.

29. State Machines and Events

Events represent successful transitions.

Current:

AddItemToCollection
      ↓
ItemAddedToCollection


RemoveItemFromCollection
      ↓
ItemRemovedFromCollection

Potential future:

ArchiveCollection
      ↓
CollectionArchived


RestoreCollection
      ↓
CollectionRestored

Again, future events are not currently part of the committed model.

30. State Machine and Error Mapping

A useful relationship is:

invalid current state
        ↓
   transition guard
        ↓
    domain error

Examples:

PRESENT + AddItem
      ↓
DuplicateMembership


ABSENT + RemoveItem
      ↓
MembershipNotFound


ARCHIVED + AddItem
      ↓
CollectionModificationNotAllowed

The last example only becomes valid if archival is formally introduced.

31. State Transitions and Events

A domain event should describe the completed transition.

Preferred:

ItemAddedToCollection

Not:

AddItemCommandReceived

The first represents domain truth.

The second represents application activity.

32. State Transition Atomicity

A transition must be atomic within its aggregate boundary.

For:

Collection
    ABSENT membership
        ↓
    PRESENT membership

the aggregate must not expose an intermediate state where:

membership exists partially;
invariants are temporarily violated;
event claims success before the state is committed.
33. Concurrency and State Machines

State machines must remain valid under concurrent commands.

Example:

State = ABSENT

Request A:

Add Item

Request B:

Add Item

Only one transition should successfully establish:

PRESENT

The other must either:

DuplicateMembership

or be treated as an idempotent success according to the final command semantics.

The system must never end with two logically duplicated memberships.

34. State Machine and Optimistic Concurrency

State transitions are natural candidates for optimistic concurrency.

Conceptually:

Collection version 5
      ↓
Add Item
      ↓
version 6

Concurrent request based on version 5:

version mismatch
      ↓
OptimisticConcurrencyConflict

This is a consistency mechanism, not a domain state.

35. State Machine vs Workflow State

Example:

Workflow:
    VALIDATING
    EXECUTING
    PERSISTING

These do not describe CollectionHub business state.

Meanwhile:

Membership:
    ABSENT
    PRESENT

does describe business state.

This distinction must remain explicit throughout the architecture.

36. State Machine vs Entity Properties

Consider:

Item.classification = BOOK

This does not mean:

Item state = BOOK

Likewise:

Collection.name = "My Collection"

does not mean:

Collection state = NAMED

State machines should not become a disguised representation of ordinary entity properties.

37. Minimal State Principle

CollectionHub follows:

Introduce the smallest state machine that explains actual domain behavior.

Avoid:

CREATED
ACTIVE
MODIFIED
UPDATED
READY
COMPLETE

unless these states have distinct business meaning.

CRUD operations alone do not justify lifecycle states.

38. Current Formal State Machines

At the current stage, only one state machine is sufficiently justified:

Collection Membership
ABSENT ↔ PRESENT

The Collection and Item lifecycle state machines remain intentionally unresolved.

This is a deliberate modeling decision.

39. Future Process State Machines

Some future concepts are likely to require explicit state machines.

Import

Potential:

REQUESTED
   ↓
RUNNING
   ↓
COMPLETED

with failure:

RUNNING
   ↓
FAILED
Synchronization

Potential:

PENDING
   ↓
SYNCHRONIZING
   ↓
SYNCHRONIZED

These should be modeled only when the corresponding domain concepts are introduced.

40. Import Is Not Yet a Domain State Machine

The existence of an import workflow does not automatically mean Import is a domain entity.

We must first establish:

identity;
lifecycle;
business meaning;
persistence requirements;
invariants;
ownership;
events.

Only then should an Import state machine be introduced.

41. State Transition Table

Current committed model:

Concept	From	Command	Guard	To	Event
Membership	ABSENT	AddItemToCollection	valid membership	PRESENT	ItemAddedToCollection
Membership	PRESENT	AddItemToCollection	uniqueness violated	ERROR	none
Membership	PRESENT	RemoveItemFromCollection	valid removal	ABSENT	ItemRemovedFromCollection
Membership	ABSENT	RemoveItemFromCollection	removal invalid	ERROR	none
42. Provisional Transition Table

Potential future transitions:

Concept	From	Command	To	Status
Collection	ACTIVE	ArchiveCollection	ARCHIVED	Provisional
Collection	ARCHIVED	RestoreCollection	ACTIVE	Provisional
Item	ACTIVE	ArchiveItem	ARCHIVED	Provisional
Item	ARCHIVED	RestoreItem	ACTIVE	Provisional
Import	REQUESTED	StartImport	RUNNING	Future
Import	RUNNING	CompleteImport	COMPLETED	Future
Import	RUNNING	FailImport	FAILED	Future

No implementation should depend on provisional transitions.

43. Forbidden Transitions

The following transitions are currently invalid:

PRESENT → PRESENT
via AddItemToCollection

unless idempotency is explicitly defined.

Likewise:

ABSENT → ABSENT
via RemoveItemFromCollection

remains undecided until removal semantics are finalized.

Future lifecycle states must also explicitly define forbidden transitions.

44. State Machine Invariants

The following principles are mandatory:

Every committed state must have clear business meaning.
Every committed transition must have a valid trigger.
Invalid transitions must have defined semantics.
State changes must occur inside the appropriate aggregate boundary.
Successful transitions should produce corresponding domain events where event semantics require them.
Failed transitions must not produce success events.
Application workflow states must not be confused with domain states.
Technical persistence states must not be confused with domain states.
45. State Machine Traceability

Every committed transition should be traceable:

Command
   ↓
Workflow
   ↓
Aggregate Behavior
   ↓
State Transition
   ↓
Invariant
   ↓
Event
   ↓
Error on invalid transition

Example:

AddItemToCollection
      ↓
WF-COL-003
      ↓
Collection.addItem(...)
      ↓
ABSENT → PRESENT
      ↓
membership uniqueness
      ↓
ItemAddedToCollection
      ↓
DuplicateMembership if invalid
46. State Machine Questions
OPEN-STATE-001

Does Collection have an actual lifecycle beyond existence?

OPEN-STATE-002

Should Collections be archivable?

OPEN-STATE-003

Can archived Collections still receive Items?

OPEN-STATE-004

Can Items be archived?

OPEN-STATE-005

Is Item deletion a domain operation or infrastructure concern?

OPEN-STATE-006

Should membership removal be idempotent?

OPEN-STATE-007

Should membership addition be idempotent?

OPEN-STATE-008

Will Import become a first-class domain concept?

OPEN-STATE-009

Will Synchronization become a first-class domain concept?

OPEN-STATE-010

Which lifecycle transitions require explicit authorization?

47. Completion Criteria

This document is considered sufficiently mature when:

every meaningful domain lifecycle has been identified;
artificial states have been rejected;
committed states have explicit semantics;
committed transitions have explicit commands;
invalid transitions have explicit behavior;
state invariants are documented;
domain events are associated with meaningful transitions;
aggregate ownership is explicit;
concurrency implications are known;
provisional states are clearly separated from committed domain truth.
48. Final Principle

CollectionHub should not model states merely because software systems commonly use them.

The governing principle is:

A domain state exists only when the current state changes what the business allows, means, or guarantees.

Therefore:

Property
    ≠
State


CRUD operation
    ≠
Lifecycle transition


Workflow status
    ≠
Domain state

The current strongest state machine is:

Collection Membership


    ABSENT
       ↕
    PRESENT

Everything beyond this remains intentionally provisional until the business model requires it.

This restraint protects the domain model from accidental complexity and keeps state semantics explicit.