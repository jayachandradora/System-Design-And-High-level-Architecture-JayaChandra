# Notification System Design 

![image](https://user-images.githubusercontent.com/115500959/201484751-cc1c7e2a-c4b1-4fd0-8d46-487c94754de7.png)

## Problem Statement:

1. Single Point of failure
2. Server overloaded
3. Message lost
4. Deficult to scal

![image](https://user-images.githubusercontent.com/115500959/201484524-e3fc0e1f-818a-4904-8544-56418cdbf9cf.png)

![image](https://user-images.githubusercontent.com/115500959/201484601-1a963c80-96d9-472a-b51a-81ab688bd1af.png)

![image](https://user-images.githubusercontent.com/115500959/201484640-1d2d0afb-4a0b-4bf5-8c37-7636d8336591.png)


![image](https://user-images.githubusercontent.com/115500959/201504916-01f4b99f-3568-48de-b262-2c1decfdd952.png)

![image](https://user-images.githubusercontent.com/115500959/201505025-7763c3d5-1afa-480f-aa4d-dc71e0a4a434.png)

![image](https://user-images.githubusercontent.com/115500959/201505091-1b5e6b0d-b21b-4c26-8142-fbbac1bcf353.png)

# Handling a fanout message

<img width="937" alt="image" src="https://github.com/user-attachments/assets/eee8ef0c-a45f-4567-8f74-03ebb1a36e6b">


Handling a fanout message to send notifications, such as when a celebrity posts a message to all their followers, can be effectively managed using several strategies. Here’s an outline of best practices and approaches:

### 1. **Message Queue or Streaming Platform**
   - **Use Kafka or RabbitMQ**: These tools can help fan out messages efficiently. Each follower can be represented as a consumer that subscribes to messages from the celebrity.

### 2. **Topics and Subscriptions**
   - **Single Topic for Celebrity Posts**: Create a single topic (e.g., `celebrity-posts`) where all messages from celebrities are published.
   - **Partitioning**: Use partitions to distribute messages across multiple consumers for better performance and scalability.

### 3. **Fanout Mechanism**
   - **Direct Fanout**: When a celebrity posts a message, publish it to a topic that all followers subscribe to. Each follower's notification service consumes from this topic.
   - **Multiple Topics**: If needed, you can create a topic per celebrity. This can help in isolating messages and managing subscriptions more granularly.

### 4. **Notification Service Design**
   - **Service Architecture**:
     - **Producer Service**: Responsible for publishing messages to the `celebrity-posts` topic.
     - **Notification Service**: Consumes messages and handles notification delivery to followers.
     - **Database**: Store follower information, including which notifications have been sent.
   - **Batch Processing**: To reduce load, consider processing notifications in batches (e.g., sending notifications to multiple followers in one API call).

### 5. **Scaling**
   - **Horizontal Scaling**: Scale the notification service horizontally by adding more instances to handle increased load, especially during high-traffic events (e.g., celebrity events or trending topics).
   - **Dynamic Consumer Groups**: Allow dynamic scaling of consumers based on load. For example, you can spin up more consumer instances when a celebrity posts.

### 6. **Handling Delivery and Failures**
   - **Retry Mechanism**: Implement a retry mechanism for failed notification deliveries (e.g., if a push notification fails, try again later).
   - **Dead Letter Queue**: Use a dead letter queue to handle messages that fail after a certain number of retries, enabling further investigation.

### 7. **Rate Limiting**
   - **Throttle Notifications**: To avoid overwhelming users, implement rate limiting on the number of notifications sent within a specific timeframe.

### 8. **User Preferences**
   - **Notification Settings**: Allow users to customize their notification preferences (e.g., types of posts they want to receive notifications for).
   - **Opt-in/Opt-out**: Provide options for users to opt-in or opt-out of notifications from specific celebrities.

### 9. **Example Workflow for Celebrity Posting**
1. **Celebrity Posts Message**: A celebrity creates a new post (e.g., “Excited for my upcoming concert!”).
2. **Publish to Topic**: The producer service publishes the message to the `celebrity-posts` topic.
3. **Notification Service Consumes Message**: The notification service listens to the topic and receives the new post.
4. **Send Notifications**:
   - The notification service fetches the list of followers from the database.
   - It sends notifications (e.g., push notifications, emails) to all followers who opted in.
5. **Track Delivery**: The service logs the status of each notification for tracking and potential retries.

### 10. **Monitoring and Analytics**
- **Track Engagement**: Monitor how users interact with notifications to refine strategies.
- **Alerting**: Set up alerts for failures in the notification delivery process to ensure timely responses to issues.

By employing these strategies, you can effectively manage fanout messaging for notification services, ensuring timely and efficient delivery to followers of celebrities or any other entities in your application.

### Fanout message

When a celebrity posts a message, the publisher sends it to the corresponding celebrity topic. A fanout service then reads these messages and collects user data from various databases, including a graph database, user database, preference database (for opt-outs), and an infinity database (which stores information about messages received by consumers each day).

The service aggregates this data and categorizes it based on priority levels (P0, P1, P2, P3, etc.) to determine how to send the messages effectively.

<img width="937" alt="image" src="https://github.com/user-attachments/assets/f905a1b8-5945-4468-84cb-3571fb9c4441">


