---
id: error-handling
title: Error Handling
domain: general
description: "Language-agnostic, high-frequency rules for failure classification, raising and propagating errors, recovery, cleanup, concurrency, and public error boundaries."
scope: "All production code that raises, propagates, retries, or reports failures; generated and third-party code excluded."
audience: [ai-agents, engineers]
status: active
version: 1.0.0
updated: 2026-11-25
language: en
tags: [error-handling, reliability, api-design]
related: [logging, naming, code-style, code-quality]
---

# Error Handling

Applies to all production code that raises, propagates, retries, or reports failures: services, libraries, CLIs, and batch or background jobs. Generated and third-party code are excluded; tests are in scope only where this document requires failure-path coverage. New and modified code MUST comply; do not restructure unrelated code solely to satisfy this rule. `MUST` / `MUST NOT` are required; `SHOULD` permits a documented deviation.

## Failure Model
- Every failure MUST be described on three orthogonal dimensions, with mutually exclusive values within each: contract (`expected` / `unexpected`), attribution (`caller` / `system` / `dependency`), and recoverability (`transient` / `deterministic`). A single label MUST NOT replace the three dimensions.
- The dimensions drive separate decisions: contract decides who handles the failure, attribution decides its external protocol class, and recoverability decides whether it may be retried. MUST NOT infer one dimension from another — dependency failures are not necessarily transient, and expected failures are not necessarily recoverable.
- Expected failures are part of the contract and MUST be handled by the caller or converted to a terminal outcome at the boundary. Unexpected failures are programming errors or undocumented failures; business logic MUST NOT catch and continue, and the process or request boundary MUST fail fast or produce a terminal outcome.
- Programming errors MUST be marked unexpected and MUST NOT be retryable.
- Absence that is valid in the domain MUST be expressed as an absence value. When the contract requires a value and it is missing, that is a failure — not-found is in this class — and the failure MUST state what is missing.
- Failures MUST NOT substitute for ordinary branching that the domain models as values, and MUST NOT fake success: null, empty, zero, or sentinel values MUST NOT represent failure when the caller cannot distinguish them. A sentinel that is a language idiom MUST be documented and unambiguous.
- One failure MUST map to one failure value; changing any dimension's marking across a boundary MUST accompany a translation and be documented.

## Contracts
- Public functions and APIs MUST declare the failures they can produce: the three-dimension marking, the concrete type or stable code, retryability, and whether unknown failures may escape.
- The set of recoverable failures is part of the contract. Adding one to a closed contract is a breaking change; an open contract MUST allow unknown failures and MUST document additions.
- Every recoverable failure MUST carry a type or stable code that callers can distinguish. Returning the language's general error type is permitted only when the value itself carries a specific type or code.
- Error codes have two layers. Internal codes identify precise failures for diagnostics, tests, and routing and MAY change at implementation granularity; public codes are contract, MUST be stable and catalogued, MUST remain forward compatible, and MUST be the only codes consumers branch on.
- Public codes MUST NOT expose internal detail such as library names, queries, or internal paths. Internal codes MUST NOT cross a public boundary; the internal-to-public mapping MUST happen once, at the boundary.
- Codes MUST live in a dedicated machine-readable field. Branching on message text is forbidden, and an unknown code MUST degrade safely at the boundary.
- An error crossing a process boundary MUST be serializable: public code, safe message, and bounded structured context.

## Raising
- Input and preconditions MUST be validated at the boundary, and a violation MUST fail before observable state changes.
- The message MUST name the operation, safe identifiers, and the violated expectation, and MUST NOT be only "failed", "error", or an equivalent.
- The failure MUST carry its stable code and structured context. Secrets, personal data, and unbounded input MUST NOT be interpolated into messages.
- Messages MUST NOT carry control decisions.
- The message SHOULD state what the caller can do next — retry, correct input, contact support — and MUST NOT assert an unverified root cause.
- Validation MUST report every field error together with its field path, code, and safe message, except when continuing would be unsafe or would leak information; in that case it MUST fail immediately.
- Translation MUST preserve the original failure as a cause or equivalent context, using the language's idiom.

## Propagation and Translation
- Failures MUST NOT be swallowed: an empty catch, a catch that ignores, and a catch that records and continues are all forbidden.
- Only failures that can be handled, translated, or given context MUST be caught; everything else MUST propagate unchanged.
- Translation MUST happen once, at the boundary that adds meaning, and MUST preserve the original failure. Wrapping at every layer is forbidden.
- A failure MUST NOT be replaced by a new failure that drops the original.
- Broad catching is permitted only at a top-level boundary or where an open third-party error set must be translated; it MUST rethrow programming errors and cancellation, and MUST produce a terminal outcome. Catching and rethrowing unchanged, or catching with neither recovery nor translation, is forbidden.
- An API MUST use one channel for recoverable failures and document it; a recoverable failure MUST NOT be returned in one place and thrown in another.
- An ignored failure MUST be explicit and justified. A failure MUST NOT be discarded to silence a warning or a check.

