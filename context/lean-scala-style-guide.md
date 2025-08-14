# Lean Scala Style Guide

*A pragmatic functional approach for modern Scala 3 projects.*

> **Audience** – Written for humans **and** AI coding agents.  Statements like “you SHOULD…” apply to the code you emit; “avoid…” means **never** emit such code unless explicitly asked.

---

## 1  Purpose & Scope

* Encourage **simple, transparent, and maintainable** Scala 3 code bases that leverage:
  * the JDK 24+ **Project Loom** virtual‑thread runtime, and
  * **Ox 1.0** for structured error handling and concurrency helpers, plus strict Scala evaluation semantics.
* Embrace a **functional‑first** style without dogma: aim for an **≈ 80 % pure functional core / 20 % imperative shell**.  Keep business logic pure where practical and push unavoidable effects to well‑defined edges.
* APIs must be **lawful** — given identical arguments *and identical explicit dependencies*, they yield the same observable result (value **or** typed error). APIs should not be unpredictable or depend on unrelated state changes. This guarantee is weaker than that of purity.
* Mandate **truthful APIs** and make **illegal states unrepresentable** (“Parse, don’t validate”).

---

## 2  Core Principles

* **P1 Truthful APIs** – Recoverable outcomes are encoded in types—no hidden nulls or unchecked exceptions.
* **P2 Make Illegal States Unrepresentable** – Design domain models so invalid data cannot be constructed.
* **P3 Functional Simplicity** – Prefer immutable data and declarative transformations; abandon purity only when it harms clarity or performance.
* **P4 Structured Error Handling** – Compose `Either` / `Try` with Ox’s `either:` blocks instead of ad‑hoc try‑catch chains.
* **P5 Simplicity Beats Cleverness** – A few extra lines are cheaper than abstraction leaks or type gymnastics.

---

## 3  Toolchain Requirements

* **Scala** – use the **latest stable release** for applications; for libraries target the **latest LTS** series.
* **JDK** – **24 or newer**.
* **Ox** – **1.0** (currently *RC2*).
* **Scalafmt** – All code **MUST** be formatted with *scalafmt*; version is project‑specific.

---

## 4  Error Handling & Validation

1. **Return `Either[E, A]` or `Try[A]` for fallible operations.**  Use `Either` for domain‑level errors; `Try` for wrappers around exception‑throwing APIs.
2. **Exceptions signal defects.**  Throw only when the program is already in undefined territory.
3. **either:** **blocks** – declaratively compose success paths while short‑circuiting on the first error.  `.ok()` extracts from `Either` **and** `Try`.
4. **Wrapping throwing APIs** – prefer `either.catching[E]` or `Try { … }.toEither`.
5. **Typed errors** – define sealed‑trait hierarchies; avoid “stringly‑typed” errors.
6. `Option` is **not** for errors; use it only for **presence/absence** where absence is perfectly ordinary.
7. **Never use `require` or `assert` in constructors**; the correct way to validate case class arguments on creation is to create a smart constructor.

---

## 5  Resource Management

* Prefer `Using` or `Using.Manager` over manual `try/finally`, especially when composing multiple resources.
* Within Ox **structured‑concurrency scopes**, shared resources should be handled by the `useInScope` family of combinators so they are released exactly when the scope ends.

---

## 6  Function Design Rules

* **F1 Truthful Signatures** – Return types reveal all recoverable outcomes; no hidden nulls or unchecked exceptions.
* **F2 Prefer Pure Functions** – Strive for an ≈ 80 % pure codebase.  Pure functions may depend on capability interfaces but perform no I/O themselves.
* **F3 Keep Functions Focused** – One coherent task per function; keep them short enough to scan.
* **F4 Explicit Dependencies** – Pass required services (`Clock`, `Random`, `Files`, etc.) explicitly; avoid hidden globals or thread‑locals. Services like that can be captured by a class constructor and used by multiple class methods to keep signatures simple.
* **F5 Return Domain Types, Not Primitives** – Use opaque types or small case classes (`UserId`, `Money`, …) instead of raw primitives whenever they carry domain constraints.
* **F6 Eliminate Boolean Blindness** – Swap Boolean flags for small enums / ADTs so intent is explicit and exhaustiveness is checked.
* **F7 Declare Return Types** – Every public function must specify its return type; this prevents accidental signature drift during refactors.
* **F8 Prefer Total Functions** – Cover every valid input; capture failure in the return type rather than throwing or using partial functions.
* **F9 Limit Implicit (`given`) Parameters** – Reserve context parameters for true ambient context (logging, clock, Ox scope).  Pass everything else explicitly to keep call‑sites obvious.
* **F10 Composition Over Inheritance** – Build behaviour by function composition or small traits; inheritance hierarchies are a last resort.
* **F11 Design for Testability** – Architect functions so pure logic is isolated and effectful collaborators are passed as parameters—facilitating unit tests without heavy mocks.

---

## 7  Data & State

* **Illegal states are unrepresentable** – model invariants via enums, case classes, opaque types, smart constructors, or refined types.
* Default to immutable `case class`es and `enum`s; use local mutation only inside private helpers when it cannot leak.
* When concurrency demands mutation, encapsulate it in `Atomic*`, `ConcurrentHashMap`, or Ox‑managed structures; avoid sharing mutable globals.

---

## 8  Structured Concurrency

* Use high-level Ox combinators (`mapPar`, `race`, `timeout`, `par`) to work with concurrency and parallelism.
* **No shared mutable state across fibers**; communicate through immutable messages via Ox channels or concurrency-safe queues.

---

## 9  Testing Guidelines

* **Design for testability first** – pure business logic should be invocable without touching I/O; inject collaborators for side‑effects.
* **Framework** – use **MUnit**; add **ScalaCheck** property tests where algebraic laws or invariants exist.
* Prefer **deterministic** tests; use in‑memory DBs, stub HTTP servers, or Testcontainers for integration boundaries.

---

## 10  Logging & Monitoring

* Use **Scribe** for structured, asynchronous logging.
* Inject a `scribe.Logger` per module; avoid global, mutable logging state.

---

## 11  Formatting & Linting

* **Every file MUST be formatted with Scalafmt**; commit a shared `.scalafmt.conf`.
* Enable helpful compiler warnings:

  ```bash
  -Xfatal-warnings \
  -Wunused:all \
  -Wvalue-discard \
  -Wnonunit-statement
  ```
* **Scalafix** is optional but encouraged for automated refactors and extra linting.

---

## 12  Performance Hints

* **Lazy evaluation where valuable** – use `lazy val`, views (`.view`), or by‑name parameters to defer expensive work; beware space leaks.
* **Stream / iterate large data** – prefer collection views, iterators, or Ox `Flow`s over materialising whole datasets in memory.

---

## 13  Summary

These rules aim to deliver:

* **Robustness** via truthful types, explicit errors, and immutable data.
* **Predictability** through functional‑first design and lawful APIs.
* **Testability** thanks to pure functions and .
* **Concurrency safety** with Ox’s structured scopes.
* **Sustainable resource use** through targeted laziness and streaming.

Constrain how you write code—gain power in reasoning, testing, and maintenance.