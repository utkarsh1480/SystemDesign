# System Design – The Story of Every Application

## 1. Introduction: How Does an Application Work?

Every application follows a basic communication flow:

1. A user sends a request.
2. The application processes the request.
3. The application communicates with the database if required.
4. The application generates a response.
5. The response is sent back to the user.

**Basic architecture:**

```text
       USER
         |
         | Request
         v
  APPLICATION
  (Code + Database)
         |
         | Response
         v
       USER
```

Initially, we can run the entire application on a single machine. However, as the number of users increases, several problems start appearing.

Let's understand this journey step by step.

---

## 2. Stage 0 – Everything on One Machine

In the beginning, we run everything on a single computer:

- Application code (Backend)
- Database (DB)
- CPU
- RAM
- Disk Storage

**Architecture:**

```text
             USER
               |
          Request/Response
               |
               v
       +------------------+
       |     MACHINE      |
       |                  |
       |  Application     |
       |      Code        |
       |                  |
       |    Database      |
       |                  |
       |   CPU / RAM      |
       |   Disk Storage   |
       +------------------+
```

### Problems with this architecture

**1. Bottleneck**

A bottleneck occurs when one component cannot handle the workload efficiently.

For example:
- 100 users access the application: Everything works fine.
- 1,000 users access the application: Response time increases.
- 10,000 users access the application: The system may become extremely slow or crash.

The machine has limited CPU, RAM and disk resources.

**2. Single Point of Failure (SPOF)**

SPOF means a single component whose failure can bring down the entire application.

For example:

If the machine crashes, both the backend and database become unavailable.

**3. Resource Competition**

The application code and database compete for the same CPU, RAM and disk resources.

For example:
- Database queries consume RAM and CPU.
- Backend requests also consume RAM and CPU.
- Heavy database operations can affect backend performance.

**4. Limited Scalability**

We cannot efficiently handle unlimited users using one machine because hardware resources are limited.

---

## 3. Stage 1 – Separating Application and Database

To solve some of the problems of Stage 0, we separate the application code and database onto different machines.

This introduces the concept of **Separation of Responsibilities**.

### Architecture

```text
              USER
                |
          Request/Response
                |
                v
       +------------------+
       |    MACHINE 1     |
       |                  |
       |  Application     |
       |      Code        |
       |                  |
       |   CPU / RAM      |
       |   Disk           |
       +------------------+
                |
                | Database Query
                | Database Response
                v
       +------------------+
       |    MACHINE 2     |
       |                  |
       |    Database      |
       |                  |
       |   CPU / RAM      |
       |   Disk           |
       +------------------+
```

### Why do we separate them?

1. **Separate Responsibilities:** The application handles business logic, while the database manages data storage and retrieval.

2. **Independent Resource Management:** We can allocate CPU and RAM according to the requirements of each machine.

3. **Independent Scaling:** We can upgrade the application server without necessarily upgrading the database server, and vice versa.

4. **Better Performance Isolation:** Heavy database operations are less likely to directly compete with application processing for the same machine resources.

### Important Concept: Two-Tier Architecture

This setup is commonly called a two-tier architecture:

- Tier 1: Application Server
- Tier 2: Database Server

**Important:** Separating the database does not completely eliminate bottlenecks or single points of failure. Both machines can still become overloaded or fail independently.

---

## 4. Stage 2 – Increasing Users and Server Capacity

Now imagine our application becomes popular.

Initially, one application server might have:

- 8-core CPU
- 16 GB RAM
- 128 GB Storage

As the number of users increases, the application server starts consuming more resources.

For example:

- More requests require more CPU processing.
- More concurrent requests require more RAM.
- More operations can increase disk and network usage.

Eventually, the server becomes slow or may crash.

This is the situation described in the original notes:

**As users increase, the laptop/server starts choking and becomes slow.**

### What happens?

```text
        MORE USERS
             |
             v
      APPLICATION SERVER
             |
       CPU / RAM usage
          increases
             |
             v
       HIGH LOAD
             |
             v
    SLOW RESPONSE / CRASH
```

