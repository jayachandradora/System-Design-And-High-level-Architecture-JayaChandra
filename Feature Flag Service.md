# Feature Flag Service

Feature Flag Service
Requirements
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


For a **Principal Engineer (15+ years)** system design interview, the expectation is not just drawing components but demonstrating trade-offs, failure handling, scalability, consistency, operability, and evolution.

---

# Feature Flag Service Design (Principal Engineer Level)

## 1. Problem Statement

Design a Feature Flag Service similar to:

* LaunchDarkly
* Split.io
* Unleash
* AWS AppConfig

Requirements

Functional

* Applications evaluate feature flags through an API
* Support

  * Boolean flags
  * Percentage rollout
  * User targeting
* Customers manage flags from UI

Non-functional

* 150K evaluations/sec
* 500 writes/min
* Updates visible within 10 seconds
* Reads available even if DB is down
* Low latency (<5ms preferred)

---

# High-Level Architecture

```text
                     +-----------------------+
                     |     Web Dashboard     |
                     +-----------+-----------+
                                 |
                          REST/gRPC API
                                 |
               +-----------------v------------------+
               |      Feature Flag Management       |
               |        (Write Service)             |
               +-----------------+------------------+
                                 |
                           Persist Flags
                                 |
                       +---------v---------+
                       |   SQL Database    |
                       +---------+---------+
                                 |
                          Change Events
                                 |
                     Kafka / Pulsar / SNS
                                 |
        ---------------------------------------------------
        |                 |                 |             |
+-------v------+  +-------v------+  +-------v------+ +----v------+
| Cache Node 1 |  | Cache Node 2 |  | Cache Node 3 | | Cache N   |
+-------+------+  +-------+------+  +-------+------+ +----+------+
        |                 |                 |             |
        ---------------------------------------------------
                           |
                 Feature Evaluation API
                           |
                 +---------v---------+
                 |   Applications    |
                 +-------------------+

```

---

# Separate Read and Write Path

This is the most important architectural decision.

Writes are rare

```
500/min
≈8 writes/sec
```

Reads are huge

```
150,000/sec
```

Never let write path impact reads.

---

# Components

## 1. Dashboard

Customers

* Create flag
* Enable/disable
* Add rollout rules
* Add targeting rules

Example

```
Flag:
CheckoutV2

Enabled

Rollout:
20%

Rules

Country = US

Premium User

Employee
```

---

## 2. Management Service

Responsibilities

Validation

Authentication

Versioning

Audit logs

Store flags

Publish update event

---

## 3. Database

Need transactions.

Use

Postgres

or

CockroachDB

Schema

```
FeatureFlag

id

name

environment

enabled

rules

percentage

version

updated_at
```

Rules stored as JSON.

Example

```
{
   "country":"US",
   "premium":true
}
```

---

## 4. Event Bus

Whenever flag changes

```
Dashboard

↓

Write DB

↓

Publish Event

↓

All caches update
```

Kafka example

```
Topic

feature-flags
```

Message

```
Flag Updated

Version

Timestamp

Payload
```

---

# Why Event Bus?

Requirement

```
Updates within 10 sec
```

Without events

```
Every node polls DB
```

High DB load

Instead

```
Push updates
```

Latency

```
100ms

500ms

1 sec
```

Far below requirement.

---

# Cache Layer

This is the heart of the system.

Every API node stores all flags in memory.

```
Memory Cache

CheckoutV2

PaymentsV3

Pricing

ExperimentABC
```

Reads

```
O(1)
```

No DB lookup.

---

# Evaluation API

Applications call

```
GET /evaluate
```

Example

```
{
 userId:123,
 flag:"CheckoutV2",
 attributes:
 {
   country:"US",
   premium:true
 }
}
```

Response

```
{
 enabled:true
}
```

---

# Why In-memory Cache?

Requirement

```
150K/sec
```

DB cannot serve this.

Memory lookup

```
Microseconds
```

---

# Boolean Flag Evaluation

```
Flag

Enabled = true

Return true
```

Very simple.

---

# Percentage Rollout

Suppose

```
20%
```

Need deterministic result.

Wrong

```
Random()
```

User flips every request.

Correct

```
Hash(userId)

↓

100 buckets

↓

bucket<20

Enabled
```

Example

```
userId=5678

Hash

Bucket=17

20%

Enabled
```

Another user

```
Bucket=87

Disabled
```

Consistent.

---

# Hash Algorithm

```
MurmurHash

xxHash

CRC32
```

Compute

```
hash(userId + flagName)

%

100
```

Important

```
Flag name included

Different experiments

Independent buckets
```

---

# User Targeting

Rules

```
Employee

OR

Premium

OR

Country=US
```

Evaluation

```
Rule Engine

↓

Boolean expression

↓

Enabled
```

Example

```
country=="US"

AND

premium==true

AND

age>25
```

---

# Rule Engine

Internally

Compile rules

Instead of JSON every request

Compile into executable predicates.

```
Rule

↓

AST

↓

Compiled predicate
```

Evaluation becomes fast.

---

# Read Flow

```
Application

↓

API

↓

Memory Cache

↓

Evaluate

↓

Return
```

No DB.

---

# Write Flow

