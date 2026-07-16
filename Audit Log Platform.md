Requirements
Design an audit logging platform for a B2B SaaS product.
Requirements
* Every customer action (login, CRUD operations, permission changes) must be recorded.
* Logs must be immutable.
* Customers can search logs from the last 90 days.
* Older logs must remain available for compliance (7 years).
* Search does not need to be real-time (up to 1 minute delay is acceptable).
* System must continue accepting events even if search infrastructure is unavailable.
* Scale:
    * ~20,000 audit events/sec
    * 10x growth expected within 18 months.

Part 1: Design
1. Core architecture:
    * Components
    * Data flow
    * Storage choices
    * 2–3 hardest-to-reverse decisions and rationale
2. Phased build plan:
    * Phase boundaries
    * Why you would stop after each phase

Part 2: MVP
Implement Phase 1.
Remove everything unnecessary.
Document assumptions.

Part 3: AI Reflection
Same as original.


Here are several **principal backend engineer** system design exercises in the same style as your notification service prompt. They emphasize architecture, trade-offs, phased delivery, and pragmatic implementation rather than designing the "perfect" distributed system.

---

# Problem 1: Audit Log Platform

## Requirements

Design an audit logging platform for a B2B SaaS product.

Requirements

* Every customer action (login, CRUD operations, permission changes) must be recorded.
* Logs must be immutable.
* Customers can search logs from the last 90 days.
* Older logs must remain available for compliance (7 years).
* Search does not need to be real-time (up to 1 minute delay is acceptable).
* System must continue accepting events even if search infrastructure is unavailable.
* Scale:

  * ~20,000 audit events/sec
  * 10x growth expected within 18 months.

---

### Part 1: Design

1. Core architecture:

   * Components
   * Data flow
   * Storage choices
   * 2–3 hardest-to-reverse decisions and rationale

2. Phased build plan:

   * Phase boundaries
   * Why you would stop after each phase

---

### Part 2: MVP

Implement Phase 1.

Remove everything unnecessary.

Document assumptions.

---

### Part 3: AI Reflection

Same as original.

---

# Problem 2: Feature Flag Service

## Requirements

Design a feature flag service.

Requirements

* Applications request flag evaluations through an API.
* Flags support:

  * boolean
  * percentage rollout
  * user targeting
* Updates should propagate within 10 seconds.
* Reads must remain available during database outages.
* Customers manage flags through a dashboard.
* Scale:

  * 150k evaluations/sec
  * 500 writes/minute

---

### Deliverables

Same structure as original.

---

# Problem 3: Payment Retry Orchestrator

## Requirements

Build a payment retry orchestration service.

Requirements

* Receives failed payment events.
* Supports configurable retry policies.
* Retry through different payment providers.
* Never retry after a payment has succeeded.
* Providers occasionally timeout.
* Customers can cancel pending retries.
* Scale:

  * 2M payment failures/day

---

Deliverables identical.

---

# Problem 4: Webhook Delivery Platform

## Requirements

Build a webhook delivery service for third-party integrations.

Requirements

* Internal services publish events.
* Customers register webhook endpoints.
* At-least-once delivery.
* Automatic retries with exponential backoff.
* Dead-letter queue after retry exhaustion.
* Customers can inspect delivery history.
* Slow customer endpoints should not affect others.
* Scale:

  * 50M webhook deliveries/day

---

Deliverables identical.

---

# Problem 5: Distributed Job Scheduler

## Requirements

Design a distributed job scheduler.

Requirements

* Customers schedule:

  * one-time jobs
  * recurring cron jobs
* Jobs trigger HTTP endpoints.
* No duplicate execution.
* Scheduler survives machine failures.
* Pause/resume jobs.
* Expected trigger accuracy:

  * ±5 seconds.
* Scale:

  * 10M scheduled jobs
  * 150k executions/hour

---

Deliverables identical.

---

# Problem 6: Multi-Tenant Search Indexing Service

## Requirements

Design a search indexing pipeline.

Requirements

* Customer documents change continuously.
* Changes appear in search within 2 minutes.
* Re-index entire tenant on demand.
* Search infrastructure may become unavailable.
* One noisy tenant should not affect others.
* Scale:

  * 500M documents
  * 5k document updates/sec

---

Deliverables identical.

---

# Problem 7: API Rate Limiting Service

## Requirements

Design a centralized rate-limiting service.

Requirements

* Shared across all APIs.
* Per-user
* Per-tenant
* Per-endpoint limits
* Burst support.
* Decisions must be under 5ms.
* Continue serving requests during cache failures.
* Configuration updates propagate within 30 seconds.
* Scale:

  * 250k requests/sec

---

Deliverables identical.

---

# Problem 8: Data Export Service

## Requirements

Customers request exports of their data.

Requirements

* Export formats:

  * CSV
  * JSON
* Exports may take hours.
* Notify customers when complete.
* Customers may cancel exports.
* Multiple exports per tenant.
* Large exports should not affect API latency.
* Scale:

  * 50k exports/day

---

Deliverables identical.

---

# Problem 9: Metrics Aggregation Platform

## Requirements

Design a metrics aggregation service.

Requirements

* Applications continuously send metrics.
* Support:

  * counters
  * gauges
  * histograms
* Dashboards refresh every 30 seconds.
* Raw metrics retained for 7 days.
* Aggregates retained for 2 years.
* Scale:

  * 1M metric points/sec

---

Deliverables identical.

---

# Problem 10: File Processing Pipeline

## Requirements

Customers upload files for processing.

Requirements

* Virus scan
* Metadata extraction
* Thumbnail generation
* OCR (optional)
* Processing stages may fail independently.
* Customers can view progress.
* Support retries.
* Scale:

  * 5M uploads/day
  * Files up to 5GB

---

Deliverables identical.

---

# Problem 11: Inventory Reservation Service (Hard)

## Requirements

Build an inventory reservation service.

Requirements

* Multiple buyers reserve inventory concurrently.
* Prevent overselling.
* Reservations expire after configurable TTL.
* Confirmed purchases make reservations permanent.
* Support reservation cancellation.
* Survive service restarts.
* Scale:

  * 80k reservation requests/sec

---

Deliverables identical.

---

# Problem 12: Email Campaign Platform

## Requirements

Design an email campaign system.

Requirements

* Customers upload recipient lists.
* Schedule campaigns.
* Personalization tokens.
* Respect unsubscribe preferences.
* Pause/resume campaigns.
* Email providers have rate limits.
* Support multiple providers.
* Scale:

  * 300M emails/day

---

Deliverables identical.

---

## Why these are good Principal-level exercises

These scenarios naturally test discussion around:

* Queue-based architectures
* Async processing
* Idempotency
* Backpressure
* Retry strategies
* Dead-letter queues
* Event-driven systems
* Multi-tenancy
* Horizontal scaling
* Data partitioning
* Consistency vs. availability trade-offs
* Operational visibility (metrics, tracing, alerts)
* Disaster recovery
* Phased delivery and MVP scoping

They also encourage candidates to justify irreversible architectural choices and prioritize customer value, which are common expectations for principal backend engineering interviews.
