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

For each item file, generate starter content:

- **Agents**: Write a role description, approach, and workflow outline.
- **Skills**: Write a `SKILL.md` with usage instructions and a step-by-step procedure.
- **Commands**: Write trigger syntax, parameters, and expected behavior.
- **Rules**: Write the constraints and guidelines to enforce.
- **Contexts**: Write background knowledge and reference material.
- **Instructions**: Write ordered procedural steps.

### Step 5 -- Validate

Verify that:
- A `README.md` exists at the package root.
- Every item in `invariant-package.json` has a corresponding file or folder.
- All skill items are folders with a `SKILL.md`.
- The package name is lowercase with only hyphens.
- The version follows semantic versioning.

### Step 6 -- Summary

Present the user with:
- The generated directory tree.
- The contents of `invariant-package.json`.
- A reminder that they can publish with `invariant publish`.
