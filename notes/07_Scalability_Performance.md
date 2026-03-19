# 🚀 7. Scalability & Performance (Advanced Notes)

## ⚖️ Load Balancing
- Distributes incoming network traffic across a group of backend servers.
- **Layer 4 vs Layer 7:**
  - `Layer 4 (Transport)`: Fast, looks only at IP and Port (e.g. AWS Network LB).
  - `Layer 7 (Application)`: Slower, but can route traffic based on HTTP Headers, Cookies, or Path (e.g. route `/api` to EC2 and `/static` to S3). Excellent for microservices (AWS ALB, Nginx).
- **Algorithms:** Round Robin (default), Least Connections (best for long living connections like WebSockets), IP Hash (ensures sticky sessions).

## 🔀 Reverse Proxy (Nginx / HAProxy)
- Sits in front of the web servers and shields them.
- Uses: Load Balancing, SSL/TLS Termination (CPU heavy encryption happens here instead of the app server), Caching static assets, Compression (Gzip/Brotli).

## 📈 Auto-Scaling (Horizontal vs Vertical)
- **Vertical (Scale-Up):** Add more RAM and CPU to the single server. Easy, requires no code changes. Has an absolute physical limit and massive downtime when upgrading.
- **Horizontal (Scale-Out):** Add more identical servers. Infinite scale, high availability. 
- *Caveat:* The application code **MUST** be stateless! No storing JWTs or uploads on the local disk. Use S3 for uploads, Redis for sessions.

## 🛡️ Rate Limiting & Abuse Prevention
- Used to prevent DDoS attacks and scraping, and maintain fair usage across users.
- **Token Bucket:** Add tokens at a steady rate. Burst limits allowed.
- **Leaky Bucket:** Add requests to a queue. Process strictly at a constant rate. Smooths out traffic.
- **Fixed Window:** Allow X requests per minute boundary (00:00 to 00:01). Issue: 2x traffic can hit precisely at boundary switch.
- **Sliding Window:** Tracks exactly how many requests occurred in the exact sliding `(T - 60 seconds)` block. Very expensive but accurate (Use Redis Sorted Sets).

## 🛑 Backpressure & Load Shedding
- If your system cannot handle traffic, rejecting traffic is better than crashing fully.
- **Backpressure:** The system tells upstream systems "stop sending me data, I'm too full". Often used in reactive streams or message queues.
- **Load Shedding:** If the server CPU is >95%, instantly drop all non-critical API requests (e.g., return `503 Service Unavailable` for loading likes, but process checkout payments). 

## 🌍 Content Delivery Network (CDN) & Edge Computing
- **CDN:** Servers distributed globally holding physical copies of static assets (Images, JS/CSS).
- Decreases latency dramatically for users across the world.
- **Edge Computing (Cloudflare Workers, Lambda@Edge):** Running lightweight serverless functions directly on the CDN POP closest to the user. E.g., validating a JWT before it even hits your primary datacenters, or A/B testing logic.

## ⚡ Latency Optimization Checklist
1. **DB Indexes:** Missing indexes are the #1 cause of slow latency.
2. **N+1 Queries:** Fast individually, disastrous sequentially.
3. **Payload Compression:** Gzip/Brotli your JSON APIs.
4. **Caching Strategy:** Put reads on Redis. Put static objects on CDNs.
5. **Connection Pools:** Reusing HTTP connections and Database connections to avoid slow handshakes!
