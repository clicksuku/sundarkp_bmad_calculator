---
status: final
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-simpleMathcalc-2026-10-19/prd.md
  - _bmad-output/planning-artifacts/prds/prd-simpleMathcalc-2026-10-19/addendum.md
  - _bmad-output/planning-artifacts/architecture/architecture-simpleMathcalc-2026-10-21/ARCHITECTURE-SPINE.md
  - _bmad-output/planning-artifacts/ux-designs/ux-simpleMathcalc-2026-10-21/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-simpleMathcalc-2026-10-21/EXPERIENCE.md
---

# simpleMathcalc (CalcHub) - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for simpleMathcalc (CalcHub), decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: Four-function arithmetic. A user can add, subtract, multiply, and divide; `0.1 + 0.2` shows `0.3`.
FR2: Decimal input and sign toggle. One decimal point per number; `+/-` toggles the current entry.
FR3: Percent. After an operator, `%` is a percentage of the left operand (200 + 10% = 220); standalone it divides by 100.
FR4: Clear, clear entry, backspace. C resets display and pending operation (not Memory or History); CE keeps the pending operation and left operand.
FR5: Chained operations. Basic-mode keypad entry chains left to right as entered.
FR6: Error handling. Clear, non-technical errors for division by zero and invalid input; the user can continue without reloading.
FR7: Keyboard support and accessible layout. Every Basic-mode action is operable from the keyboard; buttons have accessible names and a logical focus order.
FR8: Exact decimal arithmetic. Arbitrary-precision decimal math, no floating-point artifacts.
FR9: Evaluate expressions with precedence. Standard precedence, associativity, parentheses, unary minus (`2 + 3 * (4 - 1)^2` = 29).
FR10: Syntax errors with position. Invalid expressions report the 1-based character position; unbalanced parentheses are reported.
FR11: Scientific functions. sin, cos, tan, asin, acos, atan, log10, ln, sqrt, x^y, x^2, 1/x, n!, abs; domain errors give an Error, not NaN.
FR12: Constants. `pi` and `e`.
FR13: Angle Unit toggle. DEG, RAD, GRAD, with the current unit visible.
FR14: Memory operations. MC, MR, M+, M-, MS on a single Memory Register, with a non-zero indicator.
FR15: Memory persistence. The Memory Register survives reloads and browser restarts (browser storage in Phase I).
FR16: Record history. A History Entry (expression, result, timestamp, Mode) for every successful Calculation; errors are not stored.
FR17: List and paginate history. Newest first, 20 per page.
FR18: Search history. By text in the expression or result.
FR19: Recall into display. Load a History Entry's expression or result into the display.
FR20: Delete one / clear all history, with confirmation before clear-all.
FR21: Convert between units. Length, mass, temperature, volume, time; temperature supports offset conversions.
FR22: Bidirectional entry. Editing either converter field updates the other.
FR23: Data-defined conversion factors. Units and factors are data; adding a unit needs no code change.
FR24: Base conversion. Enter a value in BIN, OCT, DEC, or HEX and see all four bases.
FR25: Bitwise operations. AND, OR, XOR, NOT, left/right shift (unsigned in Phase I).
FR26: Word Size selection. 8, 16, 32, or 64 bits constrain results (overflow semantics deferred to Phase II).
FR27: Set preferences. Theme (light/dark/system), Precision, default Angle Unit, number grouping.
FR28: Persist and apply preferences. Saved (browser storage in Phase I) and applied at load; System theme follows the OS.
FR29: Generate a real Trace. Tokenize, parse (AST), plan, evaluate; steps reflect the real evaluation order, no simulated steps.
FR30: Per-step detail. Tokens, AST node, or intermediate value per Trace Step.
FR31: Trace metrics. Step count and total duration.
FR32: BMAD phase grouping. Steps grouped under Analysis, Planning, Solutioning, Implementation; a notice states BMAD is a development-time workflow, not a runtime component.
FR33: Split-pane layout. Calculator left, Trace Panel right; collapses on small screens.
FR34: Keyboard-accessible Trace Panel. All controls operable by keyboard with visible focus.

### NonFunctional Requirements

NFR1: Performance. Server-side evaluation p95 < 100 ms for expressions under 200 characters (indicative in Phase I, not a release gate).
NFR2: Trace overhead. Trace generation adds < 30 ms p95 for typical expressions under 50 characters (indicative in Phase I).
NFR3: Accuracy. Correct to the configured Precision (default 15 significant digits).
NFR4: Accessibility. WCAG 2.1 AA; full keyboard operation including the Trace Panel.
NFR5: Security. Expression length and complexity limits; safe parser, never executes input as code.
NFR6: Observability. Structured logging, health endpoints, basic metrics.
NFR7: Quality. At least 80% automated test coverage on the calculation and explainability capabilities.
NFR8: Data isolation. *Phase II, out of scope for Phase I.*

### Additional Requirements

