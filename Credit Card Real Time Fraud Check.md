# Credit Card Real Time Fraud Check 

Certainly! Below is a simplified example of a real-time fraud detection system using **Apache Storm** in Java. The implementation will consist of three main components: **Spout**, **Bolts**, and the **Topology**.

### Requirements
- **Apache Storm** installed
- **Kafka** or another message broker for real-time data streaming (we’ll assume Kafka for this example)
- **Java 8+**
- **Maven** for project dependency management

### Key Components
- **Spout**: Consumes real-time credit card transactions from Kafka.
- **Bolts**: Several bolts will process these transactions. One will handle basic rule-based checks, another will apply machine learning (ML) models, and a final one will aggregate results and trigger alerts.

### Project Setup

First, ensure you have the necessary dependencies in your `pom.xml` file.

```xml
<dependencies>
    <!-- Apache Storm Dependencies -->
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>2.4.0</version>
    </dependency>

    <!-- Kafka Spout for consuming data from Kafka -->
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-kafka-client</artifactId>
        <version>2.4.0</version>
    </dependency>

    <!-- Machine Learning Libraries (optional, for ML models) -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-math3</artifactId>
        <version>3.6.1</version>
    </dependency>

    <!-- Other dependencies like logging, etc. -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.30</version>
    </dependency>
</dependencies>
```

### 1. Spout: Kafka Spout for Streaming Data

The **Spout** will connect to Kafka and consume real-time credit card transactions. Here’s a basic implementation of a Kafka Spout:

```java
import org.apache.storm.kafka.spout.KafkaSpout;
import org.apache.storm.kafka.spout.KafkaSpoutConfig;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Values;

public class TransactionSpout extends BaseRichSpout {
    private KafkaSpout<String, String> kafkaSpout;

    @Override
    public void open(Map<String, Object> conf, TopologyContext context, SpoutOutputCollector collector) {
        KafkaSpoutConfig<String, String> kafkaSpoutConfig = KafkaSpoutConfig.builder("localhost:9092", "credit-card-transactions")
            .setGroupId("fraud-detection")
            .build();
        kafkaSpout = new KafkaSpout<>(kafkaSpoutConfig);
    }

    @Override
    public void nextTuple() {
        kafkaSpout.nextTuple();
    }

    @Override
    public void ack(Object msgId) {
        kafkaSpout.ack(msgId);
    }

    @Override
    public void fail(Object msgId) {
        kafkaSpout.fail(msgId);
    }

    @Override
    public void emit(Values values) {
        collector.emit(values);
    }
}
```

In the example above, the **KafkaSpout** consumes data from the Kafka topic `credit-card-transactions`. The `nextTuple()` method is called to pull data and pass it to the Storm topology.

### 2. Bolt 1: Rule-based Fraud Check

The first **bolt** performs basic checks, such as verifying whether a transaction amount exceeds a certain threshold (e.g., $1000).

```java
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

public class RuleBasedFraudCheckBolt extends BaseBasicBolt {

    private static final double MAX_TRANSACTION_AMOUNT = 1000.0;

    @Override
    public void execute(Tuple tuple) {
        // Parse the transaction data
        String transactionJson = tuple.getStringByField("transaction");
        Transaction transaction = parseTransaction(transactionJson);

        // Basic rule-based check: amount > $1000
        if (transaction.getAmount() > MAX_TRANSACTION_AMOUNT) {
            // Emit a flag indicating potential fraud
            collector.emit(new Values(transaction, "RuleBasedFraud"));
        } else {
            collector.emit(new Values(transaction, "NoFraud"));
        }
    }

    private Transaction parseTransaction(String json) {
        // Assuming we have a utility function to parse the JSON into a Transaction object
        return new Gson().fromJson(json, Transaction.class);
    }
}
```

Here, we define a simple rule to flag any transactions over $1000 as potential fraud.

### 3. Bolt 2: Machine Learning Fraud Detection

This **bolt** could apply a machine learning model to compare the transaction against historical data patterns (e.g., transaction frequency, location, etc.). For simplicity, let’s assume we have a pre-trained model that can be used.

```java
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

public class MLFraudCheckBolt extends BaseBasicBolt {

    @Override
    public void execute(Tuple tuple) {
        Transaction transaction = (Transaction) tuple.getValueByField("transaction");

        // Machine learning model inference (e.g., predict whether this is fraud)
        boolean isFraud = applyMLModel(transaction);

        if (isFraud) {
            collector.emit(new Values(transaction, "MLFraudDetected"));
        } else {
            collector.emit(new Values(transaction, "NoFraud"));
        }
    }

    private boolean applyMLModel(Transaction transaction) {
        // This is a placeholder for actual ML model inference logic
        // Assume we are using a simple rule-based classifier or a more sophisticated ML model
        return transaction.getAmount() > 500.0;  // Simple rule for example purposes
    }
}
```

