Below is a **practical, real-world system design** for a **Real-Time Promotion & Pricing Platform** that matches your points:

* **Real-time pricing during promotions**
* **Cost & margin control (30%+ margin guardrails)**
* **High-throughput streaming (20,000+ msgs/sec)**
* **Storm + Kafka + REST–based architecture**
* **Retail domain focused**

---

## 1. High-Level Goals of the System

**Business goals**

* Maximize revenue during promotions
* Never violate **minimum margin (≥30%)**
* React instantly to demand, inventory, and competitor price changes

**Technical goals**

* Real-time decisioning (<100 ms latency)
* Horizontally scalable (20k+ events/sec)
* Fault-tolerant and auditable

---

## 2. High-Level Architecture (Logical View)

```
[ POS / App / Web ]
        |
        v
[ API Gateway ]
        |
        v
[ Kafka Topics ]
        |
        v
[ Apache Storm ]
  |      |      |
Pricing  Cost   Forecast
Engine  Engine  Engine
  |      |      |
        v
[ Promotion Decision Engine ]
        |
        v
[ Price Publish Service ]
        |
        v
[ Cache (Redis) ] → [ UI / POS / Cart ]
```

---

## 3. Core Components Explained

### 3.1 Event Producers (Sources)

**Events generated in real time**

* Cart add/remove
* Checkout completed
* Inventory updates
* Supplier cost updates
* Competitor price feeds
* Promotion rule changes

**Technology**

* REST APIs
* Mobile/Web SDKs
* POS systems

👉 Events are published into **Kafka topics**

---

### 3.2 Kafka (Event Backbone)

**Why Kafka?**

* Handles **20,000+ messages/sec**
* Decouples producers & consumers
* Replay capability for audits & debugging

**Typical Topics**

* `sales_events`
* `inventory_events`
* `cost_updates`
* `promo_rules`
* `price_updates`

---

### 3.3 Apache Storm (Real-Time Processing)

Storm is used for **low-latency stream processing**.

#### Storm Topology Example

```
Kafka Spout
   |
   v
Normalization Bolt
   |
   +--> Pricing Bolt
   |
   +--> Cost & Margin Bolt
   |
   +--> Forecasting Bolt
   |
   v
Promotion Decision Bolt
   |
   v
Publish Price Bolt
```

---

## 4. Real-Time Pricing Engine

**Inputs**

* Base price
* Promotion type
* Inventory level
* Demand velocity
* Competitor price

**Pricing Logic (simplified)**

```
Effective Price =
Base Price
- Promotion Discount
+ Demand Surge Adjustment
- Overstock Clearance Adjustment
```

**Real-time decisions**

* Increase price if demand spikes
* Reduce discount if inventory is low
* Stop promo if margin < 30%

---

## 5. Cost & Margin Management Engine (Critical)

This engine **protects profitability**.

### Margin Formula

```
Margin % =
(Effective Price - Cost - Logistics - Tax) / Effective Price
```

### Guardrail Rules

* ❌ Block promotion if margin < 30%
* ⚠️ Reduce discount dynamically
* ✅ Allow promo if volume compensates margin

**Example**

* Original Discount: 40%
* Margin drops to 26%
* System auto-adjusts discount to 32%

---

## 6. Forecasting Engine (Real-Time + Predictive)

**Responsibilities**

* Predict sales uplift during promotion
* Detect demand anomalies
* Prevent stockouts

**Models Used**

* Short-term time series (last 15–60 mins)
* Historical promo uplift models
* Seasonality & event-based multipliers

**Output**

* Expected sales/hour
* Inventory depletion ETA
* Revenue vs margin trade-off

---

## 7. Promotion Decision Engine

This is the **brain** of the system.

**Decision Inputs**

* Pricing engine output
* Cost & margin checks
* Forecasted demand
* Inventory availability

**Actions**

* Approve / reject promotion
* Modify discount in real time
* End promotion early
* Switch promotion type dynamically

---

## 8. Price Publishing & Caching

**Why caching?**

* Ultra-fast read for cart & checkout
* Prevent DB overload

**Tech**

* Redis / Memcached

**TTL**

* Very short (seconds) during promotions

---

## 9. Monitoring & Control

**Metrics**

* Margin % per SKU
* Revenue per minute
* Promo success rate
* Kafka lag
* Storm bolt latency

**Alerts**

* Margin breach
* Inventory exhaustion
* Abnormal traffic spikes

