# Movie Review and Rating 

To design a system for movie reviews and ratings based on information gathered from three different sources, and then provide functionalities such as real-time calculations, top movies list, and individual movie ratings, we will break the design into key components. The solution will focus on:

- **Data Collection**: Collecting movie data from three websites
- **Backend API Design**: Exposing an API to serve user queries
- **Real-time Calculations**: Handling real-time review and rating calculations
- **Top 10 Movies**: Displaying the top 10 movies based on ratings
- **Scalability Considerations**: Making the system capable of handling 10 million daily users and 10,000 movies

### System Design Overview

---

### 1. **Data Collection from 3 Websites**

We need to gather movie data from three sources. This could be done using web scraping or APIs if the sites provide them.

- **Web Scraping**: We can use libraries like **JSoup** (for Java) to scrape movie data (titles, reviews, ratings) periodically.
- **API Integration**: If the websites provide APIs, we can integrate with them directly.

The data we collect will include:
- Movie Title
- Movie ID
- User Reviews
- Ratings (possibly on a scale of 1-10 or 1-5)
- Number of Reviews
- Release Date
- Other Metadata (actors, genres, etc.)

For real-time updates, data synchronization will be important. We can periodically update our database with new reviews or ratings (e.g., every minute or hour).

---

### 2. **Backend API Design**

We will design a RESTful API to interact with the system, supporting both read and write operations:

#### API Endpoints:
1. **GET /movies/top**  
   *Fetches the top 10 movies based on their ratings and reviews.*
   - **Response:**
     ```json
     [
       {
         "movie_id": 1,
         "title": "Movie A",
         "average_rating": 8.7,
         "review_count": 1500
       },
       ...
     ]
     ```

2. **GET /movies/{movie_id}**  
   *Fetches a specific movie's details including individual ratings and reviews.*
   - **Response:**
     ```json
     {
       "movie_id": 1,
       "title": "Movie A",
       "average_rating": 8.7,
       "reviews": [
         {
           "user_id": 101,
           "rating": 9,
           "review": "Great movie!"
         },
         ...
       ]
     }
     ```

3. **GET /movies/reviews**  
   *Fetch movies based on user-provided filters (e.g., movies in a certain genre with average ratings above a certain threshold).*
   - **Request:**
     ```json
     {
       "genre": "Action",
       "min_rating": 8
     }
     ```
   - **Response:**
     ```json
     [
       {
         "movie_id": 2,
         "title": "Movie B",
         "average_rating": 8.5,
         "review_count": 500
       },
       ...
     ]
     ```

4. **POST /movies/{movie_id}/reviews**  
   *Allows users to post their review and rating for a specific movie.*
   - **Request:**
     ```json
     {
       "user_id": 102,
       "rating": 7,
       "review": "Not bad, but could have been better."
     }
     ```
   - **Response:**
     ```json
     {
       "status": "success",
       "message": "Review submitted successfully"
     }
     ```

#### Database Design

We need a database to store movie data, user reviews, and ratings. Here’s an example schema:

- **Movies Table**: Stores metadata for each movie.
  - `movie_id` (Primary Key)
  - `title`
  - `release_date`
  - `genre`
  - `average_rating`
  - `review_count`

- **Reviews Table**: Stores individual user reviews for each movie.
  - `review_id` (Primary Key)
  - `movie_id` (Foreign Key)
  - `user_id`
  - `rating`
  - `review`
  - `timestamp`

- **Users Table**: Stores user information (if needed for registration and to prevent spamming reviews).
  - `user_id` (Primary Key)
  - `username`
  - `email`

#### Real-Time Rating Calculation

To calculate real-time ratings efficiently:
- **Caching**: Use a distributed caching system like **Redis** to cache the average rating of movies. This avoids recalculating the rating every time a query is made.
- **Incremental Updates**: When a new review is posted, we can update the cache incrementally without needing to recompute the entire dataset.
  - Example: If a new review for a movie comes in with a rating of 8, we update the average rating like this:

    ```
    new_average_rating = ((current_average_rating * review_count) + new_rating) / (review_count + 1)
    ```

- **Batch Updates**: Reviews could be processed in batches during periods of low activity (e.g., overnight), so the system doesn't become overloaded during peak times.

---

### 3. **Handling Scalability**

Since we expect 10 million users per day and 10,000 movies, we need to design the system with scalability in mind.