## Recovery
- Only an operation that is transient and idempotent MAY be retried; a non-idempotent write MUST have an idempotency key before it is retried.
- Deterministic failures MUST NOT be retried: invalid credentials, insufficient permission, failed validation, and unsupported operations will fail again on the same input.
- Credential or permission rejection is deterministic and caller-attributed: the same credentials MUST NOT be retried. Authentication or authorization dependency unavailability is transient and dependency-attributed and MAY follow the normal transient policy, including a single refresh where the protocol defines one.
- Retries MUST be bounded, MUST use backoff with jitter, and MUST share an overall deadline. Unbounded retry and retry that amplifies a dependency outage are forbidden.
- Every I/O operation that can block or cross a network MUST have a timeout; a timeout MUST cancel the underlying work and propagate the deadline. A timeout is a failure; cancellation is a control signal and MUST NOT be reported as a failure.
- Fail-fast versus degradation MUST be an explicit, documented decision per failure; degradation MUST be observable and MUST NOT be silent.
- A batch or partial failure MUST report per-item outcomes and declare all-or-nothing or best-effort; recovery MUST NOT conceal partially applied state or data loss.

## Resources and Cleanup
- Resources MUST be released deterministically on every path; correctness MUST NOT depend on garbage collection or finalizers.
- Release MUST follow acquisition in reverse order, and cleanup MUST cover already-acquired resources when acquisition fails partway.
- A cleanup failure MUST NOT mask the primary failure; when a primary failure exists, the cleanup failure is attached as a secondary failure and reported to diagnostics.
- When no primary failure exists and the cleanup failure means the operation's effect may be incomplete — an unflushed write, a failed commit, an unclosed writable stream, a failed persistence step, an unreleased lock — it MUST propagate as the operation's failure and MUST NOT be discarded.
- A cleanup failure that cannot affect the outcome and has no available action MAY be suppressed, but it MUST remain observable.
- Cleanup MUST be bounded and MUST NOT retry indefinitely or block shutdown.
- A failed operation MUST NOT invalidate a shared resource such as a pool, connection, handle, or lock.

## Async and Concurrency
- Failures from background and fire-and-forget work MUST NOT be lost; every task MUST be awaited, joined, or given a failure handler.
- An unobserved asynchronous failure MUST surface at a boundary that can report it.
- Aggregating concurrent results MUST NOT silently discard failures.
- Shared failure state MUST be thread-safe, and failures MUST NOT be used for synchronization.
- In all-or-nothing work, one failure MUST cancel sibling tasks or MUST be reported together with their results.

## Boundaries
- Internal diagnostics and user messages MUST be separate; stack traces, queries, internal paths, hostnames, secrets, and internal codes MUST NOT be exposed.
- Every external failure MUST map to exactly one code in the catalogued public set. Adding a public code is a contract change that follows the deprecation rule, and public codes MUST NOT be invented at call sites.
- An unknown or unmapped internal failure MUST map to the single fallback public code in the catalogue, accompanied by a correlation id, with the internal code recorded in diagnostics. Exposing the internal code and inventing a per-site generic error are both forbidden.
- The fallback MUST NOT conceal a failure that has a specific public code, and MUST NOT absorb a distinguishable caller-caused failure.
- A public code's message MUST be safe and localizable, and caller-caused and system-caused failures MUST be distinguishable at the protocol level.
- An API MUST report failures through the protocol's failure channel — status, result field, or exit status — MUST NOT return success carrying a failure payload, and MUST NOT collapse distinct failures into one code except the fallback.
- Every reported failure SHOULD carry a correlation id that links it to logs and traces, following `logging`.

## Enforcement
- Mechanical checks — empty catch, broad catch, ignored results, missing timeout — SHOULD be delegated to linters or static analysis; review focuses on the three-dimension marking, translation points, and recovery policy.
- Every recoverable failure in a public contract MUST have at least one test that injects it.
- Public codes MUST be registered in the public catalogue; internal codes MAY be owned by their module; the two namespaces MUST NOT be mixed.
- Every `SHOULD` deviation MUST be documented at the definition.

## Checklist
- [ ] Every failure is marked on contract, attribution, and recoverability; no single-label classification
- [ ] Public contracts declare failure modes; callers can distinguish a concrete type or public code
- [ ] Internal and public codes are separated; every external failure maps to one catalogued code
- [ ] Unknown internal failures map to the single fallback code with a correlation id
- [ ] Messages name the operation and safe identifiers, carry no secrets, and validation reports every field unless unsafe
- [ ] No swallowed failure and no lost cause; translation happens once with the original preserved
- [ ] Retry is limited to transient idempotent work; deterministic credential and permission rejections are not retried
- [ ] Timeouts cancel underlying work; cancellation propagates as a signal, not a failure; degradation is observable
- [ ] Resources are released on every path; cleanup never masks the primary failure and propagates when the effect is incomplete
- [ ] Asynchronous failures are observed; no internal detail leaks; failure paths are tested and codes registered
