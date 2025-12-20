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

If you want, I can:

* Draw a **system design diagram**
* Convert this into **interview-ready explanation**
* Map this to **AWS / GCP services**
* Provide **sample Storm + Kafka flow**

Just tell me 👍
