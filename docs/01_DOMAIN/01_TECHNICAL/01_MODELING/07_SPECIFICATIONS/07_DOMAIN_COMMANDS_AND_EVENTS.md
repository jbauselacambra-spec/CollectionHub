# Domain Commands and Events

> Document: `07_DOMAIN_COMMANDS_AND_EVENTS.md`
> Phase: 2.1 — Domain Modeling
> Status: Draft
> Depends on:
> - 00_DOMAIN_CONCEPT_INVENTORY.md
> - 01_DOMAIN_CONCEPT_CLASSIFICATION.md
> - 02_DOMAIN_IDENTITY_AND_INVARIANTS.md
> - 03_AGGREGATE_BOUNDARY_ANALYSIS.md
> - 04_DOMAIN_MODEL.md
> - 05_DOMAIN_BEHAVIOR_AND_OPERATIONS.md
> - 06_DOMAIN_POLICIES_AND_RULES.md
>
> Purpose:
> Define the domain Commands and Domain Events emerging from the
> previously established domain behavior and rules.

1. Purpose

The previous documents established:

Concepts
    ↓
Identity
    ↓
Invariants
    ↓
Aggregates
    ↓
Behavior
    ↓
Rules and Policies

This document introduces the temporal dimension of the domain:

Intent
   ↓
Command
   ↓
Domain Operation
   ↓
State Change
   ↓
Domain Event

The objective is to clearly distinguish:

what someone asks the system to do;
what the domain actually does;
what the domain reports as having happened.
2. Fundamental Distinction

Three concepts must remain separate.

Command

A Command expresses intent.

ChangeItemStatus

means:

Someone is requesting that an Item change status.

Domain Operation

The domain evaluates that intent and potentially changes state.

Item.changeStatus(...)
Domain Event

The Event represents a fact.

ItemStatusChanged

means:

The Item status has changed.

3. Fundamental Flow

The canonical flow is:

Actor
  │
  │ intent
  ▼
Command
  │
  ▼
Domain Operation
  │
  ├── rejected
  │
  └── accepted
          │
          ▼
      State Change
          │
          ▼
      Domain Event

The important distinction is:

Command = desired action


Event = completed fact
4. Command Principles
CMD-001

Commands represent intent.

CMD-002

Commands must not be confused with Domain Events.

CMD-003

A Command may be rejected.

CMD-004

A Domain Event represents something that has already happened.

CMD-005

A Command should contain enough information to express the requested operation.

CMD-006

Commands should not contain infrastructure-specific information unless explicitly required by the domain.

5. Domain Event Principles
EVT-001

Events represent domain facts.

EVT-002

Events should use domain language.

EVT-003

Events should describe meaningful state changes.

EVT-004

Events must not merely expose persistence operations.

Bad:

ItemRowUpdated

Good:

ItemStatusChanged
EVT-005

An Event should identify the domain entity or Aggregate whose state changed.

6. Collection Commands

The current Collection commands are:

CreateCollection
RenameCollection
ChangeCollectionDescription
ChangeCollectionOwner
ChangeCollectionConfiguration
ArchiveCollection
7. CreateCollection
Command
CreateCollection
Intent

Create a new Collection.

Conceptual Inputs
OwnerId
Name
Description?
InitialConfiguration?
Target
Collection Aggregate
Domain Operation
Collection.create(...)
Preconditions
Valid owner
Valid name
Valid initial configuration
Result

A new Collection exists.

Event
CollectionCreated
8. CollectionCreated
Meaning

A Collection has successfully been created.

Conceptual Payload
CollectionId
OwnerId
Name
OccurredAt

Potentially:

ConfigurationVersion

if configuration versioning becomes part of the domain.

Important Principle

The event should contain domain facts required by consumers.

It should not mirror the database row automatically.

9. RenameCollection
Command
RenameCollection
Inputs
CollectionId
NewName
Target
Collection Aggregate
Preconditions
Collection exists
Name is valid
Collection permits renaming
Domain Operation
Collection.rename(...)
Event
CollectionRenamed
10. CollectionRenamed

Conceptual payload:

CollectionId
OldName
NewName
OccurredAt

The inclusion of OldName is useful if consumers need to understand the transition without querying previous state.

This remains a payload-design decision rather than an invariant.

