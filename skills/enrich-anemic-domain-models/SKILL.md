---
name: enrich-anemic-domain-models
description: Find anemic domain models in a specified DDD domain/context and guide refactoring toward rich domain models. Use when the user mentions anemic models, rich models, DDD refactoring, domain invariants, business rules in services, or moving behavior into domain entities/value objects.
license: MIT
---

# Refactor Anemic Domain Models

Surface anemic-domain-model candidates in a specified domain/context, then guide one refactor at a time toward a richer model.

This skill is deliberately narrow: it targets domain models only. Do not review DTOs, request/response models, persistence-only projections, generated code, or simple CRUD records unless the user explicitly identifies them as part of the domain model.

## Language

Use [LANGUAGE.md](LANGUAGE.md) exactly for terms such as domain model, anemic domain model, rich domain model, invariant, application service, and domain service.

## Process

### 1. Establish scope before scanning

If the user did not specify a domain/context, ask one question before exploring:

> Which domain or bounded context is in scope?

Do not perform a broad codebase architecture review. Stay inside the specified domain/context and adjacent callers needed to understand behavior.

### 2. Explore the domain model and behavior leakage

Read existing domain documentation first: `CONTEXT.md`, `CONTEXT-MAP.md`, and relevant ADRs if they exist. Use existing domain language in all findings.

Then explore code in the target domain/context. When an Explore subagent is available, use it to map:

- domain entities, value objects, aggregates, repositories, and domain services
- application services, command handlers, controllers, jobs, and listeners that mutate domain state
- validation rules, calculations, state transitions, policies, and duplicated checks outside the domain model
- public setters, exposed mutable collections, parameterless constructors, and state changes spread across callers
- tests that verify behavior through services because the domain model has no behavior to test directly

Use [DETECTION.md](DETECTION.md) to classify each candidate. Do not flag simple data carriers unless business rules or invariants are present.

### 3. Present candidates as an HTML review

Create a self-contained HTML file in the OS temp directory, not in the repository. Resolve the temp directory from `$TMPDIR`, falling back to `/tmp` on Unix-like systems or `%TEMP%` on Windows. Name it `anemic-domain-review-<timestamp>.html`.

Open the file for the user when the environment allows it, then provide the absolute path.

Each candidate card must include:

- **Files** — domain model and behavior locations
- **Anemia signal** — what shows behavior is outside the domain model
- **Leaked behavior** — rules, invariants, calculations, or state transitions currently elsewhere
- **Refactor direction** — what behavior moves into entities, value objects, aggregates, or domain services
- **Benefits** — locality, discoverability, invariant protection, and test surface
- **Before / After diagram** — visual comparison of scattered behavior versus a richer model
- **Recommendation strength** — `Strong`, `Worth exploring`, or `Speculative`

Use [HTML-REPORT.md](HTML-REPORT.md) for the report format. Do not propose final interfaces or edit code yet.

End by asking:

> Which candidate gets explored first?

### 4. Grill the selected candidate

After the user chooses a candidate, interview the user one question at a time, following the `grill-with-docs` style:

- Ask about one unresolved domain decision at a time.
- Provide a recommended answer with each question.
- If the answer can be found in code, inspect the code instead of asking.
- Challenge vague or conflicting domain language immediately.
- Stress-test the model with concrete scenarios and edge cases.
- Update `CONTEXT.md` inline when a domain term, invariant, state transition, or aggregate responsibility becomes clear.

Do not create ADRs for rejected rework. If the user declines a refactor, leave the issue visible so future reviews can surface it again until the underlying model issue is resolved.

### 5. Design the refactor path

Use [REFACTORING.md](REFACTORING.md) to design the smallest safe path:

1. Preserve current behavior with tests.
2. Move one invariant or behavior into the domain model.
3. Narrow constructors, setters, and collection access only as needed.
4. Keep application services thin: load, delegate, persist, publish/trigger.
5. Use domain services only when behavior spans multiple aggregates or needs collaborators and has no natural entity/value-object owner.
6. Move side effects behind domain events or application orchestration; do not put infrastructure calls inside entities.

Ask for explicit confirmation before editing code.

### 6. Implement only after confirmation

When the user confirms implementation, work in small steps:

- add or adjust tests first
- refactor one behavior slice at a time
- run focused tests after each slice
- keep compatibility unless the user approves a breaking domain interface change
- delete obsolete service-level duplicated rules only after domain tests cover them

Use Java/Spring/JPA examples when examples are needed, but keep principles stack-agnostic.