We now need a solution to handle increasing traffic.

There are two important scaling approaches.

---

## 5. Important Concept – Vertical Scaling (Scale Up)

Vertical scaling means increasing the hardware capacity of an existing machine.

For example:

| Resource | Before | After |
|---|---|---|
| CPU | 8 cores | 16 cores |
| RAM | 16 GB | 64 GB |
| Storage | 128 GB | 1 TB |

We upgrade the existing server instead of adding more servers.

### Architecture

```text
          USER
            |
            v
    +------------------+
    |  POWERFUL SERVER |
    |                  |
    |  Application     |
    |  CPU: 16 Cores   |
    |  RAM: 64 GB      |
    |  Disk: 1 TB      |
    +------------------+
```

### Advantages

- Easy to implement.
- No major architectural changes required.
- Suitable for applications with moderate traffic.

### Disadvantages

- Hardware upgrades have limits.
- Powerful machines can become expensive.
- The machine can still be a Single Point of Failure.

---

## 6. Horizontal Scaling (Scale Out)

Vertical scaling has limitations.

Instead of making one server more powerful, we can add multiple application servers.

This is called horizontal scaling.

### Architecture

```text
                 USERS
                   |
                   v
            +-------------+
            |    LOAD     |
            |  BALANCER   |
            +-------------+
              /    |    \
             /     |     \
            v      v      v
        +------+ +------+ +------+
        | App  | | App  | | App  |
        |Server| |Server| |Server|
        |  1   | |  2   | |  3   |
        +------+ +------+ +------+
             \     |     /
              \    |    /
                   v
             +-----------+
             | DATABASE  |
             +-----------+
```

### What is a Load Balancer?

A load balancer distributes incoming requests across multiple application servers.

For example:

Suppose 3,000 users send requests.

Instead of sending every request to one server:

- Server 1 handles approximately 1,000 requests.
- Server 2 handles approximately 1,000 requests.
- Server 3 handles approximately 1,000 requests.

This is only an illustrative distribution; actual traffic depends on the balancing algorithm and request workload.

### Advantages

1. Handles more concurrent users.
2. Reduces the load on individual application servers.
3. Makes it possible to add or remove servers as traffic changes.
4. Improves availability when combined with health checks and redundant infrastructure.

### Important Point

Horizontal scaling works best when application servers are stateless.

**Stateless Application:** An application server does not depend on its local memory to remember a user's session between requests.

For example, user session information can be stored in Redis or managed through tokens rather than being stored only in one server's memory.

---

## 7. Important System Design Problems

### A. Bottleneck

A bottleneck is a component that limits the overall performance of a system.

Examples:
- Slow database queries.
- Insufficient CPU.
- Low RAM.
- Network limitations.
- A slow external API.

### B. Single Point of Failure (SPOF)

A component whose failure can make the entire system unavailable.

Example:

If an application has only one database server and that database goes down, users may not be able to access their data.

**Solution:** Redundancy, replication and failover mechanisms.

### C. Redundancy

Redundancy means keeping additional resources or components so that the system can continue working if one component fails.

Example:

```text
             APPLICATION
                  |
          +-------+-------+
          |               |
          v               v
      DATABASE 1      DATABASE 2
       (Primary)      (Replica)
```

If the primary database fails, a properly configured replica can be promoted to become the new primary.

Redundancy improves fault tolerance and availability.

**Note:** Redundancy does not mean unnecessary repetition of code. In system design, it means providing backup capacity or duplicate components.

---

## 8. Important Terms to Remember

| Term | Meaning |
|---|---|
| Scalability | Ability to handle increasing workload |
| Vertical Scaling | Increasing the resources of one machine |
| Horizontal Scaling | Adding more machines |
| Bottleneck | Component limiting system performance |
| SPOF | Single Point of Failure |
| Redundancy | Additional components for fault tolerance |
| Load Balancer | Distributes incoming traffic across servers |
| Availability | How often a system remains operational |
| Fault Tolerance | Ability to continue working despite failures |
| Stateless | Server does not rely on its local memory for persistent session state |

