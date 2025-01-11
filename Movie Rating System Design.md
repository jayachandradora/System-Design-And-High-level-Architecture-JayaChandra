
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
