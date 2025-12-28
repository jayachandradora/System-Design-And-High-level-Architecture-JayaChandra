# Payment Gateway-PSP-NPCI-Switch-Banks

Below is a **complete, interview-ready system design walkthrough** for a **High-Throughput Payment Processing System**, structured exactly how senior/staff/principal candidates (15+ YOE) are expected to present.
This mirrors how **FAANG / fintech / staff-level interviews** are evaluated.

---

# 1️⃣ How to Frame the Problem (1–2 mins)

### Problem Overview (Say this first)

> “We need to design a **high-throughput, highly reliable payment processing system** that can process millions of transactions per day with **low latency**, **strong consistency**, **exactly-once processing**, and **regulatory compliance**.”

### Example Use Cases

* Card payments (credit/debit)
* Bank transfers
* Wallet payments
* Refunds & chargebacks
* Payment status tracking

---

# 2️⃣ Business Goals (What interviewer listens for)

Explicitly call these out.

| Goal                        | Why it matters                |
| --------------------------- | ----------------------------- |
| High availability (99.99%+) | Revenue critical              |
| No double charge            | Trust & compliance            |
| Fast confirmation (<300ms)  | User experience               |
| Scalability                 | Peak events (sales, holidays) |
| Auditable                   | Legal & compliance            |

📌 **Highlight**: *“Correctness is more important than latency in payments.”*

---

# 3️⃣ Technical Goals (Translate business → tech)

Mention tradeoffs.

* **Strong consistency** for balances
* **Idempotency** for retries
* **Exactly-once semantics**
* **Horizontal scalability**
* **Fault tolerance**
* **Event-driven architecture**
* **Observability & auditability**

📌 **Senior signal**: Explicitly say *CAP trade-offs*

> “We prefer **CP over AP** for money movement.”

---

# 4️⃣ Core Functional Requirements

### Must-Have

* Create payment
* Authorize payment
* Capture / settle payment
* Refund
* Get payment status

### Non-Functional

* 50k–100k TPS
* P99 latency < 500ms
* Multi-region DR
* PCI-DSS compliance

---

# 5️⃣ High-Level Architecture (Draw This)

```
Client
  |
API Gateway
  |
Payment Orchestrator
  |
------------------------------
|        |        |          |
Auth   Ledger   Risk     Notification
Service Service Engine      Service
  |
External Payment Providers
```

📌 **Highlight**

* Separation of **orchestration vs execution**
* Ledger is **source of truth**

---

# 6️⃣ Core Components Breakdown (This is critical)

### 1. API Gateway

* Auth, rate limiting
* Idempotency key validation
* Request validation

### 2. Payment Orchestrator

* Stateless
* Handles workflow:

  * INIT → AUTHORIZED → CAPTURED → SETTLED
* Publishes events

📌 Mention **Saga pattern**

---

### 3. Ledger Service (MOST IMPORTANT)

> “Ledger is append-only and immutable.”

* Double-entry accounting
* Strong consistency
* ACID transactions
* Never update balances directly

📌 **This is where senior candidates shine**

---

### 4. Risk / Fraud Engine

* Async scoring
* Rules + ML models
* Can block or delay payments

---

### 5. External Payment Processors

* Stripe, Visa, Bank APIs
* Network failures expected
* Wrapped with retry + circuit breakers

---

# 7️⃣ API Design (Show clarity)

### Create Payment

```
POST /payments
Headers:
  Idempotency-Key: uuid

Request:
{
  "amount": 100,
  "currency": "USD",
  "source": "card_123",
  "destination": "merchant_456"
}
```

### Response

```
{
  "payment_id": "pay_789",
  "status": "PENDING"
}
```

📌 **Call out idempotency explicitly**

---

# 8️⃣ Database Design (VERY IMPORTANT)

### Payments Table

| Field           | Notes   |
| --------------- | ------- |
| payment_id (PK) | UUID    |
| status          | ENUM    |
| amount          | DECIMAL |
| currency        |         |
| created_at      |         |

---

### Ledger Entries (Append Only)

| entry_id | account_id | debit | credit | txn_id |
| -------- | ---------- | ----- | ------ | ------ |

📌 **Explain double-entry accounting**

> “Sum(debits) == Sum(credits) — always.”

---

### Idempotency Table

| key | request_hash | response |

---

# 9️⃣ High-Level Data Flow (Walk Through)

### Payment Flow

1. Client → API Gateway
2. Idempotency check
3. Create payment record
4. Risk check (async)
5. Call external processor
6. On success → ledger entry
7. Emit `PaymentCompleted` event

