# 🛠️ 9. DevOps & Production (Advanced Notes)

## 🐳 Docker Mastery
- **Containers vs VMs:** VMs package the OS, slow to boot. Containers share the host OS kernel, run in isolated namespaces/cgroups, boot in milliseconds.
- **Multi-Stage Builds:** Compile your Go or Java code in a heavy `Builder` image, then copy just the binary into a tiny, secure `Alpine` or `Distroless` image. Drastically reduces attack surface and image size.
- **The PID 1 Problem:** The main process in a Docker container runs as PID 1. If you run a shell script that starts a Node app, signals (SIGTERM) won't reach Node, causing non-graceful shutdowns. Always use `exec` or specialized tools like `dumb-init`.

## 🔄 CI/CD (Continuous Integration / Continuous Deployment)
- **CI (GitHub Actions, GitLab CI):** On Pull Request. Run Linters -> Run Unit Tests -> Run Integration Tests -> Build Docker Image.
- **CD (ArgoCD, Spinnaker):** Deploy the new Docker Image.
- **Immutability Principle:** Build the Docker image EXACTLY ONCE. Promote that exact same image from Dev -> Staging -> Prod. Never rebuild it per environment.

## ☸️ Kubernetes (K8s) Basics
- The orchestrator for thousands of containers.
- **Pod:** The smallest unit (usually 1 Pod = 1 Container).
- **Deployment:** Manages Pods. Guarantees that X number of Pods are always running. Handles rolling updates (zero downtime deployments).
- **Services:** Internal load balancers. Pod IPs change constantly. Services provide a stable IP/DNS name.
- **Ingress / Gateway API:** The entry point from the outside internet (routing HTTP traffic based on hostnames/paths) into your Services.

## 🔐 Secrets Management
- **Rule #1:** NEVER hardcode credentials in code or commit `.env` files to git.
- Pass secrets purely as Environment Variables (`process.env.DB_PASS`) at runtime.
- **Tools:** HashiCorp Vault, AWS Secrets Manager, K8s Secrets. A Vault can dynamically rotate database passwords every 12 hours so leaked credentials are automatically invalidated.

## 📡 Observability (The Three Pillars)
1. **Metrics (Prometheus + Grafana):** "Is there a problem?" Pull-based time-series data. (CPU %, Memory %, Requests per minute, P99 Latency).
2. **Logs (ELK, Datadog):** "What was the error message?" Centralized, structured JSON logging parsed by Logstash/FluentBit. Includes stack traces.
3. **Distributed Tracing (Jaeger, OpenTelemetry):** "Where is the bottleneck?" Request hits API Gateway -> Auth Service -> Cart Service -> DB. Tracing passes a `Trace-ID` everywhere to visualize the exact millisecond breakdown of a request across 10 different network hops.

## ☁️ Cloud Infrastructure (AWS/GCP/Azure)
- **IaaS (Infrastructure):** EC2. You manage OS, patching, scaling.
- **PaaS (Platform):** Elastic Beanstalk, Heroku. They manage OS, you give code.
- **FaaS (Serverless):** AWS Lambda. Runs functions on demand per event. No idle costs.
- **Managed Services:** 
  - *RDS / Aurora:* Managed Postgres. 
  - *ElastiCache:* Managed Redis. 
  - *SQS/SNS:* Managed queues. *Use managed services over hosting your own database unless you have a dedicated ops team.*
