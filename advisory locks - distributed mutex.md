# advisory locks - distributed mutex

Let’s slow down and build the understanding **from first principles**, step by step, with **one concrete purge example** and **real code**.

I’ll explain:

1. **What problem exists in distributed systems**
2. **What advisory lock really is internally**
3. **How multiple servers coordinate without talking to each other**
4. **Step-by-step execution with 3 pods**
5. **Concrete Java + SQL code**
6. **Why this completely solves your purge problem**

---

# 1️⃣ The core problem in distributed jobs

You have:

* 3 pods (servers)
* Same Java code
* Same `@Scheduled` time
* Same database
* Same table: `emp_details`

At 2:00 AM:

```
Pod A → starts purge
Pod B → starts purge
Pod C → starts purge
```

They **do not know about each other**.

Result:

* Multiple DELETEs
* Serializable conflicts
* Errors

---

# 2️⃣ What an advisory lock REALLY is (internals)

Think of the database as holding a **global in-memory map**:

```
LOCK_KEY → OWNER_SESSION
```

Example:

```
"emp_details_purge" → Session 42 (Pod A)
```

This map:

* Lives inside the DB cluster
* Is shared across **all connections, all pods**
* Is managed by the database engine itself

⚠️ Important:

* Advisory locks are **not on tables**
* They are **just entries in this map**

---

# 3️⃣ How coordination happens without servers talking

### All pods run this SQL:

```sql
SELECT pg_try_advisory_lock('emp_details_purge');
```

### Database logic (simplified):

```
if lock_key not present:
    store (lock_key → this_session)
    return true
else:
    return false
```

That’s it.
No network calls between pods.
No Kubernetes awareness.
Only the **database** is the coordinator.

---

# 4️⃣ Step-by-step timeline (3 pods)

### ⏰ Time = 02:00:00 AM

All pods wake up.

---

### 🟢 Pod A

```
DB: "Is emp_details_purge free?"
DB: Yes
DB: Assign lock to Pod A session
DB → Pod A: true
```

Pod A continues.

---

### 🔴 Pod B

```
DB: "Is emp_details_purge free?"
DB: No (Pod A owns it)
DB → Pod B: false
```

Pod B exits job.

---

### 🔴 Pod C

Same as Pod B.

---

### 🟢 Pod A runs purge

```sql
DELETE FROM emp_details
WHERE last_updated < now() - interval '30 days';
```

* DB row locks ONLY old rows
* New inserts/updates continue normally

---

### 🟢 Pod A finishes

```sql
SELECT pg_advisory_unlock('emp_details_purge');
```

DB removes lock entry.

---

### 🕑 Next day

Same process again.

---

# 5️⃣ Why this works in distributed systems

| Problem                    | Solution                            |
| -------------------------- | ----------------------------------- |
| No shared memory           | Database acts as shared coordinator |
| Pods don’t know each other | They all know the DB                |
| Network partitions         | DB guarantees correctness           |
| Pod crash                  | Lock released when connection dies  |

---

# 6️⃣ What happens if Pod A crashes?

Very important question.

* Advisory lock is **owned by DB session**
* Pod crashes → DB connection closes
* DB automatically removes the lock
* Next pod can acquire it

✔ No deadlocks
✔ No manual cleanup

---

# 7️⃣ Full working Java example (detailed)

---

## Step 1: Lock service (low level)

```java
@Component
public class AdvisoryLockService {

    private final JdbcTemplate jdbcTemplate;

    public AdvisoryLockService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public boolean tryLock(String lockName) {
        return Boolean.TRUE.equals(
            jdbcTemplate.queryForObject(
                "SELECT pg_try_advisory_lock(hashtext(?))",
                Boolean.class,
                lockName
            )
        );
    }

    public void unlock(String lockName) {
        jdbcTemplate.execute(
            "SELECT pg_advisory_unlock(hashtext('" + lockName + "'))"
        );
    }
}
```

Why `hashtext()`?

* Converts string → int lock key
* Same string = same lock across all pods

---

## Step 2: Purge job (business logic)

