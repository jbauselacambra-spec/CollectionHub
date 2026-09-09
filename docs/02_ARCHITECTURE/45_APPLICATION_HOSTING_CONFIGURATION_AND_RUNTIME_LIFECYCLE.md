1. Purpose

This document defines the hosting, runtime configuration, startup, execution, health, shutdown, and operational lifecycle architecture of CollectionHub.

It builds upon:

32_CONFIGURATION_AND_RUNTIME_INFRASTRUCTURE.md
33_EXTERNAL_INTEGRATION_INFRASTRUCTURE.md
38_TECHNICAL_SOLUTION_STRUCTURE_AND_PROJECT_DEPENDENCIES.md
44_APPLICATION_RUNTIME_COMPOSITION_AND_DEPENDENCY_INJECTION_ARCHITECTURE.md

The purpose is to establish how the CollectionHub application is hosted and operated across environments while preserving the architectural boundaries already defined.

This document defines runtime architecture, not concrete deployment scripts or production infrastructure.

2. Scope

The document covers:

application hosting model;
process lifecycle;
runtime configuration;
environment configuration;
configuration precedence;
startup;
initialization;
readiness;
application execution;
background processing;
health checks;
graceful shutdown;
failure behavior;
runtime observability;
dependency availability;
database readiness;
migration execution boundaries;
local development;
test execution;
staging;
production;
operational responsibilities.
3. Architectural Objective

CollectionHub must behave consistently regardless of the environment in which it executes.

The environment may change:

configuration values;
infrastructure endpoints;
credentials;
logging verbosity;
telemetry;
feature availability;
scaling characteristics.

The environment must not change:

domain behavior;
application dependency direction;
aggregate rules;
persistence ownership;
architectural boundaries.
4. Hosting Model

The initial runtime architecture is based on the standard .NET hosting model.

Conceptually:

Operating System / Container
          |
          v
    .NET Host Process
          |
          v
   CollectionHub Host
          |
          +------------------+
          |                  |
          v                  v
 Application Runtime   Background Workers
          |
          v
   Infrastructure

The host process owns the application lifecycle.

5. Host Responsibilities

The Host is responsible for:

constructing the application host;
loading configuration;
configuring dependency injection;
configuring logging;
configuring telemetry;
registering health checks;
initializing runtime services;
starting application endpoints;
starting hosted services;
coordinating graceful shutdown.

The Host must not contain business rules.

6. Composition Root

The Composition Root remains inside the Host boundary.

Its responsibilities are:

Configuration
      |
      v
Service Registration
      |
      v
Application Composition
      |
      v
Infrastructure Composition
      |
      v
Runtime

The Composition Root should remain small and declarative.

7. Process Lifecycle

The CollectionHub process follows this lifecycle:

Created
   |
   v
Configuring
   |
   v
Building
   |
   v
Initializing
   |
   v
Starting
   |
   v
Ready
   |
   v
Running
   |
   v
Stopping
   |
   v
Stopped

Any unrecoverable initialization failure moves the process toward:

Failed Initialization
        |
        v
Process Termination
8. Startup Phases

Startup is divided into explicit conceptual phases.

1. Process creation
2. Configuration loading
3. Configuration validation
4. Dependency registration
5. Service provider construction
6. Infrastructure initialization
7. Database readiness verification
8. Hosted service initialization
9. Endpoint activation
10. Readiness publication
11. Normal execution

The exact .NET hosting implementation may combine some phases internally, but the architectural distinction must remain.

9. Configuration Loading

Configuration is loaded before application services requiring configuration are executed.

Sources may include:

base configuration;
environment-specific configuration;
environment variables;
development secrets;
deployment-provided secret stores;
command-line arguments where appropriate.

The architecture must define an explicit precedence order.

10. Configuration Precedence

The effective configuration should follow the standard .NET configuration hierarchy.

Conceptually:

Base Configuration
       |
       v
Environment Configuration
       |
       v
Environment Variables
       |
       v
Secret / Deployment Configuration
       |
       v
Runtime Configuration

The highest-precedence source wins.

