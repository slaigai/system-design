# Preface

# 1. Trade-Offs in Data Systems Architecture
- data intensive means an app where data management is one of the primary challenges in developing the app
- in contrast, compute intensive is parallelizing large computation
- data intensive apps worry more about storing and processing large volumes of data, managing changes to the data, and ensuring consistency in the face of failures and concurrency, and making services highly available
- different people need data for different reasons, one of the key challenges in designing
- focus on understanding choices and trade offs

## Operational Versus Analytical Systems
- Backend engineers, business analysts, data scientists
- Operational and analytical systems are often kept separate
- New role data engineers have emerged to integrate the operational and analytical systems and who take responsibility for the organization’s data infrastructure more widely

### Characterizing Transaction Processing and Analytics
- transaction: group of reads and writes that form a logical unit
  - Main read pattern: Point queries (fetch individual records by key)
  - Main write pattern: Create, update, and delete individual records
  - Human user example: End user of web/mobile application
  - Machine use example: Checking if an action is authorized
  - Type of queries: Fixed, predefined by application
  - Query volume: Lots of small queries
  - Data represents: Latest state of data (current point in time)
  - Dataset size: Gigabytes to terabytes
- analytics has a different access pattern, usually processes large number of records for aggregate metrics
  - Main read pattern: Aggregate over large number of records
  - Main write pattern: Bulk import (ETL) or event stream
  - Human user example: Internal analyst, for decision support
  - Machine use example: Detecting fraud/abuse patterns
  - Type of queries: Arbitrary, ad-hoc exploration by analysts
  - Query volume: Few queries, each is complex
  - Data represents: History of events that happened over time
  - Dataset size: Terabytes to petabytes
- OLTP: usually fixed queries
- OLAP: usually custom queries

### Data Warehousing
- Data warehouse is a separate database (separate from OLTP) where analysts can query to their heart's content
- OLTP DBs usually operated by individual teams in silos, not ideal for analysts
  - Data is spread across multiple sources, difficult to combine in single query
  - OLTP data layouts not well suited for OLAP
  - Analytical queries expensive to run on OLTP, causing performance hit
  - OLTP DB may be on a separate network
- data warehouse contains read only data optimized for analysis
- OLTP data -> ETL -> data warehouse
- enterprise usually has one data warehouse and many separate smaller OLTP DBs
- hybrid transactional/analytical processing (HTAP) exists but internally made of OLTP and analytical system. Still common to separate OLTP from analytical
- Data warehouse is often relational, queried through SQL
- Not well suited for feature engineering, turning data into a matrix. Use dataframes instead
- Data lake: contains files without any particular format, model, or schema. Can use object stores as backing instead of expensive relational DB
- ETL processes have been generalized to data pipelines
- Often ETL flows OLTP -> data lake -> data warehouse
  - Raw data is better, consumers can transform data into whatever form they like
- Data increasingly made available as streams of events (in addition to files and relational tables)
- Reverse ETL: outputs of analytical systems made available in OLTP

### Systems of Record and Derived Data
- system of record: source of truth. canonical data. typically normalized. system of record is the correct version of data if there is any discrepancy with other data.
- derived data: data transformed from another source. if derived data is lost, re-transform it. e.g. a cache
  - is redundant by nature
  - is essential for performance, copying the data is a trade off
- analytical systems are usually derived data
- By being clear about which data is derived from which other data, you can bring clarity to an otherwise confusing system architecture.
- when data in one system is derived from another, need a way to update derived data when system of record changes.
  - most dbs assume you're using one db system, hard to integrate between multiple systems
  - data pipelines used to compose multiple data systems to achieve what one alone cant

## Cloud Versus Self-Hosting
- build or buy
- usually about business priorities
  - if core competency, build
  - if not core competency, buy

### Pros and Cons of Cloud Services
- cloud saves time and money to move fast without spinning up own ops for infra
- less staff

### Cloud Native System Architecture
- cloud native architecture: arch designed to take advantage of cloud services
- some benefits of designing for cloud native: performance, fast recovery, scaling to match load, 
- most evident when looking at managed cloud native DBs (compared to self hosted)
  - OLTP: postgres VS spanner
  - OLAP: spark VS bigquery
- virtual disk: mimics a file system for durability while allowing ephermal storage attachments
- cloud generally separates storage from compute, incurs network transfer cost
- multitenant: many customer's data on one machine. shared hardware makes for better utilization, easier scalability, easier management
  - requires careful engineering to isolate customers from each other for security and correctness

### Operations in the Cloud Era
- traditionally DBAs and sysadmins handled data infra
- modern devs handle these systems
- devops and sre focus on
  - set up automation for repeatable processes
  - use ephemeral services rather than long-running servers
  - enabling frequent application updates
  - learning from incidents
  - preserving org knowledge
- capacity planning -> financial planning. performance optimization -> cost optimization
- also have to plan for quota limits

## Distributed Versus Single-Node Systems
- distributed system: a system of several machines communicating in a network. each process in a distributed system is called a node
- distributed systems used for:
  - inherently distributed: two people communicating across two devices is unavoidably distributed
  - requests between cloud services: data from one service is transferred to another
  - fault tolerance / high availability: use multiple machines for redundancy
  - scalability: data volume or computing requirements grow beyond what a single machine can handle, spread the load across multiple machines
  - latency: place servers close to users to minimize network latency
  - elasticity: can scale up or down depending on load. scalability is "spread the load", elasticity is "load adjusts automatically"
  - hardware tuning: object storage requires little CPU, some workloads require GPU, dont all have to be on the same hardware
  - legal compliance: data residency laws in some countries require data to be processed geographically within the country
  - sustainability: run where/when electricity is cheap

### Problems with Distributed Systems
- every request over network deals with possibility of failure
  - network interruption
  - service unavailable
  - request time out
  - retrying might not be safe, what do?
- consider bringing computation to the machines that already have the data to avoid network transfer
- more nodes not always better, overhead of concurrency may cost more than single threaded computation
- troubleshooting a distributed system is hard
  - where is the problem?
  - observability: collecting data about the system execution
  - tracing: which client called which server
- maintaining consistency across multiple databases is the application's problem
- distributed transactions: can be used but rare because services generally want to be independent and most dbs dont support it
- single-machine much simpler than distributed

### Microservices and Serverless
- SOA: service oriented architecture
- microservices: a service with one well-defined purpose
- can decompose a system into multiple services
- advantages of services
  - each service can be updated independently, reducing coordination across teams
  - each service assigned hardware resources it needs
  - hiding implementation behind API means decoupling with clients, can change implementation
- usually services have their own DB
  - effectively makes DB part of multiple service's API, difficult to change
  - one service's queries can affect another's
- many services = complexity
  - testing requires other services
  - delayed breaking changes, discovered during integration testing
- Microservices are primarily a technical solution to a people problem: allowing different teams to make progress independently without having to coordinate with each other.
- serverless / function as a service: cloud provider manages starting and stopping infra depending on load
  - only pay for time spent executing

### Cloud Computing Versus Supercomputing
- Supercomputers are typically used for computationally intensive scientific computing tasks
- typically runs large batch jobs that checkpoint the state of their computation to disk from time to time
- Supercomputer nodes typically communicate through shared memory and RDMA, which support high bandwidth and low latency but assume a high level of trust among the users of the system
- Supercomputers often use specialized network topologies, such as multidimensional meshes and toruses which yield better performance for HPC workloads with known communication patterns
- supercomputers generally assume that all their nodes are close together

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