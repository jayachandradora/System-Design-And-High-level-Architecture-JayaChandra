Designing a hotel booking system for Agoda involves considering both functional and non-functional requirements, as well as various design aspects to ensure scalability, performance, and user experience. Here’s a comprehensive overview of how such a system can be structured:

### Functional Requirements:

1. **User Roles and Authentication**:
   - **Guest Users**: Browse hotels and view basic information.
   - **Registered Users**: Book hotels, manage bookings, and access personalized features.
   - **Admin Users**: Manage hotel listings, user accounts, and system configurations.

2. **Hotel Search and Listings**:
   - **Search Filters**: Allow users to filter hotels based on location, price range, amenities, ratings, etc.
   - **Detailed Listings**: Display comprehensive information about each hotel, including photos, room types, availability, and reviews.

3. **Booking Process**:
   - **Availability Check**: Real-time availability of rooms based on user-selected dates.
   - **Booking Management**: Allow users to select rooms, make reservations, and receive confirmation.
   - **Payment Integration**: Secure payment gateway integration for handling transactions.

4. **User Reviews and Ratings**:
   - **Review System**: Enable users to leave reviews and ratings for hotels.
   - **Moderation**: Admin tools to moderate reviews and ensure authenticity.

5. **Notifications and Alerts**:
   - **Booking Confirmations**: Send email/SMS notifications upon booking confirmation.
   - **Updates**: Notify users about special offers, booking changes, or cancellations.

6. **Customer Support**:
   - **Help Desk**: Provide a support system for handling inquiries, complaints, and assistance requests.

### Non-Functional Requirements:

1. **Performance**:
   - **Response Time**: Ensure fast response times for search queries and booking transactions.
   - **Scalability**: Handle peak loads during high-traffic periods (e.g., holiday seasons) without performance degradation.

2. **Reliability**:
   - **Availability**: Maintain high availability (e.g., 99.99%) to prevent service disruptions.
   - **Fault Tolerance**: Implement redundancy and failover mechanisms to ensure system reliability.

3. **Security**:
   - **Data Protection**: Secure user data and payment information with encryption and compliance with data protection regulations (e.g., GDPR).
   - **Authentication**: Use secure authentication methods to protect user accounts and prevent unauthorized access.

4. **Usability**:
   - **Intuitive Interface**: Design a user-friendly interface for seamless navigation and booking experience.
   - **Accessibility**: Ensure accessibility compliance for users with disabilities.

5. **Integration**:
   - **Third-Party Integration**: Integrate with external systems such as payment gateways, hotel APIs for real-time availability, and mapping services for location details.

6. **Maintenance and Support**:
   - **Monitoring**: Implement monitoring tools for performance metrics, error logs, and system health checks.
   - **Regular Updates**: Plan for regular updates and maintenance to improve functionality and security.

Designing a hotel booking system involves creating APIs for managing bookings, availability, and user interactions, as well as designing a database schema to store relevant information efficiently. Here's a structured approach to designing both the API and database for a hotel booking system:

### API Design

#### 1. Hotel Search API

- **Endpoint**: `/api/hotels/search`
- **HTTP Method**: GET
- **Parameters**:
  - `checkin_date`: Date of check-in.
  - `checkout_date`: Date of check-out.
  - `location`: Location or city.
  - Optionally: `guests`, `rooms`, `price_range`, etc.

- **Response**:
  ```json
  {
    "hotels": [
      {
        "hotel_id": "123",
        "name": "Hotel ABC",
        "location": "City A",
        "description": "Description of the hotel.",
        "rating": 4.5,
        "reviews": 100,
        "price_per_night": 150,
        "availability": {
          "available_rooms": 5,
          "total_rooms": 10
        }
      },
      // More hotels
    ]
  }
  ```

#### 2. Hotel Booking API

- **Endpoint**: `/api/bookings/create`
- **HTTP Method**: POST
- **Request Payload**:
  ```json
  {
    "hotel_id": "123",
    "user_id": "456",
    "checkin_date": "2024-07-01",
    "checkout_date": "2024-07-05",
    "guests": 2,
    "rooms": 1,
    "total_price": 750
  }
  ```
- **Response**: HTTP status code (e.g., 200 OK, 400 Bad Request).

#### 3. User Management API