Configuration precedence must be documented rather than accidentally emerging from implementation order.

11. Configuration Categories

Runtime configuration is divided into:

Application
Persistence
External Integrations
Background Processing
Observability
Security
Hosting
Feature Management

Each category should have an explicit owner.

12. Application Configuration

Application configuration may include:

application behavior settings;
limits;
feature switches;
operational policies;
use-case execution settings.

Application configuration must not be used to bypass domain invariants.

13. Persistence Configuration

Persistence configuration includes:

database connection;
command timeout;
retry configuration;
database provider options;
operational database settings.

Persistence configuration belongs to Infrastructure/Persistence.

14. Integration Configuration

External integration configuration includes:

provider endpoints;
authentication configuration;
request timeouts;
retry policy;
rate limits;
provider-specific options.

Integration configuration must remain isolated from domain concepts.

15. Background Processing Configuration

Background processing configuration may include:

enabled workers;
polling interval;
batch size;
concurrency;
retry limits;
shutdown timeout;
retention processing.

Background configuration must be bounded to prevent uncontrolled resource consumption.

16. Observability Configuration

Observability configuration may include:

logging level;
telemetry enablement;
exporter configuration;
sampling;
metrics configuration;
tracing configuration.

Production observability must provide sufficient information to diagnose failures without exposing secrets.

17. Strongly Typed Options

Runtime configuration must use strongly typed options wherever practical.

Conceptually:

Configuration
      |
      v
Options Binding
      |
      v
Validated Options
      |
      v
Runtime Service

Application services should not repeatedly access arbitrary configuration keys.

18. Configuration Validation

Configuration validation should occur as early as possible.

Validation categories:

Structural

Is the configuration present?

Semantic

Are the configured values valid?

Dependency-specific

Can the configuration be used to initialize the dependency?

Examples:

malformed URI;
invalid timeout;
missing database connection;
unsupported provider;
invalid concurrency limit.
19. Fail-Fast Configuration

Critical invalid configuration should prevent normal startup.

Examples:

Missing database connection
Invalid required integration endpoint
Invalid authentication configuration
Invalid required runtime option

The application should fail visibly rather than start in a partially functional state when the missing configuration is critical.

20. Optional Dependencies

Not every dependency must block startup.

For example:

CollectionHub
    |
    +-- Database       Critical
    |
    +-- Primary API    Critical/Conditional
    |
    +-- Optional Feed  Non-critical

Criticality must be explicitly classified.

An optional integration being unavailable should not automatically make the entire application unready.

21. Environment Model

The initial environment model is:

Development
Test
Staging
Production

Each environment must use the same architectural composition.

Only environment-specific configuration and operational infrastructure should differ.

22. Development Environment

Development should optimize for:

fast startup;
debugging;
local diagnostics;
deterministic test data;
developer productivity.

Development configuration must not become a special architecture.

23. Test Environment

Tests must be able to construct the application runtime with controlled dependencies.

The test environment should support:

isolated database;
deterministic configuration;
fake external providers;
deterministic clock;
controlled background processing.
24. Staging Environment

Staging should approximate production architecture sufficiently to validate:

configuration;
migrations;
persistence;
integrations;
health checks;
observability;
deployment lifecycle.

Staging should not become a manually maintained special environment.

25. Production Environment

Production requires:

externalized secrets;
controlled configuration;
explicit database migration process;
health checks;
structured logging;
telemetry;
graceful shutdown;
controlled scaling;
operational alerting.
26. Database Migration Boundary

Database migrations are part of deployment lifecycle, not ordinary application startup.

Preferred flow:

Build
  |
  v
Validate Migration
  |
  v
Deploy Application Artifacts
  |
  v
Execute Approved Migration
  |
  v
Start / Update Application

The runtime should not silently mutate production schema.

27. Migration Failure

If a migration fails:

Migration Failure
       |
       +-- Do not assume runtime schema is valid
       |
       +-- Report failure
       |
       +-- Stop deployment progression
       |
       +-- Execute recovery procedure

The application must not conceal migration failures.

28. Database Readiness

Database connectivity is a runtime dependency.