---

## 9. Complete Journey of an Application

Let's summarize the evolution:

**Stage 0 – Single Machine**

```text
User → Application Code + Database
```

Everything runs on one machine.

Problem: Resource competition, bottlenecks and SPOF.

**Stage 1 – Separate Application and Database**

```text
User → Application Server → Database Server
```

Responsibilities and resources are separated.

Problem: Application server or database can still become overloaded.

**Stage 2 – Increasing Server Capacity**

```text
User → Powerful Application Server → Database
```

We increase CPU, RAM and storage.

Approach: Vertical Scaling.

Problem: Hardware has limitations and the server remains a possible SPOF.

**Stage 3 – Multiple Application Servers**

```text
             Users
               |
          Load Balancer
           /    |    \
          v     v     v
        App1   App2   App3
           \    |    /
            Database
```

Approach: Horizontal Scaling.

Benefit: Better traffic handling and scalability.

---

## 10. Interview Questions

1. What is system design?
2. What is a bottleneck?
3. What is a Single Point of Failure?
4. Why do we separate application servers and database servers?
5. What is the difference between vertical and horizontal scaling?
6. What is a load balancer?
7. What is redundancy?
8. What is fault tolerance?
9. Why should application servers be stateless?
10. Does adding more application servers automatically solve database bottlenecks?

### Quick Revision

- Scale Up = Increase the power of an existing machine.
- Scale Out = Add more machines.
- Load Balancer = Distribute incoming traffic.
- Redundancy = Keep backup components.
- Bottleneck = Performance-limiting component.
- SPOF = One failure can bring down the system.

**Golden Rule of System Design:**

As the number of users increases, we need to evolve our architecture so that the application remains scalable, available, reliable and performant.


# System Design – The Story of Every Application
## Part 2: Scaling, Database Optimization and Distributed Architecture

---

# Stage 3 – Horizontal Scaling + Load Balancer

## Problem: Vertical Scaling Has Limitations

In the previous stage, we used vertical scaling to increase server capacity.

For example:

- 8-core CPU → 16-core CPU
- 16 GB RAM → 32 GB RAM
- 128 GB storage → 1 TB storage

But vertical scaling has two major problems.

### 1. Cost increases

As we upgrade hardware, the cost increases.

For example (illustrative prices):

- 16 GB RAM server: ₹20,000
- 32 GB RAM server: ₹60,000

The cost may increase disproportionately as we move towards high-end hardware.

### 2. Single Point of Failure

Even after upgrading the server, we still have only one application server.

If that server crashes, the entire application becomes unavailable.

## Solution: Horizontal Scaling

Instead of increasing the power of one machine, we add multiple application servers.

### Architecture

```text
               CLIENT
                  |
                  v
           +--------------+
           | LOAD BALANCER|
           +--------------+
             /     |     \
            /      |      \
           v       v       v
       +------+ +------+ +------+
       | App  | | App  | | App  |
       |Server| |Server| |Server|
       |  1   | |  2   | |  3   |
       +------+ +------+ +------+
            \     |     /
             \    |    /
                  v
             +----------+
             | DATABASE |
             +----------+
```

Now, multiple servers run the same application code.

### What is a Load Balancer?

A Load Balancer distributes incoming client requests among multiple application servers.

For example:

Suppose 3,000 users send requests.

- Server 1 handles approximately 1,000 requests.
- Server 2 handles approximately 1,000 requests.
- Server 3 handles approximately 1,000 requests.

The distribution depends on the load-balancing algorithm and actual server capacity.

### How does a Load Balancer decide which server receives a request?

Some common algorithms are:

1. Round Robin – Sends requests to servers one by one.
2. Least Connections – Sends requests to the server with the fewest active connections.
3. Weighted Round Robin – Distributes requests according to server capacity.
4. IP Hash – Uses the client's IP address to select a server.

