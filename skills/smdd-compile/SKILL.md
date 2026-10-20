---
name: smdd-compile
description: Orchestrator that builds a batch of SMDD components with parallel sub-agents. Use when asked to "build batch N", "implement all schemas in parallel", "compile this folder", or kick off a multi-component build. Determines which targets in the batch can run in parallel (disjoint file sets, satisfied dependencies), spawns one Agent per target with the right model (haiku for mechanical, sonnet for coupled), collects diffs, runs smdd-check over the touched folders, commits. Respects CLAUDE.md. Never parallelizes targets that share a file.
---

# smdd-compile — SMDD batch orchestrator

You are coordinating many `smdd-code` workers. Your job is three things: partition the batch into parallel-safe waves, spawn workers with the right model per target, and post-verify with `smdd-check`.

## Input

A batch definition, provided by the caller:

```
batch: <name>
targets:
  - src: <path>
    spec: <md path>
    pair: <md path, optional>
    deps: [<src paths>]
    model: haiku | sonnet
    notes: <optional>
```

## Procedure

### 1. Pre-flight

- Verify SMDD project: `about.md` + `docs/{ui,backend,config}` at root.
- Verify clean git tree. If dirty, STOP and ask the caller.
- Verify no two targets declare the same `src` path — ever. That's a config error.
- Verify every `pair` and `spec` file exists on disk.

### 2. Partition into waves

Build a dependency DAG from the `deps` field. A target belongs to **wave N** iff all its deps are either (a) already merged to main, or (b) in a wave < N. Within a wave, targets are fully independent.

**Key rule:** if two targets touch the same file (even via imports that would require mutual edits), they go in consecutive waves, NOT one. Err on the side of more waves.

### 3. Branch

Create `compile/<batch-name>` off main. This is the batch branch. All workers commit to it in sequence.

### 4. Execute each wave

For a wave with N targets, issue a SINGLE message with N parallel `Agent` tool calls. Per-target prompt template:

```
You are implementing one SMDD component. Use the smdd-code skill.

Inputs:
- target: <src path>
- spec: <md path>
- pair: <md path or "none">
- deps: [<already-committed src paths>]
- branch: compile/<batch-name>

Read CLAUDE.md and the spec (and pair) before writing. Write the target,
write a test, run the test until green (max 3 tries), commit on the
batch branch. Return the smdd-code structured output — nothing else.

If you cannot complete, return status: failed with a clear reason.
Do not edit any file outside your target set.
```

Pass `model: <target.model>` on each Agent spawn. Use `subagent_type: general-purpose`.

Wait for all workers in the wave to return. Collect their structured outputs.

### 5. Handle wave failures

- If a worker returns `status: failed`, attempt ONE retry with the same prompt. If it fails again, stop the wave and return a batch-failure report. Do not proceed to the next wave.
- If a worker returns `status: ok` but `test_result: fail`, that's a bug in the worker — escalate.

### 6. Post-wave check

After the final wave, for each folder the batch touched, invoke `smdd-check`:

```
smdd-check {docs_folder: docs/<...>, src_folder: src/<...>}
```

If `smdd-check` returns WARN, note in the report. If FAIL, stop and escalate.

### 7. Integration tests

Run the full test suite for the tree(s) the batch touched:

- backend: `uv run pytest -q`
- web: `pnpm -C src/web test`

If either fails, escalate.

### 8. Report back

Return:

```
batch: <name>
branch: compile/<batch-name>
waves_run: <n>
targets_passed: <n>
targets_failed: <n>
check_result: PASS | WARN | FAIL
integration_tests: PASS | FAIL
notes: <one paragraph>
next_action: merge | debug | escalate
```

## Model selection heuristics

**Haiku** (cheap, good-enough for mechanical translation):
- Pydantic schema from a field-list md
- Pure-function util with no cross-module state (e.g., url validator, dedupe helper)
- Single-file React component with no shared state

**Sonnet** (default for anything with judgment):
- Config files (auth, db wiring)
- Core orchestrators (run_engine_probe, compute_visibility)
- API routers wiring multiple schemas + utils
- UI pages with state + multiple API calls
- Any target flagged `coupled` or `keystone` in the batch def

**Opus** (avoid): if you think you need Opus inside a compile, stop and ask the human — it likely means the md is too thin, not that you need a bigger model.

## Parallelism rules (non-negotiable)

1. Siblings in one wave MUST NOT touch the same file. Enforced at pre-flight.
2. Pair targets (ui and backend halves of one concept) belong in CONSECUTIVE WAVES, not the same wave. Build the backend half first, commit it, then the UI half reads the committed backend src as a dep. This preserves `smdd-code`'s one-file-per-worker invariant.
3. Siblings can both `git add`, but serialize commits through the batch branch (workers commit independently; smdd-compile doesn't re-order).
4. A target whose dep is in the same wave is a config error — re-partition.

## Cost awareness

- Wave of 7 Haiku ≈ $0.10-0.50
- Wave of 3 Sonnet ≈ $1-3
- Full mid-size project build end-to-end ≈ $15-30

If a projected wave exceeds $10, flag the human before spawning.

## When NOT to use this skill

- Single-component build → `smdd-code` directly.
- Docs-only work → edit docs/ directly.
- Fixing a failed previous batch → `smdd-debug` (planned).
- No SMDD project structure → refuse; this skill only operates in SMDD trees.

## Example invocation

User says: "Build batch 1 — the schemas."

You:
1. Receive the batch 1 definition from the caller.
2. Pre-flight passes (clean tree, 7 distinct src targets).
3. One wave, 7 parallel Haiku workers, one message with 7 Agent calls.
4. Collect results. All 7 return status: ok.
5. Run `smdd-check docs/backend/schemas/ src/api/schemas/` — PASS.
6. `uv run pytest -q` — PASS.
7. Return batch report, `next_action: merge`.
