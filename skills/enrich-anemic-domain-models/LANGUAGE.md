# Language

Shared vocabulary for this skill. Use these terms exactly in findings, questions, reports, and refactor plans.

## Terms

**Domain/context**
The business area or bounded context the user asked to review. The skill only scans and reports inside this scope, plus adjacent callers needed to understand behavior.

**Domain model**
Code that represents business concepts, state, relationships, rules, and behavior. In Java projects this often includes entities, value objects, aggregates, domain events, and domain services.

**Entity**
A domain object with identity and lifecycle. Entity behavior protects lifecycle rules and state transitions.

**Value object**
A domain object defined by its values rather than identity. Value objects are good homes for validation, calculations, and comparison rules that belong to a concept.

**Aggregate**
A consistency boundary around one or more entities and value objects. Invariants that must be protected together belong inside the aggregate.

**Invariant**
A rule that must stay true for the domain model to be valid. Invariants are enforced through constructors, factories, and behavior methods, not scattered caller checks.

**Anemic domain model**
A domain model with business state but little or no business behavior. Common signals include public setters, exposed mutable collections, parameterless construction, and service methods that inspect and mutate domain state directly.

**Rich domain model**
A domain model that exposes behavior in the domain language and protects invariants near the state it governs. Rich does not mean large. It means behavior and rules have a clear owner.

**Behavior leakage**
Business rules, validations, calculations, state transitions, or invariant checks living outside the domain model that owns the state.

**Application service**
Thin orchestration. It loads domain objects, calls domain behavior, persists changes, and coordinates side effects. It does not own domain rules.

**Domain service**
Domain behavior that has no natural single entity or value-object owner, often because it spans multiple aggregates or needs a domain collaborator. A domain service is still part of the domain model.

**Domain event**
A fact that something meaningful happened in the domain. Use domain events to express side effects without putting infrastructure inside entities.

**Factory**
A domain creation method or object that creates valid domain objects. Use it when constructors would expose too much setup detail or when creation has domain rules.

## Principles

- **Focus scope first.** If the user did not name a domain/context, ask before scanning.
- **Domain model only.** Do not flag DTOs, request/response models, persistence projections, generated code, or simple CRUD records.
- **Behavior belongs near state.** Put rules where the state and language make the rule discoverable.
- **Invariants are not suggestions.** A valid domain object must not depend on every caller remembering the same checks.
- **Application services stay thin.** They coordinate work; they do not become the domain model.
- **Domain services are valid.** Do not force behavior into an entity when the rule spans multiple aggregates or needs collaborators and has no natural owner.
- **Rich is not infrastructure-aware.** Entities and value objects do not call repositories, email senders, queues, HTTP clients, or clocks directly unless the project has an established domain abstraction for that collaborator.
- **One refactor at a time.** Move one behavior slice, verify, then continue.

## Rejected framings

- **"All services are bad."** That is incorrect. Application services and domain services have valid roles.
- **"Every class with getters is anemic."** That is incorrect. Data carriers are valid outside the domain model, and simple domain records can be enough for CRUD.
- **"Rich model means no JPA annotations."** That is a separate persistence design question. The skill focuses on behavior and invariants.
- **"Move all logic into entities."** That is too broad. Use entities, value objects, aggregates, domain services, factories, and domain events based on ownership.
