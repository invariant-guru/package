# Installed Names and Paths

The full rule for what an item is called once it is installed in someone else's
project. Read this when authoring content that links to your own items, or when a
`unprefixed_path_reference` warning shows up in `invariant package validate`.

> Source of truth: the CLI's `package-naming.md` spec. Rewritten here from the package
> author's point of view; if the two ever disagree, the CLI spec wins.

## Why the prefix exists

Two packages may both ship a `frontend` skill. Without prefixing both write
`.agents/skills/frontend`: the second install silently overwrites the first, and
removing either one deletes the survivor's files. Prefixing makes the path a function
of the owning package, so collisions cannot happen and every file on disk is
attributable.

The name is a **pure function of the package name and the item name** — no filesystem
state, no install order, no collision counters, no content rewriting. You can compute
every installed path by hand, before publishing.

## The slug

```
slug(packageName):
  1. drop a leading "@"
  2. replace "/" with "-"
  3. lowercase
  4. replace every character outside [a-z0-9._-] with "-"
  5. collapse runs of "-" into one "-"
  6. trim leading/trailing "-" and "."
```

| package name | slug |
|---|---|
| `nest-clean-architecture` | `nest-clean-architecture` |
| `@invariant/skills` | `invariant-skills` |
| `Caveman.Tools` | `caveman.tools` |
| `my pkg!` | `my-pkg` |

The scope is kept, not dropped: two scopes owning the same short name must not
collapse onto one folder.

## The qualified name

```
qualified(packageName, itemName):
  s = slug(packageName)
  if itemName === s               -> itemName    (item named after its package)
  if itemName.startsWith(s + "-") -> itemName    (already prefixed by the author)
  else                            -> s + "-" + itemName
```

Idempotent by construction: `qualified(p, qualified(p, x)) === qualified(p, x)`.

| package | item | installed as |
|---|---|---|
| `docs` | `frontend` | `docs-frontend` |
| `docs` | `docs-frontend` | `docs-frontend` |
| `caveman` | `caveman` | `caveman` |
| `@invariant/skills` | `frontend` | `invariant-skills-frontend` |
| `@vercel-labs/skills` | `find-skills` | `vercel-labs-skills-find-skills` |

## Where the prefix applies

| Location | Prefixed | Path |
|---|---|---|
| Canonical store | yes | `.agents/<type>/<qualified>` (folder) or `.agents/<type>/<qualified>.md` (single file) |
| Claude Code | yes | `.claude/<type>/<qualified>` — a symlink into the canonical store |
| Cursor | yes | `.cursor/rules/<qualified>.mdc` |
| Copilot | yes | `.github/instructions/<qualified>.instructions.md` |
| Codex, Aider | inherited | they read `.agents/` or embed the content, so they see the prefixed name |
| Package cache | **no** | `.invariant/cache/<package>@<version>/<type>/<item>` — already scoped by package |
| Contexts, instructions | **no** | `.invariant/contexts/<package>/<name>.md`, `.invariant/instructions/<package>/<name>.md` — already scoped by package, and folded into each target's context file rather than resolved by path |
| Your package repository | **no** | keep authoring `skills/frontend/` — the prefix is an install-time projection |
| CLI arguments | **no** | `invariant add skill:<package>/<item>` takes the item's own name |

Item type is one of `agents`, `skills`, `commands`, `rules`, `contexts`,
`instructions`.

`.agents/` and every target folder are **generated** — rebuilt by `invariant sync` and
kept out of git, the way `node_modules` is. Never author into them, never commit them.

## Which name is the package name

One resolution order, used for the cache folder, the `invariant.json` key,
`active[].package`, and the slug:

1. **Registry install** — the registry package name.
2. **GitHub install with a manifest** — `invariant.json` (or `.invariant.json`) in the
   repository declaring a usable `name`: that declared name wins.
3. **GitHub install without a usable manifest name** — the **repository** segment.
   `github:acme/skills-collection` → `skills-collection`. Never `owner/repo`, never the
   branch.
4. **skills.sh install** — `@<owner>/<repo>`, both segments normalized by the slug
   alphabet. `skills:vercel-labs/skills/find-skills` → `@vercel-labs/skills`. The owner
   is part of the name here because the catalogue's repositories are overwhelmingly
   called `skills` — the repository segment alone would put four catalogues on one
   name. The skill inside keeps its own folder name, so it installs as
   `vercel-labs-skills-find-skills`.
5. **Local authoring** — `invariant.json#name` of the working directory.

## Writing content that references your own items

Nothing rewrites your content. An instruction folded into a target's context file must
spell the installed path itself:

```md
<!-- package name: nest-clean-architecture -->
See [implement-command](.claude/skills/nest-clean-architecture-implement-command/SKILL.md).
```

Two conforming strategies — pick one per package and stay consistent:

| Strategy | What you do | Best when |
|---|---|---|
| **Compute the prefix** | Keep short folder names (`skills/implement-command/`), write the full prefixed path in content | The slug is long or repeats the item name (`@acme/package` + `package-creation` → `acme-package-package-creation`) — short folders keep the repository readable |
| **Pre-prefix the folders** | Name the folder `skills/nest-clean-architecture-implement-command/` | The slug is short and meaningful — repository layout and installed layout become byte-identical, and a skill's `name:` frontmatter matches its installed folder with no install-time mutation |

Pre-prefixing is safe by idempotence: an already-prefixed name is left alone.

## Tooling

```bash
invariant inspect <package> --paths
```

Prints, tab-separated: package, item type, installed name, canonical path. The
copy-paste source when writing references.

```bash
invariant package validate
```

Warns (`unprefixed_path_reference`) when an item's content references
`.claude/<type>/<name>`, `.agents/<type>/<name>`, `.cursor/rules/<name>` or
`.github/instructions/<name>` for one of **your own** items without the prefix, and
names the path that will actually exist. References to another package's files stay
quiet. Warnings are advisory — they never make a package invalid.

## Collisions

Cross-package collisions are impossible: different slug, different folder. Two cases
remain, and both are refused rather than overwritten:

- One package shipping `frontend` **and** `<slug>-frontend` — they resolve to the same
  name. `invariant add` fails naming both; `validate` warns before publish.
- A path already owned by a different package's active item (reachable only through
  slug arithmetic — package `docs` with item `web-frontend` versus package `docs-web`
  with item `frontend`). `invariant add` refuses and names the current owner.

## Upgrading a project installed before prefixing

Run `invariant sync`. Unprefixed canonical items fall outside the active set, so they
are pruned along with their mirrors, and prefixed items are rebuilt from the package
cache. This is a rebuild, not a data migration — `.agents/` and the target folders are
generated. `invariant sync --check` reports it as drift first, without writing.

`invariant.json` needs no edit: `active[].item` keeps the item's own name, and the
installed name is always derived.
