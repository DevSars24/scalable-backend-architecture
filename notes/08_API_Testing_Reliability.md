# 🧪 8. API Testing & Reliability (Advanced Notes)

## 🏗️ Testing Pyramid
- **Unit Testing (70%):** Tests a single function or class in complete isolation. Mocks dependencies (Mocking vs Stubbing). Extremely fast. Provides coverage.
- **Integration Testing (20%):** Tests how the application talks to the immediate infrastructure (Database, Redis).
  - *Pro-tip:* Never mock the DB in an integration test. Use **Testcontainers** (Docker containers spun up purely for the test run) to test real queries against a real Postgres container.
- **E2E / System Testing (10%):** Browser-based or fully deployed environment testing. Slow, brittle, but acts exactly like a real user.

## 🤝 Contract Testing (Pact)
- Solves the problem: "The backend changed a field name from `userId` to `user_id` and broke the mobile app."
- Backend and Frontend agree on a JSON "Contract".
- Whenever either side pushes code, CI runs a check against the contract to ensure no breaking schema changes were made. Eliminates the need to spin up entire E2E environments just to check basic payload compatibility.

## 📊 Performance Testing
- **Load Testing (k6, JMeter):** "Can we handle our *expected* peak traffic of 500 requests per second with < 200ms latency?"
- **Stress Testing:** Pushing the system until it BREAKS (e.g., 5000 requests per second). Identifies the exact bottleneck (Is it CPU? DB Connections? Memory?).
- **Spike Testing:** Simulating a sudden surge of traffic (e.g., Ticketmaster opening Taylor Swift sales). Tests how fast Auto-Scaling can react.
- **Soak Testing:** Running a steady, moderate load for 48 hours to discover deep Memory Leaks.

## 🔥 Chaos Engineering & Resilience
- Deliberately injecting failures into the production or staging systems to ensure the architecture actually survives.
- **Chaos Monkey (Netflix):** Randomly terminating EC2 instances during business hours. Forces engineers to build entirely stateless, auto-scaling, fault-tolerant logic.
- Testing network latency using tools like `Toxiproxy` to simulate a slow Database response. Does your Circuit Breaker actually trip?

## 🏥 Health Checks & Probes (Kubernetes)
- Standard `/health` endpoint is not enough.
- **Liveness Probe:** "Is the app totally dead?" (e.g. process is deadlocked). If it fails, K8s restarts the Pod.
- **Readiness Probe:** "Is the app ready to accept traffic?" (e.g. is the DB connection established, is the cache warmed?). If it fails, K8s removes it from the Load Balancer, but does *not* restart it.
- **Deep vs Shallow Checks:** Should `/health` check the DB? (Shallow: No, just app. Deep: Yes, query `SELECT 1`). Be careful: If the DB is slow, deep health checks will fail, K8s will kill *all* your instances, causing a massive outage!