```java
@Scheduled(cron = "0 0 2 * * *")
public void purgeEmpDetails() {

    log.info("Starting emp_details purge job");

    if (!lockService.tryLock("emp_details_purge")) {
        log.info("Another pod is running purge. Exiting.");
        return;
    }

    try {
        log.info("Lock acquired. Purging old records.");

        int deleted = jdbcTemplate.update("""
            DELETE FROM emp_details
            WHERE last_updated < now() - interval '30 days'
        """);

        log.info("Deleted {} old records from emp_details", deleted);

    } finally {
        lockService.unlock("emp_details_purge");
        log.info("Lock released");
    }
}
```

---

# 8️⃣ What happens to live traffic?

While purge runs:

| Operation         | Result  |
| ----------------- | ------- |
| INSERT new emp    | Works   |
| UPDATE recent emp | Works   |
| SELECT            | Works   |
| Another purge     | Skipped |

Because:

* Advisory lock affects **only code that asks for it**
* DELETE locks only the rows it deletes

---

# 9️⃣ Why this avoids your original error

Original error:

```
could not serialize access due to concurrent update
```

Cause:

* Multiple DELETEs running concurrently

With advisory lock:

* Only one DELETE ever runs
* No serialization conflict
* No retries
* No failures

---

# 🔑 Key mental model (this is the “aha”)

> **Advisory lock = “Before doing this job, ask the database for permission.”**

And:

> **The database is the only place all servers can reliably agree.**

---

# Final summary (memorize this)

* Advisory lock is NOT a data lock
* It is a **distributed mutex**
* DB stores it
* Pods coordinate through DB
* Perfect for jobs
* Zero impact on live traffic

---

The **distributed mutex (advisory lock) is handled entirely by the PostgreSQL / YugabyteDB server**, **not** by your job servers.

## Where the mutex actually lives

### ❌ NOT on:

* Job server
* Spring Boot application
* Kubernetes
* JVM memory

### ✅ It lives in:

* **PostgreSQL / YugabyteDB**
* Inside the **database engine**
* Shared across **all connections from all servers**

---

## Why the database is the right place

In a distributed system:

| Component         | Shared by all pods? |
| ----------------- | ------------------- |
| JVM memory        | ❌ No                |
| Local file system | ❌ No                |
| Kubernetes pod    | ❌ No                |
| Database          | ✅ Yes               |

So the DB becomes the **single source of truth** for coordination.

---

## What PostgreSQL/Yugabyte actually does internally (simplified)

Inside the DB server, there is an internal structure like:

```
AdvisoryLockTable
-------------------------
Lock Key        | Session ID
-------------------------
emp_details_purge | 83421
```

* Session ID = DB connection
* If session dies → entry is removed automatically

---

## How your job servers interact with it

Each job server:

1. Opens a DB connection
2. Executes:

   ```sql
   SELECT pg_try_advisory_lock(...)
   ```
3. DB checks internal lock table
4. DB returns `true` or `false`

The job server does **no coordination logic** itself.

---

## Example with your purge job

### Infrastructure

```
Pod A ─┐
Pod B ─┼──► PostgreSQL / YugabyteDB ◄── distributed mutex
Pod C ─┘
```

### Execution

```
Pod A → lock acquired
Pod B → lock denied
Pod C → lock denied
```

Only Pod A proceeds.

---

## What happens if a pod crashes?

Very important and often misunderstood:

* Pod crashes
* DB connection closes
* PostgreSQL/Yugabyte automatically releases the advisory lock

✔ No orphan locks
✔ No manual cleanup
✔ No deadlocks

---

## Why this is called a “distributed mutex”

Because:

* Multiple independent processes
* On different machines
* Coordinate access
* Using a shared authority (DB)

That is exactly the definition of a distributed mutex.

---

## Final mental model (one-liner)

> **Advisory lock = “Ask the database for exclusive permission before running the job.”**

The database enforces it.
The job servers simply obey it.

---
You’re welcome 🙂
You asked exactly the *right* questions, so here are **real, production-grade use cases** where **advisory locks (distributed mutex)** are *intentionally* used.

I’ll group them from **most common** to **advanced**, with concrete examples so you can immediately relate.

---

# 1️⃣ Scheduled jobs / batch jobs (MOST COMMON)

