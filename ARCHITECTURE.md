# Architecture

> System overview and structural map. For navigation, see [AGENTS.md](AGENTS.md).

## Overview

<!-- Brief description of what this system does -->
*TODO: Add system purpose and scope*

## Domain Map

<!-- High-level domains and their responsibilities -->

| Domain | Purpose | Location |
|--------|---------|----------|
| *Example: Auth* | *User authentication and authorization* | *`src/auth/`* |

## Package Layering

<!-- Dependency rules between layers -->

```
┌─────────────────────────────────┐
│           Presentation          │  ← UI, CLI, API handlers
├─────────────────────────────────┤
│            Application          │  ← Use cases, orchestration
├─────────────────────────────────┤
│             Domain              │  ← Business logic, entities
├─────────────────────────────────┤
│          Infrastructure         │  ← External services, persistence
└─────────────────────────────────┘

Rules:
- Upper layers may depend on lower layers
- Lower layers must not depend on upper layers
- Domain layer has no external dependencies
```

## Key Components

<!-- Major components and their interactions -->

### Component Index

| Component | Description | Owner |
|-----------|-------------|-------|
| *TODO* | *Add components as system grows* | *TBD* |

## Data Flow

<!-- How data moves through the system -->
*TODO: Document primary data flows*

## External Dependencies

<!-- Third-party services and integrations -->

| Dependency | Purpose | Documentation |
|------------|---------|---------------|
| *TODO* | *Add as dependencies are introduced* | *Link to docs/references/* |

## Cross-Cutting Concerns

### Observability
*TODO: Logging, metrics, tracing approach*

### Security
See [docs/SECURITY.md](docs/SECURITY.md) *(when created)*

### Reliability
See [docs/RELIABILITY.md](docs/RELIABILITY.md) *(when created)*

## Decision Log

Major architectural decisions are documented in `docs/design-docs/`.

| Decision | Date | Document |
|----------|------|----------|
| *Initial architecture* | *TBD* | *Link to ADR* |

---

*Last verified: 2026-02-14*

