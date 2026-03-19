# 👨‍💻 10. System Design Mastery (Interview Gold)

## 🧩 High Level Design (HLD) Macro-Architecture
- The "Boxes and Arrows" phase.
- **Typical Flow:** Client → Route 53 (DNS) → Cloudflare (WAF/CDN) → API Gateway (AWS API Gateway/Kong) → Load Balancer → Auto-Scaling Group of Stateless App Servers (EC2/K8s/ECS).
- **Synchronous vs Asynchronous:** Decide what must happen immediately (Payment validation) vs what can happen in the background (Sending confirmation email via SQS/RabbitMQ).

## 🗃️ DB Tradeoffs (SQL vs NoSQL)
- The #1 question in System Design.
- **Use SQL (Postgres/MySQL) when:** Data is heavily relational, requires ACID transactions (Banking), and the schema is strict and understood upfront.
- **Use NoSQL (Cassandra/DynamoDB) when:** Extreme horizontal read/write scale is needed, data model is simple Key-Value or Wide-Column, eventual consistency is okay, and schema iterates rapidly.
- **Use Graph (Neo4j) when:** You are querying N-depth relationships (e.g., "Find friends of friends who liked my photo").

## 📈 Scalability Patterns
- **CQRS (Command Query Responsibility Segregation):** Splitting the read model from the write model. Writes go to a Postgres DB. Events sync the data into an Elasticsearch cluster optimized purely for massive complex search queries.
- **Strangler Fig Pattern:** Gradually replacing a legacy Monolith by putting an API Gateway in front, and slowly routing specific endpoints (`/api/users`) to new microservices while the rest goes to the monolith.
- **Outbox Pattern:** Safely publishing events in a Microservice. Write the domain change AND the event to the same DB in *one transaction*. A separate worker reads the event table and publishes to Kafka. Guarantees an event is never missed if Kafka is briefly down.

## 🛡️ Failure Handling & Resilience Patterns
- Dependencies WILL fail. You must design for it.
- **Retry with Exponential Backoff + Jitter:** If a 3rd party API times out, don't instantly retry (it will DDos them). Wait `1s`, then `2s`, then `4s`. Add *Jitter* (randomness) so not all blocked clients retry at the exact same millisecond.
- **Circuit Breaker:** If a downstream service fails 50% of requests in the last minute, trip the breaker to `OPEN`. Instantly fail any new requests (saving your API from hanging and exhausting threads). After 30s, go to `HALF-OPEN`, send one request to test if it recovered.
- **Bulkhead Pattern:** Isolate thread pools. If the User Service is slow, it shouldn't consume all threads and crash the separate Payment Service.

## 📚 Real-world Architectural Case Studies (MUST KNOW)
1. **WhatsApp (Erlang):** Handling millions of concurrent persistent WebSocket connections on a single machine due to Erlang's extremely lightweight processes and Actor model.
2. **Discord (Cassandra/ScyllaDB):** Migrating from MongoDB to Cassandra to handle trillions of chat messages. Why? Because Cassandra writes are appending logs continuously in memory/disk with zero locking, perfect for an infinite streaming chat system. ScyllaDB was the final C++ rewrite for extreme performance.
3. **Instagram/Twitter (Fanout on Write vs Read):**
  - Justin Bieber posts a tweet. *Fanout on Write* (pushing the tweet to 100M followers' timelines) takes hours and kills the DB.
  - Solution: Standard users use Fanout on Write. Mega-celebrities use *Fanout on Read* (Pull model). When you open your feed, it actively fetches standard user tweets from your pre-computed timeline cache + fetches Justin Bieber's timeline simultaneously and merges them.
