---
id: code-style
title: Code Style
domain: general
description: "Language-agnostic, high-frequency style rules for the decisions a formatter cannot make: restructuring, block form, imports, declarations, literals, and comments."
scope: "All handwritten source code; mechanical formatting delegated to the repository formatter; naming, structure, failures, logging, security, and tests governed by related rules; generated and third-party code excluded."
audience: [ai-agents, engineers]
status: active
version: 1.0.0
updated: 2026-11-25
language: en
tags: [code-style, formatting, readability, code-review]
related: [naming, code-quality, error-handling, logging]
---

# Code Style

Applies to all handwritten source code. Generated, vendored, lock, and minified files are excluded. Mechanical formatting is delegated to the repository's formatter; this document governs only decisions a formatter cannot make. Identifier semantics and file, module, and package names are governed by `naming`; change discipline, structure, nesting, operator precedence, magic values, dead code, state and object mutation, TODO lifecycle, and exception recording by `code-quality`; failure channels and empty failure handlers by `error-handling`; debug output and record shape by `logging`; secrets, source-encoding attacks, and sensitive data by security rules. New and modified code MUST comply; `MUST` / `MUST NOT` are required, and `SHOULD` permits a deviation.

## 1. Style Authority and the Formatter Boundary

| Formatter-owned — do not hand-edit or review as style | Reviewed here — formatter cannot decide |
|---|---|
| Indentation, tabs/spaces, operator spacing, brace placement, quote style, trailing commas, line-break position, trailing whitespace, final newline, import sorting where supported, maximum consecutive blank-line runs | When to restructure instead of accepting the formatter's shape; import intent (explicitness, unused symbols, aliases, local imports); one-line readability; comment truthfulness and doc completeness; suppression scope and reason; consistency with local convention |

- Each file MUST be governed by one checked-in authoritative formatter configuration; nested or monorepo configurations are allowed and the nearest applicable configuration wins. CI MUST run the same pinned formatter version and configuration and MUST verify that committed code is formatter-clean. This document MUST NOT restate a formatter-decidable detail; the formatter's output stands.
- Precedence: nearest committed tool configuration > established repository convention > this document > ecosystem default; conflicts resolve to the tool. Where no formatter exists or no configuration is committed, the project MUST record one formatting convention following the ecosystem's canonical style and MUST NOT hand-format toward a competing convention.
- One style per project or package; new code MUST be indistinguishable from surrounding code. In an inconsistent file, match the dominant pattern; MUST NOT introduce a third.
- Generated, vendored, lock, and minified files are excluded through the checked-in ignore list and MUST NOT be hand-edited; a new project or package MUST add its formatter configuration and CI check in the same change, with formatter version upgrades as separate changes.

## 2. Layout and Restructuring

- The formatter's configured line width is authoritative; MUST NOT hand-break to a different width. Unbreakable tokens such as URLs, hashes, and long strings MAY exceed it.
- MUST NOT fight the formatter with manual breaks or reordering. If formatted output spans too many continuation lines to read, restructure the expression by extracting a predicate, variable, or unit.
- Blank lines SHOULD separate logical groups; the formatter controls consecutive blank-line limits and mechanical placement where supported.
- One statement per line: MUST NOT use separators to stack statements except within a construct that requires them, and MUST NOT hand-align or pad declarations and assignments; where the formatter aligns, its output stands.

## 3. Blocks and Control-Flow Surface

- Where the language makes block delimiters optional, the explicit block form SHOULD be used; a one-line body is allowed only when it is the ecosystem's canonical idiom and stays single-purpose.
- A one-liner MUST NOT hide a side effect, an assignment, or more than one operation; code-golf and comma expressions are forbidden. A ternary MUST NOT encode a multi-branch decision, and nested ternaries SHOULD NOT be used.
- Assignment inside a condition MUST NOT be used unless the ecosystem idiom requires it; when required, isolate it and make intent explicit.
- An empty block MUST be removed unless the language requires a placeholder or its emptiness is intentional; an intentional, non-obvious empty block SHOULD carry a comment. Empty failure handlers are governed by `error-handling`.

## 4. Imports and Namespaces

- Imports SHOULD be explicit; wildcard or star imports, and whole-module imports for a single symbol, SHOULD NOT be used unless the ecosystem or local convention treats them as normal, and then MUST stay in the narrowest module scope.
- Imports MUST be grouped and sorted by the formatter or linter; MUST NOT be hand-ordered into a competing scheme. Unused and duplicate imports MUST be removed in the same change.
- Aliases MUST resolve a real collision or follow a long-standing convention, not merely shorten a name; use an import when it is clearer than a fully qualified name, and qualification MAY remain to resolve a collision.
- A function-local or conditional import MUST state a reason (cycle, optional heavy dependency, test isolation) and SHOULD be avoided.

## 5. Declarations and Expressions

- Declare at the narrowest scope and closest to first use; initialize at declaration where the language permits; MUST NOT reuse one binding for unrelated purposes.
- Prefer a binding that is not reassigned; a mutable binding is justified only when rebinding is required. This concerns bindings only: a non-reassignable binding does not make the referenced object immutable. Object mutation, ownership, and shared state are governed by `code-quality`.
- One declaration per statement; grouping is allowed only for destructuring the language supports idiomatically.
- Follow the ecosystem's inference policy; where both forms are acceptable, inference SHOULD be used when the type is obvious from the right-hand side and the explicit type SHOULD be written when inference hides meaning or forms a public contract. Data flow SHOULD read top-to-bottom, with no forward references beyond the language's normal declaration rules.

## 6. Literals and Strings

- Quote style is formatter-owned; when writing before formatting, choose the form that minimizes escaping and do not alternate styles arbitrarily.
- Where the language provides interpolation or formatting, prefer it; otherwise use the ecosystem's idiomatic builder or join. A short, fixed concatenation MAY use the language's concatenation operator; repeated concatenation in a loop MUST NOT be used to assemble text.
- Use the language's multiline, template, or raw facility where one exists; otherwise use the documented local idiom instead of escaped newline chains.
- Numeric literals SHOULD use digit separators where supported and a base that matches meaning; where a raw-string form exists, use it for regex, byte, and escape-heavy literals to avoid double escaping.

## 7. Comments and Documentation

- Comments MUST explain why, constraints, invariants, trade-offs, or references; they MUST NOT restate what the code does.
- A comment MUST be updated in the same change as the code; a stale or contradictory comment is a defect.
- Public APIs MUST use the ecosystem's documentation-comment format: purpose, parameters, return, failures, side effects, and concurrency guarantees where relevant.
- Comments MUST NOT compensate for unclear names or structure; those are governed by `naming` and `code-quality`.
- A suppression MUST target the narrowest scope, name the specific rule, and carry a reason; blanket or file-wide suppression MUST NOT be used.
- Comments MUST use the repository's chosen language consistently; banner comments SHOULD aid navigation only, and documentation examples SHOULD be runnable or clearly marked illustrative.

## Checklist

- [ ] Formatter configuration is checked in, CI runs the same pinned version, and committed code is formatter-clean
- [ ] Mechanical layout is left to the formatter; logical blank-line grouping and restructuring are reviewed, not hand-broken
- [ ] Block form, one-liners, ternaries, and condition assignments are readable and single-purpose
- [ ] Imports are explicit, grouped, sorted, and free of unused, duplicate, or gratuitous aliases
- [ ] Bindings are narrowly scoped and initialized; binding reassignment is distinguished from object mutation
- [ ] Strings and literals use the ecosystem's interpolation, multiline, raw, and numeric idioms
- [ ] Comments explain why, stay current, document public APIs, and use narrow justified suppressions