11. ChangeCollectionDescription
Command
ChangeCollectionDescription
Inputs
CollectionId
Description
Event
CollectionDescriptionChanged

This event should only be introduced if the change has meaningful domain consequences.

Otherwise, the event catalog should avoid becoming excessively granular.

12. ChangeCollectionOwner
Command
ChangeCollectionOwner
Inputs
CollectionId
NewOwnerId
Target
Collection Aggregate
Event
CollectionOwnershipChanged
Meaning

Ownership has successfully changed.

13. ChangeCollectionConfiguration
Command
ChangeCollectionConfiguration
Inputs

Conceptually:

CollectionId
ConfigurationChange

The exact representation of ConfigurationChange remains unresolved.

Potential operations:

AddItemType
RemoveItemType
AddAttribute
RemoveAttribute
ModifyAttribute
14. Configuration Change

This command is special because it may affect existing Items.

The domain must first establish:

Is the resulting configuration compatible
with the current Item state?

Possible outcomes:

Valid
    ↓
ConfigurationChanged

or:

Invalid
    ↓
Command rejected

or, if migration is introduced:

Valid only through migration
    ↓
Migration workflow
15. CollectionConfigurationChanged

Conceptual payload:

CollectionId
ConfigurationVersion?
Changes
OccurredAt

The exact payload depends on whether configuration versioning becomes part of the final model.

16. ArchiveCollection
Command
ArchiveCollection
Input
CollectionId
Domain Operation
Collection.archive()
Event
CollectionArchived
17. Collection Events

Current candidate event catalog:

CollectionCreated
CollectionRenamed
CollectionDescriptionChanged
CollectionOwnershipChanged
CollectionConfigurationChanged
CollectionArchived

Not every event is necessarily final.

The event catalog should remain meaningful rather than exhaustive.

18. Item Commands

Current candidate commands:

CreateCollectionItem
ChangeItemType
SetAttributeValue
RemoveAttributeValue
SetMetadata
AddTag
RemoveTag
AddExternalReference
RemoveExternalReference
ChangeItemStatus
ArchiveItem

Potential future:

RestoreItem
MoveItem

These remain unresolved.

19. CreateCollectionItem
Command
CreateCollectionItem
Inputs
CollectionId
ItemType
InitialAttributes
InitialMetadata
InitialTags
InitialExternalReferences
Target
Collection Item Aggregate
Cross-Aggregate Requirement

The operation depends on Collection context.

Conceptually:

Collection
     │
     │ configuration
     ▼
CreateCollectionItem
     │
     ▼
Item

The Item Aggregate should not contain the entire Collection Aggregate merely to perform this validation.

20. CollectionItemCreated
Meaning

A new Item has successfully entered the domain.

Conceptual payload:

ItemId
CollectionId
ItemType
OccurredAt

Potential initial state may be included depending on consumer needs.

21. ChangeItemType
Command
ChangeItemType
Inputs
ItemId
NewItemType
Preconditions
New Item Type valid
Existing state remains compatible
Event
ItemTypeChanged
22. ItemTypeChanged

Conceptual payload:

ItemId
PreviousItemType
NewItemType
OccurredAt

This event describes the semantic transition.

23. SetAttributeValue
Command
SetAttributeValue
Inputs
ItemId
AttributeId
Value
Preconditions
Attribute valid
Attribute applicable
Value valid
Required rules satisfied
Event
ItemAttributeValueChanged
24. ItemAttributeValueChanged

Conceptual payload:

ItemId
AttributeId
PreviousValue?
NewValue
OccurredAt

The event should not expose internal implementation details of the Attribute Value representation.

25. RemoveAttributeValue
Command
RemoveAttributeValue
Inputs
ItemId
AttributeId
Preconditions

The removal must not violate applicable requiredness rules.

Event
ItemAttributeValueRemoved
26. SetMetadata
Command
SetMetadata
Inputs
ItemId
MetadataChange
Event

Potentially:

ItemMetadataChanged

However, this event should only be retained if Metadata changes have meaningful domain semantics.

