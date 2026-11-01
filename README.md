# FlowForge

FlowForge is a workflow orchestration and background job processing platform built with .NET.

The goal is to support workflows made up of dependent jobs, track their execution and later run independent jobs in parallel across background workers.

The project is being built incrementally, with reliability, messaging, observability and failure handling added only when they solve a real problem.

## Current Status

The initial solution structure and architecture are in place.

Current work is focused on the workflow model:

- `WorkflowDefinition`
- `JobDefinition`
- `JobDependency`
- `Workflow validation`
- `DAG cycle detection`

Background execution, RabbitMQ, retries and worker coordination will be introduced after the workflow model and persistence are working.

## Tech Stack

- C#
- .NET 10
- ASP.NET Core Web API
- Entity Framework Core
- SQL Server
- .NET Worker Service
- xUnit
- Testcontainers

Planned infrastructure:

- RabbitMQ
- Docker
- OpenTelemetry
- Redis where justified

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
├── LICENSE.txt
└── README.md
```

## Architecture

FlowForge uses a layered architecture with thin controllers and interface-driven services.

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
Entity Framework Core
    ↓
SQL Server
```

The main responsibilities are separated across projects:

- `FlowForge.Api` handles HTTP concerns.
- `FlowForge.Models` contains entities, requests, responses and shared models.
- `FlowForge.Interface` contains service and persistence abstractions.
- `FlowForge.Application` contains service implementations and application logic.
- `FlowForge.Data` contains SQL Server and Entity Framework Core persistence.
- `FlowForge.Infrastructure` contains external infrastructure integrations.
- `FlowForge.Worker` is an independent Worker Service for background job execution.

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

## Documentation

Project documentation is kept under `docs/`.

### Architecture

`docs/architecture.md`

Describes the current system structure, project responsibilities, application flow, workflow model and failure-handling approach.

It represents how the system is currently designed rather than a list of future ideas.

### Design Decisions

`docs/decisions.md`

Records the main technical decisions made while building FlowForge and the reason behind each one.

Current decisions include:

- Layered architecture
- SQL Server and Entity Framework Core
- Separation of Data and Infrastructure
- Identifier strategy
- Generic workflow engine
- Independent Worker Service
- Job handlers
- RabbitMQ introduction strategy
- Workflow validation
- Testing strategy
- Failure handling

### Development Notes

`docs/development/`

Contains implementation notes for individual milestones and features.

`docs/development/001-project-foundation.md`

records the work completed while setting up the initial solution, API, Worker Service, tests, repository and documentation.

Future development entries will record important changes, issues encountered, tests performed and the next step.

### Changelog

`docs/development/CHANGELOG.md`

Provides a concise history of notable changes to the project.

Detailed implementation notes remain in the milestone files, while the changelog gives a quick overview of what has been added, changed or fixed.

## Development Approach

FlowForge is developed in small milestones.

For each feature:

1. Define the problem.
2. Consider the available approaches.
3. Choose the simplest suitable design.
4. Implement the feature.
5. Add tests.
6. Verify the result.
7. Document important decisions and changes.
8. Commit the completed step.

The goal is to keep the project understandable and avoid adding infrastructure or abstractions before they are needed.

## Failure Handling

Failure handling is part of the system design rather than an afterthought.

The planned approach includes:

- `Result<T>` for expected application failures
- Global handling for unexpected exceptions
- Database transactions for related persistence operations
- Retry with backoff for transient job failures
- Permanent failure states for non-retryable jobs
- Dead-letter handling for repeatedly failing messages
- Worker recovery
- Idempotency for duplicate delivery
- Transactional outbox for database and messaging consistency

These capabilities will be introduced as the execution engine develops.

## Planned Work

### Workflow Model

- Workflow definitions
- Job definitions
- Job dependencies
- Workflow validation
- DAG cycle detection

### Persistence

- SQL Server integration
- Entity Framework Core mappings
- Repository and Unit of Work implementations
- Database migrations
- Integration tests with SQL Server Testcontainers

### Workflow Execution

- Workflow executions
- Job executions
- Job attempts
- Dependency resolution
- Parallel job execution
- Workflow completion tracking

### Messaging and Workers

- RabbitMQ
- Job dispatching
- Background job consumers
- Job handlers
- Multiple workers

### Reliability

- Retry policies
- Exponential backoff
- Dead-letter handling
- Idempotency
- Worker recovery
- Transactional outbox
- Cancellation

### Operations

- Structured logging
- OpenTelemetry
- Health checks
- Docker
- Docker Compose
- CI/CD

## Licence

FlowForge is licensed under the MIT License.
