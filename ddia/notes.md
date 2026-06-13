# Preface

# 1. Trade-Offs in Data Systems Architecture
## Operational Versus Analytical Systems
## Characterizing Transaction Processing and Analytics
## Data Warehousing
## Systems of Record and Derived Data
## Cloud Versus Self-Hosting
## Pros and Cons of Cloud Services
## Cloud Native System Architecture
## Operations in the Cloud Era
## Distributed Versus Single-Node Systems
## Problems with Distributed Systems
## Microservices and Serverless
## Cloud Computing Versus Supercomputing
## Data Systems, Law, and Society
## Summary

# 2. Defining Nonfunctional Requirements
## Case Study: Social Network Home Timelines
## Representing Users, Posts, and Follows
## Materializing and Updating Timelines
## Describing Performance
## Latency and Response Time
## Average, Median, and Percentiles
## Use of Response Time Metrics
## Reliability and Fault Tolerance
## Fault Tolerance
## Hardware and Software Faults
## Humans and Reliability
## Scalability
## Understanding Load
## Shared-Memory, Shared-Disk, and Shared-Nothing Architectures
## Principles for Scalability
## Maintainability
## Operability: Making Life Easy for Operations
## Simplicity: Managing Complexity
## Evolvability: Making Change Easy
## Summary

# 3. Data Models and Query Languages
## Relational Versus Document Models
## The Object-Relational Mismatch
## Normalization, Denormalization, and Joins
## Many-to-One and Many-to-Many Relationships
## Stars and Snowflakes: Schemas for Analytics
## When to Use Which Model
## Graph-Like Data Models
## Property Graphs
## The Cypher Query Language
## Graph Queries in SQL
## Triple Stores and SPARQL
## Datalog: Recursive Relational Queries
## GraphQL
## Event Sourcing and CQRS
## DataFrames, Matrices, and Arrays
## Summary

# 4. Storage and Retrieval
## Storage and Indexing for OLTP
## Log-Structured Storage
## B-Trees
## Comparing B-Trees and LSM-Trees
## Multicolumn and Secondary Indexes
## Storing Values Within the Index
## Keeping Everything in Memory
## Data Storage for Analytics
## Cloud Data Warehouses
## Column-Oriented Storage
## Query Execution: Compilation and Vectorization
## Materialized Views and Data Cubes
## Multidimensional and Full-Text Indexes
## Full-Text Search
## Vector Embeddings
## Summary

# 5. Encoding and Evolution
## Formats for Encoding Data
## Language-Specific Formats
## JSON, XML, and Binary Variants
## Protocol Buffers
## Avro
## The Merits of Schemas
## Modes of Dataflow
## Dataflow Through Databases
## Dataflow Through Services: REST and RPC
## Durable Execution and Workflows
## Event-Driven Architectures
## Summary

# 6. Replication
## Single-Leader Replication
## Synchronous Versus Asynchronous Replication
## Setting Up New Followers
## Handling Node Outages
## Implementation of Replication Logs
## Problems with Replication Lag
## Solutions for Replication Lag
## Multi-Leader Replication
## Geographically Distributed Operation
## Sync Engines and Local-First Software
## Dealing with Conflicting Writes
## Leaderless Replication
## Writing to the Database When a Node Is Down
## Single-Leader Versus Leaderless Replication Performance
## Multi-Region Operation
## Detecting Concurrent Writes
## Summary

# 7. Sharding
## Pros and Cons of Sharding
## Sharding for Multitenancy
## Sharding of Key-Value Data
## Sharding by Key Range
## Sharding by Hash of Key
## Skewed Workloads and Relieving Hot Spots
## Operations: Automatic Versus Manual Rebalancing
## Request Routing
## Sharding and Secondary Indexes
## Local Secondary Indexes
## Global Secondary Indexes
## Summary

# 8. Transactions
## What Exactly Is a Transaction?
## The Meaning of ACID
## Single-Object and Multi-Object Operations
## Weak Isolation Levels
## Read Committed
## Snapshot Isolation and Repeatable Read
## Preventing Lost Updates
## Write Skew and Phantoms
## Serializability
## Actual Serial Execution
## Two-Phase Locking
## Serializable Snapshot Isolation
## Distributed Transactions
## Two-Phase Commit
## Distributed Transactions Across Different Systems
## Database-Internal Distributed Transactions
## Exactly-Once Message Processing Revisited
## Summary

# 9. The Trouble with Distributed Systems
## Faults and Partial Failures
## Unreliable Networks
## The Limitations of TCP
## Network Faults in Practice
## Fault Detection
## Timeouts and Unbounded Delays
## Synchronous Versus Asynchronous Networks
## Unreliable Clocks
## Monotonic Versus Time-of-Day Clocks
## Clock Synchronization and Accuracy
## Relying on Synchronized Clocks
## Process Pauses
## Knowledge, Truth, and Lies
## The Majority Rules
## Distributed Locks and Leases
## Byzantine Faults
## System Model and Reality
## Formal Methods and Randomized Testing
## Summary

# 10. Consistency and Consensus
## Linearizability
## What Makes a System Linearizable?
## Relying on Linearizability
## Implementing Linearizable Systems
## The Cost of Linearizability
## ID Generators and Logical Clocks
## Logical Clocks
## Linearizable ID Generators
## Consensus
## The Many Faces of Consensus
## Consensus in Practice
## Coordination Services
## Summary

# 11. Batch Processing
## Batch Processing with Unix Tools
## Simple Log Analysis
## Chain of Commands Versus Custom Program
## Sorting Versus In-Memory Aggregation
## Batch Processing in Distributed Systems
## Distributed Filesystems
## Object Stores
## Distributed Job Orchestration
## Batch Processing Models
## MapReduce
## Dataflow Engines
## Shuffling Data
## Joins and Grouping
## Query Languages
## DataFrames
## Batch Use Cases
## Extract-Transform-Load
## Analytics
## Machine Learning
## Serving Derived Data
## Summary

# 12. Stream Processing
## Transmitting Event Streams
## Messaging Systems
## Log-Based Message Brokers
## Databases and Streams
## Keeping Systems in Sync
## Change Data Capture
## State, Streams, and Immutability
## Processing Streams
## Uses of Stream Processing
## Reasoning About Time
## Stream Joins
## Fault Tolerance
## Summary

# 13. A Philosophy of Streaming Systems
## Data Integration
## Combining Specialized Tools by Deriving Data
## Batch and Stream Processing
## Unbundling Databases
## Composing Data Storage Technologies
## Designing Applications Around Dataflow
## Observing Derived State
## Aiming for Correctness
## The End-to-End Argument for Databases
## Enforcing Constraints
## Timeliness and Integrity
## Trust, but Verify
## Summary

# 14. Doing the Right Thing
## Predictive Analytics
## Bias and Discrimination
## Responsibility and Accountability
## Feedback Loops
## Privacy and Tracking
## Surveillance
## Consent and Freedom of Choice
## Privacy and Use of Data
## Data as Assets and Power
## Remembering the Industrial Revolution
## Legislation and Self-Regulation
## Summary

# Glossary

# Index