---

## 10. Promotion Deals in Retail Domain

### 10.1 Price-Based Promotions

* Flat % off (10%, 20%)
* Flat amount off ($5 off)
* Dynamic discount (real-time)

---

### 10.2 Volume-Based Deals

* Buy 1 Get 1 Free (BOGO)
* Buy 2 Get 30% off
* Buy more save more

---

### 10.3 Time-Based Promotions

* Flash sales
* Happy hour pricing
* Limited-time offers

---

### 10.4 Basket-Level Promotions

* ₹500 off on cart value > ₹3000
* Free shipping above threshold
* Combo pricing

---

### 10.5 Loyalty & Personalized Promotions

* Member-only pricing
* Personalized discounts
* Cashback offers

---

### 10.6 Inventory-Driven Promotions

* Overstock clearance
* Near-expiry discounts
* Seasonal clearance

---

### 10.7 Channel-Specific Promotions

* App-only deals
* Store-only pricing
* Online exclusive offers

---

## 11. Why This Design Works Well

✅ Real-time & scalable
✅ Protects 30%+ margin automatically
✅ Handles flash sales & traffic spikes
✅ Flexible for multiple promotion types
✅ Enterprise-ready retail architecture

---

Below is a **deep, step-by-step flow explanation** of **how vendor feeds enter the system, how pricing & promotions are decided in real time, and how the final promotion reaches the customer at purchase time**—mapped exactly to your architecture.

I’ll explain this **from data ingestion → decisioning → checkout**, using **retail-real examples**.

---

# 1️⃣ Vendor & Source Feeds (Entry Point)

### Sources That Send Data

Different systems continuously push data:

| Source              | Example Events                  |
| ------------------- | ------------------------------- |
| Vendors / Suppliers | Cost price updates, MOQ changes |
| POS systems         | Item sold, quantity, store id   |
| Web / Mobile App    | Cart add/remove, search views   |
| Inventory system    | Stock in/out                    |
| Competitor feeds    | External price comparison       |
| Promo admin UI      | Rule creation or changes        |

---

## 🔹 Example Vendor Feed (Cost Update)

Vendor sends:

```json
{
  "vendorId": "V123",
  "sku": "SKU789",
  "newCost": 420,
  "effectiveFrom": "2025-12-20T10:00:00Z"
}
```

---

# 2️⃣ API Gateway (Ingestion Layer)

### Responsibilities

* Authentication & authorization
* Rate limiting (vendors can flood data)
* Schema validation
* Routing to Kafka

### What Happens

1. Vendor feed hits `/vendor/cost-update`
2. Gateway validates token
3. Adds metadata:

```json
{
  "source": "vendor",
  "receivedAt": "2025-12-20T10:00:05Z"
}
```

4. Publishes event to Kafka

---

# 3️⃣ Kafka Topics (Event Backbone)

Each event type goes to a **dedicated topic**.

| Topic                 | Purpose               |
| --------------------- | --------------------- |
| `vendor_cost_updates` | Supplier pricing      |
| `sales_events`        | POS / checkout        |
| `inventory_events`    | Stock changes         |
| `promo_rules`         | Promotion definitions |
| `competitor_prices`   | External pricing      |

Kafka guarantees:

* High throughput (20k+/sec)
* Ordered processing per SKU
* Replayability

---

## 🔹 Kafka Message Example

```json
{
  "sku": "SKU789",
  "cost": 420,
  "vendorId": "V123"
}
```

---

# 4️⃣ Apache Storm (Real-Time Processing Core)

Storm consumes Kafka streams and processes **in milliseconds**.

---

## 4.1 Normalization Bolt (First Step)

### Purpose

* Convert all incoming feeds into a **common internal format**
* Enrich with cached data (category, brand, tax)

### Output Example

```json
{
  "sku": "SKU789",
  "cost": 420,
  "category": "Electronics",
  "tax": 18
}
```

---

## 4.2 Pricing Engine Bolt

### Inputs

* Base price
* Current promotions
* Competitor prices
* Demand velocity

### Logic

* Decide **base effective price**
* Apply promo discount if eligible

```text
Base Price = 799
Promo Discount = 20%
Effective Price = 639
```

---

## 4.3 Cost & Margin Engine Bolt (Profit Guard)

### Inputs

* Vendor cost
* Logistics
* Tax
* Effective price

### Margin Calculation