27. AddTag
Command
AddTag
Inputs
ItemId
TagId
Event
ItemTagAdded
28. RemoveTag
Command
RemoveTag
Inputs
ItemId
TagId
Event
ItemTagRemoved
29. AddExternalReference
Command
AddExternalReference
Inputs
ItemId
Source
ExternalIdentifier
Preconditions
Source valid
Identifier valid
Duplicate policy satisfied
Event
ItemExternalReferenceAdded
30. RemoveExternalReference
Command
RemoveExternalReference
Inputs
ItemId
ExternalReferenceId
Event
ItemExternalReferenceRemoved
31. ChangeItemStatus
Command
ChangeItemStatus
Inputs
ItemId
NewStatus
Preconditions
Transition allowed
Domain Operation
Item.changeStatus(...)
Event
ItemStatusChanged
32. ItemStatusChanged

Conceptual payload:

ItemId
PreviousStatus
NewStatus
OccurredAt

This event represents a meaningful lifecycle transition.

33. ArchiveItem

Archival can be represented either as:

ArchiveItem
    ↓
ItemStatusChanged

or:

ArchiveItem
    ↓
ItemArchived

The preferred option depends on whether:

ARCHIVED

is simply an Item status or a domain concept with additional semantic consequences.

34. Current Recommendation

If archival is simply a lifecycle state:

ArchiveItem
    ↓
ItemStatusChanged

is sufficient.

If archival has independent business meaning:

ArchiveItem
    ↓
ItemArchived

may be preferable.

Current recommendation:

Treat archival as a lifecycle transition until the domain demonstrates a need for a distinct semantic event.

35. Command/Event Matrix
Command	Aggregate	Resulting Event
CreateCollection	Collection	CollectionCreated
RenameCollection	Collection	CollectionRenamed
ChangeCollectionDescription	Collection	CollectionDescriptionChanged
ChangeCollectionOwner	Collection	CollectionOwnershipChanged
ChangeCollectionConfiguration	Collection	CollectionConfigurationChanged
ArchiveCollection	Collection	CollectionArchived
CreateCollectionItem	Item	CollectionItemCreated
ChangeItemType	Item	ItemTypeChanged
SetAttributeValue	Item	ItemAttributeValueChanged
RemoveAttributeValue	Item	ItemAttributeValueRemoved
SetMetadata	Item	ItemMetadataChanged
AddTag	Item	ItemTagAdded
RemoveTag	Item	ItemTagRemoved
AddExternalReference	Item	ItemExternalReferenceAdded
RemoveExternalReference	Item	ItemExternalReferenceRemoved
ChangeItemStatus	Item	ItemStatusChanged
36. Command/Event Relationship

The relationship is not:

Command = Event

It is:

Command
    │
    │ request
    ▼
Domain
    │
    ├── reject
    │
    └── accept
          │
          ▼
        Event

Therefore:

ChangeItemStatus

does not mean:

Item status changed.

It means:

Someone requested a status change.

While:

ItemStatusChanged

means:

The status actually changed.

37. Rejected Commands

Rejected Commands do not generate successful state-change Events.

Example:

ChangeItemStatus
        │
        ▼
Invalid transition
        │
        ▼
Rejected

There is no:

ItemStatusChanged

event.

This distinction is fundamental.

38. Domain Errors

Rejected operations require domain-level reasons.

Examples:

InvalidCollectionName
InvalidItemType
InvalidAttributeValue
AttributeNotApplicable
RequiredAttributeMissing
InvalidStatusTransition
DuplicateExternalReference
InvalidCollectionConfiguration

These are candidate domain error concepts.

They are not yet HTTP errors.

39. Error vs Event

An error describes:

Something requested could not happen.

An Event describes:

Something happened.

Therefore:

InvalidStatusTransition

is not a Domain Event.

While:

ItemStatusChanged

is.

40. Command Idempotency

Some Commands may naturally be idempotent.

Example:

AddTag("rare")

when rare already exists.

Possible semantics:

Option A

Reject as duplicate.

Option B

Treat as no-op.

Option C

Allow duplicates.

Current recommendation:

For Tag membership modeled as a set, repeated addition should preferably be idempotent.

Still provisional.

41. Event Granularity

Events should be meaningful but not unnecessarily fragmented.

For example, changing five attributes could potentially produce:

ItemAttributeChanged
ItemAttributeChanged
ItemAttributeChanged
ItemAttributeChanged
ItemAttributeChanged

