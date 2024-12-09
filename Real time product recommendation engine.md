# Real time product recommendation engine using Apache Storm 

Building a **real-time product recommendation engine** using **Apache Storm** that integrates **purchase history**, **last viewed products**, **similar products**, and **geolocation data** can be an exciting and complex task. Below is an outline of how this can be structured, including a **machine learning model** to predict recommended products based on purchasing behavior.

### Steps to Build the Recommendation Engine

### 1. **Define Data Sources**

You need multiple sources of data to drive recommendations:
- **Purchase History**: Data about past user purchases.
- **Last Viewed Products**: Information on the most recent products a user has interacted with.
- **Similar Products**: Based on item-item or user-item similarity algorithms (e.g., collaborative filtering).
- **Geolocation Data**: Can be derived from user's IP address or GPS data to suggest products that are relevant to the user's location.

### 2. **Streaming Data Pipeline Architecture**

Apache Storm is a real-time stream processing framework that will allow us to continuously process and aggregate data in real-time. You will need several components in your pipeline:

- **Spouts**: Components that will pull data from various sources like purchase history, last viewed products, geolocation, etc.
- **Bolts**: Components that process this data and perform transformations like filtering, grouping, and running the machine learning model.
- **Data Sink**: Where the final recommendations are sent, either to a database, an API, or a front-end application.

### 3. **Machine Learning for Purchase History Data**

For the recommendation engine, you can leverage a **Collaborative Filtering** approach (Matrix Factorization, Singular Value Decomposition (SVD), or K-nearest neighbors) based on **purchase history**.

#### Example Machine Learning Workflow for Purchase History:
1. **Data Preparation**: The user purchase history is stored in a **user-item interaction matrix**, where each row represents a user and each column represents an item.
    - Example: A user-item matrix where rows represent users and columns represent products they’ve purchased. The values can be binary (1 if purchased, 0 if not) or more advanced (e.g., rating or number of purchases).

2. **Train a Collaborative Filtering Model**:
    - Use algorithms such as **Matrix Factorization** or **KNN** to learn item-item or user-item similarity.
    - For example, a **user-item collaborative filter** uses similarities between users and items to predict the likelihood of a user purchasing an item they haven’t yet bought.

3. **Real-time Prediction**:
    - As users interact with the platform, you can update predictions on-the-fly using **streaming data** and a model that can continuously learn from new user purchases.

#### Example Model: Matrix Factorization (MF)
This would decompose the user-item interaction matrix into latent factors. The predicted rating for a user and product can be computed as:

\[
\hat{r}_{ui} = \mathbf{P_u} \cdot \mathbf{Q_i}^T
\]

Where:
- \( \hat{r}_{ui} \) is the predicted rating for user \(u\) and item \(i\)
- \( \mathbf{P_u} \) is the user’s latent feature vector
- \( \mathbf{Q_i} \) is the item’s latent feature vector

You can deploy this in Apache Storm by updating the latent feature vectors as new interactions come in.

### 4. **Real-time Recommendation Engine in Apache Storm**

Below is a step-by-step breakdown of how to implement this in Apache Storm:

#### **Step 1: Set up Spouts (Data Sources)**

Each spout pulls data from different sources:
- **Purchase History Spout**: Continuously pulls purchase history data, either from a database or a message queue (e.g., Kafka).
- **Last Viewed Products Spout**: Collects recent product views.
- **Geolocation Spout**: Captures user geolocation (IP or GPS).
- **Item Similarity Spout**: Provides similar products based on collaborative filtering or content-based methods.

#### **Step 2: Implement Bolts (Processing Units)**

Bolts process data from spouts and compute the recommendations.

##### **Purchase History & Machine Learning Model Bolt**
- Process the **purchase history** data to update user-product interaction matrices.
- Apply a **Collaborative Filtering** or other ML algorithm to predict which products a user may be interested in.
- You may use an algorithm like **ALS (Alternating Least Squares)** or **KNN** for collaborative filtering.
  
##### **Last Viewed Products & Geolocation Bolt**
- For **last viewed products**, identify the most recent products the user has interacted with and recommend related items.
- For **geolocation**, filter recommendations based on the user’s location. For example, suggest nearby stores or products that are popular in the user's region.