- AR1: No starter template is named. The build is a Gradle (Kotlin DSL) single-build project, Java 21, wrapper committed, plus a Vite 8 React 18 TypeScript frontend; project scaffolding is Epic 1 Story 1.
- AR2: Spring Boot 3.5.x with Spring Modulith 1.4.x. Modules: `gateway`, `calculation-engine`, `explainability`, `conversion`, `shared-kernel`. A `ModularityTests` test runs `ApplicationModules.verify()`; ArchUnit forbids script engines and reflection-based evaluation.
- AR3: The backend is stateless: no database, no session or identity. PostgreSQL, Flyway and Testcontainers are dropped from Phase I.
- AR4: Contract-first API. `api/openapi.yaml` (OpenAPI 3.1) is the source of truth; server interfaces/DTOs and TypeScript types are generated from it; CI fails if regenerated output differs.
- AR5: Errors use RFC 9457 `application/problem+json` with stable `code` values (`DIVISION_BY_ZERO`, `SYNTAX_ERROR`, `DOMAIN_ERROR`, `LIMIT_EXCEEDED`) and a 1-based `position` for syntax errors; 422 for calculation errors, 400 for malformed requests.
- AR6: One evaluation pipeline in `calculation-engine`: lexer, Pratt/recursive-descent parser to AST, planner, `BigDecimal` evaluator (`DECIMAL128` internally, display rounding last). Limits: 256 characters, nesting depth 32. `%` semantics live only in the engine. Scientific functions use `ch.obermuhlner:big-math` 2.3.2.
- AR7: Trace: engine publishes `TraceEvent`s (type in `shared-kernel`) via a synchronous Spring `ApplicationEventPublisher` only when tracing is requested; `explainability` listens synchronously, buffers by `evaluationId`, and assembles the Trace with Phase Labels (tokenize to Analysis, parse to Planning, plan to Solutioning, evaluate to Implementation). `gateway` orchestrates and returns result and trace in one REST response.
- AR8: Unit data in `backend/src/main/resources/units/*.yaml`, one file per Unit Category, exact decimal-string factors; `GET /api/v1/units` serves the list.
- AR9: Programmer mode is an engine mode (`mode=PROGRAMMER`, `base`, `wordSize`) using `BigInteger` masked to the Word Size; unsigned in Phase I.
- AR10: Frontend: React 18, TypeScript, Vite 8, Tailwind CSS 4.2, React Query 5 for server state, Zustand 5 for UI state; feature slices (`calculator`, `scientific`, `programmer`, `history`, `memory`, `conversion`, `preferences`, `trace`) that expose only `index.ts`.
- AR11: Browser persistence behind a `StoragePort` in `shared/storage`: IndexedDB for History, localStorage for Memory and Preferences; no direct storage calls elsewhere.
- AR12: Decimals and integers cross the API as JSON strings; timestamps ISO-8601 UTC; History ids are UUIDv4 generated in the browser; browser never does arithmetic.
- AR13: Delivery: `docker compose up` runs `web` (nginx serving the Vite build and proxying `/api`) and `api` (Spring Boot).
- AR14: CI on GitHub Actions: backend build and tests, frontend lint and tests, API regenerate-and-diff, Playwright E2E against Compose.
- AR15: Observability: Spring Actuator health and metrics, JSON structured logs, `evaluationId` in the MDC, expression text logged only at DEBUG.
- AR16: Testing: JUnit 5, ArchUnit, Vitest + React Testing Library, Playwright; JaCoCo coverage gate 80% on `calculation-engine` and `explainability`.

### UX Design Requirements

- UX-DR1: Design tokens: implement the Light and Dark color tokens (page, surface, display, border, text, six key families, focus ring, error, four trace phases), typography, radii and spacing from DESIGN.md as Tailwind theme variables; theme modes Light, Dark, System (follows `prefers-color-scheme`).
- UX-DR2: Key component with six families (digit, operator, function, memory, clear, equals) and states default, hover, pressed, focus, disabled; native `<button>`, 48 px minimum size.
- UX-DR3: Display component: two lines (expression 20 px above, result 40 px right-aligned), tabular monospace, horizontal scroll with end in view, error replaces the result line with icon and message, `aria-live="polite"`.
- UX-DR4: Top bar with view switch (Simple, Scientific, Programmer, Converter), History, Trace toggle and Settings; switching view keeps the current expression and result.
- UX-DR5: Simple view layout: 4-column keypad with digits, four operations, %, +/-, memory row, C/CE/backspace, `=`.
- UX-DR6: Scientific view: function block (functions, constants, parentheses, powers, factorial); on narrow screens `main` and `func` tabs.
- UX-DR7: Segmented DEG/RAD/GRAD toggle above the keypad; changes only future evaluations; saved as default Angle Unit.
- UX-DR8: Memory badge `M` in the display corner when the Memory Register is non-zero.
- UX-DR9: History drawer: newest-first rows (48 px), search box, page controls, per-row delete with accessible name, clear-all confirmation, empty and no-match states.
- UX-DR10: Trace Panel: title, analogy notice, four collapsible phase groups with `trace-phase-chip`, ordered step rows (number, stage, detail, value), expandable detail, footer with step count and duration; shows the last successful evaluation; idle message when empty.
- UX-DR11: Responsive layout: at least 1024 px split pane with History as a drawer; 640 to 1023 px centered calculator with Trace as a side sheet; under 640 px bottom sheets for Trace and History.
- UX-DR12: Settings sheet for theme, Precision, default Angle Unit, number grouping.
- UX-DR13: Keyboard map: digits, `.`, `+ - * / ^ % ( )`, Enter/`=`, Backspace, Delete (CE), Escape (C), Alt+1 to Alt+4 for views, Alt+H and Alt+T for History and Trace, arrow keys inside lists; shortcut shown in tooltip and accessible name.
- UX-DR14: Accessibility: accessible names for all keys, visible focus ring, focus trapped in sheets and drawers and returned to the opener, targets 44 px or larger, `prefers-reduced-motion` respected, color never the only signal.
- UX-DR15: State patterns: idle, evaluating (indicator after 150 ms, new keypress cancels the pending request), result, error, service unreachable (inline message, keypad still usable), storage unavailable (page-only fallback with a notice in Settings).
- UX-DR16: Voice and tone strings in one `shared/strings` module with the specified error and empty-state texts.
- UX-DR17: Converter view: category tabs, two linked fields, swap button, unit pickers.
- UX-DR18: Programmer view: BIN/OCT/DEC/HEX selector, word size selector, hex keys A-F, bitwise keys, value shown in all four bases.
- UX-DR19: Entry rule: keypad presses chain left to right; a typed parenthesized expression switches the entry to expression evaluation on `=`.

