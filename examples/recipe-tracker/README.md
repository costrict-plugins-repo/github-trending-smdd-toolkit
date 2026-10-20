# recipe-tracker (SMDD example)

A minimal filled-in SMDD project, shipped inside the `smdd-toolkit` plugin to demonstrate the folder and file contracts.

This example stops at the `docs/` tree — `src/` is intentionally not built, because the point is to show what a project looks like after **Step 2 (outline)** and **Step 3 (per-component deep-dive)** of the SMDD collaboration flow, just before build begins.

## Read it in this order

1. **`about.md`** — the stable scope: V1 cut, V2 defer.
2. **`STATUS.md`** — live dashboard: which components are filled in.
3. **`LOG.md`** — one dated entry per working day.
4. **`docs/ui/home.md`** — the app-wide UI rules for the home route.
5. **`docs/ui/pages/recipe-list.md`** and **`docs/ui/pages/recipe-detail.md`** — the two page specs. Each declares its pair with `docs/backend/apis/recipes.md`.
6. **`docs/backend/apis/recipes.md`** — the API surface. Declares its pair back to the page specs and to the schema.
7. **`docs/backend/schemas/recipe.md`** — the single data model.
8. **`docs/config/env.md`** — the project-level env config.

## What this example demonstrates

- Four-dash frontmatter (`----`) consistently across every md.
- Pairs: `ui/pages/*.md` ↔ `backend/apis/recipes.md`, `backend/apis/recipes.md` ↔ `backend/schemas/recipe.md` (via the `Recipe` type).
- V2 markers inline in each file; no V2-only files upfront.
- File sizes well under 40 lines — short prose, no pseudocode.

## What it does not demonstrate

- The `ui/common/auth.md` ↔ `backend/config/auth.md` pair — V1 is single-user local-only, so auth is out of scope. This is the flagship SMDD pair; a future example project will cover it.
- `docs/backend/core/` and `docs/backend/utils/` — the app has no orchestration-layer work in V1; CRUD is thin enough that the API handlers speak directly to the schema.

## To adapt this for your own project

1. Copy this folder to a new location, rename to your project name.
2. Rewrite `about.md` with your V1/V2 scope.
3. Replace every md under `docs/` with your own components.
4. Keep the folder shape; keep the four-dash frontmatter; keep the pair references bidirectional.
5. When ready, use `smdd-compile` to build component-by-component into `src/`.