The application must distinguish:

Database reachable

from:

Database schema compatible

Migration validation belongs to deployment.

Runtime health verifies operational availability.

29. Startup Initialization

Runtime initialization may include:

validating configuration;
initializing infrastructure clients;
registering background workers;
validating required dependencies;
loading immutable runtime metadata.

It must not perform arbitrary business operations.

30. Seed Data

Reference-data initialization must be deterministic.

Seed operations must distinguish between:

schema migrations;
immutable reference data;
application-owned data.

The application must not overwrite user-managed data during normal startup.

31. Endpoint Activation

Application endpoints should become available only after critical initialization has completed.

Conceptually:

Process Started
      |
      v
Initialization
      |
      v
Critical Dependencies Ready
      |
      v
Ready = true
      |
      v
Traffic Accepted

This avoids exposing an application that cannot yet process requests correctly.

32. Readiness vs Liveness

The runtime must distinguish:

Liveness

Indicates whether the process is alive.

Readiness

Indicates whether the process is prepared to serve work.

A process may be:

Alive = true
Ready = false

during startup or dependency recovery.

33. Health Check Categories

Health checks should be categorized into:

Liveness
Readiness
Dependency
Operational

Not every dependency belongs in liveness.

34. Liveness Rules

Liveness must be lightweight.

It should answer:

Is the application process fundamentally alive?

It should not execute expensive database or external-provider operations.

35. Readiness Rules

Readiness should answer:

Can this instance safely receive application work?

Critical dependencies may participate.

Examples:

database unavailable;
required configuration invalid;
critical infrastructure unavailable.
36. External Dependency Health

External systems should only affect readiness when they are required for the application to perform its core responsibility.

For example:

Optional Provider Unavailable
        |
        v
Application remains Ready

whereas:

Mandatory Persistence Unavailable
        |
        v
Application Not Ready
37. Runtime State

The runtime may conceptually expose:

Starting
Ready
Degraded
Stopping
Failed

Degraded is useful when optional capabilities are unavailable but core application functionality remains operational.

38. Background Service Lifecycle

Background services follow:

Registered
   |
   v
Created
   |
   v
Started
   |
   v
Running
   |
   v
Stopping
   |
   v
Stopped

Each service must respect application cancellation.

39. Background Processing Cancellation

Long-running operations must accept cancellation.

Conceptually:

CancellationToken
       |
       v
Application Operation
       |
       v
Persistence / Integration

Shutdown should signal cancellation rather than terminating active work abruptly where possible.

40. Background Processing Isolation

A failure in one background worker should not automatically terminate unrelated workers.

Workers should have isolated execution boundaries.

Example:

Worker A  ----+
              |
Worker B  ----+---- Host
              |
Worker C  ----+

A failure in Worker A should be handled according to its failure policy.

41. Retry and Backoff

Transient failures may be retried.

Retry configuration must define:

maximum attempts;
delay;
backoff;
jitter where appropriate;
maximum execution duration.

Infinite retries are prohibited.

42. Recovery Strategy

Recovery must distinguish:

Transient

Retry.

Persistent infrastructure failure

Back off and report.

Invalid configuration

Fail or disable affected component.

Business rejection

Do not retry blindly.

Permanent external failure

Record and escalate according to integration policy.

43. Graceful Shutdown

Shutdown begins when:

process termination is requested;
deployment replaces the instance;
orchestration sends termination;
host receives a shutdown signal.

The application should stop accepting new work before disposing critical dependencies.

44. Shutdown Sequence

Preferred sequence:

Shutdown Requested
       |
       v
Mark Instance Not Ready
       |
       v
Stop Accepting New Work
       |
       v
Cancel Background Processing
       |
       v
Allow Active Operations to Complete
       |
       v
Flush Telemetry
       |
       v
Dispose Services
       |
       v
Terminate
45. Shutdown Timeout

Shutdown must be bounded.

A configured timeout prevents:

hung workers;
stuck network calls;
infinite database operations;
indefinite process lifetime.

Operations exceeding the timeout must be terminated according to the hosting environment's policy.

46. Connection Management