```
Margin % = (Price - Cost - Logistics - Tax) / Price
```

### Decision

* If margin ≥ 30% → continue
* If margin < 30% → adjust or block

🔹 Example:

```
Margin = 26% ❌
Discount reduced from 20% → 12%
```

---

## 4.4 Forecasting Engine Bolt

### Purpose

Predict **what will happen if promo continues**.

### Uses

* Recent sales velocity
* Past promo uplift
* Time of day

### Output

```json
{
  "expectedSalesPerHour": 320,
  "stockOutInMinutes": 45
}
```

---

# 5️⃣ Promotion Decision Engine (Central Brain)

### Inputs from Storm

* Adjusted price
* Margin status
* Inventory forecast
* Promotion rules

### Decisions Made

* Approve promo
* Modify discount
* End promo early
* Switch promo type

### Output Decision

```json
{
  "sku": "SKU789",
  "finalPrice": 679,
  "promoType": "FLASH_SALE",
  "validFor": "15 minutes"
}
```

---

# 6️⃣ Price Publish Service

### Role

* Convert decisions into **customer-consumable prices**
* Push to cache
* Notify downstream systems

### Actions

* Publish to `price_updates` Kafka topic
* Update Redis cache
* Send webhook to POS & UI

---

# 7️⃣ Cache (Redis) – Ultra-Fast Access

### Why Redis?

* Cart & checkout require **<10ms reads**
* Prices change frequently

### Cache Key Example

```
price:SKU789:STORE12
```

### Cached Value

```json
{
  "price": 679,
  "promoLabel": "Flash Sale",
  "expiresAt": "10:15"
}
```

---

# 8️⃣ Customer Purchase Flow (End-to-End)

### Step-by-Step

1. Customer opens app / POS scans item
2. UI calls:

```
GET /price?sku=SKU789
```

3. Backend reads from Redis
4. Displays:

```
₹679 – Flash Sale (15 mins left)
```

---

## 🛒 Cart & Checkout Validation

At checkout:

* System **re-validates price**
* Ensures promo still active
* Locks price for order

### Order Event Sent Back

```json
{
  "orderId": "ORD123",
  "sku": "SKU789",
  "price": 679
}
```

➡ Sent to Kafka → Storm updates demand & inventory

---

# 9️⃣ Feedback Loop (Self-Correcting System)

Every purchase feeds back into:

* Demand forecasting
* Inventory consumption
* Promo effectiveness

This allows:

* Dynamic discount tuning
* Early promo shutdown
* Margin protection

---

# 🔄 End-to-End Flow Summary

```
Vendor Feed → API Gateway → Kafka
→ Storm (Pricing + Cost + Forecast)
→ Promotion Decision Engine
→ Price Publish Service
→ Redis Cache
→ UI / POS
→ Customer Purchase
→ Kafka (Feedback Loop)
```

---

## ✅ Why This Works in Real Retail

* Handles **vendor cost changes instantly**
* Adjusts promotions mid-sale
* Guarantees **30%+ margin**
* Supports flash sales & peak traffic
* Real-time personalization ready

---

Perfect—below is a **very concrete, numbers-driven walkthrough**, followed by **failure scenarios with recovery**, and finally a **clean interview-ready explanation** you can use verbatim.

I’ll keep this **practical, realistic, and retail-focused**.

---

# 1️⃣ Concrete End-to-End Example (With Real Numbers)

### Product Details

| Attribute     | Value     |
| ------------- | --------- |
| SKU           | TV-55-4K  |
| Base Price    | ₹50,000   |
| Vendor Cost   | ₹32,000   |
| Logistics     | ₹2,000    |
| Tax (18%)     | ₹9,000    |
| Target Margin | ≥30%      |
| Inventory     | 400 units |

---

## Step 1: Vendor Sends Cost Update

Vendor increases cost from ₹30,000 → ₹32,000.

**Event**

```json
{
  "sku": "TV-55-4K",
  "cost": 32000
}
```

➡ Kafka topic: `vendor_cost_updates`

---

## Step 2: Promotion Rule

Marketing creates:

> **Flash Sale – 25% OFF for 1 hour**

➡ Kafka topic: `promo_rules`

---

## Step 3: Pricing Engine Calculation

### Initial Discounted Price

```
₹50,000 - 25% = ₹37,500
```

---

## Step 4: Cost & Margin Engine

### Margin Calculation

