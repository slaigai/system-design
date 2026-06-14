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
- software itself does not suffer material fatigure
  - requirements change
  - constraints change
  - environments change
  - dependencies change
  - bugs are introduced
- majority of cost is ongoing maintenance
- **Every system we create today will one day become a legacy system if it is valuable enough to survive for a long time.**
- principles
  - operability: make it easy for the org to keep the system running smoothly
  - simplicity: make it easy for people to understand
  - evolvability: make it easy for future changes

### Operability: Making Life Easy for Operations
- good operations can often work around the limitations of bad software
- good software cannot run reliably with bad operations
- more automation is not always better for ops
  - requires higher skill to operate
  - harder to deal with bad automation
  - find the balance
- good operability
  - make routine tasks easy
  - allow ops team to focus on high-value activities
- data systems can make operable by:
  - observing runtime behavior
  - avoid SPOF
  - provide good docs, simple, easy to understand, clear
  - good defaults, override levers
  - self-healing, give admins control when needed
  - predictable behavior, minimize surprises

### Simplicity: Managing Complexity
- big ball of mud: software project mired in complexity
- reducing complexity greatly improves maintainability
- simplicity is a key goal od systems we design, lowers risks
- no objective measure of simplicity
- one approach
  - essential complexity: inherent to the problem
  - accidental complexity: limitations of our tooling
  - not clear cut in practice
- abstraction: one of the best tools for managing complexity
  - reuse
  - decoupling
  - propagate quality improvements to consumers
  - design patterns and DDD can help

### Evolvability: Making Change Easy
- new use cases, business priorities, etc change
- ease of which you can modify a system to adapt to changing requriements
- irreversibility is the enemy

## Summary
- examined nonfunctional requirements
  - performance
  - reliability
  - scalability
  - maintainability

# 3. Data Models and Query Languages
- The limits of my language mean the limits of my world
- Meta point: learning the language of system design, reliability, architecture, etc. expands the toolkit
- Data models = how we think about the problem
- most apps are one data model layered on top of another
- each layer key question: how is it represented in terms of the next lower layer
- example of app layers
  - app layer: model people in objects/data structures, apis to manipulate them
  - data structures: expressed in general data model like json, or tables in DB, or vertices in graphs
  - db represents the data in disk layout or on a network
  - hardware represents memory or storage in voltage, current etc
- In general, each layer hides complexity of layers below it by providing a clean data model
- allows different groups of people to work effectively at different layers
- some types of data are easy to express in one model and awkward to express in other models
- some models
  - relational
  - document
  - graph
  - event sourcing
  - dataframes
- declarative query languages: specify the pattern of data you want but not how to achieve it. DB system handles forming a plan and executing
- algorithm: operations in order to perform a task
- declarative query language generally more concise and easier to write than algorithms
- declarative hides implementation details of the query engine

## Relational Versus Document Models
- SQL is best known example of relational model
- relations (called tables in SQL): unordered collection of tuples (rows in SQL)
- RDBMS: relational database management systems
- Early alternatives like the network model and hierarchical model existed but fell out of favor for relational models
- object storage also came and went
- XML databases came and was only niche
- SQL has grown to incorporate the others (XML, JSON, GRAPH)
- NoSQL: loos set of ideas around new data models, schema flexibility, scalability
- The term NoSQL has faded but the principles/ideas are still very influential
- document model:
  - one lasting NoSQL data model
  - usually represented as JSON
  - popularized by MongoDB and Couchbase
  - most relational DBs now support JSON
  - JSON more flexible than rigid fixed schema

### The Object-Relational Mismatch
- If most app data is object oriented, there's an awkward translation layer to SQL relational model.
- Impedance mismatch:
  - the friction that occurs when translating data between differing architectural paradigms. It primarily refers to the structural clash between object-oriented programming languages and relational database (SQL) tables
  - borrowed from electronics
  - when connecting one circuit to another, impedance mismatch leads trouble
- object relational mapping (ORM):
  - mapping between object model and relational model
  - frameworks often handle boilerplate code of translating between models
  - attempt to abstract the translation but often leak underlying details that developers still need to think about, but now with indirection
  - generally only for OLTP app dev
  - generally only work with relational OLTP DBs
  - Customizing ORM behavior can be complex if more control is wanted
  - make easy to write inefficient queries
  - N+1 query problem:
    - one query results in many other queries
    - e.g. display list of user comments on a page. one query returns N comments each with ID of its author. Need to show the name of each author so need to look up the IDs in the user's table.
    - using SQL, might join efficiently. using ORM, might come up with a dumb algorithm and cause many queries by accident
  - advantages include easy handling of common simple repetitive tasks, cache results, manage schema migrations