### FR Coverage Map

FR1: Epic 1 - Four-function arithmetic
FR2: Epic 1 - Decimal input and sign toggle
FR3: Epic 1 - Percent
FR4: Epic 1 - Clear, clear entry, backspace
FR5: Epic 1 - Chained operations
FR6: Epic 1 - Error handling
FR7: Epic 1 - Keyboard support and accessible layout
FR8: Epic 1 - Exact decimal arithmetic
FR9: Epic 2 - Evaluate expressions with precedence
FR10: Epic 2 - Syntax errors with position
FR11: Epic 2 - Scientific functions
FR12: Epic 2 - Constants
FR13: Epic 2 - Angle Unit toggle
FR14: Epic 5 - Memory operations
FR15: Epic 5 - Memory persistence
FR16: Epic 5 - Record history
FR17: Epic 5 - List and paginate history
FR18: Epic 5 - Search history
FR19: Epic 5 - Recall into display
FR20: Epic 5 - Delete one / clear all history
FR21: Epic 6 - Convert between units
FR22: Epic 6 - Bidirectional entry
FR23: Epic 6 - Data-defined conversion factors
FR24: Epic 7 - Base conversion
FR25: Epic 7 - Bitwise operations
FR26: Epic 7 - Word Size selection
FR27: Epic 4 - Set preferences
FR28: Epic 4 - Persist and apply preferences
FR29: Epic 3 - Generate a real Trace
FR30: Epic 3 - Per-step detail
FR31: Epic 3 - Trace metrics
FR32: Epic 3 - BMAD phase grouping
FR33: Epic 3 - Split-pane layout
FR34: Epic 3 - Keyboard-accessible Trace Panel

## Epic List

### Epic 1: Basic Calculator
A user opens CalcHub and does everyday arithmetic with exact decimal results, from the keypad or the keyboard, in a colorful Simple view. Includes project scaffolding (Gradle backend, Vite frontend, API contract, Docker Compose, CI skeleton) as the first story.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR8

### Epic 2: Expressions & Scientific Functions
A user types full expressions with parentheses and operator precedence, uses scientific functions and constants with a DEG/RAD/GRAD toggle, and sees the exact position of any syntax error.
**FRs covered:** FR9, FR10, FR11, FR12, FR13

### Epic 3: Pipeline Trace Visualizer
A workshop learner sees how any expression is really evaluated, step by step, grouped under the BMAD phase names, beside the calculator.
**FRs covered:** FR29, FR30, FR31, FR32, FR33, FR34

### Epic 4: Preferences & Themes
A user sets theme (light, dark, system), precision, default angle unit and number grouping, and they persist across visits.
**FRs covered:** FR27, FR28

### Epic 5: Memory & History
A user stores values in memory and searches, recalls and deletes earlier calculations.
**FRs covered:** FR14, FR15, FR16, FR17, FR18, FR19, FR20

### Epic 6: Unit Converter
A user converts length, mass, temperature, volume and time in either direction.
**FRs covered:** FR21, FR22, FR23

### Epic 7: Programmer Mode
A developer converts between BIN, OCT, DEC and HEX and uses bitwise operations at a chosen word size.
**FRs covered:** FR24, FR25, FR26

## Epic 1: Basic Calculator

A user opens CalcHub and does everyday arithmetic with exact decimal results, from the keypad or the keyboard, in a colorful Simple view. Includes project scaffolding as the first stories.

### Story 1.1: Backend skeleton with enforced module boundaries

As a tutorial author,
I want a runnable Spring Boot service whose module boundaries fail the build when broken,
So that every later story builds on a verified foundation.

**Requirements:** AR1, AR2, AR3, NFR6

**Acceptance Criteria:**

**Given** a fresh checkout
**When** I run `./gradlew build`
**Then** the build succeeds on Java 21 with Spring Boot 3.5.x and Spring Modulith 1.4.x
**And** modules `gateway`, `calculationengine`, `explainability`, `conversion` and `sharedkernel` exist under `com.bmad.calculator`

**Given** a class in one module imports another module's `internal` package
**When** `ModularityTests` runs
**Then** the build fails

**Given** the service is running
**When** I call `/actuator/health`
**Then** it returns `UP`
**And** logs are structured JSON
**And** the Gradle wrapper is committed and there is no database dependency

### Story 1.2: Frontend skeleton with colorful light/dark theme

As a user,
I want CalcHub to open in a clean, colored page that follows my system theme,
So that it feels polished from the first visit.

**Requirements:** AR10, UX-DR1, UX-DR4

**Acceptance Criteria:**

**Given** `npm run dev`
**When** I open the app
**Then** I see a top bar and an empty Simple-view shell built with React 18, TypeScript, Vite and Tailwind

