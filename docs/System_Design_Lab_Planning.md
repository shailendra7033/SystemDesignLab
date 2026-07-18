# System Design Lab - Planning & Decisions

## Purpose

This repository is **not** a product clone. It is a structured learning
journey to understand System Design by evolving one real backend
application.

## Primary Goal

Learn **why** architectural components are introduced instead of simply
learning **how** to use them.

Examples:

- Why Redis?
- Why a Queue?
- Why a Load Balancer?
- Why Horizontal Scaling?
- Why Background Workers?
- Why Docker?
- Why API Versioning?

Every architectural component must solve a real bottleneck that we
encounter while building the project.

---

## Repository Vision

Repository Name: **system-design-lab**

First Project: **URL Monitoring Platform**

The URL Monitoring Platform is only the learning vehicle.

---

## Technology Stack

| Component        | Choice                   | Rationale                                                                                     |
| ---------------- | ------------------------ | --------------------------------------------------------------------------------------------- |
| Language         | **Go**                   | Compiled, statically typed (familiar from C++). First-class concurrency. Language of infra.    |
| Router           | **Chi**                  | Lightweight, idiomatic, stdlib-compatible. No framework magic.                                |
| Database         | **PostgreSQL**           | Industry standard relational DB. Rich querying, JSONB, indexing.                              |
| Migrations       | **golang-migrate**       | Version-controlled schema changes from Phase 1.                                               |
| Containerization | **Docker + Compose**     | Reproducible dev environment. One command to spin up entire stack. Required for scaling phases.|
| Cache            | **Redis**                | Introduced only in Phase 7 when we measure a real DB bottleneck.                              |
| Message Queue    | **RabbitMQ**             | Introduced only in Phase 6 when we need to decouple workers.                                  |
| Observability    | **Prometheus + Grafana** | Introduced in Phase 9 for metrics and dashboards.                                             |
| Load Testing     | **k6**                   | Scriptable, generates clear reports.                                                          |

---

## Engineering Principles

1. Start simple.
2. Never over-engineer.
3. Measure first, optimize later.
4. Introduce only one new system design concept at a time.
5. Every architecture decision must be documented.
6. Every new component must solve a measurable problem.
7. Prefer understanding over using trendy technologies.
8. Containerize from day one — no "works on my machine" excuses.
9. Version your API — breaking changes will happen.

---

## Documentation Structure

```
docs/
├── phase-00-planning.md
├── phase-01-single-server.md
├── phase-02-background-worker-scheduler.md
├── phase-03-notifications.md
├── phase-04-worker-pool.md
├── phase-05-retry-strategy.md
├── phase-06-message-queue.md
├── phase-07-caching.md
├── phase-08-rate-limiting-auth.md
├── phase-09-observability.md
├── phase-10-load-balancer-scaling.md
├── phase-11-performance-testing.md
├── phase-12-architecture-review.md
├── architecture/
└── adr/
```

---

## Architecture Decision Record (ADR) Template

Every phase should answer:

- **Problem** — What bottleneck or limitation exists?
- **Current Architecture** — What does the system look like right now?
- **Symptoms** — How did we detect the problem? (metrics, logs, manual observation)
- **Measurements** — What numbers prove the problem? (p95 latency, error rate, throughput)
- **Alternatives Considered** — What other solutions could work?
- **Selected Solution** — What did we pick and why?
- **Trade-offs** — What complexity does this introduce?
- **Benchmarks** — Before vs. after measurements.
- **Future Improvements** — What would we do differently at 10x scale?

---

## Metrics to Track (Per Phase)

Every phase must record these before and after introducing a change:

| Metric         | How to Measure                  |
| -------------- | ------------------------------- |
| Response time  | p50, p95, p99 latency           |
| Throughput     | Requests per second             |
| Error rate     | % of 4xx/5xx responses          |
| DB query time  | Slow query log, EXPLAIN ANALYZE |
| Memory usage   | Container stats, runtime metrics|
| CPU usage      | Container stats, runtime metrics|
| Queue depth    | (Phase 6+) Messages waiting     |
| Cache hit rate | (Phase 7+) Hits vs misses       |

If you cannot show numbers proving a bottleneck, you do not add a new component.

---

## Engineering Journal

Maintain a weekly journal answering:

- What did I build?
- Why did I build it?
- What problem appeared?
- What alternatives did I evaluate?
- What metrics did I collect?
- What did I learn?
- What would I improve?

---

## Success Criteria

By the end of the project, I should be able to explain every major
system design concept using my own implementation and measurements
instead of textbook definitions.