### Important Concept: Health Checks

A load balancer continuously checks whether application servers are healthy.

For example:

```text
Load Balancer
     |
     |---- App Server 1 (Healthy)
     |
     |---- App Server 2 (Unhealthy)
     |
     |---- App Server 3 (Healthy)
```

If App Server 2 crashes or becomes unhealthy:

- The load balancer detects the failure.
- It stops sending new requests to Server 2.
- It continues routing requests to Servers 1 and 3.

This is called health-check-based routing.

**Important:** A load balancer itself can become a single point of failure if it is deployed without redundancy.

---

# Stage 4 – Database Bottleneck + Caching

## Problem: Database Becomes the Bottleneck

Now imagine our application has:

- 3 application servers.
- 10,000 active users.
- 1 database server.

Every application server communicates with the same database.

Consider an example:

A user uploads a photo.

1. The application receives the upload request.
2. The application stores information in the database.
3. The database performs the required operations.
4. The application returns a response.

Suppose a database operation takes 10 seconds because of heavy load.

Now imagine hundreds of users repeatedly requesting the same data.

The database keeps processing similar read requests.

This increases database load and response time.

## Solution: Caching

Caching means storing frequently accessed data in a faster storage layer so that future requests can retrieve it without repeatedly accessing the database.

A commonly used caching technology is Redis.

### Architecture

```text
           USER
             |
             v
       APPLICATION
             |
             v
          REDIS
          CACHE
             |
        Data Found?
          /    \
        YES     NO
         |       |
         v       v
      Return   DATABASE
       Data       |
                  |
                  v
             Store in Cache
                  |
                  v
              Return Data
```

### How does caching work?

Let's understand using a practical example.

Suppose a user requests their profile.

**First Request: Cache Miss**

1. Application receives the request.
2. Application checks Redis.
3. Data is not available in Redis.
4. Application fetches data from the database.
5. Application stores the result in Redis.
6. Application returns the data.

**Second Request: Cache Hit**

1. Application receives the request.
2. Application checks Redis.
3. Data is already available.
4. Application returns data directly from Redis.

The database does not need to process that read request again.

### Important Terms

**Cache Hit:** Requested data is found in the cache.

**Cache Miss:** Requested data is not found in the cache.

**TTL (Time To Live):** The time for which cached data remains valid.

Example:

```text
User Profile
     |
     v
Redis Cache
     |
     +---- Data: User Details
     |
     +---- TTL: 10 Minutes
```

After 10 minutes, the cached entry expires and may need to be fetched again.

### Cache Invalidation

What happens if a user updates their profile?

The old cached information may become outdated.

We need a strategy to update or invalidate the cache.

Common approaches:

- Cache Aside: Application reads from cache and loads from the database on a miss.
- Write Through: Data is written to the cache and database as part of the write operation.
- Write Behind: Data is written to the cache first and persisted to the database asynchronously.

**Important:** Caching improves read performance, but it introduces the challenge of keeping cached data consistent with the database.

---

# Stage 5 – Database Replication

## Problem: Database Becomes a Bottleneck Again

Caching helps with frequently accessed data.

However, consider an application with millions of users.

There are still many database operations, especially writes.

We also need better availability.

## Solution: Database Replication

Replication means maintaining copies of database data on multiple database servers.

A common architecture is Primary-Replica Replication.

### Architecture

```text
              APPLICATION
                   |
                   v
             +-----------+
             |  PRIMARY  |
             | DATABASE  |
             +-----------+
                /     \
               /       \
          Replication  Replication
             /           \
            v             v
       +----------+  +----------+
       | REPLICA 1|  | REPLICA 2|
       | DATABASE |  | DATABASE |
       +----------+  +----------+
```

### How does it work?

**Primary Database:**

- Handles write operations.
- Maintains the main copy of the data.
- Sends changes to replica databases.

**Replica Database:**

- Receives replicated changes.
- Can handle read operations.
- Maintains a copy of primary database data.

