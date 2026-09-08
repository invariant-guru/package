# Writing a Skill

The method for the `skills/` item type: what goes in the frontmatter, how to shape the
body so a host loads only what it needs, and how Invariant's install-time renaming
interacts with both. Read this before writing any `SKILL.md`; the create-package
procedure covers the package around it, this covers the skill itself.

## Contents

- [The portable baseline](#the-portable-baseline)
- [Frontmatter: the two fields every host reads](#frontmatter-the-two-fields-every-host-reads)
- [Writing the description](#writing-the-description)
- [Invariant caveats: folder name, `name:`, and the manifest description](#invariant-caveats-folder-name-name-and-the-manifest-description)
- [Body structure and progressive disclosure](#body-structure-and-progressive-disclosure)
- [Degrees of freedom](#degrees-of-freedom)
- [Workflows and feedback loops](#workflows-and-feedback-loops)
- [Inputs, outputs, and failure handling](#inputs-outputs-and-failure-handling)
- [scripts/, references/, assets/](#scripts-references-assets)
- [Activation examples](#activation-examples)
- [Worked example: a complete small skill](#worked-example-a-complete-small-skill)
- [Host differences](#host-differences)
- [Iterating on a skill](#iterating-on-a-skill)
- [Anti-patterns](#anti-patterns)
- [Sources](#sources)

## The portable baseline

OpenAI and Anthropic converged on the same shape, so a skill written to the
intersection runs unmodified in Claude Code, claude.ai, the Claude Skills API, ChatGPT
and Codex:

```
my-skill/
├── SKILL.md          # required: YAML frontmatter + markdown body
├── references/       # optional: docs read on demand
├── scripts/          # optional: executables run, not read
├── assets/           # optional: templates, icons, output resources
└── agents/openai.yaml  # optional: Codex/ChatGPT presentation + policy
```

Both vendors state the same four rules:

1. One skill, one job. A skill that "helps with documents" never fires reliably.
2. Prefer instructions over scripts, unless you need determinism or external tooling.
3. Write imperative steps with explicit inputs and outputs.
4. Only metadata is preloaded; everything else is read on demand. Budget accordingly.

## Frontmatter: the two fields every host reads

`name` and `description` are the only fields that are required and portable. Anthropic
publishes hard validation limits; write to these and no host will reject the skill:

| Field | Required | Limit | Rules |
|---|---|---|---|
| `name` | yes | 64 characters | lowercase letters, numbers and hyphens only; no XML tags; must not contain the reserved words `anthropic` or `claude` |
| `description` | yes | 1,024 characters, non-empty | no XML tags; states what the skill does **and** when to use it |

```yaml
---
name: writing-release-notes
description: Generates release notes from the git log between two refs and writes them to CHANGELOG.md. Use when the user asks for release notes, a changelog entry, or a summary of what shipped between two tags.
---
```

Naming convention: gerund form (`processing-pdfs`, `analyzing-spreadsheets`,
`writing-release-notes`) reads best because it names the activity. Noun phrases
(`pdf-processing`) and imperatives (`process-pdfs`) are acceptable. `helper`, `utils`,
`tools`, `documents`, `data` are not — they are unmatchable.

Everything beyond these two fields is host-specific. See
[Host differences](#host-differences) before adding a third key.

## Writing the description

The description is the whole discovery mechanism. At startup a host loads only the
`name` and `description` of every installed skill and picks from that list; the body is
read after the pick. A description that does not name its triggers is a skill that never
runs.

Four rules:

- **Third person.** The text is injected into a system prompt alongside dozens of others.
  `Generates release notes…` — not `I can help you write release notes` and not
  `You can use this to…`.
- **What, then when.** One clause for the capability, one clause naming the triggers.
- **Front-load the trigger words.** Some hosts truncate the list to a budget — Codex caps
  the skills list at 2% of context or 8,000 characters. Put the discriminating words
  first so a shortened description still matches.
- **Name the artifacts, not the abstraction.** `.xlsx`, `CHANGELOG.md`, `git tag`,
  `Dockerfile` are matchable; "documents" and "data" are not.

```yaml
# Good — capability, triggers, concrete nouns
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# Bad — no triggers, no nouns, unmatchable against 100 other skills
description: Helps with documents
```

## Invariant caveats: folder name, `name:`, and the manifest description

Three Invariant-specific behaviours change how you fill in the frontmatter.

**1. The installed folder name is the invocation name.** Invariant installs
`skills/<item>/` as `.claude/skills/<qualified>/`, and Claude Code derives the slash
command and the skill's identity from that *directory*, not from `name:`. So
`skills/create-package/` in the `@invariant-guru/cli` repository is invoked as
`/invariant-guru-cli-create-package`. Set `name:` to the qualified name so the two agree:

```yaml
# repository: skills/create-package/SKILL.md   in package @invariant-guru/cli
name: invariant-guru-cli-create-package
```

Or pre-prefix the folder (`skills/invariant-guru-cli-create-package/`) and the two match
by construction — see [package-naming.md](package-naming.md) for the trade-off. Either
way, check the qualified name against the 64-character limit: a long scope plus a long
item name adds up.

**2. `invariant package refresh` does not understand frontmatter.** The CLI derives each
manifest description from the item's first file by taking the first non-empty line and
stripping leading `#` characters. It does not skip YAML frontmatter. On a `SKILL.md`
that opens with `---`, the manifest description becomes the literal string `---`:

```jsonc
// after `invariant package refresh` on a frontmatter-bearing SKILL.md
"skills": [{ "name": "create-package", "description": "---" }]
```

The frontmatter is still correct and still what the host reads — only the registry-facing
manifest description is wrong. Handle it explicitly:

- After any `refresh`, read `invariant.json` and check every skill description.
- If one reads `---`, restore the real one-line description before publishing. It is
  registry copy; the host never uses it.
- Do not delete the frontmatter to work around this. Frontmatter is what makes the skill
  discoverable; the manifest description is a search-result string.

Adding frontmatter does not make the extractor YAML-aware. This is a CLI limitation
documented here, not a fixed behaviour.

**3. Instructions and contexts are not skills.** They are folded verbatim into every
target's context file and are read on *every* interaction. Skills are read only when
selected. Put procedure in a skill; put the always-on summary in an instruction.

## Body structure and progressive disclosure

The body is loaded in full once the skill is selected, so it competes with conversation
history from that point on. Keep `SKILL.md` **under 500 lines** and push depth into
sibling files that are read only when the task needs them.

Assume the model is already competent. Only add what it cannot know: your table names,
your filtering rules, your output paths, your failure modes.

```markdown
## Extract PDF text          ← ~50 tokens, assumes Claude knows what a PDF is

Use pdfplumber:

    import pdfplumber
    with pdfplumber.open("file.pdf") as pdf:
        text = pdf.pages[0].extract_text()
```

Three organising patterns:

| Pattern | Shape | Use when |
|---|---|---|
| Guide with references | `SKILL.md` holds the quick start, links `FORMS.md`, `reference.md`, `examples.md` | One job, with advanced corners |
| Domain split | `SKILL.md` is a navigation table into `reference/finance.md`, `reference/sales.md`, … | Several disjoint knowledge domains |
| Conditional detail | Basic case inline, `→ See REDLINING.md` for the hard case | The hard case is rare |

Two structural rules:

- **Keep references one level deep from `SKILL.md`.** A file reached through another file
  is often only partially read (`head -100`), so the model gets a truncated answer and
  does not know it. Every reference file links directly from `SKILL.md`.
- **Give any reference over 100 lines a table of contents**, so a partial read still shows
  the full scope of what the file covers.

Name files for their content — `form_validation_rules.md`, not `doc2.md` — and always use
forward slashes, including on Windows.

## Degrees of freedom

Match specificity to how fragile the task is.

| Freedom | Form | Use when | Example |
|---|---|---|---|
| High | Prose steps | Several approaches are valid; context decides | "1. Analyze structure 2. Check for edge cases 3. Suggest improvements" |
| Medium | Template with parameters | A preferred pattern exists, variation is fine | `generate_report(data, format="markdown", include_charts=True)` |
| Low | One exact command, no options | Fragile, destructive, or order-dependent | "Run exactly `python scripts/migrate.py --verify --backup`. Do not add flags." |

The test: a narrow bridge with cliffs on both sides gets exact instructions; an open
field gets a direction and trust. Over-specifying an open field wastes tokens and blocks
better routes; under-specifying a bridge produces a plausible command that corrupts data.

## Workflows and feedback loops

For anything multi-step, give a checklist the agent copies into its response and ticks
off. It stops steps being silently skipped.

```markdown
## Release notes workflow

Copy this checklist and check items off as you complete them:

    - [ ] Step 1: Resolve the two refs
    - [ ] Step 2: Collect the commit range
    - [ ] Step 3: Group commits by type
    - [ ] Step 4: Write the entry into CHANGELOG.md
    - [ ] Step 5: Verify every commit is accounted for

**Step 1: Resolve the two refs**
…
```

Then close the loop: **run a check → fix what it reports → run it again → only proceed
when it passes.** The check can be a script, or a document the agent reads and compares
against. Both work; state which one explicitly.

```markdown
## Content review process

1. Draft the content following STYLE_GUIDE.md
2. Review against the checklist: terminology, example format, required sections
3. If issues found: note each one with its section, revise, review again
4. Only proceed when every requirement is met
```

## Inputs, outputs, and failure handling

A skill that does not name its inputs and outputs is a topic, not a procedure. Every
skill states:

- **Inputs** — each field, where it comes from, and its default. Reuse what the user
  already said; ask only for what is missing and consequential.
- **Outputs** — exact paths and formats. `CHANGELOG.md`, `fields.json`, `dist/report.pdf`
  — not "a report".
- **Preconditions** — what must exist first, and what to do when it does not.
- **Failure modes** — the actual error strings the agent will see, and the recovery for
  each. This is the part that is almost always missing.
- **Done** — the condition under which the skill stops.

```markdown
## Inputs

| Input | Source | Default |
|---|---|---|
| `from` | git ref the user names, else the latest tag | `git describe --tags --abbrev=0` |
| `to` | git ref the user names | `HEAD` |
| `output` | path the user names | `CHANGELOG.md` |

## Failure modes

- `fatal: No names found, cannot describe anything.` — the repository has no tags. Ask
  the user for an explicit `from` ref; do not fall back to the root commit.
- Empty commit range — report "no commits between <from> and <to>" and stop. Do not
  write an empty section.
```

For scripts, solve rather than defer: handle `FileNotFoundError` and `PermissionError`
inside the script instead of letting the agent improvise. Justify every constant
(`REQUEST_TIMEOUT = 30  # HTTP requests typically complete within 30s`) — an unexplained
`47` is a number nobody, agent or human, can safely change.

## scripts/, references/, assets/

| Directory | Loaded how | Put here |
|---|---|---|
| `scripts/` | executed; only the output costs tokens | deterministic work, validators, anything fragile enough that generated code would be a risk |
| `references/` | read on demand, in full | schemas, API docs, long-form procedure, examples |
| `assets/` | copied or referenced, never read into context | templates, boilerplate, icons declared in `agents/openai.yaml` |

Pre-written scripts beat generated code: more reliable, cheaper, consistent across runs.
Always state which mode you mean —

- "Run `analyze_form.py` to extract fields" → execute
- "See `analyze_form.py` for the extraction algorithm" → read as reference

Never assume a package is installed. `Install required package: pip install pypdf`, then
the usage. And if the skill calls MCP tools, use fully qualified names
(`GitHub:create_issue`) — a bare tool name fails to resolve when several servers are
connected.

## Activation examples

Include, in the skill itself, requests that should fire it and one nearby request that
should not. This is how you find out the description is too broad before a user does.

```markdown
## When this fires

- "write the release notes for v2.3.0"
- "what shipped between v2.2.0 and main?"
- "update the changelog"

## When it does not

- "write a commit message for these changes" — that is a commit message, not a release
  entry. Do not open CHANGELOG.md.
```

## Worked example: a complete small skill

`skills/writing-release-notes/SKILL.md`, in a package named `@acme/dev` (slug `acme-dev`,
so it installs as `acme-dev-writing-release-notes`):

````markdown
---
name: acme-dev-writing-release-notes
description: Generates release notes from the git log between two refs and writes them into CHANGELOG.md under a version heading. Use when the user asks for release notes, a changelog entry, or a summary of what shipped between two tags.
---

# Writing release notes

Turns a commit range into a CHANGELOG.md entry grouped by change type.

## Inputs

| Input | Source | Default |
|---|---|---|
| `from` | ref the user names | `git describe --tags --abbrev=0` (latest tag) |
| `to` | ref the user names | `HEAD` |
| `version` | the version being released | ask; there is no safe default |
| `output` | file to write | `CHANGELOG.md` |

Ask only for `version` when the user has not said it. Infer the rest.

## Procedure

Copy this checklist and check items off as you go:

    - [ ] 1 Resolve refs
    - [ ] 2 Collect commits
    - [ ] 3 Group by type
    - [ ] 4 Write the entry
    - [ ] 5 Verify

### Step 1 — Resolve refs

    git rev-parse --verify <from> && git rev-parse --verify <to>

If either fails with `fatal: Needed a single revision`, the ref does not exist: list
`git tag --sort=-creatordate | head -10` and ask which one was meant. Do not guess.

### Step 2 — Collect commits

    git log --no-merges --pretty=format:'%h %s' <from>..<to>

An empty result means nothing shipped: say so and stop. Do not write an empty section.

### Step 3 — Group by type

Bucket by Conventional Commit prefix: `feat:` → Added, `fix:` → Fixed, `perf:` →
Performance, `refactor:`/`chore:` → Internal. Anything unprefixed goes under Other —
never drop it.

### Step 4 — Write the entry

Insert directly below the `# Changelog` heading, never at the end of the file:

    ## <version> — <YYYY-MM-DD>

    ### Added
    - <subject> (<short sha>)

    ### Fixed
    - <subject> (<short sha>)

Omit any section with no commits.

### Step 5 — Verify

Every sha from Step 2 appears exactly once in the new entry. If any is missing, return
to Step 3. Report the count written.

## When this fires

"write the release notes for v2.3.0" · "what shipped between v2.2.0 and main?" ·
"update the changelog"

## When it does not

"write a commit message for these changes" — that is a commit message. Do not touch
CHANGELOG.md.
````

Note what the example does *not* do: no explanation of what git is, no list of four
changelog libraries to choose from, no "follow best practices". Every line is either a
command, a decision rule, or a failure recovery.

## Host differences

The two portable fields work everywhere. Beyond them, hosts diverge — and a key one host
ignores is not always a key another host ignores safely.

| Concern | Anthropic API / claude.ai | Claude Code | ChatGPT / Codex |
|---|---|---|---|
| Required frontmatter | `name`, `description` | `description` recommended; `name` defaults to the directory name and does **not** change the invocation name | `name`, `description` |
| Also portable | `license`, `compatibility`, `metadata`, `allowed-tools` | same | — |
| Explicit invocation | — | `/skill-name` | `@skill` in ChatGPT, `$skill` in Codex CLI |
| Suppress auto-invocation | — | `disable-model-invocation: true` | `policy.allow_implicit_invocation: false` in `agents/openai.yaml` |
| Hide from the user menu | — | `user-invocable: false` | — |
| Other host-only keys | — | `when_to_use`, `argument-hint`, `arguments`, `disallowed-tools`, `model`, `effort`, `context: fork`, `agent`, `background`, `hooks`, `paths`, `shell` | presentation and dependency keys live in `agents/openai.yaml`, not in the frontmatter |
| Discovery roots | uploaded skill | `.claude/skills/`, `~/.claude/skills/`, plugin and enterprise dirs | `.agents/skills` upward from cwd, `$REPO_ROOT/.agents/skills`, `$HOME/.agents/skills`, `/etc/codex/skills` |
| Metadata budget | metadata preloaded, body on demand | same | skills list capped at 2% of context or 8,000 characters |

The Claude Code-only keys **fail on upload to claude.ai** and mean nothing to Codex. So:

- Keep the frontmatter to `name` + `description` unless the workflow genuinely needs an
  invocation change.
- `disable-model-invocation: true` turns a skill into a manual-only command. Add it only
  when automatic firing would be wrong, and say so in the body — a user who cannot see
  the frontmatter will otherwise wonder why the skill never triggers.
- `agents/openai.yaml` is Codex/ChatGPT presentation and policy only. It never replaces
  the frontmatter:

  ```yaml
  interface:
    display_name: "Release Notes"
    short_description: "Turn a commit range into a CHANGELOG entry"
    icon_small: "./assets/small-logo.svg"
    brand_color: "#3B82F6"
  policy:
    allow_implicit_invocation: false   # defaults to true
  dependencies:
    tools:
      - type: "mcp"
        value: "openaiDeveloperDocs"
  ```

  Invariant bundles the whole skill folder, so this file ships intact to hosts that read
  it and is inert everywhere else.

Codex reads `.agents/skills` — which is exactly Invariant's canonical store. A package
installed with the `codex` target is therefore discovered by Codex under its *qualified*
name, one more reason to keep `name:` and the installed folder in agreement.

## Iterating on a skill

Skills are tuned against observed behaviour, not imagined requirements. The published
loop, in order:

1. **Find the gap.** Run the task with no skill. Write down what the agent got wrong or
   kept asking for.
2. **Write the evaluations first** — three scenarios covering those gaps, before writing
   documentation. Each is a query, its input files, and the behaviour you expect.
3. **Baseline.** Measure without the skill.
4. **Write the minimum** that closes the gaps.
5. **Test with a fresh agent instance** that has the skill loaded, on real tasks rather
   than rehearsed ones.
6. **Watch how it navigates.** Files read in an unexpected order means the structure is
   not as obvious as you thought. A reference file never opened is either unnecessary or
   badly signalled. The same file read repeatedly belongs in `SKILL.md`.
7. **Refine and repeat.**

```json
{
  "skills": ["writing-release-notes"],
  "query": "Write release notes for v2.3.0",
  "files": ["test-repo/"],
  "expected_behavior": [
    "Resolves the previous tag rather than asking for it",
    "Groups commits by Conventional Commit prefix",
    "Inserts the entry below the # Changelog heading, not at end of file"
  ]
}
```

There is no built-in runner for this format — it is a rubric you execute yourself. Test
against every model you intend to ship for: what Opus infers, Haiku may need spelled out.

## Anti-patterns

- **Vague description.** `Helps with documents` never wins a selection.
- **First person.** `I can help you…` degrades discovery; the text is system-prompt copy.
- **Offering four options.** `pypdf, or pdfplumber, or PyMuPDF, or…` — give one default
  and one escape hatch for the case that breaks it.
- **Nested references.** `SKILL.md → advanced.md → details.md` gets partially read.
- **Time-sensitive text.** "Before August 2025, use the old API." Put superseded material
  in a collapsed `## Old patterns` section instead.
- **Drifting terminology.** Pick "field" or "box" or "control" and never mix them.
- **Windows paths.** `scripts\helper.py` breaks on Unix. Forward slashes only.
- **Explaining what the model knows.** A paragraph on what a PDF is costs tokens forever
  and teaches nothing.
- **"Follow best practices."** Not a step. Name the practice or delete the line.
- **Dropping frontmatter to appease `invariant package refresh`.** Fix the manifest
  description instead — see the caveats above.

## Sources

Verified 2026-09-08:

- [OpenAI — Build skills](https://learn.chatgpt.com/docs/build-skills): layout,
  frontmatter, discovery roots, the 2% / 8,000-character metadata budget,
  `agents/openai.yaml`.
- [Anthropic — Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices):
  `name`/`description` limits, progressive disclosure, degrees of freedom, workflows and
  feedback loops, evaluation loop, anti-patterns.
- [Anthropic — Skills in Claude Code](https://code.claude.com/docs/en/skills): Claude
  Code frontmatter keys, invocation control, which keys fail elsewhere.

Invariant-specific behaviour was read from the installed CLI (`invariant package --help`,
`invariant inspect --paths`, and the manifest description extractor) rather than from
these pages. Recheck the vendor claims against the links above before treating any of
them as a requirement.