### Problem

Multiple pods start the same job at the same time.

### Example

* Purge old data
* Rebuild aggregates
* Generate reports
* Archive records

### Solution

Use advisory lock to allow **only one executor**.

```text
Lock: "emp_details_purge"
```

✔ Exactly your case
✔ Used in almost every enterprise system

---

# 2️⃣ Idempotent background processing

### Problem

Same event/message can be processed by multiple workers.

### Example

* Kafka consumer rebalance
* Retry of failed job
* Duplicate message delivery

### Use case

Ensure:

> “This business operation runs only once.”

### Example

```text
Lock: "process_order_12345"
```

Only one worker processes order `12345`.

---

# 3️⃣ Data migration / backfill jobs

### Problem

Long-running migration accidentally started twice.

### Example

* Backfilling historical data
* Schema evolution scripts
* One-time data fixes

### Why advisory lock

✔ Prevents accidental re-execution
✔ Survives restarts
✔ No extra infra

---

# 4️⃣ Leader-only logic inside a microservice

### Problem

You want **leader behavior** without Kubernetes leader election.

### Example

* One service publishes metrics
* One service syncs config
* One service polls external API

### Example lock

```text
"metrics_publisher_leader"
```

The holder acts as leader.

---

# 5️⃣ Preventing concurrent business operations

### Problem

Two requests try to modify the same logical entity.

### Example

* Payroll run
* Monthly invoice generation
* End-of-day settlement

### Example

```text
Lock: "payroll_run_2025_01"
```

✔ Ensures only one payroll run happens.

---

# 6️⃣ Coordinating multiple microservices

### Problem

Different services share the same DB and must not overlap work.

### Example

* Service A cleans data
* Service B aggregates data

Both coordinate via:

```text
Lock: "daily_data_maintenance"
```

---

# 7️⃣ Cache rebuild / warm-up

### Problem

All pods try to rebuild cache after restart.

### Example

* Load reference data
* Warm in-memory cache

### With advisory lock

Only one pod rebuilds; others wait or skip.

---

# 8️⃣ Rate-limiting critical operations (advanced)

### Problem

Limit concurrent execution of expensive DB jobs.

### Example

* Max 1 heavy report at a time
* Prevent DB overload

Advisory lock enforces global limit.

---

# 9️⃣ Alternative to Quartz (lightweight)

### Problem

Quartz is heavy; you just need **single execution**.

### Advisory lock gives:

✔ Simplicity
✔ Reliability
✔ No extra tables

Used by many teams instead of Quartz.

---

# 10️⃣ Safety switch for manual operations

### Problem

Two engineers run the same script accidentally.

### Example

```sql
SELECT pg_try_advisory_lock('manual_cleanup');
```

Only one script proceeds.

---

# When advisory locks are a BAD idea ❌

Important to know limits.

| Scenario                      | Why                        |
| ----------------------------- | -------------------------- |
| Cross-database coordination   | Locks are DB-local         |
| Non-cooperating tools         | They ignore advisory locks |
| Very fine-grained row locking | Use DB locks               |
| External systems (no DB)      | Use Redis / Zookeeper      |

---

# Advisory lock vs other distributed locks

| Lock Type                  | Complexity | Reliability |
| -------------------------- | ---------- | ----------- |
| Advisory lock              | Low        | High        |
| Redis lock                 | Medium     | Medium      |
| Zookeeper                  | High       | Very High   |
| Kubernetes leader election | Medium     | High        |

---

# Simple rule of thumb (memorize this)

> **If all participants already talk to the same database → advisory lock is the best choice.**

---

# Final takeaway

Advisory locks are:

* Simple
* Safe
* Database-enforced
* Production-proven
* Ideal for jobs & coordination

They are not magic — just **well-placed shared memory inside the DB**.

---

If you want next:

* Compare advisory lock vs Redis lock in failure scenarios
* Learn how big companies avoid Quartz
* See how Yugabyte differs from Postgres
* Explore batch purge best practices

Here’s a **clear ASCII architecture diagram** showing **how advisory locks work as a distributed mutex** for your **purge job** in a multi-pod production environment.

---

