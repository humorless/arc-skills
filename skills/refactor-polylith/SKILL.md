---
name: refactor-polylith
description: >
  Use when the same business logic appears in multiple services, or when a
  service was created solely to share code rather than for operational reasons.
  Specify a path to scan.
argument-hint: <path or directory to scan>
---

Scan the specified codebase or directory for violations of the principle that
code boundaries and deployment boundaries should be independent decisions.

## Core Concept

The Polylith insight applies beyond its specific tooling: the mistake of
conflating "what gets deployed together" with "what belongs together as code"
causes two chronic problems:

- **Copy-paste sharing** — the same business logic duplicated across services
  because sharing across deployment units feels too expensive.
- **Premature service extraction** — logic pulled into its own microservice
  solely to enable sharing, not because it needed independent deployment.

The remedy is to separate the two concerns explicitly:

- **Code boundary** (component/module) — unit of logical cohesion and reuse.
  Decided by domain, not by deployment.
- **Deployment boundary** (project/service) — unit of independent deployment.
  Decided by operational need, not by code structure.

## What to Look For

### Duplicated business logic across deployment units

Look for functions, classes, or modules that implement the same rule or
transformation in more than one place. Common signals:

- Near-identical validation logic in two services
- The same calculation or transformation copy-pasted with minor variations
- Comments like "same as X service" or "copied from Y"

### Premature service extraction driven by sharing

Look for services that exist primarily to be called by other services, with no
independent operational reason (no distinct scaling, latency, or failure
isolation need). Signals:

- A service whose only callers are internal services in the same system
- A service extracted specifically when a second consumer appeared, with no
  other justification
- Thin wrapper services with no logic of their own

### Boundary misalignment

Look for places where the code structure follows the deployment structure rather
than domain cohesion. Signals:

- A single business concept split across multiple modules/services because they
  happen to be deployed separately
- Changes to one business rule require edits in multiple services

## Your Task

For each finding, report:

1. **The problem** — describe the specific duplication or misalignment found,
   with file/module references.

2. **Root cause** — is this copy-paste sharing, premature extraction, or
   boundary misalignment? Explain which of the two boundaries was conflated
   with the other.

3. **Proposed resolution** — suggest a concrete restructuring:
   - For duplication: what shared component/module should be extracted, and
     what both deployment units would depend on instead.
   - For premature extraction: whether the service should be collapsed back
     into a shared library/component and which deployment units would include it.
   - Show the before/after structure at the directory or module level.

## Constraints

- This command analyzes structure, not behavior. Do not propose changes to
  business logic.
- Only flag duplication that reflects a shared business rule, not incidental
  similarity.
- If the codebase uses Polylith tooling (Clojure: `poly` CLI; Python:
  `polylith` library), note whether the existing workspace structure is being
  used correctly and where it is being bypassed.