**Given** my OS is set to dark
**When** the page loads
**Then** the dark tokens from DESIGN.md apply, and the light tokens apply when the OS is light
**And** all color, type, radius and spacing tokens are defined as Tailwind theme variables
**And** the feature-folder structure from the architecture exists, with each feature exposing only `index.ts`

### Story 1.3: Exact decimal evaluation engine

As a user,
I want the calculation engine to compute exactly,
So that results never show floating-point artifacts.

**Requirements:** FR1, FR8, NFR3, NFR5, AR6

**Acceptance Criteria:**

**Given** `0.1+0.2`
**When** the engine evaluates it
**Then** the result is `0.3`
**And** `1.1*1.1` gives `1.21`

**Given** `1/3` and precision 15
**When** the engine evaluates it
**Then** the result is rounded to 15 significant digits

**Given** a number or two operands joined by `+`, `-`, `*` or `/`
**When** the engine evaluates it
**Then** the pipeline runs lexer, parser, planner, evaluator in that order, using `BigDecimal` with `DECIMAL128` internally

**Given** `5/0`
**When** the engine evaluates it
**Then** it raises a division-by-zero error that carries the code `DIVISION_BY_ZERO`

**Given** text longer than 256 characters
**When** the engine evaluates it
**Then** it raises a limit-exceeded error that carries the code `LIMIT_EXCEEDED`
**And** the engine's published API is `Evaluator.evaluate(...)`, everything else is in an `internal` package, and `ModularityTests` still passes
**And** the engine has unit tests and stays above 80% coverage

### Story 1.4: Evaluation API with a contract-first OpenAPI

As a user,
I want my browser to ask the server to calculate,
So that every result comes from the exact engine.

**Requirements:** FR1, FR8, NFR1, AR4, AR5, AR12

**Acceptance Criteria:**

**Given** `api/openapi.yaml` defines `POST /api/v1/evaluations`
**When** I build
**Then** the server interfaces and TypeScript types are generated from it
**And** a check task fails if regenerating them changes the committed output (Story 1.10 runs it in CI)

**Given** `{"expression":"0.1+0.2"}`
**When** it is posted
**Then** the response is 200 with the result as the string `"0.3"`

**Given** an optional `precision` in the request
**When** it is omitted
**Then** the default is 15 significant digits

**Given** `5/0`, or an expression over 256 characters
**When** it is posted
**Then** the server returns 422 with `application/problem+json` and code `DIVISION_BY_ZERO` or `LIMIT_EXCEEDED`

**Given** a malformed request body
**When** it is posted
**Then** the server returns 400 with `application/problem+json`
**And** `gateway` owns the controller and calls the engine only through its published API, numbers cross the wire as strings, and evaluation latency is logged per request so the 100 ms p95 target can be checked locally (indicative, not a gate)

### Story 1.5: Keypad and display for digits, decimals and the four operations

As an everyday user,
I want to tap digits and operators and see the answer,
So that I can do quick sums.

**Requirements:** FR1, FR2, FR5, UX-DR2, UX-DR3, UX-DR5, UX-DR19

**Acceptance Criteria:**

**Given** the Simple view
**When** I tap `7 + 5 =`
**Then** the display shows `7 + 5` above and `12` below

**Given** `2 + 3 × 4` entered on the keypad
**When** I press `=`
**Then** the result is `20` (left to right)

**Given** a number already has a decimal point
**When** I press `.` again
**Then** nothing changes
**And** digit and operator keys use their color families and the result line uses the monospace display style
**And** pressing an operator after another operator replaces the pending operator

### Story 1.6: Sign toggle, clear, clear entry and backspace

As an everyday user,
I want to fix mistakes quickly,
So that I don't have to start over.

**Requirements:** FR2, FR4

**Acceptance Criteria:**

**Given** the entry `5`
**When** I press `+/-`
**Then** it shows `-5`, and pressing again shows `5`

**Given** `8 + 3` pending
**When** I press CE
**Then** `8 +` remains and the entry is cleared

**Given** any state
**When** I press C
**Then** display, operands and pending operator reset

**Given** the entry `123`
**When** I press backspace
**Then** it shows `12`
**And** clear keys use the coral clear family

### Story 1.7: Percent

As an everyday user,
I want a percent key,
So that I can add tips and discounts.

**Requirements:** FR3, AR6

**Acceptance Criteria:**

**Given** `200 + 10 %`
**When** I press `=`
**Then** the result is `220`

**Given** `50 %` alone
**When** it is evaluated
**Then** the result is `0.5`

**Given** `200 - 10 %`
**When** it is evaluated
**Then** the result is `180`
**And** percent semantics are defined only in the engine grammar, and the browser sends the expression unchanged

### Story 1.8: Plain-language errors and recovery

As a user,
I want clear messages when something can't be calculated,
So that I know what to fix.

**Requirements:** FR6, NFR5, AR5, UX-DR3, UX-DR15, UX-DR16

**Acceptance Criteria:**

**Given** `5 / 0 =`
**When** it is evaluated
**Then** the server returns 422 with code `DIVISION_BY_ZERO`
**And** the display shows "Can't divide by zero." with an icon

**Given** an error is showing
**When** I press C or a digit
**Then** normal entry resumes without a reload

**Given** the server is unreachable
**When** I press `=`
**Then** an inline "Can't reach the calculator service" message appears and the keypad stays usable
**And** errors use RFC 9457 `application/problem+json` with a stable `code`, and all texts come from `shared/strings`
**And** an in-flight request is cancelled when a new key is pressed
**And** the result line announces changes through an `aria-live="polite"` region

### Story 1.9: Keyboard operation and accessible keypad

