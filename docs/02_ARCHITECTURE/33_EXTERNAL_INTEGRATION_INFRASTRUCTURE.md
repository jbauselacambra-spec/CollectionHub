# 33. External Integration Infrastructure

## 1. Purpose

This document defines the architecture for integrating CollectionHub with external systems.

Its purpose is to establish how CollectionHub communicates with systems outside its own application boundary while protecting the Domain and Application layers from external protocols, transport mechanisms, data formats and infrastructure-specific failures.

This document builds upon:

- `24_ARCHITECTURAL_BOUNDARIES_AND_LAYERS.md`
- `25_ARCHITECTURAL_COMPONENTS_AND_RESPONSIBILITIES.md`
- `27_COMPONENT_INTERACTIONS_AND_DEPENDENCY_RULES.md`
- `28_APPLICATION_USE_CASE_INTERACTION_MAP.md`
- `29_INFRASTRUCTURE_COMPONENTS_AND_ADAPTERS.md`
- `32_CONFIGURATION_AND_RUNTIME_INFRASTRUCTURE.md`

It defines architectural rules rather than concrete technology choices.

---

# 2. Architectural Context

External systems are considered outside the CollectionHub architectural boundary.

Conceptually:

```text
                    CollectionHub
                         |
             +-----------+-----------+
             |                       |
        Application              Infrastructure
             |                       |
             |                 Integration Adapters
             |                       |
             +-----------+-----------+
                         |
                         v
                 External Systems
```

External systems may include:

- third-party APIs,
- identity providers,
- payment providers,
- notification services,
- messaging platforms,
- file/object storage,
- external catalogues,
- analytics platforms,
- AI services,
- future partner integrations.

The architecture must remain independent from any specific external provider whenever practical.

---

# 3. Integration Boundary

The integration boundary separates CollectionHub's internal model from external models.

The fundamental relationship is:

```text
Internal Model
      |
      v
Integration Port
      |
      v
External Adapter
      |
      v
External System
```

The external system must never directly manipulate the Domain Model.

---

# 4. Integration Ports

Application-level integration requirements should be expressed through ports.

A port represents what CollectionHub needs from an external capability.

Examples:

```text
ExternalCatalogPort
NotificationPort
FileStoragePort
IdentityProviderPort
PaymentPort
ExchangeRatePort
```

The names and responsibilities must reflect business/application capabilities rather than vendor names.

Prefer:

```text
CatalogProvider
```

over:

```text
AmazonCatalogClient
```

when the application does not fundamentally depend on a specific vendor.

---

# 5. Port Ownership

The port should be owned by the layer that requires the capability.

Conceptually:

```text
Application
    |
    +-- ExternalServicePort
             ^
             |
      Infrastructure Adapter
```

The adapter implements the port.

The dependency direction remains:

```text
Application -> Port
Infrastructure -> Port implementation
```

The reverse dependency is prohibited.

---

# 6. External Adapters

Adapters belong to Infrastructure.

Their responsibility is to translate between:

```text
CollectionHub Application Model
```

and:

```text
External System Model
```

Conceptually:

```text
Application
    |
    v
Internal DTO / Port Contract
    |
    v
External Adapter
    |
    v
External Request
    |
    v
External System
```

The adapter must absorb external technical complexity.

---

# 7. Anti-Corruption Layer

External integrations should behave as an Anti-Corruption Layer.

The adapter protects CollectionHub from:

- external naming conventions,
- external identifiers,
- external data structures,
- transport protocols,
- vendor-specific errors,
- vendor-specific status codes,
- authentication mechanisms,
- retry semantics,
- serialization formats.

The domain should never need to understand these details.

---

# 8. External Models

External API models must remain separate from internal models.

For example:

```text
CollectionHub Item
       |
       v
Integration Mapper
       |
       v
ExternalItemRequest
```

and:

```text
ExternalItemResponse
       |
       v
Integration Mapper
       |
       v
CollectionHub Item Data
```

External DTOs must not be reused as domain entities.

---

# 9. Request Mapping

External requests should be constructed explicitly.

The adapter is responsible for:

1. Translating internal data.
2. Applying external naming conventions.
3. Serializing data.
4. Adding required metadata.
5. Adding authentication information.
6. Applying protocol-specific headers.
7. Sending the request.

The application must not construct HTTP requests, SDK calls or vendor-specific payloads directly.

---

