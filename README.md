# agent-dev-stack

Language-agnostic engineering rules for AI coding agents — and the engineers who work with them.

Rules are plain Markdown, independent of language and framework. They use normative language (`MUST`, `MUST NOT`, `SHOULD`) so an agent can apply them directly, while staying readable for humans. Every rule ends with a `Checklist`; rules that need it also include an `Enforcement` section.

## Quick start

Pick the lightest option that fits your workflow.

**1. Reference rules per task.** Point the agent at the rules that matter for the change:

```text
Follow rules/general/code-style.md and rules/api/errors.md for this change.
```

**2. Import into your agent's instruction file.** Claude Code and compatible tools support `@path` imports. Add the files you need to `CLAUDE.md`, `AGENTS.md`, or an equivalent:

```markdown
## Engineering rules
@rules/general/code-style.md
@rules/general/code-quality.md
@rules/general/naming.md
@rules/general/error-handling.md
@rules/general/logging.md
```

Import only the domains your project needs. Other agents (Cursor, Copilot, …) can reference the same files through their own instruction mechanisms.

**3. Vendor the rules into your repository.** Copy or symlink the `rules/` subdirectories you need into your project and version them with your code. The paths above assume the repository root; adjust them to your layout.

All rules are written in English so agents parse them consistently.

## Rule catalog

| Domain | Rule | Covers |
|---|---|---|
| general | [code-style](rules/general/code-style.md) | Style decisions a formatter cannot make: restructuring, blocks, imports, declarations, literals, comments |
| general | [code-quality](rules/general/code-quality.md) | Change discipline, unit design, control flow, duplication, state, boundaries |
| general | [naming](rules/general/naming.md) | Identifiers, files and modules, public APIs, data and configuration, renames |
| general | [error-handling](rules/general/error-handling.md) | Failure classification, raising and propagation, recovery, cleanup, concurrency, public boundaries |
| general | [logging](rules/general/logging.md) | Levels, structured records, correlation, sensitive data, errors, boundaries, operations |
| api | [design](rules/api/design.md) | Contracts, compatibility, request and response shape, pagination, idempotency, limits, async delivery |
| api | [errors](rules/api/errors.md) | Outcome layers, status selection, error envelopes, public codes, validation, retry and replay |

Planned domains — `security/`, `testing/`, and `database/` exist as placeholders; their rules are not written yet.

## How rules are written

- Each file starts with YAML frontmatter; the schema is defined in [rules/README.md](rules/README.md).
- `MUST` / `MUST NOT` mark requirements; `SHOULD` permits a deviation that must be documented.
- Each rule states its scope up front and defers adjacent topics to the rules listed under `related`.
- `Enforcement`, when present, describes CI and review expectations; every rule closes with a `Checklist` of verifiable acceptance items.

## Repository layout

```text
rules/
├── README.md          # rule metadata schema
├── general/           # code-style, code-quality, naming, error-handling, logging
├── api/               # design, errors
├── security/          # input-validation (planned)
├── testing/           # test-strategy, test-structure (planned)
└── database/          # queries, migrations (planned)
```

## Contributing

- One topic per file, one domain per folder. Rules stay language-agnostic.
- Start from an existing rule, fill in the required frontmatter fields per [rules/README.md](rules/README.md), and set `status: draft` until the rule is reviewed.
- State the rule's scope, cross-link neighboring rules with `related`, and finish with `Enforcement` and `Checklist`.
- Add the new rule to the catalog above.

## License

[Apache-2.0](LICENSE)
