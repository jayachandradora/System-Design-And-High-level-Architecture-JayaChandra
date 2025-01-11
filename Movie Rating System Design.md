
# Movie Rating System Design


### Functional Requirements

1. **Rate a Movie**  
   Endpoint: `POST /movies/{movie_id}/ratings`  
   - Allow users to submit a rating between 1 and 5.
   - Reject invalid ratings with an error message.

2. **Retrieve All Ratings for a Movie**  
   Endpoint: `GET /movies/{movie_id}/ratings`  
   - Return a list of individual ratings for the specified movie.

3. **Rating Summary**  
   Endpoint: `GET /movies/{movie_id}/rating-summary`  
   - Calculate and return:
     - Total Rating (sum of all individual ratings)
     - Average Rating (rounded to 1 decimal place)

4. **Get Movie Details**  
   Endpoint: `GET /movies/{movie_id}`  
   - Retrieve movie details (e.g., title, genre, release year).

5. **Update/Delete Rating**  
   Endpoint: `PUT /movies/{movie_id}/ratings/{rating_id}` or `DELETE /movies/{movie_id}/ratings/{rating_id}`  
   - Allow users to modify or delete their rating for a movie.

---

### Non-Functional Requirements

1. **Performance**  
   - Total and average ratings should be available within 200-500ms.  
   - The system should handle 1,000+ requests per second.

2. **Scalability**  
   - Efficient database indexing and sharding to support millions of ratings.
   - Use of caching (e.g., Redis) for frequently accessed data.

3. **Reliability & Availability**  
   - API should have 99.9% uptime and be resilient to failures.  
   - Database replication for high availability and basic monitoring.

4. **Data Consistency**  
   - Ensure correct recalculation of total and average ratings on new submissions.  
   - Use transactions for consistent updates to ratings and summaries.  
   - Implement eventual consistency for cached data.

5. **Security & Authentication**  
   - API endpoints protected by authentication (e.g., OAuth, JWT tokens).  
   - Rate limiting to prevent abuse.  
   - Securely store sensitive data (e.g., user ID, ratings) and validate inputs to prevent vulnerabilities (e.g., SQL injection).  


To implement an API and database design for calculating movie ratings (including the total rating and average rating), we'll need to structure both the **API** endpoints and the **Database schema** to efficiently store and compute ratings.

### **Database Design**

We'll design a simple relational database with two main tables:

1. **Movies Table**: Stores movie details (like name, genre, etc.).
2. **Ratings Table**: Stores individual user ratings for each movie.

### **Database Schema Design:**

#### **1. Movies Table**
This table will store information about each movie.

| Column Name     | Data Type  | Description                             |
|-----------------|------------|-----------------------------------------|
| movie_id        | INT        | Primary key, unique identifier for a movie |
| title           | VARCHAR(255)| Title of the movie                     |
| release_year    | INT        | Release year of the movie              |
| genre           | VARCHAR(100)| Genre of the movie                     |

#### **2. Ratings Table**
This table will store the individual ratings given by users for movies.

| Column Name     | Data Type  | Description                             |
|-----------------|------------|-----------------------------------------|
| rating_id       | INT        | Primary key, unique identifier for the rating |
| movie_id        | INT        | Foreign key referencing `movie_id` in the `Movies` table |
| user_id         | INT        | Unique identifier for the user who gave the rating |
| rating_value    | INT        | Rating between 1 and 5 (inclusive)      |
| rating_date     | DATETIME   | Timestamp of when the rating was given |

### **Database Relationships:**
- **Movies ↔ Ratings**: One movie can have many ratings, so there is a one-to-many relationship between the **Movies** table and the **Ratings** table.
- **Ratings ↔ Users**: Each rating is linked to a specific user, but we don't need a full users table in this case unless you want to add user details (like email, name, etc.). For now, we just assume `user_id` is sufficient for identification.

### **API Design**

Next, we’ll design the API endpoints to interact with this data and perform the necessary calculations for ratings. We can create the following main endpoints:

1. **POST /movies/{movie_id}/ratings** – Add a new rating for a movie.
2. **GET /movies/{movie_id}/ratings** – Get all ratings for a specific movie.
3. **GET /movies/{movie_id}/rating-summary** – Get the total and average ratings for a specific movie.