- **Endpoint**: `/api/users/:user_id`
- **HTTP Methods**: GET, PUT, DELETE
- **Response**:
  ```json
  {
    "user_id": "456",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@example.com",
    "phone": "+1234567890",
    // Additional user information
  }
  ```

### Database Design

#### 1. Hotels Table

- **Table Name**: `hotels`
- **Columns**:
  - `hotel_id`: Unique identifier (primary key).
  - `name`: Hotel name.
  - `location`: Hotel location.
  - `description`: Description of the hotel.
  - `rating`: Average rating given by users.
  - `reviews`: Number of reviews received.
  - `price_per_night`: Base price per night.
  - `total_rooms`: Total number of rooms.
  - `created_at`, `updated_at`: Timestamps for record creation and updates.

#### 2. Bookings Table

- **Table Name**: `bookings`
- **Columns**:
  - `booking_id`: Unique identifier (primary key).
  - `hotel_id`: Foreign key reference to `hotels` table.
  - `user_id`: Foreign key reference to `users` table.
  - `checkin_date`, `checkout_date`: Dates of stay.
  - `guests`: Number of guests.
  - `rooms`: Number of rooms booked.
  - `total_price`: Total price for the booking.
  - `booking_status`: Status of the booking (confirmed, cancelled, etc.).
  - `created_at`, `updated_at`: Timestamps for booking creation and updates.

#### 3. Users Table

- **Table Name**: `users`
- **Columns**:
  - `user_id`: Unique identifier (primary key).
  - `first_name`, `last_name`: User's name.
  - `email`: User's email address.
  - `phone`: User's phone number.
  - `created_at`, `updated_at`: Timestamps for user creation and updates.

### Considerations

- **Data Relationships**: Maintain foreign key relationships between tables for data integrity.
- **Indexing**: Index columns used frequently in queries (e.g., `hotel_id`, `user_id`, `checkin_date`) to optimize performance.
- **Concurrency**: Implement locking mechanisms or optimistic concurrency control to manage concurrent bookings and updates.
- **Security**: Secure APIs with authentication (e.g., OAuth, JWT) and enforce authorization rules.
- **Scalability**: Design for horizontal scaling to handle increasing numbers of users, hotels, and bookings.

By following this structured approach, you can create a robust and scalable hotel booking system that efficiently handles hotel searches, bookings, and user management, ensuring a seamless experience for both customers and administrators. Adjust the schema and APIs based on specific business requirements and scalability considerations.

## Estimating the capacity

Estimating the capacity for Agoda's system design involves assessing various factors to ensure the platform can handle expected traffic, data volumes, and user interactions efficiently. Here’s an outline of capacity estimation considerations:

### Traffic Volume:

1. **Expected User Base**:
   - Determine the number of active users visiting Agoda's platform daily, weekly, and during peak periods (e.g., holiday seasons).
   - Estimate concurrent users accessing the system simultaneously.

2. **Page Views and Requests**:
   - Calculate the number of page views per user session.
   - Estimate the number of API requests and database queries per second.

### Data Storage:

1. **Transactional Data**:
   - Estimate the size and growth rate of transactional data (e.g., user profiles, bookings, payments).
   - Determine daily, weekly, and monthly data ingestion rates.

2. **Content Storage**:
   - Assess storage requirements for multimedia content (e.g., hotel images, reviews).
   - Plan for scalability to accommodate increasing content volume.

### Processing and Compute Requirements:

1. **Compute Resources**:
   - Estimate CPU and memory requirements based on anticipated workload.
   - Consider distributed computing for handling concurrent requests and data processing.

2. **Database Throughput**:
   - Calculate read and write throughput for databases (e.g., MySQL, NoSQL) based on transaction rates and query complexity.
   - Implement caching strategies (e.g., Redis, Memcached) for frequently accessed data.

### Networking and Infrastructure:

1. **Bandwidth Requirements**:
   - Determine bandwidth needs for data transfer between servers, clients, and external services (e.g., payment gateways, hotel APIs).

2. **Network Latency**:
   - Assess network latency requirements to ensure responsive user experience, especially for global users.

### Scalability and Elasticity:

1. **Scaling Strategy**:
   - Plan for horizontal scaling using cloud services (e.g., AWS, Azure) to accommodate traffic spikes and future growth.
   - Implement auto-scaling policies based on predefined metrics (e.g., CPU utilization, request rate).

2. **High Availability**:
   - Design for fault tolerance and redundancy to minimize downtime and ensure continuous service availability.
   - Use load balancers and failover mechanisms to distribute traffic and mitigate single points of failure.

