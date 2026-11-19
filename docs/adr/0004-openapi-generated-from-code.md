# ADR 0004: OpenAPI generated from code, typed client in the frontend

**Status:** accepted

## Context

The API contract was once written by hand next to the code. The two drifted apart: some endpoints had no typed responses or operation ids, and the frontend learned about a change only at runtime.

## Alternatives considered

- **Keep the hand-written contract and compare it with the code in a test.** Rejected. Two sources still have to be edited for every change.
- **A second generator library.** Rejected. The framework's own generator already runs at build time.
- **Generate the contract from the code at build time.** Chosen.

## Decision

- The backend build writes the OpenAPI document to the repository. It is the only machine-readable contract.
- CI fails when the committed file differs from what the code produces.
- Every endpoint declares an operation id, a summary, typed responses and typed errors. A test checks this for all endpoints.
- The frontend downloads the contract, generates TypeScript types with `openapi-typescript` and calls the API only through `openapi-fetch`. Its CI fails when its copy is behind.

## Consequences

- The contract cannot drift from the code.
- A breaking backend change becomes a type error in the frontend, not a bug in production.
- Examples and long prose live in the code's summaries and in feature docs.

## Revisit when

A client appears that cannot use generated TypeScript types, or the API is opened to outside partners. Then add explicit API versioning.
