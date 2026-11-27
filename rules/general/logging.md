---
id: logging
title: Logging
domain: general
description: "Language-agnostic production logging rules: levels, structured records, correlation, sensitive data, errors, boundaries, and operations."
scope: "All production code that emits logs; test fixtures excluded."
audience: [ai-agents, engineers]
status: active
version: 1.0.0
updated: 2026-11-24
language: en
tags: [logging, observability]
related: [error-handling]
---

# Logging

Applies to all production code that emits logs: services, libraries, CLIs, and batch or background jobs. Test fixtures are excluded. New and modified code MUST comply; do not refactor unrelated logging. `MUST` / `MUST NOT` are required; `SHOULD` permits a documented deviation.

## Levels
- Use only `DEBUG`, `INFO`, `WARN`, and `ERROR`; do not invent levels.
- `DEBUG` diagnostics (off in production); `INFO` state changes and business events; `WARN` handled or retried anomalies while the operation continues; `ERROR` failures that stop the current operation or require action.
- Default to `INFO` in production.
- Expected conditions — retries, cache misses, validation failures, client errors — MUST NOT be logged as `ERROR`.
- Log each event once, at one level.

## Records
- Emit structured records (JSON or key=value); never interpolated strings.
- Store a stable dotted event identifier, such as `user.login.failed`, in a dedicated field; keep it free of variable data and prose.
- Use consistent `snake_case` field names across services and reuse existing keys.
- Core fields: `timestamp` (UTC, ISO-8601 in text form), `level`, `event`, `service`, `version`, `environment`; add `trace_id` and `span_id` when tracing is in use. The runtime or logging layer MAY inject `timestamp`, `service`, `version`, `environment`, `trace_id`, and `span_id`; code MUST NOT duplicate configured fields.
- One record per line; keep stack traces out of multi-line text fields.
- Escape untrusted values written to unstructured text; otherwise pass them as data.

## Context and correlation
- Propagate `trace_id` and `span_id` across process boundaries and into asynchronous, background, and queued work.
- When tracing is in use, every record emitted while handling a request or task MUST carry its `trace_id`.
- Attach request or operation, tenant, and pseudonymized actor identifiers when they exist.
- Log at process boundaries: operation, outcome, duration with an explicit unit, and dependency errors.

## Sensitive data
- MUST NOT log credentials or secrets: passwords, access tokens, API keys, session credentials, authentication material, or private keys.
- MUST NOT log complete input or output payloads or metadata collections by default; log only allow-listed fields.
- MUST redact personal data — email, phone, government identifiers, address, payment data — before logging.
- MUST remove sensitive values from identifiers, resource locators, and parameters before logging, such as tokens or signatures.
- Exception messages, causes, and stack traces MUST be scrubbed for secrets and personal data before logging.

## Errors
- Log errors once, at the boundary where they are finally handled, with `type`, `message`, `stack`, and cause.
- MUST NOT log and rethrow at every layer.
- MUST NOT swallow exceptions silently.
- Include the operation, key identifiers, and outcome; never log only "failed".
- SHOULD rate-limit or aggregate repeated identical errors.

## Code boundaries
- Libraries MUST NOT configure global logging state; log through a caller-provided or library-scoped logger.
- MUST NOT log from implicitly invoked code such as accessors, string conversion, or constructors.
- MUST NOT log inside tight loops; aggregate or sample high-volume events.
- MUST NOT construct log payloads that level filtering would discard.
- Never put side-effecting calls in log arguments; logging MUST NOT change behavior or return values.

## Operations
- Logging MUST NOT perform synchronous I/O or network calls on a request or task's critical path.
- Field values MUST be bounded: no unbounded collections or raw free-form input; cap free-text length.
- MUST NOT use logs as metrics; high-frequency counts SHOULD go to an aggregation or metrics system.

## Checklist
- [ ] Stable event identifier and structured fields; `trace_id` when tracing is in use
- [ ] No secrets, personal data, complete payloads, sensitive parameters, or unsafe exception text
- [ ] Correct level; error logged once with cause
- [ ] No synchronous I/O, loop, accessor, or string-conversion logging
- [ ] Untrusted input escaped; field values bounded
