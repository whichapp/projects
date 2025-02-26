# Senior Software Engineer Task (Revised & Expanded)

## 1. Product Overview

You are creating a **massively interactive polling platform** for mobile and web. The platform provides a vertical feed of polls, and each poll has:
- A text title
- Multiple-choice options
- One or more tags (e.g., `sports`, `news`, `entertainment`, etc.)

### Core User Interactions

1. **Vote** on a poll by selecting exactly one of the options.
2. **Skip** a poll if it’s not interesting.
3. Each user **never** sees the same poll twice (once voted or skipped, it’s removed from their feed).
4. **Filter** the feed by tag or search criteria.
5. **Daily Vote Limit**: A user can only vote on **up to 100 polls per day** (skips are unlimited).

### System Scale & Performance

The platform has:
- A **large user base** that can generate high read/write concurrency.
- A **rapidly growing** dataset of polls, votes, and skips.
- Requirements for **fast** feed loading (read operations) and **instant** vote/skip feedback (write operations).

---

## 2. Core Requirements

Below are the core endpoints you must design and implement. While these are standard CRUD-like patterns, keep in mind the high scale and concurrency environment.

---

### 2.1 Create a Poll

- **Endpoint**: `POST /polls`
- **Request Body (JSON)**:
    ```json
    {
        "title": "Your favorite programming language?",
        "options": ["Go", "Python", "Rust"],
        "tags": ["programming", "favorites"]
    }
    ```
- **Sample cURL**:
    ```bash
    curl -X POST \
        -H "Content-Type: application/json" \
        -d '{
                "title": "Your favorite programming language?",
                "options": ["Go", "Python", "Rust"],
                "tags": ["programming", "favorites"]
            }' \
        http://localhost:8080/polls
    ```
- **Expected Response**:  
  - **HTTP Code**: `201 Created`  
  - No response body is required.

---

### 2.2 Retrieve Polls for Feed

- **Endpoint**: `GET /polls`
- **Query Parameters**:
    - `tag` (optional): String value to filter by tag
    - `page` (optional): Pagination page number
    - `limit` (optional): Number of polls per page
    - `userId` (required): The ID of the user making the request
- **Behavior**:
    - Returns the most recent polls available.
    - Excludes polls the user has voted on or skipped.
    - Must handle **fast** reads, even with large datasets.
- **Sample cURL**:
    ```bash
    curl -X GET "http://localhost:8080/polls?tag=programming&page=1&limit=10&userId=999"
    ```
- **Sample Response (JSON)**:
    ```json
    [
        {
            "id": 123,
            "title": "Your favorite programming language?",
            "options": ["Go", "Python", "Rust"],
            "tags": ["programming", "favorites"],
            "createdAt": "2025-01-01T12:00:00Z"
        },
        ...
    ]
    ```

---

### 2.3 Vote on a Poll

- **Endpoint**: `POST /polls/{id}/vote`
- **Request Body (JSON)**:
    ```json
    {
        "userId": 999,
        "optionIndex": 1
    }
    ```
    - `userId`: The user casting the vote
    - `optionIndex`: Index of the chosen option (e.g., 0, 1, 2…)
- **Behavior**:
    - Records a vote for the given poll and user.
    - Enforces the **daily vote limit**: a user can only vote on up to 100 polls per day.
- **Sample cURL**:
    ```bash
    curl -X POST \
        -H "Content-Type: application/json" \
        -d '{
                "userId": 999,
                "optionIndex": 1
            }' \
        http://localhost:8080/polls/123/vote
    ```
- **Expected Response**:  
  - **HTTP Code**: `200 OK` (or `204 No Content`)  
  - No response body is required.

---

### 2.4 Skip a Poll

- **Endpoint**: `POST /polls/{id}/skip`
- **Request Body (JSON)**:
    ```json
    {
        "userId": 999
    }
    ```
- **Sample cURL**:
    ```bash
    curl -X POST \
        -H "Content-Type: application/json" \
        -d '{
                "userId": 999
            }' \
        http://localhost:8080/polls/123/skip
    ```
- **Expected Response**:  
  - **HTTP Code**: `200 OK` (or `204 No Content`)  
  - No response body is required.

---

### 2.5 Retrieve Poll Statistics

- **Endpoint**: `GET /polls/{id}/stats`
- **Behavior**:
    - Returns aggregated vote counts for each option.
    - Must remain performant even if the poll has a large number of votes.
