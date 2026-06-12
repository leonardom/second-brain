# Designing Data-Intensive Applications

**Author:** Martin Kleppmann
**Year:** 2017
**Rating:** ⭐⭐⭐⭐⭐
**Category:** Distributed Systems / Data Engineering

---

## Summary

This book is arguably the best single resource on modern data systems. It covers the foundations of reliable, scalable, and maintainable data-intensive systems — from storage engines and query languages to distributed consensus and stream processing.

---

## Key Ideas

### Part 1: Foundations of Data Systems

**Reliability, Scalability, Maintainability**
- *Reliability*: system works correctly even when things go wrong (hardware faults, software bugs, human errors)
- *Scalability*: ability to handle increased load (requests, data volume, complexity)
- *Maintainability*: system is easy to operate, understand, and evolve

**Percentiles over averages** — p99 latency is what your slowest 1% of users experience. Optimize for percentiles, not averages, when setting SLOs.

**Data Models**
- Relational: good for many-to-many relationships, complex queries, join-heavy workloads
- Document: good for self-contained, hierarchical data (schema flexibility)
- Graph: good for highly connected data (social networks, recommendations, fraud detection)

**Storage Engines**
- *Log-structured* (LSM-trees): fast writes, compaction in background — used by Cassandra, LevelDB, RocksDB
- *Page-oriented* (B-trees): predictable read performance — used by most relational DBs

### Part 2: Distributed Data

**Replication**
- *Leader-follower* (single-leader): simple, strong consistency on reads from leader; failover complexity
- *Multi-leader*: write anywhere, conflict resolution needed; good for geo-distributed writes
- *Leaderless* (Dynamo-style): high availability, eventual consistency; quorum reads/writes (R + W > N)

**Consistency Models** (weakest → strongest)
1. Eventual consistency
2. Read-your-writes consistency
3. Session consistency
4. Monotonic reads
5. Linearizability (strongest — every op appears to take effect atomically at a single point in time)

**Partitioning (Sharding)**
- By key range: efficient range scans; risk of hot spots
- By hash of key: even distribution; loses range scan ability
- Consistent hashing: minimizes rebalancing when nodes join/leave

**CAP Theorem** (Brewer, 2000)
- In a network partition, choose between Consistency and Availability
- Most systems choose AP (Cassandra, DynamoDB) or CP (HBase, ZooKeeper)
- CP doesn't mean "always consistent" — it means unavailability in partition rather than inconsistency

**ACID vs BASE**
- ACID: Atomicity, Consistency, Isolation, Durability — traditional RDBMS guarantee
- BASE: Basically Available, Soft state, Eventually consistent — NoSQL trade-off

**Transactions & Isolation Levels** (weakest → strongest)
1. Read Uncommitted
2. Read Committed
3. Repeatable Read
4. Serializable

**Distributed Transactions**
- 2PC (Two-Phase Commit): coordinator + participants; blocking on coordinator failure
- Sagas: sequence of local transactions with compensating transactions on failure (preferred in microservices)

### Part 3: Derived Data

**Batch Processing (MapReduce)**
- Process large datasets in parallel by splitting into map and reduce phases
- Fault tolerant: retry individual tasks
- Tools: Hadoop, Spark (in-memory), Flink

**Stream Processing**
- Process events as they arrive (low latency vs batch)
- Exactly-once semantics is hard: at-least-once + idempotent consumers is common
- Stream-table duality: a stream is a changelog of a table; a table is an accumulated snapshot of a stream
- Tools: Kafka Streams, Flink, Spark Structured Streaming

**The Lambda Architecture** (now largely superseded)
- Batch layer (correct, slow) + Speed layer (approximate, fast) = Serving layer
- Replaced by the Kappa Architecture: streaming only, with reprocessing capability

---

## Actionable Takeaways

1. **Design for failure** — assume hardware will fail; use replication and redundancy
2. **Understand your consistency requirements** before choosing a database
3. **Beware of distributed transactions** — prefer sagas and idempotency over 2PC
4. **Use the right tool**: relational for complex queries + ACID; document for flexible schema; graph for connected data
5. **Monitor p99 latency**, not averages, for SLOs
6. **Event sourcing** (log as source of truth) enables audit trails, replay, and derived views
7. **Idempotency** is the key to handling at-least-once delivery safely

---

## Memorable Quotes

> "The complexity of software comes from the fact that it must be reliable, scalable, and maintainable — all at the same time."

> "A single-leader replication setup is like a monarchy: there can only be one king at a time, and succession can be messy."

> "Transactions are a way of grouping several reads and writes together into a logical unit. Either all succeed or all are aborted."

> "The truth is the log. The database is a cache of a subset of the log."

---

## See Also

- [Event-Driven Architecture notes](../../system-design/event-driven.md) — CQRS, event sourcing
- [Microservices notes](../../system-design/microservices.md) — Saga pattern, distributed data
