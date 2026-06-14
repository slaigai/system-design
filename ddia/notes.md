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
- GDPR grants individuals the right to have their data erased on request
- many data systems rely on immutable constructs such as append-only logs as part of their design.
- How can we ensure deletion of some data in the middle of a file that is supposed to be immutable?
- How do we handle deletion of data that has been incorporated into derived datasets
- risk of legal costs and fines if the storage and processing of the data is found not to be compliant with the law
- data minimization: keep only what is necessary, cost-benefit

## Summary
- theme of this chapter: understand trade offs
- OLTP vs OLAP: manage data differently, serve different audiences
- data warehouse (ready for analytics) and data lake (raw files)
- cloud is intrinsically distributed, comes with the engineering challenges of distributed systems

# 2. Defining Nonfunctional Requirements
- performance, reliability, scalability, maintainability

## Case Study: Social Network Home Timelines
### Representing Users, Posts, and Follows
### Materializing and Updating Timelines
- fan-out: when one request results in several downstream requests
- for the twitter/X case, timelines can be served from cache, eventually updated
- materialization: precomputing and updating the results of a query
- materialized view: cache of precomputed results at the cost of more writes
- what about users who follow many accounts?
  - would naively write many times to their timeline
  - okay to sample posts because user is probably not reading all of them from all accounts they're following. dropping some writes is okay
- what about users with many followers?
  - would naively need to write to many other account timelines
  - dropping writes is not okay, maybe they're a celebrity
  - can store celebrity posts separately and merge them with timelines when they're viewed

## Describing Performance
- Performance usually focuses on response time and throughput
  - response time: elapsed time from request to response
  - throughput: rate of requests, volume flow rate
- queueing: when load increases, system gets heavily loaded and throughput goes down, requests take longer to process. basically backing up
- retry storm: latency increases until clients time out. clients all retry at once, making the problem worse
- metastable failure: system degrades due to a transient stressor and remains in a severely degraded state. unable to recover without external intervention
  - technically up but effectively down
  - positive feedback loop keeps it degraded
  - common causes: retry storm, cache miss cascades, garbage collection overload
  - Use circuit breakers and bulkheads to stop cascading failures from spreading across the architecture
  - load shedding: server can proactively start rejecting requests
  - backpressure: send back responses telling clients to slow down
  - Implement exponential backoff and jitter into client and service retries to prevent retry storms and work amplification
  - Avoid over-optimizing specific pathways that, when removed, leave the system unable to handle the standard workload
  - Proactively monitor the rate of increase in queue lengths rather than just tracking CPU or memory spikes
- A system is scalable if its max throughput can be significantly increased by adding compute resources

### Latency and Response Time
- Specific terms
  - response time: what the client sees, all delays incurred anywhere in the system
  - service time: duration that the service is actively processing the request
  - queueing delays: waiting for CPU, buffered before going over network, etc
  - latency: time which a request is not actively being processed (i.e. latent), network latency is time spent traveling on the wire
- head-of-line blocking: a small number of slow requests "hold up the line" for everyone, slows everything down hogging compute resources

### Average, Median, and Percentiles
- response times vary, need to think in distributions
- median is a good for typical wait time
- 95 and 99 percentiles are good for knowing how bad your outliers are
- tail latencies: high response time percentiles
  - usually the worst cases directly affecting user experience
  - e.g. customers with slowest request may have the most data, may be customers who have made the most purchases
- research data is contradictory on how much latency affects the user experience

### Use of Response Time Metrics
- it takes just one slow call (even in parallel) to make the whole user experience slow
- tail latency amplification: The frustration of tail latency lies in probability compounding. If a user request fans out to 10 parallel services to compile a page, and each individual service has a perfectly fine 99% success rate of being fast, you might assume the overall system is safe. However, the probability of all 10 services returning quickly is 0.99^10 ≈ 0.904
- service level objectives (SLOs): target metrics (e.g. < 200ms median and < 1000ms 99p and < 0.1% error rate)
- service level agreements (SLAs): contract that specifies what happens if an SLOs is not met (e.g. customers get a refund)
- in practice, defining good SLOs and SLAs is not straightforward

