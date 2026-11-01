# FlowForge Design Decisions

## Layered Architecture

### Decision

Use a layered architecture with thin controllers and interface-driven services.

```
Controller
    ↓
Interface
    ↓
Service
    ↓
Unit of Work / Repository
    ↓
Data
```

### Reason

This keeps HTTP, application logic and persistence concerns separated.

It also follows the architecture style I already use in .NET projects, which keeps the codebase consistent and easier to maintain.

---

## Keep the Initial Architecture Simple

### Decision

Start with one application and an independent Worker Service.

Microservices and additional service boundaries will only be introduced if a clear requirement appears.

### Reason

FlowForge already introduces enough complexity through workflows, background jobs, messaging and workers.

Starting with microservices would add deployment, networking and service coordination problems before the core workflow engine is working.

The project should remain simple enough to build and finish.

---

## SQL Server

### Decision

Use SQL Server with Entity Framework Core for persistence.

### Reason

FlowForge needs relational data, transactions, constraints, concurrency support and reliable persistence.

SQL Server provides these requirements and fits naturally with the .NET and EF Core stack.

There is currently no requirement that justifies introducing another relational database.

---

## Separate Data from Infrastructure

### Decision

`FlowForge.Data` contains database and persistence code:

- DbContext
- EF Core configuration
- Repositories
- Unit of Work
- Migrations

`FlowForge.Infrastructure` contains non-database infrastructure such as:

- Messaging
- Caching
- Observability
- External services
- File or storage integrations

### Reason

Persistence and external infrastructure have different responsibilities.

Keeping them separate prevents `FlowForge.Infrastructure` from becoming a large project containing unrelated technical concerns.

It also makes the database layer easier to understand and test independently.

---

## Identifier Strategy

### Decision

Use string GUIDs for major entities.

```
public string Id { get; set; } = Guid.NewGuid().ToString();
```

Examples:

- `WorkflowDefinition`
- `JobDefinition`
- `WorkflowExecution`
- `JobExecution`

Use `long` for small internal records where a globally unique identifier is unnecessary.

Use composite keys for relationship tables where appropriate.

### Reason

Major FlowForge identifiers may move between the API, database, RabbitMQ and Worker processes.

GUIDs can be generated independently without relying on the database.

Using them as strings also keeps the identifier format simple across .NET and future services written in other languages.

Small internal records do not need this flexibility, so numeric IDs can remain simpler where appropriate.

---

## Models

### Decision

Keep entities, requests, responses, enums and common models in `FlowForge.Models`.

### Reason

These types are shared across multiple layers.

Keeping them in one project avoids coupling them to the API or persistence layer and follows the structure used in my existing .NET projects.

---

## Generic Workflow Engine

### Decision

Keep FlowForge independent of a specific business domain.

The engine understands:

- Workflows
- Jobs
- Dependencies
- Executions
- Failures
- Retries

Specific job behaviour is implemented separately.

### Reason

FlowForge should be able to execute different kinds of workflows without changing the core engine.

For example, the same engine could later support:

- Data processing
- Report generation
- Notifications
- Document processing
- Order-processing demonstrations

The workflow engine only needs to understand how jobs relate and execute.

---

## Worker Service

### Decision

Use a separate .NET Worker Service for background job execution.

The API and Worker run as separate processes.

### Reason

Long-running and background work should not depend on an HTTP request remaining open.

A separate Worker also allows background processing to run independently from the API and makes it possible to scale workers separately later.

---

## Job Handlers

### Decision

Executable job types use job handlers.

```
JobType
   ↓
JobHandler
```

A handler may call an application service when additional business logic is required.

### Reason

The workflow engine should decide when a job runs without knowing how every type of job is implemented.

Handlers separate job execution behaviour from workflow orchestration and make new job types easier to add.

---

## RabbitMQ

### Decision

RabbitMQ will be introduced when FlowForge begins sending work to independent workers.

It will not be part of the initial workflow-definition and persistence work.

### Reason

RabbitMQ solves communication between separate processes.

Before distributed background execution is implemented, it does not solve a problem we currently have.

Delaying it keeps the first implementation simpler and allows the workflow model to be tested before distributed messaging is introduced.

---

## Workflow Validation

### Decision

Keep workflow validation in `FlowForge.Application`.

Examples include:

- Unique job keys
- Existing dependency references
- Self-dependency prevention
- Duplicate dependency prevention
- Cycle detection

Database constraints remain in `FlowForge.Data`.

### Reason

These rules describe whether a workflow is valid for the application rather than how the data is represented.

`FlowForge.Models` should contain data structures, while `FlowForge.Application` owns application behaviour and validation.

---

## Testing

### Decision

Use:

- Unit tests for application and workflow logic
- Integration tests for API and infrastructure behaviour
- Testcontainers for SQL Server, RabbitMQ and other infrastructure when required

### Reason

Core workflow rules should be fast to test without external dependencies.

Infrastructure behaviour should be tested against real services where possible rather than relying only on mocks.

Testcontainers provides reproducible test infrastructure without requiring developers to manually configure local databases or message brokers.

---

## Failure Handling

### Decision

Handle failures based on their type rather than using one generic retry strategy.

Expected application failures return `Result<T>`. Unexpected exceptions are handled globally and logged.

Database operations use transactions where multiple related changes must succeed together.

Background jobs distinguish between transient and permanent failures. Transient failures may be retried with backoff, while permanent failures are marked as failed.

Later messaging support will include dead-letter handling, idempotency, worker recovery and a transactional outbox.

### Reason

Different failures require different responses.

Retrying validation errors or permanent failures would not help, while transient infrastructure failures may recover on retry.

This approach keeps failure handling predictable and helps prevent inconsistent workflow state or duplicate side effects.