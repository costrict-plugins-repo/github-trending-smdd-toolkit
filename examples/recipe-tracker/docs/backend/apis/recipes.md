----
name: recipes api
type: api
----

# recipes

All CRUD for recipes. Single router, mounted at `/recipes`.

## Endpoints

- `GET /recipes?tag=<optional>` — list all recipes. If `tag` provided, filter to recipes whose `tags` array contains that tag. Returns `list[Recipe]`.
- `GET /recipes/<id>` — one recipe. Returns `Recipe`.
- `POST /recipes` — create. Body is `Recipe` without `id`. Returns created `Recipe` with `id`.
- `POST /recipes/<id>` — edit. Body is full `Recipe`. Returns updated `Recipe`.
- `POST /recipes/<id>/delete` — delete. Returns `{"ok": true}`.

Only `GET` and `POST` methods per the project's CLAUDE.md. No `PUT`/`DELETE`/`PATCH`.

## Pairs with

- `docs/backend/schemas/recipe.md` — the `Recipe` type used in all request/response shapes.
- `docs/ui/pages/recipe-list.md` and `docs/ui/pages/recipe-detail.md` — the consumers.

## V2

- `GET /recipes/search?q=<text>` — full-text search over title, ingredients, steps.
