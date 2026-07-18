# Phase 1 Challenge — Single Server MVP

> **Rule:** Do NOT look at `docs/phase-01-single-server.md` until you've
> attempted every section below. That file is the "reference answer" —
> compare your thinking against it AFTER you've done the work.

---

## How This Works

Each section gives you a **challenge** and **hints**. You write your
answers below each challenge. When you're done, share your answers
and we'll review together — mistakes are where the real learning happens.

---

## Challenge 1: Define the Product

You're building a URL Monitoring Platform. Before writing any code,
you need to know exactly what you're building.

**Questions to answer:**

- Who is this product for? (think about the user persona)
- What are the absolute minimum features needed for a usable v1?
- For each feature, assign a priority: P0 (must have) or P1 (nice to have)
- What is explicitly OUT of scope for v1? (this is just as important)
- What non-functional requirements matter? (response time? availability? data retention?)

**Hint:** A common mistake is making the MVP too big. If you can't
build it in a week, it's too much.

**Your Answer:**

```
(write here)
```

---

## Challenge 2: Design the API

You need REST endpoints for your MVP features. Think about:

**Questions to answer:**

- What resources exist in your system? (nouns, not verbs)
- What operations does each resource need?
- How will you structure your URL paths?
- What HTTP methods map to what operations?
- What status codes should each endpoint return?
- How will you handle errors? (think about a consistent format)
- How will you handle pagination for list endpoints?
- Should your API be versioned? Why or why not?

**Hint:** Think about what a client needs. If you were writing curl
commands to test your API, what would feel natural?

**Your Answer:**

```
(write here)
```

---

## Challenge 3: Design the Database

Your data needs to live somewhere. Think about the entities and their
relationships.

**Questions to answer:**

- What tables do you need? (map from your resources)
- What columns does each table need? Think about:
  - Primary keys — what type? auto-increment integer or UUID? Why?
  - Foreign keys — what references what?
  - Timestamps — which ones matter?
  - Constraints — what should be unique? what can't be null?
  - Defaults — what sensible defaults can you set?
- What indexes do you need? (think about your query patterns)
  - Which columns will you filter by?
  - Which columns will you sort by?
  - Any composite indexes needed?
- What happens when a user is deleted? What about their monitors?

**Hint:** Draw the ER diagram on paper first. Write the actual
CREATE TABLE statements with constraints.

**Your Answer:**

```sql
(write here)
```

---

## Challenge 4: Choose Your Tech Stack

For each component below, pick a specific tool/library and write
**why** you chose it. "Because it's popular" is not a valid reason.

**Decisions to make:**

- HTTP router/framework — Chi, Gin, Echo, Fiber, or stdlib? Why?
- Database driver — pgx, database/sql, GORM, sqlx? Why?
- Migration tool — golang-migrate, goose, atlas? Why?
- Authentication — JWT, sessions, OAuth? Why? What library?
- Password hashing — what algorithm? Why?
- Configuration — how will your app read config? hardcoded? env vars? config file?
- UUID generation — who generates IDs? the app or the database? Why?
- Input validation — framework or custom? Why?

**Hint:** For each choice, think about what happens in Phase 5, 10, 12.
Will your choice still work at scale?

**Your Answer:**

```
(write here)
```

---

## Challenge 5: Design the Project Structure

How will you organize your Go code? Think about:

**Questions to answer:**

- Where does your `main.go` live?
- How do you separate HTTP handlers from business logic from database access?
- What is "layered architecture"? Draw the layers and data flow.
- Where do your database models (structs) live?
- Where do migration SQL files live?
- If someone new joins the project, can they understand the structure in 5 minutes?

**Hint:** Look up "standard Go project layout" but don't blindly copy it.
Think about WHY each folder exists. What problem does `internal/` solve
that a regular folder doesn't?

**Your Answer:**

```
(write here)
```

---

## Challenge 6: Docker — Why?

Don't just use Docker because everyone does. Answer these:

**Questions to answer:**

- What problem does Docker solve for this project specifically?
- What goes in your Dockerfile? What is a multi-stage build and why use one?
- What goes in docker-compose.yml? What services do you need for Phase 1?
- What ports need to be exposed?
- How does your Go app connect to PostgreSQL inside Docker?
  (think about networking, hostnames, and environment variables)
- What is the difference between building your Go binary locally vs inside Docker?

**Hint:** Think about what happens when you share this project with
someone on a Mac while you're on Windows.

**Your Answer:**

```
(write here)
```

---

## Challenge 7: Authentication Flow

You need user registration and login. Think through the full flow:

**Questions to answer:**

- What happens step-by-step when a user registers? (from HTTP request to database)
- What happens step-by-step when a user logs in?
- How does the server know who is making a request after login?
- What is a JWT? What goes inside it? (claims)
- Where does the client send the JWT on subsequent requests?
- What happens when the JWT expires?
- How do you hash passwords? Why can't you just encrypt them?
- What is bcrypt and why is it preferred over SHA-256 for passwords?

**Hint:** Draw the sequence diagram on paper:
Client → Server → Database → Server → Client

**Your Answer:**

```
(write here)
```

---

## Challenge 8: Error Handling Strategy

Every API needs to handle errors consistently. Think about:

**Questions to answer:**

- What types of errors can occur? (validation, not found, unauthorized, internal, etc.)
- Should error responses have a consistent structure? What fields?
- Should your API expose internal error details to the client? Why not?
- What HTTP status codes map to what error types?
- What should your server do when an unexpected panic occurs?

**Hint:** Think about what a frontend developer (or your future self using curl)
needs to see when something goes wrong.

**Your Answer:**

```
(write here)
```

---

## When You're Done

1. Share your answers with me
2. I'll review each section — point out mistakes, gaps, and things you nailed
3. We'll discuss trade-offs you might have missed
4. THEN we create the actual documentation and code together

**The goal is not to get the "right" answer. The goal is to think
through the problem yourself, make decisions, and understand the
trade-offs.** Wrong answers you've thought about teach more than
right answers you've copied.
