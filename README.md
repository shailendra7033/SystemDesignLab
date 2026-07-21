# System Design Lab

A hands-on learning journey through system design — by building, breaking, and evolving a real backend system.

> **Philosophy:** Every architectural component is introduced only when a real bottleneck demands it. No cargo-culting.

**Stack:** Go · PostgreSQL · Docker · Swagger · Redis · RabbitMQ · Nginx · Prometheus · Grafana

---

## Episodes

| #  | Topic                              | Key Concepts                                                  |
| -- | ---------------------------------- | ------------------------------------------------------------- |
| 0  | Planning & Environment Setup       | HLD vs LLD, Docker, 12-factor app, API-first thinking         |
| 1  | Single Server MVP                  | REST APIs, DB schema design, layered architecture, JWT auth   |
| 2  | Background Worker & Scheduler      | Goroutines, async processing, graceful shutdown                |
| 3  | Notifications                      | Event-driven design, state change detection, idempotency      |
| 4  | Worker Pool                        | Bounded concurrency, fan-out/fan-in, channels                 |
| 5  | Retry Strategy                     | Exponential backoff, jitter, thundering herd, circuit breaker |
| 6  | Message Queue                      | Producer-consumer, delivery guarantees, dead letter queues     |
| 7  | Caching                            | Cache-aside, TTL, invalidation strategies, cache stampede      |
| 8  | Rate Limiting & API Versioning     | Token bucket, sliding window, versioning strategies            |
| 9  | Observability                      | Logs, metrics, traces, RED/USE methods, SLIs/SLOs             |
| 10 | Load Balancer & Horizontal Scaling | SPOF, stateless services, connection pooling, LB algorithms   |
| 11 | Performance Testing                | Load/stress/soak/spike tests, capacity planning               |
| 12 | Architecture Review                | Evolution reflection, 10x/100x planning, lessons learned      |

---

## Project: URL Monitoring Platform

A platform to monitor website uptime, latency, and availability. The product is intentionally simple — the learning comes from evolving the architecture.

## Learning Workflow

Every phase follows this loop — **no shortcuts**:

```
1. Challenge    → Read challenges/phase-XX-challenge.md (hints, not answers)
2. Think        → Research, sketch, write your own answers
3. Review       → Share answers for feedback (mistakes = learning)
4. Build        → Only now do we write actual docs and code together
```

The `docs/` folder contains reference answers — don't look until you've attempted the challenge.

## Rules

- Measure before optimizing
- One new concept per phase
- Every decision documented in an ADR
- No new tech without proving a bottleneck exists
- **Think first, build second** — no copying without understanding

---

## Learning Resources

Use this section to keep links for articles, docs, and tools referenced during the lab.

### Articles and Docs

- Database indexing overview: https://medium.com/@akashsdas_dev/database-indexing-e10362624ed3
- DB diagram tool: https://dbdiagram.io/
- OpenAPI tags guide: https://learn.openapis.org/specification/tags.html

### VS Code Extensions Used

- OpenAPI by 42Crunch
- DBML

---

📄 [Planning](System_Design_Lab_Planning.md) · 📄 [Roadmap](System_Design_Lab_Roadmap.md) · 📄 [Project Setup](docs/phase-00-projectsetup.md) · 📄 [Current Challenge](challenges/phase-01-challenge.md) · 📄 [Your Answers](challenges/answers/)
