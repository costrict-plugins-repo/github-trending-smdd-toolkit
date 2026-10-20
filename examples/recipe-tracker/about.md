----
name: recipe-tracker
type: about
----

# recipe-tracker

A minimal personal recipe manager. The user adds recipes they cook regularly, tags them, and pulls up the tag view when deciding what to cook this week.

## Who it is for

One user, single-tenant, local-only V1. Not a social app, not a meal planner, not a shopping-list generator — those are V2 asks.

## V1 cut

- List recipes (home page).
- Add, edit, delete a recipe (title, ingredients, steps, tags).
- Filter the list by a single tag.

## V2 defer

- Multi-user accounts and auth.
- Shopping-list generation from a selected week's recipes.
- Image upload for each recipe.
- Recipe import from a URL.

## Stack

Backend Python 3.12 + FastAPI + Pydantic, SQLite for storage. Frontend Next.js + shadcn. Single-repo monorepo with `src/api/` and `src/web/`.