Database and external connections must be managed through framework-supported mechanisms.

The application must avoid:

manually managed global connections;
static database connections;
unmanaged socket lifetimes;
per-operation unnecessary client construction.
47. Resource Ownership

Every runtime resource must have a clear owner.

Examples:

Resource	Owner
DbContext	DI scope
HTTP client	HttpClientFactory
Background worker	Host
Configuration	Host
Repository	DI scope
External adapter	DI container
Telemetry pipeline	Host
Database connection	EF Core/provider
48. Logging Lifecycle

Logging must be available early enough to diagnose startup failures.

Startup failures must produce structured diagnostic information.

Shutdown should flush pending telemetry/logging where supported.

49. Startup Logging

Startup logging should record high-level lifecycle events such as:

application starting;
environment;
configuration validation result;
dependency initialization;
readiness achieved;
application stopping.

It must not log secrets.

50. Runtime Metrics

Relevant metrics may include:

request/use-case duration;
persistence failures;
concurrency conflicts;
background worker executions;
synchronization failures;
retry counts;
queue/backlog size where applicable;
health state.

Metrics should focus on operational behavior rather than exposing sensitive business data.

51. Distributed Tracing

Tracing should support the execution chain:

Request
  |
  v
Use Case
  |
  +--> Database
  |
  +--> External Provider

Tracing identifiers belong to infrastructure.

The domain remains unaware of tracing infrastructure.

52. Correlation IDs

Correlation should be propagated across:

incoming requests;
application operations;
background operations;
external calls.

Background jobs that do not originate from an incoming request must generate an appropriate operation context.

53. Security Configuration

Security-sensitive configuration includes:

credentials;
tokens;
certificates;
encryption keys;
database credentials.

These must be injected through secure runtime configuration.

54. Secret Rotation

The architecture should not assume secrets are immutable forever.

Where the deployment environment supports rotation, the runtime design should allow credentials to be replaced without redesigning application components.

The exact hot-reload behavior is implementation-specific.

55. Configuration Reloading

Configuration should be divided into:

Startup-only configuration

Requires restart when changed.

Runtime-reloadable configuration

Can be changed safely while the application is running.

Only configuration explicitly designed for reload should be hot-reloaded.

Database credentials and critical infrastructure settings should not be assumed reloadable without verification.

56. Configuration Immutability

Business-critical runtime configuration should be treated as immutable for the lifetime of an operation.

A single operation must not observe inconsistent configuration halfway through execution.

57. Feature Configuration

Feature switches must have explicit ownership.

The runtime may control feature activation, but the domain model remains responsible for business invariants.

A feature switch must not be used to create contradictory domain states.

58. Dependency Availability Model

Dependencies should be classified:

Dependency	Criticality
Database	Critical
Core configuration	Critical
Core application services	Critical
Optional external provider	Conditional
Telemetry exporter	Non-critical
Optional background worker	Conditional
Optional cache	Non-critical/Conditional

The classification determines startup and readiness behavior.

59. Degraded Mode

Degraded mode is permitted only where the application can continue operating safely.

Examples:

Telemetry unavailable
       |
       v
Application continues

But:

Database unavailable
       |
       v
Core application becomes Not Ready

Degraded mode must never silently violate business guarantees.

60. Runtime Error Boundaries

Errors are handled according to their layer.

Domain Error
    |
    v
Application Translation
    |
    v
Endpoint / Runtime Response

Infrastructure errors:

Infrastructure Error
    |
    v
Application Boundary
    |
    v
Meaningful Application Result

Technical exceptions must not become accidental public API contracts.

61. API / Endpoint Startup

Where CollectionHub exposes HTTP endpoints, endpoint startup must occur through the Host.

Endpoint handlers/controllers must depend on application abstractions.

They must not:

access DbContext directly;
instantiate repositories;
contain domain business rules;
manage external API clients.
62. Request Execution Lifecycle

A request follows:

HTTP Request
     |
     v
Endpoint
     |
     v
Application Use Case
     |
     v
Domain
     |
     +--> Repository
     |
     +--> Integration Port
     |
     v