##### **Recommendation Aggregator Bolt**
- Combine the results from the **purchase history model**, **last viewed products**, and **geolocation** data to generate the final recommendation list.
- Apply business logic, such as promoting trending items, discounts, or seasonal products.

#### **Step 3: Output Sink**

Once recommendations are calculated, output them to a **database**, **front-end API**, or messaging queue for real-time user interaction.

### 5. **Flow Example:**

1. **Spouts**:
   - **Purchase History Spout**: Stream of purchase events.
   - **Last Viewed Products Spout**: Stream of recently viewed products.
   - **Geolocation Spout**: Stream of location data.
   - **Similar Products Spout**: Stream of similar products based on ML or static item similarity algorithms.

2. **Bolts**:
   - **Collaborative Filtering Bolt**: Processes purchase history data and applies the ML model (e.g., Matrix Factorization) to predict products for the user.
   - **Location-based Filtering Bolt**: Processes geolocation data to filter recommended products based on user location.
   - **Item Similarity Bolt**: Processes data from "last viewed" products and fetches similar products.
   - **Recommendation Aggregation Bolt**: Combines results from the previous bolts and generates a final recommendation list.

3. **Sink**: The recommended products are output to a database or an API for delivery to the user’s device.

### 6. **Sample Code for Apache Storm Topology**

```java
public class ProductRecommendationTopology {
    public static void main(String[] args) throws Exception {
        // Create Storm config
        Config conf = new Config();
        
        // Define spouts
        PurchaseHistorySpout purchaseHistorySpout = new PurchaseHistorySpout();
        LastViewedProductsSpout lastViewedProductsSpout = new LastViewedProductsSpout();
        GeolocationSpout geolocationSpout = new GeolocationSpout();
        ItemSimilaritySpout itemSimilaritySpout = new ItemSimilaritySpout();

        // Define bolts (Recommendation processing)
        CollaborativeFilteringBolt collaborativeFilteringBolt = new CollaborativeFilteringBolt();
        LocationBasedFilteringBolt locationBasedFilteringBolt = new LocationBasedFilteringBolt();
        ItemSimilarityBolt itemSimilarityBolt = new ItemSimilarityBolt();
        RecommendationAggregatorBolt recommendationAggregatorBolt = new RecommendationAggregatorBolt();

        // Create Topology builder
        TopologyBuilder builder = new TopologyBuilder();
        
        builder.setSpout("purchase-history-spout", purchaseHistorySpout);
        builder.setSpout("last-viewed-products-spout", lastViewedProductsSpout);
        builder.setSpout("geolocation-spout", geolocationSpout);
        builder.setSpout("item-similarity-spout", itemSimilaritySpout);
        
        builder.setBolt("collaborative-filtering-bolt", collaborativeFilteringBolt).shuffleGrouping("purchase-history-spout");
        builder.setBolt("location-filtering-bolt", locationBasedFilteringBolt).shuffleGrouping("geolocation-spout");
        builder.setBolt("item-similarity-bolt", itemSimilarityBolt).shuffleGrouping("last-viewed-products-spout");
        builder.setBolt("recommendation-aggregator-bolt", recommendationAggregatorBolt)
                .shuffleGrouping("collaborative-filtering-bolt")
                .shuffleGrouping("location-filtering-bolt")
                .shuffleGrouping("item-similarity-bolt");

        // Set up the local cluster
        LocalCluster cluster = new LocalCluster();
        cluster.submitTopology("product-recommendation-topology", conf, builder.createTopology());
        
        // Run topology for a specific duration
        Thread.sleep(60000);
        cluster.shutdown();
    }
}
```

### 7. **Conclusion**

By using **Apache Storm** to create a real-time product recommendation engine, you can:
- Dynamically adjust recommendations based on user interactions and geolocation.
- Incorporate machine learning models such as **Collaborative Filtering** to improve recommendation quality over time.
- Leverage streaming data to provide personalized, location-based, and contextually relevant product suggestions.


Certainly! Below is a detailed example of how to implement **Spouts** and **Bolts** for the real-time product recommendation engine using **Apache Storm**. I'll provide example code for each component (spouts and bolts), following the structure we discussed earlier. The components will work together to process streams of purchase history, viewed products, geolocation, and item similarity data in real time.