📌 Mention **eventual consistency for notifications**

---

# 🔟 Deep Dive Topics (Choose 2–3 based on interviewer)

### A) Exactly-Once Processing

* Idempotency keys
* Deduplication table
* Transactional outbox

---

### B) Failure Handling

* Network timeout → retry
* Partial failure → compensation
* Circuit breakers

---

### C) Scaling Strategy

* Stateless services
* Partition by payment_id
* Shard ledger by account_id
* Kafka for event streams

---

### D) Concurrency Control

* Optimistic locking
* Serializable isolation for ledger

---

# 1️⃣1️⃣ Consistency & CAP Discussion (Senior Signal)

> “Payments require **linearizability**. We sacrifice availability during partitions to avoid double spending.”

---

# 1️⃣2️⃣ Security & Compliance

* PCI DSS
* Tokenization
* Encryption at rest & transit
* Role-based access
* Audit logs

---

# 1️⃣3️⃣ Observability

* Payment lifecycle metrics
* Dead-letter queues
* Distributed tracing
* Reconciliation jobs

📌 Mention **reconciliation between ledger & external providers**

---

# 1️⃣4️⃣ Bottlenecks & Optimizations

| Area           | Strategy                |
| -------------- | ----------------------- |
| DB             | Sharding                |
| External calls | Async + retries         |
| Ledger         | Write-optimized storage |
| Traffic spikes | Queue buffering         |

---

# 1️⃣5️⃣ How to Present This as 15+ Years Experience

### What to Emphasize

✔ Trade-offs
✔ Failure modes
✔ Data correctness
✔ Financial-grade guarantees
✔ Real-world constraints

### What NOT to Over-focus On

❌ UI
❌ Micro-optimizations
❌ Framework choices

---

# 1️⃣6️⃣ Sample Closing Statement (Very Powerful)

> “The key insight is that **payments are accounting problems**, not CRUD problems.
> The system is optimized for correctness, auditability, and failure recovery before throughput.”

---

Below is a **crisp, interview-ready explanation** of **Exactly-Once Processing** using **short, clear answers with examples** — perfect for system design deep dives.

---

## Exactly-Once Processing (What & Why)

**What:**
Each payment must be **processed once and only once**, even if the client retries or services crash.

**Why:**
Duplicate processing = **double charge or double credit** (financial disaster).

---

## 1️⃣ Idempotency Keys

### What

A **unique client-generated key** sent with every payment request to ensure retries don’t create duplicates.

### How it works

* Client sends the same key on retry
* Server stores key → response mapping
* If key already exists → return previous response

### Example

```
POST /payments
Idempotency-Key: abc-123
```

**First request**

* Key not found → process payment
* Store result

**Retry**

* Key found → return same payment_id

📌 **Interview one-liner**

> “Idempotency keys make retries safe.”

---

## 2️⃣ Deduplication Table

### What

A **database table** that prevents processing the same logical request multiple times.

### How it works

* Store `(idempotency_key, request_hash)`
* Enforce **unique constraint**
* DB guarantees only one insert succeeds

### Example Table

| idempotency_key | request_hash | payment_id |
| --------------- | ------------ | ---------- |
| abc-123         | hash(req)    | pay_789    |

**Race condition scenario**

* Two identical requests arrive simultaneously
* One insert succeeds
* Other fails → read existing record

📌 **Interview one-liner**

> “The database enforces exactly-once via uniqueness.”

---

## 3️⃣ Transactional Outbox

### Problem it solves

> “DB commit succeeded but event publish failed.”

### What

A pattern to **atomically save state changes and events** in the same DB transaction.

### How it works

1. Write payment + outbox event in one transaction
2. Commit
3. Background worker publishes event
4. Mark event as sent

### Example Outbox Table

| event_id | type             | payload | status  |
| -------- | ---------------- | ------- | ------- |
| evt_1    | PaymentCompleted | {...}   | PENDING |

### Example Flow

```
BEGIN TRANSACTION
  Insert payment
  Insert outbox event
COMMIT
```

📌 **Interview one-liner**

> “Outbox guarantees we never lose events.”

---

## How They Work Together (Important)

| Mechanism            | Protects Against      |
| -------------------- | --------------------- |
| Idempotency key      | Client retries        |
| Deduplication table  | Concurrent duplicates |
| Transactional outbox | Partial failures      |

### End-to-End Example

