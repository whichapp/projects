
# Senior Software Engineer Task

## Product Overview

Imagine a mobile/web application where users can scroll through a feed of interactive polls. Each poll consists of a text title, multiple selectable options, and relevant tags. As users scroll:
- They can **vote** on a poll by selecting one of the available options.
- They can **skip** a poll if it doesn’t interest them.
- Once a user votes on or skips a poll, it will no longer appear in their feed.
- The feed is continuously updated with the most recent polls and can be filtered based on tags.

The goal is to simulate the core API functionalities needed for such a vertical scrolling feed while ensuring the solution is scalable, performs well under load, and provides observability metrics.

---

## Task Requirements

### Core Functionality

You are to design and implement a backend service (preferably in **Go**, that provides the following endpoints:

1. **Create Text Poll**  
   - **Endpoint**: `POST /polls`  
   - **Payload Example**:
     ```json
     {
       "title": "Your favorite programming language?",
       "options": ["Go", "PHP", "JavaScript"],
       "tags": ["programming", "favorites"]
     }
     ```
   - **Response Example**:
     ```json
     {
       "id": 123,
       "title": "Your favorite programming language?",
       "options": ["Go", "PHP", "JavaScript"],
       "tags": ["programming", "favorites"]
     }
     ```

2. **Retrieve Polls for Feed**  
   - **Endpoint**: `GET /polls`  
   - **Query Parameters**: 
     - `tag` (for filtering by tag)
     - `page` and `limit` (for pagination)
   - **Behavior**: 
     - Returns polls ordered by recency.
     - Excludes any polls that the user (identified by a `userId` parameter) has already voted on or skipped.
   - **Sample `curl` request**:
     ```bash
     curl -X GET "http://localhost:8080/polls?tag=programming&page=1&limit=10&userId=999"
     ```
   - **Response Example**:
     ```json
     [
       {
         "id": 123,
         "title": "Your favorite programming language?",
         "options": ["Go", "PHP", "JavaScript"],
         "tags": ["programming", "favorites"],
         "createdAt": "2025-01-01T12:00:00Z"
       },
       ...
     ]
     ```

3. **Vote on a Poll**  
   - **Endpoint**: `POST /polls/{id}/vote`  
   - **Payload Example**:
     ```json
     {
       "optionIndex": 1,
       "userId": 999
     }
     ```
   - **Response Example**:
     ```json
     {
       "message": "Vote recorded successfully",
       "pollId": 123,
       "optionIndex": 1
     }
     ```

4. **Skip a Poll**  
   - **Endpoint**: `POST /polls/{id}/skip`  
   - **Payload Example**:
     ```json
     {
       "userId": 999
     }
     ```
   - **Response Example**:
     ```json
     {
       "message": "Poll skipped successfully",
       "pollId": 123
     }
     ```

5. **Get Poll Statistics**  
   - **Endpoint**: `GET /polls/{id}/stats`  
   - **Response Example**:
     ```json
     {
       "pollId": 123,
       "votes": [
         { "option": "Go", "count": 10 },
         { "option": "PHP", "count": 5 },
         { "option": "JavaScript", "count": 7 }
       ]
     }
     ```

### Additional Technical Requirements

- **Data Persistence & Caching**:  
  - Use a database of your choice (e.g., MySQL, PostgreSQL, etc.) and a caching mechanism (e.g., Redis) to support fast retrieval of feed data.
  
- **Observability**:  
  - Instrument the service with Prometheus metrics. Expose a `/metrics` endpoint that includes request counts, latencies, and any custom metrics (e.g., total polls created, total votes, etc.).

- **Benchmarking & Performance**:  
  - Provide a method or script (using tools like `k6`, `wrk`, or Go benchmarks) to evaluate the performance of key endpoints (e.g., poll creation, feed retrieval, voting).

- **Scalability & Architecture**:  
  - Clearly describe in your documentation how your codebase is designed to scale and handle increased load.
  - Explain how you modeled the product vision and requirements in your code.
  - Describe any architectural decisions or patterns used (e.g., microservices, caching strategy, database schema design, etc.).

- **Testing**:  
  - Include unit tests (and integration tests if possible) to ensure correctness of functionality, such as:
    - Creating polls
    - Retrieving polls without showing already voted/skipped items
    - Voting and skipping behavior

- **Dockerization**:  
  - Provide a single `docker-compose.yml` that sets up:
    - Your service
    - The database
    - The caching layer
    - Prometheus (or the observability stack)
  - The entire stack should be easy to run, for example:
    ```bash
    docker-compose up --build -d
    ```

- **Documentation**:  
  - Supply a `README.md` that includes:
    - An architectural overview.
    - Instructions on how to run the service, tests, and benchmarks.
    - Descriptions of the API endpoints along with `curl` examples.
    - Explanations of how scalability is addressed.
    - Any assumptions made during development.

- **Future Modifications & Technical Debt**:  
  - Provide a section in your documentation discussing:
    - Possible future modifications and updates to the system.
    - Any technical debt incurred due to time or scope constraints.
    - How the design might evolve with increased usage or new features (e.g., additional poll types, more advanced filtering, real-time updates).

---

## Bonus

- **Message Queue Integration**:  
  - Optionally, integrate a message queue (e.g., Kafka or RabbitMQ) to publish events such as “poll created” or “vote recorded”. Explain how this would fit into the overall architecture and what benefits it would provide.

---



**Your submission should demonstrate not only your coding abilities but also your approach to system design, scalability, and future-proofing. We look forward to seeing how you model the product vision into robust, maintainable code!**
