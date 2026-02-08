# Invariant Package Creation Guide

When creating or modifying an Invariant package, follow the structure and rules below.

## Package Manifest

Every package requires an `invariant-package.json` at its root with these fields:

| Field         | Type   | Description                                                        |
|---------------|--------|--------------------------------------------------------------------|
| `name`        | string | Unique lowercase name, may contain hyphens.                        |
| `version`     | string | Semantic version (`major.minor.patch`).                            |
| `description` | string | Short human-readable summary shown in search results.              |
| `author`      | string | Author name or handle.                                             |
| `license`     | string | License identifier (e.g. `MIT`, `Apache-2.0`).                     |
| `repository`  | string | URL to the source repository.                                      |
| `items`       | object | Maps item type directories to arrays of `{ name, description }`.   |

## Item Types

There are six item types. Each lives in its own directory:

- **agents/** -- High-level AI behavior specifications (role, approach, workflows).
- **skills/** -- Repeatable procedures invoked on demand. Always a folder with a `SKILL.md` entry point.
- **commands/** -- Custom slash-command definitions with trigger, parameters, and behavior.
- **rules/** -- Constraints and guidelines the AI must always follow.
- **contexts/** -- Background knowledge (architecture, tech stack, domain terms).
- **instructions/** -- Procedural step-by-step guides for specific situations.

## Item Formats

- All types except skills can be a single `.md` file or a folder of `.md` files.
- Skills must always be a folder containing at least `SKILL.md`.
- The `name` in the manifest matches the filename (without extension) or folder name.

## README

Every package must include a `README.md` at its root. The README is the human-facing description of the package and should contain:

- The package name as a heading.
- A short description of what the package provides.
- A table or list of included items with their types and descriptions.
- Install instructions (`invariant install <name>`).
- Author and license information.

## Directory Layout

```
my-package/
├── invariant-package.json
├── README.md
├── agents/
│   ├── my-agent.md
│   └── complex-agent/
│       ├── overview.md
│       └── checklist.md
├── skills/
│   └── my-skill/
│       ├── SKILL.md
│       └── templates/
├── commands/
│   └── my-command.md
├── rules/
│   └── my-rule.md
├── contexts/
│   └── overview.md
└── instructions/
    └── setup.md
```

Only include directories for the item types your package provides.

## Validation Rules

- A `README.md` must exist at the package root.
- Every item declared in `invariant-package.json` must have a matching file or folder.
- Skills must be folders with a `SKILL.md` inside.
- Package names must be lowercase and may only contain hyphens.
- Version must follow semantic versioning.
