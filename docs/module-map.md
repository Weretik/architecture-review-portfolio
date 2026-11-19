# Module map

## The system

```mermaid
flowchart LR
  staff["Staff"] --> admin["Admin<br/>React SPA, not indexed"]
  visitor["Site visitor"] --> site["Public site<br/>Next.js, server-rendered"]
  admin -- "typed client<br/>from OpenAPI" --> api
  site -- "typed client<br/>from OpenAPI" --> api
  subgraph backend["Backend: one deployable modular monolith"]
    api["API host<br/>composition only"]
    modules["Business modules"]
    api --> modules
  end
  modules --> db[("PostgreSQL<br/>one schema per module")]
  scheduler["Cloud scheduler<br/>every few minutes"] -- "protected tick" --> api
  modules -- "outbox" --> external["External services<br/>CRM, e-mail, messaging, ERP"]
```

- **One backend, many modules.** One ASP.NET Core application runs on Google Cloud Run. It is split into business modules, such as catalogue, orders, CRM, content and identity.
- **Two clients.** The admin is a React SPA for staff, kept out of search engines. The public site is Next.js, server-rendered for SEO. Both call the API through a client generated from the OpenAPI contract.
- **Background work without a resident process.** The host scales to zero, so there is no in-process timer. A cloud scheduler calls one protected tick endpoint. The tick sends the outbox and starts due tasks.

## One module

Every business module has the same five projects.

```mermaid
flowchart TB
  Api["Module.Api<br/>thin endpoints: HTTP to one command or query"]
  App["Module.Application<br/>use cases, validators, ports"]
  Dom["Module.Domain<br/>aggregates, value objects, rules"]
  Inf["Module.Infrastructure<br/>DbContext, migrations, repositories, external clients"]
  Con["Module.Contracts<br/>what other modules may use"]
  Api --> App
  Api -. "registration only" .-> Inf
  App --> Dom
  App --> Con
  Inf --> App
  Inf --> Dom
```

| Project | Owns |
| --- | --- |
| Domain | Aggregates, value objects, domain errors and events, business rules |
| Application | One folder per use case (command or query), validators, ports for persistence and integrations |
| Infrastructure | Its own database schema, migrations, repositories, external clients, implementations of its Contracts |
| Api | Minimal API endpoints grouped by feature, plus one registration call for the host |
| Contracts | Small read and command interfaces, their DTOs, integration events. This is the only part other modules may see |

Shared building blocks (outbox, result types, caching, base classes) hold no business rules.

## Inside a module

Folders follow features and aggregates, never kinds of classes:

```text
Module.Application/
└── Features/<Feature>/
    ├── Commands/<UseCase>/   command, handler, validator
    └── Queries/<UseCase>/    query, handler, read model
```

A folder holds at most six code files. When it would hold more, it is split by meaning into named subfolders.
