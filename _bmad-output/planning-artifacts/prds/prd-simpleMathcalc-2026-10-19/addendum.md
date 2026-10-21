---
title: CalcHub PRD Addendum
created: 2026-10-19
updated: 2026-10-21
---

# CalcHub PRD Addendum

Technical constraints supplied by the author. They are inputs for the Architect and are deliberately kept out of the PRD body, which states capabilities rather than implementation. The Architect may refine these and must record decisions as ADRs.

## Architecture style
- Modular monolith: a single deployable with strictly separated modules.

## Backend
- Java 21, Spring Boot 3.5.x, Spring Modulith 1.4.x (module boundaries verified by tests). Chosen 2026-10-21 as the stable, mutually compatible pair; Boot 4.x was considered and rejected.
- Maven or Gradle (Architect to choose and justify).
- No database in Phase I; the backend is stateless (PostgreSQL and Flyway are deferred to Phase II with user identity).
- OpenAPI-first REST API.
- BigDecimal-based arithmetic.
- Package root: `com.bmad.calculator.<module>`, consistent across modules.

## Candidate backend modules (Architect may refine)
`gateway`, `calculation-engine`, `explainability`, `conversion`, `shared-kernel`. History, Memory, and Preferences are frontend features in Phase I. `identity-session` is deferred to Phase II (no user identity in Phase I).

## Module rules
- Modules communicate only via published APIs or application events.
- No cross-module repository or entity access.
- One database schema per module, or table prefixes.
- `explainability` consumes trace events published by `calculation-engine` and has no direct dependency on engine internals.
- Trace delivery transport (REST response vs. SSE) is decided by the Architect and recorded in an ADR.

## Frontend
- React 18 and TypeScript, Vite, Tailwind CSS.
- Feature-based folder structure mirroring backend modules.
- React Query for server state; a lightweight store for UI state.

## Testing
- JUnit 5, Modulith/ArchUnit verification (Testcontainers deferred with the database).
- Vitest and React Testing Library.
- Playwright for end-to-end tests.

## Delivery
- Docker Compose for local run.
- GitHub Actions CI.
