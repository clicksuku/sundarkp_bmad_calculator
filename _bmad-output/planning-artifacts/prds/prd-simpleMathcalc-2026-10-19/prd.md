---
title: CalcHub
status: final
created: 2026-10-19
updated: 2026-10-21
---

# PRD: CalcHub
*Working title — confirm.*

## 0. Document Purpose

This PRD defines CalcHub, a small web calculator built as the running example for a tutorial on the BMAD Method. It is written for the author, tutorial readers, and the downstream workflow owners (Architect, UX, Epics/Stories, Dev, QA). Vocabulary is anchored in the Glossary (§3); features are grouped in §4 with globally numbered FRs nested under them; cross-cutting NFRs are in §5; assumptions are tagged inline and indexed in §11. Technical constraints (stack, module layout, test tooling) are recorded in `addendum.md`, not here. Because CalcHub is a tutorial artifact, the PRD is kept short enough to show live while still carrying real traceability: Requirement → Epic → Story → Acceptance Criteria.

## 1. Vision

CalcHub is a fast, accurate, keyboard-friendly web calculator that starts as a basic four-function calculator and grows into a scientific and utility toolkit: expression entry, scientific functions, memory, history, unit conversion, programmer mode, and saved preferences.

Its distinguishing feature is the **Pipeline Trace Visualizer**: for any expression, CalcHub shows the real steps it took to evaluate it (tokenize, parse, plan, evaluate), grouped under the four BMAD phase names as an explanatory analogy. A tutorial reader sees, in a domain they already understand, how a staged pipeline turns an input into a verified result.

The domain is deliberately simple so the audience can focus on process and artifacts. It is still treated as a real product: correct arithmetic, accessible UI, testable requirements, and clear module boundaries.

## 2. Target User

### 2.1 Jobs To Be Done

- **Everyday user:** get a quick, correct answer to simple arithmetic and percentages without thinking about the tool.
- **Student/engineer:** evaluate parenthesized and scientific expressions with a chosen angle unit and trust the result.
- **Developer:** convert between number bases and try bitwise operations at a given word size.
- **Power user:** recall earlier calculations, stash values in memory, and convert units without leaving the app.
- **Workshop learner:** see how an expression is actually evaluated, step by step, to understand a staged pipeline.
- **Tutorial author (the builder):** have a small, realistic product whose artifacts demonstrate each BMAD phase.

### 2.2 Non-Users (v1)

Users who need graphing, symbolic algebra, live-currency conversion, shared or multi-user workspaces, or a native mobile app.

### 2.3 Key User Journeys

*Lighter form (solo tutorial project). FRs reference journeys by ID.*

- **UJ-1. Maya works out a restaurant bill.** Maya, splitting dinner, types `84.50 * 1.18 / 4` with the keyboard and gets `24.9275` with no floating-point artifacts, then applies `%` for a tip. A mistyped `84.50 / 0` shows a plain-language error instead of a crash.
- **UJ-2. Dev checks a physics expression.** Dev, an engineering student, switches the angle unit to DEG and enters `2 + 3 * (4 - 1)^2 + sin(30)`. A typo (`2 + * 3`) is rejected with the error position marked. He fixes it, gets the result, and the next day finds his DEG setting still applied.
- **UJ-3. Priya masks a register value.** Priya, a firmware developer, switches to Programmer mode, selects 16-bit word size, enters `0xFF`, and ORs it with `0x0F00`. She sees the same value in BIN, OCT, DEC and HEX.
- **UJ-4. Sam re-uses an earlier result.** Sam, a project estimator, opens History, searches for "1.18", recalls an old expression into the display, stores a result with M+, and converts 12.5 miles to kilometres in the unit converter.
- **UJ-5. Lena sees how `2 + 3 * 4` is evaluated.** Lena, attending a BMAD workshop, enters an expression and opens the trace panel beside the calculator. She steps through tokens, the AST, the evaluation order, and the intermediate values, sees the phase labels, and reads the notice that BMAD is a development-time workflow, not a runtime component.