### Monitoring and Optimization:

1. **Monitoring Tools**:
   - Implement monitoring solutions (e.g., Prometheus, Nagios) for real-time performance metrics, resource utilization, and system health.

2. **Optimization Strategies**:
   - Continuously optimize code, database queries, and infrastructure configurations to improve efficiency and reduce costs.
   - Conduct load testing and performance tuning exercises to validate capacity estimates and identify bottlenecks.

### Conclusion:

Capacity estimation for Agoda's system design involves a detailed assessment of traffic patterns, data storage requirements, compute resources, networking needs, scalability strategies, and monitoring practices. By accurately estimating these factors, Agoda can build and maintain a robust, scalable platform that delivers a seamless and responsive user experience while handling varying levels of demand and growth.

### Design Aspects: https://medium.com/nerd-for-tech/system-design-architecture-for-hotel-booking-apps-like-airbnb-oyo-6efb4f4dddd7

![image](https://github.com/jayachandradora/System-Design-And-High-level-Architecture-JayaChandra/assets/115500959/c67bd4ee-0adc-4082-a3c5-6f7e77449e9f)


1. **System Architecture**:
   - **Microservices**: Design modular components for scalability and independent deployment.
   - **Service-Oriented Architecture (SOA)**: Use services like booking service, user management, payment service, etc., communicating through APIs.

2. **Database Design**:
   - **Relational Database**: Store structured data like user profiles, bookings, and transactions.
   - **NoSQL Database**: Handle unstructured data like reviews and user interactions.

3. **Caching Strategy**:
   - **In-Memory Caching**: Cache frequently accessed data (e.g., hotel listings, user sessions) for faster retrieval and improved performance.

4. **Load Balancing and Distribution**:
   - **Content Delivery Network (CDN)**: Distribute static content (e.g., images, CSS) to reduce server load and improve latency.

5. **Data Analytics and Reporting**:
   - **Analytics Dashboard**: Provide insights into user behavior, booking trends, and revenue generation.
   - **Reporting Tools**: Generate reports for business analysis and decision-making.

6. **Compliance and Regulations**:
   - **Legal Compliance**: Adhere to local and international regulations governing online booking systems, data protection, and consumer rights.

### Handling distributed transactions

By addressing these functional and non-functional requirements and considering key design aspects, Agoda can develop a robust and efficient hotel booking system that meets user expectations while ensuring reliability, security, and scalability.

Handling distributed transactions in distributed systems is crucial for ensuring data consistency and reliability across multiple independent services or databases. Here are some common patterns and strategies for managing distributed transactions:

### 1. Two-Phase Commit (2PC):

- **Description**: Two-Phase Commit is a classic distributed transaction protocol that ensures all participating services agree on committing or rolling back a transaction.
  
- **Workflow**:
  1. **Prepare Phase**: Coordinator (typically the transaction manager) sends a prepare request to all participants, asking if they can commit the transaction.
  2. **Commit Phase**: If all participants confirm readiness (acknowledge prepare), the coordinator sends a commit request to all. If any participant fails or votes to abort during the prepare phase, the coordinator sends a rollback request to all participants.

- **Pros**: Provides strong consistency guarantees.
- **Cons**: Suffers from blocking issues, increased latency, and is prone to single-point-of-failure with the coordinator.

### 2. Saga Pattern:

- **Description**: Saga is a distributed transaction pattern where a large transaction is split into multiple smaller, local transactions that can be individually committed or rolled back.
  
- **Workflow**:
  - Each service involved in the saga maintains its own transaction state and compensating actions to rollback changes if needed.
  - Services communicate asynchronously and can continue with local transactions even if one or more previous steps fail.

- **Pros**: Provides better scalability, fault tolerance, and can handle long-running transactions.
- **Cons**: Requires careful design of compensating transactions (compensation logic) for rollback scenarios.

### 3. Compensating Transaction Pattern:

- **Description**: Also known as Compensation-Based Transaction, this pattern focuses on explicitly defining compensating actions to undo changes made by a transaction if it fails.
  
- **Workflow**:
  - Each transaction includes a compensation phase that is executed if the transaction fails or is rolled back.
  - Compensating actions are idempotent and designed to revert the effects of the original transaction.

- **Pros**: Simplifies rollback handling and is resilient to failures.
- **Cons**: Requires careful design of compensating actions and may not provide the same level of strong consistency as two-phase commit.

### 4. Event Sourcing and CQRS (Command Query Responsibility Segregation):

- **Description**: Event Sourcing involves capturing all changes to application state as a sequence of events, while CQRS separates read and write operations.
  
- **Workflow**:
  - Events are stored in an event log and serve as the source of truth.
  - Commands modify state based on events, and queries derive state from events.

- **Pros**: Supports scalable and resilient architectures, enabling eventual consistency and handling of complex transactional workflows.
- **Cons**: Introduces complexity, especially around event storage, event processing, and consistency models.

### 5. Idempotent Operations and Idempotent APIs:

- **Description**: Designing operations and APIs to be idempotent ensures that repeating the same operation multiple times has the same effect as performing it once.
  
- **Workflow**:
  - Clients generate and include unique identifiers (idempotency keys) with requests.
  - Servers check these keys to prevent processing duplicate requests.

- **Pros**: Simple to implement and improves fault tolerance.
- **Cons**: Limited to scenarios where idempotency can be guaranteed, and may not provide full ACID transactional guarantees.

### Considerations for Distributed Transactions:

- **Latency and Performance**: Distributed transactions often incur higher latency due to network communication and coordination.
- **Consistency Models**: Choose the appropriate consistency model (strong, eventual, etc.) based on application requirements.
- **Failure Handling**: Plan for failure scenarios and implement retry and fallback mechanisms.
- **Monitoring and Logging**: Implement robust monitoring and logging to track transactional states and diagnose issues.

Each pattern and strategy has its strengths and trade-offs, so the choice depends on factors such as transactional complexity, scalability requirements, fault tolerance, and desired consistency guarantees in your distributed system architecture.

# DB Design based on Apps level

Below is a **complete, practical DB design** mapped to this architecture, broken down **per app/service**, with **table schemas, keys, and reasoning**.
This is **interview-ready** and also realistic for production systems.

I’ll organize it by **logical plane + storage technology**.

---

# 1. Hotel Manager App (Control Plane)

### Primary DB: **Hotel DB (MySQL)**

Purpose: **authoritative source of truth for hotel configuration**

### Core Tables

## `hotels`

```sql
hotels (
  hotel_id        BIGINT PK,
  name            VARCHAR(255),
  description     TEXT,
  address         TEXT,
  city            VARCHAR(100),
  country         VARCHAR(100),
  latitude        DECIMAL(9,6),
  longitude       DECIMAL(9,6),
  status          ENUM('ACTIVE','INACTIVE'),
  created_at      TIMESTAMP,
  updated_at      TIMESTAMP
)
```

---

## `rooms`

```sql
rooms (
  room_id         BIGINT PK,
  hotel_id        BIGINT FK,
  room_type       VARCHAR(100),
  capacity        INT,
  base_price      DECIMAL(10,2),
  total_rooms     INT,
  created_at      TIMESTAMP,
  updated_at      TIMESTAMP
)
```

---

## `room_inventory`

*(Date-based inventory control)*

```sql
room_inventory (
  inventory_id    BIGINT PK,
  room_id         BIGINT FK,
  date            DATE,
  available_rooms INT,
  price           DECIMAL(10,2),
  created_at      TIMESTAMP,
  updated_at      TIMESTAMP,
  UNIQUE(room_id, date)
)
```

---

## `hotel_amenities`

```sql
hotel_amenities (
  hotel_id        BIGINT FK,
  amenity         VARCHAR(100),
  PRIMARY KEY (hotel_id, amenity)
)
```

---

## `hotel_images`

```sql
hotel_images (
  image_id        BIGINT PK,
  hotel_id        BIGINT FK,
  image_url       VARCHAR(500),
  is_primary      BOOLEAN
)
```

---

### Events Published to Kafka

* `HotelCreated`
* `HotelUpdated`
* `RoomInventoryUpdated`
* `PriceUpdated`

These events **feed Search and downstream systems**.

---

# 2. Search App (Read-Optimized)

### Primary Store: **Elasticsearch**

Purpose: **fast, denormalized search**

### Index: `hotels_search_index`

```json
{
  "hotel_id": 123,
  "name": "Ocean View Resort",
  "city": "Goa",
  "location": { "lat": 15.2993, "lon": 74.1240 },
  "amenities": ["POOL", "WIFI", "SPA"],
  "min_price": 120.00,
  "rating": 4.5,
  "available_rooms": 12,
  "room_types": [
    {
      "room_type": "DELUXE",
      "capacity": 2,
      "price": 150.00
    }
  ]
}
```

---

### How Search DB is Built

* Kafka consumer listens to:

  * Hotel DB updates
  * Inventory updates
  * Booking events
* Data is **denormalized**
* Elasticsearch is **NOT source of truth**

---

# 3. Booking App (Transactional Core)

### Primary DB: **Hotel Booking DB (MySQL)**

Purpose: **ACID transactions**

---

## `bookings`

```sql
bookings (
  booking_id      BIGINT PK,
  user_id         BIGINT,
  hotel_id        BIGINT,
  status          ENUM('PENDING','CONFIRMED','CANCELLED','FAILED'),
  check_in_date   DATE,
  check_out_date  DATE,
  total_price     DECIMAL(10,2),
  created_at      TIMESTAMP,
  updated_at      TIMESTAMP
)
```

---

## `booking_rooms`

```sql
booking_rooms (
  booking_room_id BIGINT PK,
  booking_id      BIGINT FK,
  room_id         BIGINT FK,
  price_per_night DECIMAL(10,2)
)
```

---

## `payments`

```sql
payments (
  payment_id      BIGINT PK,
  booking_id      BIGINT FK,
  payment_status  ENUM('INITIATED','SUCCESS','FAILED'),
  payment_method  VARCHAR(50),
  transaction_id  VARCHAR(255),
  amount          DECIMAL(10,2),
  created_at      TIMESTAMP
)
```

---

## `booking_audit_log`

```sql
booking_audit_log (
  audit_id        BIGINT PK,
  booking_id      BIGINT,
  event_type      VARCHAR(100),
  created_at      TIMESTAMP
)
```

---

### Redis (Booking App)

Purpose:

* Inventory locking
* Idempotency
* Fast reads

#### Keys

```text
lock:room:{room_id}:{date}
booking_cache:{booking_id}
```

---

# 4. View Booking App (Read-Heavy)

### Primary Store: **Cassandra**

Purpose:

* High-volume reads
* Historical data
* No joins

---

## `booking_by_user`

```sql
CREATE TABLE booking_by_user (
  user_id       BIGINT,
  booking_date  DATE,
  booking_id    BIGINT,
  hotel_name    TEXT,
  status        TEXT,
  total_price   DECIMAL,
  PRIMARY KEY (user_id, booking_date, booking_id)
) WITH CLUSTERING ORDER BY (booking_date DESC);
```

---

## `booking_by_hotel`

```sql
CREATE TABLE booking_by_hotel (
  hotel_id      BIGINT,
  booking_date  DATE,
  booking_id    BIGINT,
  user_id       BIGINT,
  status        TEXT,
  PRIMARY KEY (hotel_id, booking_date, booking_id)
);
```

---

### Redis (View Booking)

```text
recent_bookings:user:{user_id}
```

---

# 5. Analytics / Archival

### Hadoop / Data Lake

Schema (Parquet / ORC):

```text
booking_id
hotel_id
user_id
booking_date
revenue
city
payment_method
```

Used for:

* Revenue reports
* Forecasting
* ML models

---

# 6. DB Design Summary Table

| App / Service | DB            | Purpose                  |
| ------------- | ------------- | ------------------------ |
| Hotel Manager | MySQL         | Source of truth (config) |
| Search        | Elasticsearch | Fast search              |
| Booking       | MySQL         | ACID transactions        |
| Booking       | Redis         | Locks & cache            |
| View Booking  | Cassandra     | Read-heavy history       |
| Analytics     | Hadoop        | Long-term analytics      |

---

# 7. Key Design Principles Used

* **CQRS**
* **Polyglot persistence**
* **Event-driven sync**
* **Strong consistency only where needed**
* **Denormalization for reads**

---

# Redis Locks in Booking

Let’s deep-dive into **how Redis is used for booking locks** in your system, step by step, including **use cases, Java implementation, and best practices**. I’ll make it practical and “production-ready” like your DB design.

---

# **1. Purpose of Redis Locks in Booking**

In a hotel booking system, **inventory consistency is critical**. For example:

* Two users try to book the **last available room** at the same time.
* Without a lock, **both could succeed**, causing **overbooking**.
* Redis is used as a **distributed lock** system to prevent this.

Redis is perfect because:

* Extremely fast (`O(1)` get/set operations)
* Supports **atomic operations** (`SETNX`, `GETSET`)
* Can work **across multiple app instances** (distributed environment)

---

# **2. Use Cases for Redis Locks**

### a) **Inventory Locking**

Before a booking is confirmed:

1. Lock the room for the requested date(s)
2. Reduce available inventory
3. Process payment
4. Release lock after success or rollback

### b) **Idempotency**

If a booking request retries (network issues, client retry):

* Use Redis key per booking attempt
* Ensure **duplicate bookings are not created**

### c) **Timeouts**

* Prevent deadlocks
* Automatically release locks if user abandons booking

---

# **3. Redis Lock Implementation Concepts**

### **Key Structure**

```text
lock:room:{room_id}:{date}
```

* `{room_id}` – the room being booked
* `{date}` – the date of the stay
* Value – usually **UUID or timestamp**, used to **ensure only the owner releases**

---

### **Redis Commands Used**

1. **SET key value NX PX 5000**

   * `NX` → only set if key does **not exist**
   * `PX` → expiration in milliseconds (auto-release)

   ```bash
   SET lock:room:101:2026-01-01 "uuid-1234" NX PX 5000
   ```
2. **DEL key** → release lock after successful booking
3. **Lua script** → safe unlock (ensure only the owner can release)

---

# **4. Step-by-Step Flow in Java**

Here’s a practical flow for **booking a room with Redis lock**.

### **Step 1: Acquire Lock**

```java
String lockKey = "lock:room:" + roomId + ":" + date;
String lockValue = UUID.randomUUID().toString(); // unique owner
int expireTimeMs = 5000; // 5 seconds

Boolean isLocked = jedis.set(lockKey, lockValue, SetParams.setParams().nx().px(expireTimeMs));

if (Boolean.FALSE.equals(isLocked)) {
    throw new RuntimeException("Room is currently being booked. Try again later.");
}
```

* If `isLocked` is false → another user is booking, fail fast
* If true → proceed to Step 2

---

### **Step 2: Check Inventory**

```java
int availableRooms = roomInventoryRepository.getAvailableRooms(roomId, date);
if (availableRooms <= 0) {
    releaseLock(lockKey, lockValue);
    throw new RuntimeException("No rooms available");
}
```

---

### **Step 3: Create Booking (ACID Transaction)**

```java
Booking booking = new Booking();
booking.setRoomId(roomId);
booking.setUserId(userId);
booking.setCheckInDate(date);
booking.setStatus("PENDING");
bookingRepository.save(booking);

// Reduce inventory
roomInventoryRepository.decrementAvailable(roomId, date);
```

---

### **Step 4: Release Lock Safely**

```java
private void releaseLock(String key, String value) {
    String luaScript = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
                       "return redis.call('del', KEYS[1]) " +
                       "else return 0 end";
    jedis.eval(luaScript, Collections.singletonList(key), Collections.singletonList(value));
}
```

* Only releases **if the lock is still owned by this thread** (prevents accidentally deleting another user’s lock)

---

# **5. Handling Timeouts and Retries**

* Set `PX` (expire) to auto-release lock if something goes wrong
* Retry a few times if `SETNX` fails, with **exponential backoff**
* Optionally, implement a **queue** for high-demand rooms

---

# **6. Flow Diagram (Logical)**

```text
User Request → Check Redis Lock
        │
        ├─ Locked? → Fail / Retry
        │
        └─ Acquire Lock
             │
        Check Inventory
             │
        Available? → Fail + Release Lock
             │
        Create Booking (DB Transaction)
             │
        Reduce Inventory
             │
        Release Lock
             │
        Respond Success
```

---

# **7. Benefits of This Approach**

1. **Prevent overbooking** → ensures accurate inventory
2. **Fast** → Redis is in-memory, milliseconds
3. **Distributed safe** → multiple app instances can coordinate
4. **Supports high concurrency** → critical during flash sales, peak seasons
5. **Automatic recovery** → lock auto-expires if something crashes

---

# **8. Production Considerations**

* **Lock expiration**: must be longer than average booking processing time
* **Deadlock prevention**: use Lua script for safe unlock
* **Monitoring**: track failed lock attempts → indicates high load
* **Clustered Redis**: use Redis Cluster or Redisson for high availability