As a keyboard or screen-reader user,
I want to operate every key without a mouse,
So that CalcHub works for me.

**Requirements:** FR7, NFR4, UX-DR13, UX-DR14

**Acceptance Criteria:**

**Given** focus is on the page
**When** I type digits, `. + - * / %` and Enter
**Then** they behave like the on-screen keys (`*` shows as `×`, `/` as `÷`)

**Given** Backspace, Delete and Escape
**When** they are pressed
**Then** they act as backspace, CE and C

**Given** the keypad
**When** I tab through it
**Then** the order follows visual order, every key is a native button with an accessible name, and a visible focus ring shows
**And** targets are at least 44 px and key colors meet WCAG AA contrast
**And** an axe accessibility check of the Simple view reports no serious or critical violations

### Story 1.10: One-command run and automated checks

As a tutorial author,
I want one command to run the whole app and a pipeline to check it,
So that readers can follow along reliably.

**Requirements:** NFR7, AR13, AR14, AR16

**Acceptance Criteria:**

**Given** Docker is installed
**When** I run `docker compose up`
**Then** `web` (nginx) and `api` (Spring Boot) start and CalcHub works at one URL with `/api` proxied

**Given** a push or pull request
**When** GitHub Actions runs
**Then** it executes backend build and tests, frontend lint and tests, the API regenerate-and-diff check, and a Playwright smoke test (`0.1 + 0.2 = 0.3`)
**And** JaCoCo enforces 80% coverage on `calculationengine`, and the build fails below it

## Epic 2: Expressions & Scientific Functions

A user types full expressions with parentheses and operator precedence, uses scientific functions and constants with a DEG/RAD/GRAD toggle, and sees the exact position of any syntax error.

### Story 2.1: Evaluate expressions with precedence and parentheses

As a student,
I want `2 + 3 * (4 - 1)^2` to give the right answer,
So that I can enter a whole expression at once.

**Requirements:** FR9, NFR5, AR6

**Acceptance Criteria:**

**Given** `2 + 3 * (4 - 1)^2`
**When** it is posted to the evaluation endpoint
**Then** the result is `"29"`

**Given** `-(2+3)`
**When** it is evaluated
**Then** the result is `"-5"`

**Given** `2^3^2`
**When** it is evaluated
**Then** the result is `"512"` (power is right-associative)
**And** `10 - 4 - 3` gives `"3"` (subtraction is left-associative)

**Given** nesting deeper than 32 levels
**When** it is posted
**Then** the server returns 422 with code `LIMIT_EXCEEDED`
**And** the planner fixes the evaluation order, and the engine rejects any text it cannot parse safely (no code execution)

### Story 2.2: Syntax errors that show where the problem is

As a student,
I want to be told exactly where my expression is wrong,
So that I can fix it quickly.

**Requirements:** FR10, AR5

**Acceptance Criteria:**

**Given** `2 + * 3`
**When** it is evaluated
**Then** the server returns 422 with code `SYNTAX_ERROR` and `position` 5 (1-based)

**Given** `(2 + 3`
**When** it is evaluated
**Then** the error reports the unclosed parenthesis and its position

**Given** a syntax error
**When** it is shown
**Then** the display reads "Check the expression: unexpected `*` at position 5." and marks that character in the expression line
**And** the entry stays editable so I can correct it without retyping

### Story 2.3: Type or paste a full expression into the display

As a student,
I want the display to accept typed text,
So that I can enter expressions without tapping keys.

**Requirements:** FR9, UX-DR19

**Acceptance Criteria:**

**Given** the display is focused
**When** I type `(2+3)*4` and press Enter
**Then** the result `20` appears

**Given** I paste an expression
**When** I press Enter
**Then** it is evaluated, and pasted text over 256 characters is rejected with the "too long" message

**Given** a typed expression with parentheses
**When** I press `=`
**Then** it is evaluated with full precedence, while plain keypad chains still evaluate left to right
**And** `×` and `÷` typed or pasted are accepted as `*` and `/`

### Story 2.4: Scientific view with core functions and constants

As a student,
I want a Scientific view with the common functions,
So that I can do more than basic arithmetic.

**Requirements:** FR11, FR12, UX-DR4, UX-DR6

**Acceptance Criteria:**

**Given** the top bar
**When** I choose Scientific (or press Alt+2)
**Then** the view shows the function block in the function key color, and my current expression and result carry over

**Given** `sqrt(16)`, `5!`, `log10(1000)`, `ln(e)`, `abs(-3)`, `2^10`, `4^2` and `1/4`
**When** they are evaluated
**Then** the results are `4`, `120`, `3`, `1`, `3`, `1024`, `16` and `0.25`

**Given** the constants `pi` and `e`
**When** they are evaluated
**Then** they use the engine's arbitrary-precision values

**Given** `sqrt(-1)`, `ln(0)` or `(-3)!`
**When** they are evaluated
**Then** the server returns 422 with code `DOMAIN_ERROR` and the display shows a plain-language message, not NaN
**And** on narrow screens the function keys sit behind `main` and `func` tabs, and every function key has an accessible name

### Story 2.5: Trigonometry with a DEG/RAD/GRAD toggle

As an engineer,
I want trig functions in my chosen angle unit,
So that angles mean what I expect.

**Requirements:** FR11, FR13, UX-DR7

**Acceptance Criteria:**

**Given** DEG is selected
**When** I evaluate `sin(30)`
**Then** the result is `0.5`