## 3. Glossary

- **Calculation** — one evaluation of an Expression or Basic-mode operation chain, producing a Result or an Error.
- **Expression** — a text input combining numbers, operators, parentheses, functions, and constants, evaluated by Expression mode.
- **Result** — the Decimal value produced by a Calculation, rounded to the configured Precision for display.
- **Error** — a user-facing failure of a Calculation (division by zero, invalid input, syntax error, complexity limit) with a message and, for syntax errors, a character position.
- **Precision** — number of significant digits shown for a Result (default 15).
- **Angle Unit** — DEG, RAD, or GRAD; applies to trig functions.
- **Memory Register** — the single stored value manipulated by MC, MR, M+, M-, MS; stored in the browser in Phase I.
- **History Entry** — a record of a Calculation (expression, result, timestamp, mode), stored in the browser in Phase I.
- **Mode** — Basic, Expression, Scientific, or Programmer calculation context recorded on each History Entry.
- **Unit Category** — length, mass, temperature, volume, or time; each has units and conversion data.
- **Word Size** — 8, 16, 32, or 64 bits; bounds Programmer-mode values.
- **Phase I / Phase II** — delivery phases. Phase I is the working calculator with no user identity; Phase II adds identity and the deferred items listed in §8.
- **Preference** — a saved setting (theme, Precision, default Angle Unit, number grouping), stored in the browser in Phase I.
- **Trace** — the ordered list of Trace Steps generated by one Calculation.
- **Trace Step** — one real pipeline action (tokenize, parse, plan, evaluate) with per-step detail.
- **Phase Label** — one of Analysis, Planning, Solutioning, Implementation, assigned to Trace Steps as an analogy only.
- **Trace Panel** — the UI pane displaying the Trace beside the calculator.

## 4. Features

### 4.1 Basic Calculator (Tier 1, MVP)

**Description:** The default mode: a button-and-keyboard calculator for everyday arithmetic with exact decimal results. Chained operations behave as on a standard calculator. Errors never crash the display and are explained in plain language. Realizes UJ-1.

**Functional Requirements:**

#### FR-1: Four-function arithmetic
A user can add, subtract, multiply, and divide numbers. Realizes UJ-1.

**Consequences (testable):**
- `0.1 + 0.2` displays `0.3`.
- `1 / 3` displays a Result rounded to the configured Precision.

#### FR-2: Decimal input and sign toggle
A user can enter decimal numbers and toggle the sign of the current entry with `+/-`.

**Consequences (testable):**
- Only one decimal point is accepted per number.
- `+/-` on `5` shows `-5`; pressing again shows `5`.

#### FR-3: Percent
A user can apply `%` to the current entry. After an operator, % means "percent of the left operand" (200 + 10% = 220); standalone, % divides by 100 (50% = 0.5).

#### FR-4: Clear, clear entry, backspace
A user can clear everything (C), clear the current entry (CE), or delete the last digit (backspace).

**Consequences (testable):**
- CE keeps the pending operation and left operand.
- C resets display, pending operation, and operands (not Memory Register or History).

#### FR-5: Chained operations
A user can chain operations (`2 + 3 × 4 =`) with correct display behavior. Basic mode evaluates left to right as entered, as on a standard handheld calculator; operator precedence is provided by Expression mode (FR-9).

#### FR-6: Error handling
The system shows a clear, non-technical Error for division by zero and invalid input, and lets the user continue without a reload.

**Consequences (testable):**
- `5 / 0` shows "Cannot divide by zero".
- After an Error, pressing C or a digit resumes normal entry.

#### FR-7: Keyboard support and accessible layout
Every Basic-mode action is operable from the keyboard, and buttons have accessible names and a logical focus order. Realizes UJ-1.

**Consequences (testable):**
- Digits, operators, Enter/=, Backspace, Escape (C), and `.` work from the keyboard.

