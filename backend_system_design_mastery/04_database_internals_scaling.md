# Part 4 — Database Internals & Scaling 🧱

> **How modern databases like MongoDB, PostgreSQL, and Cassandra actually store and move your data.**

---

## 26. Replication

### 💡 One-Line Definition
Copying data from one server (leader) to multiple other servers (followers) to ensure **High Availability**.

### 🏢 Real-World Application: Amazon Orders
When you buy a product, your order is written to a "Leader" DB in Ohio. That data is immediately **replicated** to "Follower" DBs in California and Oregon. If Ohio's server dies, California takes over instantly.

### 🧠 Detailed Technical Explanation
*   **Synchronous Replication**: Wait for all followers to confirm. (Safe but Slow).
*   **Asynchronous Replication**: Don't wait for followers. (Fast but Risky).
*   **Leaderless (Quorum)**: Any node can take writes (e.g., Cassandra).

---

## 27. Partitioning & 28. Sharding

### 💡 One-Line Definition
**Partitioning**: Splitting a large table into smaller pieces inside the *same* database.  
**Sharding**: Splitting a large table across *entirely different* database servers.

### 🏢 Real-World Application: Twitter User Data
Twitter has 450 million users. Storing all of them on one hard drive is impossible. They **Shard** users based on `user_id`. 
*   Users 1 to 10M → DB Server A.
*   Users 10M to 20M → DB Server B.

### 🧠 Detailed Technical Explanation
**Methods**:  
1.  **Key-based Sharding**: Using a hash of the ID (most common).
2.  **Range-based Sharding**: Users with names A-M go to Server 1.
3.  **Directory-based**: A separate service "maps" where every user lives.

---

## 29. Indexing

### 💡 One-Line Definition
Creating a **lookup table** (like an index in a physical book) to find rows quickly without reading every single row.

### 🏢 Real-World Application: Swiggy Search
When you search for "Pizza," Swiggy doesn't search through 1 million restaurant descriptions. It uses an **Index** on the `cuisine` column to show you results in 50ms.

### 🧠 Detailed Technical Explanation
*   **Primary Index**: Unique, usually on the ID.
*   **Secondary Index**: On common columns like `email` or `city`.
*   **Clustered Index**: The actual data is stored in the order of the index.

---

## 30. Denormalization

### 💡 One-Line Definition
Intentionally **duplicating data** in multiple tables to avoid slow "Joins" and speed up Reads.

### 🏢 Real-World Application: Instagram News Feed
When you see a post, you see the `username` next to it. Instead of searching for the `user_id` inside the `users` table for every post, Instagram stores the `username` directly inside the `posts` table (Denormalization).

### 🧠 Detailed Technical Explanation
*   **Normalization (SQL way)**: No duplication. Saves space. Slow reads (many joins).
*   **Denormalization (NoSQL way)**: High duplication. Wastes space. Fast reads (one query).

---

## 31. ACID vs 32. BASE

### 💡 One-Line Definition
**ACID**: Guarantees **Strict Consistency** (used in Banking).  
**BASE**: Guarantees **Eventual Consistency** (used in Social Media).

### 🏢 Real-World Application: Bank Transfer vs Facebook Comments
*   **ACID (PostgreSQL)**: You send $100. Either the money leaves your account and arrives in your friend's, or nothing happens. There is no middle ground. 
*   **BASE (DynamoDB)**: You post a comment. It might show up for you now, but for your friend in 5 seconds. This is "Basically Available."

### 🧠 Detailed Technical Explanation
*   **ACID**: Atomicity, Consistency, Isolation, Durability.
*   **BASE**: Basically Available, Soft-state, Eventually-consistent.

---

## 79. LSM Trees vs 80. B-Trees

### 💡 One-Line Definition
**B-Trees**: Optimized for fast **Reads** (SQL Databases).  
**LSM Trees**: Optimized for fast **Writes** (NoSQL Databases like Cassandra/LevelDB).

### 🏢 Real-World Application: PostgreSQL vs Bigtable
PostgreSQL uses **B-Trees** to retrieve rows instantly. Google Bigtable uses **LSM Trees** to handle billions of log entries written every second.

### 🧠 Detailed Technical Explanation
*   **B-Tree**: A balanced search tree. Finding a row takes `O(log n)`.
*   **LSM (Log-Structured Merge) Tree**: Appends data to a memory buffer (fast) and merges to disk later.

---

## 81. Merkle Trees

### 💡 One-Line Definition
A "Hash Tree" used to quickly verify that **two copies of a dataset are identical**.

### 🏢 Real-World Application: Bitcoin & Git
Git uses **Merkle Trees** to compare two branches. Instead of comparing 50,000 files, it compares one "Top Hash." If the Top Hash matches, the files are identical.

### 🧠 Detailed Technical Explanation
Every leaf is a hash of a data block. Every parent is a hash of its children hashes. If one bit of data changes at the bottom, the **Root Hash** at the top changes completely.

---

## 82. Bloom Filter

### 💡 One-Line Definition
A fast, memory-efficient way to ask: **"Is this item definitely NOT in the database?"**

### 🏢 Real-World Application: Medium & Browsers
*   **Medium**: Shows you "Recommended for you." To avoid showing you a story you've already read, it uses a **Bloom Filter**.
*   **Chrome Cache**: Before checking the disk for a file, it checks a Bloom Filter to see if the file exists at all.

### 🧠 Detailed Technical Explanation
*   If it says **"No"**, the item is 100% not there.
*   If it says **"Yes"**, it might be there (False Positive). But it saves 90% of useless DB queries.

---

## 83. HyperLogLog

### 💡 One-Line Definition
A mathematical algorithm used to **estimate** the count of unique elements in massive datasets using almost zero memory.

### 🏢 Real-World Application: YouTube Unique Viewers
A video has 100 million views. Counting exactly how many *unique* IP addresses watched it using a standard Set would take GBs of RAM. **HyperLogLog** estimates it with 99% accuracy using only 12KB of memory.

### 🧠 Detailed Technical Explanation
It uses the probability of seeing a sequence of zeros in the leading bits of hashes to calculate the number of unique items. 

---

## ✅ Summary Checklist
- [ ] Replication (Leader/Follower)
- [ ] Sharding (Divide the DB)
- [ ] Indexing (Fast search)
- [ ] ACID vs BASE (Strict vs Eventual)
- [ ] B-Trees (Read heavy)
- [ ] LSM Trees (Write heavy)
- [ ] Merkle Trees (Comparison)
- [ ] Bloom Filter (Existence check)
- [ ] HyperLogLog (Approximate counting)
