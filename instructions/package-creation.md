# Invariant Package Creation

When you need to create an Invariant package, use this instruction as your reference. To scaffold a package interactively with prompts and file generation, invoke the **`create-package`** skill.

## Package Structure

A package is a directory with a manifest, a README, and item directories:

```
my-package/
├── invariant-package.json   # manifest (required)
├── README.md                # human-facing description (required)
├── agents/                  # AI behavior specs
├── skills/                  # repeatable procedures (always folders)
├── commands/                # slash-command definitions
├── rules/                   # constraints and standards
├── contexts/                # background knowledge
└── instructions/            # procedural guides (embedded in AI context)
```

Only include directories for item types the package provides.

## Manifest

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "Short summary shown in search results.",
  "author": "my-handle",
  "license": "MIT",
  "repository": "https://github.com/my-handle/my-package",
  "items": {
    "rules": [{ "name": "naming", "description": "Enforces naming conventions." }],
    "skills": [{ "name": "setup", "description": "Scaffolds the project." }],
    "instructions": [{ "name": "guide", "description": "How to use this package." }]
  }
}
```

- `name`: lowercase, hyphens allowed, scoped (`@author/name`) if needed.
- `version`: semver (`major.minor.patch`).
- `items`: maps type directories to `{ name, description }` arrays. Each entry must match a file or folder on disk.

## Item Types at a Glance

| Type           | Format                     | Purpose                                           |
|----------------|----------------------------|---------------------------------------------------|
| `agents/`      | `.md` file or folder       | Shape AI behavior for a class of tasks            |
| `skills/`      | folder with `SKILL.md`     | Repeatable on-demand procedures                   |
| `commands/`    | `.md` file or folder       | Custom `/slash` commands                          |
| `rules/`       | `.md` file or folder       | Constraints the AI must always follow             |
| `contexts/`    | `.md` file or folder       | Background knowledge for the AI                   |
| `instructions/`| `.md` file or folder       | Step-by-step guides embedded in AI context        |

All types except skills can be a single `.md` file or a folder of `.md` files. Skills must always be a folder containing at least `SKILL.md`.

## Writing Each Item Type

### Agents

Define role, approach, and workflow. Example:

```markdown
# Agent: Reviewer
You review PRs for architectural violations.
## Role
Expert in clean architecture layer dependencies.
## Workflow
1. Read the diff. 2. Flag violations. 3. Suggest fixes.
```

### Skills

Must be a folder with `SKILL.md`. Include `## Usage` (when to invoke) and `## Procedure` (ordered steps). Example:

```markdown
# Skill: Setup
## Usage
Invoke when the user wants to scaffold the project.
## Procedure
### Step 1 -- Gather inputs
Ask for project name and framework.
### Step 2 -- Generate files
Create the directory structure and config files.
```

### Commands

Define trigger, parameters, and behavior:

```markdown
# Command: scaffold
## Trigger
/scaffold <name>
## Parameters
- name: Component name to create.
## Behavior
1. Create folder. 2. Generate template files.
```

### Rules

State constraints directly:

```markdown
# Rule: Naming
- camelCase for variables and functions.
- PascalCase for classes and types.
- UPPER_SNAKE_CASE for constants.
```

### Contexts

Provide background knowledge:

```markdown
# Context: Tech Stack
- Frontend: React + TypeScript
- Backend: Node.js + Express
- Database: PostgreSQL
```

### Instructions

See the dedicated section below -- instructions require special attention.

## Crafting Good Instructions

Instructions are the most important item type to get right. Unlike skills (invoked on demand) or agents (activated explicitly), **instructions are always embedded directly into the AI context** (e.g. `claude.md`). Every token counts, and the AI reads them on every interaction.

### Principles

1. **Self-contained but concise.** The instruction must give the AI enough to act without invoking anything else. Include small inline examples, key rules, and the essential "how". But keep it tight -- avoid walls of text.

2. **Point to skills for depth.** When a topic has an interactive, multi-step workflow, summarize the key points inline and then explicitly direct the AI to invoke the skill. Example:

   > To scaffold a new package interactively, invoke the **`create-package`** skill.

   This way, the AI can handle simple cases from the instruction alone, and delegate complex cases to the skill.

3. **Lead with action, not theory.** Start with what to do, not background. The AI doesn't need a history lesson -- it needs a clear procedure.

4. **Include examples for every concept.** A 3-line example is worth a paragraph of explanation. Use fenced code blocks with realistic content.

5. **Use checklists for validation.** Checkboxes make it easy for the AI to verify its work systematically.

### Instruction Template

```markdown
# <Topic>

<One sentence: what this instruction covers and when to use it.>
<One sentence: reference the relevant skill for interactive/deep workflows.>

## <Core Section>

<Concise explanation with inline example.>

## <Another Section>

<Concise explanation with inline example.>

## Checklist

- [ ] First validation point.
- [ ] Second validation point.

For interactive guidance, invoke the **`<skill-name>`** skill.
```

### Anti-patterns to Avoid

- **No skill reference.** If the package has a skill related to the instruction, the instruction MUST reference it. Otherwise the AI won't know the skill exists.
- **Too verbose.** Instructions are loaded on every interaction. A 500-line instruction wastes tokens. Aim for the minimum needed to act.
- **Too sparse.** If the instruction just says "use the skill", the AI can't handle simple cases without invoking it. Include enough inline detail.
- **No examples.** Abstract rules without examples lead to inconsistent AI behavior. Always show, don't just tell.

## README.md

Every package must include a `README.md`:

```markdown
# my-package
Short description.

## Items
| Type        | Name   | Description                    |
|-------------|--------|--------------------------------|
| Rule        | naming | Enforces naming conventions.   |
| Skill       | setup  | Scaffolds the project.         |
| Instruction | guide  | How to use this package.       |

## Install
invariant install @author/my-package

## Author
my-handle

## License
MIT
```

## Validation Checklist

- [ ] `README.md` exists at the package root.
- [ ] `invariant-package.json` has `name`, `version`, `description`, `author`, `license`.
- [ ] Every item in the manifest has a matching file or folder on disk.
- [ ] Skills are folders containing `SKILL.md`.
- [ ] Instructions reference related skills when they exist.
- [ ] Instructions are concise with inline examples.
- [ ] Package name is lowercase, hyphens only.
- [ ] Version follows semver.

To scaffold a complete package interactively, invoke the **`create-package`** skill.
