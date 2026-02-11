# Skill: Create Invariant Package

Create a new Invariant package by scaffolding the directory structure, manifest, and item files.

## Usage

Invoke this skill when a user wants to create a new Invariant package. Gather the required information, then generate all files.

## Procedure

### Step 1 -- Gather package metadata

Ask the user for:
- **Package name** (lowercase, hyphens allowed)
- **Description** (one sentence)
- **Author** name or handle
- **License** (default: `MIT`)
- **Repository URL** (optional)

### Step 2 -- Determine items to include

Ask the user which item types they need. For each selected type, collect:
- **Item name** (lowercase, hyphens allowed)
- **Item description** (one sentence)

Available types: `agents`, `skills`, `commands`, `rules`, `contexts`, `instructions`.

### Step 3 -- Scaffold the directory structure

Create the package root directory with the package name. Inside it, create:

1. `invariant-package.json` with all metadata and item declarations.
2. `README.md` describing the package for humans (name, description, items table, install instructions, author, license).
3. A directory for each selected item type.
4. For each item:
   - If it is a **skill**: create a folder with the item name containing `SKILL.md`.
   - Otherwise: create a single `.md` file with the item name.

### Step 4 -- Generate item content

For each item file, generate starter content following these guidelines:

- **Agents**: Write a role description (`## Role`), approach (`## Approach`), and workflow (`## Workflow`).
- **Skills**: Write a `SKILL.md` with `## Usage` (when to invoke) and `## Procedure` (ordered `### Step N` sections).
- **Commands**: Write `## Trigger`, `## Parameters`, and `## Behavior`.
- **Rules**: State constraints directly as bullet points.
- **Contexts**: Provide background knowledge and reference material.
- **Instructions**: Follow the instruction-crafting principles below.

#### Crafting instructions

Instructions are embedded directly into the AI context on every interaction. They must be:

1. **Package-contents-first** -- open with a "This Package Contains" table listing every item in the package (name, type, when to use). Add a note that Invariant allows partial installs, so some items may not be in the `.claude` folder and that's expected.
2. **Self-contained but concise** -- include small inline examples and key rules so the AI can act without invoking anything else. Avoid walls of text.
3. **Skill-aware** -- if the package includes a skill related to the instruction, the instruction MUST explicitly reference it (e.g. "For interactive scaffolding, invoke the **`my-skill`** skill."). This is mandatory.
4. **Example-driven** -- every concept should have a short fenced code block example. Show, don't just tell.
5. **Action-first** -- lead with what to do, not background theory.
6. **Checklist-ended** -- close with a validation checklist using `- [ ]` checkboxes.

Use this template:

```markdown
# <Topic>

<What this covers.> For interactive guidance, invoke the **`<skill>`** skill.

## This Package Contains

| Item | Type | When to use |
|------|------|-------------|
| `<instruction>` | Instruction | <short description> |
| `<skill>` | Skill | <short description> |

> **Partial installs:** Some items above may not be present in your
> `.claude` folder. Invariant lets users install only a subset of a
> package. Use what's available.

## <Section>

<Concise explanation.>

\`\`\`example
...
\`\`\`

## Checklist

- [ ] Validation point 1.
- [ ] Validation point 2.

For deeper workflows, invoke the **`<skill>`** skill.
```

### Step 5 -- Validate

Verify that:
- A `README.md` exists at the package root.
- Every item in `invariant-package.json` has a corresponding file or folder.
- All skill items are folders with a `SKILL.md`.
- Instructions open with a "This Package Contains" table and a partial-install note.
- Instructions reference related skills.
- The package name is lowercase with only hyphens.
- The version follows semantic versioning.

### Step 6 -- Summary

Present the user with:
- The generated directory tree.
- The contents of `invariant-package.json`.
- A reminder that they can publish with `invariant publish`.