#### **API Endpoint 1: POST /movies/{movie_id}/ratings**

This endpoint allows users to submit a rating for a specific movie.

##### Request:
- **Method**: POST
- **URL**: `/movies/{movie_id}/ratings`
- **Body**:
```json
{
  "user_id": 123,
  "rating_value": 4
}
```
- **Parameters**:
  - `movie_id` (Path parameter): The ID of the movie being rated.
  - `user_id`: ID of the user submitting the rating.
  - `rating_value`: Rating value between 1 and 5.

##### Logic:
- Ensure that `rating_value` is between 1 and 5.
- Store the new rating in the **Ratings** table.
- Optionally, you can calculate and update the **average** and **total ratings** for the movie on the fly (in the API response or in the background).

##### Response:
- **Status Code**: `201 Created`
- **Body**:
```json
{
  "message": "Rating successfully submitted",
  "movie_id": 1,
  "user_id": 123,
  "rating_value": 4
}
```

#### **API Endpoint 2: GET /movies/{movie_id}/ratings**

This endpoint returns all individual ratings for a specific movie. This can be used to see the raw data or validate something like rating distribution.

##### Request:
- **Method**: GET
- **URL**: `/movies/{movie_id}/ratings`

##### Response:
- **Status Code**: `200 OK`
- **Body**:
```json
{
  "movie_id": 1,
  "ratings": [
    {
      "rating_id": 1,
      "user_id": 123,
      "rating_value": 4,
      "rating_date": "2025-01-11T10:00:00"
    },
    {
      "rating_id": 2,
      "user_id": 124,
      "rating_value": 3,
      "rating_date": "2025-01-12T10:00:00"
    },
    ...
  ]
}
```

#### **API Endpoint 3: GET /movies/{movie_id}/rating-summary**

This endpoint returns the summary of the ratings for a specific movie, including the **average rating** and **total rating**.

##### Request:
- **Method**: GET
- **URL**: `/movies/{movie_id}/rating-summary`

##### Logic:
- Calculate the **Total Rating** by summing up all ratings for the movie.
- Calculate the **Average Rating** by dividing the total rating by the number of ratings.
  - Use SQL queries to calculate these values on the fly:
    ```sql
    SELECT SUM(rating_value) AS total_rating, AVG(rating_value) AS average_rating
    FROM ratings
    WHERE movie_id = {movie_id};
    ```

##### Response:
- **Status Code**: `200 OK`
- **Body**:
```json
{
  "movie_id": 1,
  "total_rating": 14,
  "average_rating": 3.5
}
```

### **SQL Queries for Rating Calculations**

You can calculate the total and average ratings using SQL queries:

#### **1. Calculate Total Rating:**
```sql
SELECT SUM(rating_value) AS total_rating
FROM ratings
WHERE movie_id = {movie_id};
```

#### **2. Calculate Average Rating:**
```sql
SELECT AVG(rating_value) AS average_rating
FROM ratings
WHERE movie_id = {movie_id};
```

### **Optimizations & Additional Features**

1. **Caching**: To improve performance, especially for popular movies with a lot of ratings, you might consider caching the **total rating** and **average rating** in memory (e.g., Redis) so that these values don’t need to be recalculated with every request.
  
2. **Validation**: Ensure that the rating value is always between 1 and 5 before storing it in the database.

3. **Bulk Insert**: If users can rate multiple movies at once, you might want to create an endpoint for bulk rating submissions.

4. **Normalization**: If you’re planning on handling a large volume of ratings, you might want to consider **denormalization** (storing total and average ratings directly on the movie record and updating them as new ratings come in), though this can lead to data inconsistency in rare cases (like when ratings are deleted or updated).

### **API Example Flow**

1. **User submits a rating**:
   - The `POST /movies/{movie_id}/ratings` API is hit, and the rating is saved in the **Ratings** table.

2. **Client requests summary**:
   - The client hits the `GET /movies/{movie_id}/rating-summary` endpoint, which calculates and returns the **Total Rating** and **Average Rating**.

3. **Optional: Caching the result**:
   - You can cache the calculated ratings (total and average) to optimize future requests.

---

### **Summary**

