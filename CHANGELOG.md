# Changelog

All notable changes to `smdd-toolkit` are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project adheres to [Semantic Versioning](https://semver.org/).

## [1.0.0] — 2026-04-20

Initial public release.

### Added

- `smdd` — anchor skill that triggers on SMDD mentions and enforces the three-step collaboration flow (plan, outline, per-component deep-dive, build). Encodes the folder contract (`docs/{ui,backend,config}` + `about.md`) and the four-dash frontmatter file contract.
- `smdd-code` — atomic WORK task for building one SMDD component. Reads the component's md, pair, and deps; writes the code file and a test; runs the test until green; commits on a feature branch; returns a structured summary. Enforces exclusive file ownership so sibling agents stay parallel-safe.
- `smdd-compile` — multi-component batch orchestrator. Partitions targets into parallel-safe waves, spawns one `smdd-code` worker per target with the appropriate model (Haiku for mechanical targets, Sonnet for coupled ones), post-verifies with `smdd-check`.
- `smdd-check` — folder-level coherence audit. Cross-checks every md against every source file for naming, field alignment, pair-contract compliance, and CLAUDE.md-rule violations. Report-only.
- `docs/paradigm.md` — full writeup of the SMDD paradigm for outside readers.
- `examples/recipe-tracker/` — minimal filled-in SMDD project layout.

### Known gaps

- No `smdd-init` skill yet — bootstrapping the folder tree for a new project is manual.
- No `smdd-debug` skill yet — bug-to-fix flow is manual.
- No `smdd-test` skill yet — testing is delegated to `smdd-code`'s per-component tests.
- `smdd-check` CLAUDE.md-rule pass is tuned for Python + TypeScript projects using `uv`, `pnpm`, and Pydantic. Other stacks work but rule-level checks may miss.
