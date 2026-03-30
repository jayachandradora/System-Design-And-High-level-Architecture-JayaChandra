# Design a system to migrate 50TB with handling ~100k Writes Per Sec

Great—this is exactly how you should structure it for interviews. I’ll give you a **clean, end-to-end system design answer** based on that video.

---

# 🎯 1. Problem Statement

👉
**Design a system to migrate 50TB of existing data from an old database to a new database while handling ~100k writes/sec, ensuring:**

* Zero downtime
* No data loss
* Strong consistency

---

# ✅ 2. Functional Requirements (FR)

1. Migrate all **historical data (50TB)**
2. Replicate **live incoming writes (100k RPS)**
3. Keep **old and new DB in sync during migration**
4. Allow **gradual traffic shift** to new system
5. Support **verification of migrated data**

---

# ⚙️ 3. Non-Functional Requirements (NFR)

* 🚀 High throughput (100k RPS)
* ⏱ Low latency
* 🔒 Strong consistency (or eventual with guarantees)
* 💯 No data loss
* 🔁 Fault tolerance & recovery
* 📈 Scalability (horizontal)
* 👀 Observability (lag, errors, metrics)

---

# 🏗 4. High-Level Design (Simple Flow)

```
        +------------------+
        |   Old Database   |
        +------------------+
                 |
         (CDC - Change Logs)
                 |
                 v
        +------------------+
        |   Event Stream   |  (Kafka)
        +------------------+
                 |
     +-----------+-----------+
     |                       |
     v                       v
Snapshot Loader        Stream Consumers
 (Bulk Copy)           (Live Updates)
     |                       |
     +-----------+-----------+
                 |
                 v
        +------------------+
        |   New Database   |
        +------------------+
```

---

# 🧠 5. Best Approach (Step-by-Step)

## 🟢 Step 1: Enable CDC (Change Data Capture)

* Read DB logs (binlog/WAL)
* Capture all INSERT/UPDATE/DELETE
  👉 No missed writes

---

## 🟢 Step 2: Stream changes via queue (Kafka)

* Acts as buffer
* Guarantees durability
* Enables replay

---

## 🟢 Step 3: Bulk load (Snapshot)

* Copy 50TB in chunks
* Parallel workers
* Doesn’t block live traffic

---

## 🟢 Step 4: Replay missed changes

* Use Kafka offsets
* Apply changes after snapshot

---

## 🟢 Step 5: Idempotent consumers

* Handle duplicates safely
* Ensure correctness

---

## 🟢 Step 6: Partitioning

* Partition by key (user_id)
* Maintain order per key
* Scale horizontally

---

## 🟢 Step 7: Traffic shifting (Strangler pattern)

* Shadow reads
* Gradual rollout (1% → 100%)

---

## 🟢 Step 8: Validation

* Row counts
* Checksums
* Sample queries

---

## 🟢 Step 9: Final cutover

* Ensure zero lag
* Switch fully to new DB

---

# 🔍 6. Deep Dive Topics

## 🔸 A. Ordering Problem

* Same user updates must be ordered
  ✅ Solution: partition by key

---

## 🔸 B. Duplicate Events

* Kafka = at-least-once
  ✅ Solution: idempotency
* Use unique event IDs
* Upsert instead of insert

---

## 🔸 C. Lag Handling

* Consumers fall behind

✅ Solutions:

* Scale consumers
* Monitor lag
* Backpressure

---

## 🔸 D. Schema Changes

* Old DB vs new DB mismatch

✅ Solution:

* Versioned schema
* Transformation layer

---

## 🔸 E. Failure Recovery

* Consumer crash

✅ Solution:

* Replay from Kafka
* Offset management

---

# ⚖️ 7. Trade-offs

| Decision             | Trade-off                         |
| -------------------- | --------------------------------- |
| CDC vs Dual Write    | CDC = complex but reliable        |
| Kafka                | Adds infra but ensures durability |
| Eventual consistency | Slight delay but scalable         |
| Idempotency          | Extra logic but prevents bugs     |
| Partitioning         | Fast but tricky ordering          |

