# Case study: reviewing an ERP for a workwear manufacturer

Brevi is a Ukrainian company that sews workwear and resells protective equipment. I built and run its ERP and public website. This page shows three reviews of that system: 14, 17 and 19 November 2026. For each one it shows what I found, what was fixed and how things stand now.

## The system

- **Backend:** a modular monolith on .NET 10, with Minimal API, EF Core and PostgreSQL. It has 14 business modules: catalogue and prices, commercial offers, CRM, production, warehouse, purchasing, accounting, site content and others.
- **Hosting:** Google Cloud Run for the API and a separate Cloud Run job for background tasks, triggered by Cloud Scheduler. Cloudflare sits in front.
- **Frontend:** an Nx workspace with a React admin for staff and a Next.js public site. Both use a typed client generated from OpenAPI.
- **Users:** a small team that uses the admin every day, plus site visitors.

## How I reviewed it

1. **14 November: structure audit.** I compared the code with the project's own architecture rules.
2. **17 November: full review.** I used my [checklist](checklist.md) across both backends and frontends. Each finding was marked with severity and evidence.
3. **19 November: Standard review.** I ran the same checklist with step 0 first, quality targets in numbers, and compared the result with both earlier reviews.

Each finding was fixed in its own pull request, with a test or CI check so it cannot come back.

## Before → after: the code structure

| What | 14 Nov | 19 Nov |
| --- | --- | --- |
| Public contract between modules | None of 12 modules had one. 54 adapters in the host wired modules together | Every module has a Contracts project. Modules talk only through it, and the host only composes |
| Use cases in `Commands/` and `Queries/` folders | 0 of 408 handlers | 448 of 453 (the other 5 are notification handlers) |
| Architecture tests | 9 modules covered, with no checks on contracts, folders or the host | 30 test files: layers, folder size, module isolation, cache content, aggregates, no cycles in the module graph |
| API | MVC controllers in 3 modules, contract written by hand in YAML | Minimal API by feature, OpenAPI generated from code, 437 endpoints in a reviewed snapshot with their access rule and rate limit |
| Architecture decisions | 5 ADRs | 21 ADRs |
| Quality targets | None | 18 targets, with a measured baseline for P95/P99 and peak load |

## Before → after: findings of 17 November

| Found | What was done | Now |
| --- | --- | --- |
| No quality targets | Targets written: 8 confirmed by the owner, 10 proposed. Baseline measured from about 196,000 requests | ✅ Fixed |
| Deploy waited for a manual step | Every merge to the main branch now deploys automatically, with keyless credentials (ADR) | ✅ Fixed |
| A repeated request with the same idempotency key but different data returned the first result | The API returns 409 Conflict. The admin renews the key when the form changes | ✅ Fixed |
| Errors in several formats, and domain types leaking into the API layer | One Problem Details format with a trace id for every error. A test forbids domain types in the API layer | ✅ Fixed |
| One module and the migration history were in the shared `public` schema | Every module keeps its tables and migration history in its own schema | ✅ Fixed |
| No readiness check | A readiness endpoint checks the database | ⚠️ It exists, but the platform does not use it yet |
| No scanning for vulnerable packages | Dependabot on both repositories, plus a vulnerable-package check in CI | ✅ Fixed |
| No alert when an outbox message gives up | The team gets a message when a message runs out of retries | ✅ Fixed |
| No table of "dependency down → what breaks" | A table of 14 dependencies with the effect and the action for each | ✅ Fixed |
| Dead end-to-end tests that used mocks | Removed. End-to-end tests now run against the real backend | ✅ Fixed |
| Traces did not reach production | Traces now arrive in Cloud Trace after the deploy change | ✅ Fixed |
| Activity log kept forever | Retention of one year | ✅ Fixed |
| No alerts or metrics | Alerts added for part of the targets, starting with failed outbox messages | ⚠️ Partly done |

## Before → after: other improvements, 17–19 November

| Topic | Before | After |
| --- | --- | --- |
| Domain model | Parts of aggregates were changed around the root, and some ids were raw Guids | Parts change only through the root. `Create` factories return a Result. Each root has its own id type. Architecture tests check all of this |
| Module graph | 5 pairs of modules depended on each other, plus one three-module cycle | No cycles. An upstream module declares a port in its Contracts, and the downstream module implements it (ADR) |
| Ids shared across modules | Modules stored other modules' ids as raw Guids | Shared typed ids in the domain kernel (ADR) |
| Caching | 8 reads used the in-memory cache, each in its own way | One read-cache port over HybridCache, dropped by tag after a command, holding only immutable read models (ADR plus a test) |
| Rate limiting | Limits only on sign-in and the site request form | A global soft limit plus 6 named policies. Every endpoint's limit is in the reviewed snapshot |
| Webhooks | No rules | Written rules. The incoming secret is compared in constant time |
| Documentation maps | System, module and feature maps kept by hand | A CI script fails when a module or feature is missing from the maps |
| Frontend file size | Long files listed as known exceptions | Split in 5 pull requests. The exception list is now empty |

## Step 0: one month in numbers

The main quality targets, measured from production logs for the last month (30 Oct – 19 Nov).

| Metric | Target | Last month | Result |
| --- | --- | --- | --- |
| Availability (requests without a server error) | 99.5 % | 99.99 % | ✅ Met |
| Reads, P95 / P99 | 1 s / 3 s | 89 ms / 627 ms | ✅ Met |
| Writes, P95 | 2 s | 1.6 s | ✅ Met |
| Server errors | under 1 % | 0.01 % | ✅ Met |
| Traffic | — | about 10,400 requests a day, peak 1,910 per minute | — |
| Cold start (first request after 15 quiet minutes) | — | 4.3 s median, 6.6 s P95 | — |
| Database size | ×10 growth without redesign | 73 MB | ✅ ×10 is still under 1 GB |
| Running cost | — | about $15–20 a month for everything | ✅ |

## Roadmap

1. Extend alerts to every quality target and add an uptime check.
2. Move autoscaling limits and the database pool size into the deploy config.
3. Use the readiness check in the platform and add a smoke test after each deploy.
4. Write a threat model for the public entry points, set a spend cap and add frontend error tracking.
5. Bring the remaining docs up to date, add a "revisit when" condition to every ADR and turn on secret scanning in CI.

## What this gave the business

- **Safer changes.** Architecture rules are enforced by 30 test files and CI checks, not by memory. A change that breaks a module boundary, the API contract or the folder rules does not merge.
- **No double work from double clicks.** Orders, money, stock and outgoing messages can no longer be created twice by a retry.
- **Faster delivery.** A merge deploys by itself, with no manual step.
- **Honest numbers.** The system now has written targets and a measured baseline, so "is it fast enough?" has an answer. The next steps are planned in the roadmap.