#### FR-8: Exact decimal arithmetic
All arithmetic uses arbitrary-precision decimal math.

**Consequences (testable):**
- No floating-point artifacts: `0.1 + 0.2 = 0.3`, `1.1 * 1.1 = 1.21`.

### 4.2 Expression Mode (Tier 2-A)

**Description:** The user types a complete Expression with parentheses and standard operator precedence. Syntax problems are reported with the position of the first problem. Realizes UJ-2.

**Functional Requirements:**

#### FR-9: Evaluate expressions with precedence
A user can enter an Expression and evaluate it with standard precedence, associativity, and parentheses. Realizes UJ-2.

**Consequences (testable):**
- `2 + 3 * (4 - 1)^2` evaluates to `29`.
- Unary minus is supported (`-(2+3)` = `-5`).

#### FR-10: Syntax errors with position
The system rejects an invalid Expression with an Error that states the character position of the problem.

**Consequences (testable):**
- `2 + * 3` reports position 5 (1-based) (1-based character index, confirmed).
- Unbalanced parentheses are reported.

### 4.3 Scientific Functions (Tier 2-B)

**Description:** Scientific functions and constants usable in Expression mode and through Scientific buttons, with a global Angle Unit toggle. Realizes UJ-2.

**Functional Requirements:**

#### FR-11: Scientific functions
A user can use sin, cos, tan, asin, acos, atan, log10, ln, sqrt, x^y, x^2, 1/x, n!, and abs.

**Consequences (testable):**
- `sqrt(16)` = `4`; `5!` = `120`; `log10(1000)` = `3`.
- Domain errors (`sqrt(-1)`, `ln(0)`, `asin(2)`, non-integer or negative factorial) produce an Error, not `NaN`.

#### FR-12: Constants
A user can use the constants `pi` and `e`.

#### FR-13: Angle Unit toggle
A user can switch the Angle Unit between DEG, RAD, and GRAD, and the current unit is visible.

**Consequences (testable):**
- `sin(30)` in DEG = `0.5`; `sin(pi/6)` in RAD = `0.5` (to Precision).

### 4.4 Memory Registers (Tier 2-C)

**Description:** Classic memory keys backed by a Memory Register stored in the browser. Realizes UJ-4.

**Functional Requirements:**

#### FR-14: Memory operations
A user can MC (clear), MR (recall), M+ (add), M- (subtract), and MS (store) the Memory Register. Realizes UJ-4.

**Consequences (testable):**
- MS stores the displayed value; M+ adds to it; M- subtracts; MR recalls; MC sets to zero.
- An indicator shows when the Memory Register is non-zero.

#### FR-15: Memory persistence
The Memory Register persists across page reloads and browser restarts. In Phase I, Memory, History, and Preferences are stored in the browser only (no server-side persistence and no user identity); server-side per-user storage arrives in Phase II.

### 4.5 History (Tier 2-D)

**Description:** Every evaluated Calculation is stored in the browser and can be browsed, searched, recalled, and deleted. Realizes UJ-4.

**Functional Requirements:**

#### FR-16: Record history
The system stores a History Entry (expression, result, timestamp, Mode) for every successfully evaluated Calculation. Calculations that produce an Error are not stored.

#### FR-17: List and paginate
A user can view History Entries newest-first in pages.

**Consequences (testable):**
- Page size is 20; paging controls appear when entries exceed one page.

#### FR-18: Search history
A user can search History Entries by text in the expression or result.

#### FR-19: Recall into display
A user can select a History Entry to load its expression (or result) into the display. Realizes UJ-4.

#### FR-20: Delete one / clear all
A user can delete a single History Entry or clear all, with confirmation before clear-all.

### 4.6 Unit Converter (Tier 2-E)

**Description:** Conversion across five Unit Categories with bidirectional entry. Conversion factors are data, so adding a unit needs no code change. Realizes UJ-4.

**Functional Requirements:**

#### FR-21: Convert between units
A user can select a Unit Category (length, mass, temperature, volume, time) and convert a value between two units. Realizes UJ-4.