---

# ❌ 8. What NOT to Do

* ❌ Stop system → downtime
* ❌ DB dump & restore → outdated data
* ❌ Dual writes → inconsistency
* ❌ No idempotency → duplicates

---

# 🧠 9. Final Interview Summary (Perfect Answer)

👉
**“We use CDC to capture all changes from the old database and stream them through Kafka. Meanwhile, we perform a parallel bulk snapshot of the 50TB dataset. After the snapshot, we replay all missed changes from Kafka to ensure consistency. The system uses partitioning for scalability, idempotent consumers for correctness, and gradual traffic shifting for safe migration, ensuring zero downtime and no data loss.”**

---

# 💡 Bonus Mental Model

👉
**“Copy past data + stream present data + merge both safely.”**

Here’s a **clean, interview-ready architecture diagram** for this system 👇

```
                         ┌──────────────────────┐
                         │     Application      │
                         │   (100k RPS Writes)  │
                         └─────────┬────────────┘
                                   │
                                   v
                         ┌──────────────────────┐
                         │     Old Database     │
                         │   (Primary Source)   │
                         └─────────┬────────────┘
                                   │
                    (CDC - Binlog / WAL Reader)
                                   │
                                   v
                         ┌──────────────────────┐
                         │   CDC Connector      │
                         │   (e.g., Debezium)   │
                         └─────────┬────────────┘
                                   │
                                   v
                         ┌──────────────────────┐
                         │     Kafka Cluster    │
                         │  (Event Streaming)   │
                         └─────────┬────────────┘
                                   │
                ┌──────────────────┼──────────────────┐
                │                  │                  │
                v                  v                  v
     ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
     │ Stream Consumer  │  │ Stream Consumer  │  │ Stream Consumer  │
     │  (Partition 1)   │  │  (Partition 2)   │  │  (Partition N)   │
     └─────────┬────────┘  └─────────┬────────┘  └─────────┬────────┘
               │                     │                     │
               └──────────────┬──────┴──────────────┬──────┘
                              │                     │
                              v                     v
                     ┌────────────────────────────────────┐
                     │         New Database               │
                     │      (Target System)               │
                     └────────────────────────────────────┘


   ─────────────────────────────────────────────────────────────

        (Parallel Path for Historical Data Migration)

                         ┌──────────────────────┐
                         │   Snapshot Service   │
                         │  (Bulk Data Loader)  │
                         └─────────┬────────────┘
                                   │
             Reads in chunks       │        Writes in parallel
        (50TB Historical Data)     v
                         ┌──────────────────────┐
                         │     Old Database     │
                         └──────────────────────┘
                                   │
                                   v
                         ┌──────────────────────┐
                         │     New Database     │
                         └──────────────────────┘


   ─────────────────────────────────────────────────────────────

              (Traffic Shift / Read Routing Layer)

                         ┌──────────────────────┐
                         │   API / Router Layer │
                         │  (Strangler Pattern) │
                         └─────────┬────────────┘
                                   │
                ┌──────────────────┼──────────────────┐
                │                                     │
                v                                     v
     ┌──────────────────────┐             ┌──────────────────────┐
     │     Old Database     │             │     New Database     │
     │   (Gradual Phaseout) │             │    (Final System)    │
     └──────────────────────┘             └──────────────────────┘
```

---

## 🧠 How to explain this diagram (in 20 seconds)

👉

* **Top path**: CDC → Kafka → Consumers → New DB (handles live data)
* **Middle path**: Snapshot service copies 50TB historical data
* **Bottom layer**: Router gradually shifts traffic to new DB

---
Great—this is the **core concept** people struggle with. Let’s make it very clear and practical.

---

# 🧠 What is a “Snapshot” of a Database?

👉
A **snapshot** is a **point-in-time copy of your database**.

* It represents data **exactly as it existed at one moment**
* Even if new writes happen later, snapshot remains unchanged