# 10. Response Mapping

External responses must be mapped into internal representations.

The mapping should:

1. Validate the external response shape.
2. Extract required values.
3. Translate external identifiers.
4. Convert external types.
5. Translate external status information.
6. Discard irrelevant provider-specific data.
7. Return an internal representation.

Malformed external responses must be treated as integration failures.

---

# 11. External Identifiers

External systems may use their own identifiers.

These must not automatically become CollectionHub aggregate identities.

Conceptually:

```text
CollectionHub Identity
        |
        +-- internal identity
        |
        +-- external reference
```

Where an external reference must be retained, it should be explicitly modeled as an integration/reference concept.

---

# 12. External Reference Mapping

A conceptual representation may be:

```text
ExternalReference
    provider
    externalId
```

The exact model depends on the business requirement.

The domain should only contain such a concept when the external identity has genuine domain significance.

Otherwise it should remain in the integration layer.

---

# 13. Protocol Independence

The application layer should not depend on:

- HTTP,
- REST,
- GraphQL,
- SOAP,
- gRPC,
- SDK-specific abstractions,
- message broker protocols.

These are infrastructure concerns.

For example, an application requirement should be:

```text
retrieve external catalogue information
```

rather than:

```text
send HTTP GET request
```

---

# 14. HTTP Integration

When an external service uses HTTP, the adapter is responsible for:

- HTTP method,
- URL construction,
- headers,
- serialization,
- deserialization,
- authentication,
- timeouts,
- connection management,
- status-code interpretation.

Application code must not contain HTTP client calls.

---

# 15. Authentication

External authentication belongs to the integration infrastructure.

Potential mechanisms include:

```text
API Keys
OAuth 2.0
Bearer Tokens
Client Certificates
Signed Requests
Service Credentials
```

The selected mechanism must remain encapsulated inside the adapter or its supporting infrastructure.

Secrets must be supplied through the configuration/secret architecture defined in `32_CONFIGURATION_AND_RUNTIME_INFRASTRUCTURE.md`.

---

# 16. Authorization

External authorization is distinct from CollectionHub's internal authorization.

The adapter may need to provide credentials proving that CollectionHub is authorized to call an external service.

The application remains responsible for deciding whether the current user/use case is allowed to perform the requested operation.

Therefore:

```text
Internal Authorization
    -> Application/Security

External Authentication
    -> Integration Infrastructure
```

---

# 17. Timeouts

Every external call must have an explicit timeout policy.

The architecture must avoid unbounded external calls.

Conceptually:

```text
Application
    |
    v
External Adapter
    |
    +-- timeout
    |
    v
External System
```

Timeout values should be configuration-driven where appropriate.

---

# 18. Timeout Semantics

A timeout must be treated as an infrastructure failure.

The adapter should translate it into an internal failure category such as:

```text
ExternalServiceTimeout
```

The application may then decide whether to:

- retry,
- fail the use case,
- degrade gracefully,
- notify the user,
- continue without the optional integration.

---

# 19. Retry Policy

Retries must be deliberate.

Not every external operation is safe to retry.

Retry policies must consider:

- idempotency,
- operation type,
- failure type,
- retry count,
- backoff,
- maximum elapsed time.

Avoid blindly retrying all failures.

---

# 20. Exponential Backoff

Where retries are appropriate, exponential backoff should be considered.

Conceptually:

```text
Attempt 1
    |
    +-- failure
    |
    v
short delay
    |
Attempt 2
    |
    +-- failure
    |
    v
longer delay
    |
Attempt 3
```

Jitter may be introduced to prevent synchronized retry storms.

---

# 21. Retryable Failures

Potential retryable conditions may include:

- temporary network failure,
- connection reset,
- service unavailable,
- rate limiting,
- transient gateway failure.

Non-retryable conditions may include:

- invalid credentials,
- malformed request,
- invalid business data,
- unsupported operation,
- permanent authorization failure.

The exact classification must be defined per integration.

---

# 22. Idempotency

Idempotency is mandatory for integrations where retries can repeat an externally visible operation.

Examples include:

```text
Create external resource
Submit payment
Publish command
Send notification
Update remote state
```

Where supported, the adapter should provide an idempotency key.

Conceptually:

```text
Application Operation
        |
        v
Idempotency Key
        |
        v
External Adapter
        |
        v
External System
```

---

# 23. Idempotency Ownership

