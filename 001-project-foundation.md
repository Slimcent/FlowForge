# `001-project-foundation.md`

# Project Foundation

**Date:** 2026-11-01

## Changes

Created the initial solution structure:

```
src/
├── FlowForge.Api
├── FlowForge.Application
├── FlowForge.Data
├── FlowForge.Infrastructure
├── FlowForge.Interface
├── FlowForge.Models
└── FlowForge.Worker

tests/
├── FlowForge.IntegrationTests
└── FlowForge.UnitTests
```

All projects target .NET 10.

`FlowForge.Api` was created as an ASP.NET Core Web API with:

- Controllers
- HTTPS
- OpenAPI
- Traditional `Program` class
- No authentication yet
- No Aspire
- No container support yet

`FlowForge.Worker` was created as a .NET Worker Service.

Verified:

- API starts successfully
- `/openapi/v1.json` is available

Repository setup:

- Added `.gitignore`
- Removed `.vs` files from Git tracking
- Added MIT license
- Added project documentation

## Decisions

- Use layered architecture
- Use SQL Server with EF Core
- Keep persistence in `FlowForge.Data`
- Keep external integrations in `FlowForge.Infrastructure`
- Use string GUIDs for major entities
- Use `long` IDs where appropriate
- Keep the workflow engine generic
- Use an independent Worker Service for background jobs
- Add RabbitMQ later

## Next

- Add project references
- Build the complete solution
- Create `WorkflowDefinition`
- Create `JobDefinition`
- Create `JobDependency`
- Add workflow validation


---

# `CHANGELOG.md`

# Changelog

## Unreleased

### Added

- Initial FlowForge solution structure
- ASP.NET Core Web API project
- Application, Data, Infrastructure, Interface and Models class libraries
- .NET Worker Service
- Unit and integration test projects
- OpenAPI support
- MIT license
- Initial project documentation

## Project References

Project references were added between the FlowForge projects according to the dependency direction defined in `docs/architecture.md`.

The API and Worker act as composition roots, while application logic remains independent from concrete persistence and infrastructure implementations.

The complete solution was built after configuring the references to verify that the dependency graph is valid and contains no circular references.