- Not all data lends itself well to a relational representation
- a JSON document more naturally represents an object structure than relational
- JSON model reduced impedance mismatch between app and storage
- lack of schema is an advantage sometimes, flexible. but comes with problems (e.g. data duplication)
- json represntation has better locality than multi-table schema. no need to join multiple tables to fetch one object record
- locality: all the data is in one place. trade off: duplication
- query is both faster and simpler
- one-to-few:
  - sometimes one-to-many relation actually only maps to a few items, well-suited for document model
  - sometimes one-to-many relation maps to many items. well-suited for relational model. document can get too big

### Normalization, Denormalization, and Joins
- some advantages of having a standardized list of things:
  - consistent style and spelling
  - avoid ambiguity between similar strings
  - easily updated across system
  - localization support
  - better search
- normalization: giving an ID to pieces of data that share the same meaning
  - ID has no meaning to humans, never needs to change
  - stuff that's meaningful to humans may need to change at any time
  - if the info is dupliucated, all copies must be updated. messy, imposes risk
  - downside: need to look up ID every time to resolve.
- denormalization: storing the thing directly
- document DBs can store both normalized and denormalized data
  - often associated with denormalization because JSON model makes it easier to store stuff denormalized
  - weak support for joins
  - in mongodb, possible to join using a lookup on json fields
- not a black and white choice between fully normalized or denormalized
  - usually have a mixture of them
  - decide carefully what needs to be normalized to keep things consistent
  - usually normalize if we want to reference one standard central instance of a thing over and over without needing to change it everywhere in case it needs to change
- normalized data usually faster to write (exactly one copy) but slower to query (requires joins)
- denormalized data usually more expensive to write (more copies to create) but faster to query (data is localized)
- not all DBs offer atomicity across multiple documents, may lost consistency in document DB for certain operations
- noramlization often better for OLTP systems
  - reads/updates need to be fast
- denormalization usually better for analytical systems
  - bulk updates, performance of read-only queries is dominant
- small to moderate scale systems normalized usually best to avoid multiple copies of inconsistent data and join performance is acceptable
- cost of joins in large scale systems may be problematic
- hydrating: process of looking up human readable info using the ID. basically a join in application code
- normalize fast-changing data so latest state can be looked up without expensive writes (updating copies) when needed
- joins are not always bad for performance
- most scalable approach may involve denormalizing some things, normalizing others
- need to carefully consider how information changes and the cost of reads and writes (can be dominated by outliers)
- norm/denorm are not inherently good or bad, simply represent tradeoffs in performance of reads/writes and implementation effort

### Many-to-One and Many-to-Many Relationships
- many-to-many often represented in associative tables or join tables
- many-to-one and many-to-many relationships done easily fit in one json document
  - better off normalized
  - often better to represent relationships with IDs in document rather than the whole copy in there
- many-to-many relationships often need to be queries in both directions
  - denormalized approach can become inconsistent, relation stored in multiple places can get out of sync
  - normalized approach is consistent, has one place where it's all captured
  - secondary indexes: allow relationship to be efficiently queried in both directions
- many document databases can create indexes on values inside a document
- data warehouses usually relational
  - widely used conventions for schemas: star, snowflake, dimensional modeling, one bug table (OBT)
  - optimized for needs of business analysts
  - ETL processes translate data from operational systems to the schema

### Stars and Snowflakes: Schemas for Analytics
- star schema
  - at center is a "fact table": each row repesents an event that occured at some time
  - usually fact table captured as individual events because maximally flexible for analysis later
  - can become huge, transactions table can be extremely large
  - some foreign key references to dimension tables: lookup stuff
  - queries often involve multiple joins
- snowflake schema
  - variation of star schema
  - dimensions further broken down into subdimensions
  - more normalized than star
  - star often preferred because simpler to work with
- tables in data warehouse are typically fairly wide
  - fact tables often have hundreds of columns
- star/snowflake usually consists of many many-to-one relationships
- one big table (OBT): precompute some joins so all the common fields are in one table
  - more storage space, quicker queries
  - in data warehouse, OBT for denormalized data is often not a problem, data doesnt change much
  - in OLTP, denormalized data is often a problem, data consistency problems and overhead of writes