## 🏗️ Architecture: Distributed Purge Job with Advisory Lock

```
                    ┌───────────────────────────┐
                    │      Kubernetes Cluster    │
                    │                           │
                    │   ┌──────────┐            │
                    │   │  Pod A   │            │
                    │   │ (Job)    │            │
                    │   └────┬─────┘            │
                    │        │                  │
                    │   ┌────▼─────┐            │
                    │   │  Pod B   │            │
                    │   │ (Job)    │            │
                    │   └────┬─────┘            │
                    │        │                  │
                    │   ┌────▼─────┐            │
                    │   │  Pod C   │            │
                    │   │ (Job)    │            │
                    │   └────┬─────┘            │
                    │        │                  │
                    └────────┼──────────────────┘
                             │
                             │  JDBC / SQL
                             │  pg_try_advisory_lock()
                             ▼
                ┌──────────────────────────────────┐
                │     PostgreSQL / YugabyteDB       │
                │                                  │
                │   ┌──────────────────────────┐  │
                │   │  Advisory Lock Table      │  │
                │   │  (Internal, in-memory)   │  │
                │   ├──────────────────────────┤  │
                │   │  emp_details_purge  → A  │◄─┼── Pod A owns lock
                │   └──────────────────────────┘  │
                │                                  │
                │   ┌──────────────────────────┐  │
                │   │      emp_details table    │  │
                │   │                          │  │
                │   │  old rows (< 30 days) ◄──┼──┼── DELETE
                │   │  recent rows             │  │
                │   └──────────────────────────┘  │
                └──────────────────────────────────┘
```

---

## 🔄 Execution Flow (Step-by-Step)

```
TIME = 02:00 AM (cron fires)

Pod A → SELECT pg_try_advisory_lock("emp_details_purge") → TRUE
Pod B → SELECT pg_try_advisory_lock("emp_details_purge") → FALSE
Pod C → SELECT pg_try_advisory_lock("emp_details_purge") → FALSE
```

Only **Pod A proceeds**.

---

## 🧹 Purge Phase

```
Pod A
  |
  |-- DELETE FROM emp_details
  |   WHERE last_updated < now() - 30 days
  |
  |-- (DB locks only those old rows)
  |
  |-- INSERT / UPDATE by other services → still allowed
```

---

## 🔓 Completion

```
Pod A → SELECT pg_advisory_unlock("emp_details_purge")

DB → removes lock entry
Next scheduled run → repeat
```

---

## 🧠 Key Things This Diagram Shows

✔ Pods **do not talk to each other**
✔ Database is the **single coordinator**
✔ Advisory lock is **logical (not table/row lock)**
✔ Business traffic is **not blocked**
✔ Exactly **one job executor**

---

## 📌 One-line mental model

```
Multiple servers → One database → One lock → One job
```

Perfect — let’s go **step by step**. I’ll explain **how leader election works**, why it’s needed, and give **concrete examples with Kubernetes and Zookeeper** for your purge job scenario.

We’ll assume the same use case: **purge old records from `emp_details` table**.

---

# 1️⃣ Why leader election?

In a distributed system with multiple pods:

* You **cannot rely on @Scheduled alone**, because every pod would execute the job.
* **You want only one pod to act as the “leader”** and run the job.
* If the leader crashes, another pod must take over automatically.

Leader election solves this.

---

# 2️⃣ Leader election patterns

There are three common approaches:

| Approach                         | Coordinator       | Example use                      |
| -------------------------------- | ----------------- | -------------------------------- |
| **Kubernetes Lease API**         | K8s API server    | Microservices inside K8s cluster |
| **Zookeeper / etcd**             | Zookeeper cluster | Complex HA jobs, cross-cluster   |
| **Spring Cloud Leader Election** | K8s / DB          | Spring Boot native               |

We’ll focus on **Kubernetes and Spring Boot** for simplicity.

---

# 3️⃣ Using Kubernetes Lease API (built-in)

### Concept

* K8s provides a **`Lease` resource** in a namespace.
* Pods try to acquire the lease.
* Lease holder becomes **leader**.
* If leader pod dies, the lease expires, and another pod becomes leader.

### Step-by-step

