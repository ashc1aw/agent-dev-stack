---
id: naming
title: Naming
domain: general
description: "Language-agnostic, high-frequency naming rules for identifiers, files and modules, public APIs, persistence, configuration, and tests."
scope: "All handwritten source code, public interfaces, schemas, configuration, and tests; generated and third-party code excluded."
audience: [ai-agents, engineers]
status: active
version: 1.0.0
updated: 2026-11-25
language: en
tags: [naming, readability, api-design]
related: [code-style, code-quality, error-handling, logging]
---

# Naming

Applies to all handwritten source code, public interfaces, schemas, configuration, and tests. Generated and third-party code are excluded. New and modified code MUST comply; do not rename unrelated code solely to satisfy this rule. `MUST` / `MUST NOT` are required; `SHOULD` permits a documented deviation.

## Principles
- Names MUST state what a value is or does, not how it is implemented.
- Use the project's domain vocabulary: one term per concept and one concept per term; MUST NOT use synonyms for the same concept in code.
- Follow language idiom first, then existing repository convention; MUST NOT introduce a second naming style within the same layer.
- MUST NOT encode type, scope, lifetime, or storage in a name, and MUST NOT imply a type, unit, or behavior the value does not have; markers required by language idiom are exempt.
- Name length SHOULD match scope: exported or public names MUST be fully descriptive; local names may be short but MUST remain meaningful.
- Names MUST use ASCII and MUST NOT differ only by case or spelling; they SHOULD be pronounceable and searchable. Placeholder names (`data`, `info`, `item`, `value`, `tmp`, `foo`, `thing`) are permitted only in a scope of a few lines where context makes the meaning unambiguous; exported declarations, fields, persisted data, and names used across functions MUST NOT use placeholders. When the domain term itself is `data`, qualify it (`requestPayload`, `rawResponse`).

## Identifiers
- Variables and fields MUST be noun phrases; collections plural, scalars singular; MUST NOT add a suffix that repeats the declared type (`userList`, `nameString`).
- Booleans SHOULD read as positive assertions (`is`, `has`, `can`, `should`, `enabled`). A negative form is acceptable when the negative state is the domain term (`disabled`, `isOffline`), but the same concept MUST NOT expose both positive and negative names, and names MUST NOT use double negation (`isNotDisabled`) or leave the true/false meaning ambiguous.
- Functions and methods MUST be verb phrases; predicates use `is`/`has`/`can`; queries and commands MUST be distinguishable. A function that changes observable state or performs I/O MUST NOT use a query-style name (`get`, `find`, `calculate`) and MUST use a verb that names the effect (`create`, `update`, `send`, `delete`). Transparent internal optimizations such as caching or lazy initialization are exempt, and names need not enumerate every side effect.
- MUST NOT use `handle`, `process`, `do`, or `run` as the only semantic content of a name unless the word is a domain term.
- Constants MUST be named for meaning, not for their value; MUST NOT use magic numbers or magic strings; constants with units MUST include the unit.
- Quantities with an ambiguous unit MUST include the unit (`timeout_ms`, `size_bytes`, `price_cents`); use one unit spelling throughout a repository.
- Types, classes, and interfaces MUST be noun phrases; exception types end with `Error` or `Exception` per language idiom. `Manager`, `Helper`, `Util`, `Processor`, `Data`, `Impl`, and `Base` are not forbidden words; they are defects only when they hide the absence of a specific responsibility. A type name MUST distinguish the type from its neighbors; MUST NOT use a generic suffix in place of a specific role (`PaymentProcessor` can be a domain role; `DataProcessor` for arbitrary processing is empty naming).
- Abbreviations MUST be widely understood or listed in the project glossary, and MUST use one casing; MUST NOT shadow language built-ins, standard-library names, reserved words, or types in the same domain.
- MUST NOT use `old`, `new`, `v2`, or numeric suffixes to distinguish a revised concept from its predecessor; rename the concept or put the version in a dedicated field or path. Exceptions: ordered data (`line1`, `line2`), migration code that names both schemas in the same change, and API version identifiers.

