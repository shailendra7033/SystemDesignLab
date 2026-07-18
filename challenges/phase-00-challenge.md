# Phase 0 Challenge — Planning & Environment Setup

> **Status:** Completed (2026-07-18)
>
> This phase was finalized during initial planning. The challenge is
> documented here so the learning loop is consistent across all phases.

---

## Challenge 1: Why This Project?

Before writing any code, you need to justify the project choice itself.

**Questions to answer:**

- Why is a URL Monitoring Platform a good learning vehicle for system design?
- What system design concepts will naturally emerge as this project grows?
  (list at least 5)
- Why is the product intentionally kept simple?

---

## Challenge 2: Monolith or Microservices?

You haven't written any code yet, but you already need to make this call.

**Questions to answer:**

- Should your Phase 1 be a monolith or microservices? Why?
- At what point would you consider breaking the monolith apart?
- What are the trade-offs of starting with microservices on day one?

**Hint:** Think about your team size (1 person), your experience with
distributed systems, and the operational overhead of multiple services.

---

## Challenge 3: Pick a Language

You need to choose a language for the entire project.

**Questions to answer:**

- What are your options? (list 3-4 realistic choices)
- For each, what are the pros and cons for THIS project?
- Which one maximizes YOUR learning given your current skills?
- Will this choice still work when you add concurrency (Phase 4),
  queues (Phase 6), and horizontal scaling (Phase 10)?

---

## Challenge 4: Why Docker from Day One?

Your plan says "containerize from day one." Justify it.

**Questions to answer:**

- What specific problems does Docker solve in Phase 0?
- What would go wrong WITHOUT Docker when you add PostgreSQL, Redis,
  RabbitMQ in later phases?
- What is Docker Compose and why do you need it?
- What is the difference between a Dockerfile and docker-compose.yml?

---

## Challenge 5: Repository Structure

How should your project be organized before writing any code?

**Questions to answer:**

- What folders do you need from day one?
- What is the purpose of a `docs/` folder with phase documents?
- Why keep an `adr/` folder separate from phase docs?
- What is the purpose of a `challenges/` folder?
- What belongs in `.gitignore` for a Go + Docker project?

---

## Challenge 6: Development Workflow

How will you build, run, and test your project daily?

**Questions to answer:**

- What repetitive commands will you run every day?
- How can you simplify them? (think Makefile)
- What should `make run` do vs `go run`?
- What does a basic CI pipeline need to check? (lint? test? build?)

---

## Challenge 7: Documentation Strategy

You plan to document everything. But what, where, and how?

**Questions to answer:**

- What is an ADR and when do you write one?
- What is the difference between an ADR and an architecture diagram?
- What should an engineering journal capture that code comments can't?
- What metrics should you track from Phase 1 onward?

---

## Decisions Made

These were finalized during planning:

| Decision         | Choice            | Documented In                  |
| ---------------- | ----------------- | ------------------------------ |
| Language          | Go                | System_Design_Lab_Planning.md  |
| Router            | Chi v5            | docs/phase-00-projectsetup.md  |
| Database          | PostgreSQL 16     | docs/phase-00-projectsetup.md  |
| Containerization  | Docker + Compose  | docs/phase-00-projectsetup.md  |
| Architecture      | Monolith          | Roadmap (evolves over phases)  |
| API Docs          | Swagger (swaggo)  | docs/phase-00-projectsetup.md  |
| Migrations        | golang-migrate    | docs/phase-00-projectsetup.md  |

---

## Next

Move to [Phase 1 Challenge](phase-01-challenge.md) — Design the MVP.