**Consequences (testable):**
- 12.5 miles → `20.1168` km (exact factor).
- Temperature supports offset conversions (°C ↔ °F ↔ K).

#### FR-22: Bidirectional entry
Editing either the "from" or "to" field updates the other.

#### FR-23: Data-defined conversion factors
Units and factors are defined as data (not hard-coded in logic); adding a unit requires only a data change.

### 4.7 Programmer Mode (Tier 2-F)

**Description:** Integer-oriented mode with base conversion and bitwise operations bounded by a Word Size. Realizes UJ-3.

**Functional Requirements:**

#### FR-24: Base conversion
A user can enter a value in BIN, OCT, DEC, or HEX and see it in all four bases. Realizes UJ-3.

#### FR-25: Bitwise operations
A user can apply AND, OR, XOR, NOT, and shifts (left/right).

**Consequences (testable):**
- With 8-bit Word Size, `NOT 0` = `255` (unsigned; signed display is deferred to Phase II).

#### FR-26: Word Size selection
A user can select 8, 16, 32, or 64-bit Word Size; results are constrained to it.

**Consequences (testable):**
- Phase I: values are non-negative and unsigned. Overflow handling beyond this is deferred to Phase II.

### 4.8 Preferences (Tier 2-G)

**Description:** Settings saved in the browser and applied on every load. Realizes UJ-2.

**Functional Requirements:**

#### FR-27: Set preferences
A user can set theme (light/dark/system), Precision, default Angle Unit, and number grouping.

#### FR-28: Persist and apply
Preferences are saved in the browser (see FR-15) and applied at load, before first render of results where feasible.

**Consequences (testable):**
- After reload, theme, Precision, default Angle Unit, and grouping match the saved values.
- "System" theme follows the OS setting.

### 4.9 Pipeline Trace Visualizer (Tier 2-H, demo feature)

**Description:** For any Expression, a Trace of the real evaluation pipeline is shown in a Trace Panel beside the calculator. Trace Steps are grouped under Phase Labels named after BMAD as an explanatory analogy; the UI states that BMAD is a development-time workflow, not a runtime component. Realizes UJ-5.

**Functional Requirements:**

#### FR-29: Generate a real Trace
For a Calculation, the system produces a Trace covering tokenize, parse (AST), plan (precedence/evaluation order), and evaluate. Realizes UJ-5.

**Consequences (testable):**
- Trace Steps reflect the actual AST evaluation order; no simulated or placeholder steps.
- For `2 + 3 * 4`, multiplication is traced before addition, with intermediate value `12`.

#### FR-30: Per-step detail
Each Trace Step shows its relevant detail: tokens, AST node, or intermediate value.

#### FR-31: Trace metrics
The Trace Panel shows step count and total duration.

#### FR-32: BMAD phase grouping
Trace Steps are grouped under the Phase Labels Analysis, Planning, Solutioning, and Implementation. Mapping: tokenize→Analysis, parse→Planning, plan→Solutioning, evaluate→Implementation (confirmed).

**Consequences (testable):**
- The panel shows a visible notice that the phases are an analogy and BMAD is a development-time workflow, not a runtime component.

#### FR-33: Split-pane layout
The calculator is on the left and the Trace Panel on the right; on small screens the panel collapses and can be expanded.

#### FR-34: Keyboard-accessible Trace Panel
All Trace Panel controls (expand/collapse, step navigation) are operable from the keyboard with visible focus.

**Feature-specific NFRs:**
- Trace generation adds < 30 ms p95 for typical Expressions (see NFR-2).
- The Trace is produced by an `explainability` capability that consumes trace events published by the calculation capability, with no dependency on its internals (see `addendum.md`).

**Notes:** `[NOTE FOR PM]` Trace delivery (single response vs. streamed) is an Architect decision recorded as an ADR.

## 5. Cross-Cutting Non-Functional Requirements

