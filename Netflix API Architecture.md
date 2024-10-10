# Netflix API architecture

The Netflix API architecture went through 4 main stages. 

𝐌𝐨𝐧𝐨𝐥𝐢𝐭𝐡. The application is packaged and deployed as a monolith, such as a single Java WAR file, Rails app, etc. Most startups begin with a monolith architecture.

𝐃𝐢𝐫𝐞𝐜𝐭 𝐚𝐜𝐜𝐞𝐬𝐬. In this architecture, a client app can make requests directly to the microservices. With hundreds or even thousands of microservices, exposing all of them to clients is not ideal.

𝐆𝐚𝐭𝐞𝐰𝐚𝐲 𝐚𝐠𝐠𝐫𝐞𝐠𝐚𝐭𝐢𝐨𝐧 𝐥𝐚𝐲𝐞𝐫. Some use cases may span multiple services, we need a gateway aggregation layer. Imagine the Netflix app needs 3 APIs  (movie, production, talent) to render the frontend. The gateway aggregation layer makes it possible.

𝐅𝐞𝐝𝐞𝐫𝐚𝐭𝐞𝐝 𝐠𝐚𝐭𝐞𝐰𝐚𝐲. As the number of developers grew and domain complexity increased, developing the API aggregation layer became increasingly harder. GraphQL federation allows Netflix to set up a single GraphQL gateway that fetches data from all the other APIs.

## Designing a Netflix Clickstream Data Pipeline

A Netflix Clickstream Data Pipeline is responsible for collecting, processing, and analyzing user interaction data to understand viewing behavior, recommend content, and improve the overall user experience.

### Functional Requirements

1. **Data Collection:**
   * **Event Capture:** Capture user interactions like clicks, hovers, scrolls, and play/pause events.
   * **Data Enrichment:** Augment captured data with additional context (e.g., user ID, device type, location, time).

2. **Data Ingestion:**
   * **Real-time Ingestion:** Ingest data in real-time to enable immediate analysis.
   * **Batch Ingestion:** Process historical data in batches for offline analysis.

3. **Data Storage:**
   * **Scalable Storage:** Choose a scalable storage solution like S3, Azure Data Lake Storage, or Google Cloud Storage.
   * **Data Lake or Data Warehouse:** Decide whether to use a data lake for raw data or a data warehouse for structured data.

4. **Data Processing:**
   * **ETL/ELT:** Extract, Transform, Load or Extract, Load, Transform processes to clean, normalize, and prepare data for analysis.
   * **Batch Processing:** Use tools like Apache Spark or Hadoop for large-scale data processing.
   * **Streaming Processing:** Use tools like Apache Flink or Kafka for real-time data processing.

5. **Data Analysis:**
   * **Descriptive Analytics:** Analyze basic statistics and trends (e.g., most watched shows, popular genres).
   * **Predictive Analytics:** Use machine learning models to predict user behavior and recommend content.
   * **Prescriptive Analytics:** Suggest actions based on insights and predictions (e.g., personalized recommendations).

6. **Visualization:**
   * **Dashboards and Reports:** Create interactive dashboards to visualize key metrics and trends.
   * **Data Visualization Tools:** Use tools like Tableau, Power BI, or Looker.

### Non-Functional Requirements

1. **Scalability:** The pipeline should be able to handle increasing amounts of data and user traffic.
2. **Performance:** Data processing and analysis should be efficient to deliver timely insights.
3. **Reliability:** The system should be highly available and resilient to failures.
4. **Security:** Protect user data and prevent unauthorized access.
5. **Cost-Efficiency:** Optimize resource utilization and costs.
6. **Flexibility:** The pipeline should be adaptable to changes in data sources, processing requirements, and analysis needs.

### Example Architecture

[Image of a Netflix Clickstream Data Pipeline architecture]

A typical pipeline might include:

* **Data Collection:** JavaScript SDKs on Netflix clients capture user interactions.
* **Data Ingestion:** Kafka or Kinesis for real-time ingestion.
* **Data Storage:** S3 for raw data, and a data warehouse like Redshift for structured data.
* **Data Processing:** Spark for batch processing, Flink for streaming processing.
* **Data Analysis:** SQL queries, machine learning models.
* **Visualization:** Tableau for creating dashboards.

**Note:** This is a general overview, and the specific components and technologies may vary depending on Netflix's internal requirements and infrastructure.


![image](https://user-images.githubusercontent.com/115500959/206849624-09068de7-a7c3-47be-878e-e6c95d2d81a7.png)