1. Create a `Lease` in your namespace:

```yaml
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata:
  name: emp-details-purge-leader
  namespace: default
spec:
  holderIdentity: ""
  leaseDurationSeconds: 30
```

2. Use **client code** in each pod to try to acquire the lease.

---

### Java Example with Spring Boot + K8s

```java
@Component
public class PurgeJobLeader {

    private final KubernetesClient k8sClient;

    public PurgeJobLeader(KubernetesClient k8sClient) {
        this.k8sClient = k8sClient;
    }

    @Scheduled(cron = "0 0 2 * * *")
    public void runPurgeJob() {

        // Try to acquire lease
        boolean isLeader = tryAcquireLease();
        if (!isLeader) {
            log.info("Another pod is leader. Skipping purge.");
            return;
        }

        log.info("Leader acquired. Running purge job...");
        purgeOldEmpDetails();
    }

    private boolean tryAcquireLease() {
        // Simplified: acquire Lease via K8s API
        // If lease holderIdentity is empty or expired, set it to this pod
        // Return true if success, false otherwise
        // Libraries like fabric8 Kubernetes client handle this
        return k8sClient.tryAcquireLease("emp-details-purge-leader");
    }

    private void purgeOldEmpDetails() {
        jdbcTemplate.update(
            "DELETE FROM emp_details WHERE last_updated < now() - interval '30 days'"
        );
    }
}
```

**Notes:**

* Only the pod holding the lease runs the purge.
* Lease automatically expires if pod dies → failover.
* Lease duration can be tuned (30–60 seconds usually).

---

# 4️⃣ Using Zookeeper for Leader Election

### Concept

* Zookeeper maintains a **special node (znode)** for leader election.
* All pods try to create the node (`/leader/jobName`).
* First to succeed becomes leader.
* Others watch the node and try again if it disappears.

### Flow

1. Pod A creates `/leader/emp_details_purge` → becomes leader.
2. Pod B watches `/leader/emp_details_purge` → waits.
3. Pod A finishes → deletes znode → Pod B tries → becomes new leader if running.

---

### Java Example using Curator

```java
LeaderSelector leaderSelector = new LeaderSelector(client, "/leader/emp_details_purge", 
    new LeaderSelectorListenerAdapter() {
        @Override
        public void takeLeadership(CuratorFramework client) throws Exception {
            // This pod is now leader
            purgeOldEmpDetails();

            // Hold leadership until done
        }
    });

leaderSelector.autoRequeue(); // ensures continuous election
leaderSelector.start();
```

**Notes:**

* Very robust for distributed systems outside Kubernetes.
* Zookeeper ensures **exactly one leader**.
* Handles failover automatically.

---

# 5️⃣ Comparison: Advisory Lock vs Leader Election

| Feature        | Advisory Lock             | Leader Election                     |
| -------------- | ------------------------- | ----------------------------------- |
| Coordinator    | DB                        | K8s / Zookeeper                     |
| Failover       | Automatic (lock released) | Automatic (new leader elected)      |
| Multiple jobs  | Separate locks            | Single leader can run multiple jobs |
| Implementation | Simple SQL                | Requires K8s client or ZK client    |
| Use case       | Simple batch jobs         | Long-running services, HA tasks     |

---

# 6️⃣ Production recommendation

* **If jobs are short, isolated, DB-heavy** → advisory lock is simpler.
* **If jobs are long-running, multiple jobs, or HA needed** → leader election is better.
* You can **combine both**: leader pod acquires advisory lock to ensure no double execution even during failover.

---

Below is a **clear comparison** of **ShedLock**, **Advisory Lock**, **Optimistic Lock**, and **Pessimistic Lock**.
I’ve given **bullet summaries** first, followed by a **comparison table** for quick reference.

---

## 🔹 Bullet-Point Explanation

### **1. ShedLock**

* Used **only for scheduled tasks** in distributed systems
* Ensures **only one instance** of a scheduled job runs at a time
* Works across multiple nodes (microservices / pods)
* Uses **external storage** (DB, Redis, Mongo, etc.) to manage lock
* Common in **Spring Boot @Scheduled jobs**
* Not meant for row-level or transactional locking