**Given** RAD is selected
**When** I evaluate `sin(pi/6)`
**Then** the result is `0.5` to the configured precision

**Given** `cos`, `tan`, `asin`, `acos` and `atan`
**When** they are evaluated in each angle unit
**Then** inverse functions return angles in the selected unit

**Given** `asin(2)`
**When** it is evaluated
**Then** the server returns 422 with code `DOMAIN_ERROR`
**And** the segmented DEG/RAD/GRAD toggle is visible in Scientific view, shows the current unit, changes only future evaluations, and is sent as `angleUnit` in the request

## Epic 3: Pipeline Trace Visualizer

A workshop learner sees how any expression is really evaluated, step by step, grouped under the BMAD phase names, beside the calculator.

### Story 3.1: The engine reports what it actually does

As a workshop learner,
I want the trace to come from the real evaluation,
So that nothing in it is faked.

**Requirements:** FR29, AR7

**Acceptance Criteria:**

**Given** an evaluation started with tracing on
**When** the pipeline runs
**Then** the engine publishes `TraceEvent`s for tokenize, parse, plan and evaluate, each with `evaluationId`, `stage`, `detail`, `value` and a nanosecond timestamp

**Given** an evaluation started with tracing off
**When** the pipeline runs
**Then** no events are published

**Given** `2 + 3 * 4`
**When** it is traced
**Then** the evaluate events show multiplication before addition and the intermediate value `12`
**And** the `TraceEvent` type lives in `sharedkernel`, and a Modulith test proves the engine does not depend on `explainability`

### Story 3.2: Assemble the trace from events

As a workshop learner,
I want the trace built from the engine's events,
So that every step I see is a real one.

**Requirements:** FR29, FR30, FR31, FR32, NFR7, AR7

**Acceptance Criteria:**

**Given** `TraceEvent`s for an `evaluationId`
**When** `explainability` collects them
**Then** it returns ordered steps, each with its phase label, detail and value

**Given** the stages tokenize, parse, plan and evaluate
**When** the trace is built
**Then** they are labelled Analysis, Planning, Solutioning and Implementation, and the mapping lives only in `explainability`

**Given** a trace
**When** it is built
**Then** it includes the step count and the total duration in milliseconds, computed from the event timestamps

**Given** a trace that was begun but never collected, or an evaluation that failed
**When** the buffer is discarded
**Then** no events for that `evaluationId` remain
**And** `explainability` listens with a synchronous `@EventListener`, depends only on `sharedkernel`, and stays above 80% test coverage

### Story 3.3: Return the trace with the result

As a workshop learner,
I want the trace in the same response as the result,
So that the panel always matches what I just calculated.

**Requirements:** FR29, NFR2, AR4, AR7

**Acceptance Criteria:**

**Given** `trace: true` in the evaluation request
**When** it is posted
**Then** the response contains the result and the trace

**Given** `trace` is omitted or false
**When** it is posted
**Then** the response has no trace and the engine publishes no events

**Given** a failed evaluation posted with tracing on
**When** the server responds
**Then** it returns the usual error response and keeps no leftover trace buffer
**And** `gateway` orchestrates the call (begin, evaluate, collect), `api/openapi.yaml` and the generated types are updated, and trace overhead is measured and logged against the 30 ms target (indicative, not a gate)

### Story 3.4: Trace Panel beside the calculator

As a workshop learner,
I want a panel next to the calculator,
So that I can watch the evaluation as I work.

**Requirements:** FR33, UX-DR10, UX-DR11

**Acceptance Criteria:**

**Given** a screen at least 1024 px wide
**When** I open CalcHub
**Then** the calculator is on the left and the Trace Panel on the right, in a split pane

**Given** a narrower screen
**When** I tap the Trace toggle in the top bar
**Then** the panel opens as a side sheet (640 to 1023 px) or a bottom sheet (under 640 px) and can be closed

**Given** nothing has been evaluated
**When** the panel is open
**Then** it shows "Evaluate an expression to see its trace."

**Given** I evaluate an expression in Simple or Scientific view
**When** the result arrives
**Then** the request asks for a trace and the panel updates to the last successful evaluation
**And** an error evaluation leaves the previous trace in place

### Story 3.5: Phases, steps, detail and metrics in the panel

As a workshop learner,
I want to read each step in order under its phase,
So that I understand the pipeline.

**Requirements:** FR30, FR31, FR32, UX-DR10

**Acceptance Criteria:**

**Given** a trace
**When** the panel renders
**Then** four collapsible groups (Analysis, Planning, Solutioning, Implementation) appear, each with its phase chip, color and name

**Given** a step row
**When** I expand it
**Then** I see its detail: tokens, the AST node, or the intermediate value

**Given** the panel
**When** it renders
**Then** the footer shows the step count and total duration
**And** the header shows "The phase names are an analogy. BMAD is a development-time workflow, not part of how CalcHub runs."
**And** step order matches the real evaluation order, and phase colors use the phase tokens in both themes

### Story 3.6: Keyboard and screen-reader access to the Trace Panel

As a keyboard or screen-reader user,
I want the whole panel to work without a mouse,
So that I can learn from it too.

**Requirements:** FR34, NFR4, UX-DR13, UX-DR14

**Acceptance Criteria:**

**Given** the app
**When** I press Alt+T
**Then** the panel opens or closes and focus moves into it, returning to the opener on close

**Given** focus in the step list
**When** I press the arrow keys
**Then** focus moves between rows, and Enter expands or collapses the row

**Given** a sheet is open
**When** I press Tab
**Then** focus stays inside it, and Escape closes it