## Files, Modules, and Packages
- A file name MUST match its primary declaration or concept; a file MUST contain one primary concept.
- File and directory names MUST follow the ecosystem's casing convention and remain ASCII without spaces.
- Package, module, and directory names MUST be lowercase and domain-meaningful. `utils`, `common`, `misc`, and `helpers` are not forbidden words; an unbounded container is the defect.
- Such a module MUST document its single responsibility and what may be added in the file or directory description; when content spans domains, it MUST be split by domain and renamed for purpose. The test is whether the name plus description lets a reader decide what belongs; new modules SHOULD prefer purpose-specific names.
- Test files MUST be named after the unit under test and follow the ecosystem's placement convention.

## Public APIs and Data Contracts
- A public name is a contract: renaming it is a breaking change; the old name MUST be deprecated with a migration note before removal.
- One API MUST use one field casing; MUST NOT mix snake_case and camelCase.
- Field names MUST express meaning, and units MUST be explicit where they apply; ambiguous `date`, `time`, and `value` SHOULD be replaced with names such as `created_at`, `expires_at`, and `duration_ms`.
- Boolean fields SHOULD be positive; a negative domain state (`disabled`, `isOffline`) is acceptable but the same concept MUST NOT expose both forms; enum values MUST be stable, readable strings without undocumented abbreviations.
- MUST NOT expose internal implementation names (database columns, internal classes) as public contracts; map them at the boundary. Put version identifiers in dedicated fields or paths, not inside business names.

## Data and Configuration
- When the system includes a database, the schema MUST follow one documented convention across the repository; an existing convention wins and MUST NOT be rewritten solely to match this document. Only when no convention exists SHOULD defaults apply: plural table names, singular column names, snake_case, foreign keys as `<referenced_entity>_id`, and one documented prefix scheme for indexes and constraints (`idx_`, `uq_`, `fk_`, `ck_`).
- Timestamps SHOULD use the paired `created_at` / `updated_at`; boolean columns SHOULD use `is_` / `has_` prefixes unless the existing schema uses another consistent pattern.
- Environment variables for an application or service MUST use `SCREAMING_SNAKE_CASE`, a service namespace prefix, and a unit suffix when the value has a unit; configuration keys MUST reuse code terminology; renames MUST update examples, documentation, and deployment manifests.

## Renames and Legacy
- When a name is wrong or its meaning has changed, it MUST be renamed in the same change that touches the code, and every reference in code, tests, documentation, configuration, and schema MUST be updated.
- A public name MUST be deprecated (annotation or documentation plus migration note) before removal and MUST wait for one release according to the project's versioning policy.
- An external or legacy name that cannot change MUST be isolated behind an adapter; use an accurate domain name internally and document the exception at the boundary.
- MUST NOT keep dead aliases; a compatibility alias MUST have an explicit removal condition and deadline.

## Enforcement
- Mechanical checks such as casing, reserved words, and length SHOULD be delegated to linters and formatters; human or agent review focuses on semantic quality.
- A new domain term MUST enter the project glossary before it is used in code.
- Every `SHOULD` deviation MUST be documented at the definition, not only in review discussion.

## Checklist
- [ ] Names reveal intent and match domain vocabulary; one term per concept
- [ ] No type or scope encoding; placeholders absent from exports, fields, and persisted data
- [ ] Generic suffixes carry a specific responsibility; `old`/`new`/versioned names only where allowed
- [ ] Booleans positive or a documented domain-negative state, with no double negation; functions are verbs and no query name hides a side effect; collections plural; units explicit
- [ ] File and module names match the primary concept; `utils`/`common` have a documented boundary or have been split
- [ ] Public API, schema, and configuration names are consistent; renames are complete or deprecated
- [ ] Language idiom and repository convention followed; exceptions documented at the definition