1. Client retries payment due to timeout
2. Idempotency key prevents duplicate payment
3. Dedup table blocks race condition
4. Outbox ensures event is published exactly once

---

## Final Senior-Level Summary (Say This)

> “Exactly-once processing is achieved by combining **idempotent APIs**, **database-enforced deduplication**, and **transactional outbox** to handle retries, concurrency, and failures.”

---

If you want, I can also:

* Compare this with **Kafka exactly-once semantics**
* Show **whiteboard-friendly diagrams**
* Explain **why distributed transactions are avoided**

Below is a **short, crisp, interview-ready explanation** of **Failure Handling** patterns, with **simple examples and one-liners** — exactly what interviewers expect in deep dives.

---

## Failure Handling (Why It Matters)

**Why:**
Failures are normal in distributed systems. Payment systems must **recover safely without double charging or losing money**.

---

## 1️⃣ Network Timeout → Retry

### What

Retry the request when a **temporary network failure** occurs.

### How

* Use **timeouts + exponential backoff**
* Retry only **idempotent operations**

### Example

* Payment service calls bank API
* Network times out (no response)
* Retry with same **idempotency key**

```
Retry after: 100ms → 300ms → 1s
```

📌 **Interview one-liner**

> “Retries are safe only with idempotency.”

---

## 2️⃣ Partial Failure → Compensation

### What

Undo completed steps when later steps fail.

### How

* Use **Saga pattern**
* Each step has a compensating action

### Example

1. Debit customer wallet ✅
2. Credit merchant wallet ❌ (fails)
3. Trigger compensation → refund customer

```
Debit → Credit ❌ → Refund
```

📌 **Interview one-liner**

> “Compensation restores consistency without distributed transactions.”

---

## 3️⃣ Circuit Breakers

### What

Prevent repeated calls to a failing dependency.

### States

* **Closed** → normal traffic
* **Open** → fail fast
* **Half-open** → test recovery

### Example

* Bank API starts failing
* Circuit opens after 5 failures
* New payments fail fast or go to queue

```
Client → ❌ Bank API (blocked)
```

📌 **Interview one-liner**

> “Circuit breakers protect the system from cascading failures.”

---

## How They Work Together (Important)

| Failure           | Solution        |
| ----------------- | --------------- |
| Temporary glitch  | Retry           |
| Partial success   | Compensation    |
| Dependency outage | Circuit breaker |

### End-to-End Example

1. Payment call times out → retry
2. Debit succeeds, credit fails → refund
3. Bank API down → circuit breaker opens

---

## Senior-Level Summary (Say This)

> “We handle failures by combining **safe retries**, **compensating actions**, and **circuit breakers** to maintain correctness and prevent cascading outages.”

---

# PSP Design

Below is a **complete, interview-ready system design** for **UPI-like systems (GPay / Paytm / PhonePe)**, structured **exactly** how you should present it in a **staff / principal (15+ years)** interview.

This is **India-scale, real-money, bank-to-bank** design — interviewers care more about **correctness, failure handling, and protocol understanding** than UI or tech stack.

---

# 1️⃣ How to Start (1–2 minutes)

### Problem Overview (Say this first)

> “We need to design a **real-time, bank-to-bank payment system** like UPI that enables **instant money transfer** between users using virtual identifiers, with **strong consistency**, **high availability**, and **regulatory-grade correctness**.”

### Key Characteristics

* Real-time (24x7)
* Bank-backed (not wallet-only)
* Interoperable across banks
* Zero/low latency user experience

📌 **Senior signal:**

> “UPI is fundamentally a **distributed banking protocol**, not just a payments app.”

---

# 2️⃣ Business Goals

Clearly separate business vs technical.

| Goal              | Why                     |
| ----------------- | ----------------------- |
| Instant payments  | User adoption           |
| No money loss     | Trust & RBI compliance  |
| Interoperability  | Any bank → any bank     |
| High availability | National infrastructure |
| Auditability      | Regulatory requirement  |

📌 Highlight: **Trust > Latency**

---

# 3️⃣ Technical Goals (Translate business → tech)

* Strong consistency for balances
* Exactly-once money movement
* Idempotent APIs
* Fault tolerance
* Horizontal scalability
* Bank-grade security
* End-to-end audit trail

📌 **CAP tradeoff**

> “UPI prefers **CP** over AP — correctness over availability.”

---

# 4️⃣ Core Functional Requirements

### Functional

* Register VPA (Virtual Payment Address)
* Initiate payment
* Approve payment (PIN)
* Credit beneficiary
* Payment status query
* Refunds