**Given** every control in the panel
**When** it has focus
**Then** a visible focus ring shows, and each has an accessible name
**And** an axe check of the panel reports no serious or critical violations, and `prefers-reduced-motion` turns off panel animation

## Epic 4: Preferences & Themes

A user sets theme (light, dark, system), precision, default angle unit and number grouping, and they persist across visits.

### Story 4.1: Settings sheet with a persisted theme

As a user,
I want to choose Light, Dark or System and have CalcHub remember it,
So that the app looks the way I like every time.

**Requirements:** FR27, FR28, AR11, UX-DR1, UX-DR12, UX-DR15

**Acceptance Criteria:**

**Given** the top bar
**When** I open Settings
**Then** a sheet shows the theme choice (Light, Dark, System) with System selected by default

**Given** I choose Dark
**When** I reload the page
**Then** the dark theme is applied from the first render, saved through the `StoragePort` (localStorage adapter)

**Given** I choose System
**When** my OS theme changes
**Then** CalcHub follows it

**Given** localStorage or IndexedDB is blocked
**When** I change a setting
**Then** it works for the current page only and Settings shows a one-line notice that settings will not be saved
**And** no feature code touches `localStorage` or `indexedDB` directly, only the `StoragePort`
**And** the sheet traps focus while open and returns it to the opener on close

### Story 4.2: Precision and number grouping

As a user,
I want to choose how many digits results show and whether they are grouped,
So that numbers read the way I prefer.

**Requirements:** FR27, FR28, NFR3

**Acceptance Criteria:**

**Given** Settings
**When** I set Precision to 8
**Then** `1/3` shows `0.33333333` and Precision is sent as `precision` in every evaluation request

**Given** number grouping is on
**When** a result is `1234567.89`
**Then** it shows `1,234,567.89`, and with grouping off it shows `1234567.89`

**Given** a reload
**When** the app loads
**Then** Precision and grouping match the saved values
**And** the default Precision is 15 significant digits

### Story 4.3: Default angle unit preference

As an engineer,
I want my usual angle unit remembered,
So that I don't set DEG every day.

**Requirements:** FR27, FR28

**Acceptance Criteria:**

**Given** Settings
**When** I set the default Angle Unit to RAD
**Then** new sessions of CalcHub start with RAD selected in the toggle

**Given** I switch the toggle to DEG during use
**When** I reload
**Then** the saved default (not the temporary choice) applies unless I changed the default
**And** all preference changes apply immediately without a reload

## Epic 5: Memory & History

A user stores values in memory and searches, recalls and deletes earlier calculations.

### Story 5.1: Memory keys

As a power user,
I want MC, MR, M+, M- and MS keys,
So that I can hold a value while I work on something else.

**Requirements:** FR14, UX-DR8

**Acceptance Criteria:**

**Given** the display shows `12`
**When** I press MS
**Then** `12` is stored, and an `M` badge appears in the display corner

**Given** a stored `12` and the display shows `5`
**When** I press M+
**Then** memory becomes `17`, and M- from `17` with `5` shown makes it `12`

**Given** a stored value
**When** I press MR
**Then** it is placed into the current entry

**Given** a stored value
**When** I press MC
**Then** memory becomes zero and the badge disappears
**And** memory keys use the teal memory color family and have accessible names

### Story 5.2: Memory survives reloads

As a power user,
I want my stored value to still be there tomorrow,
So that I don't lose it when I close the tab.

**Requirements:** FR15, AR11

**Acceptance Criteria:**

**Given** a stored value
**When** I reload the page or restart the browser
**Then** MR still recalls it and the badge still shows
**And** the value is saved through the `StoragePort`, and with blocked storage it works for the current page only

### Story 5.3: Record every successful calculation

As a power user,
I want each result saved automatically,
So that I can find it later.

**Requirements:** FR16, AR11

**Acceptance Criteria:**

**Given** `2 + 3 =` succeeds
**When** the result arrives
**Then** a History Entry is saved with a UUIDv4 id, the expression, the result, an ISO-8601 UTC timestamp and the Mode (Simple or Scientific)

**Given** an evaluation that returns an error
**When** the error is shown
**Then** no History Entry is saved
**And** entries are stored in IndexedDB through the `StoragePort`

### Story 5.4: History drawer with pages

As a power user,
I want to browse my past calculations,
So that I can see what I did.

**Requirements:** FR17, UX-DR9, UX-DR13

**Acceptance Criteria:**

**Given** History entries
**When** I open the drawer (Alt+H or the top-bar icon)
**Then** entries are listed newest first, 20 per page, with page controls when there are more than 20

**Given** no entries
**When** I open the drawer
**Then** it shows "No calculations yet. Results you evaluate will show up here."
**And** rows are 48 px high, the drawer traps focus while open, and Escape closes it

### Story 5.5: Search history

As a power user,
I want to search my history,
So that I can find a specific calculation quickly.

**Requirements:** FR18, UX-DR9

**Acceptance Criteria:**

**Given** entries including `84.50*1.18/4`
**When** I search `1.18`
**Then** only entries whose expression or result contains `1.18` are listed

**Given** a search with no matches
**When** it runs
**Then** the drawer shows "No matches for “x”."
**And** clearing the search restores the full list

### Story 5.6: Recall an earlier calculation

As a power user,
I want to bring an old expression back to the display,
So that I can reuse or edit it.

**Requirements:** FR19

**Acceptance Criteria:**

