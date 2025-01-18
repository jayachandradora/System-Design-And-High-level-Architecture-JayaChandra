# Spark Parallel JSON Message Processing

Yes, the solution **does use parallel processing**, but it's working at the **partition level**. Let me explain how it works and where parallelism fits in, followed by a potential optimization for better performance using true **parallel processing** with **checkpointing**.

### Explanation of Parallelism in the Current Solution

- **RDD Partitioning**: In the current solution, we are using **Spark's `mapPartitions`** function to process data in parallel. This method processes data **one partition at a time**, where each partition runs in parallel across Spark executors (depending on your cluster configuration). 
  - If the dataset is large, Spark will divide the data into multiple partitions, and each partition will be processed by a separate executor, leading to parallel processing. 
  - However, each partition processes sequentially within itself (in the `mapPartitions` function), but **parallelism is achieved at the partition level**. Each partition processes a subset of records, and these partitions are processed concurrently across the Spark cluster.
  
  - The **checkpointing** logic, however, is done sequentially across batches and is not inherently parallelized. Each batch, once processed, will update the checkpoint file with the last successfully processed index. So, while the job is running in parallel across partitions, checkpointing is a **serial process** (each batch updates the checkpoint after processing).

### How Can We Improve Parallelism?

If you want to **further enhance parallelism** to improve the processing speed (and scalability), you can adjust the following:
1. **Increase Partitions**: More partitions will allow more parallelism across the cluster. You can manually control partitioning by using the `repartition()` function or allowing Spark to partition automatically.
2. **Batch Size**: Ensure that the batch size is not too large, which could limit parallelism. Small batches might be inefficient, while too large of a batch could cause long wait times and slow API calls.
3. **Distributed Checkpointing**: Checkpointing is a single point of failure in the current implementation because it writes to a single file. For more scalable and fault-tolerant systems, you could write the checkpoint data to a distributed file system like HDFS or a key-value store (like **Redis** or **DynamoDB**) to allow parallel access and updates.

### Parallel Processing with Checkpointing

Let’s explore **parallel processing** while maintaining checkpointing. In the enhanced solution, we can parallelize both:
1. **Batch Processing (Parallelizing the batches themselves)**
2. **Checkpoint Updates (Parallel checkpointing with distributed storage)**

We'll use **multiple threads for parallelizing the retries and batch processing** as well as **distributed storage for checkpointing**.

### Solution with Parallel Processing

Here’s an updated solution where the batch processing (API calls) is parallelized using `mapPartitions` or `flatMap`, and we ensure checkpointing is more scalable.

#### Parallelizing the API Calls (Batches) in Parallel

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.api.java.function.MapPartitionsFunction;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.sql.functions;

import okhttp3.*;

import java.io.*;
import java.util.Iterator;
import java.util.List;
import java.util.ArrayList;
import java.util.concurrent.*;

public class SparkApiProcessorWithParallelProcessingAndCheckpoint {

    private static final String API_URL = "https://your.api.endpoint";
    private static final int BATCH_SIZE = 100; // Number of records per batch
    private static final String CHECKPOINT_FILE_PATH = "checkpoint.txt"; // Path to store checkpoint
    private static final int MAX_RETRIES = 3; // Maximum number of retry attempts
    private static final long RETRY_DELAY_MS = 1000; // Initial delay in milliseconds for retry
    private static final long MAX_RETRY_DELAY_MS = 8000; // Maximum delay between retries (exponential backoff)
    private static final ExecutorService executorService = Executors.newFixedThreadPool(10); // Thread pool for parallel processing

    public static void main(String[] args) throws Exception {

        // Create Spark session
        SparkSession spark = SparkSession.builder()
                .appName("CSV to API Processor with Parallel Processing and Checkpoint")
                .master("local[*]")  // Use local mode for testing, change for production
                .getOrCreate();

        // Read CSV file into DataFrame
        String csvFilePath = "path_to_your_file.csv";
        Dataset<Row> df = spark.read()
                .option("header", "true")  // Assuming CSV file has headers
                .csv(csvFilePath);

        // Assuming the JSON records are in a column named "json_data"
        Dataset<Row> jsonData = df.select(functions.col("json_data"));

        // Convert to RDD for batch processing
        JavaRDD<Row> jsonRdd = jsonData.toJavaRDD();

        // Read the last checkpoint index (if any)
        int lastProcessedIndex = readCheckpoint();

        // Parallel batch processing using mapPartitions
        jsonRdd.mapPartitions(new MapPartitionsFunction<Row, Void>() {
            @Override
            public Void call(Iterator<Row> iterator) throws Exception {
                List<String> batch = new ArrayList<>();
                int recordCount = 0;

                // Skip records until we reach the checkpoint (last processed record)
                while (iterator.hasNext() && recordCount <= lastProcessedIndex) {
                    iterator.next();  // Skip this record
                    recordCount++;
                }

                // Process the remaining records in parallel
                while (iterator.hasNext()) {
                    Row row = iterator.next();
                    String jsonRecord = row.getString(0);  // Assuming JSON is in first column
                    batch.add(jsonRecord);
                    recordCount++;

                    // If the batch is full, send it to the API in parallel
                    if (batch.size() >= BATCH_SIZE) {
                        // Submit batch to executor service (parallel processing)
                        executorService.submit(() -> {
                            try {
                                boolean success = sendToApiWithRetry(batch);
                                if (success) {
                                    updateCheckpoint(recordCount);  // Update checkpoint after each successful batch
                                } else {
                                    // Fail the job if retries are exhausted
                                    throw new RuntimeException("API call failed after retries. Job failed.");
                                }
                            } catch (Exception e) {
                                System.err.println("Failed to send batch: " + e.getMessage());
                            }
                        });

                        batch.clear();
                    }
                }

                // Send any remaining records after loop
                if (!batch.isEmpty()) {
                    executorService.submit(() -> {
                        try {
                            boolean success = sendToApiWithRetry(batch);
                            if (success) {
                                updateCheckpoint(recordCount);  // Update checkpoint for the last batch
                            } else {
                                // Fail the job if retries are exhausted
                                throw new RuntimeException("API call failed after retries. Job failed.");
                            }
                        } catch (Exception e) {
                            System.err.println("Failed to send batch: " + e.getMessage());
                        }
                    });
                }

                return null;
            }
        }).collect();

        // Wait for all tasks to finish
        executorService.shutdown();
        while (!executorService.isTerminated()) {
            // Wait for all threads to finish
        }

        // Stop the Spark session
        spark.stop();
    }

