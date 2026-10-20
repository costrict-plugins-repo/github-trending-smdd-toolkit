----
name: recipe-list
type: ui page
----

# recipe-list

Route: `/`.

Shows all recipes as cards. Each card has title, first-line-of-ingredients preview, and the tags. Click a card to navigate to `/recipes/<id>` (see `recipe-detail.md`).

## Filter bar

A single row of tag chips above the list. Clicking a chip filters the list to recipes with that tag. Clicking again clears. Only one tag at a time in V1.

## Empty state

"No recipes yet. Click 'Add recipe' to create your first." with a link.

## Pairs with

`docs/backend/apis/recipes.md` — this page calls `GET /recipes?tag=<x>` on mount and on filter change.

## V2

- Multi-tag AND/OR filtering.
- Sort options (most recent, alphabetical, most cooked).