**Given** a History row
**When** I click it or press Enter on it
**Then** its expression loads into the display and the drawer closes

**Given** a History row
**When** I choose "Use result"
**Then** its result loads into the current entry
**And** both actions work from the keyboard

### Story 5.7: Delete one entry or clear all

As a power user,
I want to remove entries,
So that my history stays tidy and private.

**Requirements:** FR20

**Acceptance Criteria:**

**Given** a History row
**When** I press its delete button
**Then** only that entry is removed, and the button's accessible name includes the expression

**Given** the drawer
**When** I choose Clear all
**Then** a confirmation asks "Delete all history? This can't be undone." with "Delete all" and "Keep"
**And** choosing "Delete all" empties the history and "Keep" leaves it unchanged

## Epic 6: Unit Converter

A user converts length, mass, temperature, volume and time in either direction.

### Story 6.1: Units defined as data

As a tutorial author,
I want conversion factors in data files,
So that adding a unit never needs a code change.

**Requirements:** FR23, AR8

**Acceptance Criteria:**

**Given** `backend/src/main/resources/units/*.yaml`
**When** the service starts
**Then** one file per Unit Category (length, mass, temperature, volume, time) is loaded with exact decimal-string factors

**Given** `GET /api/v1/units`
**When** it is called
**Then** it returns the categories and their units
**And** adding a new unit entry to a YAML file makes it appear in the response without changing code, proven by a test
**And** `api/openapi.yaml` and the generated types are updated

### Story 6.2: Convert length, mass, volume and time

As a power user,
I want to convert between units,
So that I get exact answers without looking up factors.

**Requirements:** FR21

**Acceptance Criteria:**

**Given** `POST /api/v1/conversions` with 12.5 miles to kilometres
**When** it is posted
**Then** the result is `"20.1168"`

**Given** a conversion between units in the same category
**When** it is posted
**Then** it uses `BigDecimal` math from the data file only

**Given** units from different categories or an unknown unit
**When** it is posted
**Then** the server returns 422 with a stable error `code`

### Story 6.3: Temperature conversions

As a power user,
I want Celsius, Fahrenheit and Kelvin to convert correctly,
So that temperature results are right even though they use offsets.

**Requirements:** FR21

**Acceptance Criteria:**

**Given** 100 °C
**When** it is converted to °F
**Then** the result is `"212"`

**Given** 0 K
**When** it is converted to °C
**Then** the result is `"-273.15"`
**And** offsets are defined in the temperature YAML entry, not hard-coded in logic

### Story 6.4: Converter view with two linked fields

As a power user,
I want a converter screen where editing either side updates the other,
So that I can go in either direction.

**Requirements:** FR22, UX-DR17

**Acceptance Criteria:**

**Given** the top bar
**When** I choose Converter (or press Alt+4)
**Then** I see category tabs, two fields with unit pickers, and a swap button

**Given** I edit the "from" field
**When** I stop typing
**Then** the "to" field updates, and editing "to" updates "from"

**Given** the swap button
**When** I press it
**Then** the units swap while the values stay consistent
**And** the unit list comes from `GET /api/v1/units`, the Converter view does not show the Trace Panel, and all controls are keyboard operable with accessible names

## Epic 7: Programmer Mode

A developer converts between BIN, OCT, DEC and HEX and uses bitwise operations at a chosen word size.

### Story 7.1: Programmer view with four-base display

As a developer,
I want to enter a value in one base and see it in all four,
So that I can read register values quickly.

**Requirements:** FR24, AR9, UX-DR18

**Acceptance Criteria:**

**Given** the top bar
**When** I choose Programmer (or press Alt+3)
**Then** I see a BIN/OCT/DEC/HEX selector, hex keys A-F, and the current value shown in all four bases

**Given** HEX is selected
**When** I enter `FF`
**Then** the display shows BIN `11111111`, OCT `377`, DEC `255` and HEX `FF`

**Given** a base
**When** it is selected
**Then** keys not valid in that base are disabled
**And** the request uses `mode=PROGRAMMER` with `base`, the engine uses `BigInteger`, and the default Word Size is 32 bits
**And** with the Trace Panel open, evaluating in Programmer view updates the trace as it does in Simple and Scientific views

### Story 7.2: Word size selection

As a developer,
I want to pick 8, 16, 32 or 64 bits,
So that results behave like my target hardware.

**Requirements:** FR26

**Acceptance Criteria:**

**Given** the Word Size selector
**When** I choose 8, 16, 32 or 64
**Then** values are limited to that many bits (bitwise results are masked to it from Story 7.3), and the choice is shown in the view

**Given** 8-bit selected
**When** I enter a value larger than 255
**Then** a plain-language error says the value does not fit in 8 bits (Phase I: values are non-negative and unsigned; signed display and overflow behavior are deferred to Phase II)
**And** changing the Word Size re-evaluates the current value and updates all four base displays

### Story 7.3: Bitwise operations

As a developer,
I want AND, OR, XOR, NOT and shifts,
So that I can work out masks and flags.

**Requirements:** FR25

**Acceptance Criteria:**

**Given** 16-bit and `0xFF OR 0xF00`
**When** it is evaluated
**Then** the result is HEX `FFF` (DEC `4095`)

**Given** 8-bit and `NOT 0`
**When** it is evaluated
**Then** the result is `255` (unsigned)

**Given** `1 << 3` and `16 >> 2`
**When** they are evaluated
**Then** the results are `8` and `4`
**And** AND, OR, XOR, NOT and the shifts use the operator key color and have accessible names
