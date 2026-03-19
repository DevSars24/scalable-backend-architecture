# ⚡ 4. Caching Layer (Advanced Notes)

## 🧠 Redis Internals
- In-memory data store. Single-threaded architecture using I/O multiplexing (epoll). This means a slow O(N) command like `KEYS *` will block the entire Redis server! Always use `SCAN` instead.
- Supports data structures: Strings, Hashes, Lists, Sets, Sorted Sets.
- **Sorted Sets (ZSET):** Incredible for building scalable Leaderboards or Rate Limiters.

## 🗑️ Cache Invalidation
- *The hardest problem in computer science!*
- Strategies to clear old data when the DB changes.
- **Time-To-Live (TTL):** Evict after exactly X minutes. Very simple but might serve stale data.
- **Event-driven Invalidation:** Listen to DB events (e.g. via Debezium CDC or Application logic) and actively delete/update the Redis key.

## ♻️ Eviction Policies (LRU & LFU)
- What happens when Redis memory is 100% full? 
- **LRU (Least Recently Used):** Removes keys that haven't been read in the longest time. (Best default).
- **LFU (Least Frequently Used):** Removes keys that are generally requested less often.

## 🔄 Cache Strategies (Write-through vs Write-back vs Cache-Aside)
- **Cache-Aside (Lazy Loading):** App queries Cache. If miss, app queries DB, then writes to Cache. 
  - *Pros:* Only caches requested data. 
  - *Cons:* Cache miss penalty on first load. Cache Stampede risk.
- **Write-Through:** App writes to Cache AND DB at the same time. 
  - *Pros:* Data is always fresh. 
  - *Cons:* High write latency, cache is filled with data that might not be read.
- **Write-Back (Write-Behind):** App writes to Cache, returns success quickly. Async worker writes from Cache to DB later. 
  - *Pros:* Extreme write performance! 
  - *Cons:* High risk of data loss if the cache server crashes before sync.

## 🌐 Distributed Cache
- A single Redis node maxes out memory/CPU. Need a cluster.
- **Redis Cluster:** Fragments data into 16,384 Hash Slots. Supports high availability, automatic primary failover.
- **Memcached:** Extremely simple multi-threaded distributed cache. Best for basic key-value string caching, but lacks persistent data structures unlike Redis.

## ⚖️ Cache Consistency & The "Cache Stampede"
- **Cache Stampede (Thundering Herd):** A highly requested cache key expires. Suddenly, 50k requests hit the DB at the exact same millisecond. DB crashes.
- **Solution 1 (Mutex/Lock):** When key expires, let only ONE request hit the DB and lock the rest. Once completed, unlock and others read from cache.
- **Solution 2 (Probabilistic Early Expiration):** Refresh the cache *before* it actually expires randomly to space out DB hits.

## 🚀 Advanced Use Cases
1. **Session Store:** Storing stateful user sessions scaling horizontally.
2. **Ranking/Leaderboards:** Using `ZADD`, `ZREVRANGE`.
3. **Geospatial Queries:** Using `GEOADD`, `GEORADIUS` for finding nearest drivers/restaurants.
4. **Pub/Sub:** Lightweight message broadcasting (e.g., sending chat messages to multiple WebSocket servers).