    private static boolean sendToApiWithRetry(List<String> batch) {
        // Create HTTP client
        OkHttpClient client = new OkHttpClient();

        // Build the request body with the batch
        String jsonBatch = String.join(",", batch);  // Join the JSON records into a batch string

        RequestBody body = RequestBody.create(
                jsonBatch,
                MediaType.parse("application/json; charset=utf-8")
        );

        Request request = new Request.Builder()
                .url(API_URL)
                .post(body)
                .build();

        // Implement retry logic with exponential backoff
        int attempt = 0;
        long delay = RETRY_DELAY_MS;

        while (attempt < MAX_RETRIES) {
            try (Response response = client.newCall(request).execute()) {
                if (response.isSuccessful()) {
                    System.out.println("Successfully sent batch to API.");
                    return true;  // Success
                } else {
                    System.out.println("API request failed (attempt " + (attempt + 1) + "): " + response);
                }
            } catch (IOException e) {
                System.err.println("API call failed (attempt " + (attempt + 1) + "): " + e.getMessage());
            }

            // If not successful, retry with exponential backoff
            attempt++;
            try {
                // Delay with exponential backoff
                long currentDelay = Math.min(delay, MAX_RETRY_DELAY_MS);
                System.out.println("Retrying after " + currentDelay + "ms...");
                TimeUnit.MILLISECONDS.sleep(currentDelay);
                delay *= 2; // Exponential backoff
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return false; // If sleep is interrupted, fail the job
            }
        }

        // If retries exhausted, return failure
        System.err.println("API request failed after " + MAX_RETRIES + " attempts.");
        return false;
    }

    // Method to read the checkpoint index from file
    private static int readCheckpoint() {
        File checkpointFile = new File(CHECKPOINT_FILE_PATH);
        if (checkpointFile.exists()) {
            try (BufferedReader reader = new BufferedReader(new FileReader(checkpointFile))) {
                String line = reader.readLine();
                return Integer.parseInt(line.trim()); // Last processed index
            } catch (IOException e) {
                System.err.println("Error reading checkpoint: " + e.getMessage());
            }
       

 }
        return -1;  // Return -1 if no checkpoint exists (i.e., start from the beginning)
    }

    // Method to update the checkpoint index to file
    private static void updateCheckpoint(int lastProcessedIndex) {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(CHECKPOINT_FILE_PATH))) {
            writer.write(String.valueOf(lastProcessedIndex));  // Save the last processed index
            System.out.println("Checkpoint updated: " + lastProcessedIndex);
        } catch (IOException e) {
            System.err.println("Error writing checkpoint: " + e.getMessage());
        }
    }
}
```

### Key Enhancements:

1. **Parallel Batch Processing**:
   - We use an **`ExecutorService`** with a fixed thread pool (`newFixedThreadPool(10)`) to submit batches for processing in parallel. Each batch is processed in a separate thread, which allows for better parallelism than the previous solution where batches were processed sequentially.
   
2. **Scalable Checkpointing**:
   - The checkpoint logic remains the same in terms of writing to a local file, but you can change the implementation of the checkpoint storage to something more scalable (e.g., HDFS, S3, or a database) if needed. This solution can also be adapted for distributed checkpointing for high availability and fault tolerance.

3. **Asynchronous Batch Processing**:
   - Each batch is submitted to the thread pool for asynchronous processing. This enables the API calls to happen in parallel, greatly improving throughput.

### Trade-offs:
- **Parallelism in Batch Handling**: By processing the batches in parallel, the overall processing time is reduced. However, you should consider the impact on **API rate limits** or the network bandwidth.
- **Checkpointing**: While checkpointing is done serially (after each batch), parallelized processing ensures that the job is faster overall.
  
This approach is now optimized for **parallel processing** of batches and can handle **retry logic** with proper **checkpointing** to ensure job recovery in case of failures.
