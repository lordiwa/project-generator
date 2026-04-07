---
template: architecture
scale: full
description: System architecture, technology stack, data model, and infrastructure design.
---

# Architecture — {Project Name}

## Architecture Overview
{High-level description of the system design}

## System Diagram
```
{ASCII diagram of major components and their interactions}
```

## Components

### {Component Name}
- **Responsibility**: {what it does}
- **Technology**: {tech choice}
- **Interfaces**: {APIs, events, protocols}
- **Data Owned**: {what data it manages}

## Data Model

### {Entity}
| Field | Type | Description | Constraints |
|-------|------|-------------|-------------|
| {field} | {type} | {purpose} | {nullable, unique, etc.} |

### Relationships
- {Entity A} → {Entity B}: {relationship type and description}

## Technology Stack
| Layer | Choice | Rationale | Alternatives Considered |
|-------|--------|-----------|------------------------|
| {layer} | {tech} | {why} | {what else was evaluated} |

## API Design
### {Endpoint Group}
| Method | Path | Description | Auth |
|--------|------|-------------|------|
| {GET/POST/...} | {path} | {what it does} | {auth level} |

## Infrastructure
- **Hosting**: {approach and provider}
- **CI/CD**: {pipeline design}
- **Monitoring**: {observability approach}
- **Scaling**: {strategy}

## Architecture Decision Records

### ADR-001: {Decision Title}
- **Date**: {date}
- **Context**: {why this decision is needed}
- **Decision**: {what was decided}
- **Alternatives**: {what was considered}
- **Consequences**: {trade-offs accepted}

## Integration Points
| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| {system} | {REST/gRPC/events} | {in/out/both} | {why} |