### Example

Suppose a user uploads a photo.

1. Application sends the write operation to the primary database.
2. Primary database stores the photo's metadata.
3. The changes are replicated to secondary databases.
4. Future read requests can be served by replicas.

### Read/Write Separation

We can distribute database operations:

| Operation | Database |
|---|---|
| INSERT | Primary |
| UPDATE | Primary |
| DELETE | Primary |
| SELECT | Replicas (when appropriate) |

### Advantages

1. Improves read scalability.
2. Reduces read load on the primary database.
3. Provides additional copies of data.
4. Can help with disaster recovery.

### Important Concepts

**Replication Lag:** The time difference between a change being written to the primary and becoming visible on a replica.

For example:

A user updates their profile on the primary database.

Immediately after the update, a read request reaches a replica.

The replica may still contain the old information.

This is called replication lag.

**Failover:** If the primary database fails, an eligible replica can be promoted to become the new primary.

Note: Replication is not the same as backup. Accidental deletion can also be replicated, so independent backups are still necessary.

---

# Stage 6 – Object Storage (Photo/File Storage)

## Problem: Database and Disk Storage Limitations

Imagine we are building Instagram.

Users upload:

- Profile pictures.
- Posts.
- Reels.
- Videos.
- Documents.

Initially, we might store images directly on the application server or inside the database.

But as the number of users increases, storage requirements become enormous.

For example:

- 1 million users upload photos.
- Each photo is 5 MB.
- Total storage required = 5 TB.

Storing all these files directly on application server disks is difficult to manage and scale.

## Solution: Object Storage

Object storage is designed to store large amounts of unstructured data, such as:

- Images.
- Videos.
- PDFs.
- Audio files.
- Documents.

Examples:

- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

### Architecture

```text
             USER
               |
               v
          APPLICATION
               |
               v
         OBJECT STORAGE
            (S3)
               |
               v
          IMAGE FILE
```

### How does it work?

Suppose a user uploads a profile picture.

1. User uploads the image.
2. Backend receives the image.
3. Backend uploads the image to object storage.
4. Object storage stores the actual file.
5. Object storage provides an object key or URL.
6. Backend stores the object key or URL in the database.
7. When the user requests their profile, the application retrieves the image using that reference.

### Database Structure Example

Instead of storing the complete image inside the database:

```text
Users Table

id       name       profile_image
-------------------------------------------
1        Utkarsh    users/1/profile.jpg
2        Rahul      users/2/profile.jpg
```

The actual image is stored in object storage.

The database only stores its reference.

### Advantages

1. Can store massive amounts of files.
2. Separates file storage from application servers.
3. Reduces database storage requirements.
4. Supports scalable and durable storage.
5. Can integrate with CDNs for faster delivery.

### Important Concept: CDN

A Content Delivery Network (CDN) stores or caches content at geographically distributed edge locations.

For example:

If your application server is in India and a user accesses an image from the USA, a CDN can serve the image from a nearby edge location instead of repeatedly fetching it from the original storage location.

---

# Stage 7 – Message Queues

## Problem: Time-Consuming Operations

Imagine a user uploads a photo to Instagram.

After uploading, the application needs to:

1. Save the image.
2. Generate thumbnails.
3. Compress the image.
4. Send notifications.
5. Update analytics.
6. Perform other background operations.

If the backend performs all these operations before responding, the user may have to wait a long time.

This creates a poor user experience.

## Solution: Message Queues

A message queue allows the application to process time-consuming tasks asynchronously.

Examples of queue technologies:

- RabbitMQ
- Apache Kafka
- Amazon SQS
- Redis-based queues (such as BullMQ)

### Architecture

```text
             USER
               |
               v
          APPLICATION
               |
               v
          MESSAGE QUEUE
               |
          +----+-----+
          |          |
          v          v
       Worker 1   Worker 2
          |          |
          v          v
       Thumbnail  Notification
       Generation   Service
```

### How does it work?