## Reliability and Fault Tolerance
- app performs the function as the user expected
- app can tolerate the user making mistakes or using the software in unexpected ways
- performance is good enough for the required use case under expected load and data volume
- system prevents any unauthorized access and abuse
- reliability: system continues working correctly even when things go wrong
- fault: part of a system stops working correctly (doesn't mean the system has failed necessarily)
- failure: whole system stops providing the required service to the user (does not meet SLO)

### Fault Tolerance
- fault-tolerant: continues providing the required service to users even when faults occur
- single point of failure (SPOF): system cannot tolerate a certain part becoming faulty. fault in that one part becomes a failure of the whole system
- fault tolerance is always limited to a certain number of certain types of faults
  - e.g. can tolerate a max of 1 out of 3 nodes crashing at a time
  - wouldnt make sense to tolerate any number of faults
  - if all nodes crash, it cannot possibly be tolerated
- fault injection: deliberately triggering faults (e.g. randomly killing processes)
- chaos engineering: using fault injection to improve tolerance in fault tolerance mechanisms
- we ideally want to tolerate faults. however, preventing them can be even better (e.g. security)

### Hardware and Software Faults
- 2-5% of magnetic hard drives fail per year. on average with 10,000 disks, one failure per day
- 0.5-1% of SSDs fail per year
- CPU manufacturing defects
- RAM corruption
- data center smoking hole in the ground
- hardware faults are part of normal system operation in large-scale systems
- redundancy is most effective when component faults are independent, one fault doesnt change the likelihood of another
- cloud focuses less on single machine reliability and more on service high availability by tolerating nodes at the software level
  - availability zones: resources physically colocated, resources in same place are more likely to fail at the same time (compared to geographically separated resources)
  - can have one datacenter take over when another fails
- rolling restart: multi-node fault-tolerant system doesn't require downtime for maintenance/upgrades
- software faults are often highly correlated
  - common for many nodes to run the same software, therefore having the same bugs
  - tend to be a higher cause of failures than hardware
- latent faults: bugs that lie dormant until certain unusual specific circumstances arise.
  - reveals that the software is making some kind of assumption about its environment
  - even if assumption is usually true, it can eventually stop being true for some reason
- impossible to eliminate software faults but can try
  - critical thinking assumptions and interactions in the system
  - thorough testing
  - process isolation
  - allowing processes to crash and restart
  - avoiding feedback loops such as retry storms
  - observing prod behavior

### Humans and Reliability
- humans make errors
- config changes by operators account for the vast majority of failures
- blaming people for mistakes is counterproductive.
  - What we call “human error” is not really the cause of an incident, but rather a symptom of a problem with the sociotechnical system in which people are trying their best to do their jobs
  - Often complex systems have emergent behavior, in which unexpected interactions between components may also lead to failures
- minimizing human mistakes
  - thorough testing
  - rollback mechanisms
  - gradual rollouts
  - detailed and clear monitoring
  - observability tools for diagnosing prod issues
  - well-designed interfaces that encourage the "right-thing" and discourage the "wrong-thing"
- practically, can't have it all. it's a business
  - organizations often prioritize revenue-generating activities over measures that increase their systems’ resilience against mistakes.
  - Given a choice between more features and more testing, many organizations understandably choose features.
  - Then, when a preventable mistake inevitably occurs, blaming the person who made the mistake does not make sense
  - the problem is the organization’s priorities.
- blameless postmortems: everyone invited to share full details without fear of punishment, prevents similar problems in the future
  - may uncover need to change business priorities, invest in certain areas, change incentives, or bring another systemic issue to management's attention

## Scalability
- system working today may degrade tomorrow as load grows
- scalability: system's ability to cope with increased load
- sometimes counterproductive to worry about scale if another conflicting factor has a much higher priority
  - best case: premature optimization wasted effort
  - worst case: vendor lockin with baggage and slows velocity
- meaningless to say "X is scalable" or "Y doesn't scale". instead consider questions like:
  - if the system grows in a particular way, what are our options for coping with the growth?
  - how can we add computing resources to handle the additional load?
  - based on current growth projections, when will we hit the limits of our current architecture?

### Understanding Load
- load: throughput, rate, number of something per time. sometimes peak of a number (e.g. number of simultaneously active users)
  - number of requests per second
  - gigs of data per hour
  - number of checkouts per hour
- load can mean access pattern (ratio of reads vs writes), cache hit rate, number of insights per user
- maybe you care about average, maybe care about extreme cases. depends
- investigate what happens when load increases
  - When you increase the load in a certain way and keep the system resources (CPUs, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
  - When you increase the load in a certain way, how much do you need to increase the resources if you want to keep performance unchanged?
- Goal is usually keep the performance within SLAs while minimizing cost of system
- linear scalability: doubling resources leads to handling twice the load
- sublinear scalability is amazing
- typically, superlinear scaling. cost grows faster than linearly

### Shared-Memory, Shared-Disk, and Shared-Nothing Architectures
- vertical scaling, scaling up: beefier machine
- shared memory architecture: bunch of threads sharing the same memory
  - high end machine with more cores costs much more than linear
  - overhead means performance scales sublinearly
- shared-disk architecture: bunch of machines share disk
  - network-attached storage: fast network shared storage
  - storage area network: same ^
  - usually used for on prem
  - contention, overhead of locking hampers scalability
- shared-nothing architecture, horizontal scaling, scaling out
  - each node has its own CPU, RAM, disk
  - software-level coordination via conventional network
  - can often scale linearly
  - can use whatever hardware gives best price/performance ratio
  - can adjust hardware resources as load changes
  - can achieve greater fault tolerance (distribute across datacenters and regions)
  - downside is requires explicit sharding, incurs complexity

### Principles for Scalability
- no such thing as a one-size fits all solution
- an architecture suitable for one level of load is unlikely to cope with 10x a load
- needs of application are likely to evolve so not worth planning more than 1 order of magnitude in advance
- A good general principle for scalability is to break a system into smaller components that can operate largely independently from one another
- The challenge lies in knowing where to draw the line between things that should be together and things that should be apart
- Another good principle is not to make things more complicated than necessary. If a single-machine database will do the job, it’s probably preferable to a complicated distributed setup
- Autoscaling systems (which automatically add or remove resources in response to demand) are cool, but if your load is fairly predictable, a manually scaled system may have fewer operational surprises

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