- **Movies Table** stores movie details.
- **Ratings Table** stores individual ratings by users.
- **API Endpoints** for adding ratings, getting individual ratings, and retrieving rating summaries.
- **SQL Queries** to calculate **Total Rating** and **Average Rating**.

This design allows you to efficiently calculate and manage movie ratings, scale well as the number of users and ratings grows, and provide a robust API for interacting with the system. Let me know if you'd like more details on any part!

# Count-Min Sketch (CMS)

The **Count-Min Sketch (CMS)** algorithm is an efficient probabilistic data structure that allows for approximating frequency counts of items in a large dataset with significantly reduced memory usage. It is typically used in scenarios where you need to track the frequency of items (such as word counts in a text stream, or movie ratings in a large-scale system) but where exact counts are not necessary. Instead, CMS provides an approximate count within an error margin.

In the context of your **movie ratings system**, CMS can be utilized to efficiently calculate approximate **Total Rating** and **Average Rating** for movies, especially when you are dealing with a large number of ratings and want to reduce the memory and computational cost of exact aggregation.

### **How Count-Min Sketch Can Be Applied to Movie Rating System**

#### **Problem:**
- You want to keep track of the **Total Rating** and **Average Rating** for movies that receive a large number of ratings.
- A typical approach would involve summing all individual ratings, but this becomes inefficient as the number of ratings grows (both in terms of memory and processing time).
  
#### **Solution Using Count-Min Sketch:**
1. **Track Total Rating and Frequency of Ratings**: 
   - Use a **Count-Min Sketch** to keep approximate counts of the ratings between 1 to 5 for each movie.
   - Each **rating value** (1, 2, 3, 4, 5) will be treated as an item, and the sketch will store the count of how many times each rating has been given for a specific movie.
   
2. **Calculate Approximate Total Rating**: 
   - The **Total Rating** can be approximated by multiplying the rating value by its count in the CMS sketch and summing them across all rating values.
   
3. **Calculate Approximate Average Rating**: 
   - The **Average Rating** can be approximated by dividing the **Total Rating** by the total number of ratings, which can also be tracked in the CMS.

### **How Count-Min Sketch Works**

- The Count-Min Sketch algorithm uses multiple hash functions to map items (in this case, ratings) to counts in a **two-dimensional array (the sketch)**. Each cell in the array holds a count that represents the number of times an item has been seen, though with some error introduced due to hash collisions.
- **Error margin**: The error in the counts is bounded by a user-defined parameter (ε, δ) that determines the trade-off between memory usage and the accuracy of the count.

The general steps are:
1. **Update the Count-Min Sketch** when a rating is added (incrementing counts for the respective rating in the sketch).
2. **Query the sketch** to get an approximate count of how many times each rating (1-5) has been received for a movie.
3. Use the approximate counts to compute the approximate total and average ratings.

### **Advantages of Using CMS in Movie Ratings System**
- **Memory Efficiency**: CMS allows storing counts for millions of ratings in a small space, making it especially useful when dealing with large-scale data.
- **Approximate Computation**: Since CMS only provides approximate counts, the error margin is controlled and often small enough for practical purposes (i.e., approximating total and average ratings).

### **Count-Min Sketch Java Implementation**

Here’s a simple Java implementation of the **Count-Min Sketch** algorithm, and how it can be used in your movie ratings system to calculate the **Total Rating** and **Average Rating**.

#### **CountMinSketch Class Implementation**

```java
import java.util.Arrays;
import java.util.Random;

public class CountMinSketch {
    private int[][] table;
    private int width;
    private int depth;
    private Random[] hashFunctions;
    
    // Constructor
    public CountMinSketch(int width, int depth) {
        this.width = width;
        this.depth = depth;
        this.table = new int[depth][width];
        this.hashFunctions = new Random[depth];
        
        // Initialize hash functions
        for (int i = 0; i < depth; i++) {
            hashFunctions[i] = new Random(i);  // Use seed based on depth
        }
    }

    // Hash function to map an item to a position in the table
    private int hash(int item, int i) {
        return (Math.abs(hashFunctions[i].nextInt()) + item) % width;
    }

    // Update the Count-Min Sketch with a given item (rating)
    public void add(int item) {
        for (int i = 0; i < depth; i++) {
            int index = hash(item, i);
            table[i][index]++;
        }
    }

    // Query the Count-Min Sketch for the count of an item (rating)
    public int estimate(int item) {
        int min = Integer.MAX_VALUE;
        for (int i = 0; i < depth; i++) {
            int index = hash(item, i);
            min = Math.min(min, table[i][index]);
        }
        return min;
    }

    // Query the total rating (sum of ratings * their counts)
    public int totalRating() {
        int total = 0;
        for (int i = 1; i <= 5; i++) { // Ratings 1 to 5
            total += i * estimate(i);
        }
        return total;
    }

    // Query the total number of ratings
    public int totalRatingsCount() {
        int count = 0;
        for (int i = 1; i <= 5; i++) { // Ratings 1 to 5
            count += estimate(i);
        }
        return count;
    }
}
```

