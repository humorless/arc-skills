---
name: arc
description: >
  Use when writing or reviewing code in an ARC-compliant codebase, or when
  establishing structural rules for a new session. Load at session start.
---

# ARC — Agent Ready Codebase

This skill encodes six structural principles. Apply all that are relevant to the
current codebase. Each principle includes an applicability note so you can skip
what does not apply.

---

## 1. Strict Doc String

**Applies to:** every file, module, namespace, or class that groups related logic.

### Rules

- Every module/file must have a top-level docstring that states its **single
  responsibility** in one or two sentences.
- If functions exist whose behavior falls outside that stated responsibility,
  either update the docstring to reflect the expanded scope, or extract those
  functions into a more appropriate module.

### Violation signals

- No docstring at all.
- Docstring describes more than one distinct responsibility.
- The module's actual contents contradict or exceed what the docstring claims.

---

## 2. Data Spec

**Applies to:** any codebase that passes structured data across module or service
boundaries (backends, APIs, data pipelines, etc.).

### Rules

- Every data structure that crosses a module boundary must have a **named
  schema or type** defined in a dedicated location.
- Never define schemas inline at the call site.
- The dedicated schema location is the single source of truth for all
  cross-boundary data shapes.

### Language-idiomatic equivalents

This principle is language-agnostic. Use the idiomatic tool for the current
codebase:

| Language / ecosystem | Schema tool | Dedicated location |
|---|---|---|
| Python | Pydantic `BaseModel` or `TypedDict` | `app/schemas.py` or `app/types.py` |
| TypeScript | Zod schema or `interface` | `src/types.ts` or `src/schemas/` |
| Go | named `struct` | `internal/types/` package |
| Clojure | malli or spec | `myapp.schema` namespace |
| Other | idiomatic equivalent | a single, dedicated file or module |

### Violation signals

- An anonymous dict, map, or plain object crosses a module boundary.
- A schema is defined inline at the call site rather than in the dedicated
  schema location.
- Two modules define separate schemas for the same data shape.

---

## 3. Functional Programming

**Applies to:** all codebases. In OOP-dominant languages, apply the spirit
rather than the letter (see note below).

### Rules

- Every function should be **pure by default**: given the same input, always
  return the same output, with no observable side effects.
- Never mutate arguments or external state inside a function. If transformation
  is needed, return a new value.
- Side effects (I/O, database writes, HTTP calls, logging, randomness) are only
  allowed at explicitly designated boundary layers.

### Note for OOP languages

In Java, C#, Python OOP, and similar: the equivalent rule is that methods should
not mutate external or shared state; side effects should be isolated to
service/repository/adapter layers, not scattered through domain logic.

### Violation signals

- A function mutates an argument or global variable.
- A function performs I/O but is not in a designated boundary layer.
- A function's return value is ignored and it is called only for its side effect,
  but it is placed inside domain logic rather than at the boundary.

---

## 4. Policy / Mechanism Separation

**Applies to:** all codebases.

### Rules

- **Mechanism** is code that is unopinionated, context-free, reusable across
  domains, easy to test in isolation, and changes slowly.
- **Policy** is code that reflects current business decisions, is tied to a
  specific domain, and is expected to change frequently.
- Separate the two. Mechanism belongs in its own module or layer. Policy belongs
  at the edges.
- Unit tests should be written on **mechanism**, not on policy.

### Diagnostic questions

- Does this code accept all its dependencies as explicit parameters, or does it
  rely on defaults, globals, or implicit context? (More explicit → mechanism)
- If you needed to reuse this logic elsewhere, would you call it with different
  parameters, or copy-paste and modify it? (Parametric reuse → mechanism)
- Is this code expressing a stable technical fact, or a current business
  decision? (Business decision → policy)

### Violation signals

- Hard-coded business constants inside a utility or helper function.
- A generic-looking function that silently assumes a specific domain context.
- Tests that can only be written by mocking business state rather than passing
  plain values.

---

## 5. Container / Presenter Separation

**Applies to:** codebases with a UI component layer (React, Vue, Svelte,
ClojureScript/Reagent, etc.).
**Skip if:** no UI layer is present.

### Rules

- Every UI component belongs to exactly one category: **container** or
  **presenter**. Never mix both responsibilities in a single component.
- A **presenter** renders UI from props only. It knows nothing about where data
  comes from, how it is fetched, or what happens after a user interaction beyond
  calling a callback it was given.
- A **container** owns data fetching, state management, and business logic. It
  knows nothing about how things look. Its only rendering job is to pass data
  down to a presenter.
- When modifying an existing mixed component, prefer splitting it rather than
  extending the mixing further.

### Violation signals

- A component both fetches data and renders detailed UI.
- A presenter component contains conditional logic based on fetched state.
- A container component contains layout or styling decisions.

---

## 6. Polylith (Code Boundary ≠ Deployment Boundary)

**Applies to:** projects large enough to have multiple services or deployment
targets, or any project where shared business logic is being copy-pasted between
modules.
**Skip if:** single-service project with no shared logic problem.

### Core idea

Do not conflate **code boundaries** with **deployment boundaries**. They are
independent decisions:

- **Code boundary** (component/module): unit of logical separation and reuse.
- **Deployment boundary** (project/service): unit of independent deployment.

A component can be shared across many deployment targets without becoming a
separate service.

### Rules

- When the same business logic is needed in two places, the answer is neither
  copy-paste nor extracting a new microservice. It is extracting a shared
  component that both deployment targets depend on.
- Code boundaries should be decided by domain cohesion, not by what happens to
  be deployed together.
- If you find yourself about to copy business logic into a second location, stop
  and ask the user whether a shared module should be extracted first.

### Violation signals

- The same business logic exists in two places because "they're different
  services."
- A module was split into a separate service solely to share code, not because
  it needed independent scaling or deployment.
- Changing a shared rule requires editing multiple files in multiple services.