Suppose a user uploads a photo.

1. Application receives the upload.
2. Application stores the photo.
3. Application creates a background task.
4. Task is placed inside the message queue.
5. Application immediately responds to the user.
6. Background workers consume tasks from the queue.
7. Workers perform thumbnail generation, notifications, etc.

The user does not need to wait for all background operations to finish.

### Important Terms

**Producer:** The component that creates and sends messages to the queue.

**Queue:** Stores messages until workers process them.

**Consumer/Worker:** The component that receives and processes messages.

### Advantages

1. Improves response time.
2. Handles background processing.
3. Reduces direct dependency between services.
4. Helps absorb sudden traffic spikes.
5. Allows independent worker scaling.

### Important Concepts

**Retries:** If a task fails, the system can try processing it again.

**Dead Letter Queue (DLQ):** Stores messages that repeatedly fail processing.

**Idempotency:** Processing the same message multiple times should not create unintended duplicate effects.

For example, a payment task should not charge a customer twice because a queue message was retried.

---

# Stage 8 – Database Sharding

## Problem: Database Data Becomes Too Large

Imagine our application has:

- 100 million users.
- Billions of posts.
- Millions of daily database operations.

Even after replication, the primary database may become overloaded.

Replication creates copies of data, but it does not automatically divide the write workload among independent databases.

We need another approach.

## Solution: Database Sharding

Sharding means dividing a large database into smaller parts called shards.

Each shard stores a subset of the overall data.

### Architecture

```text
               APPLICATION
                    |
                    v
              SHARD ROUTER
                    |
          +---------+---------+
          |         |         |
          v         v         v
       +------+  +------+  +------+
       |Shard1|  |Shard2|  |Shard3|
       +------+  +------+  +------+
        Users     Users     Users
        1-1000   1001-2000 2001-3000
```

### How does it work?

Suppose we have 3,000 users.

We divide them into three shards.

- Shard 1 stores users 1–1000.
- Shard 2 stores users 1001–2000.
- Shard 3 stores users 2001–3000.

When a user requests their data, the application or shard router determines which shard contains that user's information.

### Sharding Strategies

**1. Range-Based Sharding**

Data is divided according to a range.

Example:

- User ID 1–1000 → Shard 1.
- User ID 1001–2000 → Shard 2.

**2. Hash-Based Sharding**

A hash function determines the shard.

Example:

```text
shard = hash(userId) % numberOfShards
```

**3. Geographic Sharding**

Data is divided according to geographical location.

Example:

- Indian users → India shard.
- US users → US shard.
- European users → Europe shard.

### Advantages

1. Distributes data across multiple databases.
2. Reduces the workload on individual database servers.
3. Supports large datasets.
4. Can improve performance for appropriately routed queries.

### Disadvantages

1. More complex architecture.
2. Cross-shard queries become difficult.
3. Transactions across shards are complicated.
4. Rebalancing data can be challenging.

### Important Difference: Replication vs Sharding

| Replication | Sharding |
|---|---|
| Creates copies of data | Divides data into subsets |
| Improves read scalability | Distributes data and workload |
| Replicas contain overlapping data | Shards contain different data |
| Helps availability | Helps horizontal database scaling |

---

# Stage 9 – Microservices Architecture

## Problem: Monolithic Application Becomes Difficult to Manage

Initially, our application may have everything inside one backend codebase.

For example, an Instagram-like application:

- Photo service.
- Reels service.
- Post service.
- User service.
- Notification service.
- Live streaming service.

All these functionalities are implemented inside one large application.

This is called a Monolithic Architecture.

### Monolithic Architecture

```text
             APPLICATION
                   |
        +----------+----------+
        |          |          |
        v          v          v
      Users      Posts      Photos
        |          |          |
        +----------+----------+
                   |
               Database
```

As the application grows:

- Code becomes harder to maintain.
- Deployments become more complex.
- A small change may require redeploying the entire application.
- Different modules may have different scaling requirements.

