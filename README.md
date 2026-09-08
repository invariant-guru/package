# @invariant-guru/cli

Helps create, structure, and publish Invariant packages — with an always-on instruction, a detailed authoring skill, and a dedicated agent.

## What's inside

| Type        | Name             | Installed as                            | Description |
|-------------|------------------|-----------------------------------------|-------------|
| Instruction | package-creation | folded into each target's context file  | Always-on reference: package structure, the three `invariant.json` modes, item types, installed naming, and the essentials of writing a skill. |
| Skill       | create-package   | `invariant-guru-cli-create-package`     | Full procedure for scaffolding, validating and publishing a package. Bundles `references/skill-authoring.md` (frontmatter limits, progressive disclosure, workflows, host differences, worked example) and `references/package-naming.md`. |
| Agent       | package-creator  | `invariant-guru-cli-package-creator`    | Dedicated agent for a session spent designing and building a package. |

Item names are prefixed with a slug of the package name at install time, so the skill
lands at `.claude/skills/invariant-guru-cli-create-package/SKILL.md` (a symlink into
`.agents/skills/`). Instructions and contexts are not prefixed — they are folded into
each target's context file. See
[`skills/create-package/references/package-naming.md`](skills/create-package/references/package-naming.md).

## Install

```sh
invariant install @invariant-guru/cli
```

## Author

invariant.guru

## License

MIT
