# ADR 0001: Modular monolith, not microservices

**Status:** accepted

## Context

The systems cover many business areas: catalogue and prices, orders, CRM, production, accounting, site content. A very small team builds and runs them, and areas are added one after another. The load is that of a small or medium business.

## Alternatives considered

- **Microservices.** Rejected. They add network calls, distributed transactions, more deployments and more monitoring, with no benefit at this scale.
- **A plain layered monolith.** Rejected. Without hard edges between areas, it turns into one tangled codebase as it grows.
- **A modular monolith.** Chosen.

## Decision

- One ASP.NET Core application, deployed as one container to Google Cloud Run.
- It is split into business modules, each with Domain, Application, Infrastructure, Api and Contracts projects.
- Each module owns its own PostgreSQL schema, DbContext and migrations. There are no foreign keys between schemas.
- Architecture tests check the allowed references.

## Consequences

- One deploy, one database and one place to debug.
- A new area is added without touching the internals of others.
- There is no database-level integrity across modules. Application code and tests keep that consistency.
- All modules share one process. Long jobs run as separate cloud jobs.

## Revisit when

A module needs its own scaling, release cycle or team. The schema per module keeps the way open to extract it.