---

### 1. **Purchase History Spout**
This spout will simulate or stream user purchase history data.

```java
public class PurchaseHistorySpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private Random random;

    @Override
    public void open(Map<String, Object> config, TopologyContext context, SpoutOutputCollector collector) {
        this.collector = collector;
        this.random = new Random();
    }

    @Override
    public void nextTuple() {
        // Simulate purchase events (for demo purposes)
        String userId = "user_" + random.nextInt(1000); // Example user
        String productId = "product_" + random.nextInt(100); // Example product
        long timestamp = System.currentTimeMillis(); // Timestamp for the purchase
        
        // Emit purchase history event
        collector.emit(new Values(userId, productId, timestamp));
        
        // Sleep for a while to simulate real-time processing
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void close() {
        // Clean up resources
    }

    @Override
    public void activate() {}

    @Override
    public void deactivate() {}
}
```

### 2. **Last Viewed Products Spout**
This spout will simulate or stream user interactions with products.

```java
public class LastViewedProductsSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private Random random;

    @Override
    public void open(Map<String, Object> config, TopologyContext context, SpoutOutputCollector collector) {
        this.collector = collector;
        this.random = new Random();
    }

    @Override
    public void nextTuple() {
        // Simulate viewed product events
        String userId = "user_" + random.nextInt(1000); // Example user
        String productId = "product_" + random.nextInt(100); // Example product
        long timestamp = System.currentTimeMillis(); // Timestamp for the view

        // Emit view event
        collector.emit(new Values(userId, productId, timestamp));

        // Sleep to simulate continuous data stream
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void close() {
        // Clean up resources
    }

    @Override
    public void activate() {}

    @Override
    public void deactivate() {}
}
```

### 3. **Geolocation Spout**
This spout will stream location data based on user's IP address or GPS coordinates.

```java
public class GeolocationSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private Random random;

    @Override
    public void open(Map<String, Object> config, TopologyContext context, SpoutOutputCollector collector) {
        this.collector = collector;
        this.random = new Random();
    }

    @Override
    public void nextTuple() {
        // Simulate geolocation data
        String userId = "user_" + random.nextInt(1000); // Example user
        String location = "Location_" + random.nextInt(50); // Example location (city, region, etc.)
        
        // Emit geolocation event
        collector.emit(new Values(userId, location));

        // Sleep to simulate real-time data stream
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void close() {
        // Clean up resources
    }

    @Override
    public void activate() {}

    @Override
    public void deactivate() {}
}
```

### 4. **Similar Products Spout**
This spout would provide similar products, either based on pre-calculated similarity or dynamically generated from machine learning models.

```java
public class SimilarProductsSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private Random random;

    @Override
    public void open(Map<String, Object> config, TopologyContext context, SpoutOutputCollector collector) {
        this.collector = collector;
        this.random = new Random();
    }

    @Override
    public void nextTuple() {
        // Simulate similar product data
        String productId = "product_" + random.nextInt(100); // Example product
        String similarProductId = "similar_product_" + random.nextInt(100); // Similar product

        // Emit similar products data
        collector.emit(new Values(productId, similarProductId));

        // Sleep to simulate real-time data stream
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void close() {
        // Clean up resources
    }

    @Override
    public void activate() {}

    @Override
    public void deactivate() {}
}
```

---

### 5. **Collaborative Filtering Bolt**
This bolt processes purchase history data and applies a collaborative filtering model (e.g., Matrix Factorization, KNN) to generate recommendations.