### Non-Functional

* <300ms P99 latency
* 100k+ TPS at peak
* Multi-region DR
* RBI compliance

---

# 5️⃣ Key Actors (Very Important for UPI)

| Actor         | Role                     |
| ------------- | ------------------------ |
| Payer App     | GPay / Paytm             |
| PSP           | Payment Service Provider |
| NPCI          | Central switch           |
| Issuer Bank   | Payer’s bank             |
| Acquirer Bank | Receiver’s bank          |

📌 **Senior signal**

> “UPI is a **4-party system** with a central switch.”

---

# 6️⃣ High-Level Architecture (Draw This)

```
User App
   |
PSP (GPay / Paytm)
   |
NPCI Switch
   |
-------------------------
|                       |
Issuer Bank        Acquirer Bank
```

📌 Emphasize:

* Apps do NOT directly talk to banks
* NPCI routes & validates

---

# 7️⃣ Core Components Breakdown

### 1. PSP Backend (GPay / Paytm)

* User authentication
* VPA resolution
* Idempotency handling
* Payment orchestration
* Retry & reconciliation

---

### 2. NPCI Switch

* Routing logic
* Validation
* Rate limiting
* Settlement coordination

📌 Say:

> “NPCI is logically stateless but operationally critical.”

---

### 3. Issuer Bank (Payer Bank)

* Balance check
* Debit account
* PIN verification
* Ledger entry

---

### 4. Acquirer Bank (Payee Bank)

* Credit account
* Beneficiary confirmation
* Ledger update

---

# 8️⃣ API Design (Simplified)

### Initiate Payment

```
POST /upi/pay
{
  "payer_vpa": "a@bank",
  "payee_vpa": "b@bank",
  "amount": 100,
  "txn_id": "uuid"
}
```

### Response

```
{
  "status": "PENDING"
}
```

📌 Highlight **txn_id = idempotency key**

---

# 9️⃣ Database Design (Critical for Senior Level)

### Transaction Table

| txn_id | payer | payee | amount | status |
| ------ | ----- | ----- | ------ | ------ |

### Ledger (Bank Side – Double Entry)

| entry_id | account | debit | credit | txn_id |
| -------- | ------- | ----- | ------ | ------ |

📌 **Say explicitly**

> “Ledger is append-only; balances are derived.”

---

# 🔟 End-to-End Data Flow (Explain Slowly)

### Happy Path

1. User initiates payment
2. PSP validates & sends to NPCI
3. NPCI routes to Issuer Bank
4. Issuer debits payer
5. NPCI routes to Acquirer Bank
6. Acquirer credits payee
7. Success propagated back

📌 Emphasize **two-phase flow**: debit → credit

---

# 1️⃣1️⃣ Failure Scenarios (VERY IMPORTANT)

### Scenario 1: Debit Success, Credit Failure

* Money debited
* Credit fails
* Transaction marked **PENDING**
* Reversal initiated

📌 Mention **T+1 reconciliation**

---

### Scenario 2: Network Timeout

* Retry with same txn_id
* Idempotency prevents double debit

---

### Scenario 3: PSP Crash

* Bank & NPCI continue
* PSP reconciles later

---

# 1️⃣2️⃣ Exactly-Once Processing (UPI Style)

| Layer      | Mechanism             |
| ---------- | --------------------- |
| Client     | txn_id                |
| PSP        | Idempotency table     |
| Bank       | Unique txn constraint |
| Settlement | Reconciliation        |

📌 Say:

> “Exactly-once is enforced collectively, not by one service.”

---

# 1️⃣3️⃣ Settlement & Reconciliation (Senior Highlight)

* Real-time authorization
* Deferred settlement (netting)
* Daily bank reconciliation
* Dispute handling

📌 Interviewers LOVE this topic.

---

# 1️⃣4️⃣ Security & Compliance

* PIN-based auth
* HSM for key storage
* Tokenization
* End-to-end encryption
* Audit logs (immutable)

---

# 1️⃣5️⃣ Scaling Strategy

| Layer     | Scaling            |
| --------- | ------------------ |
| PSP       | Stateless services |
| NPCI      | Horizontal routing |
| Banks     | Sharded ledgers    |
| Messaging | Async queues       |

📌 Mention **backpressure handling**

---

# 1️⃣6️⃣ What to Highlight as 15+ Years Experience

### MUST Highlight

✔ Trade-offs
✔ Failure handling
✔ Ledger correctness
✔ Reconciliation
✔ Regulatory constraints