The idempotency mechanism may belong to:

- application logic,
- integration infrastructure,
- external provider,

depending on the semantics of the operation.

The ownership must be explicit.

An adapter must not falsely assume that a provider guarantees idempotency.

---

# 24. Rate Limiting

External providers may impose rate limits.

The integration layer should be prepared to handle:

```text
429 / rate limited
quota exceeded
throttling
```

The response should be mapped into a meaningful internal error.

Retrying rate limits must respect provider guidance.

---

# 25. Circuit Breaking

Circuit breakers may be introduced for integrations where repeated failures could damage application stability.

Conceptually:

```text
Normal
  |
  v
External Call
  |
  +-- repeated failures
  |
  v
Open Circuit
  |
  v
Fast Failure
  |
  +-- recovery
  |
  v
Half Open
  |
  v
Normal
```

Circuit breakers are infrastructure concerns.

---

# 26. Bulkhead Isolation

Critical external integrations may require resource isolation.

For example:

```text
CollectionHub
    |
    +-- Integration A resources
    |
    +-- Integration B resources
```

A failure in one integration should not necessarily exhaust resources needed by another.

Potential isolation mechanisms include:

- separate connection pools,
- concurrency limits,
- queue isolation,
- dedicated worker resources.

---

# 27. External Failure Translation

External errors must be translated before crossing into Application or Domain layers.

Conceptually:

```text
External Error
      |
      v
Adapter
      |
      v
Integration Error
      |
      v
Application Error
```

The application should not need to understand:

```text
HTTP 502
HTTP 429
VendorException
SDKException
SocketException
```

unless such details are explicitly part of an infrastructure contract.

---

# 28. Error Categories

External integration errors should distinguish at least:

```text
InvalidRequest
AuthenticationFailure
AuthorizationFailure
RateLimited
Unavailable
Timeout
NetworkFailure
MalformedResponse
Conflict
UnexpectedFailure
```

The exact taxonomy may vary per integration.

---

# 29. Domain Error Isolation

External failures must not be modeled as domain failures unless they have genuine domain meaning.

For example:

```text
External API unavailable
```

is not automatically:

```text
Domain invariant violated
```

The distinction must remain explicit.

---

# 30. Optional Integrations

Some integrations may be optional.

For example:

```text
Primary CollectionHub functionality
        |
        +-- Optional enrichment service
```

If an optional integration fails, the application may continue if the use case permits degraded behavior.

This must be an explicit application decision.

---

# 31. Mandatory Integrations

If an external service is mandatory for a use case, failure must prevent completion of that use case.

The application should receive a meaningful failure rather than an external technical exception.

---

# 32. Synchronous Integrations

Synchronous integration is appropriate when the application requires an immediate response.

Conceptually:

```text
Use Case
   |
   v
External Adapter
   |
   v
External System
   |
   v
Response
   |
   v
Use Case continues
```

Timeouts and failure handling are mandatory.

---

# 33. Asynchronous Integrations

Asynchronous integration should be used when:

- immediate response is unnecessary,
- processing is long-running,
- external systems are unreliable,
- work should be decoupled,
- eventual consistency is acceptable.

Conceptually:

```text
Application
    |
    v
Message/Event
    |
    v
Broker
    |
    v
Integration Worker
    |
    v
External System
```

---

# 34. Integration Events

If CollectionHub publishes integration events, these must be distinguished from internal domain events.

```text
Domain Event
    |
    v
Integration Translator
    |
    v
Integration Event
    |
    v
External Consumer
```

A domain event must not automatically become an externally exposed contract.

---

# 35. Integration Event Contracts

External event contracts should be versioned.

They should define:

- event name,
- version,
- identifier,
- timestamp,
- producer,
- payload,
- correlation identifier,
- causation information where required.

The external contract must be treated as a compatibility boundary.

---

# 36. Event Delivery Guarantees

The integration architecture must explicitly define whether delivery is:

```text
At-most-once
At-least-once
Effectively-once
```

Exactly-once semantics should not be assumed without a concrete technical guarantee.

Consumers should generally be designed to tolerate duplicate delivery when using at-least-once delivery.

---

# 37. Outbox Integration

Where CollectionHub needs reliable event publication, the Outbox pattern defined in the persistence architecture should be used.

Conceptually:

```text
Application Transaction
        |
        +-- Domain State
        |
        +-- Outbox Record
        |
        v
      Commit
        |
        v
Integration Publisher
        |
        v
External System
```