```java
import backtype.storm.task.TopologyContext;
import backtype.storm.tuple.Tuple;
import backtype.storm.tuple.Values;
import backtype.storm.spout.BaseRichSpout;
import backtype.storm.bolt.BaseRichBolt;
import backtype.storm.outputcollector.OutputCollector;

import java.util.*;
import org.apache.commons.math3.linear.*;

public class CollaborativeFilteringBolt extends BaseRichBolt {
    private OutputCollector collector;
    
    // A mock user-item interaction matrix
    private Map<String, Map<String, Integer>> userProductMatrix;

    @Override
    public void prepare(Map<String, Object> config, TopologyContext context, OutputCollector collector) {
        this.collector = collector;

        // Initialize the user-product interaction matrix (userId -> productId -> rating)
        // In practice, this should come from a data source like a database or stream of purchase history
        userProductMatrix = new HashMap<>();
        
        // Example data: users and products they've purchased (rating is 1 for simplicity)
        userProductMatrix.put("user_1", Map.of("product_1", 1, "product_2", 1));
        userProductMatrix.put("user_2", Map.of("product_2", 1, "product_3", 1));
        userProductMatrix.put("user_3", Map.of("product_1", 1, "product_3", 1));
    }

    @Override
    public void execute(Tuple input) {
        String userId = input.getStringByField("userId");
        String productId = input.getStringByField("productId");
        long timestamp = input.getLongByField("timestamp");

        // Apply collaborative filtering logic to generate recommendations
        // Example: Predict products for the user based on purchase history
        List<String> recommendedProducts = getRecommendedProducts(userId);

        // Emit recommended products for the user
        for (String recommendedProduct : recommendedProducts) {
            collector.emit(input, new Values(userId, recommendedProduct));
        }

        // Acknowledge tuple processing
        collector.ack(input);
    }

    /**
     * Generates recommendations for the user based on collaborative filtering (KNN-based).
     *
     * @param userId the user for whom to generate recommendations
     * @return a list of recommended product IDs
     */
    private List<String> getRecommendedProducts(String userId) {
        List<String> recommendedProducts = new ArrayList<>();

        // Step 1: Find similar users to the current user (userId)
        List<String> similarUsers = findSimilarUsers(userId);

        // Step 2: Collect products purchased by similar users but not by the current user
        Set<String> userPurchasedProducts = new HashSet<>(userProductMatrix.getOrDefault(userId, Collections.emptyMap()).keySet());

        // Step 3: For each similar user, recommend products they have purchased
        for (String similarUser : similarUsers) {
            Map<String, Integer> similarUserProducts = userProductMatrix.getOrDefault(similarUser, Collections.emptyMap());
            for (String product : similarUserProducts.keySet()) {
                if (!userPurchasedProducts.contains(product) && !recommendedProducts.contains(product)) {
                    recommendedProducts.add(product);
                }
            }
        }

        return recommendedProducts;
    }

    /**
     * Finds the most similar users to the given user based on purchase history.
     * For simplicity, we'll use cosine similarity to calculate similarity between users.
     *
     * @param userId the user for whom to find similar users
     * @return a list of similar users
     */
    private List<String> findSimilarUsers(String userId) {
        Map<String, Integer> currentUserProducts = userProductMatrix.getOrDefault(userId, Collections.emptyMap());
        List<String> similarUsers = new ArrayList<>();

        // Step 1: Compute the similarity of the current user with all other users
        Map<String, Double> userSimilarities = new HashMap<>();
        for (String otherUserId : userProductMatrix.keySet()) {
            if (!otherUserId.equals(userId)) {
                Map<String, Integer> otherUserProducts = userProductMatrix.get(otherUserId);
                double similarity = calculateCosineSimilarity(currentUserProducts, otherUserProducts);
                userSimilarities.put(otherUserId, similarity);
            }
        }

        // Step 2: Sort users by similarity score (highest similarity first)
        userSimilarities.entrySet().stream()
            .sorted((e1, e2) -> Double.compare(e2.getValue(), e1.getValue())) // Sort descending by similarity score
            .limit(3) // Limit to top 3 most similar users
            .forEach(entry -> similarUsers.add(entry.getKey()));

        return similarUsers;
    }

    /**
     * Calculates cosine similarity between two users' product purchases.
     *
     * @param user1Products the product purchase map for the first user
     * @param user2Products the product purchase map for the second user
     * @return the cosine similarity between the two users
     */
    private double calculateCosineSimilarity(Map<String, Integer> user1Products, Map<String, Integer> user2Products) {
        // Find common products between the two users
        Set<String> commonProducts = new HashSet<>(user1Products.keySet());
        commonProducts.retainAll(user2Products.keySet());

        // If there are no common products, return similarity of 0
        if (commonProducts.isEmpty()) {
            return 0;
        }

        // Calculate the dot product and magnitudes for cosine similarity
        double dotProduct = 0;
        double magnitudeUser1 = 0;
        double magnitudeUser2 = 0;

        for (String product : commonProducts) {
            dotProduct += user1Products.get(product) * user2Products.get(product);
        }

        for (String product : user1Products.keySet()) {
            magnitudeUser1 += Math.pow(user1Products.get(product), 2);
        }

        for (String product : user2Products.keySet()) {
            magnitudeUser2 += Math.pow(user2Products.get(product), 2);
        }

        return dotProduct / (Math.sqrt(magnitudeUser1) * Math.sqrt(magnitudeUser2));
    }

    @Override
    public void cleanup() {
        // Clean up resources
    }
}
```

