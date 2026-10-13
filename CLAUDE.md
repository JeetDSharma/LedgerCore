# LedgerCore: Build Plan & Learning Guide

This file is the working agreement between me (the learner) and Claude for building **LedgerCore**, a transactional payments ledger service. Read it fully at the start of every session.

---

## 1. Who I am and what I want

- **Background:** I'm comfortable programming (mostly Python, ML/LLM work). **I have never used Spring Boot.** Java: know the basics and OOP (classes, interfaces, inheritance); explain less-common Java features (generics in depth, lambdas/streams, annotations, records) briefly when they first appear.
- **Goal:** Build a resume-worthy backend project **and actually understand it**. I should be able to explain every design decision in an interview.
- **Not the goal:** Memorizing Spring syntax or annotations. I want the **mental model**: what happens, in what order, who does it (me or the framework), and why.
- **Biggest risk to avoid:** Me blindly accepting code I don't understand. If I can't explain a piece of code, we haven't finished that step.

---

## 2. The project

A REST service that moves money between accounts using **double-entry bookkeeping** and stays correct under retries and concurrent requests.

**Stack:** Java (latest LTS supported by current Spring Boot), Spring Boot (latest stable from start.spring.io), Spring Data JPA (Hibernate), PostgreSQL, Flyway, Maven, Docker / docker-compose, JUnit 5, Testcontainers.

**Resume entry this project must back up** (every claim must be true by the end):
- Double-entry ledger REST service where every transfer stays balanced through atomic PostgreSQL transactions
- Idempotency keys + unique constraints so retried/replayed requests produce exactly one entry
- Optimistic locking + row-level locks for concurrent transfers (no lost updates, no negative balances)
- Transactional outbox pattern for reliable event publishing
- Flyway schema versioning; Testcontainers integration tests against real PostgreSQL

---

## 3. How Claude should teach (rules)

1. **One phase at a time.** Don't start the next phase until I say so. Within a phase, work in small steps (one concept or file group at a time).
2. **Explain before code.** Before writing something new, give a 2–5 sentence explanation: what we're about to do, why, and what Spring will do for us automatically.
3. **Demystify the magic.** Spring does a lot implicitly (dependency injection, auto-configuration, proxies, transactions). When an annotation or convention appears **for the first time**, explain what actually happens at runtime. After that, don't re-explain it.
4. **Prefer explicit over clever.** Use the plain, readable way. **No Lombok.** Use Java `record`s for DTOs. Introduce shortcuts only after I've seen the long form.
5. **Make things visible.** Turn on SQL logging, show request/response with `curl`, show logs. I should *see* the framework doing things.
6. **Break it, then fix it.** For correctness features (transactions, concurrency, idempotency), first write a test or demo that **shows the bug**, then fix it. I should watch the test go red → green.
7. **Ask me questions.** At key points, ask me to predict behavior ("What happens if this method throws halfway?") before running it. Don't just quiz; wait for my answer and correct my mental model.
8. **"Your turn" tasks.** Each phase includes 1–2 small tasks I write myself, with Claude reviewing. Don't write those for me unless I ask.
9. **No surprise dependencies.** Don't add libraries or tools not listed in this plan without asking and explaining why.
10. **Debug together.** When something fails, explain the error message and how you found the cause, not just the fix.
11. **Commit per step.** Small commits with clear messages, so I can look back at how the project grew.

---

## 4. Learning docs (required every phase)

At the end of each phase, write `docs/learning/phase-N-<topic>.md` using this template. Keep it concise. This is my revision notes for interviews, not a textbook.

```markdown
# Phase N: <Topic>

## What we built
2–4 sentences + list of files added/changed (with one line on each file's role).

## Mental model
The core concepts in plain English. Use analogies. Include a simple diagram
(ASCII or Mermaid) of the flow, e.g. HTTP request → Controller → Service → Repository → DB.

## What Spring does for us (the "magic")
What happens implicitly at runtime, and what we'd have to write by hand without Spring.

## Key decisions & trade-offs
Why we chose X over Y. (These are interview answers.)

## How to see it working
Exact commands (curl, test commands) and what output to expect.

## Common mistakes / gotchas
What breaks and how it looks when it breaks.

## Check your understanding
5–8 questions without answers inline (answers in a collapsed <details> block at the end).

## Interview talking points
3–5 bullets I could say out loud about this phase.
```

Also maintain `docs/learning/GLOSSARY.md`: one-line definitions of every term introduced (bean, DI, entity, persistence context, isolation level, etc.), added as we go.

---

## 5. Phases

Each phase lists: **Build** (what we make), **Learn** (concepts), **Done when** (checklist).

### Phase 0: Setup & orientation
- **Build:** Generate the project from start.spring.io (Web, Validation, Data JPA, PostgreSQL Driver, Flyway, Actuator). `docker-compose.yml` with PostgreSQL. A `GET /hello` endpoint. Run it.
- **Learn:** What Spring Boot is vs Spring. Maven and `pom.xml` (dependencies, starters). Project layout. `main()` → application context startup. Embedded web server. `application.yml`. Inversion of control & dependency injection (the single most important concept). What a "bean" is.
- **Done when:** App starts, `curl localhost:8080/hello` works, Postgres runs in Docker, I can explain what happens from `main()` to the request being served.