- **NFR-1 Performance:** Server-side evaluation p95 < 100 ms for Expressions under 200 characters. *Indicative target in Phase I, not a release gate.*
- **NFR-2 Trace overhead:** Generating a Trace adds < 30 ms p95 for typical Expressions (under 50 characters). *Indicative target in Phase I, not a release gate.*
- **NFR-3 Accuracy:** Results are correct to the configured Precision (default 15 significant digits).
- **NFR-4 Accessibility:** WCAG 2.1 AA; full keyboard operation of all features including the Trace Panel.
- **NFR-5 Security:** Enforce Expression length limits and complexity limits (e.g. nesting depth); Expressions are parsed by a safe parser and never executed as code.
- **NFR-6 Observability:** Structured logging, health endpoints, and basic metrics.
- **NFR-7 Quality:** Automated test coverage of at least 80% on the calculation and explainability capabilities.
- **NFR-8 Data isolation:** *(Phase II)* Memory Register, History, and Preferences are visible only to their owning user. Not required in Phase I.

## 6. Constraints and Guardrails

- **Process:** Requirements are numbered (FR-n, NFR-n) with traceability Requirement → Epic → Story → Acceptance Criteria.
- **Scope discipline:** No scope beyond this document; additions must be flagged as assumptions or open questions.
- **Storage (Phase I):** Memory Register, History, and Preferences are stored in the browser. The server keeps no user data, and no database (PostgreSQL or otherwise) is required in Phase I.
- **Technology and architecture:** see `addendum.md` (modular monolith, Java/Spring Boot, React/TypeScript, and so on).

## 7. Non-Goals (Explicit)

- Graphing and symbolic algebra.
- Currency conversion with live rates.
- Multi-user collaboration.
- User identity, accounts, or authentication in Phase I (deferred to Phase II).
- A server-side database in Phase I (PostgreSQL is not required; it may return in Phase II with user identity).
- Native mobile apps.
- Payments or monetization.
- Any AI/LLM calls at runtime (BMAD itself is a development-time workflow, not a runtime component).

## 8. MVP Scope

### 8.1 In Scope
- **MVP:** Basic Calculator (FR-1 to FR-8).
- **Phase I, remaining (in the order of §4):** Expression mode, Scientific, Memory, History, Unit Converter, Programmer mode, Preferences, Pipeline Trace Visualizer. No user identity: persisted data (Memory, History, Preferences) is held in the browser, and no database is required.

### 8.2 Out of Scope for Phase I
- Everything in §7.
- **Phase II:** user identity and per-user scoping of Memory, History, and Preferences (NFR-8); signed (two's complement) display and word-size overflow semantics in Programmer mode (FR-25, FR-26).

`[NOTE FOR PM]` The Trace Visualizer depends on Expression mode (FR-9), so those two must ship together or in that order.

## 9. Success Metrics

- **SM-1:** The calculator works: every in-scope FR meets its acceptance criteria. Validates FR-1 to FR-34.

No further metrics or counter-metrics are required for this tutorial project.

## 10. Open Questions

None blocking Phase I.

**Resolved**
- Artifact location: all BMAD artifacts live under `_bmad-output` and its sub-folders (`/docs` is not used).
- Basic and Expression modes: one display accepts both.
- Session and identity: not required in Phase I; deferred to Phase II.
- Programmer-mode signed display and overflow semantics: deferred to Phase II.

## 11. Assumptions Index

- §4.1 FR-3: percent semantics (relative to left operand after an operator; ÷100 standalone) — confirmed.
- §4.1 FR-5: Basic mode evaluates left to right as entered — confirmed.
- §4.4 FR-15: Phase I persistence is browser-only (decided by the author) — confirmed.
- §4.5 FR-16, FR-17: Error Calculations are not stored; History page size 20 — confirmed.
- §4.2 FR-10: error position is 1-based — confirmed.
- §5 NFR-1, NFR-2: performance targets are indicative in Phase I, not release gates — confirmed.
