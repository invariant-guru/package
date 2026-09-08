# Invariant Package Creation — package structure, manifest format, item types, installed naming and skill authoring

Reference for creating Invariant packages and writing the items inside them. For the full procedure — three-mode `invariant.json`, the installed-name map, the complete skill-authoring method with a worked example — invoke the [`invariant-guru-cli-create-package`](.claude/skills/invariant-guru-cli-create-package/SKILL.md) skill.

## This Package Contains

| Item | Type | Installed as | When to use |
|------|------|--------------|-------------|
| `package-creation` | Instruction | folded into this context file | You're reading it. Always-on reference for packages and skills. |
| `create-package` | Skill | [`invariant-guru-cli-create-package`](.claude/skills/invariant-guru-cli-create-package/SKILL.md) | Invoke to build or restructure a package end to end; also carries `references/skill-authoring.md` and `references/package-naming.md`. |
| `package-creator` | Agent | [`invariant-guru-cli-package-creator`](.claude/agents/invariant-guru-cli-package-creator.md) | Activate for a session dedicated to designing and building a package. |

> **Partial installs:** Invariant lets users install only a subset of a package's items. Some items listed above may not be present in your `.claude` folder — that's expected. Use what's available and refer to the descriptions above to understand what each item does.

## Package Structure

A package is a directory with a manifest, a README, and item directories:

```
my-package/
├── invariant.json           # manifest (required)
├── README.md                # human-facing description (required)
├── agents/                  # AI behavior specs
├── skills/                  # repeatable procedures (always folders)
├── commands/                # slash-command definitions
├── rules/                   # constraints and standards
├── contexts/                # background knowledge
└── instructions/            # procedural guides (folded into each target context file)
```

Only include directories for item types the package provides.

## `invariant.json` — one file, three modes

Auto-detected by which keys are present.

**Project mode** (consumes packages) — `targets` chooses which assistants get
projections; `packages` and `active` are CLI-owned:

```json
{
  "name": "my-workspace", "version": "1.0.0",
  "targets": ["claude", "codex"],
  "packages": { "@acme/dev": { "name": "@acme/dev", "version": "1.2.0",
                "source": "github", "sourceUrl": "github:acme/dev" } },
  "active": [{ "package": "@acme/dev", "type": "skills", "item": "release-notes" }]
}
```

**Package mode** (publishable) — the presence of `items` is what marks it publishable:

```json
{
  "name": "@acme/dev", "version": "1.2.0",
  "description": "Release tooling for Acme services.",
  "author": "acme", "license": "MIT",
  "repository": "https://github.com/acme/dev",
  "items": {
    "skills": [{ "name": "release-notes", "description": "Generates a CHANGELOG entry." }]
  }
}
```

**Hybrid mode** — both key sets in one file, for a repository that publishes and consumes.

- `name`: lowercase, hyphens, optional `@scope/`. `version`: semver. Only these two are enforced by the CLI.
- `items` and `active`/`packages` are **generated** by `invariant package create` / `refresh` and `invariant install` / `add`. Never hand-write them. Each `items` entry must match a file or folder on disk.
- `invariant-package.json` is a legacy filename. Leave existing ones alone; add `invariant.json` beside it.

## Installed Names and Paths

Items are **renamed at install time**: the CLI prefixes each one with a slug of the
package name, so two packages shipping a `frontend` skill land in different folders.

```
slug(pkg)    = drop "@", "/"->"-", lowercase, non-[a-z0-9._-]->"-", collapse "-"
qualified()  = item, if item === slug or item starts with slug + "-"
               otherwise slug + "-" + item        # idempotent
```

`docs` + `frontend` → `docs-frontend`. `@invariant-guru/cli` + `create-package` →
`invariant-guru-cli-create-package`.

| Location | Prefixed | Path |
|---|---|---|
| Canonical store | yes | `.agents/<type>/<qualified>` (or `.md` for single files) |
| Claude Code | yes | `.claude/<type>/<qualified>` — symlink into `.agents/` |
| Cursor | yes | `.cursor/rules/<qualified>.mdc` |
| Copilot | yes | `.github/instructions/<qualified>.instructions.md` |
| Contexts, instructions | **no** | `.invariant/{contexts,instructions}/<package>/<name>.md` — already scoped by package, and folded into each target's context file |
| Your repository | **no** | keep authoring `skills/frontend/` — the prefix is an install-time projection |

Nothing rewrites your content, so **every link from one item to another must spell the
prefixed path**. Files *inside* one skill folder are the exception — they travel together,
so `references/guide.md` stays relative. `.agents/` and the target folders are generated
and gitignored, rebuilt by `invariant sync`; never author into them, never commit them.
Unrelated files in `.claude/` or `.github/` are not generated and not disposable.
`invariant inspect <package> --paths` prints the exact installed names;
`invariant package validate` warns (`unprefixed_path_reference`) about links that 404.

## Item Types at a Glance