**Use case:** Preventing cron jobs from running simultaneously on multiple servers

---

### **2. Advisory Lock**

* A **database-level lock** controlled explicitly by the application
* Not automatically tied to rows or transactions
* Can lock **any arbitrary resource** (e.g., user ID, job name)
* Other sessions must **cooperate** and check the lock
* Common in PostgreSQL, MySQL

**Use case:** Coordinating work across multiple processes using the same database

---

### **3. Optimistic Lock**

* Assumes **conflicts are rare**
* No lock during read
* Uses **versioning or timestamps**
* Fails on update if data was changed by another transaction
* Lightweight and scalable

**Use case:** High-read, low-write systems (e.g., REST APIs, web apps)

---

### **4. Pessimistic Lock**

* Assumes **conflicts are likely**
* Locks data immediately on read
* Prevents others from reading or writing
* Can reduce concurrency
* Implemented via `SELECT ... FOR UPDATE`

**Use case:** Financial transactions, inventory management

---

## 📊 Comparison Table

| Feature             | ShedLock                         | Advisory Lock                  | Optimistic Lock        | Pessimistic Lock          |
| ------------------- | -------------------------------- | ------------------------------ | ---------------------- | ------------------------- |
| Purpose             | Prevent duplicate scheduled jobs | Application-controlled locking | Detect write conflicts | Prevent concurrent access |
| Scope               | Distributed systems              | Database-wide                  | Row-level              | Row-level                 |
| Lock Type           | External lock (DB/Redis)         | Logical/manual lock            | Version check          | Physical DB lock          |
| Tied to Transaction | ❌ No                             | ⚠️ Optional                    | ✅ Yes                  | ✅ Yes                     |
| Blocks Other Reads  | ❌ No                             | ❌ No                           | ❌ No                   | ✅ Sometimes               |
| Blocks Other Writes | ❌ No                             | ❌ No                           | ❌ No (fails on commit) | ✅ Yes                     |
| Performance Impact  | Low                              | Low                            | Very Low               | High                      |
| Failure Handling    | Lock timeout                     | Must release manually          | Retry on failure       | Wait or deadlock          |
| Common Technologies | Spring ShedLock                  | PostgreSQL, MySQL              | JPA/Hibernate          | SQL, JPA                  |
| Best For            | Scheduled jobs                   | Cross-process coordination     | High concurrency apps  | Critical data consistency |

---

## 🧠 Quick Recommendation

* **Scheduled jobs?** → **ShedLock**
* **Cross-service coordination?** → **Advisory Lock**
* **High concurrency web apps?** → **Optimistic Lock**
* **Strict consistency (money, stock)?** → **Pessimistic Lock**

```java
// ===============================
// 1. SHEDLOCK (Spring Boot)
// ===============================

@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "PT10M")
public class ShedLockConfig {

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }
}

@Component
public class ScheduledJob {

    @Scheduled(cron = "0 */5 * * * *")
    @SchedulerLock(name = "myScheduledJob", lockAtMostFor = "PT5M", lockAtLeastFor = "PT1M")
    public void executeJob() {
        System.out.println("Running scheduled job safely with ShedLock");
    }
}
```

```java
// ===============================
// 2. ADVISORY LOCK (PostgreSQL)
// ===============================

@Repository
public class AdvisoryLockRepository {

    private final JdbcTemplate jdbcTemplate;

    public AdvisoryLockRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public boolean acquireLock(long lockKey) {
        return jdbcTemplate.queryForObject(
                "SELECT pg_try_advisory_lock(?)",
                Boolean.class,
                lockKey
        );
    }

    public void releaseLock(long lockKey) {
        jdbcTemplate.execute(
                "SELECT pg_advisory_unlock(" + lockKey + ")"
        );
    }
}
```

```java
// ===============================
// 3. OPTIMISTIC LOCK (JPA)
// ===============================

@Entity
@Table(name = "product")
public class Product {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int quantity;

    @Version
    private Long version;
}

@Service
@Transactional
public class ProductService {

    @PersistenceContext
    private EntityManager entityManager;

    public void updateQuantity(Long productId, int newQty) {
        Product product = entityManager.find(Product.class, productId);
        product.setQuantity(newQty); // version checked on commit
    }
}
```