---

## 📌 Simple analogy

👉 Taking a **photo of a moving train**

* Train = database
* Photo = snapshot
* Even if train keeps moving, photo is fixed

---

# 🎯 Why Snapshot is Needed?

Because:

* You have **50TB existing data**
* You need a **baseline copy** in new DB

👉 Snapshot = “copy all old data once”

---

# ⚠️ Problem During Snapshot

While you are copying:

* New writes keep coming (100k RPS)

👉 So snapshot becomes **outdated immediately**

✔ That’s why we combine it with **CDC (live changes)**

---

# 🚀 How to Perform Parallel Bulk Snapshot (Step-by-Step)

---

## 🟢 Step 1: Choose Snapshot Strategy

There are 3 common ways:

### Option A: DB-native snapshot

* MySQL → `mysqldump`, `xtrabackup`
* Postgres → `pg_dump`

### Option B (BEST): Chunk-based snapshot (used in big systems)

* Divide table into small chunks
* Copy in parallel

👉 We’ll focus on this (most scalable)

---

## 🟢 Step 2: Divide Data into Chunks

👉 Split large tables using a key:

Example:

```text
user_id 1–1M
user_id 1M–2M
user_id 2M–3M
...
```

📌 Each chunk = independent task

---

## 🟢 Step 3: Run Parallel Workers

👉 Start multiple workers:

* Worker 1 → chunk 1
* Worker 2 → chunk 2
* Worker N → chunk N

📌 This speeds up migration massively

---

## 🟢 Step 4: Read from Old DB

Each worker runs queries like:

```sql
SELECT * FROM users
WHERE user_id BETWEEN X AND Y;
```

👉 Reads small, manageable batches

---

## 🟢 Step 5: Write to New DB

* Insert into new DB
* Use batch inserts for speed

---

## 🟢 Step 6: Track Progress

👉 Maintain metadata:

* Which chunks completed
* Which failed
* Retry failed chunks

---

## 🟢 Step 7: Ensure Consistency (IMPORTANT)

👉 Take snapshot at a **specific point in time**

How?

* Record **binlog offset / WAL position**
* This marks snapshot start

📌 This is CRITICAL

---

## 🟢 Step 8: Run CDC in Parallel

While snapshot runs:

* CDC captures all new changes
* Sends them to Kafka

👉 So nothing is missed

---

## 🟢 Step 9: Replay Missing Changes

After snapshot completes:

* Replay events from saved offset
* Apply all updates that happened during snapshot

📌 Now both DBs are in sync

---

# 🔄 Full Flow (Easy View)

1. Start CDC
2. Mark offset (time checkpoint)
3. Start snapshot (parallel copy)
4. Snapshot finishes
5. Replay CDC events
6. New DB catches up

---

# ⚠️ Key Challenges

## 1. Large Tables

* 50TB = slow
  ✅ Solution: parallel chunking

---

## 2. Hot Rows (frequently updated)

* Data changes during copy
  ✅ Solution: CDC replay

---

## 3. DB Overload

* Snapshot can overload DB
  ✅ Solution:
* Throttle queries
* Run during off-peak

---

## 4. Consistency Issues

* Partial data copy
  ✅ Solution:
* Use checkpoint (binlog offset)

---

# ⚖️ Trade-offs

| Approach             | Pros           | Cons                |
| -------------------- | -------------- | ------------------- |
| Full dump            | Simple         | Very slow, downtime |
| Chunking             | Fast, scalable | Complex             |
| DB snapshot (backup) | Consistent     | Infra heavy         |

---

# 💡 Interview One-Liner

👉
**“We take a point-in-time snapshot by chunking large tables and copying them in parallel while recording a log offset. Meanwhile, CDC captures ongoing changes, which are replayed after the snapshot to ensure consistency.”**

---

# 🔥 Final Mental Model

👉
**Snapshot = Past data**
**CDC = Present data**

👉 Combine both = **Fully consistent new system**

---


