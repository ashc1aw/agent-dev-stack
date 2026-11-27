# Rule Metadata

Every newly authored or modified rule file under `rules/` MUST begin with YAML frontmatter using this schema.

| Field | Required | Type | Purpose |
|---|---|---|---|
| `id` | yes | kebab-case, equals filename | stable identifier |
| `title` | yes | string | display name |
| `domain` | yes | folder name, e.g. `general` or `security` | grouping |
| `description` | yes | one sentence | retrieval summary |
| `scope` | yes | string | when the rules apply |
| `audience` | yes | list | `ai-agents`, `engineers` |
| `status` | yes | `draft`, `active`, `deprecated` | lifecycle |
| `version` | yes | semver | change tracking |
| `updated` | yes | ISO 8601 date | freshness |
| `language` | yes | BCP 47 | locale |
| `tags` | no | list | search |
| `related` | no | list of rule `id`s | cross-links |
| `owner` | no | string | accountability |

Example: `general/logging.md`.
