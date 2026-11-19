# ADR 0002: Modules talk only through Contracts, without cycles

**Status:** accepted

## Context

When one module reads another module's tables or calls its handlers, the modules can no longer change on their own. When two modules need each other, a cycle appears, and neither can be understood or tested alone.

## Alternatives considered

- **Shared database access.** Rejected. It is fast to write, but it couples every module to every schema.
- **A message bus for everything.** Rejected. It is too heavy for in-process calls, and it hides simple reads behind async messages.
- **A public Contracts project per module, plus ports against cycles.** Chosen.

## Decision

- Each module has a Contracts project with small interfaces, plain DTOs and integration events. Other modules may reference only this project.
- Reads use a contract interface. Reactions of the form "after X happens" use an integration event, so the provider does not know who consumes it.
- The module graph must be acyclic. When an upstream module needs something from a downstream one, it declares a port in its own Contracts, and the downstream module implements it.
- An id that one module stores for another module's object is a shared typed id, not a raw Guid.

## Consequences

- A module can change its internals freely. Its Contracts project is its only promise.
- Unit tests replace another module with a fake of its interface.
- There are a few more small interfaces and DTOs.
- Architecture tests fail the build on a forbidden reference or a cycle.

## Revisit when

A module needs data or reactions from many others at once, or the number of small contract interfaces becomes hard to follow. Then group them by use case before changing the rule.