In a real-world scenario, this part would involve loading a pre-trained ML model and applying it to the incoming transaction to flag fraudulent activity.

### 4. Bolt 3: Aggregator and Alerting

This **bolt** will aggregate results from the previous two bolts and trigger an alert if fraud is suspected.

```java
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

public class FraudAlertBolt extends BaseBasicBolt {

    @Override
    public void execute(Tuple tuple) {
        String result = tuple.getStringByField("result");

        // If any bolt has flagged the transaction as potential fraud
        if (result.equals("RuleBasedFraud") || result.equals("MLFraudDetected")) {
            // Trigger an alert (could be writing to a database, sending an email, etc.)
            System.out.println("ALERT: Fraudulent transaction detected!");
        }
    }
}
```

### 5. Topology: Wiring Everything Together

Now we’ll wire everything into a **Topology**.

```java
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;

public class FraudDetectionTopology {

    public static void main(String[] args) throws Exception {
        // Create a topology builder
        TopologyBuilder builder = new TopologyBuilder();

        // Add Spout (Kafka Spout to consume transactions)
        builder.setSpout("transaction-spout", new TransactionSpout(), 1);

        // Add Rule-based Fraud Check Bolt
        builder.setBolt("rule-fraud-check", new RuleBasedFraudCheckBolt(), 2)
               .shuffleGrouping("transaction-spout");

        // Add ML-based Fraud Check Bolt
        builder.setBolt("ml-fraud-check", new MLFraudCheckBolt(), 2)
               .shuffleGrouping("transaction-spout");

        // Add Fraud Alerting Bolt
        builder.setBolt("fraud-alert", new FraudAlertBolt(), 1)
               .fieldsGrouping("rule-fraud-check", new Fields("result"))
               .fieldsGrouping("ml-fraud-check", new Fields("result"));

        // Configuring the topology and submitting it to the Storm cluster
        Config config = new Config();
        StormSubmitter.submitTopology("fraud-detection-topology", config, builder.createTopology());
    }
}
```

### Explanation:

- **Spout (`TransactionSpout`)**: The Kafka spout consumes real-time credit card transactions and emits them to downstream bolts.
- **Bolts**:
  - **`RuleBasedFraudCheckBolt`**: Checks transactions based on predefined rules (e.g., amount > $1000).
  - **`MLFraudCheckBolt`**: Applies a machine learning model to detect fraud based on historical data.
  - **`FraudAlertBolt`**: Aggregates results from the rule-based and ML checks and triggers alerts when fraud is detected.
- **Topology (`FraudDetectionTopology`)**: The topology wires the spout and bolts together, defining the flow of data from the spout to the bolts.

### Running the Application

To run this Storm topology:

1. **Set up Storm cluster** (local or distributed mode).
2. **Kafka cluster** running with the `credit-card-transactions` topic.
3. **Run the topology** on the Storm cluster.

```bash
mvn clean package
storm jar target/fraud-detection-1.0-SNAPSHOT.jar com.example.F

### **Overview of Storm Components and Flow**

Let's break down the **Fraud Detection Topology** and explain each component and how the **`fieldsGrouping`** mechanism works in Apache Storm.

### 1. **Spout: `TransactionSpout`**
- **Purpose**: The `TransactionSpout` is responsible for pulling **credit card transaction data** from a messaging system like **Kafka**.
- The **spout** emits individual **transactions** to downstream **bolts**.
- In this example, the spout will be connected to a Kafka topic (`credit-card-transactions`), consuming each transaction and passing it to the topology.

### 2. **Bolts:**
   - **Rule-based Fraud Check (`RuleBasedFraudCheckBolt`)**
     - **Purpose**: This bolt processes incoming transactions and applies simple **rule-based checks**. For example, it flags any transaction with an amount greater than $1000 as "potential fraud."
     - It emits two possible outcomes: **"RuleBasedFraud"** for flagged transactions and **"NoFraud"** for those that are not flagged.
  
   - **ML-based Fraud Check (`MLFraudCheckBolt`)**
     - **Purpose**: This bolt performs fraud detection based on an **ML model**. It might use historical transaction patterns, user behavior, or other factors to classify the transaction as **fraudulent** or **non-fraudulent**.
     - Similarly to the previous bolt, it emits either **"MLFraudDetected"** or **"NoFraud"**.

   - **Fraud Alerting (`FraudAlertBolt`)**
     - **Purpose**: This bolt is the final processing step, where it receives the results from both the rule-based and machine learning fraud detection bolts.
     - If either of the previous bolts flags a transaction as **fraudulent**, this bolt can trigger an alert (e.g., log, notify, store the result in a database, etc.).
     - It is here that we make use of **`fieldsGrouping`** to define how tuples are routed between bolts.

### 3. **`fieldsGrouping` Explanation**

In **Apache Storm**, **grouping** determines how tuples are routed between bolts. There are different types of groupings like **shuffleGrouping**, **globalGrouping**, **allGrouping**, and **fieldsGrouping**. Let’s focus on **`fieldsGrouping`**, as used in your example.

#### What is `fieldsGrouping`?

**`fieldsGrouping`** routes tuples to bolts based on the values of **specific fields** in the tuple. It ensures that tuples with the same value for the specified field(s) are sent to the same instance of a bolt. 

In your example:

```java
builder.setBolt("fraud-alert", new FraudAlertBolt(), 1)
       .fieldsGrouping("rule-fraud-check", new Fields("result"))
       .fieldsGrouping("ml-fraud-check", new Fields("result"));
