# System Design Curriculum Roadmap

A structured index of foundational primitives, core trade-offs, and architectural components required to design scalable, fault-tolerant, and reliable distributed systems.

### Priority Marker Legend
* **`🔥` High-Priority Interview Anchor:** These concepts are heavily tested, form the backbone of core architectural decisions (e.g., sharding keys, state synchronization), or serve as primary tools for handling scale and failure modes under pressure. Focus deeply on filling out templates for these first.

---

## 1. System Design Fundamentals & Core Metrics
* Vertical vs. Horizontal Scaling 🔥
* Performance vs. Scalability
* Latency vs. Throughput vs. Bandwidth
* Availability vs. Reliability
* SLA / SLO / SLI
* Single Point of Failure (SPOF) 🔥
* Back-of-the-Envelope Estimation 🔥

## 2. Distributed Systems Theory & Guarantees
* CAP Theorem (CP vs. AP) 🔥
* PACELC Theorem
* Consistency Models
    * Strong Consistency 🔥
    * Eventual Consistency 🔥
    * Weak Consistency
    * Read-Your-Writes Consistency
* ACID vs. BASE
* Distributed Consensus Algorithms (Paxos, Raft)
* Leader Election Patterns
* Time and Ordering (Logical Clocks, Vector Clocks, TrueTime)

## 3. Networking & Application Protocols
* OSI Model Layers
* IP Routing & Domain Name System (DNS) 🔥
* Transport Protocols (TCP vs. UDP) 🔥
* Application Protocols
    * HTTP/1.1 vs. HTTP/2 vs. HTTP/3 (QUIC)
    * REST APIs
    * GraphQL
    * gRPC & Protocol Buffers 🔥
    * Webhooks
* Network Security (SSL/TLS, mTLS)

## 4. Traffic Management & Edge Routing
* Proxies vs. Reverse Proxies
* Load Balancing
    * Layer 4 vs. Layer 7 Load Balancing 🔥
    * Load Balancing Algorithms (Round Robin, Least Connections, IP Hash)
    * Consistent Hashing 🔥
* Content Delivery Networks (CDNs) (Push vs. Pull) 🔥
* API Gateways 🔥

## 5. Caching Strategies
* Client-Side & Edge Caching
* Application & Distributed Caching (In-Memory Data Stores) 🔥
* Cache-Aside Pattern 🔥
* Write-Through Caching
* Write-Behind / Write-Back Caching
* Refresh-Ahead Caching
* Cache Eviction Policies (LRU, LFU, FIFO)
* Cache Invalidation Strategies 🔥
* Cache Penalty Scenarios (Cache Penetration, Cache Stampede, Cache Avalanche) 🔥

## 6. Data Management & Storage Architecture
* SQL (Relational) vs. NoSQL (Non-Relational) 🔥
* Storage Engines & Data Layouts
    * Row-oriented vs. Column-oriented Storage
    * Log-Structured Merge-Trees (LSM-Trees) vs. B-Trees
* Database Scaling Patterns
    * Read Replicas & Replication Lag 🔥
    * Multi-Leader & Leaderless Replication
    * Database Federation
    * Database Sharding / Horizontal Partitioning (Range-based, Hash-based, Directory-based) 🔥
* Advanced Storage Primitives
    * Database Indexes (B-Tree, Inverted Indexes) 🔥
    * Bloom Filters
    * Write-Ahead Logging (WAL)
    * Object Storage vs. Block Storage vs. File Storage

## 7. Asynchronous Communication & Messaging
* Message Queues vs. Task Queues 🔥
* Publish-Subscribe (Pub/Sub) Pattern 🔥
* Point-to-Point Messaging
* Message Delivery Guarantees (At-least-once, At-most-once, Exactly-once) 🔥
* Change Data Capture (CDC)
* Event Sourcing
* Command and Query Responsibility Segregation (CQRS)

## 8. Resilience, Fault Tolerance & Traffic Control
* Rate Limiting Algorithms 🔥
    * Token Bucket
    * Leaky Bucket
    * Fixed Window Counter
    * Sliding Window Log / Counter
* Circuit Breaker Pattern 🔥
* Retries & Exponential Backoff with Jitter 🔥
* Backpressure Management
* Bulkheads & Throttling
* Dead Letter Queues (DLQ)
* Disaster Recovery (Active-Active vs. Active-Passive)

## 9. Distributed System Coordination & Microservices Patterns
* Service Discovery & Service Registries
* Service Mesh Architecture
* Distributed Transactions
    * Two-Phase Commit (2PC) / Three-Phase Commit (3PC)
    * SAGA Pattern (Orchestration vs. Choreography) 🔥
* Distributed Locking (Chubby, Redlock)
* Gossip Protocols & Heartbeats
* Vector Clocks & Merkle Trees
* Idempotency Keys & Patterns 🔥

## 10. Observability & Monitoring
* Centralized Logging & Log Aggregation
* Metrics Collection & Alerting
* Distributed Tracing
* Health Check API Patterns

## 11. Specialized Architecture Sub-Domains
* Geohashing, Quadtrees, & Spatial Indexes (Location-Based Services) 🔥
* Stream Processing vs. Batch Processing (Lambda vs. Kappa Architectures)
* User Authentication Architecture (OAuth 2.0, OIDC, SSO, JWT Session Management)
* Video Transcoding and Segmented Streaming Architecture