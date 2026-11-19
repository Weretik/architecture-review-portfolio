# Dependency rules

A rule that is not checked automatically will be broken sooner or later. So almost every rule below has a check that fails CI; the rest are caught in review.

| Rule | Why | Checked by |
| --- | --- | --- |
| A module uses another module only through that module's Contracts project | Modules can change their internals freely | Architecture tests on project references |
| The module graph has no cycles. When an upstream module needs a downstream one, it declares a port that the downstream module implements | No module needs to know about everything | Architecture test on the module graph |
| Domain references only the shared domain kernel. No framework, no database | Business rules are testable and stable | Architecture tests |
| Application does not reference Infrastructure | Use cases depend on ports, not on EF Core or HTTP clients | Architecture tests |
| The host holds only composition: one registration call per module | No business code hides in startup | Architecture tests |
| Each module owns its schema, with no foreign keys into other schemas | A module can be moved to its own service later | Migration review and tests |
| An id stored by another module's domain is a shared typed id, not a raw Guid | Ids of different things cannot be mixed up | The compiler |
| Expected business failures are typed results, mapped to HTTP in one place | One error format, no exceptions for normal flow | Code review and tests |
| Side effects outside the transaction go through the outbox | Nothing is lost between save and send | Architecture tests and handler tests |
| Caches hold only immutable read models, never entities, money or stock | Stale or shared mutable state cannot leak | Architecture test on cached types |
| Every endpoint's access rule and rate-limit policy is listed in a reviewed snapshot | A new anonymous or unprotected endpoint cannot slip in unnoticed | Endpoint inventory test: a new or changed endpoint fails until its line is reviewed |
| A folder holds at most six code files. No "Utils", "Helpers" or "Common" dumps | Code is found by meaning, not by searching | Folder-size test, structure script in CI |
| Frontend components stay under 150 lines, hooks and models under 200 | Files stay readable and testable | Structure script in CI |
| The OpenAPI contract is generated from the code, and the committed copy must match | The contract cannot drift | CI build compares the file |
| The frontend calls the API only through the generated typed client | A backend change shows as a type error | Contract drift check and typecheck in CI |
| Frontend libraries depend only on allowed layers | The admin, the site and shared code stay apart | Nx module boundary lint |
| Every module, host and feature is on the system, module and feature maps | Documentation stays true | Map-check script in CI |
