# package

Helps create, structure, and publish Invariant packages with instructions, a skill, and a dedicated agent.

## What's inside

| Type        | Name             | Installed as                                | Description |
|-------------|------------------|---------------------------------------------|-------------|
| Instruction | package-creation | folded into each target's context file      | Reference guide for Invariant package structure, manifest format, item types, and installed naming. |
| Skill       | create-package   | `invariant.guru-package-create-package`     | Step-by-step skill to scaffold a new Invariant package from scratch. |
| Agent       | package-creator  | `invariant.guru-package-package-creator`    | Dedicated agent that guides users through creating a complete Invariant package. |

Item names are prefixed with a slug of the package name at install time, so the skill
lands at `.claude/skills/invariant.guru-package-create-package/SKILL.md` (a symlink into
`.agents/skills/`). Instructions and contexts are not prefixed — they are folded into
each target's context file. See `skills/create-package/references/package-naming.md`.

## Install

```sh
invariant install @invariant.guru/package
```

## Author

invariant.guru

## License

MIT
