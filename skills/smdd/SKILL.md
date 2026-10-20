---
name: smdd
description: Use this skill when the user mentions SMDD (Structured Markdown Driven Development), asks to "plan in SMDD style," asks to "create an SMDD outline," is working inside a directory that has the layout `docs/{ui,backend,config}` + `about.md` at its root, or asks to scaffold a new project as markdown-first docs before writing code. Enforces the three-step collaboration flow (plan then outline then per-component deep-dive then build) and the SMDD file and folder conventions.
---

# SMDD — Structured Markdown Driven Development

SMDD is a pre-code paradigm: describe the whole product as short prose markdown files organized in a strict folder shape. Code comes last, mirroring the docs tree under `src/`.

The full paradigm writeup lives in this plugin at `docs/paradigm.md`.

## Folder contract

A compliant SMDD project has this exact shape at its root:

```
<project-name>/
  about.md            # stable scope: what, who for, V1 cut, V2 defer
  STATUS.md           # live state: phase, checklist, open decisions, risks
  LOG.md              # append-only history: one dated entry per working day
  docs/
    ui/
      home.md              # first route user hits; no landing page
      design_language.md   # colors, typography, spacing, component style
      uber_navigation.md   # top-level nav map — which pages reach which
      common/              # header, sidebar, auth, theme — app-wide UI rules
      pages/               # one md per page/route
    backend/
      apis/            # one md per API surface group
      core/            # workflows and orchestrators
      config/          # auth, db, llm, engines
      schemas/         # one md per pydantic model
      utils/           # one md per util
    config/            # env, firebase, deploy, secrets
  src/                 # written LAST, mirrors docs/ structure
```

The top-level `about.md` plus the `docs/{ui,backend,config}` split is the compliance test. Missing either means the project is not SMDD.

## File contract

Every md file begins with frontmatter using four-dash delimiters (not YAML three-dash — the paradigm uses four dashes deliberately to distinguish SMDD files from generic markdown):

```
----
name: <short file title>
type: <one or two tags from the taxonomy below>
----
```

Type taxonomy:

- **Tracking artifacts** — `about`, `status`, `log` (used only on the three root-level files of the same names).
- **UI** — `ui rules` (app-wide rules; home.md, design_language.md, uber_navigation.md, common/*), `ui page` (one per route under pages/), `ui config` (UI-side config that pairs with backend config, e.g. ui/common/auth.md).
- **Backend** — `api` (per surface group under apis/), `core` (workflow orchestrators under core/), `utils` (stateless helpers under utils/ — the `core` vs `utils` split is orchestration vs computation), `schema` (data models under schemas/), `backend config` (auth, db, llm, engines under config/).
- **Project config** — `config` (env, deploy, secrets under the top-level `docs/config/`, distinct from backend config).

Body rules:

- Prose first. Short sentences, bullet lists. Small illustrative snippets are fine (routes, env keys, collection paths), but no function bodies.
- Say WHAT the component does and WHY. Not HOW (no algorithm pseudocode beyond the essentials).
- Short. 5-40 lines is the sweet spot. Past 60 lines, split the file.
- Cross-reference other files by their doc path, e.g. `pairs with /docs/backend/config/auth.md`.
- Mark deferred work as "V2" inline. Do NOT create V2-only files upfront.

## Pairs

Some concepts always come in paired docs that must stay aligned. The canonical pairs:

- `ui/common/auth.md` pairs with `backend/config/auth.md`
- `ui/pages/*.md` pairs with `backend/apis/*.md` (the UI page names which endpoints it calls)
- `backend/core/*.md` pairs with `backend/utils/*.md` (workflows orchestrate; utils stay stateless)
- `backend/schemas/*.md` pairs with the pydantic class of the same name in `src/`

When one half of a pair changes, flag the other half for review before drift sets in.

## Three-step collaboration flow

SMDD is a conversation in three numbered steps, then the build. NEVER skip ahead, even in auto mode.

**Step 1 — plan.** Read the user's ask. Surface your understanding. Ask clarifying questions. Name the V1 cut and the V2 defer explicitly. No files written at this stage.

**Step 2 — create the SMDD outline.** Scaffold the full folder tree. Stub every md with frontmatter + 5-15 lines of intent. Create `STATUS.md` and `LOG.md` alongside `about.md`.

**Step 3 — per-component deep-dive.** Walk every md with the user. Deepen each to the level where an engineer (or an agent) can implement with zero further questions. Only after step 3 is the build phase authorized.

**Step 4 — build + test.** Each component becomes an atomic WORK task — see the `smdd-code` skill for the WORK definition (branch → plan.md → code → test → commit → MR).

## Tracking artifacts

Every SMDD project maintains three root-level files, each with a distinct role:

- **`about.md`** — stable scope. Rewritten only when the product's intent itself changes.
- **`STATUS.md`** — live master dashboard. Phase, component checklist, open decisions, risks, links. Updated on every material change.
- **`LOG.md`** — append-only daily log. One entry per working day: what got done, decisions made, next-day intentions, open questions. Never rewrite history; append only.

## Companion skills

Three are built and shipped in this plugin:

- **`smdd-code`** — atomic WORK task: read md + pair + deps → write file + test → commit on a feature branch → return structured diff. Single-file scope, disjoint ownership, zero drift by contract.
- **`smdd-compile`** — batch orchestrator: partitions a batch into parallel-safe waves, spawns one `smdd-code` worker per target via the Agent tool (haiku for mechanical, sonnet for coupled), post-verifies with `smdd-check`, reports.
- **`smdd-check`** — folder-level coherence audit: reads every md + every source file in a matched folder pair, flags naming / field / pair / CLAUDE.md violations. Report-only — fixes route through `smdd-code`.

Planned for future releases (not yet shipped):

- `smdd-init` — bootstrap the folder tree and git repo for a new project.
- `smdd-debug` — take a bug description, point to the relevant md, propose a debug plan, iterate with the user, hand to `smdd-code` on approval.
- `smdd-test` — atomic test task; scope matures with the project.

When a user asks to "build this component" or "compile this batch" inside an SMDD project, route to the companion skills — don't reimplement their work inside this skill.

## When to trigger this skill

- User says "SMDD" or "structured markdown driven development."
- User asks to scaffold a new project as markdown-first before code.
- User references `about.md` alongside `docs/{ui,backend,config}` folders in the conversation.
- User asks to "plan, then outline, then per-component" for a build.

## When NOT to apply

- Quick one-off scripts or single-file edits.
- Bug fixes in non-SMDD codebases (do not retrofit without explicit request).
- Documenting an already-built project — that's documentation work, not SMDD.

## Anti-patterns

- Writing code before steps 1-3 complete.
- YAML three-dash frontmatter instead of four-dash.
- Files past 60 lines — split instead.
- Creating V2-only files upfront.
- Duplicating content between `ui/` and `backend/` instead of using the pairs convention.
- Treating `STATUS.md` as append-only or `LOG.md` as live state — the roles are distinct and must stay that way.
