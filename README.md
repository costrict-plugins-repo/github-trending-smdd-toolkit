# smdd-toolkit

> Structured Markdown Driven Development — plan projects as prose markdown before writing code, then build component-by-component with atomic, parallel-safe sub-agents.

SMDD is a pre-code paradigm: describe the whole product as short markdown files in a strict folder shape first. Code comes last, mirroring the docs. Parallel agents work only if each one owns a disjoint file set and reads the same spec — SMDD makes that possible.

## Install

```text
/plugin marketplace add cyvnrs/smdd-toolkit
/plugin install smdd-toolkit@cyvnrs-marketplace
```

That adds four skills to Claude Code: `smdd`, `smdd-code`, `smdd-compile`, `smdd-check`.

## The skills

| Skill | What it does |
|---|---|
| `smdd` | Anchor skill. Triggers on SMDD mentions, enforces the three-step collaboration flow (plan → outline → per-component → build), explains folder and file contracts. |
| `smdd-code` | Atomic WORK task. Builds one component: reads the md + pair + deps, writes the code file + test, commits on a feature branch, returns a structured summary. Never edits files outside the target set. |
| `smdd-compile` | Batch orchestrator. Partitions a set of targets into parallel-safe waves, spawns one `smdd-code` worker per target (Haiku for mechanical, Sonnet for coupled), verifies with `smdd-check`. |
| `smdd-check` | Folder-level coherence audit. Reads every md and every source file in a matched folder pair, flags mismatched fields, orphan files, broken pairs, CLAUDE.md violations. Report-only. |

## Quickstart — 60 seconds

1. In any Claude Code session, say: **"let's scaffold a new project in SMDD style — a recipe tracker."**
2. The `smdd` skill activates, walks you through Step 1 (plan), Step 2 (outline), Step 3 (per-component deep-dive) before writing a line of code.
3. Once the `docs/` tree is filled, say: **"build batch 1"** — `smdd-compile` partitions components into parallel waves and spawns sub-agents that each build one file with tests.
4. Before merging, say: **"run smdd-check on the schemas folder"** — coherence audit runs over docs and code.

See [`examples/recipe-tracker/`](examples/recipe-tracker/) for a minimal filled-in SMDD project layout.

## Why this exists

Two failure modes keep showing up when you hand a coding task to an LLM:

1. **The model fills gaps with assumptions you wouldn't make.** Vague specs become over-engineered code with invented patterns.
2. **Parallel sub-agents drift apart.** Five agents produce five individually-reasonable files that don't cohere — mismatched field names, duplicated utilities, endpoints that don't match their consumers.

SMDD installs a single source of truth (the `docs/` tree) that every agent reads before writing. The three-step flow prevents gap-filling. The `docs/`-mirrors-`src/` convention keeps parallel agents in disjoint file sets. Coherence audits catch what slips through.

Read the full paradigm writeup at [`docs/paradigm.md`](docs/paradigm.md).

## Project layout after `/plugin install`

A project that uses SMDD will look like this:

```text
your-project/
  about.md               # stable scope: what, who for, V1 cut, V2 defer
  STATUS.md              # live state: phase, checklist, open decisions, risks
  LOG.md                 # append-only history: one dated entry per working day
  docs/
    ui/
      home.md            # first route the user hits
      design_language.md # colors, typography, spacing
      uber_navigation.md # top-level nav map
      common/            # header, sidebar, auth, theme
      pages/             # one md per page/route
    backend/
      apis/              # one md per API surface group
      core/              # workflows and orchestrators
      config/            # auth, db, llm, engines
      schemas/           # one md per data model
      utils/             # one md per util
    config/              # env, deploy, secrets
  src/                   # mirrors docs/, written last
```

## Requirements

- Claude Code (CLI, IDE extension, or desktop) with plugin support enabled.
- A `CLAUDE.md` at your project root describing your local conventions (language version, style, logging). SMDD skills read it before writing code.

## Companion conventions

SMDD works best alongside:

- **`uv` + Python 3.12** for backends (the default target in `smdd-code`; any language works — these are just what `smdd-code` knows how to run tests for out of the box).
- **pnpm + vitest** or **playwright** for web frontends.
- **Pydantic** for all non-scalar data movement (no `@dataclass`, no `Field()` sugar — bare `name: type` fields). `smdd-check` audits this.

None of these are hard requirements, but `smdd-check`'s CLAUDE.md-rule pass assumes them. Override by editing your project's `CLAUDE.md`.

## Roadmap

Shipped in v1.0.0: `smdd`, `smdd-code`, `smdd-compile`, `smdd-check`.

Planned:

- `smdd-init` — bootstrap the folder tree and git repo for a new project.
- `smdd-debug` — structured bug-to-fix flow.
- `smdd-test` — atomic test task.
- More example projects beyond `recipe-tracker` — a backend-only service, and a project with the full auth pair (`ui/common/auth.md` ↔ `backend/config/auth.md`).

## Contributing

Issues and PRs welcome at [github.com/cyvnrs/smdd-toolkit](https://github.com/cyvnrs/smdd-toolkit).

When proposing changes to a skill's contract, include a concrete scenario that the change enables or fixes — the skills are tight by design and every addition is a tax on readers.

## License

MIT. See [`LICENSE`](LICENSE).
