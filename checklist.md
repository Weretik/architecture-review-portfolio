# Architecture review checklist

This is the short version of the checklist I use to review a system. It works for a modular monolith or a small set of services, with a web or mobile client and, if present, AI features.

## How I run a review

1. **Step 0: quality targets in numbers.** Before anything else I read the targets and their current values: P95 and P99 per kind of request, peak load, data volume, availability, cost. If there are no targets, that is the first finding. Without numbers I give no verdict on speed or reliability.
2. **Review gives facts.** Each question gets a mark and evidence (file, test, config or dashboard). No fixes and no prices at this step.
3. **Decision gives a plan.** Each finding gets an option, effort, monthly cost and a decision: now, later (with a trigger) or not needed. Choices that are expensive to change get an ADR.

Three levels: **Quick** (★ questions only, about an hour), **Standard** (all questions, half a day), **Deep** (all questions with evidence, plus running the tools, several days).

Marks: ✅ meets · ⚠️ partly · ❌ missing · ➖ not applicable (with reason) · ❓ cannot verify. A ❓ is not a pass.

Severity: **Critical** means data, money or uptime can be lost now. **High** will hurt with growth. **Medium / Low** are quality and convenience.

## 0. Context and targets

- ★ What problem does the system solve, and for whom?
- ★ Are quality targets written down in numbers?
- ★ Are they measured, shown on a dashboard and alerted? Are the alerts real, not just named?

## 1. Module boundaries and layers

- ★ Does each module talk to others only through its public contract (no other module's tables, entities or handlers)?
- ★ Is the module graph free of cycles? When an upstream module needs a downstream one, does it use a port that the downstream module implements?
- ★ Do dependencies point inwards? The domain has no framework or database code.
- Is code organised by feature and use case, not by technical type? Are there no "Helpers" or "Utils" dumps?
- Does the host only compose modules and hold no business code?
- Are these rules checked by architecture tests that fail CI?

## 2. Domain and use cases

- ★ Is one command one transaction with one save?
- ★ Are expected failures typed results, mapped to HTTP in one place? Are exceptions only for the unexpected?
- ★ Do business rules live in aggregates where rules exist, while plain CRUD stays plain?
- Are reads separate from writes? Queries do not load aggregates built for writing.
- Is every input validated, with messages users understand?

## 3. Data

- ★ Does each module own its schema, with no foreign keys or joins into other modules?
- Do reads use projections or no-tracking queries, and page in SQL?
- Are there no queries or saves inside loops?
- Is search indexed for the way it is actually queried?
- Are the slowest queries known and checked with EXPLAIN?
- Are migrations a separate step before deploy that keeps the previous release working?

## 4. Events, outbox and idempotency

- ★ Can data be lost between "save" and "send"? Side effects and integration events must be written in the same transaction (transactional outbox).
- Are outbox handlers idempotent? Do failed messages stay visible instead of disappearing?
- ★ Does a repeated request do the work only once? Creates of orders, money, stock and outgoing messages take an idempotency key with a unique index. A repeat returns the first result, and the same key with different data returns 409.
- Do webhooks, scheduled jobs and syncs check state or dedupe before acting?

## 5. Concurrency

- ★ Is shared state (stock, balances, numbering, limits) protected by a named mechanism: a conditional update, a row version or a lock?
- Is a race on a unique key caught and returned as a clear answer, never as a 500?

## 6. API contract

- ★ Is the OpenAPI contract generated from the code, with no hand-written copy?
- Does CI fail when the committed contract differs from the code?
- Does every operation have an id, a summary and typed responses and errors?
- Does the client use types generated from that contract, so a breaking change fails its build?

## 7. Security

- ★ Is every endpoint closed by default, with an explicit permission?
- ★ Can a user open someone else's object by changing an id? Ownership is checked and tested.
- ★ Are secrets outside the repository? Is secret scanning on? Is cloud access keyless?
- Are sessions safe? HttpOnly cookies in the browser, short-lived tokens with refresh on mobile.
- Is there rate limiting for anonymous and expensive endpoints, and a global soft limit?
- Are dependencies scanned in CI?
- Are webhooks signed and checked?
- Is personal data known, minimised and kept out of logs?

## 8. Reliability

- ★ Do all external calls have timeouts, with retries only where the call is safe to repeat?
- What happens when a dependency is down? Is the degraded mode documented?
- Do health checks show readiness, including the database?
- Do background jobs stop cleanly and run safely on several instances?

## 9. Performance and cost

- ★ Are current P95/P99 known and within targets?
- Is caching justified, with a clear invalidation rule? Does the cache never hold domain entities, money or stock?
- Do database connections fit at maximum scale?
- Are autoscaling limits chosen on purpose?
- ★ Is the monthly cost known? Are paid APIs capped?

## 10. Operations, delivery and observability

- ★ Is build and deploy automated, with the same artifact in every environment and keyless deploy credentials?
- Can a bad release be rolled back quickly?
- Do logs carry a trace id? Is one request visible end to end in traces?
- Are there alerts on symptoms (error rate, latency), each with a runbook?
- Are ADRs and the system and module maps current? Does CI check that every module is on the map?

## 11. Tests and fitness functions

- ★ Do tests follow the pyramid: many fast unit tests and few slow end-to-end tests?
- Do integration tests use a real database in a container?
- Is every important architecture rule a fitness function (a test or CI step that fails the build): boundaries, folder size, contract drift, anonymous endpoints, cache content?
- Does CI run only what a change affects?

## 12. Frontend (admin, public site, mobile)

- ★ Are boundaries between libraries checked automatically?
- ★ Is code organised by feature, with file and folder size limits?
- ★ Is the API called only through a client typed from the backend contract?
- ★ Is every mutation protected from double submit, reusing the idempotency key on retry?
- Is there one way to work with server data, not ad-hoc fetch in components?
- Is the public site server-rendered for search, and the admin not indexed?
- Are tokens out of JavaScript's reach? Does the backend enforce every permission the UI hides?
- Are Core Web Vitals and accessibility checked?

## 13. AI features in the product

- ★ Does model output go through normal commands, with a person confirming important actions?
- ★ Is untrusted text kept apart from instructions, with tools limited to the user's permissions?
- ★ What happens when the model fails or is slow? Timeout, fallback, validated output.
- ★ What data leaves the system? Is spend capped per user and in total?
- Is quality measured with stored evaluations on every prompt or model change?

