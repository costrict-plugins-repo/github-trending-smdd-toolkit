# SMDD — Structured Markdown Driven Development

> A pre-code paradigm: describe the entire product as short prose markdown files organized in a strict folder shape. Code comes last, mirroring the docs tree.

## The problem SMDD solves

Two things keep going wrong when you hand a coding task to an LLM agent:

1. **The model fills gaps with assumptions you wouldn't make.** Vague specs become verbose, over-engineered code. The model invents patterns, helper files, and error handling you never asked for.
2. **Parallel agents drift apart.** Spawn five sub-agents on five files, and by commit time you have five individually-reasonable files that don't cohere — mismatched field names, duplicated utilities, endpoints that don't match their consumers.

The root cause of both is the same: **no single source of truth describes the whole product in prose that every agent reads before writing code.**

SMDD is the discipline that installs that source of truth.

## The idea in one sentence

Before any code is written, the product is described as a tree of short markdown files — one per component — and every subsequent build action reads those files as the spec. Code mirrors the docs. Docs describe what and why; code is what and how.

## The folder contract

A compliant SMDD project has this shape at its root:

```
<project-name>/
  about.md            # stable scope: what, who for, V1 cut, V2 defer
  STATUS.md           # live state: phase, checklist, open decisions, risks
  LOG.md              # append-only history: one dated entry per working day
  docs/
    ui/
      home.md              # first route the user hits; no landing page
      design_language.md   # colors, typography, spacing, component style
      uber_navigation.md   # top-level nav map — which pages reach which
      common/              # app-wide UI rules (header, sidebar, auth, theme)
      pages/               # one md per page/route
    backend/
      apis/            # one md per API surface group
      core/            # workflows and orchestrators
      config/          # auth, db, llm, engines
      schemas/         # one md per data model
      utils/           # one md per util
    config/            # env, deploy, secrets
  src/                 # written LAST, mirrors docs/ structure
```

The **top-level `about.md` plus the `docs/{ui,backend,config}` split** is the compliance test. If a project is missing either, it is not SMDD.

## The file contract

Every md starts with four-dash frontmatter. Four dashes instead of YAML's three are deliberate — they mark a file as SMDD-paradigm prose, not generic markdown. Note that standard markdown renderers (GitHub, Obsidian, VSCode preview) expect three-dash YAML and will render four-dash frontmatter as visible content. That is an accepted tradeoff — SMDD files are read by agents more often than by preview panes.

```
----
name: <short title>
type: <tag from taxonomy below>
----
```

Type taxonomy:

- **Tracking artifacts** — `about`, `status`, `log` (used only on the three root-level files of the same names).
- **UI** — `ui rules` (app-wide rules; home.md, design_language.md, uber_navigation.md, common/*), `ui page` (one per route under pages/), `ui config` (UI-side config that pairs with a backend config, e.g. ui/common/auth.md).
- **Backend** — `api` (per surface group under apis/), `core` (workflow orchestrators under core/), `utils` (stateless helpers under utils/ — the `core` vs `utils` split is orchestration vs computation), `schema` (data models under schemas/), `backend config` (auth, db, llm, engines under config/).
- **Project config** — `config` (env, deploy, secrets under the top-level `docs/config/`, distinct from backend config).

Body rules:

- **Prose first.** Short sentences, bullet lists. Small snippets are fine (routes, env keys, collection paths). No function bodies.
- **WHAT and WHY, not HOW.** If you find yourself writing pseudocode, you are writing a code file in the wrong folder.
- **Short.** 5-40 lines is the sweet spot. Past 60 lines, split the file.
- **Cross-reference** other docs by their doc path: `pairs with /docs/backend/config/auth.md`.
- **Mark deferred work as "V2" inline.** Never create V2-only files upfront.

## Pairs

Some concepts always come in paired docs that must stay aligned. The canonical pairs:

| Pair | Meaning |
|---|---|
| `ui/common/auth.md` ↔ `backend/config/auth.md` | UI auth UX and backend auth config share a token shape |
| `ui/pages/*.md` ↔ `backend/apis/*.md` | A UI page names which endpoints it calls |
| `backend/core/*.md` ↔ `backend/utils/*.md` | Workflows orchestrate; utils stay stateless |
| `backend/schemas/*.md` ↔ the pydantic class in `src/` | Fields listed in md must match fields in code |

When one half of a pair changes, flag the other half for review before drift sets in. The `smdd-check` skill audits pair alignment automatically.

## The three-step collaboration flow

SMDD is a conversation in three numbered steps, then the build:

**Step 1 — plan.** Surface understanding. Ask clarifying questions. Name the V1 cut and the V2 defer explicitly. No files written.

**Step 2 — outline.** Scaffold the full folder tree. Stub every md with frontmatter plus 5-15 lines of intent. Create `STATUS.md` and `LOG.md` alongside `about.md`.

**Step 3 — per-component deep-dive.** Walk every md with the user. Deepen each to the level where an engineer (or an agent) can implement with zero further questions.

Only after Step 3 is the build phase authorized.

**This flow is non-negotiable, including in auto/agent mode.** Skipping ahead is the most common way SMDD projects drift. The outline step is what locks in intent across the project; the per-component step is what prevents LLM gap-filling during the build.

## Tracking artifacts

Three root-level files, each with a distinct role — don't let them merge:

- **`about.md`** — stable scope. Rewritten only when the product's intent itself changes.
- **`STATUS.md`** — live master dashboard. Phase, component checklist, open decisions, risks, links. Overwritten on every material change.
- **`LOG.md`** — append-only daily history. One entry per working day: what got done, decisions made, next-day intentions, open questions. Never rewrite.

Treating `STATUS.md` as append-only or `LOG.md` as live state is an anti-pattern — the roles collapse and both files become useless.

## Build phase — atomic WORK

After Step 3, each component becomes an atomic WORK task. A WORK task is:

1. Create a feature branch.
2. Read the component's md, its pair if it has one, and any already-built deps.
3. Write the code file + a test file.
4. Run the test until green (max 3 iterations).
5. Commit one file per task.
6. Return a structured summary (`status`, `branch`, `commit`, `files_changed`, `test_result`).

This is what the `smdd-code` skill does. The invariant it enforces — **one agent owns exactly one target file** — is what makes parallel builds safe. The `smdd-compile` skill layers on top: partition a batch of components into parallel-safe waves, spawn one `smdd-code` worker per target per wave.

Coherence is verified by `smdd-check`, a folder-level audit that reads every md and every source file and flags mismatches (undocumented fields, orphan files, broken pairs, CLAUDE.md rule violations).

## Why markdown, not a DSL or schema

Markdown is the lowest-friction format humans and LLMs both read and write fluently. A Pydantic schema or JSON-schema spec would encode more precisely, but:

- Humans don't review schemas as carefully as prose.
- LLMs generate prose that shapes product intent better than they generate schemas.
- Mixing prose reasoning with small embedded snippets (routes, env keys) is natural in markdown; awkward in any structured format.

The structure comes from the folder layout and frontmatter, not from inside the files. That's enough structure to drive builds without turning the writing experience into form-filling.

## What SMDD is not

- **Not documentation of existing code.** SMDD runs before the code exists; retrofitting it to a built project is documentation work and a different discipline.
- **Not spec-first for one-off scripts.** If the whole job fits in a single file, SMDD's overhead dominates the value. Use SMDD when a project has enough surface area that drift between components is a real risk.
- **Not a replacement for tests.** Coherence audits catch *spec drift*, not *logic bugs*. Tests still matter.
- **Not prescriptive about languages or frameworks.** The docs tree is language-agnostic. The `src/` tree mirrors it in whatever language you chose.

## The companion skills (shipped in this plugin)

- **`smdd`** — the anchor skill. Triggers on SMDD mentions, enforces the three-step flow, explains the folder and file contracts.
- **`smdd-code`** — atomic single-component WORK task.
- **`smdd-compile`** — multi-component batch orchestrator.
- **`smdd-check`** — folder-level coherence audit (report-only).

Planned (not yet shipped):

- `smdd-init` — bootstrap the folder tree and git repo for a new project.
- `smdd-debug` — structured bug-to-fix flow: point to md, propose debug plan, hand to `smdd-code`.
- `smdd-test` — atomic test task; scope will mature alongside the paradigm.

## A note on CLAUDE.md

SMDD pairs well with a project-root `CLAUDE.md` that encodes the user's local conventions (language version, package manager, style rules, logging format, API constraints). Every `smdd-code` task reads CLAUDE.md before writing, and `smdd-check` includes CLAUDE.md-rule violations in its audit. The paradigm doesn't prescribe what CLAUDE.md should contain — that's user preference — but it does assume one exists.
