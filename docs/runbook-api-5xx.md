# Runbook: the API returns 5xx

How I find the cause when the API starts answering with server errors. The order goes from the cheapest check to the most expensive, and every step either finds the cause or rules something out.

## 1. How big is it?

- Open the request logs of the API service, filtered to `severity>=ERROR`, for the last hour.
- Is it **all requests** or **one endpoint**? Is it **since a moment** or **slowly growing**?
- Check the error share against the target (under 1 % in any 15 minutes). That tells whether to act now or to investigate calmly.

## 2. Is the process alive, and is the database reachable?

| Check | Answers | Means |
| --- | --- | --- |
| `GET /health` | 200 | The process is running |
| `GET /health/ready` | 200 | The process can reach the database |
| `GET /health/ready` | 503 | The database is the problem: check it first, not the code |

Liveness ignores the database on purpose, so a database outage never makes the platform restart healthy instances.

## 3. Did something change?

- **A deploy in the last hour?** Compare the time of the first error with the time of the newest revision.
- **A migration?** It runs as a separate step before the deploy and must keep the previous release working.
- **A configuration or secret change?** A missing secret stops a new revision from starting.

If a deploy lines up with the first error, **roll back to the previous revision first** and investigate after. Rollback is one action, and migrations stay compatible with the previous release, so it is safe.

## 4. Read one failing request end to end

- Take one error log line and open its **trace** by the trace id.
- The trace shows the endpoint, every SQL command and every outgoing call with its duration, and the exception.
- Every error answer is Problem Details with the same trace id, so a screenshot from a user leads to the same trace.

Typical answers:

| What the trace shows | Likely cause |
| --- | --- |
| A slow or failing SQL command | Query, lock or connection limit: check connections against the pool size and the number of instances |
| A slow or failing outgoing call | An external service is down: see the dependency table below |
| An exception in our code | A bug: reproduce it with a test, then fix |
| No spans from the service at all | Telemetry export fails: read the SDK error log, the API itself may be fine |

## 5. Is an external service down?

Every dependency has a written row: what breaks, what keeps working, what the system does on its own, what a person does. For example, when a messaging API is down, requests are still saved and the outbox retries later. Nothing is half-saved, because one command is one transaction.

- **Outbox:** messages that ran out of retries stay visible with their last error, and the team gets a notification.
- **Background tasks:** a failed run stays red on the admin's task page with its error.

## 6. After it is fixed

- Write a short post-incident note: what happened, how it was noticed, the cause, the fix, and what stops it from happening again.
- If it was noticed by a user and not by the system, the first action item is an alert.
- If a rule was broken, the second action item is a test or CI check for that rule.