### Avoid

❌ UI details
❌ Framework debates
❌ Over-engineering

---

# 1️⃣7️⃣ Powerful Closing Statement

> “UPI systems succeed not because they’re fast, but because they’re **correct under failure**.
> Every design choice prioritizes **money safety, auditability, and recovery**.”

---

Below is a **clear, correctness-focused explanation** of **Scenario 2: Network Timeout** — written exactly at a **15+ years system design depth**.
I’ll walk through **why it’s hard**, **how txn_id + idempotency solve it**, and **what guarantees correctness under failure**.

---

# Scenario: Network Timeout During Payment

### What Actually Goes Wrong

> “Client doesn’t know whether the bank **received**, **processed**, or **completed** the request.”

This is the **hardest problem** in payments.

---

## Step-by-Step Failure Timeline (Realistic)

### Actors

* Client / PSP (GPay / Paytm)
* NPCI
* Issuer Bank

---

### Step 1: Request Sent

```
txn_id = TXN123
Pay ₹100 from A → B
```

PSP → NPCI → Issuer Bank

---

### Step 2: Timeout Happens

* PSP waits for response
* Network times out
* PSP does **not** know:

  * Was debit done?
  * Was request received?

📌 **Key risk**: retry may cause **double debit**

---

# Correctness Strategy (Two Pillars)

## Pillar 1️⃣: Retry with Same `txn_id`

### Why Same `txn_id` Matters

* `txn_id` is a **global unique transaction identifier**
* Used across:

  * PSP
  * NPCI
  * Issuer Bank
  * Acquirer Bank

📌 **Senior one-liner**

> “txn_id is the identity of the payment, not the request.”

---

### Retry Flow

```
Retry Payment:
txn_id = TXN123
```

The system **must treat it as the same transaction**, not a new one.

---

## Pillar 2️⃣: Idempotency at the Bank (Critical)

### Bank-Side Guarantee (MOST IMPORTANT)

At the **Issuer Bank**:

* `txn_id` has a **unique constraint**
* Debit operation is **idempotent**

### Bank DB Example

#### Transactions Table

| txn_id | status  | amount |
| ------ | ------- | ------ |
| TXN123 | SUCCESS | 100    |

#### Ledger Table

| entry_id | account | debit | txn_id |
| -------- | ------- | ----- | ------ |

📌 Unique index on `txn_id`

---

### What Happens on Retry?

#### Case 1: Debit Already Done

* Bank sees `txn_id = TXN123`
* Finds existing record
* Returns **SUCCESS (cached result)**
* ❌ No second debit

#### Case 2: Debit Not Done Yet

* Bank processes debit
* Stores ledger entry
* Returns SUCCESS

📌 **Correctness Guarantee**

> “At most one debit per txn_id.”

---

# Why PSP Retry is Safe

| Risk              | Protection        |
| ----------------- | ----------------- |
| Double debit      | Bank idempotency  |
| Duplicate request | txn_id uniqueness |
| Partial execution | Bank state lookup |

---

# Handling Ambiguous Outcomes (Very Senior Topic)

### Problem

* PSP still didn’t get response after retries

### Solution: **Status Query**

```
GET /upi/txn-status?txn_id=TXN123
```

### Bank Responds

* SUCCESS → PSP updates UI
* PENDING → wait/retry
* FAILED → retry or reverse

📌 Say explicitly:

> “Retries are bounded; after that we switch to status polling.”

---

# Why This Preserves Correctness Under Failure

### Key Properties

| Property              | How Achieved     |
| --------------------- | ---------------- |
| No double debit       | Unique txn_id    |
| Safe retries          | Idempotent debit |
| Deterministic outcome | Status query     |
| Crash recovery        | Persistent state |

---

# What Happens if Bank Crashes Mid-Debit?

### Bank uses ACID transaction

```
BEGIN
  Insert txn TXN123
  Insert ledger entry
COMMIT
```

* Either fully committed or not at all
* On recovery, txn table is source of truth

📌 **Senior signal**

> “Ledger correctness relies on database atomicity.”

---

# Why NOT Use Distributed Transactions?

Say this if asked:

> “2PC increases latency and reduces availability. UPI relies on **idempotency + reconciliation** instead.”

---

# Interview-Grade Summary (Say This)

> “Network timeouts are handled by retrying with the same `txn_id`.
> Idempotency at the bank ensures **at-most-once debit**, and status APIs resolve ambiguity — guaranteeing correctness even under failure.”

---