```java
// ===============================
// 4. PESSIMISTIC LOCK (JPA)
// ===============================

@Entity
@Table(name = "account")
public class Account {

    @Id
    @GeneratedValue
    private Long id;

    private double balance;
}

@Service
@Transactional
public class AccountService {

    @PersistenceContext
    private EntityManager entityManager;

    public void withdraw(Long accountId, double amount) {
        Account account = entityManager.find(
                Account.class,
                accountId,
                LockModeType.PESSIMISTIC_WRITE
        );

        account.setBalance(account.getBalance() - amount);
    }
}
```

### 🔑 Notes
* **ShedLock** → Distributed scheduled jobs
* **Advisory Lock** → DB-level logical locking
* **Optimistic Lock** → Version-based conflict detection
* **Pessimistic Lock** → Blocking row-level lock

# Redis ShedLock Configuration
```java
// ===============================
// Redis ShedLock Configuration
// ===============================

@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "PT10M")
public class RedisShedLockConfig {

    @Bean
    public LockProvider lockProvider(RedisConnectionFactory redisConnectionFactory) {
        return new RedisLockProvider(redisConnectionFactory);
    }
}
```

```java
// ===============================
// Scheduled Job with Redis ShedLock
// ===============================

@Component
public class RedisScheduledJob {

    @Scheduled(cron = "0 */5 * * * *")
    @SchedulerLock(
        name = "redisScheduledJob",
        lockAtMostFor = "PT5M",
        lockAtLeastFor = "PT1M"
    )
    public void runJob() {
        System.out.println("Running job with Redis ShedLock");
    }
}
```

# Use Cases of Optimistic and Pessimistic Locking

* **Optimistic Locking**

  * Use when **conflicts are rare**
  * Best for **high-read, low-write** systems
  * Improves **performance and scalability**
  * Common in **web applications / REST APIs**
  * Example: User profile updates, content management

* **Pessimistic Locking**

  * Use when **conflicts are frequent**
  * Best for **critical data consistency**
  * Prevents concurrent access upfront
  * Common in **financial or inventory systems**
  * Example: Bank transactions, seat booking, stock deduction


 # **Idempotency** focused on **avoiding repeated requests**

---

## 1️⃣ Data Flow: How Idempotency Avoids Repeated Requests

**Scenario:** Client retries the same request due to network timeout.

### 🔁 Without Idempotency

1. Client sends request
2. Server processes request
3. Client times out
4. Client retries request
5. Server processes again ❌ (duplicate action)

➡️ Result: **Duplicate records / double counting**

---

### ✅ With Idempotency (Using Idempotency Key)

1. Client generates **Idempotency-Key**
2. Client sends request with key
3. Server checks key store (Redis/DB)
4. **Key not found**

   * Process request
   * Store result with key
5. Client retries same request
6. Server finds key
7. **Returns cached response**

   * No reprocessing

➡️ Result: **Exactly-once behavior**

---

## 2️⃣ High-Level Idempotency Flow Diagram (Text)

```
Client
  │
  │ POST /click
  │ Idempotency-Key: abc123
  ▼
API Server
  │
  │ Check Redis/DB
  ▼
Idempotency Store
  │
  ├─ Key not found → Process + Save
  └─ Key found → Return previous response
```

---

## 3️⃣ Steps to Implement Idempotency (Ad Click Event System)

### 🎯 Use Case

* Prevent **double counting** of ad clicks
* Handle **retries, refreshes, network failures**

---

### 🧱 Step 1: Client Generates Idempotency Key

* UUID or hash
* One key per logical action

```
Idempotency-Key = UUID.randomUUID()
```

---

### 🧱 Step 2: Client Sends Click Event

```
POST /ad/click
Headers:
  Idempotency-Key: 123e4567
Body:
  userId, adId, timestamp
```

---

## 7️⃣ When NOT to Use Idempotency

* Read-only APIs
* Streaming events (Kafka handles deduplication differently)
* Fire-and-forget logging

# Payment vs click idempotency comparison

## 🔁 High-Level Goal