```
Net Revenue = 37,500 - 9,000 (tax) = 28,500
Total Cost = 32,000 + 2,000 = 34,000

Margin = (28,500 - 34,000) / 37,500 = -14%
```

❌ **Loss detected**

---

## Step 5: Automatic Discount Adjustment

System iteratively reduces discount:

| Discount | Price      | Margin    |
| -------- | ---------- | --------- |
| 25%      | 37,500     | ❌ -14%    |
| 15%      | 42,500     | ❌ 6%      |
| 10%      | 45,000     | ❌ 18%     |
| **5%**   | **47,500** | ✅ **32%** |

✔ Promotion auto-adjusted to **5% OFF**

---

## Step 6: Forecasting Engine

Using recent sales velocity:

* Normal: 3 TVs/hour
* Flash Sale uplift: 6×
* Expected: 18 TVs/hour

Inventory impact:

```
400 / 18 ≈ 22 hours
```

✔ Safe → no stock-out

---

## Step 7: Promotion Decision Output

```json
{
  "sku": "TV-55-4K",
  "finalPrice": 47500,
  "promoLabel": "Flash Sale",
  "validTill": "11:00 AM"
}
```

➡ Redis updated in **<10 ms**

---

## Step 8: Customer Checkout

Customer sees:

> **₹47,500 – Flash Sale (Ends in 45 mins)**

At checkout:

* Price revalidated
* Inventory decremented
* Order event sent to Kafka

---

# 2️⃣ Failure Scenarios & Recovery (Real-World)

---

## Scenario 1: Kafka Broker Failure

### Issue

* One Kafka broker goes down during flash sale

### Impact

* Event ingestion pauses briefly

### Recovery

* Kafka replicas take over
* Producers retry automatically
* No data loss (acks=all)

✔ System continues

---

## Scenario 2: Storm Bolt Failure

### Issue

* Pricing Bolt crashes

### Recovery

* Storm restarts bolt automatically
* Kafka offsets replay last messages
* Idempotent processing prevents duplicates

✔ Pricing resumes in seconds

---

## Scenario 3: Redis Cache Failure

### Issue

* Redis node crashes

### Recovery

* Read-through cache:

  * Fetch from pricing DB
* Redis cluster auto-fails over
* Short TTL prevents stale prices

✔ Checkout still works

---

## Scenario 4: Vendor Sends Wrong Cost (Too Low)

### Issue

* Vendor mistakenly sends cost = ₹20,000

### Protection

* Cost anomaly detection:

  * Compare historical variance
* Flag for manual review
* Temporarily cap discount

✔ Margin protected

---

## Scenario 5: Sudden Demand Spike

### Issue

* Influencer mention → sales jump 10×

### System Reaction

* Forecasting detects anomaly
* Promo Decision Engine:

  * Reduces discount
  * Limits per-customer quantity
  * Ends promo early

✔ Stock preserved

---

## Scenario 6: API Gateway Down

### Issue

* External feeds blocked

### Recovery

* Internal systems continue via Kafka
* Cached prices still valid
* Gateway restarts via auto-scaling

✔ Zero customer impact

---

# 3️⃣ Interview-Ready Explanation (Clean & Structured)

### ❓ “Explain your real-time promotion system”

### ✅ Answer:

> “We built a real-time promotion and pricing platform designed to handle high-traffic retail events while maintaining a minimum 30% margin.”

> “All sales, vendor cost, inventory, and promotion events are ingested via an API Gateway and streamed into Kafka, which acts as our event backbone and supports 20,000+ messages per second.”

> “Apache Storm processes these events in real time. We have dedicated bolts for pricing, cost and margin validation, and demand forecasting.”

> “The pricing engine calculates effective prices based on promotions and demand, while the cost engine enforces strict margin guardrails. If a promotion risks violating margin thresholds, discounts are automatically adjusted or blocked.”

> “A centralized promotion decision engine then finalizes the offer, which is published to Redis for sub-10-millisecond access by POS and e-commerce systems.”

> “Every purchase feeds back into Kafka, allowing the system to continuously self-correct during flash sales or demand spikes.”

> “The architecture is horizontally scalable, fault-tolerant, and has been successfully used during high-volume promotions without revenue leakage.”

---

## ⭐ Key Interview Keywords (Use These)

* Event-driven architecture
* Real-time stream processing
* Margin guardrails
* Dynamic discounting
* Feedback loop
* Fault-tolerant design
* High-throughput ingestion

---