This prevents the classic inconsistency:

```text
Database committed
but
External event not published
```

---

# 38. Inbox / Deduplication

For inbound asynchronous messages, an Inbox or deduplication mechanism may be required.

Conceptually:

```text
External Message
       |
       v
Message Identifier
       |
       v
Deduplication
       |
       +-- already processed -> ignore
       |
       +-- new -> process
```

This is particularly important for at-least-once delivery.

---

# 39. External Webhooks

If CollectionHub receives callbacks/webhooks, they must enter through a dedicated integration boundary.

Conceptually:

```text
External System
      |
      v
Webhook Endpoint
      |
      v
Webhook Adapter
      |
      v
Validated Internal Message
      |
      v
Application
```

The webhook endpoint must not directly manipulate domain objects.

---

# 40. Webhook Security

Incoming webhooks must validate:

- signature,
- authentication,
- timestamp,
- replay protection,
- message structure.

Invalid webhook requests must be rejected before application processing.

---

# 41. Webhook Idempotency

Webhook processing must assume duplicate delivery may occur.

The architecture should support deduplication using an external event/message identifier where available.

---

# 42. External Data Validation

External data must never be trusted blindly.

Adapters must validate:

- required fields,
- data types,
- identifiers,
- enumerations,
- expected ranges,
- response structure.

Malformed external data must not be allowed to corrupt the domain model.

---

# 43. External Data Normalization

External representations may use different conventions.

Examples:

```text
"active"
"ACTIVE"
"enabled"
```

or:

```text
2026-08-20T10:00:00Z
20/08/2026
```

Normalization belongs to the integration layer.

The domain receives the canonical internal representation.

---

# 44. External Time Handling

External timestamps must be normalized before entering the application/domain layers.

The adapter should account for:

- timezone,
- offset,
- precision,
- invalid timestamps.

The internal representation should follow CollectionHub's canonical time model.

---

# 45. External Currency and Numeric Data

External monetary or numeric representations must be translated carefully.

The integration layer must account for:

- decimal precision,
- currency codes,
- rounding rules,
- units,
- provider-specific formats.

The domain's monetary model remains authoritative.

---

# 46. External File and Object Storage

If CollectionHub integrates with external file/object storage, the storage adapter should expose application-level operations rather than provider-specific APIs.

Prefer:

```text
FileStoragePort
    store(...)
    retrieve(...)
    delete(...)
```

over exposing:

```text
S3Client
BlobClient
ProviderSdk
```

to application services.

---

# 47. External Search Services

If an external search engine is introduced, it should be treated as a projection/integration concern.

Conceptually:

```text
Domain State
      |
      v
Search Projection
      |
      v
External Search Index
```

The search index must not become the authoritative source of domain state unless explicitly designed as such.

---

# 48. External AI Services

If CollectionHub integrates with AI providers, provider-specific SDKs and prompts should remain outside the domain.

Conceptually:

```text
Application Capability
        |
        v
AI Port
        |
        v
AI Adapter
        |
        v
External AI Provider
```

Provider-specific response structures must be translated before entering application logic.

---

# 49. Provider Substitution

Where an external capability is not inherently vendor-specific, CollectionHub should allow alternative implementations.

For example:

```text
CatalogPort
   |
   +-- ProviderAAdapter
   +-- ProviderBAdapter
   +-- FakeCatalogAdapter
```

This supports:

- testing,
- provider migration,
- fallback strategies,
- environment-specific implementations.

---

# 50. Provider Lock-In

Provider-specific concepts may be accepted when they represent an explicit architectural decision.

The goal is not to eliminate all vendor coupling.

The goal is to ensure that coupling is:

- explicit,
- isolated,
- replaceable where practical,
- documented.

---

# 51. Integration Configuration

External integration configuration should be represented explicitly.

Conceptually:

```text
ExternalServiceSettings
    |
    +-- endpoint
    +-- timeout
    +-- retryPolicy
    +-- authentication
    +-- rateLimit
```

The actual configuration model must depend on the external service requirements.

---

# 52. Integration Observability

Every significant external integration should provide observability.

Useful metrics include:

- request count,
- success count,
- failure count,
- latency,
- timeout count,
- retry count,
- rate-limit count,
- circuit-breaker state,
- message processing duration.

---

# 53. Correlation

External calls should preserve correlation information where appropriate.