- **Sample cURL**:
    ```bash
    curl -X GET http://localhost:8080/polls/123/stats
    ```
- **Sample Response (JSON)**:
    ```json
    {
        "pollId": 123,
        "votes": [
            { "option": "Go", "count": 10 },
            { "option": "Python", "count": 25 },
            { "option": "Rust", "count": 7 }
        ]
    }
    ```

---

## 3. Additional Technical Requirements

### 3.1 Massive Usage & Persistent Storage

- Choose any database(s). You should handle **large** amounts of data (polls, votes, skips) with good read/write performance.

### 3.2 Caching & Speed

- Integrate a **caching layer** (in-memory or distributed) for high-demand data, like popular polls or aggregated stats.
- Ensure you handle **cache invalidation** or updates effectively (e.g., when new votes come in).

### 3.3 Observability & Instrumentation

- Expose system metrics on a `/metrics` endpoint (Prometheus or similar).
- Track request counts, latencies, DB query times, cache hits/misses, etc. 

### 3.4 Performance Testing & Degradation Report

- Provide a way to **load test** or **stress test** the system. This can be a script using `k6`, `wrk`, or a Go benchmarking tool.
- Capture results that show **how response times degrade** (for “Fetch Polls”, “Vote”, “Skip”) as **Requests Per Second (RPS)** increases.
- (Optional but encouraged) Provide a **plot** or a table illustrating response times vs. concurrency/RPS.

### 3.5 High Availability & Real-World Constraints

- Consider concurrency spikes (e.g., if a poll goes viral, it might receive thousands of votes in seconds).
- Describe or implement any techniques you use to ensure consistency (e.g., transactions, optimistic locking, etc.).

### 3.6 Testing

- **Unit Tests**: Cover core business logic, such as:
    - Poll creation
    - Enforcing daily vote limits
    - Ensuring feed excludes previously voted/skipped polls
- **Integration / End-to-End Tests**: Validate that all components (database, cache, etc.) work together as expected.
    - Example scenario: A user votes on 5 polls in a row, ensure daily limit logic, skip logic, etc., all function end to end.

### 3.7 Dockerization

- Provide a `docker-compose.yml` to spin up:
    - Your service
    - The database
    - Caching layer
    - Prometheus (or other observability tool)
- A single command, e.g. `docker-compose up --build -d`, should bring the system up.

### 3.8 Documentation

- Include a `README.md` with:
    - **Architecture** overview (including database schema, caching strategy, concurrency model).
    - **Usage instructions** (how to build/run the service, run tests, etc.).
    - **API endpoints** with sample `curl` commands.
    - **Performance notes** (load test results and potential bottlenecks).
    - **Assumptions** and **trade-offs** you made.

### 3.9 Future Growth

- Briefly describe how your system might evolve with:
    - 10x or 100x more users
    - More complex poll types (multi-stage polls, real-time leaderboards, scheduling, or any other wild idea a **product manager** can have in the future!)
- Highlight **areas for refactoring** or **technical debt** that might need revisiting.

---

## 4. Bonus

- **Event Streaming**: Optionally integrate a message queue (e.g., Kafka, RabbitMQ) to publish poll or vote events. Outline how this benefits analytics, real-time updates, or decoupled processing.

---

## 5. Prioritization & Evaluation

While the above requirements cover **many** aspects (data modeling, caching, concurrency, observability, etc.), **you are not expected** to finish every single detail **perfectly**. Instead, you will be evaluated on:

- **Approach & Reasoning**: How you tackle core challenges, make trade-offs, and prioritize features.
- **Quality & Scalability**: How robustly you handle concurrency, data growth, and performance.
- **Domain Modelling**: How effective does your code capture the core and supporting sub-domains of the product and its logic.
- **Prioritization**: Which features you focus on first or refine (e.g., caching critical paths vs. less critical endpoints).

**It’s acceptable** if some items remain partially implemented or are described theoretically—as long as your approach is clear, well-reasoned, and demonstrates strong software engineering skills.

---

# Deliverables

1. **Source Code** for your backend service (Go preferred) in a Github repository.
2. **Database Schema** (migrations or DDL statements) and any caching setup.
3. **Performance Testing** scripts/results, ideally showing **response-time degradation** under various RPS.
4. **Docker Compose** setup.
5. **Unit Tests** and **End-to-End Tests** illustrating both correctness and readiness for scale.
6. **Documentation** (`README.md` + instructions + architecture + future growth).

We look forward to reviewing your submission!
