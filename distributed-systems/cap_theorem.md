# CAP Theorem

In this context, a **distributed system** is a **network of nodes that work together to store and manage data across multiple locations**.

The CAP Theorem, formulated by Eric Brewer and formally proven by Seth Gilbert and Nancy Lynch, states that a distributed data store can provide only two out of three guarantees at the same time:
- C – Consistency
- A – Availability
- P – Partition Tolerance

Since **network failures (partitions) are inevitable**, a distributed system must **choose between consistency and availability** during a partition.

The CAP theorem **is critical in designing distributed databases**, helping developers prioritize trade-offs based on application needs.

## Consistency
Every read request returns the most recent write, ensuring that **all clients always see the same data**, regardless of which node they connect to.

For this to happen, whenever data is written to one node, the system must propagate this **update to all other nodes before the write is deemed "successful"** and response sent to clients.
In a system that prioritizes consistency, **some requests may fail or be delayed until data synchronization is complete**.

**Example Use Cases**: Banking transactions, inventory management, authentication systems.

## Availability
**Every request receives a response**, even if some nodes are unavailable or network failures occur. In this case, **some data inconsistencies may occur** due to delayed updates across nodes.

**Nodes do not need to be perfectly synchronized**; they return a response based on their current state, even if it may not be the most up-to-date.

**Example Use Cases**: E-commerce websites, social media feeds, content delivery networks (CDNs).

## Partition tolerance
Nodes in the distributed system send messages to each other for their cooperation. **A partition is a communication break within a distributed system**, thus a lost or temporarily delayed connection between two nodes.

A partition tolerant system **continues to function even if communication between some nodes is lost** (network partition occurs).
Nodes in the system operate independently and try to synchronize once the partition is resolved.
The system can sustain any amount of network failure that does not result in a failure for the entire network. 
In this case, intermittent data outages should not impact on the system capacity to serve client requests.

Partition tolerance is a must-have in real-world distributed systems since network failures can and do occur.
The distributed system, in particular the distributed data store, operates with network partitions making necessary the partition tolerance for a reliable service, since network errors can occur. 

# Distributed storage types
Partitions are unavoidable, thus, the system must choose between prioritizing consistency (blocking outdated responses) or availability (serving possibly stale data).

## CP database - Consistency and partition tolerance
These databases guarantee data consistency even in the presence of network partitions, but some node may become temporarily unavailable until partitions are resolved.

Databases that offer consistency and partition tolerance sacrifice availability. The practical result is that when a partition occurs, the system must make the inconsistent node unavailable until it can resolve the partition. 

**Example of CP databases**: MongoDB and Redis are examples of CP databases.

## AP database - Availability and partition tolerance
AP databases provide availability and partition tolerance but non consistency when a failure occurs. All nodes remain available, but some may return an older version of the data.
Typically uses eventual consistency, meaning all nodes will synchronize over time.

**Examples of AP databases**: CouchDB, Cassandra, and ScyllaDB

### Eventual Consistency
It is a relaxed form of consistency used in AP databases, where updates propagate asynchronously.

Eventual consistency is a guarantee that when an update is made in a distributed database, that update will be eventually reflected in all nodes that store the data, resulting in the same response every time that data is required.

## CA database - Consistency and availability
CA databases deliver consistency and availability, but it can’t deliver fault tolerance if any two nodes in the system have a partition between them (a failure of communication).

In real-world distributed systems, network partitions are inevitable, making pure CA systems impractical for large-scale applications.

However, in local, non-distributed databases, CA behavior is possible since no network partitions occur. For example, traditional Relational Databases (RDBMS) like MySQL and PostgreSQL when not deployed in a distributed mode

# Resources

CAP Theorem
- [https://www.ibm.com/topics/cap-theorem](https://www.ibm.com/topics/cap-theorem)
- [https://towardsdatascience.com/cap-theorem-and-distributed-database-management-systems-5c2be977950e](https://towardsdatascience.com/cap-theorem-and-distributed-database-management-systems-5c2be977950e)
- [https://www.scylladb.com/glossary/cap-theorem/](https://www.scylladb.com/glossary/cap-theorem/)
- [https://www.turing.com/kb/cap-theorem-for-system-design](https://www.turing.com/kb/cap-theorem-for-system-design)
- [https://www.baeldung.com/cs/db-base-meaning-cap](https://www.baeldung.com/cs/db-base-meaning-cap)
- [https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/](https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/)

Eventual Consistency
- [https://www.baeldung.com/cs/eventual-consistency-vs-strong-eventual-consistency-vs-strong-consistency](https://www.baeldung.com/cs/eventual-consistency-vs-strong-eventual-consistency-vs-strong-consistency)
