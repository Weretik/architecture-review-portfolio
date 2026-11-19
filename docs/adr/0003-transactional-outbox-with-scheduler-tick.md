# ADR 0003: Transactional outbox, driven by a scheduler tick

**Status:** accepted

## Context

Many use cases save data and then must do something outside: send a message, call a CRM or ERP, notify another module. If the save succeeds and the send fails, or the other way round, data is lost or sent twice. The host runs on Cloud Run and scales to zero, so a background loop inside the process is not reliable.

## Alternatives considered

- **Call the outside service inside or right after the transaction.** Rejected. A failure on either side breaks consistency.
- **A message broker (Pub/Sub, RabbitMQ, Kafka).** Rejected. It means more infrastructure and cost than the load needs.
- **An in-process scheduler (Hangfire, Quartz).** Rejected. It does not run while the host is scaled to zero.
- **Transactional outbox plus an external tick.** Chosen.

## Decision

- A use case writes an outbox row in the same transaction as its data.
- After commit, the request tries the row once. A cloud scheduler calls one protected internal endpoint every few minutes, and that endpoint sends what is left and starts due background tasks.
- Handlers are idempotent: the receiver gets a stable key and ignores a repeat.
- A row that fails after the retry limit stays visible with its last error and raises an alert. Nothing is dropped silently.
- Not used when the user must see the external result at once.

## Consequences

- Save and send are consistent, with no broker to run or pay for.
- A side effect may be delayed by up to one tick.
- Every handler must be safe to run twice.

## Revisit when

The volume of messages or the number of consumers outgrows a database table and a periodic tick.
