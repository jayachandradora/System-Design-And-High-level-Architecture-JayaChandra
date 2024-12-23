# Priority Based Kafka Messages Processing from two Kafka topic queues with Real time


To implement a system that processes messages from two Kafka topic queues—one for delta updates (higher priority) and the other for full-load data (lower priority)—while alternating based on their availability, we can design a message consumption system with the following logic:

### Requirements:
1. **Priority Handling**: The delta topic messages have a higher priority, so they should be processed first. The full-load messages are processed only when the delta topic queue is empty.
2. **Alternating Consumption**: If there are delta messages, they should be consumed first. The full-load topic messages are consumed only when no messages are available in the delta topic queue.
3. **Kafka Consumer Setup**: We will use two consumers, one for each topic (delta and full-load).
4. **Polling Logic**: We need to poll messages from both topics in a loop, with priority given to the delta topic.

### Approach:

1. **Set Up Kafka Consumers**:
   - Create two Kafka consumers: one for the delta topic and one for the full-load topic.
   
2. **Message Polling and Processing**:
   - The main loop will continuously poll messages from both topics.
   - If there are messages available in the delta topic, process them first.
   - If no messages are available in the delta topic, then start processing the full-load topic messages.

3. **Concurrency Management**:
   - To avoid blocking on either queue, we can use separate threads for polling each topic. However, we should manage the consumer thread for the full-load topic so that it only runs when the delta topic has no messages.

4. **Graceful Shutdown**:
   - Ensure that the system shuts down gracefully when there are no more messages to consume or the application is stopping.

Here’s how you can implement this in Java:

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.util.Collections;
import java.util.Properties;

public class KafkaPriorityConsumer {
    
    private static final String DELTA_TOPIC = "delta_topic";
    private static final String FULL_LOAD_TOPIC = "full_load_topic";
    private static final String BOOTSTRAP_SERVERS = "localhost:9092";
    
    public static void main(String[] args) {
        
        // Configure the Kafka consumer for delta topic
        Properties deltaProps = new Properties();
        deltaProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        deltaProps.put(ConsumerConfig.GROUP_ID_CONFIG, "delta-consumer-group");
        deltaProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        deltaProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        // Configure the Kafka consumer for full-load topic
        Properties fullLoadProps = new Properties();
        fullLoadProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        fullLoadProps.put(ConsumerConfig.GROUP_ID_CONFIG, "full-load-consumer-group");
        fullLoadProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        fullLoadProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        // Create the Kafka consumers
        KafkaConsumer<String, String> deltaConsumer = new KafkaConsumer<>(deltaProps);
        KafkaConsumer<String, String> fullLoadConsumer = new KafkaConsumer<>(fullLoadProps);

        // Subscribe to topics
        deltaConsumer.subscribe(Collections.singletonList(DELTA_TOPIC));
        fullLoadConsumer.subscribe(Collections.singletonList(FULL_LOAD_TOPIC));

        // Start message processing loop
        try {
            while (true) {
                // Poll for messages from both delta and full-load topics
                var deltaRecords = deltaConsumer.poll(1000);  // 1 second timeout
                var fullLoadRecords = fullLoadConsumer.poll(1000);

                // Process delta messages (higher priority)
                if (!deltaRecords.isEmpty()) {
                    deltaRecords.forEach(record -> {
                        // Process delta message here
                        System.out.println("Processing delta message: " + record.value());
                    });
                }
                
                // Process full-load messages only when delta is empty
                if (deltaRecords.isEmpty() && !fullLoadRecords.isEmpty()) {
                    fullLoadRecords.forEach(record -> {
                        // Process full-load message here
                        System.out.println("Processing full-load message: " + record.value());
                    });
                }
                
                // Add any additional processing or sleep logic as needed
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Close the consumers gracefully
            deltaConsumer.close();
            fullLoadConsumer.close();
        }
    }
}
```

### Explanation of the Code:

1. **Consumer Configuration**:
   - We create two `KafkaConsumer` instances: one for the delta topic and one for the full-load topic.
   - The properties such as `bootstrap.servers` and `group.id` are configured for both consumers. Adjust the group ID to make sure that the consumers can track their offsets independently.

2. **Polling Logic**:
   - We continuously poll both consumers using the `poll()` method, which retrieves records from the Kafka brokers.
   - The `poll()` method has a timeout (1000 milliseconds), and the consumer checks if there are messages available in the delta topic. If there are, it processes them.
   - If the delta topic has no messages (i.e., `deltaRecords.isEmpty()`), the consumer will start processing the full-load topic.

3. **Message Processing**:
   - Once records are retrieved, we process the messages. For the sake of the example, we are just printing the message content to the console, but you can replace this with your actual business logic.

4. **Graceful Shutdown**:
   - The `finally` block ensures that the consumers are closed properly when the application exits, which is a good practice to avoid leaving Kafka connections open.

### Key Points:
- **Priority Handling**: Delta topic messages are processed first. Full-load topic messages are only processed when no delta messages are available.
- **Polling Efficiency**: The `poll()` method is non-blocking with a timeout, so the consumer remains responsive.
- **Scalability**: This design can be extended for more complex priority handling (e.g., different priorities between multiple queues) or to handle more topics.

### Improvements:
- **Threading**: You could further enhance the system by using separate threads or executor services for the two consumers if you want them to operate concurrently in a more complex system.
- **Backpressure Handling**: If the message volume grows, consider implementing backpressure mechanisms or adjusting the consumer's `max.poll.records` setting to control the flow.
- **Error Handling and Dead-letter Queue**: Handle errors gracefully by moving failed messages to a dead-letter queue for further analysis.

This should give you a solid foundation to handle alternating message processing with priority between the delta and full-load Kafka topics.