### When to Use Which Model
- document model
  - schema flexibility
  - better performance for locality
  - lower impedance mismatch for object model in applications
- relational model
  - better support for joins
  - better support for many-to-one and many-to-many relationships
- if data in app has a document like structure (tree that's typically loaded all at once), use document model
- shredding: splitting a document into multiple tables in a relational model
  - can lead to cumbersome schemas if not the right fit
- document model has limitations like harder to refer to some nested item in a list
- document model supports custom user ordering of items, it's just a json array
  - in relational, would need to use common tricks to keep order
- most document DBs dont enforce a schema on documents
  - downside: arbitrary keys and values can be added to a document
  - clients have no guarantees as to what fields may exist
  - document DBs are sometimes incorrectly called schemaless but code usually expects some schema
- schema-on-read: structure of data is implicit, interpreted only when data is read
  - similar to dynamic type checking (at runtime)
- schema-on-write: schema is explicit, ensures all data conforms when written
  - similar to static type checking (at compile)
- no clear winner, useful for various use cases
- migration would be harder for document DB if need to change a field because all apps need to change the way they interpret data. relational migrations easier, changes all at once
- large migrations are a challenge no matter what
- schema-on-read useful if items in a collection dont all have the same structure (heterogeneous)
  - e.g. structure of data determined by external systems that we have no control over
  - schema can hurt more than it helps, needs to keep changing
- if whole document is usually needed, store in document, better locality better performance
- wasteful if only need small part of a large document, store in relational and query when needed
- some relational DBs offer locality controls, end up joining/grouping data into one table
- query language varies across models too
  - SQL often used for relational
  - document can be varied: key-value, secondary indexes for queries inside documents
  - document query language sometimes looks like json, matches structure
- document and relational DBs grown similar over time
  - convergence good, work best when both document and relational model combine in the same DB

## Graph-Like Data Models
- many-to-many relations are common in your data, can represent as graph
- vertices (nodes, entities) and edges (relationships, arcs)
- various representations
  - adjacency list: each vertex stores IDs of its neighbors
    - good for traversal
  - adjacency matrix: 2D array, all possible relations are mapped, boolean true/false if relation exists
    - good for machine learning where vectorization optimizes performance
- graphs are not limited to homogeneous data
  - e.g. facebook one big graph relates people, locations, events, comments, etc
- various ways to structure/query data: property graph model, triple store model. some DBs support both
- common query languages: Cypher, SPARQL, Datalog, GraphQL, SQL

### Property Graphs
- property graph:
  - vertex
    - unique ID
    - label (string) describing the type of object the vertex represents
    - outgoing edges
    - incoming edges
    - collection of properties (kv pairs)
  - edge
    - unique ID
    - vertex where edge starts (tail vertex)
    - vertex where edge ends (head vertex)
    - label describing the kind of relationship between two vertices
    - collection of properties (kv pairs)
  - can think kind of like a relational DB with 2 tables, one for vertices, one for edges
  - any vertex can have an edge connecting it with any other vertex, no restriction on what can be associated
  - can efficiently find incoming/outgoing edges of any vertex, easily traverse the graph
  - can store several kinds of information in one graph, still maintain clean data model
  - A hypergraph is a generalization of a standard graph in which an edge (called a "hyperedge") can connect any number of vertices rather than exactly two. This allows it to model complex, many-to-many relationships that simple graphs cannot represent naturally
- graphs highly flexible for data modeling
- good for evolvability, esily extended

### The Cypher Query Language
- Cypher: Query language for propery graphs originally created for Neo4j graph DB
- Later openCypher created
- Cypher also supported by memgraph, KuzuDB, Amazon Neptune, Apage AGE (with storage in PostgreSQL)

```cypher
CREATE
  (namerica :Location {name:'North America', type:'continent'}),
  (usa :Location {name:'United States', type:'country' }),
  (idaho :Location {name:'Idaho', type:'state' }),
  (lucy :Person {name:'Lucy' }),
  (idaho) -[:WITHIN ]-> (usa) -[:WITHIN]-> (namerica),
  (lucy) -[:BORN_IN]-> (idaho)

MATCH
  (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (:Location {name:'Europe'})
RETURN person.name
```

### Graph Queries in SQL
- `:WITHIN*0..` means follow a `WITHIN` edge zero or more times
- specifies a variable length traversal path
- in relational DB, would need a bunch of joins
- in relational, can use recursive common table expressions (`WITH RECURSIVE`)
- super complicated and clumsy
- 4 lines of cypher = 31 lines of SQL
- data model and query language makes a huge difference
- still havent considered handling cycles, BFS vs DFS,

### Triple Stores and SPARQL
- triple store model: equivalent to property graph model using different words
- stored in three part statements (subject, predicate, object)
- turtle: language for encoding triples
- resource description framework (RDF): data model for semantic web (encoded in XML)
- SPARQL: language for triple stores using RDF data model
  - Recursively named "SPARQL protocol and RDF query language"
  - predates cypher, cypher borrows query patterns

### Datalog: Recursive Relational Queries
- older than SPARQL and cypher
- based on relational data model, not graph
- recursive queries on graphs are a strength of datalog
- datalog rows are called facts
- datalog is a subset of prolog
- based on building up rules one by one, and referencing them like recursive functions

### GraphQL
- by design is more restrictive than others
- intended for OLTP queries
- purpose: allows client software (web, mobile) to request JSON with a particular structure containing necessary fields for rendering UI
- allows devs to rapidly change queries in client code without changing server APIs
- highly flexible, but has a cost
- need tooling to convert queries into requests to internal services
- auth, rate limiting, performance add concerns
- intentionally limited because GraphQL queries come from untrusted sources
  - doesnt allow expensive queries (avoid DoS)
  - does not allow recursive queries
  - does not allow arbitrary search conditions
- server doesn't need to know which attributes the client needs, client just requests what it wants
- GraphQL accepts larger response sizes in order to make it simpler to query
- GraphQL query might look like a document and have graph in its name but it can be implemented on top of any type of database
  - relational, document, graph

## Event Sourcing and CQRS
- data so far: queried in the same form it's written
- in complex applications, sometimes difficult to find a single data representation that is able to satisfy all ways the data needs to be queried
- beneficial to write data in one form and derive representations from it that are optimized for reads
- event log: simplest, fastest, most expressive way of writing data
  - every time you want to write some data, encode it as a self-contained string with timestamp and append it to a sequence of events
  - events in this log are immutable, append only to the log
- when events are appended to a log, we can update materialized views (aka projections, read models)
- event sourcing: using events as the source of truth and expressing every state change as an event
- CQRS command query responsibility segregation: the principle of separating read-optimized representations and deriving them from write-optimized representations
- command: request from a user
- fact: a command that has been executed and validated
- event log should only contain valid events
- consumer of an event log is not allowed to reject an event
- when modeling data in event sourcing, convention is to name them as past tense because it's a record of a fact that has already happened
- event sourcing and star schema both are collections of events that happened in the past but rows in a fact table all have the same schema
- event sourcing may have many types, each with different properties
- fact table is an unordered collection
- event sourcing has order to the events
- event sourcing and CQRS advantages
  - events better communicate WHY something happened
  - key principle of event sourcing: materialized views are derived from the event log in a reproducible way
  - should always be able to delete and recompute the materialized view usng the same events and same code
  - can have multiple materialized views optimized for particular queries
  - can build new materialized views from the same event log
  - if there are errors in events, later events can fix them (new update/delete events, original unmodified). consumers automatically take the updates
  - event log is an audit
  - event logs can handle higher write throughput than DBs because sequential access pattern
  - downstream materialized views can catch up in their own time
- event sourcing and CQRS downsides
  - be careful if events rely on external information (like currency exchange rate). interpretation changes over time. Either include it in event or have a way to query historical exchange rate
  - GDPR gets tricky, can store personal data outside of event or encrypt it with a key that you later delete (technique called crypto shredding)
  - reprocessing events requires care if there are external side effects (e.g. dont want to resend confirmation emails every time you rebuild a materialized view)
- can implement event sourcing on top of any DB but some systems specifically support this pattern (EventStoreDB, MartenDB, Axon Framework)
- can also use message brokers like Kafka, stream processors keep materialized views up to date
- the only important requirement: event storage systems must guarantee all materialized views process the events in exactly the order as they appear in the log
  - in distributed systems, not so easy

## DataFrames, Matrices, and Arrays
- so far, data models used for transactions and analytics
- Dataframes and multidimensional arrays rare in OLTP but common in scientific context
- dataframes are populat rools for data scientists prepping data for training ML models
- similar to table with relational operations
- instead, uses imperative commands to manipulate its structure
- often have a broader API for manipulating data (like converting to sparse matrix)
- matrix only contains numbers
- one-hot-encoding: booleans
- matrices used for linear algebra, which is basis for many ML algorithms
- dataframes also popular in batch processing frameworks like spark and flink

## Summary
- went through various models
  - relational
  - document
  - graph
  - dataframe
- usually, one model can be emulated in terms of another model but only awkwardly
- specialized DBs have been developed for each model
- DBs been trending toward covering multiple models
- covered event sourcing and CQRS
- not covered
  - sequence similary searches in large databases of strings
  - ledgers like in financial industry
  - full text search

# 4. Storage and Retrieval
- fundamentally, DBs store and retrieve data
- data model: the format of data
- query language: interface for asking for data
- this chapter from DBs POV: how it can store data and find it for you
- app devs should know internals because we need to select a storage enginer that's appropriate for the app
- to configure a storage engine well, need a rough idea of the storage engine under the hood
- big difference between storage engines for OLTP and analytics
- two families of OLTP
  - log-structured: write immutable data files
  - B-trees: udpate data in place
  - both used for key-value storage and secondary indexes

## Storage and Indexing for OLTP
- index: used to efficiently find keys
- idea: structure data in a certain way so it's faster to locate the data you want
- indexes incur overhead to maintain on writes
- important trade off: good indexes speed up queries at the cost of more disk space and slows down writes substantially
- DBs dont usually index everything by default

### Log-Structured Storage
- index in log could look like a hash map in memory of all the offsets of each key
- disadvantages
  - log grows forever without way to reclaim space if old logs were overwritten
  - hash map in memory needs to be rebuilt every restart of DB
  - hash table must fit in memory
  - range queries inefficient on a map
- SSTable file format
  - in practice, hash table not used often for DB indexes
  - more common to keep data sorted by key
  - Sorted Strings Table (SSTable)
  - stores key value pairs sorted by key, each key appears once in the file
  - can group keys into blocks, keep only first key of each block in memory
    - this is a sparse index
    - stored in a separate part of the SSTable (e.g. an immutable B-tree, trie, or other data structure)
    - blocks are kept small (few KB), can be scanned quickly
    - blocks can also be compressed to reduce disk space and improve IO speed
- SSTable better for reads compared to append only log but worse for writes
  - can't just append to end of log, need to write to B-tree, index, and block
- log-structured approach: hybrid between append-only log and sorted file
  - memtable: when write comes in, add it to in-memory ordered map data structure (e.g. red-black tree, skip list, trie)
  - when memtable gets bigger past a threshold, write it to disk in sorted order as an SSTable file
  - most recent new SSTable file is a segment of the DB, stored alongside older segments on diks
  - old memtable's memory is freed when segment written to disk
  - DB can write to new memtable while writing old memtable to disk
  - to read, first try to read from memtable. if key not there, look in next older segmenet until key is found or reached oldest segment
  - occaisionally run compaction in background to combine segment files, discard overwritten or deleted values
    - merging works like mergesort but keep only latest key
  - in case of crashes, a separate log is kept on disk where every write is immediately appended
    - this log is not sorted by key, just a log of everything raw
    - when memtable gets written to disk, part of the log can be discarded
- to delete a record, must append a tombstone (tells merge process to discard previous values)
- algorithm here ^ is what is used in RocksDB, Cassandra, ScyllaDB, HBase, all of which were inspired by google BigTable that introduced SStables and memtables
  - originally the algorithm was published as Log-structured merge trees (LSM-trees)
- LSM storage engines: storage engines based on the principle of merging and compacting sorted files
  - segments are immutable
  - merging done in background thread while reads continue to be served
  - when merging is done, switch reading to the new segments and delete the old stuff
- Segment files dont need to be stored on local disk, can be stored in external object storage (SlateDB and Delta Lake)
- immutable segment files simplifies crash recovery
  - if crash during merging or writing memtable, can  just delete the unfinished SSTable and start fresh
- bloom filters: fast but approximate way to check if key appears in any SSTable
  - hash the key, store in bitmap, check bitmap for existence
  - if any bit is zero, key definitely doesnt exist
  - if all bits are 1, key possibly exists (could be combination of keys, false positive)
  - false positive probability depends on number of bits and number of keys
- compaction strategies
  - size-tiered compaction: newer smaller SSTables merged into older larger SSTables. high write throughput
  - leveld compaction: keeps each level at a certain size. more efficient for reads, fewer SSTables to read

### B-Trees
### Comparing B-Trees and LSM-Trees
### Multicolumn and Secondary Indexes
### Storing Values Within the Index
### Keeping Everything in Memory

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