---

### 6. **Location-Based Filtering Bolt**
This bolt filters the recommended products based on the user's location.

```java
public class LocationBasedFilteringBolt extends BaseRichBolt {
    private OutputCollector collector;

    @Override
    public void prepare(Map<String, Object> config, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
    }

    @Override
    public void execute(Tuple input) {
        String userId = input.getStringByField("userId");
        String location = input.getStringByField("location");
        String recommendedProduct = input.getStringByField("recommendedProduct");

        // Filter recommendations based on location
        if (isLocationRelevant(location, recommendedProduct)) {
            // If the product is relevant to the user's location, emit the recommendation
            collector.emit(input, new Values(userId, recommendedProduct, location));
        }

        // Acknowledge tuple processing
        collector.ack(input);
    }

    private boolean isLocationRelevant(String location, String productId) {
        // Implement location-based filtering logic
        // For simplicity, return true to allow all products
        return true;
    }

    @Override
    public void cleanup() {
        // Clean up resources
    }
}
```

---

### 7. **Item Similarity Bolt**
This bolt fetches similar products based on user activity (e.g., last viewed products).

```java
public class ItemSimilarityBolt extends BaseRichBolt {
    private OutputCollector collector;

    @Override
    public void prepare(Map<String, Object> config, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
    }

    @Override
    public void execute(Tuple input) {
        String userId = input.getStringByField("userId");
        String viewedProductId = input.getStringByField("productId");

        // Fetch similar products for the viewed product
        List<String> similarProducts = getSimilarProducts(viewedProductId);

        // Emit similar product recommendations for the user
        for (String similarProduct : similarProducts) {
            collector.emit(input, new Values(userId, similarProduct));
        }

        // Acknowledge tuple processing
        collector.ack(input);
    }

    private List<String> getSimilarProducts(String productId) {
        // Implement item similarity logic (e.g., content-based filtering)
        // For simplicity, returning dummy similar products
        List<String> similarProducts = new ArrayList<>();
        similarProducts.add("similar_product_" + (new Random().nextInt(100)));
        similarProducts.add("similar_product_" + (new Random().nextInt(100)));
        return similarProducts;
    }

    @Override
    public void cleanup() {
        // Clean up resources


    }
}
```

---

### 8. **Recommendation Aggregation Bolt**
This bolt combines recommendations from various sources (Collaborative Filtering, Location Filtering, Item Similarity).

```java
public class RecommendationAggregatorBolt extends BaseRichBolt {
    private OutputCollector collector;

    @Override
    public void prepare(Map<String, Object> config, TopologyContext context, OutputCollector collector) {
        this.collector = collector;
    }

    @Override
    public void execute(Tuple input) {
        String userId = input.getStringByField("userId");
        String recommendedProduct = input.getStringByField("recommendedProduct");
        String location = input.getStringByField("location");

        // Aggregate recommendations (e.g., add priority, ranking, etc.)
        // Emit the final product recommendation
        collector.emit(new Values(userId, recommendedProduct, location));

        // Acknowledge tuple processing
        collector.ack(input);
    }

    @Override
    public void cleanup() {
        // Clean up resources
    }
}
```

---

### Conclusion

- **Spouts** are responsible for emitting streams of real-time data (purchase history, last viewed products, geolocation, and similar products).
- **Bolts** process these streams in parallel, applying machine learning models, filtering based on location, and aggregating results.
- The final **Recommendation Aggregator Bolt** combines all processed recommendations and outputs them to a **Sink** (e.g., database, API).

By following this structure, you can implement a scalable, real-time recommendation engine using **Apache Storm**!