or:

ItemAttributesChanged

The correct choice depends on whether individual Attribute changes have independent domain significance.

This is currently unresolved.

42. Aggregate Event Boundary

A Domain Event should normally represent a state change inside one Aggregate.

Example:

Collection
    changes configuration
        ↓
CollectionConfigurationChanged

not:

Everything affected by configuration
    ↓
CollectionConfigurationChanged
ItemUpdated
ItemUpdated
ItemUpdated

unless those Item changes are actually performed as domain operations.

43. Cross-Aggregate Workflow

Consider:

ChangeCollectionConfiguration

which requires migration.

Potential flow:

Command
   ↓
Collection
   ↓
ConfigurationChanged
   ↓
Migration Workflow
   ↓
Items updated
   ↓
Item events

This is fundamentally different from putting all Items inside the Collection Aggregate.

44. MoveItem

If later approved, a candidate Command is:

MoveItem

Inputs:

ItemId
SourceCollectionId
TargetCollectionId

Potential events:

ItemMoved

or potentially:

ItemRemovedFromCollection
ItemAddedToCollection

The final choice depends on the domain semantics of membership.

45. Relationship Commands

Once the Relationship model is finalized, candidate Commands include:

CreateRelationship
RemoveRelationship
ChangeRelationshipType

Potential Events:

RelationshipCreated
RelationshipRemoved
RelationshipTypeChanged

These remain intentionally incomplete because the relationship domain has not yet been finalized.

46. Event Naming Convention

The current convention is:

<Subject><PastTenseAction>

Examples:

CollectionCreated
CollectionRenamed
CollectionArchived


ItemTypeChanged
ItemTagAdded
ItemStatusChanged

The convention emphasizes that Events represent facts that have occurred.

47. Command Naming Convention

Commands use imperative language:

CreateCollection
RenameCollection
ArchiveCollection


ChangeItemStatus
AddTag
RemoveTag

This reinforces the distinction:

Command → imperative


Event → past tense
48. Event Payload Principle

An Event should contain:

identity of the affected domain object;
facts necessary to describe the transition;
relevant contextual information.

It should avoid:

database-specific fields;
ORM metadata;
HTTP-specific information;
authentication tokens;
infrastructure identifiers without domain meaning.
49. Event Ordering

Within a single Aggregate, event ordering may be meaningful.

Example:

CollectionCreated
       ↓
CollectionRenamed
       ↓
CollectionArchived

This creates a temporal history.

However, cross-Aggregate event ordering should not automatically be assumed to be globally deterministic.

50. Event Ownership

Events originate from the Aggregate that changes.

Collection
    ↓
CollectionCreated


Item
    ↓
ItemStatusChanged

The Application layer may publish those events.

The Aggregate does not need to know about:

Kafka
RabbitMQ
Webhooks
HTTP

or any other infrastructure mechanism.

51. Command Ownership

Commands do not belong to Aggregates in the same sense as domain state.

A Command is an intent message that targets domain behavior.

Conceptually:

Command
   ↓
Application orchestration
   ↓
Aggregate

The exact application architecture will be designed later.

52. Event Consumers

Potential future consumers may include:

Search Index
Audit Log
Statistics
Notifications
Integration
Read Models

But the domain must not define its Events merely to satisfy these technical consumers.

The domain event should remain meaningful independently.

53. Domain Event vs Integration Event

These are different concepts.

Domain Event

Expresses an internal domain fact:

ItemStatusChanged
Integration Event

Expresses information intended for an external boundary.

Example:

CollectionItemArchivedForSearchIndex

The latter should not be introduced into the domain model prematurely.

54. Current Event Classification
Core Events
CollectionCreated
CollectionOwnershipChanged
CollectionConfigurationChanged
CollectionArchived


