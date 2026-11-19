# Architecture decisions

Short records of choices that are expensive to change. Each one says what was chosen, what was rejected and why, and when to look at it again. A record is never deleted. If a decision changes, a new record replaces it.

| ADR | Decision |
| --- | --- |
| [0001](0001-modular-monolith.md) | One deployable modular monolith, not microservices |
| [0002](0002-modules-talk-through-contracts.md) | Modules talk only through Contracts, and the module graph has no cycles |
| [0003](0003-transactional-outbox-with-scheduler-tick.md) | Transactional outbox, driven by a scheduler tick instead of a message broker |
| [0004](0004-openapi-generated-from-code.md) | The OpenAPI contract is generated from the code, and clients use a typed client |
| [0005](0005-idempotency-keys-for-creates.md) | Creates that must not happen twice take an idempotency key |