```

### Breaking it Down:
1. **`fraud-alert` Bolt**: The `fraud-alert` bolt is the final one in the topology that receives the results of the fraud checks (from both rule-based and ML-based checks). This bolt checks the **"result"** field from the previous bolts.

2. **`fieldsGrouping("rule-fraud-check", new Fields("result"))`**:
   - **Meaning**: This means that the `fraud-alert` bolt will receive the output from the **`rule-fraud-check`** bolt and will route the tuples based on the **value of the "result" field**.
   - For example, if the **"result"** field in the tuple is `"RuleBasedFraud"`, the tuple will be routed to the `fraud-alert` bolt that handles fraud alerts for rule-based fraud detection.
   - In essence, `fieldsGrouping` ensures that all tuples from **`rule-fraud-check`** that have the same **"result"** value will go to the same instance of `fraud-alert`.

3. **`fieldsGrouping("ml-fraud-check", new Fields("result"))`**:
   - **Meaning**: This does the same thing as the previous `fieldsGrouping` but for the **`ml-fraud-check`** bolt. Tuples emitted by **`ml-fraud-check`** will be routed to the `fraud-alert` bolt based on the **"result"** field value.
   - If the **"result"** field is `"MLFraudDetected"`, all tuples with this value will go to the same `fraud-alert` bolt instance, which ensures that the fraud detection logic in the `fraud-alert` bolt processes those results together.

### How Does `fieldsGrouping` Work in Practice?

- **Grouping by Field Value**: The key idea is that **tuples with the same field value will be routed to the same bolt instance**.
  
  - If the `rule-fraud-check` bolt emits the value `"RuleBasedFraud"` for a transaction, and the `ml-fraud-check` bolt emits `"MLFraudDetected"`, Storm will use `fieldsGrouping` to ensure that both values are sent to the `fraud-alert` bolt, but based on the **field value** (`result`).
  
  - This ensures that **all tuples that need to be processed for fraud detection are processed in the same bolt instance**, so you can keep the state (e.g., logs, alerts) consistent for those specific cases.

### Key Points About `fieldsGrouping`:
- **Fields**: You can specify one or more fields from the tuple to group by.
- **Tuple Routing**: Storm routes tuples to bolts based on the values of the grouped field(s).
- **Ensures Consistency**: In a fraud detection scenario, this is useful because you want to ensure that all information related to a specific result (e.g., rule-based fraud or ML fraud) gets processed by the same bolt, possibly triggering a consolidated alert.
  
### Example: Flow of Data

Let’s go through an example to visualize how data flows:

1. **Spout** (`TransactionSpout`):
   - Emits a credit card transaction (e.g., a JSON string) to the first bolt.
  
2. **Bolt 1** (`RuleBasedFraudCheckBolt`):
   - Checks the transaction against predefined rules.
   - If it finds that the transaction amount is over $1000, it emits a tuple with the value `"RuleBasedFraud"`.
   - Otherwise, it emits `"NoFraud"`.

3. **Bolt 2** (`MLFraudCheckBolt`):
   - Runs the transaction through an ML model to check for fraud based on historical data.
   - If the model detects fraud, it emits `"MLFraudDetected"`.
   - Otherwise, it emits `"NoFraud"`.

4. **Bolt 3** (`FraudAlertBolt`):
   - Now, this bolt will receive the results of both bolts:
     - If `"RuleBasedFraud"` was emitted from the **Rule-based bolt**, it will be grouped together by the `"result"` field and sent to `fraud-alert`.
     - Similarly, if `"MLFraudDetected"` was emitted by the **ML fraud bolt**, that will also go to the same `fraud-alert` bolt based on the `"result"` field.
   - The `fraud-alert` bolt will then trigger alerts if any fraud is detected.

### Conclusion

- **`fieldsGrouping`** is an important feature in Apache Storm that ensures that tuples with the same field value are grouped and sent to the same bolt. This allows the **Fraud Alert Bolt** to aggregate and process the results from the **Rule-based** and **ML-based** fraud detection checks in a meaningful way.
- In your example, both the **`rule-fraud-check`** and **`ml-fraud-check`** bolts are emitting a `result` field, and **`fraud-alert`** is configured to route those results by that field using **`fieldsGrouping`**.
- This guarantees that the `fraud-alert` bolt will process all fraud alerts together based on the respective fraud detection method, providing a cohesive way to handle alerts and take action.