| Aspect          | Payment Idempotency         | Click Idempotency               |
| --------------- | --------------------------- | ------------------------------- |
| Primary Goal    | **Prevent double charge**   | **Prevent double count**        |
| Business Impact | **Critical / irreversible** | Important but less critical     |
| Failure Cost    | Very high (money loss)      | Medium (analytics/billing skew) |

---

## 🔑 Idempotency Key Characteristics

| Aspect           | Payment                 | Click                           |
| ---------------- | ----------------------- | ------------------------------- |
| Key Lifetime     | Long-lived              | Short-lived                     |
| Key Scope        | Per **payment attempt** | Per **user + ad + time window** |
| Client Generated | Yes (required)          | Yes (recommended)               |
| Key Reuse        | Never reused            | May be reused after TTL         |
| Collision Risk   | Zero tolerance          | Low tolerance                   |

---

## 🧱 Data Flow Differences

### 💳 Payment Idempotency

1. Client initiates payment
2. Sends **Idempotency-Key**
3. Server checks persistent store
4. If exists → return exact same response
5. If not → charge gateway
6. Save **request + response + status**
7. Key retained for long duration

➡️ **Exactly-once monetary action**

---

### 🖱️ Click Idempotency

1. User clicks ad
2. Client sends click event + key
3. Server checks Redis
4. If exists → skip processing
5. If not → count click
6. Store key with TTL

➡️ **At-most-once counting**

---

## 🗄️ Storage Strategy

| Aspect       | Payment                 | Click            |
| ------------ | ----------------------- | ---------------- |
| Storage Type | Strong DB (SQL)         | Redis / Cache    |
| Durability   | Persistent              | Ephemeral        |
| TTL          | Days / Months           | Minutes / Hours  |
| Data Stored  | Full request + response | Minimal metadata |
| Cleanup      | Manual / archival       | Automatic        |

---

## ⚙️ Consistency & Guarantees

| Aspect                   | Payment                 | Click             |
| ------------------------ | ----------------------- | ----------------- |
| Consistency              | Strong                  | Eventual          |
| Retry Safety             | Must return same result | Can safely ignore |
| Partial Failure Handling | Required                | Optional          |
| Auditing                 | Mandatory               | Rare              |

---

## 🚦 Failure Scenarios

| Scenario                 | Payment                   | Click           |
| ------------------------ | ------------------------- | --------------- |
| Client timeout           | Retry returns same charge | Retry ignored   |
| Server crash mid-process | Resume or reconcile       | May lose count  |
| Duplicate request        | Always blocked            | Usually blocked |
| Downstream failure       | Rollback / compensate     | Skip event      |

---

## 🔒 Security & Compliance

| Aspect                  | Payment             | Click       |
| ----------------------- | ------------------- | ----------- |
| Compliance              | PCI-DSS, audit logs | None        |
| Fraud Risk              | High                | Low         |
| Validation              | Strict              | Relaxed     |
| Idempotency Enforcement | Mandatory           | Best-effort |

---

## 🧠 Implementation Complexity

| Aspect              | Payment | Click             |
| ------------------- | ------- | ----------------- |
| Complexity          | High    | Medium            |
| Atomicity Required  | Yes     | Yes (but simpler) |
| Latency Sensitivity | High    | Medium            |
| Horizontal Scaling  | Harder  | Easier            |

---

## 🧩 Example TTL Strategy

| Use Case        | TTL            |
| --------------- | -------------- |
| Card payment    | 7–30 days      |
| Wallet transfer | Permanent      |
| Ad click        | 5 min – 24 hrs |
| Impression      | Seconds        |

---

## 🧠 Rule of Thumb

* **Money involved?** → Strong idempotency, persistent storage
* **Analytics event?** → Cache-based idempotency
* **Retry must return exact same response?** → Payment
* **Duplicate suppression only?** → Click

---

## ✅ Summary

| Dimension  | Payment      | Click        |
| ---------- | ------------ | ------------ |
| Guarantee  | Exactly-once | At-most-once |
| Storage    | Durable DB   | Redis        |
| TTL        | Long         | Short        |
| Strictness | Very high    | Medium       |

---
