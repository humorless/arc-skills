# arc-skills

A [Claude Code](https://claude.ai/code) plugin that encodes the **Agent Ready
Codebase (ARC)** principles — six structural practices that keep a codebase
legible and navigable for AI agents.

## Why ARC?

As codebases grow, agent capability degrades systematically. Agents miss
dependencies, ignore side effects, misread business logic, and copy-paste
instead of integrating. The root cause is structural ambiguity: when boundaries
are blurry, agents cannot make consistent decisions.

ARC addresses this with six principles, each designed so the answer to a
structural question is visible from the code itself — without reading the full
implementation.

| Principle | Question the agent can skip reading for |
|---|---|
| Strict Doc String | What does this module do? |
| Data Spec | What shape is this data? |
| Functional Programming | Does this function change external state? |
| Policy / Mechanism | Where are the business decisions? |
| Container / Presenter | Where does this data come from? |
| Polylith | What does this change affect? |

## Installation

Add this repository as a plugin source and install:

```
/plugin marketplace add humorless/arc-skills
/plugin install arc-skills@arc-skills
```

### Updating

```
/plugin marketplace update arc-skills
/plugin update arc-skills@arc-skills
```

## Skills

### `/arc-skills:arc` — ARC principles (load once, follow throughout)

Loads all six ARC principles as behavioral rules. Claude will follow them
while writing or modifying code in the current session.

Each principle includes an `Applies to / Skip if` note, so Claude
self-selects which principles are relevant to the current codebase
(e.g., Container/Presenter is skipped automatically for backend-only projects).

**Usage:** invoke at the start of a session when working in an ARC-compliant
codebase, or when you want Claude to write new code that follows ARC principles.

```
/arc-skills:arc
```

---

### `/arc-skills:refactor-pm` — Policy / Mechanism analysis

Scans the specified files or directory for opportunities to improve
mechanism/policy separation. Reports:

- Mixed code (where the boundary should be drawn)
- Extraction candidates (mechanism buried in policy)
- Misplaced policy (business logic in utility code)
- Proposed before/after refactorings
- Unit test opportunities on extracted mechanism

```
/arc-skills:refactor-pm src/
/arc-skills:refactor-pm src/services/payment.py
```

---

### `/arc-skills:refactor-container-presenter` — Container/Presenter analysis

Scans UI components for violations of the Container/Presenter separation
principle. Reports:

- Classification of each component (container, presenter, or mixed)
- Specific violations with evidence
- Proposed split: what the new container and presenter would each own

Applies only to codebases with a UI component layer (React, Vue, Svelte,
ClojureScript/Reagent, etc.).

```
/arc-skills:refactor-container-presenter src/components/
/arc-skills:refactor-container-presenter src/components/UserDashboard.tsx
```

---

### `/arc-skills:refactor-polylith` — Code/Deployment boundary analysis

Scans the codebase for violations of the principle that code boundaries and
deployment boundaries are independent decisions. Reports:

- Duplicated business logic across services (copy-paste sharing)
- Premature service extraction (services that exist only to share code)
- Boundary misalignment (code structure mirrors deployment structure)
- Proposed restructuring at the directory/module level

```
/arc-skills:refactor-polylith .
/arc-skills:refactor-polylith services/
```

## Background

These principles were presented in the talk **ARC (Agent Ready Codebase)** by
[Laurence Chen](https://github.com/humorless). The Policy/Mechanism principle
is based on Arne Brasseur's definitions. The Polylith principle draws from the
[Polylith architecture](https://polylith.gitbook.io/).

## License

MIT