CollectionItemCreated
ItemTypeChanged
ItemAttributeValueChanged
ItemStatusChanged
Supporting Events
CollectionRenamed
CollectionDescriptionChanged
ItemAttributeValueRemoved
ItemMetadataChanged
ItemTagAdded
ItemTagRemoved
ItemExternalReferenceAdded
ItemExternalReferenceRemoved
Future Events
ItemMoved
RelationshipCreated
RelationshipRemoved
ItemRestored
CollectionRestored
55. Traceability Matrix
Domain Behavior	Command	Event
Create Collection	CreateCollection	CollectionCreated
Rename Collection	RenameCollection	CollectionRenamed
Change Owner	ChangeCollectionOwner	CollectionOwnershipChanged
Change Configuration	ChangeCollectionConfiguration	CollectionConfigurationChanged
Archive Collection	ArchiveCollection	CollectionArchived
Create Item	CreateCollectionItem	CollectionItemCreated
Change Type	ChangeItemType	ItemTypeChanged
Set Attribute	SetAttributeValue	ItemAttributeValueChanged
Remove Attribute	RemoveAttributeValue	ItemAttributeValueRemoved
Add Tag	AddTag	ItemTagAdded
Remove Tag	RemoveTag	ItemTagRemoved
Add External Reference	AddExternalReference	ItemExternalReferenceAdded
Remove External Reference	RemoveExternalReference	ItemExternalReferenceRemoved
Change Status	ChangeItemStatus	ItemStatusChanged
Archive Item	ArchiveItem	ItemStatusChanged

This matrix creates traceability across the behavioral model.

56. Important Modeling Decision

We explicitly reject the following pattern:

Controller
    ↓
CreateEvent()

Events should emerge from successful domain behavior.

The intended conceptual flow is:

Command
    ↓
Domain
    ↓
Successful state transition
    ↓
Domain Event
57. Current Command/Event Architecture

The model can now be visualized as:

                 INTENT
                    │
                    ▼
              ┌───────────┐
              │  COMMAND  │
              └─────┬─────┘
                    │
                    ▼
              DOMAIN BEHAVIOR
                    │
             ┌──────┴──────┐
             │             │
          reject         accept
             │             │
             ▼             ▼
          Domain       State Change
           Error            │
                            ▼
                       DOMAIN EVENT
58. Decisions
DEC-CMD-001

Commands represent intent.

DEC-CMD-002

Events represent completed domain facts.

DEC-CMD-003

Commands may be rejected.

DEC-CMD-004

Successful state-changing operations may produce Domain Events.

DEC-CMD-005

Domain Events originate conceptually from the Aggregate whose state changed.

DEC-CMD-006

Domain Events must remain independent of infrastructure.

DEC-CMD-007

Integration Events are not part of the current domain model.

DEC-CMD-008

Command and Event naming conventions are intentionally different.

59. Open Questions

The following remain unresolved:

Q-CMD-001
Should every state mutation emit an event?


Q-CMD-002
Should metadata changes emit events?


Q-CMD-003
What is the correct granularity for Attribute events?


Q-CMD-004
Is archival represented by ItemStatusChanged or ItemArchived?


Q-CMD-005
Is MoveItem supported?


Q-CMD-006
Are Relationships independent Aggregates?


Q-CMD-007
Does configuration versioning become necessary?


Q-CMD-008
Which Events are externally publishable?

These questions should be resolved only when their domain consequences are understood.

60. Final Domain Statement

The current Command/Event model is:

Commands express domain intent, are evaluated against Aggregate invariants and domain policies, and may be rejected. Successful domain behavior produces state changes and meaningful Domain Events representing facts that have occurred. Commands and Events remain conceptually independent from HTTP, persistence, messaging, and other infrastructure concerns.

61. What We Have Achieved

The domain model now covers:

WHAT
    Concepts


WHO
    Aggregate ownership


WHAT MUST ALWAYS BE TRUE
    Invariants


WHAT CAN HAPPEN
    Domain Behavior


WHEN IT IS ALLOWED
    Policies and Rules


WHAT IS REQUESTED
    Commands


WHAT ACTUALLY HAPPENED
    Domain Events

This is a major milestone in the modeling process.

62. Next Step

The next artifact should be:

08_DOMAIN_SCENARIOS_AND_STATE_TRANSITIONS.md

Here we will stop looking at individual operations in isolation and model complete business scenarios.

For example:

Create Collection
       ↓
Create Item
       ↓
Set Attributes
       ↓
Change Status
       ↓
Archive Item

and:

Change Collection Configuration
       ↓
Validate compatibility
       ↓
Accept / Reject
       ↓
Resulting Events

That will allow us to discover inconsistencies that are difficult to see when Commands and Events are examined independently.