## Solution: Microservices Architecture

Microservices means dividing a large application into smaller, independently deployable services.

Each service handles a specific business responsibility.

### Architecture

```text
                  CLIENT
                     |
                     v
                API GATEWAY
                     |
        +------------+------------+
        |            |            |
        v            v            v
    User Service  Post Service  Photo Service
        |            |            |
        v            v            v
     User DB       Post DB      Photo DB


             Notification Service
                     |
                     v
                Notification DB
```

### Example: Instagram

| Service | Responsibility |
|---|---|
| User Service | Registration, login, profile |
| Post Service | Creating and managing posts |
| Photo Service | Uploading and retrieving images |
| Reels Service | Managing reels |
| Notification Service | Sending notifications |
| Live Stream Service | Managing live streaming |

### Why use Microservices?

1. Independent deployment.
2. Independent scaling.
3. Separate responsibilities.
4. Teams can work on different services.
5. A failure in one service does not necessarily bring down the entire application.

### Important Concept: API Gateway

An API Gateway acts as an entry point for client requests.

For example:

```text
              CLIENT
                 |
                 v
             API GATEWAY
                 |
       +---------+---------+
       |         |         |
       v         v         v
     User      Post      Photo
    Service   Service   Service
```

The API Gateway can handle:

- Request routing.
- Authentication.
- Rate limiting.
- Request validation.
- Logging.

### Important: Microservices Have Trade-offs

Microservices are not automatically better than monolithic architecture.

They introduce additional complexity:

- Network communication.
- Distributed debugging.
- Data consistency.
- Service discovery.
- Deployment and monitoring.
- Increased infrastructure costs.

For small applications, a well-structured monolith can be a better choice.

---

# Complete Journey of an Application (Stage 0–9)

| Stage | Concept | Problem Solved |
|---|---|---|
| 0 | Single Machine | Initial application setup |
| 1 | Separate App and DB | Resource competition |
| 2 | Vertical Scaling | Limited server capacity |
| 3 | Horizontal Scaling + Load Balancer | Application server overload |
| 4 | Caching | Repeated database reads |
| 5 | Database Replication | Read scalability and availability |
| 6 | Object Storage | Large file storage |
| 7 | Message Queues | Slow background operations |
| 8 | Database Sharding | Very large database workloads |
| 9 | Microservices | Large application complexity |

---

# Important Interview Questions

1. What is horizontal scaling?
2. What is a load balancer?
3. What is a health check?
4. What is the difference between vertical and horizontal scaling?
5. What is caching and why do we use Redis?
6. What is cache invalidation?
7. What is database replication?
8. What is replication lag?
9. What is the difference between primary and replica databases?
10. What is object storage?
11. Why should we store image URLs or object keys instead of image files in a database?
12. What is a CDN?
13. What is a message queue?
14. What is the difference between synchronous and asynchronous processing?
15. What is a Dead Letter Queue?
16. What is database sharding?
17. What is the difference between sharding and replication?
18. What is monolithic architecture?
19. What is microservices architecture?
20. What is an API Gateway?

---

# Final Revision – Remember These Concepts

- Horizontal Scaling = Add more servers.
- Load Balancer = Distribute incoming requests.
- Health Check = Identify unhealthy servers.
- Redis = Frequently accessed data in fast memory.
- Cache Hit = Data found in cache.
- Cache Miss = Data not found in cache.
- Replication = Maintain copies of database data.
- Primary DB = Handles primary writes.
- Replica DB = Can serve reads.
- Object Storage = Stores images, videos and files.
- CDN = Delivers content from nearby edge locations.
- Message Queue = Holds tasks for asynchronous processing.
- Worker = Processes background tasks.
- Sharding = Divide data across multiple databases.
- Microservices = Split an application into independently deployable services.

**Golden Rule:**

System Design is about understanding the problems that appear as an application grows and choosing the right architectural solutions to handle those problems.

Do not introduce every technology from the beginning. Introduce a solution when there is a problem that justifies it.
