---
name: refactor-container-presenter
description: >
  Use when UI components are mixing data-fetching with rendering, or when adding
  features makes a component increasingly hard to test or reason about. Applies
  only to codebases with a UI component layer.
argument-hint: <path or file to scan>
---

Scan the specified files or directory for violations of the Container/Presenter
Separation principle, and report findings with concrete refactoring suggestions.

## Scope

This command applies only to codebases with a UI component layer (React, Vue,
Svelte, ClojureScript/Reagent, or similar). If no UI layer is present, report
that and stop.

## Definitions

**Presenter** — renders UI from props only. It knows nothing about where data
comes from, how it is fetched, or what happens after a user interaction beyond
calling a callback it was given.

**Container** — owns data fetching, state management, and business logic. It
knows nothing about how things look. Its only rendering job is to pass data down
to a presenter.

## What to Look For

For each component examined, determine which category it belongs to, then check
for violations:

**Mixed components** — A single component that both:
- fetches or derives data (API calls, store subscriptions, useEffect with data
  loading, etc.), AND
- contains detailed rendering logic (layout, styling, conditional display of
  UI elements)

**Presenter doing too much** — A component declared as a presenter but:
- accesses global state or context directly
- makes API calls or triggers side effects
- contains business logic beyond simple display formatting

**Container doing too much** — A component declared as a container but:
- contains layout or styling decisions
- renders anything beyond passing data down to child presenters

## Your Task

For each file or component examined, report:

1. **Classification** — Is this currently a container, a presenter, or mixed?
   State what evidence led to this classification.

2. **Violations** — For mixed components, describe specifically what the
   container responsibility is and what the presenter responsibility is, and
   where the boundary should be drawn.

3. **Proposed refactoring** — For each violation, show the proposed split:
   - What the new container component looks like (what it owns, what it passes
     down)
   - What the new presenter component looks like (what props it accepts, what
     it renders)

   Show the before/after structure, not just the principle.

## Constraints

- Do not refactor prematurely. Only flag a component if the mixing is clear and
  the proposed split has an obvious seam.
- Preserve existing behavior. This is a structural refactor, not a rewrite of
  business logic.
- Keep suggestions grounded in the actual code. Do not propose abstractions
  that have no evidence in the current codebase.