### **How to Use the CountMinSketch in the Movie Rating System**

#### **Step 1: Create an instance of the CountMinSketch for each movie**

Each movie will have a separate `CountMinSketch` object to store the ratings. We will configure the sketch with an appropriate width and depth, balancing memory usage and error tolerance.

#### **Step 2: Update the sketch when a user submits a rating**

Each time a user submits a rating for a movie, the corresponding **CountMinSketch** for that movie will be updated.

```java
public class MovieRatingSystem {
    private static final int WIDTH = 1000;  // Width of the Count-Min Sketch
    private static final int DEPTH = 5;     // Depth of the Count-Min Sketch

    // A map to hold the CountMinSketch for each movie
    private Map<Integer, CountMinSketch> movieSketches = new HashMap<>();

    // Add a rating for a movie
    public void addRating(int movieId, int ratingValue) {
        // If this movie doesn't have a sketch, create one
        movieSketches.putIfAbsent(movieId, new CountMinSketch(WIDTH, DEPTH));

        // Get the CountMinSketch for the movie
        CountMinSketch cms = movieSketches.get(movieId);

        // Update the CMS with the rating value
        cms.add(ratingValue);
    }

    // Get the total and average rating for a movie
    public String getRatingSummary(int movieId) {
        CountMinSketch cms = movieSketches.get(movieId);
        if (cms == null) {
            return "No ratings available for this movie.";
        }

        // Calculate approximate total rating
        int totalRating = cms.totalRating();

        // Calculate approximate total number of ratings
        int totalCount = cms.totalRatingsCount();

        // Calculate approximate average rating
        double averageRating = (totalCount > 0) ? (double) totalRating / totalCount : 0;

        return "Total Rating: " + totalRating + ", Average Rating: " + String.format("%.2f", averageRating);
    }
}
```

### **Step 3: Example Usage**

```java
public class Main {
    public static void main(String[] args) {
        MovieRatingSystem ratingSystem = new MovieRatingSystem();

        // Simulating user ratings for movie 1
        ratingSystem.addRating(1, 4);
        ratingSystem.addRating(1, 5);
        ratingSystem.addRating(1, 3);
        ratingSystem.addRating(1, 5);

        // Simulating user ratings for movie 2
        ratingSystem.addRating(2, 3);
        ratingSystem.addRating(2, 2);
        ratingSystem.addRating(2, 4);

        // Getting the summary for movie 1
        System.out.println(ratingSystem.getRatingSummary(1)); // Approximate total and average rating for movie 1

        // Getting the summary for movie 2
        System.out.println(ratingSystem.getRatingSummary(2)); // Approximate total and average rating for movie 2
    }
}
```

### **Explanation:**
1. **CountMinSketch Class**: The `CountMinSketch` class implements the Count-Min Sketch algorithm. It stores counts in a 2D table and supports updating the counts and querying for the estimated count of each rating value.
2. **MovieRatingSystem Class**: The `MovieRatingSystem` class manages movie ratings using a `CountMinSketch` for each movie. When a rating is submitted, it updates the sketch. The system then provides approximate totals and averages.
3. **Main Class**: The main class demonstrates adding ratings and querying the system for rating summaries.

### **Pros of Using Count-Min Sketch in This Scenario:**
- **Memory Efficiency**: Count-Min Sketch reduces memory usage significantly, especially when there are millions of ratings.
- **Fast Updates and Queries**: Rating submissions and queries are fast due to the constant-time complexity for updates and queries, regardless of the number of ratings.
- **Approximation**: While the counts are approximate, for many use cases (e.g.,
