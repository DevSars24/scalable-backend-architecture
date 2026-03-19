# 🌐 6. Distributed Systems (Advanced Notes)

## ⚖️ The CAP Theorem
- **A distributed data store can only guarantee two out of three:**
  1. **Consistency (C):** Every read receives the most recent write, or an error.
  2. **Availability (A):** Every request receives a non-error response, but without guarantee it contains the most recent write.
  3. **Partition Tolerance (P):** The system continues to operate despite network failures dropping messages between nodes.
- *Reality:* Networks **will** fail, so you must have (P). You have to choose between CP (e.g. MongoDB/Redis usually) or AP (e.g. Cassandra/DynamoDB).

## 🔄 Consistency Models
1. **Strong Consistency:** Once a complete write is acknowledged, any subsequent read will return that value. Slow, but safe (like Banking).
2. **Eventual Consistency:** Reads might return stale data immediately after a write, but eventually, all nodes will sync. Fast and highly scalable (like Instagram likes).
3. **Causal/Read-your-own-writes Consistency:** Prevents a user from updating a profile and then seeing the old profile on refresh, while accepting other users might take seconds to see the change.

## 👑 Leader Election & Consensus (Raft/Paxos)
- **Problem:** If multiple nodes try to process the exact same scheduled task (e.g. sending emails), it happens multiple times!
- **Leader Election:** The nodes talk to each other to nominate exactly ONE leader. If the leader dies, they vote for a new one. (e.g., ZooKeeper, ETCD, or Redis Redlock).
- **Consensus Algorithms:** Raft and Paxos ensure nodes agree on the exact sequence of data changes (the "log"). Without consensus, distributed systems fall apart into split-brains.

## 📨 Message Queues & Log Structures
- **Message Queues (RabbitMQ/ActiveMQ):** Push model. "Dumb" pipes, "Smart" consumers. Excellent for routing tasks to specific workers (e.g. video processing worker vs image processing worker).
- **Log Structures (Apache Kafka):** Pull model. A distributed, append-only commit log. "Smart" pipe, "Dumb" consumers.
- **Partitions:** The secret to Kafka’s scalability. Data is split into partitions; each partition guarantees strict ordering. Consumer Groups read from partitions without blocking each other.

## 💾 The SAGA Pattern (Distributed Transactions)
- In microservices, there is no generic database `COMMIT` or `ROLLBACK` for multiple databases.
- **Choreography SAGA:** Service A publishes an event. Service B listens, does work, publishes success event. If it fails, it publishes a failure event, and Service A must trigger "Compensating Transactions" (undoing the original work).
- **Orchestration SAGA:** A central controller tells Service A, then Service B what to do. Easier to debug but creates a single point of failure bottleneck.

## 🛡️ Idempotency & Delivery Guarantees
- **At-most-once:** Event sent once, might be dropped. Perfect for telemetry data.
- **At-least-once:** Guaranteed to arrive, but might arrive multiple times! (Standard in Kafka/RabbitMQ).
- **Exactly-once:** Arrives exactly once. Very complex and resource heavy.
- *Best Practice:* Use At-least-once delivery, but make all handlers **Idempotent**. (Send a unique `x-idempotency-key` UUID; check in Redis/DB if it was already processed before doing the actual DB write).

## 🔒 Distributed Locks
- Used to prevent concurrent modifications when scaling out to 10+ identical containers.
- Implemented usually via Redis `SETNX` (Set if Not Exists) + Time-To-Live (TTL).
- Be careful with TTLs: If a job takes longer than the lock TTL, the lock expires, another worker grabs it, and you get dual processing. Ensure workers renew locks constantly if they're still working (Redisson lock watchdog).
