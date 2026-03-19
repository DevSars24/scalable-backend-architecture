# 🏢 5. Architecture Styles (Advanced Notes)

## 🏛️ Monolithic
- **Concept:** All logic (Auth, Payments, Inventory) runs in a single process and usually shares ONE database.
- **Pros:** Extremely fast inter-process communication (function calls). Easy to debug, test, and deploy initially.
- **Cons:** Codebase becomes a "Big Ball of Mud". Tight coupling. A memory leak in a minor feature can bring down the entire app. Scaling is all-or-nothing (vertical or heavy horizontal replication).

## 🧱 Modular Monolith
- **Concept:** Still a single process, but strictly separated into modules (e.g. `ecommerce.auth.*`, `ecommerce.orders.*`). Modules communicate via internal API boundaries/contracts.
- **Pros:** Combines the simplicity of a Monolith deployment with the code quality of Microservices. Highly recommended by Shopify/Stripe as a starting point.
- **Cons:** Discipline is required to prevent modules from direct database reads across boundaries.

## 🔗 Microservices Architecture
- **Concept:** Small, independently deployable services around specific business domains. They communicate via Network (REST/gRPC/Messaging).
- **Pros:** 
  - Independent scaling (scale only the heavy feature).
  - Fault isolation (if search service goes down, checkout service still works).
  - Technology freedom (use Node.js for IO, Python for AI, Go for core processing).
- **Cons:** 
  - Network latency!
  - **Distributed Transactions:** Very hard to implement (Saga pattern).
  - High operational complexity (Needs K8s, Tracing, API Gateways).

## 🚀 Event-Driven Architecture (EDA)
- **Concept:** Services emit events ("UserRegistered") onto an Event Bus (Kafka, EventBridge). Other services subscribe and react independently.
- **Pros:** Decouples services beautifully. Highly resilient and scalable to extreme throughput.
- **Cons:** **Eventual Consistency**. Extremely difficult to debug when an event is lost or a consumer fails mid-process. Always requires Idempotent consumers.

## ⚡ Serverless (FaaS)
- **Concept:** Running code on highly ephemeral containers (AWS Lambda) which scale perfectly from 0 to 10k requests per second.
- **Pros:** Zero idle costs. No infrastructure to manage. Great for background jobs, Webhooks, or bursty traffic.
- **Cons:** 
  - **Cold Starts:** First request takes >1s to spin up. 
  - Vendor lock-in. Very hard to monitor and test locally. Connection exhaustion to monolithic databases.

## 🛑 Hexagonal Architecture (Ports and Adapters)
- **Concept:** Deep isolation of core business logic from outside frameworks/databases.
- **Inner Core:** Pure domain logic without knowing about HTTP or SQL.
- **Ports (Interfaces):** The expected inputs (Inbound) and expected outputs (Outbound Repository interfaces).
- **Adapters:** The implementations (e.g. `PostgresUserRepository`, `ExpressHTTPController`).
- **Pros:** Superb testability. You can swap out a database (Postgres to MongoDB) without touching a single line of your domain logic.