Application Result
     |
     v
HTTP Response

The runtime owns request infrastructure.

The application owns use-case orchestration.

The domain owns business behavior.

63. Transaction Lifecycle

Where a use case requires a transaction:

Application Operation
       |
       v
Begin Unit of Work
       |
       v
Domain Changes
       |
       v
Persistence
       |
       v
Commit

The transaction boundary must align with the application use case and aggregate consistency requirements.

64. External Call Lifecycle

External calls must be isolated:

Application
    |
    v
Port
    |
    v
Adapter
    |
    v
HTTP / SDK
    |
    v
External System

The runtime configures:

timeout;
retry;
authentication;
telemetry.
65. Idempotent Startup

Startup operations should be safe to repeat.

This is particularly important for:

deployment retries;
container restarts;
orchestration restarts;
failed initialization.

Initialization must not create duplicate data or duplicate external operations.

66. Restart Behavior

The application must assume that processes can terminate unexpectedly.

Therefore:

important state must be persisted;
background progress must not rely solely on memory;
synchronization state must survive restart;
in-flight external operations must be considered potentially interrupted.
67. Crash Recovery

After restart:

Process Restart
      |
      v
Load Persisted State
      |
      v
Identify Incomplete Work
      |
      v
Apply Recovery Policy
      |
      v
Resume Normal Processing

Recovery policy belongs to the relevant application/integration workflow.

68. Operational Idempotency

Restarting the application must not cause:

duplicate collection items;
duplicate external mappings;
duplicate synchronization effects;
duplicate event publication where delivery guarantees require otherwise.

This reinforces the importance of stable identifiers and idempotency rules.

69. Local Development Lifecycle

A developer should be able to execute:

Start
  |
  v
Load Development Configuration
  |
  v
Validate Dependencies
  |
  v
Start Application
  |
  v
Execute / Debug
  |
  v
Graceful Stop

Development tooling may simplify infrastructure startup, but it must not alter the architectural composition.

70. Test Lifecycle

Automated integration tests should support:

Create Test Environment
       |
       v
Initialize Database
       |
       v
Build Application
       |
       v
Execute Test
       |
       v
Collect Diagnostics
       |
       v
Dispose Environment

Test infrastructure must isolate state between test runs where required.

71. Environment Parity

The following should remain consistent across environments:

project dependency graph;
DI architecture;
configuration structure;
persistence model;
runtime lifecycle;
health semantics.

Only infrastructure values and operational parameters should vary.

72. Deployment Lifecycle

The expected deployment sequence is:

Source
  |
  v
Build
  |
  v
Unit Tests
  |
  v
Architecture Tests
  |
  v
Integration Tests
  |
  v
Artifact
  |
  v
Migration Validation
  |
  v
Deploy
  |
  v
Database Migration
  |
  v
Application Startup
  |
  v
Health Validation
  |
  v
Traffic
73. Deployment Failure

Deployment must stop if:

build fails;
tests fail;
migration validation fails;
production migration fails;
critical configuration is invalid;
application fails readiness.

Automatic rollback must be based on the deployment platform's capabilities and the migration compatibility strategy.

74. Version Compatibility

Application and database versions must remain compatible during deployment transitions.

Schema evolution should support controlled compatibility where rolling deployments are possible.

This reinforces the expand/contract migration strategy defined previously.

75. Runtime Compatibility Contract

The runtime must assume that:

Application Version N

may temporarily coexist with:

Database Schema Version N-1 / N

during controlled deployment.

Breaking schema changes therefore require multi-step migration strategies.

76. Operational Runbook Requirements

Before production operation, runbooks should exist for:

startup failure;
database connectivity failure;
migration failure;
external provider outage;
background worker failure;
degraded mode;
high synchronization backlog;
graceful shutdown;
emergency restart.

The exact operational runbooks belong to deployment/operations documentation.

77. Architecture Constraints

The runtime architecture imposes the following constraints:

Host owns process lifecycle.
Composition Root owns dependency composition.
Application does not construct infrastructure.
Domain does not depend on infrastructure.
DbContext remains scoped.
Background services create explicit scopes.
Configuration is strongly typed.
Secrets are externalized.
Critical configuration is validated at startup.
Production migrations are controlled separately from normal startup.
Health checks distinguish liveness and readiness.
Graceful shutdown is bounded.
Background processing is cancellation-aware.
External calls use infrastructure adapters.
Runtime failures are observable.
Restart must not corrupt persisted state.
Architecture tests enforce dependency boundaries.
78. Runtime Readiness Checklist
Hosting
 Hosting model defined.
 Host responsibilities defined.
 Composition Root defined.
 Lifecycle states defined.
Configuration
 Configuration categories defined.
 Precedence defined.
 Strongly typed options defined.
 Validation strategy defined.
 Secret boundary defined.
 Environment model defined.
Startup
 Startup sequence defined.
 Database readiness defined.
 Dependency criticality defined.
 Readiness semantics defined.
 Initialization boundary defined.
Runtime
 Request lifecycle defined.
 Background lifecycle defined.
 Retry strategy defined.
 Failure boundaries defined.
 Observability defined.
Shutdown
 Graceful shutdown defined.
 Cancellation defined.
 Timeout defined.
 Resource ownership defined.
Deployment
 Migration boundary defined.
 Deployment sequence defined.
 Compatibility strategy defined.
 Failure handling defined.
79. Open Decisions

The following remain implementation-level or deployment-level decisions:

Exact hosting mode and endpoint topology.
Containerization strategy.
Reverse proxy/gateway configuration.
Exact telemetry exporter.
Exact production secret provider.
Exact deployment platform.
Exact background scheduling mechanism if native hosted services become insufficient.
Final outbox decision.
Production scaling model.
Exact health endpoint exposure.

These decisions must not invalidate the runtime architecture established here.

80. Implementation Readiness Assessment
Status

APPLICATION HOSTING AND RUNTIME LIFECYCLE: ARCHITECTURALLY DEFINED

The architecture now defines:

how CollectionHub starts;
how it validates configuration;
how dependencies are composed;
how critical dependencies affect readiness;
how background processing is scoped;
how health is exposed;
how failures are handled;
how shutdown occurs;
how migrations are separated from runtime;
how environments differ;
how deployment progresses;
how observability participates in runtime operations.
81. Relationship With Previous Architecture

The runtime architecture now forms the following chain:

Domain Model
     |
     v
Application Use Cases
     |
     v
Infrastructure Components
     |
     v
Persistence Architecture
     |
     v
Runtime Composition
     |
     v
Hosting & Lifecycle

This represents a continuous architectural model from business behavior to executable runtime.

82. Phase Status

The following areas are now architecturally established:

Domain
    CLOSED


Application
    DEFINED


Infrastructure
    DEFINED


Persistence
    CLOSED FOR DESIGN


Runtime Composition
    DEFINED


Hosting & Lifecycle
    DEFINED

The architecture is approaching the point where the remaining work can focus increasingly on implementation contracts, technical execution pipelines, and operational architecture.

83. Next Step

The next document should define the application execution pipeline and request/use-case processing architecture.

The proposed next file is:

46_APPLICATION_EXECUTION_PIPELINE_AND_USE_CASE_PROCESSING_ARCHITECTURE.md

It should establish how an application operation travels through:

Entry Point
    |
    v
Request / Command
    |
    v
Validation
    |
    v
Use Case
    |
    v
Transaction
    |
    v
Domain
    |
    v
Persistence
    |
    v
Domain Events
    |
    v
Integration / Side Effects
    |
    v
Result

This will connect the runtime lifecycle defined here with the application use cases and workflows already modeled earlier.

84. Final Status

Hosting Architecture: DEFINED

Runtime Configuration: DEFINED

Startup Lifecycle: DEFINED

Readiness Model: DEFINED

Background Processing Lifecycle: DEFINED

Shutdown Lifecycle: DEFINED

Deployment Boundary: DEFINED

Environment Model: DEFINED

Operational Failure Model: DEFINED

Overall Runtime Architecture: READY FOR EXECUTION-PIPELINE DESIGN