```
Dashboard

↓

Management Service

↓

Postgres

↓

Kafka

↓

Every API node

↓

Memory Cache Updated
```

---

# Meeting 10-second Requirement

Actually

```
DB Write

30ms

↓

Kafka

100ms

↓

Consumer

50ms

↓

Cache refresh

20ms
```

Overall

```
<500ms
```

---

# Reads During Database Outage

Requirement

```
DB Down

Reads continue
```

How?

Entire dataset already in cache.

```
DB

❌

Cache

✓

Evaluation works
```

Writes fail

Reads succeed.

---

# Cache Warm-up

On startup

```
API Node

↓

Load all flags

↓

Memory

↓

Ready
```

---

# Cache Recovery

If Kafka missed message

Need reconciliation.

Every

```
5 min
```

Compare versions

Repair cache.

---

# Cache Versioning

Each flag

```
Version=45
```

Incoming event

```
Version=44
```

Ignore

Older event.

---

# API Scaling

Stateless.

```
Load Balancer

↓

Many API Nodes
```

Scale horizontally.

---

# Storage Estimate

Suppose

```
100,000 flags

2 KB each
```

Memory

```
200 MB
```

Easy.

Even

```
1 million flags

≈2 GB
```

Still manageable.

---

# Multi-region

```
US

EU

Asia
```

Each region

Own API cluster

Own cache

Shared event replication

or

Regional Kafka.

---

# Availability

```
API

99.99%

Cache

99.99%

DB

99.9%
```

Reads unaffected by DB.

---

# Security

Authentication

OAuth

JWT

RBAC

Roles

```
Admin

Developer

Viewer
```

Audit

Every change

```
Who

When

Old value

New value
```

---

# Monitoring

Metrics

```
Evaluation latency

Cache hit ratio

Propagation delay

Failed evaluations

Kafka lag

Flag count

Write latency
```

Alerts

```
Propagation >10 sec

Cache stale

Kafka consumer stopped
```

---

# Scaling Numbers

Reads

```
150K/sec
```

Suppose

One node handles

```
10K/sec
```

Need

```
15 nodes
```

Provision

```
25 nodes
```

for redundancy.

Writes

```
500/min

≈8/sec
```

Very small.

Single DB easily handles.

---

# Failure Scenarios

### Database Down

```
Writes fail

Reads continue
```

---

### Kafka Down

Reads continue.

Writes stored.

Retry publishing.

Background reconciliation.

---

### Cache Restart

```
Load latest snapshot

↓

Subscribe Kafka

↓

Ready
```

---

### Node Crash

LB routes elsewhere.

No impact.

---

# Possible Optimizations

## SDK Evaluation (Industry Standard)

Instead of every request calling server

Applications download flag configuration.

Evaluation happens locally.

```
Application

↓

SDK

↓

Memory

↓

Result
```

Updates

Streaming connection

or

SSE

or

WebSocket.

Latency

```
<1ms
```

Almost zero infrastructure cost.

LaunchDarkly uses this.

---

# Real-World Use Cases

## 1. Safe Production Deployment (Canary Release)

Amazon deploys a new checkout flow.

Initially:

```
CheckoutV2 = 1%
```

If no errors:

```
5%

↓

10%

↓

25%

↓

50%

↓

100%
```

If payment failures spike, they flip the flag off instantly without redeploying.

---

## 2. A/B Testing

Netflix tests a new recommendation algorithm.

```
Group A

Old algorithm

50%

Group B

New algorithm

50%
```

Metrics collected:

* Watch time
* Click-through rate
* Session duration

The winning variant becomes the default.

---

## 3. Premium Feature Gating

A SaaS platform exposes advanced analytics only to premium users.

Rule:

```
if user.plan == "Premium"
    Enable
else
    Disable
```

No separate deployments are needed.

---

## 4. Employee-Only Testing

Before public release:

```
email endsWith "@company.com"
```

Only employees see the new feature, enabling internal validation in production.

---

## 5. Geographic Rollout

A payment provider launches a new payment gateway only in the US.

Rule:

```
country == "US"
```

Once stable, they gradually enable it in Canada and Europe.

---

## 6. Emergency Kill Switch

A recommendation service starts overloading downstream systems.

Operations immediately disable:

```
RecommendationEngine = false
```

Traffic shifts to a simpler fallback, preventing an outage without redeployment.

---

## 7. Holiday Traffic Management

During Black Friday:

* Enable optimized caching
* Disable expensive personalization
* Reduce optional recommendation calls

Feature flags allow operational changes in real time as traffic spikes.

---

# Principal-Level Discussion Points

A strong principal engineer would also discuss:

* Separating the control plane (dashboard, management, storage) from the data plane (evaluation service).
* Local in-memory evaluation to eliminate database dependency for reads.
* Deterministic hashing (e.g., MurmurHash or xxHash) for stable percentage rollouts.
* Event-driven cache propagation with versioning and periodic reconciliation.
* Failure modes (database outage, event bus lag, cache misses) and graceful degradation.
* Operational excellence: audit logs, metrics, propagation SLAs, RBAC, and staged rollouts.
* Evolution toward SDK-based local evaluation for very large-scale deployments, where applications receive streamed flag updates and evaluate rules without network calls, reducing latency and infrastructure costs.
