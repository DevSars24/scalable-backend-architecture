# 🧠 2. Backend Core (Advanced Notes)

## 🌐 HTTP Internals
- **TCP/IP Handshake:** Understand the 3-way handshake (`SYN`, `SYN-ACK`, `ACK`) and how latency impacts TTFB (Time to First Byte).
- **HTTP/1.1 vs HTTP/2 vs HTTP/3:**
  - `HTTP/1.1`: Head-of-line blocking, plain text, Keep-Alive.
  - `HTTP/2`: Multiplexing, Binary framing, Server Push.
  - `HTTP/3`: Uses QUIC (UDP instead of TCP), 0-RTT handshakes, no head-of-line blocking at the transport layer.
- **Connection Keep-Alive:** Reusing existing TCP connections to avoid expensive handshake costs for subsequent API calls.

## 🔄 REST vs GraphQL
- **REST:** Resource-oriented, standard HTTP methods (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`). Great for cacheability (via ETags/CDN) but risks over/under-fetching data.
- **GraphQL:** Client specifies exactly what it needs. Solves over/under-fetching.
  - *Challenge:* `N+1 Problem` in resolvers and poor HTTP caching capability out of the box. Use tools like **DataLoader** to batch and cache DB queries.

## 📐 API Design
- **Idempotency:** Making sure an API can be safely retried without side effects (e.g., repeating a `POST /checkout` with an `Idempotency-Key` header).
- **Pagination Strategy:**
  - *Offset Pagination* (`LIMIT 10 OFFSET 1000`): Bad for performance on large tables.
  - *Cursor Pagination* (`WHERE id > cursor LIMIT 10`): Extremely performant at scale.
- **HATEOAS:** Hypermedia as the Engine of Application State, useful for discoverability but rarely used practically in high-throughput microservices.

## 🔐 Authentication & Authorization
- **JWT (JSON Web Tokens):** Stateless authentication. Do not put sensitive data in the payload. Handle **token revocation** using a short expiration time and a Redis blacklist.
- **OAuth 2.0 & OIDC:** Understand the Authorization Code Flow (with PKCE for SPAs/Mobile).
- **Refresh Tokens:** Store securely (HttpOnly cookies or secure local storage) to get new short-lived access tokens without asking the user to log in again.

## 🧩 Middleware Architecture
- **Chain of Responsibility Pattern:** Intercepting requests before they reach the controller.
- Use cases: Auth validation, logging, tracing (assigning `Trace-ID`), rate-limiting, and payload sanitization.

## 🛡️ Input Validation
- Always validate on the backend. Never trust the client.
- **Sanitization vs Validation:** Validation ensures data shape/type; Sanitization cleanses malicious inputs to prevent SQL Injection (SQLi) and XSS.
- Tools: Joi, Zod (Node.js), Hibernate Validator (Java).

## 🚨 Error Handling Strategy
- **Global Error Handler:** Centralized mechanism (e.g., Express `(err, req, res, next)` or Spring `@ControllerAdvice`) to map specific exceptions to correct standard HTTP status codes.
- Never expose internal stack traces to the client. Returns structured JSON: `{ "error": "NOT_FOUND", "message": "User not found", "correlationId": "xyz" }`.

## 📡 Logging & Observability
- **Structured Logging:** Log in JSON format so tools like ELK (Elasticsearch, Logstash, Kibana) or Datadog can query them easily.
- **Correlation IDs:** Always attach a `x-correlation-id` to every incoming request. Pass this ID across all microservices so you can trace the entire lifecycle of a request if it fails.

## 🔀 API Versioning
- **URI Path:** `/api/v1/users` (Most common).
- **Header Versioning:** `Accept-Version: 1.0` (Cleaner URIs, but harder to cache via CDNs).
- **Query Parameter:** `/api/users?version=1` (Rarely used in prod).
- Best practice: Version aggressively on breaking changes, support old versions using deprecation warnings (`Warning: 299`).