Conceptually:

```text
Incoming Request
       |
       v
Application Operation
       |
       v
External Call
       |
       v
External System
```

Correlation identifiers should allow operators to trace an operation across system boundaries without exposing sensitive information.

---

# 54. Distributed Tracing

Distributed tracing may be used for integrations supporting trace propagation.

The integration layer is responsible for:

- propagating trace context,
- creating integration spans,
- recording technical timing,
- avoiding sensitive payload capture.

The domain remains unaware of tracing.

---

# 55. Logging External Calls

External integration logs should generally contain:

- provider/integration name,
- operation,
- duration,
- outcome,
- correlation identifier,
- retry information where useful.

They should not automatically log:

- credentials,
- authorization headers,
- complete request bodies,
- complete response bodies,
- sensitive user data.

---

# 56. Integration Testing

Each external adapter should be testable at several levels.

### Unit tests

Test:

- request mapping,
- response mapping,
- error translation,
- validation.

### Contract tests

Verify compatibility with the expected external contract.

### Integration tests

Verify real communication with the external system where feasible.

### Failure tests

Verify:

- timeout,
- unavailable provider,
- malformed response,
- rate limiting,
- authentication failure.

---

# 57. Test Doubles

The application should be testable without requiring external systems.

Possible test doubles include:

```text
FakeCatalogProvider
FakeNotificationProvider
FakeStorageProvider
FakeIdentityProvider
```

These implement the same ports as production adapters.

---

# 58. External Contract Evolution

External APIs may evolve independently from CollectionHub.

The integration layer must therefore isolate version-specific changes.

Conceptually:

```text
Provider API v1
      |
      v
Adapter V1
      |
      v
Internal Port

Provider API v2
      |
      v
Adapter V2
      |
      v
Same Internal Port
```

This allows provider evolution without necessarily changing application behavior.

---

# 59. Integration Versioning

Integration contracts should explicitly define versions when required.

Versioning may occur at:

- endpoint level,
- message level,
- adapter level,
- DTO level.

Version changes must be treated as architectural changes when they affect application semantics.

---

# 60. External Availability

External availability must be classified according to business importance.

### Critical

Failure blocks a core use case.

### Important

Failure degrades functionality.

### Optional

Failure has limited impact.

This classification should drive:

- retry strategy,
- timeout,
- circuit breaker,
- readiness,
- fallback behavior.

---

# 61. Fallbacks

Fallback behavior may be introduced for selected integrations.

Examples:

```text
External enrichment unavailable
    -> use local data

Primary provider unavailable
    -> secondary provider

Optional service unavailable
    -> continue without enrichment
```

Fallbacks must be explicitly modeled.

They must not silently change business semantics.

---

# 62. External Consistency

CollectionHub should not assume immediate consistency from external systems.

When eventual consistency exists, the application must explicitly account for:

- delayed updates,
- stale reads,
- duplicate messages,
- out-of-order events.

The domain model must only represent eventual consistency when it is a genuine business characteristic.

---

# 63. Integration Transactions

A database transaction cannot normally be extended atomically across arbitrary external systems.

Avoid assuming:

```text
Database
    +
External API
    =
single ACID transaction
```

Instead use:

- Outbox,
- retries,
- compensation,
- state machines,
- idempotency,
- reconciliation.

---

# 64. Compensation

When an external operation cannot be rolled back technically, the application may require compensating behavior.

Conceptually:

```text
Operation A
    |
    v
External Side Effect
    |
    +-- later failure
    |
    v
Compensating Operation
```

Compensation must be modeled explicitly where business correctness requires it.

---

# 65. Reconciliation

For integrations where state can diverge, reconciliation processes may be required.

Conceptually:

```text
CollectionHub State
        |
        +------ compare ------+
        |                     |
        v                     v
External State          Expected State
        |
        v
Reconciliation
```

Reconciliation is particularly relevant when:

- external systems are eventually consistent,
- messages can be lost,
- external operations can partially fail.

---

# 66. Integration Security Boundary

External integration infrastructure is a security boundary.

It must protect:

- credentials,
- tokens,
- certificates,
- sensitive payloads,
- external endpoints.

Security controls should be centralized where possible rather than duplicated across every adapter.

---

# 67. Network Configuration

Network-level concerns remain infrastructure responsibilities.

Examples include:

- DNS,
- TLS,
- proxies,
- connection pools,
- firewall requirements,
- outbound restrictions.

The application should not need to understand these details.

---

# 68. Dependency Failure Isolation

An external dependency must not be able to bring down unrelated application capabilities through uncontrolled resource consumption.

The integration infrastructure should use appropriate:

- timeouts,
- concurrency limits,
- queues,
- circuit breakers,
- connection limits.

---

# 69. Integration Lifecycle

Each external adapter should have a defined lifecycle:

```text
Configuration
      |
      v
Validation
      |
      v
Initialization
      |
      v
Ready
      |
      v
Operational
      |
      v
Shutdown
```

The adapter must release resources during shutdown.

---

# 70. Architectural Dependency Diagram

The integration architecture can be summarized as:

```text
                  APPLICATION
                       |
                       v
              Integration Ports
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Adapter A    Adapter B    Adapter C
          |            |            |
          v            v            v
      Provider A   Provider B   Provider C
```

The critical rule is:

```text
Application knows the capability.
Infrastructure knows the provider.
```

---

# 71. Architectural Constraints

The following constraints are mandatory:

1. External systems remain outside the CollectionHub boundary.
2. Application code must depend on integration ports, not providers.
3. External SDKs must remain in Infrastructure.
4. External DTOs must not become domain objects.
5. External errors must be translated.
6. All external calls must have bounded timeouts.
7. Retry behavior must be explicit.
8. Retryable operations must consider idempotency.
9. Secrets must remain outside application/domain models.
10. External data must be validated before entering the application.
11. Integration events must be distinct from domain events.
12. External transactions must not be assumed to be ACID with local persistence.
13. Provider-specific coupling must be isolated and documented.
14. Integration failures must not silently become domain failures.
15. Optional dependencies must support explicit degraded behavior where appropriate.

---

# 72. Architectural Decisions

| Decision | Status |
|---|---|
| External integrations isolated in Infrastructure | Adopted |
| Application-level integration ports | Adopted |
| Provider-specific adapters | Adopted |
| External DTOs separated from internal models | Mandatory |
| Anti-Corruption Layer | Adopted |
| Explicit timeout policies | Mandatory |
| Explicit retry policies | Mandatory |
| Idempotency for retryable side effects | Mandatory |
| Circuit breakers | Conditional |
| Bulkhead isolation | Conditional |
| Integration events separate from domain events | Adopted |
| Outbox | Conditional |
| Inbox/deduplication | Conditional |
| Provider substitution | Preferred |
| Automatic fallback | Rejected unless explicitly modeled |
| Distributed tracing | Preferred |
| Reconciliation | Conditional |

---

# 73. Open Questions

The following questions remain open for later architectural phases:

1. Which external systems will CollectionHub actually integrate with?
2. Which integrations are mandatory versus optional?
3. Which providers are currently known?
4. Which external capabilities require ports?
5. Which integrations are synchronous?
6. Which integrations should be asynchronous?
7. Which operations require idempotency keys?
8. Which integrations require circuit breakers?
9. What are the provider-specific rate limits?
10. Which external events will CollectionHub consume?
11. Which integration events will CollectionHub publish?
12. Is an Outbox required?
13. Is an Inbox/deduplication mechanism required?
14. Which integrations require reconciliation?
15. What external contracts require formal versioning?
16. Which external integrations contain sensitive data?
17. Which integrations require distributed tracing?
18. What fallback behavior is acceptable for each optional integration?
19. Which external provider dependencies represent deliberate vendor lock-in?
20. What contractual/API SLAs apply to each provider?

These questions should be resolved when the concrete integration inventory is defined.

---

# 74. Final Architectural Position

CollectionHub external integration infrastructure provides a controlled boundary between the internal application model and systems outside CollectionHub.

The resulting architecture is:

```text
                         COLLECTIONHUB
                              |
                         Application
                              |
                    Integration Port
                              |
                    Anti-Corruption Layer
                              |
                    External Adapter
                              |
               +--------------+--------------+
               |              |              |
               v              v              v
          External API    Message Broker   External Storage
```

The core architectural principle is:

> CollectionHub depends on external capabilities through stable internal contracts, while Infrastructure absorbs the technical and provider-specific complexity required to communicate with external systems.

This preserves the independence of the Domain Model and Application layer while allowing CollectionHub to integrate with, replace, version and isolate external providers without allowing those providers to dictate the internal architecture.