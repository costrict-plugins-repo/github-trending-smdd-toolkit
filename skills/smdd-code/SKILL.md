---
name: smdd-code
description: Atomic WORK task for building one SMDD component. Use when asked to "implement this file", "build this component", "code the <X> md", or similar single-component asks inside an SMDD project (one with docs/{ui,backend,config}). Read the component's md, its pair md if one exists, and already-built dependencies; write the code file; write a test; run the test until green; commit on a feature branch; return a diff summary. Never edits files outside the declared target set — that's how sibling agents stay parallel-safe.
---

# smdd-code — atomic SMDD build task

This skill is invoked by a caller (human or `smdd-compile`) that hands you ONE target to build. Your contract is simple: write the target, prove it works, commit, return a structured summary. Don't do more. Don't do less.

## Inputs the caller must provide

- **target** — src path of the file to create or edit (e.g., `src/api/schemas/user.py`)
- **spec** — docs path of the md that specifies it (e.g., `docs/backend/schemas/user.md`)
- **pair** (optional) — the paired md if this is half of a pair (always read both halves before writing a single line of code)
- **deps** (optional) — list of src paths this target imports from; read them to match signatures
- **branch** — the feature branch name the caller is on (you create it if missing, but never switch branches mid-task)

If any required input is missing, stop and ask. Don't guess a path.

## Procedure

1. **Read before writing.** Read the spec md in full. If `pair` is set, read it. Read `CLAUDE.md` at the project root. Read each `deps` file. Only then write.
2. **Respect CLAUDE.md absolutely** — uv + python 3.12, pydantic everywhere non-scalar, no `Field()`, no dataclasses, no emojis, no unasked-for endpoints, one-line logs `(hh:mm:ss) (file:line) (k:v|k:v)`.
3. **Check git state.** If not on `branch`, create it from the current head. Never force-push, never rebase without the caller's say-so.
4. **Write the target file.** Prose from the md does NOT become docstrings unless the md explicitly demands it — one-line comments max per CLAUDE.md.
5. **Write a test file alongside.**
   - backend: `src/api/tests/test_<name>.py` (pytest)
   - web: `src/web/__tests__/<name>.test.tsx` (vitest) or a playwright smoke test if it's a page
6. **Run the test.**
   - backend: `uv run pytest src/api/tests/test_<name>.py -q`
   - web: `pnpm -C src/web test <name>`
7. **If test fails**, debug and retry. Max 3 iterations. If still failing, STOP — do not commit broken code. Return a failure report instead.
8. **Commit.** `git add <target> <test>` then `git commit -m "<component>: implement per <spec md path>"`. One commit per component.
9. **Return** a structured summary (see "Output" below).

## Hard constraints

- **Never touch files outside your declared target set.** If you discover you need to edit a sibling's file, return a blocker — do not silently edit, even if "it's small." Silent edits break sibling agents.
- **Never create a new route that isn't in `docs/backend/apis/*.md`.** If the md implies one, flag it and stop.
- **Never skip the test.** Untested code does not commit.
- **Never use `--no-verify`** on the commit, even if a hook fails. Hook failures are the signal to fix, not bypass.
- **No emojis anywhere** — code, comments, commit message, return summary. Zero tolerance per user's global CLAUDE.md.

## Scope check (run this BEFORE step 1)

If any of the following is true, you are the wrong skill for this task — return early and route to the right one:

- The task implies 3+ files you'd need to own → the caller should have used `smdd-compile`.
- The md is clearly too thin to build from (talks about "what" but not "where it lives" or "what it returns") → return a request to deepen the md first.
- The target has no corresponding md → this isn't an SMDD target; refuse.
- Another agent is currently holding the same branch → stop and ask the orchestrator.

## Output format

Return exactly this shape (as text the caller can parse):

```
status: ok | failed
branch: <branch name>
commit: <sha or "none">
files_changed: [<paths>]
test_result: pass | fail | skipped_with_reason
iterations: <n>
notes: <one paragraph, relevant only>
```

## When NOT to use this skill

- Multi-file batches → `smdd-compile`.
- Cross-folder refactors → not an SMDD primitive; ask the human.
- Docs-only edits → just edit the md directly.
- Bug fixes in already-built code → use `smdd-debug` (planned, not yet built).

## Why this skill exists

Parallel agents work only if each one owns a disjoint file set and returns in a predictable shape. This skill enforces both: exclusive-ownership via the scope check, predictable return via the output format. Without these two invariants, `smdd-compile` can't orchestrate reliably.
