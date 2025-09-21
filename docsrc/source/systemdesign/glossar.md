# Glossar

## APIs
The last important concept that we discuss in this article is Application Programming Interfaces (APIs).

These are sets of rules that allow different applications to communicate with each other.

Think of an API as a waiter in a restaurant. A client requests the waiter. The waiter (the API) takes the order to the kitchen (the Server), and gets back with the food for the client (the Response).

Three types of Web API architectural styles are popular and important to know about:

* REST (Representational State Transfer): This is the most common architectural style for designing web APIs. REST APIs use standard HTTP methods (GET, POST, PUT, DELETE), are stateless (so each request contains all the information for the server to process it), and usually return data in JSON format.

* SOAP (Simple Object Access Protocol): An older, highly structured API protocol that uses XML for message format. It is popularly used in enterprise environments because of its built-in error handling and security features.

* GraphQL: This is a newer API architectural style developed by Facebook (now Meta) that allows clients to request the exact data they need. Unlike REST, where multiple endpoints may be required, GraphQL uses a single endpoint to query for specific data requirements.



## CAP Theorem

The CAP theorem states that in a distributed system, you can guarantee at most two out of these three properties (consistency, availability, and partition tolerance) but never all three simultaneously.

This leads to systems that either have:

* High Consistency and Partition Tolerance (for example, databases such as MongoDB and HBase).
* High Consistency and Availability (for example, traditional relational databases such as PostgreSQL that run on a single machine).
* High availability and Partition tolerance (for example, databases such as Cassandra and CouchDB).

Extended by the PACELC Theorem


## Concurrency vs. Parallelism

* Parallelism refers to executing multiple tasks simultaneously across different processor cores or machines.

* Concurrency means executing multiple tasks simultaneously, either by running them in parallel or by rapidly switching between them on the same processor core.

*Parallelism is a subset of concurrency.*


## Consistency, Availability & Partition Tolerance

* Consistency: The guarantee that a system of all machines connected in a distributed manner sees the same data at the same time.
* Availability: The degree to which a system remains operational and responsive to requests.
* Partition Tolerance: The capability of a distributed system to continue operating despite network failures between the connected machines.

These three concepts are related to each other through the CAP theorem.

## Latency
In distributed systems, Latency is the time it takes for a request to go through the system and return a response.

Multiple techniques are used to decrease latency. Some of them are:

* Data replication and keeping copies of data in various regions closer to the users, as with Content Delivery Networks (CDNs).
* Caching: a technique where frequently accessed results are stored in memory in a cache server rather than constantly querying the central database (a slow operation) for results.
* Sharding: a technique where data is divided into smaller subsets called shards and distributed across multiple machines/ nodes.
In this way, when the system is queried, each query only reaches the nodes that store the relevant shards, which process the request, reducing load and response time.
* Load balancing: a technique of distributing incoming network requests across multiple servers to prevent any single server from becoming overloaded.
There are multiple algorithms for load balancing, and some popular ones are:

    * Round Robin: Sending requests to servers in a fixed cyclical order.
    * Least Connections: Sending requests to the server with the fewest active connections.
    * Least Response Time: Sending requests to the server with the fastest response time.
    * Weighted Round Robin: Sending requests to servers in a fixed cyclical order, but proportional to each server's capacity.




## PACELC Theorem
The CAP theorem is further extended by the PACELC theorem.

While the CAP theorem states that with network partitioning, a distributed system must choose between availability and consistency.

The PACELC theorem adds to this by stating that, where there is no network partitioning, the distributed system still faces a trade-off between Latency and Consistency.

The acronym PACELC stands for Partition (P), Availability (A), Consistency (C), Else (E), Latency (L), Consistency (C).

## Relational vs. Non-Relational Databases

* Relational/ SQL databases: These databases organize data into tables with predefined schemas and use SQL (Structured Query Language) to query them. For example, PostgreSQL and MySQL.

* Non-relational/NoSQL databases: These databases do not rely on fixed table schemas and store and query data in flexible ways. For example, MongoDB, Redis, and Cassandra.

### Transactions & Their Types

The next concept that you must know about is a Transaction, which is a logical unit of work that groups one or multiple operations.

The two types of transactions in distributed systems are:

* ACID transactions
* BASE transactions

Relational/ SQL databases use ACID transactions to maintain strict data integrity and relationships.

NoSQL databases use BASE transactions to achieve high scalability and performance by relaxing consistency requirements.

BASE transactions help large-scale applications, such as those from Amazon, Google, and Meta, work around the CAP/PACELC trade-offs by prioritizing availability and low latency over strict consistency.

#### ACID Transactions
The acronym ACID stands for:

* Atomicity: A transaction either completes entirely or fails, and no partial transactions occur.
* Consistency: A system remains in a valid state before and after each transaction, as per all rules and constraints.
* Isolation: Transactions running in parallel do so as if they’re the only ones executing. No two transactions interfere with each other’s intermediate states.
* Durability: Once a transaction is committed, its changes are permanently saved and are unaffected by system failures.


#### BASE Transactions
The acronym BASE stands for:

* Basically Available: The system remains operational even when parts of it fail, although some data may be temporarily inaccessible.
* Soft State: Data can be temporarily inconsistent across different parts of the system. These temporary inconsistencies are resolved at a later time.
* Eventual Consistency: Given sufficient time, all parts of the system synchronize and eventually converge to a consistent state.



## Scalability 
is a system’s ability to handle increased load while maintaining acceptable performance.

* Horizontal scaling: adding more machines/ servers to handle increased load by distributing work across them.
* Vertical scaling: increasing the power of existing machines/ servers by adding more CPU, RAM, or storage to handle increased load.