#### Key Considerations:
- **Load Balancing**: Use **load balancers** (e.g., AWS ELB, Nginx) to distribute incoming traffic to multiple application servers.
- **Database Sharding**: Shard the database by movie ID or user ID to distribute load and prevent any one server from becoming a bottleneck.
- **Asynchronous Processing**: Use queues (e.g., **Kafka** or **RabbitMQ**) to process time-consuming tasks like recalculating ratings, sending notifications, etc.
- **Distributed Caching**: Use a distributed cache (e.g., **Redis** or **Memcached**) to store movie ratings and reduce database load.
- **CDN (Content Delivery Network)**: Use a CDN for static content (e.g., images, movie posters, etc.) to speed up access for users.
- **Microservices Architecture**: If needed, split the application into microservices for movies, reviews, and user management to scale independently.

---

### 4. **Java API Implementation Example**

Here is an outline of how you can implement the API using **Spring Boot** (Java framework).

#### Setting Up Spring Boot
```bash
spring init --dependencies=web,data-jpa,redis spring-boot-movie-rating-api
```

#### Controller Class
```java
@RestController
@RequestMapping("/movies")
public class MovieController {

    @Autowired
    private MovieService movieService;

    @GetMapping("/top")
    public ResponseEntity<List<MovieDTO>> getTopMovies() {
        List<MovieDTO> topMovies = movieService.getTopMovies();
        return ResponseEntity.ok(topMovies);
    }

    @GetMapping("/{movieId}")
    public ResponseEntity<MovieDTO> getMovie(@PathVariable Long movieId) {
        MovieDTO movie = movieService.getMovieDetails(movieId);
        return ResponseEntity.ok(movie);
    }

    @PostMapping("/{movieId}/reviews")
    public ResponseEntity<String> postReview(@PathVariable Long movieId, @RequestBody ReviewRequest reviewRequest) {
        movieService.submitReview(movieId, reviewRequest);
        return ResponseEntity.status(HttpStatus.CREATED).body("Review submitted successfully");
    }
}
```

#### Service Class (Business Logic)
```java
@Service
public class MovieService {

    @Autowired
    private MovieRepository movieRepository;

    @Autowired
    private ReviewRepository reviewRepository;

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public List<MovieDTO> getTopMovies() {
        // Check if data is available in cache
        List<MovieDTO> topMovies = (List<MovieDTO>) redisTemplate.opsForValue().get("topMovies");
        if (topMovies == null) {
            // If not, fetch from DB, calculate and store in cache
            topMovies = movieRepository.findTopMovies();
            redisTemplate.opsForValue().set("topMovies", topMovies);
        }
        return topMovies;
    }

    public MovieDTO getMovieDetails(Long movieId) {
        Movie movie = movieRepository.findById(movieId).orElseThrow(() -> new MovieNotFoundException(movieId));
        List<Review> reviews = reviewRepository.findByMovieId(movieId);
        return new MovieDTO(movie, reviews);
    }

    public void submitReview(Long movieId, ReviewRequest reviewRequest) {
        // Save the review in the DB
        Review review = new Review(movieId, reviewRequest.getUserId(), reviewRequest.getRating(), reviewRequest.getReview());
        reviewRepository.save(review);

        // Update movie's average rating
        movieRepository.updateMovieRating(movieId, reviewRequest.getRating());
        // Clear cache for this movie
        redisTemplate.delete("movie_" + movieId);
    }
}
```

#### Database Repositories
```java
public interface MovieRepository extends JpaRepository<Movie, Long> {
    @Query("SELECT m FROM Movie m ORDER BY m.averageRating DESC")
    List<MovieDTO> findTopMovies();

    @Modifying
    @Query("UPDATE Movie m SET m.averageRating = :rating WHERE m.id = :movieId")
    void updateMovieRating(Long movieId, Double rating);
}

public interface ReviewRepository extends JpaRepository<Review, Long> {
    List<Review> findByMovieId(Long movieId);
}
```

#### Redis

 Cache Setup
```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        return template;
    }
}
```

---

### 5. **Performance and Load Testing**

Before going live, ensure to:
- Perform load testing using tools like **JMeter** or **Gatling** to simulate user traffic.
- Optimize database queries and caching mechanisms for fast read and write operations.

---

This design handles high traffic (10 million users), large datasets (10,000 movies), and real-time movie reviews and ratings. The system ensures scalability, reliability, and efficiency by utilizing caching, distributed processing, and a robust API architecture.
