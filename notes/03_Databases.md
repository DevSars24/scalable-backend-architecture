# 🗄️ 3. Databases (Advanced Notes)

## 🔍 SQL Deep Dive
- **B-Tree vs Hash Indexes:** 
  - `B-Tree` is used for range queries (`>`, `<`, `BETWEEN`) and exact matches. 
  - `Hash` indexes are faster for exact matches (`=`) but cannot handle range queries.
- **ACID Properties:** Atomicity, Consistency, Isolation, Durability. Write-Ahead Logging (WAL) is the core mechanism ensuring Durability.
- **Explain Analyze:** The most powerful tool. Shows the query execution plan (Index Scan, Seq Scan, Hash Join). If you see `Sequential Scan` on a large table, you need an index.

## 📜 Indexing Strategy
- **Composite Indexes:** Indexing multiple columns (`CREATE INDEX idx_user_status ON users(status, created_at)`). Order matters! Follow the Left-Most Prefix Rule.
- **Covering Indexes:** An index that includes all the columns needed for the query, meaning the DB doesn't have to look up the actual table rows.
- **Index Selectivity:** High selectivity (unique values like `email`) is great. Low selectivity (boolean values like `is_active`) causes the DB to skip the index entirely sometimes.

## 🚀 Query Optimization
- Avoid `SELECT *`. Retrieve only what you need to reduce memory and network overhead.
- **N+1 Problem:** Querying the DB in a loop (e.g., getting users, then querying their posts one by one). Fix using `JOIN` or batching tools like `DataLoader`.
- **Pagination:** As discussed, switch from Offset to Cursor pagination to avoid large offset performance degradation.

## 🛡️ Transactions & Isolation Levels
1. **Read Uncommitted:** Dirty reads allowed. Fastest but dangerous.
2. **Read Committed:** Default in Postgres. Solves dirty reads.
3. **Repeatable Read:** Guarantees rows read multiple times in a transaction won't change. Can still have *Phantom Reads* (new rows added).
4. **Serializable:** Highest isolation. Transactions happen as if they were strictly sequential. Prone to aborts/deadlocks under high concurrency.

## 🔌 Connection Pooling
- Opening a DB connection is expensive (TCP handshake + DB auth). Use a pooler!
- Tools: **PgBouncer** (Postgres), **HikariCP** (Java).
- Sizing: Formula `connections = ((core_count * 2) + effective_spindle_count)`. Don't over-provision connections; 50 is often better than 500 for a busy DB.

## 📊 NoSQL Modeling
- **Single Table Design (DynamoDB):** Pre-compute queries. Store multiple entity types in one table using overloaded Partition and Sort Keys (`PK`, `SK`).
- No joins allowed. Optimize for reads, pay on writes.
- **Wide Column (Cassandra):** Great for time-series data. Requires deep understanding of Partition Keys to avoid hot partitions.

## 🔪 Sharding (Horizontal Partitioning)
- Splitting a large DB into smaller ones (Shards).
- **Hash-based Sharding:** Distribute data evenly based on `hash(user_id)`. Downside: Adding/removing shards requires massive data rehashing (unless using **Consistent Hashing**).
- **Range-based Sharding:** Easy to query ranges, but prone to **Hotspots** (e.g., putting all new 2024 data on one shard).

## 🔄 Replication
- **Master-Slave (Leader-Follower):** Writes to Master, async reads from Slaves. Highly scalable for read-heavy workloads.
- **Replication Lag:** The delay between a write on Master and it being visible on the Slave. Must build logic to avoid "read-after-write" issues (e.g., user updates profile and sees old data).
- **Master-Master:** High availability, but conflict resolution logic is brutal.

## 🚚 Data Migration
- Changing schema on 100 Million rows takes hours and locks tables.
- **Zero-Downtime Migration:**
  1. Add new column.
  2. Dual writes (write to old and new).
  3. Backfill old data via batch jobs.
  4. Switch read code to new column.
  5. Drop old column.