### Phase 1: REST API & layered architecture (in-memory)
- **Build:** `Account` API: create account, get account, list accounts. Controller → Service → Repository layers, with the repository **in memory** (a `Map`) for now. Request/response DTOs as records. Input validation. Global error handling (`@RestControllerAdvice`) with consistent JSON errors. First unit tests (JUnit 5 + Mockito) for the service.
- **Learn:** Why layers exist. DTO vs domain object. How JSON ↔ Java conversion happens (Jackson). HTTP status codes. Constructor injection, and how Spring wires the layers together. Unit testing with mocks.
- **Done when:** I can trace a request through every layer and explain who calls whom and who created each object.

### Phase 2: PostgreSQL, JPA & Flyway
- **Build:** Replace the in-memory repo with Spring Data JPA. Flyway migrations for `accounts`. Money stored as `BIGINT` minor units (cents) + currency code. Enable SQL logging.
- **Learn:** ORM: what JPA/Hibernate does and doesn't do. Entities, IDs, the persistence context ("Hibernate tracks objects and flushes changes"). How a repository *interface* with no implementation works. Why Flyway instead of auto-generated schema (`ddl-auto=validate`). Why money is never a `double`. **Note how swapping the repository barely touched the service: that's DI paying off.**
- **Done when:** Data survives restarts, I can read the SQL Hibernate generates and match it to my code.

### Phase 3: Double-entry transfers & transactions
- **Build:** Tables: `transfers` and `ledger_entries` (one debit + one credit per transfer, summing to zero). `POST /transfers`. Balance checks (no overdraft). Account balance endpoint. DB constraints (amount > 0, etc.).
- **Learn:** Double-entry bookkeeping basics. ACID. `@Transactional`: what it does, that it works through a proxy, rollback rules, and the self-invocation gotcha. Stored balance vs derived-from-entries balance (trade-off). **Break it first:** show a transfer that fails halfway *without* a transaction leaving bad data, then fix.
- **Done when:** Any failure mid-transfer leaves the DB unchanged, and I can explain why.

### Phase 4: Integration testing with Testcontainers
- **Build:** Integration tests that boot the full app against a real Postgres via Testcontainers (`@ServiceConnection`). Tests for transfers, rollbacks, and constraints. Test data isolation strategy.
- **Learn:** Unit vs integration tests. Why not H2 (locking/SQL differences). How Testcontainers uses Docker. `@SpringBootTest` and what it loads. Test speed trade-offs (container reuse).
- **Done when:** `./mvnw test` spins up Postgres, runs everything green, and tears it down.

### Phase 5: Concurrency control
- **Build:** **First a failing test:** fire many concurrent transfers from the same account (`ExecutorService`) and show lost updates / negative balance. Then fix with (a) optimistic locking (`@Version`) + retry, and (b) pessimistic row locks (`SELECT ... FOR UPDATE`). Prevent deadlocks by locking accounts in a consistent order.
- **Learn:** Race conditions, lost updates. Isolation levels (Read Committed default in Postgres) and what each prevents. Optimistic vs pessimistic locking: when to use which. What a deadlock is and why lock ordering prevents it.
- **Done when:** The concurrency test passes reliably, and I can explain optimistic vs pessimistic locking with this project as the example.

### Phase 6: Idempotency
- **Build:** `Idempotency-Key` header on `POST /transfers`. `idempotency_keys` table with a unique constraint; store the request hash and response. Same key + same body → return the original response. Same key + different body → 409/422. Test concurrent duplicate requests.
- **Learn:** Why networks cause retries. At-least-once vs exactly-once. Why the DB unique constraint (not an `if` check in Java) is the real guarantee. How Stripe-style idempotency works.
- **Done when:** Replaying a request any number of times, even concurrently, creates exactly one transfer.

### Phase 7: Transactional outbox
- **Build:** `outbox_events` table written **in the same transaction** as the transfer. A scheduled publisher (`@Scheduled`) that claims unpublished events with `FOR UPDATE SKIP LOCKED`, "publishes" them (log or simple HTTP webhook to start; a real broker only as an optional stretch), and marks them sent. Retry on failure.
- **Learn:** The dual-write problem (DB commit + message send can't be atomic). How the outbox solves it. At-least-once delivery and why consumers must be idempotent. `SKIP LOCKED` as a lightweight work queue.
- **Done when:** I can kill the publisher mid-run and no event is lost; I can draw the outbox flow from memory.

### Phase 8: Packaging & polish
- **Build:** Dockerfile (multi-stage build) and a compose file running app + Postgres together. OpenAPI/Swagger UI. Actuator health endpoint. README with architecture diagram, how to run, and design decisions. **Optional:** load test with k6 to get real numbers for the resume.
- **Learn:** Container images and layers. Config via environment variables / Spring profiles. What makes a README convincing to a reviewer.
- **Done when:** `docker compose up` runs everything from a fresh clone, and the README explains the design.

---

## 6. Session workflow

At the start of each session:
1. Read this file and `docs/learning/PROGRESS.md`.
2. Summarize where we are in 2–3 lines and what's next.
3. Continue from there.

Maintain `docs/learning/PROGRESS.md`: current phase, completed steps, open questions I had, and anything I said I didn't fully understand yet (revisit those).

At the end of each phase:
1. Write the phase learning doc and update the glossary.
2. Ask me the "Check your understanding" questions conversationally.
3. Only mark the phase done in `PROGRESS.md` once I can answer them.
