---
id: code-quality
title: Code Quality
domain: general
description: "Language-agnostic structural quality rules for change discipline, unit design, control flow, duplication, state, and boundaries."
scope: "All handwritten production source code in services, libraries, CLIs, and batch or background jobs; generated and third-party code excluded; naming, formatting, failure handling, and logging are governed by related rules; tests, edge-case correctness, and performance are out of scope."
audience: [ai-agents, engineers]
status: active
version: 1.0.0
updated: 2026-11-25
language: en
tags: [code-quality, maintainability, architecture, code-review]
related: [naming, code-style, error-handling, logging]
---

# Code Quality

Applies to all handwritten production source code in services, libraries, CLIs, and batch or background jobs. Generated code, third-party code, and mechanical formatting are excluded. Naming, formatting, failure handling, and logging are governed by the related rules; tests, edge-case correctness, and performance are out of scope. New and modified code MUST comply; do not restructure unrelated code solely to satisfy this rule. `MUST` / `MUST NOT` are required; `SHOULD` permits a documented deviation.

A unit is the smallest independently named construct; a module is a file, package, or service boundary; a consumer is code that uses a unit or contract. I/O is interaction with a resource outside the unit's own state. Observable state and observable behavior are what a consumer can observe through results, effects, and documented guarantees; internal caching and lazy initialization are not observable. A reference is borrowed for the duration of a call unless ownership is explicitly transferred; ownership means responsibility for a value's lifetime and mutation. The composition root is the entrypoint or boundary at which concrete dependencies and configuration are assembled for a process or invocation; a codebase MAY have one per entrypoint. Process-wide state has a lifetime beyond a single invocation and is shared across units.

## 1. Change Discipline

- A change MUST have one primary purpose and MUST NOT include unrelated work; unrelated reformatting, renaming, restructuring, and dependency upgrades MUST be separate changes.
- A behavior change and a broad refactoring MUST NOT be combined. Restructuring required by the change MAY be included only when it is minimal, confined to the touched code, and identified in the change; otherwise it MUST be a separate behavior-preserving change.
- New code MUST follow the structure and patterns already used in the module. Introducing a second pattern for the same problem MUST migrate existing occurrences in the same change, or record an exception with an owner and a completion condition.
- A refactoring MUST preserve observable behavior. When behavior changes, every affected contract and consumer in the repository MUST be updated in the same change; external consumers MUST follow the versioning or deprecation decision.
- Dead code MUST be deleted: unused units, parameters, branches, exports, switches, and commented-out code MUST NOT remain. Version control is the archive; alias lifecycle is governed by naming.
- A TODO or FIXME MUST carry an owner and a removal condition. A TODO that records a requirement MUST be implemented or rejected; an untracked TODO MUST NOT be added.
- A temporary feature flag or conditional path MUST carry an owner and a removal condition; when the condition is met, it MUST be removed in the change that resolves it.
- A new dependency MUST be justified against the standard library and existing dependencies, and MUST have a current consumer. A dependency for a capability the platform already provides MUST NOT be added.

## 2. Unit Design

- A unit MUST serve one change trigger; unrelated actions MUST be split. Length alone MUST NOT be reported as a defect.
- A unit MUST NOT mix policy with mechanism: a domain or orchestration decision MUST NOT perform I/O directly; I/O MUST be encapsulated in an explicit dependency. Diagnostics governed by logging, trivial accessors, and one-line delegations are exempt.
- A unit SHOULD accept at most four parameters; beyond that, related inputs MUST be grouped into one typed input.
- A parameter MUST NOT select between behaviors. A behavior variant MUST be a separate unit or an explicit type.
- Arguments are borrowed by default. A unit MUST NOT mutate an argument unless in-place mutation is the unit's stated purpose and its name states it; otherwise results MUST be returned. Output parameters MUST NOT be used where the language provides a return channel.
- A unit MUST have one contract: it returns a result without changing observable state, or it changes observable state and any returned value describes the effect. Transparent caching, lazy initialization, and diagnostics or metrics are exempt.
- A unit that computes a domain result or transforms data MUST NOT depend on undeclared ambient state, including current time, randomness, locale, and process-wide mutable state. Such dependencies MUST be explicit inputs and MUST be resolved by the caller or at the composition root.

## 3. Control Flow

- A condition MUST NOT depend on operator precedence for its meaning. A condition with a domain meaning MUST be a named predicate; a condition spanning more than three propositions SHOULD be extracted, as a review trigger rather than a defect.
- A unit that nests more than three levels SHOULD be flattened with guard clauses, early exits, or extracted units; nesting depth is a review signal, not a defect by itself.
- A boolean expression MUST NOT use double negation; negation SHOULD apply to a named positive predicate rather than to a compound expression.
- A decision that guards the same outcome MUST NOT be duplicated across branches; it MUST be hoisted or the branches merged.
- Dispatch over a variant set known to be closed MUST handle every variant and MUST NOT use a default to hide a missing case. When values can arrive from outside the codebase, an unknown value MUST be rejected, preserved, or mapped to a documented fallback; it MUST NOT be silently ignored.
- A loop MUST have one purpose. It MUST NOT mutate the collection it iterates unless the iteration contract supports it, and it MUST NOT both compute a result and perform an unrelated external effect.

