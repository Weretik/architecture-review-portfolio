# ADR 0005: Idempotency keys for creates that must not happen twice

**Status:** accepted

## Context

Users double-click, mobile apps retry after a timeout, and offline queues resend. Without protection, a repeat creates a second order, payment, stock movement or outgoing message.

## Alternatives considered

- **Disable the button in the UI only.** Rejected. It does not stop network retries.
- **Detect duplicates by comparing data.** Rejected. Two real orders can look the same.
- **A client-generated key per attempt, stored with the result.** Chosen.

## Decision

- Creates of orders, money, stock, numbered documents, public form submissions and outgoing messages take an idempotency key.
- The key is saved in the same transaction as the result, under a unique index.
- A repeat with the same key and the same data returns the first result.
- The same key with different data returns 409 Conflict.
- A race between two copies is caught by the unique index and answered, never with a 500.
- The client makes one key per attempt, reuses it on retry, and renews it after success or a change of data.
- Background handlers, webhooks and scheduled tasks check state or dedupe before acting.

## Consequences

- A repeated request never does the work twice.
- Every such endpoint and every client mutation needs a little extra code and a test.

## Revisit when

Requests start to arrive from outside clients that cannot send a key. Then those endpoints need another way to detect repeats.
