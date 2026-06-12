# Microservices Architecture

> Tags: #scalability #reliability #tradeoffs

Microservices is an architectural style that structures an application as a collection of small, independently deployable services modeled around business capabilities.

---

## Core Principles

1. **Single Responsibility** — each service owns one bounded context
2. **Decentralized Data Management** — each service manages its own database
3. **Design for Failure** — assume downstream services will fail
4. **Infrastructure Automation** — CI/CD pipelines per service
5. **Smart endpoints, dumb pipes** — business logic lives in services, not middleware

---

## Key Patterns

### Service Communication

| Pattern | When to Use |
|---------|-------------|
| **Synchronous REST/gRPC** | Real-time request/response, low latency required |
| **Async Messaging (events)** | Decoupled workflows, eventual consistency is acceptable |
| **GraphQL Federation** | Multiple services, single API contract for clients |

### Data Management

- **Database per Service** — prevents tight coupling at the data layer
- **Saga Pattern** — manages distributed transactions via a sequence of local transactions
  - *Choreography*: each service publishes events, others react
  - *Orchestration*: a central coordinator (saga orchestrator) drives the flow
- **CQRS** — separate read (Query) and write (Command) models for scalability

### Resilience

- **Circuit Breaker** — stops cascading failures; states: Closed → Open → Half-Open
- **Bulkhead** — isolate failures to a pool of resources (e.g., separate thread pools)
- **Retry with Exponential Backoff** — `delay = base * 2^attempt + jitter`
- **Timeout** — always set deadlines on outgoing calls

---

## Service Discovery

- **Client-side**: client queries a registry (e.g., Consul, Eureka) and picks an instance
- **Server-side**: a load balancer (e.g., AWS ALB, Nginx) does the lookup
- **DNS-based**: Kubernetes Services use DNS for discovery within a cluster

---

## API Gateway

Responsibilities:
- Routing / path rewriting
- Auth (JWT validation, OAuth 2.0)
- Rate limiting and throttling
- Request aggregation (Backend for Frontend pattern)
- TLS termination

Examples: AWS API Gateway, Kong, Nginx, Traefik

---

## Observability (The Three Pillars)

1. **Logs** — structured (JSON), correlated by `trace_id` / `request_id`
2. **Metrics** — RED (Rate, Errors, Duration) per service; USE (Utilization, Saturation, Errors) for resources
3. **Traces** — distributed tracing with OpenTelemetry; visualize with Jaeger or Zipkin

---

## Deployment Strategies

| Strategy | Zero Downtime | Rollback Speed | Complexity |
|----------|:---:|:---:|:---:|
| Rolling Update | ✅ | Medium | Low |
| Blue/Green | ✅ | Fast | Medium |
| Canary | ✅ | Fast | High |
| Feature Flags | ✅ | Instant | Medium |

---

## Common Pitfalls

- **Distributed monolith** — services tightly coupled via synchronous chains; one failure cascades
- **Chatty services** — too many fine-grained calls; aggregate at the service boundary
- **Shared database** — breaks service autonomy and independent deployability
- **No contract testing** — consumer-driven contracts (Pact) prevent silent breakage
- **Missing idempotency** — retries must be safe; use idempotency keys on mutations

---

## When NOT to Use Microservices

- Small team (< 5–8 engineers) — operational overhead outweighs benefits
- Early-stage product with unstable domain model — start as a modular monolith
- Strong transactional consistency required across multiple entities

---

## References

- [Martin Fowler — Microservices](https://martinfowler.com/articles/microservices.html)
- [12-Factor App](https://12factor.net/)
- [AWS Microservices Lens (Well-Architected)](https://docs.aws.amazon.com/wellarchitected/latest/microservices-lens/welcome.html)
- Book: *Building Microservices* — Sam Newman