## 4. Duplication and Abstraction

- DRY applies to knowledge, not text: a business rule, validation, limit, schema, or protocol value MUST have one authoritative definition. Textual similarity alone MUST NOT trigger generalization.
- Extracting a named step for a single caller is decomposition, not abstraction, and is permitted. Identical code with two or more current consumers SHOULD be consolidated at the narrowest shared scope; code MUST NOT be placed in a shared layer before a second consumer exists.
- A generalized abstraction MUST NOT be introduced until the variation it serves is known; it SHOULD wait for three current consumers, or two with documented divergent variation. A shared unit that needs behavior switches, optional parameters, or consumer-specific branches MUST be split back into consumer-specific units.
- A value that encodes policy, a limit, a protocol constant, or configuration MUST NOT be duplicated across units; it MUST be defined once and referenced.
- When a defect is fixed in one copy of duplicated logic, every copy MUST be fixed or the copies consolidated in the same change.

## 5. State and Mutation

- Mutable state MUST be owned by the narrowest scope that needs it. Long-lived shared state MUST have a single owner with an explicit lifetime.
- Each piece of process-wide state MUST have a single explicit owner and MUST NOT be reachable as mutable global state. It SHOULD be immutable after initialization; where mutation is required, only its owner MAY mutate it through its interface. State owned and mutated by the runtime or host is exempt.
- A unit MUST NOT retain a reference to borrowed data beyond the call, and MUST NOT expose internal mutable state it continues to own; exposure MUST be an immutable view, a copy, or an explicit ownership transfer.
- Temporal coupling MUST be enforced by the API, not by documentation or convention: an order that would be wrong MUST be unrepresentable or MUST be enforced by a scoped construct.
- Domain logic MUST NOT read global registries, service locators, or hidden singletons; dependencies MUST be explicit inputs.
- A type that maintains invariants MUST declare them and enforce them at creation and at every mutation; it MUST NOT be constructible in an invalid state.
- A cache or memoized value MUST have a defined scope and an owner responsible for invalidation. An unbounded, process-lifetime mutable cache MUST NOT be introduced.

## 6. Boundaries and Separation

- Domain logic MUST NOT depend on transport, storage, UI, framework, or vendor types; dependencies MUST point from edges toward the domain.
- I/O MUST be confined to edges, adapters, and startup or composition code; domain and transformation units MUST be computable without network, filesystem, or database.
- Persistence models, wire formats, and third-party types MUST NOT cross a public contract; they MUST be mapped at the boundary.
- Configuration MUST enter through an explicit interface. Static values MUST be read and validated once at startup and passed as typed values; dynamic values MUST be read through the interface with defined refresh and validation. Logic MUST NOT read ambient configuration.
- Module and layer dependencies MUST be acyclic; a cycle that cannot be removed MUST be confined to one module or documented with a removal condition.
- A module MUST state one responsibility and MUST NOT contain subsets that change for unrelated reasons.
- Public surface MUST be minimal: a declaration not required by a current consumer or by an externally discovered contract MUST NOT be public.
- Cross-cutting behavior MUST be applied at the boundary or by a wrapper, not repeated inside domain units.

## Enforcement

- Mechanical checks — complexity thresholds, unused declarations, dependency cycles, and public surface — SHOULD be delegated to linters, static analysis, or CI.
- Review MUST focus on change purpose, cohesion, abstraction boundaries, state ownership, and temporal coupling.
- Every `SHOULD` deviation MUST be documented at the definition.
- A recorded exception MUST carry an owner and a removal condition.

## Checklist

- [ ] Change has one primary purpose; broad refactoring is separate; necessary restructuring is minimal and identified; dead code, TODO, and temporary flags carry owners
- [ ] Units serve one change trigger; decisions do not perform I/O directly; inputs are explicit; no behavior-selecting parameters; borrowed arguments are not mutated
- [ ] Conditions with domain meaning are named; nesting is a review signal; decisions are not duplicated; dispatch handles every variant and unknown external values explicitly
- [ ] Knowledge has one authoritative definition; decomposition, consolidation, and generalization are distinguished; no consumer-specific switches
- [ ] State is narrowly owned; borrowed data is not retained; exposure is immutable or transferred; temporal coupling and invariants are enforced
- [ ] Domain is independent of I/O, frameworks, storage, and vendor types; configuration enters explicitly; dependencies are acyclic; public surface is minimal
