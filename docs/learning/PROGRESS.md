# Progress

## Current phase
Phase 0: Setup & orientation (in progress)

## Environment notes
- Java 25 (Temurin, LTS) is the default JDK; Java 23 is still installed alongside.
- Spring Boot 4.1.1.
- Docker Postgres 18 runs on **host port 5433** (a local Homebrew Postgres already uses 5432).
  Start it with `docker compose up -d`.

## Phase 0 steps
- [x] Install JDK 25, `git init`
- [x] Generate project from start.spring.io; switch to `application.yml`
- [x] Saw startup fail with no DB config (auto-configuration builds beans eagerly)
- [x] Postgres in docker-compose + datasource config; saw a wrong password fail at startup (Flyway connects)
- [x] App starts; `/actuator/health` is UP
- [ ] `GET /hello` endpoint + IoC / DI / beans explanation
- [ ] Walk through `main()` → application context → request served
- [ ] "Your turn" task
- [ ] Learning doc + check-your-understanding questions

## Open questions / not fully understood yet
- (Corrected) Assumed unused libraries are inert like a Python import; in Spring Boot, a library on the
  classpath can trigger auto-configuration at startup.

## Deferred
- Warning `spring.jpa.open-in-view is enabled by default`: discuss in Phase 2 (JPA).
- Warning `No migrations found`: expected until Phase 2 adds Flyway migrations.
