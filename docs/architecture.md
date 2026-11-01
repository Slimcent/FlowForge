# FlowForge Architecture

## Overview

FlowForge is a workflow orchestration and background job processing platform.

A workflow contains jobs and dependencies between those jobs. A job becomes ready when its dependencies complete successfully, while independent jobs can eventually run in parallel.

The project starts with a simple layered architecture. Messaging, distributed workers, caching and other infrastructure are added only when needed.

## Solution Structure

```
FlowForge/
├── docs/
│   ├── architecture.md
│   ├── decisions.md
│   └── development/
│       ├── 001-project-foundation.md
│       └── CHANGELOG.md
│
├── src/
│   ├── FlowForge.Api
│   ├── FlowForge.Application
│   ├── FlowForge.Data
│   ├── FlowForge.Infrastructure
│   ├── FlowForge.Interface
│   ├── FlowForge.Models
│   └── FlowForge.Worker
│
├── tests/
│   ├── FlowForge.IntegrationTests
│   └── FlowForge.UnitTests
│
├── .gitignore
├── FlowForge.slnx
└── LICENSE.txt
```

## Projects

### FlowForge.Api

ASP.NET Core Web API and HTTP entry point.

Contains:

- Controllers
- Middleware
- OpenAPI configuration
- Dependency registration
- Authentication and authorisation later

Controllers should remain thin and delegate work to services.

### FlowForge.Models

Contains models shared across the application.

Includes:

- Entities
- Requests
- Responses
- Enums
- Common models

Initial workflow entities:

- `WorkflowDefinition`
- `JobDefinition`
- `JobDependency`

### FlowForge.Interface

Contains interfaces used across the application.

Examples:

- Service interfaces
- Repository interfaces
- Unit of Work interface
- Infrastructure abstractions when required

### FlowForge.Application

Contains service implementations and application logic.

Examples:

- Workflow creation
- Workflow validation
- Workflow execution later
- Job coordination later

### FlowForge.Data

Contains database and persistence code.

Includes:

- Entity Framework Core
- SQL Server
- `DbContext`
- Entity configurations
- Repository implementations
- Unit of Work implementation
- Database migrations

### FlowForge.Infrastructure

Contains integrations with external infrastructure.

Examples later include:

- RabbitMQ
- Redis
- OpenTelemetry
- External APIs
- File or storage integrations

Database persistence is kept in `FlowForge.Data`.

### FlowForge.Worker

.NET Worker Service that runs independently from the API.

Its responsibility is to execute background jobs.

Later it will handle:

- Consuming job messages
- Resolving job handlers
- Executing jobs
- Cancellation
- Reporting job success or failure

### Tests

`FlowForge.UnitTests` contains unit tests for application and workflow logic.

`FlowForge.IntegrationTests` contains tests that require real application or infrastructure components.

Testcontainers will be used for infrastructure-dependent tests when required.

## Application Flow

The main application flow is:

```
Controller
    ↓
Service Interface
    ↓
Service Implementation
    ↓
Unit of Work / Repository
    ↓
FlowForge.Data
    ↓
EF Core
    ↓
SQL Server
```

Background processing will later follow:

```
Application
    ↓
RabbitMQ
    ↓
FlowForge.Worker
    ↓
Job Handler
```

## Initial Workflow Model

A workflow contains multiple jobs. Job dependencies describe the relationship between those jobs.

```
WorkflowDefinition
       |
       | 1 : many
       v
JobDefinition

JobDefinition ── JobDependency ── JobDefinition
```

Example:

```
        B
       /
A ----
       \
        C
```

`B` depends on `A`.

`C` depends on `A`.

## Failure Handling

FlowForge handles failures based on where they occur.

```
Failure Handling

FlowForge will handle failures according to their type.

Application validation       -> Result<T>
Unexpected application error -> Global exception handling and logging
Database operation failure   -> Transaction rollback
Transient job failure        -> Retry with backoff
Permanent job failure        -> Mark job as failed
Repeated message failure     -> Dead-letter queue
Worker crash                 -> Recovery or requeue
Duplicate message            -> Idempotency
Database/message consistency -> Transactional outbox
```

Dependent jobs only run when all required dependencies complete successfully.

Cancellation is tracked separately from failure.

## Documentation

- `architecture.md` describes the current system structure.
- `decisions.md` records important technical decisions and why they were made.
- `development/` contains implementation notes for completed milestones or features.
- `development/CHANGELOG.md` provides a concise history of notable project changes.

## Project Dependencies

The FlowForge projects follow a one-way dependency direction.

```
FlowForge.Models
    └── no project references

FlowForge.Interface
    └── FlowForge.Models

FlowForge.Application
    ├── FlowForge.Interface
    └── FlowForge.Models

FlowForge.Data
    ├── FlowForge.Interface
    └── FlowForge.Models

FlowForge.Infrastructure
    └── FlowForge.Interface

FlowForge.Api
    ├── FlowForge.Application
    ├── FlowForge.Interface
    ├── FlowForge.Data
    ├── FlowForge.Infrastructure
    └── FlowForge.Models

FlowForge.Worker
    ├── FlowForge.Application
    ├── FlowForge.Interface
    ├── FlowForge.Data
    ├── FlowForge.Infrastructure
    └── FlowForge.Models
```

`FlowForge.Api` and `FlowForge.Worker` act as composition roots and can reference the concrete projects required to configure and run the application.

Application projects do not depend on the API.