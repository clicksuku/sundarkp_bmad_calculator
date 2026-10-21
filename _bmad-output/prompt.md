# Project Context: "CalcHub", a demo calculator for teaching the BMAD Method

## Purpose

CalcHub is a deliberately small but realistic application used to demonstrate the

BMAD workflow (Brief -> PRD -> Architecture -> Validation/Sharding -> Epics ->

Stories -> Dev/QA). The domain is simple so the audience can focus on process and

artifacts rather than business complexity. Treat it as a real product: traceable

requirements, clear module boundaries, testable acceptance criteria.

## Product Vision

A fast, accurate, keyboard-friendly web calculator that grows from a basic

calculator into a scientific and utility toolkit, with persistent history and

preferences, plus a Pipeline Trace Visualizer that shows how each expression is

evaluated, styled as an analogy to BMAD phases.

## Target Users

1. Everyday user: quick arithmetic, percentages.

2. Student/engineer: scientific functions, parenthesized expressions, DEG/RAD.

3. Developer: programmer mode (BIN/OCT/DEC/HEX, bitwise operations).

4. Power user: history recall, memory registers, unit conversion.

5. Workshop learner: wants to see how an expression is evaluated step by step.

## Functional Scope

### Tier 1: Basic (MVP)

- Addition, subtraction, multiplication, division

- Decimal input, sign toggle (+/-), percent (%)

- Clear (C), clear entry (CE), backspace

- Chained operations with correct display behavior

- Division by zero and invalid-input handling with clear user-facing errors

- Full keyboard support and accessible button layout

- Arbitrary-precision decimal arithmetic (no floating-point artifacts such as

  0.1 + 0.2)

### Tier 2: Advanced

- A. Expression mode: full expressions with parentheses and operator precedence

  (e.g. 2 + 3 * (4 - 1)^2); syntax errors report the position

- B. Scientific functions: sin, cos, tan, asin, acos, atan, log10, ln, sqrt, x^y,

  x^2, 1/x, n!, abs, constants pi and e; DEG/RAD/GRAD toggle

- C. Memory registers: MC, MR, M+, M-, MS, persisted per user/session

- D. History: every evaluated expression stored (expression, result, timestamp,

  mode); list, search, recall into display, delete one, clear all; paginated

- E. Unit converter: length, mass, temperature, volume, time; bidirectional UI;

  conversion factors defined as data, not code

- F. Programmer mode: BIN/OCT/DEC/HEX conversion; bitwise AND/OR/XOR/NOT/shift;

  word size selection (8/16/32/64)

- G. Preferences: theme (light/dark/system), decimal precision, default angle unit,

  number grouping; saved and applied across sessions

- H. Pipeline Trace Visualizer (demo feature):

  - For any expression, the UI shows a step-by-step trace of the real evaluation

    pipeline: tokenize -> parse (AST) -> plan (precedence/evaluation order) ->

    evaluate.

  - The trace is grouped into phases named after BMAD (Analysis, Planning,

    Solutioning, Implementation) as an explanatory analogy only. The UI must state

    that BMAD itself is a development-time workflow, not a runtime component.

  - Trace steps must reflect the actual AST evaluation order; no simulated or fake

    steps.

  - Show per-step detail (tokens, AST node, intermediate value) and basic metrics

    (step count, duration).

  - Layout: calculator on the left, trace visualizer on the right (split pane,

    collapsible on small screens).

  - Backend: a dedicated explainability module that consumes trace events

    published by calculation-engine (no direct dependency on engine internals).

    Delivery transport (REST response vs. SSE) is decided by the Architect and

    recorded in an ADR.

### Explicitly Out of Scope

Graphing, symbolic algebra, currency with live rates, multi-user collaboration,

mobile native apps, payments, and any AI/LLM runtime calls inside the app.

## Technical Constraints

- Architecture style: MODULAR MONOLITH (single deployable, strictly separated

  modules)

- Backend: Java 21, Spring Boot 3.x, Spring Modulith (module boundaries verified by

  tests), Maven or Gradle (Architect to choose and justify), PostgreSQL + Flyway

  migrations, OpenAPI-first REST API, BigDecimal-based math

- Package root: com.bmad.calculator.<module> (consistent for all modules)

- Candidate backend modules (Architect may refine): calculation-engine,

  explainability, history, memory, conversion, preferences, identity-session,

  shared-kernel

- Modules communicate only via published APIs or application events; no cross-module

  repository/entity access; one DB schema per module or table prefixes

- Frontend: React 18 + TypeScript, Vite, Tailwind CSS, feature-based folder

  structure mirroring backend modules, React Query for server state, a lightweight

  store for UI state

- Testing: JUnit 5, Testcontainers, Modulith/ArchUnit verification, Vitest + React

  Testing Library, Playwright for E2E

- Delivery: Docker Compose for local run, GitHub Actions CI

## Non-Functional Requirements (to be refined in PRD)

- Evaluation latency p95 < 100 ms server-side for expressions under 200 characters

- Accuracy: correct to the configured precision (default 15 significant digits)

- Trace overhead: generating a trace adds < 30 ms p95 for typical expressions

- WCAG 2.1 AA, full keyboard operation (including the trace panel)

- Security: input length limits, expression-complexity limits, no code evaluation

  (safe parser only, never eval)

- Observability: structured logging, health endpoints, basic metrics

- Test coverage target: 80%+ on calculation-engine and explainability

## BMAD Working Rules

- Produce artifacts as markdown files under /docs

- Number all requirements (FR-xx, NFR-xx) and keep traceability:

  Requirement -> Epic -> Story -> Acceptance Criteria

- Do not invent scope beyond what is listed above; flag assumptions and open

  questions in a dedicated section

- Keep each artifact concise enough to show in a live demo