| Type | Format | Use for |
|---|---|---|
| `agents/` | `.md` file or folder | A behavioural persona for a class of task |
| `skills/` | folder with `SKILL.md` | A procedure run on demand, with inputs and outputs |
| `commands/` | `.md` file or folder | A user-typed `/slash` command |
| `rules/` | `.md` file or folder | Constraints that must always hold |
| `contexts/` | `.md` file or folder | Background knowledge consulted when relevant |
| `instructions/` | `.md` file or folder | Knowledge needed on *every* turn — folded into the context file, so expensive |

Default to the fewest types that do the job. Skills get `## Usage` + ordered
`## Procedure`; agents get `## Role` / `## Approach` / `## Workflow`; commands get
`## Trigger` / `## Parameters` / `## Behavior`; rules and contexts are bullet lists.

## Writing a Skill

Every `SKILL.md` opens with YAML frontmatter on the first line. `name` and `description`
are the only portable fields — they work in Claude Code, claude.ai, the Skills API,
ChatGPT and Codex.

```yaml
---
name: acme-dev-writing-release-notes
description: Generates release notes from the git log between two refs and writes them into CHANGELOG.md. Use when the user asks for release notes, a changelog entry, or a summary of what shipped between two tags.
---
```

- `name`: max 64 characters, lowercase letters/numbers/hyphens only, must not contain `anthropic` or `claude`. Set it to the item's **qualified** name — Claude Code derives the invocation name from the installed directory, not from the frontmatter.
- `description`: non-empty, max 1,024 characters, **third person**, stating what the skill does *and* when to use it, with trigger words front-loaded (some hosts truncate the metadata list).
- Body under 500 lines. Push depth into `references/`, linked **one level deep** from `SKILL.md`; give any reference over 100 lines a table of contents.
- Name the inputs, the exact output paths, the real failure messages and their recovery, and the done condition. `scripts/` is executed; `references/` is read; `assets/` is copied.
- Match specificity to fragility: prose steps for open-ended work, one exact command with no options for anything destructive or order-dependent.
- Include requests that should fire the skill and one nearby request that should not.

Avoid: vague descriptions (`Helps with documents`), first person, offering four library
choices, nested references, Windows paths, and "follow best practices".

**Invariant caveat:** `invariant package refresh` derives each manifest description from
the item's first file — first non-empty line, leading `#` stripped, **frontmatter not
skipped** — so a `SKILL.md` starting with `---` lands in the manifest as
`"description": "---"`. Read `invariant.json` after every refresh and restore the real
description. Keep the frontmatter; the manifest string is only registry copy.

Full method, host-by-host frontmatter differences and a complete worked example:
[`invariant-guru-cli-create-package`](.claude/skills/invariant-guru-cli-create-package/SKILL.md) → `references/skill-authoring.md`.

## Writing Instructions

Instructions are folded into **every** configured target's context file (`CLAUDE.md`,
`AGENTS.md`, `.cursor/rules/invariant-context.mdc`, `.github/copilot-instructions.md`,
`CONVENTIONS.md`) and read on every interaction, so every token is paid for repeatedly.

1. Open with a **"This Package Contains"** table (item, type, installed as, when to use) plus a partial-install note. This is how the AI discovers the package's other items.
2. Self-contained but concise — enough inline detail to handle the simple case alone.
3. Reference every related skill explicitly, **by its installed name and path**. An instruction with no skill reference is a bug; an unprefixed one is a dead link.
4. Example-driven: short fenced blocks, not prose.
5. Action first.
6. End with a `- [ ]` checklist.

```markdown
<!-- package my-pkg -->
YES: [my-pkg-setup](.claude/skills/my-pkg-setup/SKILL.md)
NO:  [setup](.claude/skills/setup/SKILL.md)          <- 404 after install
NO:  [setup](.agents/skills/setup/SKILL.md)          <- same, and .agents/ is generated
```

## README.md

Every package needs one at the root: name, description, an items table with installed
names, `invariant install <name>`, author, license.

## Checklist

- [ ] `README.md` exists at the package root; `invariant.json` has `name` and `version`.
- [ ] `items` was generated by `invariant package create` / `refresh`, not hand-written.
- [ ] Every item in the manifest matches a file or folder on disk.
- [ ] Skills are folders containing `SKILL.md`, with `name` + `description` frontmatter inside the published limits.
- [ ] Each skill's `name` matches its qualified installed name; body under 500 lines; references one level deep.
- [ ] Instructions open with the package-contents table, carry the partial-install note, reference related skills by installed name, and end with a checklist.
- [ ] Installed names computed from the package slug before content was authored.
- [ ] Every cross-item reference spells the prefixed path; intra-skill links stay relative.
- [ ] `invariant.json` checked after `refresh` — no item description reads `---`.
- [ ] `invariant package validate` reports no drift and no `unprefixed_path_reference` warnings.
- [ ] `.agents/` and the target folders were never hand-edited or committed.
- [ ] Publish only when the user explicitly asked for it.
