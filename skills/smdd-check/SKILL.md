---
name: smdd-check
description: Folder-level coherence audit for an SMDD project. Reads every md in docs/<folder> and every corresponding source file under src/, cross-checks naming, field alignment, pair-contract compliance, and CLAUDE.md rule violations. Returns PASS/FAIL/WARN with a structured issue list. Use after an smdd-compile batch completes, as a pre-PR lint, or whenever coherence drift is suspected. Never auto-fixes — it only reports; fixes go back through smdd-code.
---

# smdd-check — SMDD folder coherence audit

You are a linter, not a builder. Read docs, read code, compare them, report. Do NOT edit any file.

## Input

Either:
- `{docs_folder, src_folder}` — an explicit pair (e.g., `docs/backend/schemas/`, `src/api/schemas/`)
- Or `{docs_folder}` alone — you infer the src folder by SMDD's mirroring convention

## What to check

### 1. Presence

Every non-V2 md in `docs_folder` must have a corresponding source file in `src_folder`. Every source file must have a corresponding md.

- md with no src, and md is not marked V2 → **error**
- src with no md → **error** (code without a spec is drift)
- md marked `V2` or `type: ... | V2` with no src → **expected** (silent pass)

### 2. Naming

- Pydantic class names match md titles. `user.md` titled "user schema" → `class User` in `user.py`.
- React component names match file names. `header.md` → `Header` component in `header.tsx`.
- Util singleton names follow CLAUDE.md: `llm_utils.py` exports `llm_utils = LLMUtils()` at file bottom.

### 3. Field alignment (schemas only)

This rule is Python-specific (pydantic). For other languages, adapt the check to the language's equivalent data-class type or skip.

- Every field listed in the md exists in the pydantic class with a matching type.
- Every field in the pydantic class is listed in the md.
- Extra field in code not in md → **error** ("undocumented field").
- Field in md not in code → **error** ("unimplemented field").

### 4. Pair contracts

If the md has a `pairs with /docs/<path>` reference:
- Verify the pair md exists.
- Verify the pair md references this one back (bidirectional link).
- Verify the shared concept (token shape, api contract, state shape) agrees between the two halves by reading both.

If the pair is a ui/common and backend/config pair, also verify the two implementation files (e.g., `src/web/components/common/auth.tsx` and `src/api/config/auth.py`) use matching field names for the shared payload.

### 5. CLAUDE.md compliance

The rules below reflect a typical Python+TypeScript SMDD stack. If the project's `CLAUDE.md` defines different conventions (different language, different logging format, different data-class library), substitute those rules — the intent of this pass is "audit against the project's stated rules," not "enforce these specific rules."

Default Python+TypeScript rules:
- **No `Field(...)` usage** in pydantic. Bare `name: type` only.
- **No `@dataclass` decorator.** Pydantic only.
- **No emojis** anywhere — code, comments, strings, logs. Zero tolerance.
- **Log format** matches `(hh:mm:ss) (file:line) (k:v|k:v)`. Flag any `print()` or free-form log line.
- **uv + python 3.12** in `pyproject.toml` at project root.
- **No unasked-for endpoints.** Every `@router.get/post` must map to a route named in `docs/backend/apis/*.md`.

### 6. Dead docs

An md in `docs_folder` with no corresponding code AND not marked V2 → **warn** ("doc is either V2 or orphan — classify").

## Output

Exactly this shape:

```
status: PASS | WARN | FAIL
folder: <docs_folder> <-> <src_folder>
coverage:
  docs_count: <n>
  src_count: <n>
  paired_count: <n>
  orphan_docs: <n>
  orphan_src: <n>
issues:
  - severity: error | warn
    file: <path>
    line: <n or null>
    rule: <rule id from the list above>
    message: <one line>
  ...
```

`status`:
- PASS → zero errors, zero warns
- WARN → zero errors, ≥1 warn
- FAIL → ≥1 error

## Procedure

1. Glob `docs_folder/*.md`, glob `src_folder/` for built files (`.py` for backend, `.ts/.tsx` for web).
2. Build the expected-vs-actual file map.
3. For each md, parse frontmatter (four-dash SMDD style). Note `type` tag and any `| V2` modifier.
4. For each src file, parse imports + top-level declarations (regex is fine for the lint level we need; don't invoke a real AST parser unless precision demands it).
5. Run each rule above; accumulate issues.
6. Return the structured report. **Do not auto-fix anything.** Fixes route through `smdd-code` only, after a human triages the issues.

## When NOT to use this skill

- During an active build — race with `smdd-code` writes. Run after `smdd-compile`'s final wave commits.
- On docs with no intended src (design_language.md, uber_navigation.md, home.md content) — skip those explicitly.
- As a style-only linter — this is coherence, not style. Use your language's own linter for style.

## Why this skill exists

`smdd-compile` workers are confined to one file each; none of them sees the folder-level picture. Without an audit tier, a batch of 7 parallel workers can produce 7 individually-correct files that together don't cohere (e.g., `user.py` has `email` but the md says `email_address`). This skill catches that in one pass, without a human eyeballing.
