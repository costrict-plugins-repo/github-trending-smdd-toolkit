----
name: recipe-detail
type: ui page
----

# recipe-detail

Route: `/recipes/<id>`.

Shows one full recipe: title, tag chips, ingredient list, step-by-step instructions.

## Actions

- "Edit" opens an inline edit mode; fields become editable, "Save" and "Cancel" replace the action row.
- "Delete" prompts a confirm, then deletes and navigates back to `/`.

## Pairs with

`docs/backend/apis/recipes.md` — this page calls `GET /recipes/<id>`, `POST /recipes/<id>` (edit), `POST /recipes/<id>/delete`.

## V2

- Cook-mode view (large text, step-by-step, screen wakelock).
- Image gallery per recipe.
